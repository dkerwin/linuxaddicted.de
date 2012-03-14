--- 
layout: post
title: "Gentoo: Unmerging software including configs and data"
tags: 
- Gentoo
- Linux
- Portage
status: publish
type: post
published: true
meta: 
  _edit_lock: "1228933356"
  _edit_last: "1"
---
If you're unmerging software in Gentoo some files stay on your server. This is a result of the setting CONFIG_PROTECT. 
To unmerge a package completely use this command:

{% codeblock lang:bash %}
CONFIG_PROTECT="" emerge -C [SOFTWARE]
{% endcodeblock %}

