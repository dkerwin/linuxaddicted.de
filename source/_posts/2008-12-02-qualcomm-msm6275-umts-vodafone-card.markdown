---
layout: post
title: "Qualcomm MSM6275 UMTS Vodafone card"
date: 2008-12-02 13:25
comments: true
categories:
- umts
- howto
- vodafone
- linux
---
## Preferences

First of all make sure UMTS card is recognized by your system. You need PCMCIA support enabled in your kernel. If you compiled it as modules load the modules.

{% codeblock Device check %}
deathstar ~ # lspci
...
04:00.0 Network controller: Option N.V. Qualcomm MSM6275 UMTS chip
...
{% endcodeblock %}

## Nozomi - card access

Download und build the Nozomi kernel module. You can get it **[here](http://www.pharscape.org/component/option,com_forum/Itemid,69/page,viewforum/f,5/)**

{% codeblock Nozomi install %}
cd nozomi_2.21alpha_060917
mkdir -p /lib/modules/KERNEL_VERSION/kernel/drivers/pci/hotplug (if it doesn't exists)
make
depmod
modprobe nozomi
{% endcodeblock %}

After you succeeded in loading the nozomi module you should see something like this in dmesg:

{% codeblock dmesg %}
nozomi 0000:04:00.0: Nozomi driver nozomi_tty
Initializing Nozomi driver 2.21alpha (build date: Jun  9 2008 15:00:29)
nozomi 0000:04:00.0: Version of card: 3
nozomi 0000:04:00.0: Initialization OK!
{% endcodeblock %}

##PPP Configuration

You have to supply the PIN to your SIM-card. There are two ways to accomplish this:

  * Add ``AT+CPIN=MY_SIM_PIN`` to your chat script
  * Use a little Perl script (for example if you use a PIN app)

{% codeblock Perl helper lang:perl %}
#!/usr/bin/perl

use strict;
use warnings;

$SIG{ALRM} = sub { die("timeout: no response from modem $modem\n"); };

my §pin   = shift;
my $modem = '/dev/noz0';

open(MODEM, "+<", $modem) or die("Failed to open modem $modem: $!");
print(MODEM "AT+CPIN=\"$pin\"\n\r");
while (<MODEM>) {
    if (m/OK/) {
        close(MODEM);
        print("PIN accepted\n");
        exit(0);
    }
    if (m/ERROR/) {
        close(MODEM);
        print("PIN rejected\n");
        exit(1);
    }
}

exit(0);
{% endcodeblock %}

Now it's time to create the PPP config and chat scripts.

{% codeblock /etc/ppp/peers/umts %}
# Most GPRS phones don't reply to LCP echo's
lcp-echo-failure 0
lcp-echo-interval 0
# Keep pppd attached to the terminal:
# Comment this to get daemon mode pppd
nodetach
# Debug info from pppd:
# Comment this off, if you don't need more info
debug
# Connect script:
# scripts to initialize the UMTS modem and start the connection,
connect /etc/ppp/peers/umts-connect-chat
# Disconnect script:
# AT commands used to 'hangup' the UMTS connection.
disconnect /etc/ppp/peers/umts-disconnect-chat
# Serial device to which the UMTS card is connected:
/dev/noz0
# Serial port line speed
115200
# Hardware flow control:
# Use hardware flow control with cable, Bluetooth and USB but not with IrDA.
crtscts  # serial cable, Bluetooth and USB, on some occations with IrDA too
#nocrtscts # IrDA
# Ignore carrier detect signal from the modem:
local
# IP addresses:
# - accept peers idea of our local address and set address peer as 10.0.0.1
# (any address would do, since IPCP gives 0.0.0.0 to it)
# - if you use the 10. network at home or something and pppd rejects it,
# change the address to something else
0.0.0.0:0.0.0.0
# pppd must not propose any IP address to the peer!
noipdefault
# Accept peers idea of our local address
ipcp-accept-local
# Add the ppp interface as default route to the IP routing table
defaultroute
# DNS servers from the phone:
# some phones support this, some don't.
usepeerdns
# ppp compression:
# ppp compression may be used between the phone and the pppd, but the
# serial connection is usually not the bottleneck in GPRS, so the
# compression is useless (and with some phones need to disabled before
# the LCP negotiations succeed).
novj
nobsdcomp
novjccomp
nopcomp
noaccomp
# The phone is not required to authenticate:
noauth
mtu 1500
mru 1500
{% endcodeblock %}

{% codeblock /etc/ppp/peers/umts-connect-chat %}
exec chat             \
  TIMEOUT   5       \
  ECHO    ON        \
  ABORT   '\nBUSY\r'      \
  ABORT   '\nERROR\r'     \
  ABORT   '\nNO ANSWER\r'     \
  ABORT   '\nNO CARRIER\r'    \
  ABORT   '\nNO DIALTONE\r'   \
  ABORT   '\nRINGING\r\n\r\nRINGING\r'  \
  ''    \rAT        \
  TIMEOUT   12        \
  SAY   "Press CTRL-C to close the connection at any stage!"  \
  SAY   "\ndefining PDP context...\n" \
  OK    ATH       \
  OK    ATE1        \
  OK    'AT+CGDCONT=1,"IP","web.vodafone.de","",0,0'  \
  OK    ATD*99#       \
  TIMEOUT   22        \
  SAY   "\nwaiting for connect...\n"  \
  CONNECT   ""        \
  SAY   "\nConnected." \
  SAY   "\nIf the following ppp negotiations fail,\n" \
  SAY   "try restarting the phone.\n"
{% endcodeblock %}

{% codeblock /etc/ppp/peers/umts-disconnect-chat %}
exec /usr/sbin/chat -V -s -S  \
ABORT   "BUSY"    \
ABORT   "ERROR"   \
ABORT   "NO DIALTONE" \
SAY   "\nSending break to the modem\n"  \
""    "\K"    \
""    "+++ATH"  \
SAY   "\nPDP context detached\n"
{% endcodeblock %}

## Time to go online

{% codeblock Connect %}
deathstar ~ # pppd call gprs
Press CTRL-C to close the connection at any stage!
defining PDP context...
AT
OK
ATH
OK
ATE1
OK
AT+CGDCONT=1,"IP","web.vodafone.de","",0,0
OK
waiting for connect...
ATD*99#
CONNECT
Connected.
If the following ppp negotiations fail,
try restarting the phone.
Serial connection established.
using channel 1
Using interface ppp0
Connect: ppp0 <--> /dev/noz0
sent [LCP ConfReq id=0x1 <asyncmap 0x0> <magic 0x9aa17e40>]
rcvd [LCP ConfReq id=0x0 <asyncmap 0x0> <auth chap MD5> <magic 0x90ae7a2e> <pcomp> <accomp>]
No auth is possible
sent [LCP ConfRej id=0x0 <auth chap MD5> <pcomp> <accomp>]
rcvd [LCP ConfAck id=0x1 <asyncmap 0x0> <magic 0x9aa17e40>]
rcvd [LCP ConfReq id=0x1 <asyncmap 0x0> <magic 0x90ae7a2e>]
sent [LCP ConfAck id=0x1 <asyncmap 0x0> <magic 0x90ae7a2e>]
sent [CCP ConfReq id=0x1 <deflate 15> <deflate(old#) 15>]
sent [IPCP ConfReq id=0x1 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns3 0.0.0.0>]
rcvd [LCP DiscReq id=0x2 magic=0x90ae7a2e]
rcvd [LCP ProtRej id=0x3 80 fd 01 01 00 0c 1a 04 78 00 18 04 78 00]
rcvd [IPCP ConfNak id=0x1 <ms-dns1 10.11.12.13> <ms-dns3 10.11.12.14> <ms-wins 10.11.12.13> <ms-wins 10.11.12.14>]
sent [IPCP ConfReq id=0x2 <addr 0.0.0.0> <ms-dns1 10.11.12.13> <ms-dns3 10.11.12.14>]
rcvd [IPCP ConfNak id=0x2 <ms-dns1 10.11.12.13> <ms-dns3 10.11.12.14> <ms-wins 10.11.12.13> <ms-wins 10.11.12.14>]
sent [IPCP ConfReq id=0x3 <addr 0.0.0.0> <ms-dns1 10.11.12.13> <ms-dns3 10.11.12.14>]
rcvd [IPCP ConfNak id=0x3 <ms-dns1 10.11.12.13> <ms-dns3 10.11.12.14> <ms-wins 10.11.12.13> <ms-wins 10.11.12.14>]
sent [IPCP ConfReq id=0x4 <addr 0.0.0.0> <ms-dns1 10.11.12.13> <ms-dns3 10.11.12.14>]
rcvd [IPCP ConfReq id=0x0]
sent [IPCP ConfNak id=0x0 <addr 0.0.0.0>]
rcvd [IPCP ConfNak id=0x4 <addr 77.24.36.100> <ms-dns1 139.7.30.125> <ms-dns3 139.7.30.126>]
sent [IPCP ConfReq id=0x5 <addr 77.24.36.100> <ms-dns1 139.7.30.125> <ms-dns3 139.7.30.126>]
rcvd [IPCP ConfAck id=0x5 <addr 77.24.36.100> <ms-dns1 139.7.30.125> <ms-dns3 139.7.30.126>]
rcvd [IPCP ConfReq id=0x1]
sent [IPCP ConfAck id=0x1]
Could not determine remote IP address: defaulting to 10.64.64.64
local  IP address 77.24.36.100
remote IP address 10.64.64.64
primary   DNS address 139.7.30.125
secondary DNS address 139.7.30.126
Script /etc/ppp/ip-up started (pid 8494)
Script /etc/ppp/ip-up finished (pid 8494), status = 0x1
{% endcodeblock %}

That's it!

