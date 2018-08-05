---
title: "SSH hangs when connected to Raspberry Pi 3"
date: 2018-08-05T19:39:29+02:00
draft: false
tags: [ "Raspberry Pi", "SSH" ]
---

# SSH hangs when connected to Raspberry Pi 3

Credits to [Express Hosting Blog](https://expresshosting.net/ssh-hanging-authentication/).

`Tl;DR: add "IPQoS 0x00" to the bottom of the "/etc/ssh/ssh_config" and "/etc/ssh/sshd_config" files`

From the Blog:

>One the latest version of SSH installed on the Raspberry Pi 3 uses QoS headers to ensure speedy delivery of packets over the network. For interactive connections (standard shell SSH connections) it sets the IP header for IP_TOS to be 0x10 (low delay or latency). For non-interactive connections (scp, etc) it sets the IP header for IP_TOS to be 0x08 (max throughput).  The problem in our case seems to be that our network doesn’t really like those values. We aren’t sure if this issue is with the router itself, or something in between. Since this connection is occurring over WiFi and our router handles the Wifi, we suspect that the router is where the issue lies though.