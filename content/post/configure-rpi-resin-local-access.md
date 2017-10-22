---
title: "Configure a Resin-based RPi for Local Access"
date: 2017-10-22T20:08:49+02:00
draft: false
tags: [ "IoT", "Resin", "Raspberry Pi" ]
---

# Accessing a Resin.io-based Raspberry Pi

Quick cookbook for future me.

From [Resin.io Raspberry Pi guide](https://resinos.io/docs/raspberrypi3/gettingstarted/):

* Remove the MicroSD card, under the `/boot` partition locate and edit the `/boot/config.json` file.  Change the _hostname_ accordingly:

````bash
{
  "persistentLogging": false,
  "hostname": "resin",
}
````

* Configure the local network.  I think this can also be done remotely, however is handy to know where to look:

````bash
[connection]
id=resin-sample
type=wifi

[wifi]
mode=infrastructure
ssid=I_Love_Unicorns

[wifi-security]
auth-alg=open
key-mgmt=wpa-psk
psk=superSecretPassword

[ipv4]
method=auto

[ipv6]
addr-gen-mode=stable-privacy
method=auto
````

* Pinging the device (if you haven't changed the _hostname_):

````bash
$ ping resin.local
````

* Local SSH access (if you haven't changed the _hostname_):

````bash
ssh root@resin.local -p22222
````