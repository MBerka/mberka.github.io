---
layout: post
title:  "Johnny-Five without Arduino: Flashing Firmata with PlatformIO"
date:   2019-07-25 22:54:01 -0700
permalink: /johnny-five-flashing-firmata-with-platformio
categories: software, electronics
---

I recently found myself helping with [PDXNode's]{http://pdxnode.org/} freestyle [Robot Talent Show hack night]{https://www.meetup.com/pdxnode/events/ngpncpyzkbhc/} ([notes](https://github.com/PDXNode/pdxnode/blob/master/archive/2019-jul-hacknight/index.md)) for [NodeBots Day]{https://nodebots.io/}. We needed to set up a stack of Arduino Unos for use with [Johnny-Five]{http://johnny-five.io/}, so participants could integrate hardware into their laptop-based Node.js projects.

Unfortunately, the first step in related instructions is generally to flash the [Standard Firmata Plus firmware]{https://github.com/firmata/arduino} from the Arduino IDE, and we were several full stack developers all using Linux, finding advice for [uploading from Arduino IDE over USB](https://askubuntu.com/questions/786367/setting-up-arduino-uno-ide-on-ubuntu) ineffective. Despite installing different versions, running with different permissions and setting `dialout` groups, programming this way was not progressing.

Fortunately, [Firmata is available]{https://platformio.org/lib/show/307/Firmata/installation} on [PlatformIO]{https://platformio.org/platformio-ide}, and thus in IDEs like Atom and Visual Studio Code. Once the right board-specific configurations were figured out, all our boards got uploads in 5 minutes.

# Steps

- Create a PlaformIO project.
- Run `platformio lib install "Firmata"` in the terminal. The firmware also depends on Servo for motor control.
- Add a platformio.ini configuration file, adjusted to your board - the following worked for Uno and Nano boards:

```
[env:nano]
platform = atmelavr
board = nanoatmega328
framework = arduino

lib_deps =
    Firmata
    Servo

[env:uno]
platform = atmelavr
board = uno
framework = arduino

lib_deps =
    Firmata
    Servo
```
- Copy StandardFirmataPlus.ino from the [examples tab]{https://platformio.org/lib/show/307/Firmata/installation}. In the case of VS Code, the IDE will complain less if the file is renamed to .cpp, no code changes required.
- Once the code is in order, connect Arduino to flash, then **PlatformIO > Run Task > Upload (Uno)** (or other environment matching the board).

# Repository

A ready PlatformIO project is available for easy flashing in the **firmata** directory of my [repository of code prepared for the NodeBots workshop](https://github.com/MBerka/nodebots-firmata-platformio). Good luck!
