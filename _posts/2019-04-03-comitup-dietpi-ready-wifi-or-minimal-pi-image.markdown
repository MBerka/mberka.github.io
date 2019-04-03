---
layout: post
title:  "Comitup and DietPi - ready WiFi provisioning portal or minimal Pi image, pick one"
date:   2019-04-03 09:01:00 -0700
permalink: /comitup-dietpi-ready-wifi-or-minimal-pi-image
categories: software, electronics
---

The Raspberry Pi is an economical choice for IoT systems requiring solid processing power. Many projects rely on deploying such power
* in multiple remote locations;
* relying on unfamiliar WiFi networks;
* with the help of installers who may not be able to modify files on the SD card or set up an SSH connection; and
* at minimal cost (not sending a keyboard and monitor with every device).

In this context, just getting a headless device connected is one of the most fundamental challenges. The person able to provide the credentials may have no practical way to enter them. If only they could set up via the most familiar inter-computer system, the web browser. [Captive portals]{https://en.wikipedia.org/wiki/Captive_portal} are IoT's standard solution.

# Comitup - "WiFiManager" on Raspbian

<!-- <img alt="Comitup" src="/assets/toggl/TogglInstalled.jpg" style="margin: 10px auto;" /><br/> -->

Enter [Comitup](https://davesteele.github.io/comitup/) (Com-it-up?), the Pi answer to the ESP8266's [WiFi Manager](https://github.com/tzapu/WiFiManager/). It follows the same general routine:

1. Check for WiFi networks.
2. If there are no saved credentials for any, or if the credentials do not work, create a wireless access point (e.g., *comitup-1234*).
3. The user connects to the network and goes to the portal (*comitup-1234.local*).
4. Display a list of networks. The user chooses one, enters a password if needed and confirms.
5. Save the credentials and attempt to the network. If unsuccessful, return to 2.

If this is your only requirement and you are otherwise satisfied with Raspbian, you can use the ready images advertised on the site, **[using this tutorial](https://github.com/davesteele/comitup/wiki/Tutorial)**. They boast a nice connection interface and have worked well on several devices over the last few months.

# DietPi: Keeping it lean

Idealistically, I wanted to combine this with one of the simplest, most ruthlessly headless-optimized operating systems available - [DietPi][dietpi].

**Download DietPi** from [the site][dietpi] (also available for other Pi-like single-board computers).

**Flash the image to an SD card**. Raspberry Pi [recommends](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) a cross-platform tool called [Etcher]({https://www.balena.io/etcher/). With DietPi's image under 660 MB, flashing took under 30 seconds.

**Configure DietPi**
DietPi provides substantial system configuration via textfiles in the boot partition, but they are also easy to misconfigure when setting up WiFi-only headless. I encountered the same device-finding issues as [Michał Ciesielski](http://ciesie.com/setting_pihole.html). A crucial one is **dietpi-wifi.txt** (since you need an internet connection in order to give it the software to get its own connection in the future...). Fill in at least `aWIFI_SSID[0]` and `aWIFI_KEY[0]`.

Also, DietPi Automation via **dietpi.txt** provides [default software installations, account setup and substantial hardware/utility configuration](https://github.com/Fourdee/DietPi/blob/master/dietpi.txt) that you may want to take advantage of. I would have tried this with Comitup if it was an option, but Docker and major languages are useful too. With **Avahi** (`AUTO_SETUP_INSTALL_SOFTWARE_ID=152`) installed, it was time to launch.

## Start up
**And sign in over SSH.**

I was happy with the process up to this point. Getting Comitup working became a sticking point; the following standard steps were followed but were not enough by themselves. They are only included in case someone wants to push further.

**Install Comitup**. While it is possible to just
{% highlight bash %}sudo apt-get install comitup{% endhighlight %}
Comitup is under active development. Following the [instructions for the internal repository](https://davesteele.github.io/comitup/ppa.html)

{% highlight bash %}
echo "deb http://davesteele.github.io/comitup/repo comitup main" | sudo tee -a /etc/apt/sources.list
sudo apt-key add key-366150CE.pub.txt
sudo apt-get update
sudo apt-get install comitup
{% endhighlight %}

At this point, I found that Comitup conflicts with the standard Debian WiFi configuration, and could not set up a configuration such that Comitup would run on DietPi. It proved more efficient to go with the regular Comitup image, despite the full Raspbian that came with it. Regardless, these are both great projects; if anyone manages to combine them or finds another provisioning portal for DietPi, please let me know!

[dietpi]: https://dietpi.com/
