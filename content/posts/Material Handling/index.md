+++
date = '2025-11-20T17:28:03-06:00'
draft = false
title = 'Virtual Pick-to-Light Rack: From Scratch to Production'
+++


One project I was led involved creating a virtual version of a pick-to-light system. Essentially, a multi-shelf rack with rows of bins also had addressable LEDs along the face of each shelf. LEDs were assigned to each bin and could be addressed just by passing the bin number to the correct controller. The setup process allowed for dynamic bin sizes and gaps (relative to a quantity of leds). But the accompanying tool used to visualize this on-screen assumed uniform bin and gap sizes. The difference between setup UI and display UI led me to recreate that tool. 

I've recreated (the recreation) that you can play with below. The rack configuration and bin dimensions json may seem a bit strange as text input. For the project, the configuration was built and maintained elsewhere, then imported on app startup (as json). So I've tried to replicate that faithfully, here. 

{{< picktolight >}}

