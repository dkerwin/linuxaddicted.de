--- 
layout: post
title: "Portage: Patch existing Perl module ebuild by using a overlay"
tags: 
- ebuild
- Gentoo
- Howto
- Linux
- Perl
- Portage
status: publish
type: post
published: true
meta: 
  _edit_lock: "1228949735"
  _edit_last: "1"
---
You may have had the same issue as i some time ago. You install a perl module from Portage but you have to modify the module's code. Of course you don't want to patch and install manually. Assuming the module is named "foobar" here's how i solved it:

<!--more-->

1. Create a Portage Overlay (refer to the official documentation) and enable it in /etc/make.conf
2. Create a new category directory within your overlay dir (mkdir /usr/local/portage/my-ebuilds)
3. Create application directory (mkdir /usr/local/portage/my-ebuilds/foobar)
4. Copy the existing ebuild to the new app directory
5. Create a files directory and put the patch there
6. Modify the ebuild to apply your patch

{% codeblock lang:bash %}
...

# This is the magic line:
PATCHES="${FILESDIR}/my_patch.patch"

...
{% endcodeblock %}

After you finished these tasks you have to (re)generate the Manifest:

{% codeblock lang:bash %}
ebuild [OVERLAY_PATH]/my-ebuilds/foobar/foobar-1.0.0.ebuild digest
{% endcodeblock %}

That's it. If you have trouble feel free to contact me: daniel <-AT-> linuxaddicted.de





