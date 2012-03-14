---
layout: post
title: "OpenVPN active/active cluster"
date: 2010-02-01 20:49
comments: true
categories: keepalived openvpv network security gentoo
---
This is a small howto explaining how to run a active/active Cluster (keeplaived) setup with OpenVPN. The active/active reflects that both cluster nodes run the same OpenVPN instance. In server mode this setup leads to routing problems as both nodes have the tunnel route added during startup (not after connect). This results in routing trouble as i needed the passive node to access the VPN tunnel via the active node. This is how i solved it:
<!--more-->

## Routing setup

Both firewall nodes have a static route which forwards tunnel traffic to one of the internal cluster IP's. The metric for this route is 2 so a active tunnel is preferred over that static route.

{% codeblock lang:bash %}
routes_eth2=( "192.168.44.0/24 via 192.168.20.254 metric 2" )
{% endcodeblock %}

It doesn't matter which node is active and gets the VPN connects as the other node has the right routing entries.

## OpenVPN configuration

This setup requires the use of sudo or to run the OpenVPN daemon as root (sudo, sudo, sudo!!!). First we disable to automatically adding of routes and specify scripts for client-connect and client-disconnect. Dont forget to set script security.

{% codeblock lang:bash %}
route-noexec
client-connect /etc/openvpn/cluster_routing.sh
client-disconnect /etc/openvpn/cluster_routing.sh
script-security 3
{% endcodeblock %}

## cluster_routing.sh

This is a simple version of the script but it should be sufficient to work in most scenarios.

{% codeblock lang:bash %}
#!/bin/bash

## This is useful for debugging and to get the available env vars
##exec > /tmp/ovpn.debug.$$ 2Y&1; set -x

if [ "${script_type}" == "client-connect" ];
then
    /usr/bin/sudo /sbin/route add -net ${route_network_1}/24 gw ${route_vpn_gateway}
else
    /usr/bin/sudo /sbin/route del -net ${route_network_1}/24 gw ${route_vpn_gateway}
fi

exit 0
{% endcodeblock %}

This script adds and removes the needed route to get a operational tunnel.
