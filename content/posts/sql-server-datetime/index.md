+++
date = '2025-10-07T14:55:03-05:00'
draft = false
title = 'SQL Server Datetime Fun'
+++


I went down a rabbit hole recently that started with user logouts not being recorded. This was one bit of functionality for a manufacturing production application. With margins being rather tight, keeping track of labor allocation and being able to reconcile that with production counts is a high priority item. 

I started troubleshooting this as soon as we discovered the issue. I tried creating an entry manually and then logging out manually but that was not working. So it's not the application, it must be the integration somehow. 

The issue here is around functionality that handles logins and logouts of users at a given machine. They enter their badge ID during the startup process, we get the User info, then pass that into the following query:

```SQL
INSERT INTO DBO.LaborTracking ( 
	[CreatedAt], 
	[UpdatedAt], 
	[JobID], 
	[User], 
	[UserName], 
	[LoginTime], 
	[LogoutTime], 
	[LastEditedBy], 
	[LastEditedReason] 
) 
VALUES ( 
	GETUTCDATE(),
	GETUTCDATE(),
	$JobID$, 
	$User$, 
	$UserName$,
	$LoginTime$, 
	CASE 
		WHEN $LogoutTime$ IS NULL 
			OR LTRIM(RTRIM($LogoutTime$)) = '' 
		THEN NULL 
		ELSE $LogoutTime$ 
	END,
	$LastEditedBy$,
	$LastEditedReason$
);

```


A note that this is connected to MS SQL Server. I setup the initial schema for the DEV server, using `datetime2(7)` stored in UTC. I don't have direct access to the PROD server so I wanted to confirm the type vs something like `datetimeoffset`. Since the database will be tied to one site in the same timezone, I didn't want to deal with `datetimeoffset`. Using that would've required turning every datetime type into a string in the platform, to then pass to the database. I also used `GETUTCDATE()` over `SYSUTCDATETIME()` I absolutely do not need that level of precision. 

## Ruling out the database

```SQL
SELECT FROM DBO.LaborTracking
WHERE JobID = $JobID$
  AND User = $User$
  AND LoginTime = CAST(
        SWITCHOFFSET(CONVERT(datetimeoffset, $LoginTime$), '+00:00')
      AS datetime2(7));

```
I tried to pass the exact datetime (as type datetime) that was returned for the login time to select my test entry. No combination of offsets worked, but passing the literal value returned back as a string did seem to work. Just to confirm that it wasn't a timezone issue or something like that, I tried this example: 

```SQL
DECLARE @dt datetimeoffset = $LoginTime$;

SELECT
    @dt AS Original,
    SWITCHOFFSET(@dt, '+00:00') AS AsUtcOffset,
    CAST(SWITCHOFFSET(@dt, '+00:00') AS datetime2(7)) AS AsUtcDateTime2;
```

All three matched exactly when I output them as strings.

| Index | Original                 | AsUtcOffset              | AsUtcDateTime2           |
| ----- | ------------------------ | ------------------------ | ------------------------ |
| 0     | 2025-09-02T17:40:00.000Z | 2025-09-02T17:40:00.000Z | 2025-09-02T17:40:00.000Z |
At this point I don't think it's the database causing the issue.

## Finding the real mismatch

Because the string worked, my suspicion was that the platform was doing something silly. I checked the insert query and noticed a change. Instead of providing a start time from the application, we just call the `GETUTCDATE()` function to initialize things. The issue was starting me in the face. The root cause: `GETUTCDATE()` rounds to the nearest second, while string input preserved milliseconds like `2025-09-02T19:58:12.743Z`.

Since most logouts are triggered for everyone on the job via a different function, this issue took a while to catch. Here's the logout by JobID query we use:  

```SQL
DECLARE
  @JobIDs      NVARCHAR(MAX) = $JobID$,  
  @EndTime     DATETIME     = $LogoutTime$,
  @Editor      NVARCHAR(100) = $LastEditedBy$,
  @Reason      NVARCHAR(200) = $LastEditedReason$;

IF @EndTime IS NULL
  SET @EndTime = GETUTCDATE();

UPDATE DBO.LaborTracking
SET
  LogoutTime   = @EndTime,
  UpdatedAt         = @EndTime,
  LastEditedBy      = @Editor,
  LastEditedReason  = @Reason
OUTPUT
  inserted.*
WHERE
  LogoutTime IS NULL
  AND JobID IN (
    SELECT LTRIM(RTRIM(value))
    FROM   STRING_SPLIT(@JobIDs, ',')
  );
```
## Fix
The problem boiled down to a precision mismatch:

- `GETUTCDATE()` only stores to the nearest second.
- The platform, however, was sending strings with millisecond precision (e.g. `2025-09-02T19:58:12.743Z`).

That meant a query looking for an _exact_ match on `LoginTime` would often fail, since the inserted value had already been rounded.

To work around this, I widened the search criteria by ±1500 ms. Why 1500? In practice the rounding difference is usually <1 second, but I added extra headroom to avoid edge cases and make the query easier to reason about.

```SQL
UPDATE DBO.LaborTracking
SET
  LoginTime   = GETUTCDATE(),
  LastEditedBy      = $LastEditedBy$,
  LastEditedReason  = $LastEditedReason$
WHERE
  JobID             = $JobID$
  AND User      = $User$
  AND UserName  = $UserName$
  AND LoginTime BETWEEN
        DATEADD(millisecond, -1500, CAST(TODATETIMEOFFSET($LoginTime$, '+00:00') AS datetime2(7)))
    AND DATEADD(millisecond,  1500, CAST(TODATETIMEOFFSET($LoginTime$, '+00:00') AS datetime2(7)));

```

This workaround solves the immediate problem, but I’d be interested in hearing if others have cleaner approaches — especially around reconciling datetime precision in SQL Server. In the future I will also be switching everything to `DATETIMEOFFSET` to avoid any ambiguity around timezones. 



