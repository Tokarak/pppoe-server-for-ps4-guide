# How to Setup a PPPoe Server on your Router for your PS4 to Connect to the Internet Automatically Without Interfering with the PPPwn Jailbreak
# Abstract
PPPwn is a PS4 jailbreak that uses a vulnerability in the PS4's pppoe implementation. Find the original PPPWn implementation [here](https://github.com/TheOfficialFloW/PPPwn).
Most PPPWn implementation are a dummy server, and do not forward packets. Most users either do not connect to the internet, or manually change their network configuration twice every jailbreak. This is time consuming, and is particularly difficult for users who regularly use remote play. I worked around this problem by running an internet-enabled pppoe server which utilises PADO-delay so that the ps4 always connects to the PPPwn server over this PPPoe server.

# Literature Review
https://github.com/stooged/PI-Pwn is an all-in-one ppwn server for the Raspberry Pi which supports internet. However, that server toggles the main pppoe server as needed.
https://github.com/xfangfang/PPPwn_cpp (which is the pppwn release I use) is currently working on internet functionality in fork `gateway`, but is not released yet at time of writing.

# Hardware and Software, and target audience
My implementation of this guide uses a router on [Asuswrt-Merlin](https://www.asuswrt-merlin.net/), with [entware](https://github.com/Entware/Entware) enabled. The pppoe server is [accel-ppp](https://github.com/accel-ppp/accel-ppp) (note: at the time of writing, entware's accel-ppp is buggy; a HEAD release for my router's architecture can be found [here](https://github.com/Tokarak/rtndev/releases/tag/1.13.0-2)). 

However, this guide is extensible beyond the software above. Any pppoe server will do, as long as it also supports pado-delay. Most platforms other than asuswrt-merlin have mature implementations, so you are actually in luck. Accel-ppp can be compiled for most platforms too.

Other platforms have different firewall implementations. More on opening the firewall later.

# Network Topology
The ps4 must be connected by ethernet to the PPPoe server and the jailbreak server (PPPwn server), which can be the same device. My home network has a central L2 ethernet switch to which I connected the ps4, the router, and a miniserver I use to run PPPwn. 

# Generalised summary of the steps
1. Run a pppoe server with a chap-secrets file for auth. Listen on the interface the ps4 is connected on.
2. Configure ps4 ip address and dns server in the pppoe config
3. Enabled ip forwarding (should already be enabled on a router), and open the firewall to allow packets to/from the ps4 to be forwarded to/from the lan and to the wan.
4. Check that the ps4 can access the internet
5. Increase PADO delay to 10s (10s works fine if the PPPwn implementation waits for the second PADI packet; can be much lower (e.g 1 sec) if PPPwn does not wait for second PADI)
6. Run PPPwn _ad lib_

# Steps for my set up
1. Get a working implementation of a pppoe server (I wrote about my problems in section "Hardware and Software")
2. I am running a dns server with adblocking on my router. For some reason, the ps4 could not access it on the ethernet interface ("hairpinning"), so I let the dns server run on the accel-ppp interface.
Add to `/jffs/configs/dnsmasq.conf.add`:
```
interface=ppp0
no-dhcp-interface=ppp0 # accel-ppp interface
```
4. Here is my diff from the default accel-ppp config
```diff
--- accel-ppp.conf-opkg
+++ accel-ppp.conf
@@ -1,18 +1,18 @@
 [modules]
 #log_file
-#log_syslog
+log_syslog
 #log_tcp
 #log_pgsql

-connlimit
+#connlimit

-radius
-#chap-secrets
+#radius
+chap-secrets

-pptp
-l2tp
+#pptp
+#l2tp
 #sstp
-#pppoe
+pppoe
 #ipoe

 auth_mschap_v2
@@ -22,7 +22,7 @@

 ippool

-pppd_compat
+#pppd_compat

 #shaper
 #net-snmp
@@ -51,8 +51,8 @@
 [ppp]
 verbose=1
 min-mtu=1280
-mtu=1400
-mru=1400
+mtu=1492
+mru=1492
 #accomp=deny
 #pcomp=deny
 #ccp=0
@@ -86,9 +86,9 @@

 [pppoe]
 verbose=1
-#ac-name=xxx
-#service-name=yyy
-#pado-delay=0
+ac-name=PPPWN-internet
+#service-name=PPPWN-internet
+pado-delay=10000
 #pado-delay=0,100:100,200:200,-1:500
 called-sid=mac
 #tr101=1
@@ -102,7 +102,7 @@
 #vlan-timeout=60
 #vlan-name=%I.%N
 #interface=eth1,padi-limit=1000
-interface=eth0
+interface=br0

 [l2tp]
 verbose=1
@@ -200,7 +200,7 @@
 interface=eth0

 [dns]
-#dns1=172.16.0.1
+dns1=192.168.51.1
 #dns2=172.16.1.1

 [wins]
@@ -232,22 +232,18 @@
 10.0.0.0/8

 [ip-pool]
-gw-ip-address=192.168.0.1
+gw-ip-address=192.168.51.1
 #vendor=Cisco
 #attr=Cisco-AVPair
 attr=Framed-Pool
-192.168.0.2-255
-192.168.1.1-255,name=pool1
-192.168.2.1-255,name=pool2
-192.168.3.1-255,name=pool3
-192.168.4.1-255,name=pool4,next=pool1
-192.168.4.0/24
+192.168.51.11-255

 [log]
 log-file=/opt/var/log/accel-ppp/accel-ppp.log
 log-emerg=/opt/var/log/accel-ppp/emerg.log
 log-fail-file=/opt/var/log/accel-ppp/auth-fail.log
 #log-debug=/dev/stdout
+#log-debug=/opt/var/log/accel-ppp.log
 #syslog=accel-pppd,daemon
 #log-tcp=127.0.0.1:3000
 copy=1
@@ -271,8 +267,8 @@
 #fork-limit=16

 [chap-secrets]
-gw-ip-address=192.168.100.1
-#chap-secrets=/opt/etc/ppp/chap-secrets
+gw-ip-address=192.168.51.1
+chap-secrets=/opt/etc/ppp/chap-secrets
 #encrypted=0
 #username-hash=md5
```

_I gave up on writing my guide halfway through; and I ca't remember what I missed. The general steps should be enough in most cases._
