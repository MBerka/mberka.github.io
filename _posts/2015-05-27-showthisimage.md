---
layout: post
title:  "ShowThisImage"
date:   2015-05-27 10:48:00 -0600
categories: chrome-content-settings
---
**[Download from the Chrome Web Store](https://chrome.google.com/webstore/detail/show-this-image/maihkkiapahkbejpojgcioffhmcfnbpc)**

ShowThisImage is a Chrome browser extension that loads a selected image on a page with images disabled.
# How to use
## Setup
1. Download **[ShowThisImage](https://chrome.google.com/webstore/detail/show-this-image/maihkkiapahkbejpojgcioffhmcfnbpc)** from the web store.
2. To see its default effect, make sure images are disabled on at least one page:
  - Disable all images at chrome://settings/content, or
  - Disable images on a specific site at chrome://settings/contentExceptions#images or by going to that page and using the Chrome options in the toolbar, or
  - Make a temporary change using [Temporary Content Settings]({% post_url 2015-05-27-temporary-content-settings %}).
3. Adjust the CSS in the extension options (chrome://extensions/). Default CSS and instructions are included.

# Understanding the default styling
A page with images disabled normally loads with blank spaces. Unless there is other styling on the page giving the images borders, backgrounds, or dimensions, you may have no way of knowing they are present on the page. ShowThisImage applies customizable styling to hidden images: by default, they become colored boxes (usually within less than a second of the page's loading) with minimum dimensions large enough to click on. Background images, which can be much larger, are instead marked by thin, colored outlines. Small, matching image boxes are added in the corners of these outlines.

# Showing images
Right-click on an image box (regular or in the upper left corner of a background image's space) to see the ShowThisImage option, marked by the extension's icon. Click the option to have the image loaded.

# Issues?
See the [support]({% post_url 2015-06-16-support %}) page.
