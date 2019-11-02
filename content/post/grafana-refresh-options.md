---
title: "Grafana refresh options"
date: 2019-11-02T23:15:48+01:00
draft: false
---

When launching the `kiosk` mode add:

```
chromium-browser --noerrdialogs --disable-infobars --kiosk 'http://raspberrypi-living-room:3000/d/0Duk-Qggz/home?refresh=1m&orgId=1&from=now-24h&to=now --incognito --disable-translate'
```

To allow refreshing every minute and to specify a given time lapse.  Also, in the Grafana dashboard options set a refresh rate.
