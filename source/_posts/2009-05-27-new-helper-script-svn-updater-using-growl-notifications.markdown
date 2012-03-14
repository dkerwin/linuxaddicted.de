--- 
layout: post
title: "New helper script: SVN updater using Growl notifications"
tags: 
- Python
- Scripts
status: publish
type: post
published: true
meta: 
  _edit_lock: "1268941271"
  _edit_last: "1"
  jd_tweet_this: "yes"
---
This small helper script should be added to cron and checks your local working copies against the online repository. It can automatically update your working copies if a newer revision is available. The user will be informed using [Growl](http://www.growl.info/) popups. As Growl is only available for Mac users it's a Mac only tool by now.

The script currently registers with Growl using the classes from Python bindings. This is not necessary and would also be done by growlnotify. I was just interested in how this whole Growl stuff works. Feel free to remove this parts. <!--more-->

When i started i was willing to do the whole Growl stuff manually which has worked very well from the command line but not from cron. Seems like the context or something else get's lost and no notifications pop up anymore. To be able to run via cron i switched to growlnotify.

## Depencies
  * [Growl SDK (Python bindings)](http://www.growl.info/files/Growl-1.1.4-SDK.dmg)
  * [growlnotify from Growl distribution](http://growl.info/files/Growl-1.1.4.dmg)
  * [Python >= 2.5](http://www.python.org)

## Screenshots

<img class="aligncenter size-full wp-image-223" title="svn_growl_screen_1" src="http://www.linuxaddicted.de/blog/wp-content/uploads/2009/05/picture-4.png" alt="svn_growl_screen_1" width="337" height="180" />

<h2>Download</h2>

Download: <strong>[download#5]</strong>
