--- 
layout: post
title: "Useful BASH setting: Ignore duplicate commands"
tags: 
- bash
- Linux
- MacOSX
status: publish
type: post
published: true
meta: 
  _edit_lock: "1228933364"
  _edit_last: "1"
---
There are many settings that make BASH even more usable. As many people doesn't seem to know this particular parameter i post it here:

{% codeblock lang:bash %}
HISTCONTROL="ignoredups"
{% endcodeblock %}

This setting in bashrc or profile makes BASH ignore duplicate commands when searching the history. It's pretty useful if you had entered the same command several times.

