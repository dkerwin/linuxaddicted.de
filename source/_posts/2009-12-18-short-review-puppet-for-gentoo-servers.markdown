--- 
layout: post
title: "Short review: Puppet for Gentoo servers"
tags: 
- Gentoo
- Howto
- Linux
- puppet
- ruby
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _edit_lock: "1268939863"
  jd_tweet_this: "yes"
---
I recently started to integrate [Puppet](http://reductivelabs.com/trac/puppet/) with my company's OS installer to build custom Gentoo servers in almost no time. The install/build system reached a stable state and i want to share some information's on what i did to get it working. The Gentoo support of puppet is not perfect but sufficient for my use case. There are still some issues to solve:

  * Slots don't work (I'm working on a patch for the portage provider to address this issue)
  * No nice way to manage /etc/conf.d/net
  * Only the runlevel "default" can be managed (This is sufficient for most cases)
  * No built in USE flag support (i use a binhost so this doesn't really affect my setup). Check this [site](http://log.onthebrink.de/2008/05/using-puppet-on-gentoo.html) for a possible solution

The missing slots integration is especially important when it comes to Tomcat. Tomcat requires sun-jdk-1.5 and sun-jdk-1.6. I solved this by adding sun-jdk-1.5 to our install image. Apart from this problems it works very well.<!--more-->

## Binhost setup
The easiest way to ensure that all systems run the same software (version, use flags) is to setup a portage binhost and force all clients to use this server as only package source. There are a lot of howto's out there on creating a binhost so i won't explain it detail. To force all clients to use only binary packages set the following statements in make.conf (Puppet distribution!):

{% codeblock lang:bash %}
PORTAGE_BINHOST="http://192.168.1.1/x86_64"
EMERGE_DEFAULT_OPTS="--getbinpkgonly --usepkgonly"
{% endcodeblock %}

The parameter EMERGE_DEFAULT_OPTS is important because Puppet will run the command "emerge xxx/yyy" so you can't specify extra parameters. This parameter setup ensures that the package cannot be installed when the binary package is missing on the binhost.

