--- 
layout: post
title: Inbound Traffic Shaping made (pretty) easy
tags: 
- Howto
- Kernel
- Linux
- QoS
status: publish
type: post
published: true
meta: 
  _edit_lock: "1233165042"
  _edit_last: "2"
---
It's not that easy to get trafic shaping done with Linux especially when it comes to inbound traffic. After some serious research i found this [howto](http://www.ibiblio.org/pub/Linux/docs/HOWTO/other-formats/html_single/ADSL-Bandwidth-Management-HOWTO.html) which is a excellent starting point. The supplied script can be easily customized and works pretty good. If there's interest i can post my version for incoming traffic only.

You have to apply the IMQ patch (can be found [here](http://www.linuximq.net/patches.html)) to your kernel to get this working. IMQ is necessary because Linux can only limit outgoing traffic with builtin kernel settings.

