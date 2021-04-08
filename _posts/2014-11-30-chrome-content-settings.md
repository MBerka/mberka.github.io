---
layout: post
title:  "Chrome content settings"
date:   2014-11-30 11:36:00 +0100
permalink: /chrome-content-settings
categories: software
---
My extensions ([Temporary Content Settings]({% post_url 2015-05-27-temporary-content-settings %}) and [ShowThisImage]({% post_url 2015-05-27-showthisimage %})) to the Chrome browser deal mostly with content settings, specifically with making it possible to set them as strict as possible while maintaining a fair online experience. Google provides a [brief explanation][explanation] - here are the details.

# Settings
Anyone can access the content settings at chrome://settings/content (Chrome browser pages cannot be accessed by direct link; type/paste their url's in the address bar, make a bookmark, or navigate through the Chrome options). The Chrome browser offers an ever-growing list of options to restrict various web features. As of version 49:

**Contents**

    1. Settings
    2. Cookies (and other files stored on your computer by websites)
    3. Global options:
    4. Local options:
    5. Images
    6. JavaScript
    7. Key generation
    8. Handlers (Protocol handlers)
    9. Plugins
    10. Pop-ups
    11. Location
    12. Notifications
    13. Fullscreen
    14. Mouse cursor
    15. Protected content
    16. Microphone
    17. Camera
    18. Unsandboxed plugin access
    19. Automatic downloads
    20. MIDI devices full control
    21. USB devices
    22. Zoom levels

Even the [explanation][explanation] has trouble keeping up with all of them (Key Generation, USB devices, and Protected content missing as of 2016/4/12). No wonder Google removed the option to change settings directly from the Permissions tab of the webpage information dropdown. Most settings are irrelevant to most pages; [Temporary Content Settings][temporary-content] focuses on widely used settings that have the biggest impact on data usage and/or privacy - Cookies, Images, JavaScript, and Plugins. This page describes those and everything else, so that you can knowledgeably change the default settings (indicated by "(recommended)" on the settings page). Google's standard explanations are quoted at the start of each section.

## Cookies (and other files stored on your computer by websites)
*"Cookies are files created by websites you've visited to store browsing information, such as your site preferences or profile information."*

The [cookie](https://support.google.com/chrome/answer/95647?hl=en) setting controls a wide variety of these files. In order of the frequency with which I encounter them, they are:

- Cookies: short text files with ID's, dates, etc., set by nearly everything, read by the server the next time you connect to it, expire eventually,
- Local storage: larger offline data, accessed by the client (browser), set by JavaScript downloaded from a website, no expiration,
- Database storage: a database on your computer, for faster data storage/retrieval,
- Application cache: data recently accessed with an application, for fast future retrieval (set by Google's built-in Chrome apps),
- Flash data, set by ads, videos, and games,
- [Channel ID](https://www.google.com/chrome/browser/privacy/whitepaper.html), apparently a way to verify that the same user client is connecting to a server (in practice, Google),
- File system: a system of directories with files (set by Google's built-in Chrome apps).

Nearly any of these can be used to identify your computer as the one that earlier visited some other page on the same site or where a piece of code from the same server was included. However, this can only happen if you keep the data and return it upon request.

### Global options:

- **Allow**: keeps cookies until they expire, possibly for years in the case of cookies, or never in the case of the other types,
- **Keep until you quit your browser**: empties the list when you close the window or the computer crashes,
- **Block**: No data can be set - access to cookies is denied.

Importantly, cookies can be set by the server sending you the page (whatever computer is behind the address) or by other servers from which that server loads data (third parties to the interaction between you and the main server). The **block third-party cookies** checkbox can be marked regardless of what you picked in the previous option. It will not usually keep you from logging in to the site you are visiting, but will interfere with social media buttons and ads.

### Local options:
The cookie section has two buttons. The first is the standard [Manage exceptions...](https://support.google.com/chrome/answer/3123708) button, offering the standard per-domain exceptions, with **Allow**, **Clear on exit**, and **Block** corresponding to the global options above. By making exceptions for sites you visit regularly, you can restrict the global options without having to log in to a cookie-based site again every time you reopen the browser (keep) or be unable to log in at all (block). Contrary to what you might think based on [Google's guide](https://support.google.com/chrome/answer/95647?hl=en), *setting a positive exception here will allow that site's code to recognize you on other sites* (effectively overriding a third-party block even if that code does not set new cookies).

The second button is **All cookies and site data...**, where you can view all the cookies currently stored in Chrome, organized by domain/subdomain (the same site may thus have multiple rows) Click on a cookie to see information about it or remove just that one part. You can also remove an entire subdomain's cookies, all the cookies appearing for a given search, or all the cookies.

## Images
*"Images are allowed by default."*

As with the explanation, this section is simpler. **Show all images** will load whatever images are in a page's source code. **Do not show any images** will show none. You have no control over whether the images being loaded are from the current site or elsewhere, only a yes/no for whether pages will have any. [ShowThisImage][showthisimage] is only able to display a selected image by switching the page setting to on, reloading that image by changing the source code, and then switching the setting back off. For site-by-site control of whether images are loaded, see the [Manage exceptions...](https://support.google.com/chrome/answer/3123708) button. It will provide you with a similar dialog to the Cookie local options, but with only Allow and Block as settings choices.

## JavaScript
*"JavaScript helps make sites more interactive."*

JavaScript is code that runs in the background of a page. A bare-bones page with HTML, CSS, and images is requested by your browser (a request for every file needed to make the page) and then assembled on your computer. Cookies are written and read, the site's server knows what files were exchanged, but nothing more will change on the page unless you click a link (which will take you to a different page, requesting new files, or to an anchor on the same page), mark a checkbox, or activate a mouse-based CSS condition (expand a menu by hovering over it). JS allows far greater interactivity. It is run by the browser, but it can contain a wide variety of instructions. JS can track where the mouse moves, what items are selected, virtually anything that the user does on a loaded page. It can send new requests to web servers and deduce a great deal about your browser settings, allowing tracking without cookies.

Still, without JS, pages will look rather static and may not even show all their content. Chrome allows it by default. You have the same limited per-page control as with Images.

## Key generation

## Handlers (Protocol handlers)
*"Chrome alows web services to ask if you’d like to use them to open certain links. For example, certain links can open an email program. If you're using Gmail or Hotmail, this setting lets the site handle links that would otherwise open a program outside your browser."*

In practice, this defines whether e.g. Gmail can ask whether you want to open a new (Gmail) window/tab open with a ready-to-write email addressed to user@example.com when clicking on a link like "mailto:user@example.com". "mailto" is by far the most common protocol requiring special handling - most web addresses start with protocols like http (hyper-text transfer protocol, http://www.example.com) or https ("secure" meaning encrypted communication). Others that may appear in the address bar include file (from your computer - file:///C:// will bring up the list of top-level files and folders on your hard drive) and view-source (like going to the following address and choosing View page source). Your browser knows how to handle all of these, and mailto is the most commonly encountered protocol apart from them.

[Other possible protocols do exist](https://developers.google.com/web/updates/2011/06/Registering-a-custom-protocol-handler), most obscure - mms, nntp, rtsp, webcal (this one can be used for Google Calendar). A site could also make up its own protocols starting with "web+", and then request to become the handler - I have never seen one go to the trouble, when users can be directed to regular web addresses.

A malicious site could become the default handler of your mailto protocol and know the address for every email link that you clicked. Then it could act as a man-in-the-middle and get the contents of any emails you typed into its fake interface, and/or send spam to the address? For good reason, the most generous setting available is to allow sites to ask. I never recall seeing a dubious request, and the possibility of abuse relies on a user placing unreasonable trust in an unfamiliar site. Still, if a browser is used by a non-savvy user or users, you might err on the side of caution and change **Allow sites to ask to become default handlers for protocols** (the default) to **Do not allow any sites to handle protocols**.

**Manage handlers...** brings up a list of all protocols for which handlers have been accepted - e.g., mailto. Each item has a dropdown list where you can choose any of the sites whose request for that protocol has been accepted, or choose "(none)". Hover over a protocol entry with a selected site to show the **remove this site** link to the right of the dropdown. Barring the scenario above, you are most likely to use this upon leaving an email provider.

## Plugins

## Pop-ups

## Location

## Notifications

## Fullscreen

## Mouse cursor

## Protected content

## Microphone

## Camera

## Unsandboxed plugin access

## Automatic downloads

## MIDI devices full control

## USB devices

## Zoom levels
*"You can set how much you zoom in to certain websites."*


[explanation]: https://support.google.com/chrome/answer/114662
[temporary-content]: {% post_url 2015-05-27-temporary-content-settings %}
[showthisimage]: {% post_url 2015-05-27-showthisimage %}
