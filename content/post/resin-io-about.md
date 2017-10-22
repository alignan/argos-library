---
title: "About Resin.io and ResinOS"
date: 2017-10-22T22:35:30+02:00
draft: false
tags: [ "IoT", "Resin", "Raspberry Pi", "Docker"]
---

# Understanding Resin and ResinOS

I felt I needed to write this post about [Resin.io](https://resinos.io), as I was jumping directly into deploying and building in a Raspberry Pi (see [this post]({{< relref "post/monitoring-plant-sensors.md" >}})), just jumping from one tutorial to the next.

It is good to pace down, understand, and then go full speed.

## Resin.io

>Resin.io makes it simple to deploy, update, and maintain code running on remote devices. We are bringing the web development and deployment workflow to hardware, using tools like git and Docker to allow you to seamlessly update all your embedded linux devices in the wild. We handle cross-compilation, device monitoring, VPNs, and log collection, so you can focus on your product and not the infrastructure.

Highlights from [How the code is deployed](https://docs.resin.io/understanding/understanding-code-deployment/):

* Changes tracked and deployed from a Github repository
* Each Resin.io application as an unique remote repository associated
* Changes in `master` triggers a remote update on devices
* Changes compiled in a cross-compiler docker container (i.e. `ARMv6` for the Raspberry Pi)
* Compiler container updates the docker registry
* Possible to include a `Dockerfile` to further tune the process

## ResinOS

>ResinOS is an operating system optimised for running Docker containers on embedded devices

This is the other endpoint - the device side who handles the updates.

Highlights from [What is ResinOS](https://docs.resin.io/understanding/understanding-devices/2.0.0/):

* Linux oriented
* Based on the [Yocto Framework](https://www.yoctoproject.org/)
* `2.0.0+rev2` recommended for production
* `2.0.0+rev2-dev` is used for active development.  The [local mode](https://docs.resin.io/development/local-mode/) (to build and sync in a local network) is only available in this mode
* Docker socket exposed on via port `2377`, which allows `resin local push` to do remote Docker builds on the target device
* 8 MB journald RAM buffer to avoid wear on the flash storage
* Stateless rootFS (Read-Only system), to keep configuration information a persistent partition is maintained
* However, persistent storage can be enabled by changing `"persistentLogging": true` in the `config.json` file
* `resin-boot` (as used in the Raspberry Pi) is a FAT32 partition for device boot configuration.  The `config.json` file used by ResinOS is located here
* Other partitions (read-only):
  * `resin-rootA`: holds read-only root filesystem
  * `resin-rootB`: empty partition that is only used when the rootfs is to be updated
  * `resin-state`: holds persistent data
  * `resin-data`: holds downloaded Docker images

ResinOS is composed by the [following services](https://docs.resin.io/understanding/understanding-devices/2.0.0/#resinos-components):

* `The resin.io supervisor`:  a lightweight container that manages applications and communicates with Resin.io servers
* `Systemd`: arrgh
* `Docker Engine`: a lightweight container runtime to build and run linux containers on resinOS
* `NetworkManager and ModemManager`: manage connection to the Internet, either via ethenet, wifi or cellular modem.  A `system-connections` folder in the boot partition allows to replicate content in `/etc/NetworkManager/system-connections`
* `DNSmasq`: manage the nameservers
* `Avahi`: advertises the device in the local network
* `OpenVPN server`: disabled by default, must be configured and activated

The `BSP` for each platform is built using `poki/yocto` meta packages, an example of how it is built for the Raspberry Pi is shown [Here](https://github.com/resin-os/resin-raspberrypi/blob/master/layers/meta-resin-raspberrypi/conf/samples/bblayers.conf.sample)

The following language support is available: Nodejs, Python, Go and Java.

## Wrap-up

That's it - I just wanted to grab the basic concepts as I go through Resin.io and Docker.  To run simple scripts (i.e. run a typical IoT sensor-application) seems an overkill, however it is practical as reduce the hurdles of device management, and it makes a lot of sense if scaling-up your application.