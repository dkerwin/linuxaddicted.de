---
layout: post
title: "BigIP client certificates"
date: 2008-12-02 11:14
comments: true
categories:
- bigip
- ssl
- security
- ldap
---
This howto will show you how to use client certificates to authenticate against applications hosted by a BigIP loadbalancer. You may find this howto (or parts of it) on [ask.f5.com](http://ask.f5.com) as they was no documentation before on this topic. I forwarded it to F5 some time ago. **This is the only way to use client certificates without purchasing addon modules from F5.**

### Purpose

If you host a website/application which should only be accessible by some users you you may have already thought about client certificates. This makes brute force attacks against login panels impossible. 
This increases the overall application security by adding another layer a user must pass.

The mapping between certificates and access rights is based on a LDAP directory.

### Overview

The loadbalancer reads the CN from the client certificate and searches the defined group for the virtual server for a matching entry. The loadbalancer verifies CA and CRL's before the lookup takes place.

{% img center /images/f5_ldap_auth.png 388 339 BigIP Certificate Auth %}

### LDAP Setup

{% codeblock Example LDAP tree %}
                          
dc=company,dc=com
   /         \
  /           \
...      ou=datacenter
           /        \
          /          \
        ...       ou=webauth
                   /    \
                  /      \
            ou=users    ou=groups
{% endcodeblock %}

#### User & Group LDIF

Here are two LDIF examples for users and groups:

{% codeblock User LDIF %}
dn: cn=Example User,ou=users,ou=webauth,ou=datacenter,dc=company,dc=com
cn: Example User
sn: Example User
objectClass: person
description: It's just a example user
{% endcodeblock %}

{% codeblock Group LDIF %}
dn: cn=Example Group,ou=groups,ou=webauth,ou=datacenter,dc=company,dc=com
cn: Example Group
objectClass: posixGroup
description: Group is allowed to connect to BigIP's VHOST secure.company.com
memberUid: cn=Example User,ou=users,ou=webauth,ou=datacenter,dc=company,dc=com
memberUid: cn=Example User No2,ou=users,ou=webauth,ou=datacenter,dc=company,dc=com
gidNumber: 145
{% endcodeblock %}

The "memberUid" has to match exactly the certificates CN because this is the key the BigIP is looking for.

### BigIP Configuration

#### Authentication Configuration

As a first step we create a new "Authentication Configuration". Just open the web frontend and navigate to:

``Local Traffic`` &rarr; ``Profiles`` &rarr; ``Authentication`` &rarr; ``Configuration``

You have to create a new config for every virtual host you want to secure. This is force by the LDAP group used because every config can only use one group. Here's a example configuration:

{% codeblock Authentication Configuration %}
Parameter        | Value
==========================================================================
Hosts            | 1.2.3.4
Search Type      | User
User Base        | DNou=users,ou=webauth,ou=datacenter,dc=company,dc=com
User Key         | cn
Admin DN         | DN for binding
Admin Password   | Bind password
Group Base DN    | ou=groups,ou=webauth,ou=datacenter,dc=company,dcc=com
Group Key        | cn
Group Member Key | memberUid
Valid Groups     | Corresponding LDAP group
{% endcodeblock %}

### Authentication Profile

The next step is to create a profile which inherits the new created config. You need a profile for every configuration. Navigate to 

``Local Traffic`` &rarr; ``Profiles`` &rarr; ``Authentication`` &rarr; ``Profiles``

{% codeblock Authentication Profile %}
Parameter      | Value
==========================================================================
Type           | SSL Client Certificate LDAP
Parent Profile | ssl_cc_ldap
Mode           | Enabled
Configuration  | NAME OF CONF JUST CREATED
{% endcodeblock %}

### CA Upload

If your CA certificate(s) have not been installed it's time to do so. 

``Local Traffic`` &rarr; ``SSL Certificates`` &rarr; ``Import``

### SSL Profiles

To use SSL (without client certificates too) you need a SSL profile. Navigate to

``Local Traffic`` &rarr; ``Profiles`` &rarr; ``SSL`` &rarr; ``Client``

Create a new profile based on "clientssl" or one of your already defined profiles.

{% codeblock SSL Client %}
Parameter                          | Value
==========================================================================
Chain                              | CompanyBundle
Trusted Certificate Authorities    | CompanyBundle
Client Certificate                 | require
Frequency                          | always
Advertised Certificate Authorities | CompanyBundle
{% endcodeblock %}

### Virtual Host Configuration

You may create a new virtual host or modify a existing host. Navigate here

``Local Traffic`` &rarr; ``Virtual Servers``

Modify the following parameters (if you already used SSL in this vhost):

{% codeblock Virtual Servers %}
Parameter               | Value
==========================================================================
SSL Profile (Client)    | Profile created earlier
Authentication Profiles | Profile created earlier
{% endcodeblock %}

### That's it!

Import your client certificates to your browser and enjoy your secure connection.

If you have any trouble don't hesitate to contact me: daniel {A_T} linuxaddicted {D_O_T} de

