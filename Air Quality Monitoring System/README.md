# RPJiOS

[![Raspberry Pi Zero W](https://www.vectorlogo.zone/logos/raspberrypi/raspberrypi-ar21.svg)](https://www.raspberrypi.org/)
[![Python](https://www.vectorlogo.zone/logos/python/python-ar21.svg)](https://www.python.org/)
[![Redis](https://www.vectorlogo.zone/logos/redis/redis-ar21.svg)](https://redis.io/)
[![Debian](https://www.vectorlogo.zone/logos/debian/debian-ar21.svg)](https://www.raspbian.org/)

A [pub/sub](https://en.wikipedia.org/wiki/Publish–subscribe_pattern)-based implementation of a 
Raspberry Pi data pipeline built on [redis](https://redis.io) and centered around sensors.

The general philosophy is that a "sensor" is any entity (physical or not) that operates primarily
in an output-only mode (configuration doesn't necessarily count as an "input" so is allowable).

These outputs are treated ephemerally, in a "fire and forget" manner, creating a data stream.
The intent is for interested entities to subscribe to the data stream and transform, interpret 
and/or persist it according to their requirements.


## Requirements

* Hardware:
	* a Raspberry Pi running a recent Raspbian build (for sensor nodes)
	* any Debian-like system running apt (for managment/non-sensor nodes)
* some [sensors](#sensors), configured appropriately (most easily done with `raspi-config`):
	* depending on your chosen sensors: I2C enabled, SPI enabled, 1-wire enabled
	* other sensors (LM335, Soil, TEPT5700) require an [external ADC](#devices) which itself will require SPI

## Setup

* Clone the repo
* `cd` into repo dir
* `./setup.sh` (you might need to enter your `sudo` password to install requirements)
* `source env/bin/activate` and go!

## Tools

* [sensors-src](bin/sensors-src): source daemon that manages all specified sensors and publishes their data as configured
* [downsample](bin/downsample): a very flexible data stream downsampler/forwarder/transformer. example uses:
	* an at-frequency forwarder (set `-r 1`)
	* a loopback downsampler (set `-o` to the same as `-i`)
	* a many-to-one reducer (set `-t` to `key`)
	* a one-to-many exploder (set `-m` to `flatten:[options]`)
	* a bounded ephemeral cache (set `-t` to `list:[options]`, where the `limit=X` option sets the bound)
	* ...
	* profit!
* [sqlite-sink](bin/sqlite-sink): an [SQLite](https://www.sqlite.org) sink for data streams. examples:
	* sink to an SQLite database on a different host a downsampled data stream:
		1. on the source device:
			* `downsample -i redis://localhost -o redis://sql-db-host -r ... -p ...`
		2. on the sink host "`sql-db-host`":
			* `sqlite-sink path-to-db.sqlite3`
				* (lots of "TODOs" here, obviously) 
* [oled-display](bin/oled-display): an [OLED display](https://www.adafruit.com/product/661) driver for consuming & display some sensor data, among other things
* [thingspeak](bin/thingspeak): a simple example of a [ThingSpeak](http://thingspeak.com) data forwarder for the SPS30 particulate matter sensor data. [Example resulting data set](https://thingspeak.com/channels/655525).

