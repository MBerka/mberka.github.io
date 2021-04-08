---
layout: post
title:  "Toggl Desktop on Ubuntu 18"
date:   2018-10-27 11:49:00 -0700
permalink: /toggl-desktop-ubuntu
categories: software
---
Toggl makes a particular effort to bring its time-tracking platform to a variety of operating systems. This includes a [Linux desktop client][linux-desktop]. As of this writing, the page refers to Ubuntu 16.04, and recommends additional dependencies.

Testing version 7.4.122 of the [linked 64-bit Debian package]{https://toggl.com/api/v8/installer?app=td&platform=deb64&channel=stable}, I confirmed that installation does not appear to go anywhere on 18.04, with the installation process exiting immediately after download.

# Manual dependencies

Following Toggl's advice, manually libgstreamer installation continues to work:

{% highlight bash %}
wget http://fr.archive.ubuntu.com/ubuntu/pool/main/g/gst-plugins-base0.10/libgstreamer-plugins-base0.10-0_0.10.36-1_amd64.deb
wget http://fr.archive.ubuntu.com/ubuntu/pool/universe/g/gstreamer0.10/libgstreamer0.10-0_0.10.36-1.5ubuntu1_amd64.deb
sudo dpkg -i libgstreamer*.deb

{% endhighlight %}

After that, Ubuntu software installed the package without issue:

<img alt="Toggl package in Ubuntu Software" border="1" src="/assets/toggl/TogglInstalled.jpg" style="margin: 10px auto;" /><br/>

# Desktop features: all working!

The Toggl desktop can then be found in search and added to favorites. On launch, a small window invites the user to sign in:

<img alt="Toggl client sign-in screen" border="1" src="/assets/toggl/TogglSignIn.jpg" style="margin: 10px auto;" />

There is clearly a built-in browser, with Google Sign-In running smoothly. Past sign-in, the interface matches the [screenshots][linux-desktop]. 10 minutes later, the ''Reminder to track time'' feature also revealed itself, as a popup-inside-a-window:

<img alt="Toggl tracking reminder" border="1" src="/assets/toggl/TogglReminder.jpg" style="margin: 10px auto;" />

The Pomodoro timer also integrates reasonably. Messages are created as separate windows:

<img alt="Toggl Pomodoro break popup" border="1" src="/assets/toggl/TogglPomodoro.jpg" style="margin: 10px auto;" />

Break status is shown by changing the active project (which does get added to the project history).

<img alt="Toggl during Pomodoro break" border="1" src="/assets/toggl/TogglPomodoroBreak.jpg" style="margin: 10px auto;" />

Overall, much easier to access and remember. As a final feature not listed on the site, the desktop app adds the gray/red Toggl indicator as a persistent icon to the right side of the topbar, providing the same options as the window dropdown. Timers thus continue to run if the main window is closed, and starting/stopping/continuing a timer from the icon reopens the window.

<img alt="Persistent Toggl icon" border="1" src="/assets/toggl/TogglToolbar.jpg" style="margin: 10px auto;" />


[linux-desktop]: https://support.toggl.com/toggl-desktop-for-linux/
