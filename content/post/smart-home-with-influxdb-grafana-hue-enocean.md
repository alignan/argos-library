---
title: "Smart Home with enOcean sensors, Philips Hue and Weather underground, running on influxDb and Grafana in a Raspberry Pi (Part 1 of 2)"
date: 2018-11-02T18:43:04+01:00
draft: false
---

# Smart Home with enOcean sensors, Philips Hue and Weather underground, running on influxDb and Grafana in a Raspberry Pi (Part 1 of 2)

This is part of my [Challenge to make 26 things before 2018 ends](https://github.com/alignan/things-to-do).

I wanted to eat my own dog food, scratch my own itch, follow the preaching book... in short, to add some of the technologies and concepts I use at work in my home.

## My use cases

I started to collect so-called `smart devices` at home, I wanted to put these to a good use, and avoid having to use a plethora of applications to control or visualize its information, but centralize everything into something simple.  In short, I needed to:

* Display the weather conditions (as my wife always ask me what is the temperature outside, or if it will rain later... I felt silly shouting at [Alexa](https://www.amazon.com/Amazon-Echo-And-Alexa-Devices/b?ie=UTF8&node=9818047011) from my bedroom...)
* Set the Philip Hue light bulbs to its previous color and bright when powering back on (by default the lights go to an awful warm color when powered back on)
* Compare the indoor temperature of the bedrooms vs. outside
* Depending on the weather conditions change the light bulb colors and brightness
* Know when it is time to ventilate the flat (based on CO2 level)

I wanted to have a local dashboard to display the information above, without needing Internet access, available via my laptop, mobile phones, and over a dedicated screen.

The architecture looks as following:

[![](/img/smart-home-with-influxdb-grafana-hue-enocean/00.png)](/img/smart-home-with-influxdb-grafana-hue-enocean/00.png)

If you want to skip the next sections, here's the link to the project repository:

https://github.com/alignan/home-improvements

## Setting up the Raspberry Pi and installing the dependencies

My current version:

```bash
$ cat /etc/os-release
PRETTY_NAME="Raspbian GNU/Linux 9 (stretch)"
NAME="Raspbian GNU/Linux"
VERSION_ID="9"
VERSION="9 (stretch)"
ID=raspbian
ID_LIKE=debian
HOME_URL="http://www.raspbian.org/"
SUPPORT_URL="http://www.raspbian.org/RaspbianForums"
BUG_REPORT_URL="http://www.raspbian.org/RaspbianBugs"
```

### Install influxDB

```bash
$ curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/os-release

$ test $VERSION_ID = "9" && echo 'deb https://repos.influxdata.com/debian stretch stable' | sudo tee /etc/apt/sources.list.d/influxdb.list.

$ sudo apt-get update && sudo apt-get install influxdb
$ sudo service influxdb start
```

Check version by running:
```bash
$ influx
Connected to http://localhost:8086 version 1.6.3
InfluxDB shell version: 1.6.3
> exit
```

### Install Grafana

```bash
$ sudo apt-get install apt-transport-https curl
$ curl -sL https://bintray.com/user/downloadSubjectPublicKey?username=bintray | sudo apt-key add -
$ echo 'deb https://dl.bintray.com/fg2it/deb stretch main' | sudo tee -a /etc/apt/sources.list.d/grafana.list
$ sudo apt-get update
$ sudo apt-get install grafana
$ sudo /bin/systemctl enable grafana-server

Synchronizing state of grafana-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable grafana-server
Created symlink /etc/systemd/system/multi-user.target.wants/grafana-server.service → /usr/lib/systemd/system/grafana-server.service.

```

## Integrating the Philips Hue light bulbs

I googled around and found a Python wrapper for the Philips Hue API, the [qhue](https://github.com/quentinsf/qhue) fetches and compiles the API into a python module.

Mainly, I'm just replacing the default factory light settings to something preferred.  The script checks periodically for the light settings and replaces if default settings are found.

```bash
"Main light living room": {
        "default": {
            "bri": 254,
            "ct": 156,
            "sat": 251,
            "hue": 34069
        },
        "unwanted": {
            "hue": 14988,
            "ct": 366,
            "sat": 141
        }
    },
```

Later I'm planing to refactor this with an event-based approach, but it does the work at expected.  The other alternative was to pay for an phone application just to set the lights to its previous state, as seems the light bulbs do not have on-board flash memory, and the Hub doesn't store light settings... my solution is good at the moment.

My next step was to somehow reflect in the light bulbs the weather conditions (e.g. dark blue if below 10°C, purple-ish if raining, etc.).

As the above JSON schema, I defined a mapping of light settings to weather states (as defined by the Weather Underground API):

```bash
{
	"Drizzle": "default",
	"Light Drizzle": "default",
	"Heavy Drizzle": "default",
	"Rain": "default",
	"Light Rain": "default",
	"Heavy Rain": "default",
	"Snow": "default",
	"Light Snow": "default",
	"Heavy Snow": "default",
	(...)
```

I'm retrieving the following weather information:

* ambient temperature
* ambient humidity
* weather status
* atmospheric pressure
* wind speed and gust

I selected a weather station just across my street 

```python
wunderground_url = 'http://api.wunderground.com/api/{0}/geolookup/conditions/q'.format(wunderground_api['wunderground'])

r = requests.get(wunderground_url + WUNDERSTATION_URL).json()

# build my dictionary array to write to database
meas = {}
meas['berlin_temperature'] = round(float(r['current_observation']['temp_c']), 2)
meas['berlin_humidity'] = round(float(r['current_observation']['relative_humidity'][:-1]), 2)
meas['berlin_temperature_feels'] = round(float(r['current_observation']['feelslike_c']), 2)
meas['weather_state'] = r['current_observation']['weather']
meas['berlin_pressure'] = round(float(r['current_observation']['pressure_mb']), 2)
meas['berlin_wind_speed_kph'] = round(float(r['current_observation']['wind_kph']), 2)
meas['berlin_wind_gust_kph'] = round(float(r['current_observation']['wind_gust_mph']) * 1.609344, 2)

```

To make the python application to run as a service:

```bash
$ cat /etc/systemd/user/philips_hue.service /lib/systemd/system/philips_hue.service 
[Unit]
Description=Philips hue Service
After=multi-user.target

[Service]
Type=idle
WorkingDirectory=/home/pi
ExecStart=/home/pi/home-improvements/venv/bin/python /home/pi/home-improvements/philips_hue_devices.py

[Install]
WantedBy=multi-user.target
```

Enable and run:

```bash
$ sudo systemctl enable philips_hue.service
Created symlink /etc/systemd/system/multi-user.target.wants/philips_hue.service → /lib/systemd/system/philips_hue.service

$ sudo systemctl start philips_hue.service
```

To check the logs for errors and others:

```bash
$ journalctl -u philips_hue.service
```

## Configure the HDMI monitor and graphical mode

In the `config.txt` file (boot partition):

```bash
# uncomment if hdmi display is not detected and composite is being output
hdmi_force_hotplug=1

# uncomment to force a specific HDMI mode (this will force VGA)
hdmi_group=2
hdmi_mode=4

# uncomment to force a HDMI mode rather than DVI. This can make audio work in
# DMT (computer monitor) modes
hdmi_drive=2

# uncomment to increase signal to HDMI, if you have interference, blanking, or
# no display
config_hdmi_boost=4
```

In the `raspi-config` utility also configure the `Pi` to run directly in `Desktop mode` (via the `Boot Options`), also in the `/etc/lightdm/lightdm.conf` (or use `lightdm-set-defaults` to avoid modifying the file directly).

```bash
autologin-user=pi
autologin-user-timeout=0
```

And then some:

```bash
$ sudo apt-get install --no-install-recommends xserver-xorg -y
$ sudo apt-get install --no-install-recommends raspberrypi-ui-mods lxterminal gvfs -y
$ sudo apt-get install lxde --fix-missing
$ sudo apt-get install xinit
$ startx
```

### Configure Kiosk mode

Disable screen saver or blanking, from [this page](https://www.ceos3c.com/misc/disable/) add the lines below to the `~/.config/lxsession/LXDE-pi/autostart` file:

```bash
@xset s off
@xset -dpms
@xset s noblank
@unclutter
```

At this point I avoided researching and went with [Google's top results](https://die-antwort.eu/techblog/2017-12-setup-raspberry-pi-for-kiosk-mode/):

```bash
sudo apt-get install --no-install-recommends chromium-browser
```

Configure `OpenBox` in `/etc/xdg/openbox/autostart`:

```bash
# Disable any form of screen saver / screen blanking / power management
xset s off
xset s noblank
xset -dpms

# Allow quitting the X server with CTRL-ATL-Backspace
setxkbmap -option terminate:ctrl_alt_bksp

# Start Chromium in kiosk mode
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/'Local State'
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/; s/"exit_type":"[^"]\+"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences
chromium-browser --disable-infobars --kiosk 'http://your-url-here'
```

Plus this tweak to `.bashrc` to launch the `X server` at boot without the cursor:

```bash
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx -- -nocursor
```

This was not entirely successful, in [this other guide](https://medium.com/@alexjv89/setting-up-a-dashboard-on-raspberry-pi-4e6e2e37ddbb) I tried the following steps:

```bash
$ mkdir /home/pi/.config/autostart
$ nano /home/pi/.config/autostart/kiosk.desktop

[Desktop Entry]
Type=Application
Name=Kiosk
Exec=/home/pi/kiosk.sh
X-GNOME-Autostart-enabled=true

$ nano /home/pi/kiosk.sh

#!/bin/bash
# Run this script in display 0 - the monitor
export DISPLAY=:0
# Hide the mouse from the display
unclutter &
# If Chrome crashes (usually due to rebooting), clear the crash flag so we don't have the annoying warning bar
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' /home/pi/.config/chromium/Default/Preferences
sed -i 's/"exit_type":"Crashed"/"exit_type":"Normal"/' /home/pi/.config/chromium/Default/Preferences
# Run Chromium and open tabs
chromium-browser --kiosk --app=https://grafana-url
```

To run `playlist mode` (multiple dashboards rotated after a giving period), replace the url for:

```bash
chromium-browser --kiosk --app=http://grafana-url/playlists/play/1?kiosk
```

When `kiosk.sh` runs now shows Grafana's login page, we want this to be automated - I prefer to use the proxy options, let's check other possibilities.

Configure `Grafana` to avoid login in `/etc/grafana/grafana.ini`:

```bash
[auth]
# Set to true to disable (hide) the login form, useful if you use OAuth, defaults to false
disable_login_form = true 

[auth.anonymous]
# enable anonymous access
enabled = true 
# specify organization name that should be used for unauthenticated users
org_name = Main Org.
# specify role for unauthenticated users
org_role = Editor
```

For a better visibility, I changed the resolution to `1440 x 900 (60Hz)` using the `raspi-config` utility.

