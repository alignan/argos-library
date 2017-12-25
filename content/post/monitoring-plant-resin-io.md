---
title: "An IoT Raspberry Pi powered plant monitoring system II"
date: 2017-12-25T18:54:40+01:00
draft: false
tags: [ "Challenge 2017", "Bonsai", "Resin IO", "relayr", "IoT", "Sensors", "Raspberry Pi" ]
---

# Monitoring my plant using sensors, relayr's cloud, resin.io and the Raspberry Pi

This is part of my [Challenge to make 26 years before 2017 ends](https://github.com/alignan/things-to-do/blob/master/README.md).

This post is a continuation of the [Raspberry Pi powered plant monitoring system post]({{< relref "post/monitoring-plant-sensors.md" >}}), in which I discussed the project requirements, and briefly showed how to flash the `resinOS` to a Raspberry Pi and waterproof a soil sensor.

Nothing too fancy by then.

Now I will show:

* How I actually deployed the application in `Resin.io` using `Docker`
* How to connect and read data from the soil moisture sensor (over a ADC to I2C converter) and the atmospheric sensor bundle to the Raspberry Pi
* How to publish the sensor data to the [relayr cloud](https://relayr.io) over MQTTS

**Disclaimer**: I currently work at relayr GmbH as a technical manager, however for me this project was an opportunity to better understand the company's solution stack, and to put myself in the shoes of my development team towards being a better manager.

## The project

The project lives at [The Kodama Guardian repository](https://github.com/alignan/kodama-guardian).

>Kodama (木霊, 木魂 or 木魅) are spirits in Japanese folklore that inhabit trees, similar to the dryads of Greek mythology. The term is also used to denote a tree in which a kodama supposedly resides. The phenomenon known as yamabiko, when sounds make a delayed echoing effect in mountains and valleys, is sometimes attributed to this kind of spirit

[![](/img/monitoring-plant-resin-io/00.png)](/monitoring-plant-resin-io/00.png)

### The Docker machine

I just selected one python-based `Docker` available distro from `resin.io` registry.  From the `Dockerfile` the most noticeable requirements are the `python-smbus` and `i2c-tools` used to read data from the I2C sensors.  Notice how the `i2c-dev` drivers has to be loaded prior running the application.

````bash
# Base Image
FROM resin/raspberry-pi2-python

RUN apt-get update && apt-get install -yq \
            python-smbus i2c-tools libraspberrypi-bin ca-certificates && \
            apt-get clean && rm -rf /var/lib/apt/lists/*

# Copy requirements file first for better cache on later pushes
COPY ./requirements.txt /requirements.txt 

# The resin.io python-based images has already the following installed:
# pip, python-dbus, virtualenv, setuptools

# pip install python deps from requirements.txt on the resin.io build server
RUN pip install -r /requirements.txt

# This will copy all files in our root to the working directory in the container
COPY . /usr/src/app

# Set our working directory
WORKDIR /usr/src/app

# switch on systemd init system in container
ENV INITSYSTEM on

# main.py will run when container starts up on the device
CMD modprobe i2c-dev && python src/kodama.py
````

###  Enable the I2C driver and increase the baudrate

Add the following to the `/boot/config.txt` file:
````bash 
dtparam=i2c_arm=on,i2c_baudrate=400000
````

### The specific sensor classes

I modified [Matt Hawkins' BME280 example](https://bitbucket.org/MattHawkinsUK/rpispy-misc/raw/master/python/bme280.py) and created a python class.  From [Seeedstudio's wiki page](http://wiki.seeed.cc/Grove-I2C_ADC/) I also adapted their example and created a class for the Soil Moisture sensor (over the ADC-to-I2C converter).

The implementation is quite simple as it just returns the sensor value.  At the moment of development I moved some useful checks (like validating the sensor value) to the main application itself, which is not a good idea as it makes the sensor's classes less reusable and clutters the application with more things to check... as I'm writing this post I'm creating a new issue to change this later... see [#2](https://github.com/alignan/kodama-guardian/issues/2) and [#3](https://github.com/alignan/kodama-guardian/issues/3).

### The Sensor class

The [measurement](https://github.com/alignan/kodama-guardian/blob/master/src/measurement.py) class is a wrapper for all sensors.  It allows to check if a reading is valid (boundary limits), check for alerts (thresholds), and return a JSON-formatted message to be published over MQTT according to relayr's expected ontology.

````bash
{ "name":self.name, "value":self.value, "recorded":timestamp }
```` 

### The application

The main application is implemented in the `kodama.py` file.  The most relevant parts are the relayr's cloud parameters:

````bash
CLOUD_HOST = "cloud-mqtt.relayr.io"
CLOUD_CERT = "/usr/src/app/src/cacert.pem"
CLOUD_PORT = 8883
````

And the credentials and other values configured at `resin.io` admin interface (and exposed as environmental variables at runtime) to avoid publishing in the open:

````bash
# Values set in resin.io ENV VARS
PERIOD_MEAS   = int(os.getenv('PER_MEAS', 30000))
PERIOD_SLEEP  = int(os.getenv('PER_SLEEP', 1000))
CLOUD_USER    = os.getenv('RELAYR_USER')
CLOUD_PASS    = os.getenv('RELAYR_PASS')
CLOUD_DEV     = os.getenv('RELAYR_DEV')
CLOUD_ID      = os.getenv('RESIN_DEVICE_UUID')
````

[![](/img/monitoring-plant-resin-io/01.png)](/monitoring-plant-resin-io/01.png)

The application takes sensor readings every second (as default) and check for alerts, and publishes instant readings every 30 seconds (as default).  Later I will modify the application to publish instead averaged values per hour, along with minimum and maximum values in the same period.  At the moment I chose to publish faster to play with the sensor dashboards (to be shown in a next post).

````bash
# Run the scheduled routines
threading.Timer(PERIOD_MEAS / 1000, measurements_send).start()

# Run the main loop and check for alerts to be published
while(True):

    soil = soil_hum.read_raw()
    temp, atmp, humd = bme280.read_all()
    MQTT_MEASUREMENT_MAP['measurements']['temp'].is_valid(temp)
    MQTT_MEASUREMENT_MAP['measurements']['humd'].is_valid(humd)
    MQTT_MEASUREMENT_MAP['measurements']['atmp'].is_valid(atmp)
    MQTT_MEASUREMENT_MAP['measurements']['soil'].is_valid(soil)

    print "Soil {0}% @ {1}°C {2}%RH {3}hPa".format(soil, temp, humd, atmp)

    # This will check for alerts to be sent to the cloud
    check_alerts()

    # Wait a bit
    time.sleep(PERIOD_SLEEP / 1000)
````

The `MQTT_MEASUREMENT_MAP` is a dictionary which contains the `Sensor` objects and keeps track of the published `Message ID`.

````bash
MQTT_MEASUREMENT_MAP = {
  'measurement_mid' : 0,
  'measurements' : {
    'soil' : soil_moist,
    'temp' : temperature,
    'humd' : humidity,
    'atmp' : pressure
  }
}
````

Likewise, the `MQTT_ALERTS_MAPS` is a dictionary which contains the `alerts` to be sent immediately, including the ones specific to the sensors (thresholds), as well as custom defined ones.  The block below summarizes its construction.

````bash
# This is the dictionary to keep ongoing alerts (other than sensor's)
my_alerts = {
  'alerts' :
  {
    # Sensor failure
    'sensor_failure' : 'clear',
    # No water in the tank
    'no_water'       : 'clear',
    # Flood likely!
    'valve_loose'    : 'clear',
  }
}

# Copy the alerts dictionary into this map to keep track of alerts state changes
MQTT_ALERTS_MAP = {
  'alerts_mid' : 0,
}
MQTT_ALERTS_MAP.update(deepcopy(my_alerts))

# Add sensor specific alerts
for key, value in MQTT_MEASUREMENT_MAP['measurements'].iteritems():
  if value.low_thr_msg is not None:
    MQTT_ALERTS_MAP['alerts'][value.low_thr_msg] = value.alerts[value.low_thr_msg]
  if value.hi_thr_msg is not None:
    MQTT_ALERTS_MAP['alerts'][value.hi_thr_msg]  = value.alerts[value.hi_thr_msg]
````

The alerts are sent only whenever there is a change in the alert status, either `set` or `clear`, to minimize flooding the broker.  One or more alerts can be sent at once in the same message.

The `MQTT_COMMAND_MAP` contains the supported remote commands received from relayr's cloud, at the moment none are implemented.  The `managed_mode` is my idea to either let the application decide when to water the plant (based on the sensor readings and last watering date), or only allow the user to decide when to water the plant (either manually by activating an electro-valve over a button or via the `water_on` command).

````bash
# Supported configuration values
MQTT_COMMAND_MAP = {
  # Open the sprinkler
  'water_on'     : None,
  # Enable or Disable managed mode
  'managed_mode' : None
}
````

Here's an example of the application running (logging via the `resin.io` console):

[![](/img/monitoring-plant-resin-io/02.png)](/monitoring-plant-resin-io/02.png)

## The wiring

The Raspberry Pi used for the project had already a ribbon cable exposing the following pin-out:

[![](/img/monitoring-plant-resin-io/03.png)](/monitoring-plant-resin-io/03.png)

I used the Raspberry Pi's I2C and 3.3V pins to connect the atmospheric and soil moisture sensors using a prototyping PCB I had laying around.  Later I will also route two unused GPIOs to connect the electrovalve (to water the plant) and a flow sensor (to detect leaks).  Here's the small PCB (back):

[![](/img/monitoring-plant-resin-io/04.jpg)](/monitoring-plant-resin-io/04.jpg)

And front:

[![](/img/monitoring-plant-resin-io/05.jpg)](/monitoring-plant-resin-io/05.jpg)

In the [previous post]({{< relref "post/monitoring-plant-sensors.md" >}}) the sensor references and datasheets are shown, in case you are wondering about the pin-out.

I keep my plant beside my office's desk at the moment.  As shown in the photo I'm currently using a bucket to host my plant as I prefer to water the bonsai using a fine sprinkler for a nice shower.

[![](/img/monitoring-plant-resin-io/06.jpg)](/monitoring-plant-resin-io/06.jpg)

## Next plans

No wonder, the next phase is to actually water the plant.

I have been playing with the following idea: put the plant on top of a water tank with two sections, one to collect all the water leftover from the watering (and even use it as a fish tank), and the other to store clean water.  A submerged pump in the clean water tank would pump the water to a small irrigation hose (around the trunk of the plant) and an elevated sprinkler (to shower from above).  Here's a sketch:

[![](/img/monitoring-plant-resin-io/07.jpg)](/monitoring-plant-resin-io/07.jpg)

I tested a bilge pump (over 12V), and it has way enough power to supply water to my two watering systems.  It can be easily controlled even by a relay over a RPI's GPIO.

[![](/img/monitoring-plant-resin-io/08.jpg)](/monitoring-plant-resin-io/08.jpg)