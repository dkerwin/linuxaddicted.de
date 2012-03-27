---
layout: post
title: "Postfix Relay based on OpenLDAP"
date: 2008-10-27 14:01
comments: true
categories:
- ldap
- postfix
- howto
---
This howto describes the configuration of a Postfix relay based on OpenLDAP. These are the features of the relay:

  * User verification through LDAP
  * Separation on internal and external users
  * Greylisting
  * User Relaying based on TLS and LDAP authentication
  * Forwarding to local mailserver

<!--more-->

{% codeblock /etc/postfix/main.cf lang:bash %}
#####################################################################
## Global settings
#####################################################################
queue_directory = /var/spool/postfix
command_directory = /usr/sbin
daemon_directory = /usr/lib/postfix
mail_owner = postfix
smtpd_banner = mail.company.com ESMTP MailRelay
sendmail_path = /usr/sbin/sendmail
newaliases_path = /usr/bin/newaliases
mailq_path = /usr/bin/mailq
setgid_group = postdrop
html_directory = /usr/share/doc/postfix-2.3.6/html
manpage_directory = /usr/share/man
sample_directory = /etc/postfix
readme_directory = /usr/share/doc/postfix-2.3.6/readme
home_mailbox = .maildir/
alias_database = hash:/etc/mail/aliases

#####################################################################
## Internet and domain names
#####################################################################
myhostname = mail.company.com
mydomain = company.com

#####################################################################
## Sending mail (local)
#####################################################################
myorigin = company.com

#####################################################################
## Receiving mail
#####################################################################
inet_interfaces = all
proxy_interfaces =
mydestination =
local_recipient_maps =

## As incoming mail doesn't terminate on the relay we have to remove the values from "mydestination".
## That's why "local_recipients" is unset as well.

#####################################################################
## Relay control
#####################################################################
relay_domains = company.com
relay_recipient_maps = proxy:ldap:/etc/postfix/ldap/relay_recipients.cf
mynetworks = 192.168.1.0/28

## The relay_recipient_map contains the users that mail should be relayed for. It's not a static list as you may have used it before. 
## It contains the definition where and how to find valid users within the LDAP directory.
## The parameter "mynetworks" contains the network of the internal mailservers. There are better (and more secure) ways to do this but for not it's sufficient.

#####################################################################
## Mail transport
#####################################################################
transport_maps = hash:/etc/postfix/transport

## Transport maps define the target system based on the domain.

#####################################################################
## SASL configuration
#####################################################################
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
smtpd_sasl_local_domain = $myhostname
broken_sasl_auth_clients = yes
smtpd_sasl_auth_clients = yes

#####################################################################
## TLS configuration
#####################################################################
smtpd_use_tls = yes
smtpd_tls_auth_only = yes
smtpd_tls_loglevel = 1

smtpd_tls_CAfile = /etc/postfix/ssl/company-bundle.crt
smtpd_tls_key_file = /etc/postfix/ssl/mail.company.com.key.pem
smtpd_tls_cert_file = /etc/postfix/ssl/mail.company.com.cert.pem

## The SASL configuration should be pretty self explanatory. It's madatory that "smtpd_sasl_security_options" are set to "noanonymous". 
## If not it would be possible to logon anonymously. This settings force TLS encryption for every login. No TLS -&gt; No Login. 
## The creation and maintenance of SSL certificates is not covered by this howto.

#####################################################################
## SMTP restrictions
#####################################################################
smtpd_recipient_restrictions = permit_sasl_authenticated, permit_mynetworks, check_policy_service unix:private/postgrey, reject_unauth_destination, permit
{% endcodeblock %}

These parameters declare what has to be accomplished that mail is accepted:

  * Sucessful SASL logon
  * Mail comes from a trusted network
  * Postgrey is happy and agreed

{% codeblock /etc/postfix/master.cf lang:bash %}

## Disable local transport...
#local     unix  -       n       n       -       -       local
{% endcodeblock %}

{% codeblock /etc/postfix/transport %}
company.com        smtp:[mailgw-int.company.com]
{% endcodeblock %}

This map defines the target ``mailserver mailgw-int.company.com`` for the domain ``company.com``.
The square brackets skip the DNS MX check for every delivery.

Don't forget to convert the map after editing:

{% codeblock Update transport map %}
# postmap hash:/etc/postfix/transport
{% endcodeblock %}

{% codeblock /etc/postfix/ldap/relay_recipients.cf lang:bash %}
bind             = yes
bind_dn          = cn=SMTP Lookup,dc=company,dc=com
bind_pw          = THISisABSOLUTLYsecret
server_host      = ldap://1.2.3.4
search_base      = ou=mail,o=datacenter,c=de,dc=company,dc=com
query_filter     = (&(maildrop=%s)(destinationIndicator=external))
result_attribute = uid
version          = 3
{% endcodeblock %}

Should be pretty easy to understand. The most important part is the query filter. I use a combination of maildrop and destinationIndicator. This makes in pretty easy to seperate innternl from external users. This makes it possible to protect internal mailgroups from external access. "%s" will be replaced by the recipients address.

{% codeblock Example LDAP entry %}
dn: uid=admin,ou=mail,o=datacenter,c=de,dc=company,dc=com
cn: Admin
destinationIndicator: external
gidNumber: 505
givenName: admin
homeDirectory: /var/spool/mail/admin
mail: admin@company.com
mailbox: /var/spool/mail/admin/Maildir
maildrop: admin@company.com
maildrop: postmaster@company.com
maildrop: abuse@company.com
objectClass: CourierMailAlias
objectClass: CourierMailAccount
objectClass: inetOrgPerson
sn: admin
uid: admin
uidNumber: 505
userPassword: {CRYPT}AbCdEfGhIjKlMnOpQrStUvWxYz
{% endcodeblock %}

The destinationIndicator declares if the user can receive external mails. All maildrop lines are aliases.

