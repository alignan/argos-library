---
title: "An IoT and Raspberry Pi powered plant monitoring system"
date: 2017-10-22T17:31:04+02:00
draft: false
tags: [ "Challenge 2017", "Bonsai", "IoT", "Sensors", "Raspberry Pi" ]
---

# Monitoring my new plant using sensors and the Raspberry Pi

This is part of my [Challenge to make 26 years before 2017 ends](https://github.com/alignan/things-to-do/blob/master/README.md).

This type of project is nothing new in the _Makersphere_, using environmental sensors with the Raspberry Pi (and Arduinos, ESP8266, etc.) has been done since it's early beginnings, and it is perhaps one of the most implemented use cases.

Then what is new?

Likely nothing surprisingly, however, I would attempt to take this above the normal "demo and forget" existing setups, and create something with a product-oriented approach.

Notice the title of this post: monitoring.  I will write about watering automation in a separate post, as first it would be preferred to understand what would my plant needs.

## Why a Raspberry Pi?

Why not? I have used many embedded platforms in the past, thus extracting data from sensors and forwarding back to another device/server in the sky for post-processing is not trivial... I wanted to do something different, like deploy from [Resin.io](https://resin.io), [Docker](https://www.docker.com) (see [my challenge](https://github.com/alignan/things-to-do/blob/master/README.md)), or even expand and build more features like adding a local webserver, etc.

As this is likely going to be a work in progress with room for experimentation, the [Raspberry Pi](https://www.raspberrypi.org) seems like a good choice.

Pro tip: I can even kick [RetroPie](https://retropie.org.uk) while my plant is watered auto-magically.

## Requirements

Most of the requirements were outlined in [My new Bonsai]({{< relref "post/my-new-bonsai.md" >}}) blog post, from which the following are extracted:

* Measure the humidity of the soil
* Measure the ambient temperature, humidity and why not, atmospheric pressure
* Measure the water level after in the saucer (the one collecting the excess of water at the bottom)

## Components

The following are the list of components to be integrated to the Raspberry Pi.

### Atmospheric sensor-bundle

The [Weather click from MikroElektronika](https://shop.mikroe.com/weather-click) features the [Bosch BME280](https://download.mikroe.com/documents/datasheets/BST-BME280_DS001-11.pdf) sensor, bundling together ambient temperature, humidity and atmospheric pressure over a SPI/I2C communication bus.

[![](/img/monitoring-plant-sensors/00.jpg)](/monitoring-plant-sensors/00.jpg)

### Soil moisture sensor

One of the most common (and cheap) sensors is the [Grove Moisture sensor](http://wiki.seeed.cc/Grove-Moisture_Sensor/) (which I happen to have lying around).  It is an analogue sensor, thus an external ADC-to-I2C converter (like the [Grove I2C ADC](http://wiki.seeed.cc/Grove-I2C_ADC/)) is needed.

Before connecting to the Raspberry Pi, first some conditioning needs to be done to the sensor, as it will be semi-buried in the soil, it needs to be able to withstand watering without damaging its components.

I removed the 4-pin connector to be able to fit a [Heat shrink tube](https://en.wikipedia.org/wiki/Heat-shrink_tubing), thus trying to water-proof as most as possible:

[![](/img/monitoring-plant-sensors/01.jpg)](/monitoring-plant-sensors/01.jpg)

Then solder the wires directly on to the pads of the sensor:

[![](/img/monitoring-plant-sensors/02.jpg)](/monitoring-plant-sensors/02.jpg)

And finally use the heat shrink tube in combination with plastic silicone, to try to prevent water dripping through the cable, at most it will be driven on top of the heat shrinks.

[![](/img/monitoring-plant-sensors/03.jpg)](/monitoring-plant-sensors/03.jpg)