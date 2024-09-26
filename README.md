# How to setup a pppoe server on your router for your ps4 to connect to the internet automatically without interfering with the PPPwn jailbreak
# Abstract
PPPwn is a PS4 jailbreak that uses a vulnerability in the PS4's pppoe implementation. Find the original PPPWn implementation [here](https://github.com/TheOfficialFloW/PPPwn).
Most PPPWn implementation are a dummy server, and do not forward packets. Most users either do not connect to the internet, or manually change their network configuration twice every jailbreak. This is time consuming, and is particularly difficult for users who regularly use remote play. I worked around this problem by running an internet-enabled pppoe server which utilises PADO-delay so that the ps4 always connects to the PPPwn server over this PPPoe server.

# Literature review
https://github.com/stooged/PI-Pwn is an all-in-one ppwn server for the Raspberry Pi which supports internet. However, that server toggles the main pppoe server as needed.
https://github.com/xfangfang/PPPwn_cpp (which is the pppwn release I use) is currently working on internet functionality in fork `gateway`, but is not released yet at time of writing.

# Hardware and software, and target audience
My implementation of this guide uses a router on [Asuswrt-Merlin](https://www.asuswrt-merlin.net/), with [entware](https://github.com/Entware/Entware) enabled. The pppoe server is [accel-ppp](https://github.com/accel-ppp/accel-ppp) (note: at the time of writing, entware's accel-ppp is buggy; a HEAD release for my router's architecture can be found [here](https://github.com/Tokarak/rtndev/releases/tag/1.13.0-2)). 

However, this guide is extensible beyond the software above. Any pppoe server will do, as long as it also supports pado-delay. Most platforms other than asuswrt-merlin have mature implementations, so you are actually in luck. Accel-ppp can be compiled for most platforms too.

Other platforms have different firewall implementations. More on opening the firewall later.

# Network Topology
The ps4 must be connected by ethernet to the PPPoe server and the jailbreak server (PPPwn server), which can be the same device. My home network has a central L2 ethernet switch to which I connected the ps4, the router, and a miniserver I use to run PPPwn. 

