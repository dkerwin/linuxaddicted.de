--- 
layout: post
title: Using Django with Komodo IDE
tags: 
- Howto
- Komodo
- MacOSX
- Python
status: publish
type: post
published: true
meta: 
  _edit_lock: "1234361448"
  _edit_last: "1"
---
I recently started to work on a new [Django](http://www.djangoproject.org/) project and tried use my default IDE: Komodo IDE. Unfortunately it didn't work out the way i wanted. Code cpmpletion didn't work just as the import of my app. Here's how i solved it:

<!--more-->
 * Add a new Python import directory: **Preferences &rarr; Languages &rarr; Python**

My Django resides in /Library/Python/2.5/site-packages. Your installation may be in a different location.
{% img center http://www.linuxaddicted.de/blog/wp-content/uploads/2009/02/komodo_prefs.png 477 307 komodo_prefs %}

 * Update your Komodo project: **Properties &rarr; Languages &rarr; Python**

My project resides in /Users/dkerwin/development/django/stag_party with several applications inside.
{% img center http://www.linuxaddicted.de/blog/wp-content/uploads/2009/02/project_import.png 414 207 project_import %}

