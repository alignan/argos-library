---
title: "Moving from Wunderground to OpenWeatherMap"
date: 2019-03-09T20:42:36+01:00
draft: false
tags: [ "Wunderground", "Openweather" ]
---

# Moving from Wunderground to OpenWeatherMap

From [Wunderground site](https://www.wunderground.com/weather/api/):

>The Weather Underground API has reached the end of service. To purchase access to the replacement API, please see our Weather Data Packages. If you are a contracted, paying customer and feel you have been shut off in error, please email feedback.wunderground@weather.com; be sure to include your name and API key.

My [wunderground-powered weather dashboard]({{<relref "post/smart-home-with-influxdb-grafana-hue-enocean.md">}})) was sadly a victim of this change, and as for my personal use I don't need the extended (paid) services or features offered in [The Weather Company tiers](http://biz.weather.com/WU-Data-API_Data-Package-Demo-Request.html), I had to look for a different option to enable back my dashboard.

In this [reddit thread](https://www.reddit.com/r/webdev/comments/8tjavu/now_that_the_free_wunderground_api_has_been/) alternatives to wunderground have been suggested, I chose to migrate to [OpenWeatherMap](https://openweathermap.org) for the following:

* Up to 60 calls per minute, but they recommend to keep as low as 1 per 10 minutes, which is OK for my usage
* Simple and well-documented API
* Descriptive weather condition mapped to `id` fields in the JSON payload, which makes it easier to build rules (although its numbering is a bit messy)

There are disadvantages or course:

* Not as many weather stations as Wunderground, but I found at least 5 in Berlin and one of them was close to where I live
* No `feels-like` temperature data, although it can be calculated.
* Updates are slower than Wunderground, but as I plan to update every 10 minutes is fine

I imported the [weather condition codes](https://openweathermap.org/weather-conditions) as a JSON, to allow later to map a given lighting setting to a weather status.

```json
{
	"200": {
		"description": "thunderstorm with light rain",
		"action": "default"
	},
	"201": {
		"description": "thunderstorm with rain",
		"action": "default"
	},
	"202": {
		"description": "thunderstorm with heavy rain",
		"action": "default"
	}
```

As in my latest [commit](https://github.com/alignan/home-improvements/commit/4fa2cdf4d61e3628ded17b884ec5d7cda25801c8), I just needed to modify my previous constants:

```python
OPENWEATHER_PATH    = "openweather.json"
OPENWEATHER_URL     = '/weather?id=2950159'
```

And update the getter and parser:

```python
    openweather_states = get_file(OPENWEATHER_PATH)
    openweather_api = get_file(CRED_FILE_PATH)
    openweather_url = 'http://api.openweathermap.org/data/2.5/{0}&units=metric&appid={1}'.format(
        OPENWEATHER_URL, openweather_api['openweather'])

    r = requests.get(openweather_url).json()

    # build my dictionary array to write to database
    meas = {}
    meas['berlin_temperature'] = round(float(r['main']['temp']), 2)
    meas['berlin_humidity'] = round(float(r['main']['humidity']), 2)
    meas['weather_state'] = r['weather'][0]['description']
    meas['berlin_pressure'] = round(float(r['main']['pressure']), 2)
    meas['berlin_wind_speed_kph'] = round(float(r['wind']['speed']), 2)
```
