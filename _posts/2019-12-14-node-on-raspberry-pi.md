---
layout: post
title:  "Node.js on (Raspberry) Pi: A Crash Course"
date:   2019-12-14 18:30:00 -0800
permalink: /node-on-raspberry-pi-presentation
categories: software, electronics
---

I recently delivered the following half-hour presentation at pdxnode. The description is from the [meeting summary](https://github.com/PDXNode/pdxnode/blob/master/archive/2019-dec/index.md).

* **Talk**: Node.js on (Raspberry) Pi: A Crash Course
* **Description**: A quick introduction to the hardware of a Raspberry Pi, followed by a maximally simplified procedure for getting a minimal headless Pi running Node.js without keyboard or monitor. Highlighted the [Comitup library](https://github.com/davesteele/comitup) for easy WiFi connection via captive portal.

Provided sample applications and minimal code demos for playing sound from the audio jack (mpg123 utility), sending/receiving data via a USB port (serialport module), binary input/output via GPIO pins (onoff), and communication with sensors via the 1-wire protocol (device-specific modules). Considered caveats around containerization (access to hardware), ARM architecture (software compatibility) and security (need to protect local networks).

* **Presentation link**: [NodeJS_on_Pi.pdf](https://github.com/MBerka/presentations/blob/master/node-on-pi/NodeJS_on_Pi.pdf)