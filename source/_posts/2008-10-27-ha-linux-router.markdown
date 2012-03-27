---
layout: post
title: "HA Linux Router"
tags:
- keepalived
- cluster
- ha
date: 2008-10-27 10:36
type: post
comments: true
---
This howto describes the setup of a HA Linux router based on Gentoo and Keepalived. I'm writing this because there's not really a good documentation on this topic so far. At least as i searched for it.

### Requirements

The intended router requires this config and tools:

  * Kernel with activcated VLAN support(CONFIG_VLAN_8021Q=y)
  * Keepalived installed
  * vconfig installed
  * Optionally bonding support in Kernel and ifenslave installed

### Network Configuration

This configuration example is designed for 8 NIC's and 20 VLAN's. The following config is split to make it more readable but belongs completely to /etc/conf.d/net.

**VLAN-Interface-Mapping**

*Depending on your network and traffic you have to find a VLAN-interface-mapping that matches your environment.*

{% codeblock /etc/conf.d/net - part 1 lang:bash %}

#######################################################
## VLAN <--> Interface Mapping
#######################################################

## eth0: VLAN 20 - 22
vlans_eth0="20 21 22"

## eth1: VLAN 22
vlans_eth1="22"

## eth2: VLAN 23 - 24
vlans_eth2="23 24"

## eth3: VLAN 25 26 27 28
vlans_eth3="25 26 27 28"

## eth4: VLAN 29
vlans_eth4="29"

## eth5: VLAN 30 - 34
vlans_eth5="30 31 32 33 34"

## eth6: VLAN 35 - 38 
vlans_eth6="35 36 37 38"

## eth7: VLAN 39 - 40
vlans_eth7="39 40"
{% endcodeblock %}

**VLAN Settings**

This  VLAN setup will lead to interfaces named vlanXX. See the manpage of vconfig if you prefer a different setup. Then it's time to disable the "parent interfaces". You can't use a interface in mixed mode: VLAN's **or** single interface.

{% codeblock /etc/conf.d/net - part 2 lang:bash %}
#######################################################
## VLAN Interface naming scheme
#######################################################

vconfig_eth0=( "set_name_type VLAN_PLUS_VID_NO_PAD" )
vconfig_eth1=( "set_name_type VLAN_PLUS_VID_NO_PAD" )
vconfig_eth2=( "set_name_type VLAN_PLUS_VID_NO_PAD" )
vconfig_eth3=( "set_name_type VLAN_PLUS_VID_NO_PAD" )
vconfig_eth4=( "set_name_type VLAN_PLUS_VID_NO_PAD" )
vconfig_eth5=( "set_name_type VLAN_PLUS_VID_NO_PAD" )
vconfig_eth6=( "set_name_type VLAN_PLUS_VID_NO_PAD" )
vconfig_eth7=( "set_name_type VLAN_PLUS_VID_NO_PAD" )

#######################################################
## Disable interfaces for "normal" use
#######################################################

config_eth0=( "null" )
config_eth1=( "null" )
config_eth2=( "null" )
config_eth3=( "null" )
config_eth4=( "null" )
config_eth5=( "null" )
config_eth6=( "null" )
config_eth7=( "null" )
{% endcodeblock %}

**IP Adresses**

Now it's time to assign addresses to our VLAN interfaces. I myself prefer the last 3 adresses of every subnet as router addresses.

{% codeblock /etc/conf.d/net - part 3 lang:bash %}
-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-
-            192.168.45.0/25
-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-
- Router-VIP  ==> 192.168.45.254 Cluster IP
- Router-A    ==> 192.168.45.253 Real-IP Node A
- Router-B    ==> 192.168.45.252 Real-IP Node B

config_vlan20=( "10.1.20.0/24" )
config_vlan21=( "10.1.21.0/24" )
config_vlan22=( "10.1.22.0/24" )
config_vlan23=( "10.1.23.0/24" )
config_vlan24=( "10.1.24.0/24" )
config_vlan25=( "10.1.25.0/24" )
config_vlan26=( "10.1.26.0/24" )
config_vlan27=( "10.1.27.0/24" )
config_vlan28=( "10.1.28.0/24" )
config_vlan29=( "10.1.29.0/24" )
config_vlan30=( "10.1.30.0/24" )
...
{% endcodeblock %}

**Routing**

If you're familiar with Gentoo's routing syntax you shouldn't be surprised to see how it works.

{% codeblock /etc/conf.d/net - part 4 lang:bash %}
routes_vlan21=("192.168.99.0/27 via 10.1.21.5")
routes_vlan31=("default via 10.1.31.1")
{% endcodeblock %}

### Keepalived Configuration ###

{% codeblock /etc/keepalived/keepalived.conf - MASTER lang:bash %}
## Unique identifier for every router
global_defs {
   router_id router-a
}

## Sync Group
vrrp_sync_group SG_A {
  group {
          VI_21 # VLAN 21
          VI_22 # VLAN 22
          VI_23 # VLAN 23
          VI_24 # VLAN 24
          VI_25 # VLAN 25
          VI_26 # VLAN 26
          VI_27 # VLAN 27
          VI_28 # VLAN 28
          VI_29 # VLAN 29
          VI_30 # VLAN 30
          VI_31 # VLAN 31
          
          ...
        }
}

## VLAN 21
vrrp_instance VI_21 {
    interface vlan21
    state MASTER
    virtual_router_id 21
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass >FreakShow_<
    }
    virtual_ipaddress {
        10.1.21.254
    }
}

## VLAN 22
vrrp_instance VI_22 {
    interface vlan22
    state MASTER
    virtual_router_id 22
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass >FreakShow_<
    }
    virtual_ipaddress {
        10.1.22.254
    }
}

...
{% endcodeblock %}

{% codeblock /etc/keepalived/keepalived.conf - SLAVE lang:bash %}
## Unique identifier for every router
global_defs {
   router_id router-b
}

## Sync Group
vrrp_sync_group SG_B {
  group {
          VI_21 # VLAN 21
          VI_22 # VLAN 22
          VI_23 # VLAN 23
          VI_24 # VLAN 24
          VI_25 # VLAN 25
          VI_26 # VLAN 26
          VI_27 # VLAN 27
          VI_28 # VLAN 28
          VI_29 # VLAN 29
          VI_30 # VLAN 30
          VI_31 # VLAN 31
          
          ...
        }
}

## VLAN 21
vrrp_instance VI_21 {
    interface vlan21
    state SLAVE
    virtual_router_id 21
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass >FreakShow_<
    }
    virtual_ipaddress {
        10.1.21.254
    }
}

## VLAN 22
vrrp_instance VI_22 {
    interface vlan22
    state SLAVE
    virtual_router_id 22
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass >FreakShow_<
    }
    virtual_ipaddress {
        10.1.22.254
    }
}

...

{% endcodeblock %}

