---
layout: post
title: "puppetmasterd and passenger"
date: 2010-03-18 19:30
comments: true
categories: howto apache passenger puppet rails ruby
---
It seems like some people have trouble configuring/undestanding how to combine puppetmasterd and Passenger (aka mod_rails). Let's get it on:

## Install depencies

You need the following components on your puppetmaster server:
	
  * Phusion Passenger ([http://modrails.com](http://modrails.com/))
  * Rack ([http://rubyforge.org/projects/rack](http://rubyforge.org/projects/rack))

## Configure puppetmaster

Your puppet package should contain a config.ru. I found mine in /usr/share/doc/puppet-0.25.4-r1/ext/rack/files/config.ru.bz2 (Gentoo).

{% codeblock lang:bash %}
mkdir /etc/puppet/rack
mkdir /etc/puppet/rack/public
cp [YOUR_CONFIG.RU] /etc/puppet/rack
chown puppet:root /etc/puppet/rack/config.ru
{% endcodeblock %}

<!--more-->

The final chown line is important! This way rack determines under which user to run the puppetmaster processes.

Add the following lines to your puppet.conf:
{% codeblock lang:bash %}
[puppetmasterd]
...
ssl_client_header = SSL_CLIENT_S_DN
ssl_client_verify_header = SSL_CLIENT_VERIFY
{% endcodeblock %}

## Apache config

You can keep your passenger config as is and modify it when required. Here's a example vhost config:

{% codeblock lang:apache%}
Listen 8140

<VirtualHost *:8140>
    ServerName puppet
    DocumentRoot /etc/puppet/rack/public/

    CustomLog "|/usr/sbin/rotatelogs /var/www/puppet/logs/access_log.%Y%m%d-%H%M 86400" common
    ErrorLog  "|/usr/sbin/rotatelogs /var/www/puppet/logs/error_log.%Y%m%d-%H%M 86400"

    PassengerHighPerformance on
    PassengerMaxPoolSize 15
    PassengerPoolIdleTime 300
    PassengerUseGlobalQueue on
    PassengerStatThrottleRate 120
    RackAutoDetect Off
    RailsAutoDetect Off

    RackBaseURI /

    SSLEngine on
    SSLProtocol -ALL +SSLv3 +TLSv1
    SSLCipherSuite ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:-LOW:-SSLv2:-EXP
    SSLCertificateFile /var/lib/puppet/ssl/certs/XXXXXXXXXXXX.pem
    SSLCertificateKeyFile /var/lib/puppet/ssl/private_keys/XXXXXXXXXXXX.pem
    SSLCertificateChainFile /var/lib/puppet/ssl/ca/ca_crt.pem
    SSLCACertificateFile /var/lib/puppet/ssl/ca/ca_crt.pem
    SSLCARevocationFile /var/lib/puppet/ssl/ca/ca_crl.pem
    SSLVerifyClient optional
    SSLVerifyDepth 1
    SSLOptions +StdEnvVars

    <Directory "/etc/puppet/rack/public/">
        Options None
        AllowOverride None

        Order allow,deny
        Allow from all
    </Directory>
</Virtualhost>
{% endcodeblock %}

Restart apache and when clients connect are are triggered via puppetrun you may see something like this with passenger-status:

{% codeblock lang:bash %}
# passenger-status
----------- General information -----------
max      = 20
count    = 9
active   = 0
inactive = 9
Waiting on global queue: 0

----------- Domains -----------
/etc/puppet/rack:
  PID: 19160   Sessions: 0    Processed: 39      Uptime: 20s
  PID: 19202   Sessions: 0    Processed: 70      Uptime: 17s
  PID: 18934   Sessions: 0    Processed: 95      Uptime: 45s
  PID: 18977   Sessions: 0    Processed: 66      Uptime: 42s
  PID: 19008   Sessions: 0    Processed: 63      Uptime: 40s
  PID: 19184   Sessions: 0    Processed: 2       Uptime: 19s
  PID: 19103   Sessions: 0    Processed: 8       Uptime: 28s

/var/www/puppet-dashboard/htdocs:
  PID: 19158   Sessions: 0    Processed: 6       Uptime: 22s
  PID: 19236   Sessions: 0    Processed: 3       Uptime: 10s
{% endcodeblock %}

Also refer to the offical puppet documentation on passenger [here](http://docs.puppetlabs.com/guides/passenger.html)</a>.

That's it.
