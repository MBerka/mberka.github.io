---
layout: post
title:  "Blocking everything in Firefox"
date:   2019-02-15 10:23:57 -0800
permalink: /firefox-content-blocking
categories: software
---

Firefox enjoys many of the same content-blocking options as Chrome. The following can be found in the regular privacy settings (about:preferences#privacy):
* cookies / site data;
* trackers,
* location tracking,
* camera and microphone,
* notifications,
* popups and autoplay sound
All of these include exception lists. The list is strictly practical, and lacks certain important types of content, some long included in Chrome:
* images,
* JavaScript,
* autoplaying video.
While fewer users may want to restrict these key site components, the need arises. All can be handled to varying degrees in Firefox's about:config,

# Images

A search for "image" turns up **[permissions.default.image](http://kb.mozillazine.org/Permissions.default.image)**
* 1: Allow all images to load (default)
* 2: Block all
* 3: Block third-party images (a valuable equivalent to blocking 3rd-party cookies).

# JavaScript

Globally, JavaScript can be disabled by changing **javascript.enabled** to false. The lack of built-in site-by-site settings has been discussed and is [mostly addressed with extensions](https://support.mozilla.org/en-US/questions/954712).

# Autoplay video
This turns up in the "media" class. Disabling autoplay can involve changing several settings:
* By default, **media.autoplay.default** = 0 (allow). Changing to 1 will block and 2 will require user permission.
* Of course, [there's more to it](https://support.mozilla.org/en-US/questions/1238033). If you select 2, set **media.autoplay.ask-permission** to true to enable the prompts, and **media.autoplay.enabled.user-gestures-needed** to true to require a click or other "gesture" for a video to start.

<!-- <img alt="Text" border="1" src="/assets/firefox.jpg" style="margin: 10px auto;" /><br/> -->
