---
title: "Disable screen blanking in the Raspberry Pi (Debian Buster)"
date: 2019-11-03T20:37:59+01:00
draft: false
---

# Disable screen blanking in the Raspberry Pi (Debian Buster)

From [Ozzmaker.com](http://ozzmaker.com/enable-x-windows-on-piscreen/):

```
sudo apt-get install x11-xserver-utils
sudo nano /etc/X11/Xsession.d/disableblank.sh
```

Add:
```
xset s off
xset -dpms
xset s noblank
```

Permissions:
```
sudo chmod +x /etc/X11/Xsession.d/disableblank.sh
```

Add to autostart:
```
sudo echo "/etc/X11/Xsession.d/disableblank.sh" >> /etc/xdg/lxsession/LXDE-pi/autostart
```
