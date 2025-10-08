+++
date = '2025-10-07T14:55:03-05:00'
draft = false
title = 'Hello + What You’ll Find Here'
tags = ["manufacturing", "MES", "MQTT", "computer-vision", "containers", "opnsense"]
+++

I help create manufacturing software that connects machines, people, and data to keep production flowing with minimal drama. This site will be a collection of things that worked, dead ends I wish I'd skipped, and anything personally relevant. Note that this is a static site. Posts are plain markdown with code, logs, and step-by-step procedures. I will try to incorporate visuals and interactive elements as possible.

## What to expect

- **Traceable data.** Schemas, time handling, and queries that survive audits.
- **Vision in the loop.** From “model trains” to “operators trust it,” plus monitoring and drift.
- **Events over polling.** Message design, versioned payloads, and retention strategies.
- **Operable containers.** Service layout, health checks, upgrades, and repeatable deploys.
- **Home lab as testbed.** Networks and automation that behave like small factories.
- **Energy notes.** Back-of-envelope models grounded in thermodynamics.


## What to expect
- **Manufacturing data.** Tulip/MES patterns, SQL for traceability, and connector configuration.
- **Vision in production.** Some of my notes about vision systems on the manufacturing floor
- **Event-driven OT/IT.** MQTT topic design, historians, and push vs. pull.
- **Containers.** Docker/Podman/Quadlet for reproducible services and reliable operation.
- **Home lab.** Local-first home automation, OPNsense setup / management, and OpenWRT.
- **Energy.** Back-of-envelope models grounded in thermodynamics.


For work, I typically work with **copy-pasteable configs**, **small scripts** (Go/Python/JS), and **post-mortems** with real metrics. Expect that to shape some of these posts. This is no cure for my forgetfulness, but hopefully some practical guides will help others as well. 

## Upcoming posts
- A “virtual pick-to-light” tool with SVG + MQTT control
- landing.ai vs Azure Custom Vision: training time, lighting, and false positives
- Rootless Podman + my learning to leverage systemd/Quadlet containers 
- MQTT topics for shop-floor events (UNS-style) with schema examples
- Heat-pump efficiency vs gas furnaces

## Ground rules
- Everything here is **my own opinion**. 
- There is no client data anywhere near these posts. 
- Everything is re-engineered from scratch.  
- Code and configs will favor **safety, observability, and rollback**.

If you want new posts, subscribe via RSS. If your interests overlap or would like to inquire about anything, please reach out or open an issue on the repo.
