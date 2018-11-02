---
title: "Smart Home with enOcean sensors, Philips Hue and Weather underground, running on influxDb and Grafana in a Raspberry Pi (Part 2 of 2)"
date: 2018-11-02T20:22:16+01:00
draft: false
---

# Smart Home with enOcean sensors, Philips Hue and Weather underground, running on influxDb and Grafana in a Raspberry Pi (Part 2 of 2)

This is part of my [Challenge to make 26 things before 2018 ends](https://github.com/alignan/things-to-do).

This is the second part of my blog post, if you want to read how this started click [here]({{< relref "post/smart-home-with-influxdb-grafana-hue-enocean.md" >}}))

The architecture looks as following:

[![](/img/smart-home-with-influxdb-grafana-hue-enocean/00.png)](/img/smart-home-with-influxdb-grafana-hue-enocean/00.png)

If you want to skip the next sections, here's the link to the project repository:

https://github.com/alignan/home-improvements

## enOcean gateway controller setup (USB 300)

In my setup I used the [enOcean USB 300 controller](https://www.enocean.com/en/enocean-modules/details/usb-300-oem/).

The first step was to check if the USB stick was detected by the OS:

```bash
$ dmesg
[134190.301364] usb 1-1.3: new full-speed USB device number 4 using dwc_otg
[134190.458809] usb 1-1.3: New USB device found, idVendor=0403, idProduct=6001
[134190.458819] usb 1-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[134190.458823] usb 1-1.3: Product: FT232R USB UART
[134190.458827] usb 1-1.3: Manufacturer: FTDI
[134190.458831] usb 1-1.3: SerialNumber: A90551EY
[134190.690127] usbcore: registered new interface driver usbserial
[134190.690168] usbcore: registered new interface driver usbserial_generic
[134190.690201] usbserial: USB Serial support registered for generic
[134190.698717] usbcore: registered new interface driver ftdi_sio
[134190.698759] usbserial: USB Serial support registered for FTDI USB Serial Device
[134190.698918] ftdi_sio 1-1.3:1.0: FTDI USB Serial Device converter detected
[134190.699010] usb 1-1.3: Detected FT232RL
[134190.699817] usb 1-1.3: FTDI USB Serial Device converter now attached to ttyUSB0
```

Also checked:

```bash
$ lsusb 
Bus 001 Device 004: ID 0403:6001 Future Technology Devices International, Ltd FT232 USB-Serial (UART) IC
```

I powered on an enOcean sensor to check if the Gateway would receive any reading.  It was required to change the default baudrate to `57600` first, and then dump data from the `/dev/ttyUSB0` to console:

```bash
$ stty -F /dev/ttyUSB0 57600
$ hexdump -C < /dev/ttyUSB0
00000000  55 00 0a 07 01 eb a5 00  00 54 08 01 80 f5 bc 00  |U........T......|
00000010  01 ff ff ff ff 2e 00 41  55 00 0a 07 01 eb a5 00  |.......AU.......|
00000020  00 50 08 01 9c 44 00 01  ff ff ff ff 32 00 33 55  |.P...D......2.3U|
```

The radio telegrams are described in the [enOcean ESP3 protocol](http://www.enocean.com/esp)


## Initial testing

### enOcean python client

I stumbled upon this [python-based client](https://github.com/kipe/enocean) to communicate over serial to the enOcean controller.

```bash
$ sudo pip install enocean
```

I had to modify the [enocean example](https://github.com/kipe/enocean/blob/master/examples/enocean_example.py) to initialize the `SerialCommunicator` to use the `/dev/ttyUSB0` port.

```python
communicator = SerialCommunicator(port='/dev/ttyUSB0')
```

I pressed the learn button in my [AFRISO Temperature Sensor FTM T](https://www.greenelectric.eu/Afriso-Temperature-Sensor-FTM-T-EnOcean-Afriso-Lab), this is what I received:
```bash

2018-10-13 12:35:04,506 - enocean.protocol.packet - DEBUG - learn received, EEP detected, RORG: 0xA5, FUNC: 0x02, TYPE: 0x05, Manufacturer: 0x2D
2018-10-13 12:35:04,507 - enocean.communicators.SerialCommunicator - DEBUG - 01:80:F5:BC->FF:FF:FF:FF (-60 dBm): 0x01 ['0xa5', '0x8', '0x28', '0x2d', '0x80', '0x1', '0x80', '0xf5', '0xbc', '0x0'] ['0x1', '0xff', '0xff', '0xff', '0xff', '0x3c', '0x0'] OrderedDict()
TMP: {u'value': 32.94117647058823, u'description': u'Temperature (linear)', u'unit': u'\xb0C', u'raw_value': 45}
```

### Understanding enOcean Telegrams

From the frame above there are three important fields:

* RORG: the ERP radio telegram type (8 bits)
* FUNC: basic functionality of the data content (6 bits)
* TYPE: type of device in its individual characteristics (7 bits)

The frame we received corresponds to 4BS or 4-byte communication (`RORG=0xA5`):

> A 4BS telegram carries a payload of 4 bytes. The sequence of the 4 data bytes is historically reversed, so that DB_3 appears first and DB_0 last on the radio interface. The bits are addressed in the sequence of the data flow, however (offset). Hence, DB_3.BIT_7 has the offset position 0 and DB_0.BIT_3 (LRN bit) has the offset position 28. The actual content-bits in a byte are not affected by this, i.e., they are described from right (2H0) to left (2H7) in the ascending order.

#### Temperature sensor

The bytes following the `0xa5` byte corresponds to the data payload (starting from DB_3) as follows: `'0x8', '0x28', '0x2d', '0x80'`, being `0x2d` the `raw_value` of 45, as shown in the received packet.  The calculations to obtain the actual temperature value are given by the offset, size and bit range information of the profile, per the EEP (enOcean Equipment Profile) description.

The `FUNC=0x02` and `TYPE=0x05` corresponds to the Temperature Sensors (given by the `FUNC` value) with a range from 0-40°C (as per its `TYPE` value).  The units are Celsius, and the calculation is made by taking the bits `DB1.0-DB1.7` (bitrange) with a 16-bit offset.  In this profile, `DB_3` and `DB_2` are not used, and `DB0.3` shows the `learn bit` status.  A `shortcut` identifier is used to resume the description before, in this case as `TMP` (linear temperature).

Note also the `0x01` at the start of the frame: this correspond to the `RADIO_ERP1` packet type (Radio Telegram).  This is further detailed in the Radio `Addressing Destination Telegram format` (ADT).  The `Sender ID` field follows the 4 bytes of the payload (as the `4BS` lenght), thus `'0x1', '0x80', '0xf5', '0xbc', '0x0'` are equivalent to the address `01:80:F5:BC`.

The next octets are part of the `Optional Data` fields, being `0x01` the number of sub-telegram received, `'0xff', '0xff', '0xff', '0xff'` the destination address (broadcast), and finally the `signal strength` byte (in dBm) and the `security level` byte (being zero related to telegram not processed).

#### CO2 Pure sensor with power failure detection

When I pressed the `learn` button of the [Afriso CO2 sensor](https://afrisohome.de/en/portfolio/co2-sensor-2/) I got the following frames:

```bash
2018-10-18 20:49:12,733 - enocean.protocol.packet - DEBUG - learn received, EEP detected, RORG: 0xA5, FUNC: 0x09, TYPE: 0x09, Manufacturer: 0x2D
2018-10-18 20:49:12,734 - enocean.communicators.SerialCommunicator - DEBUG - 01:9C:44:11->FF:FF:FF:FF (-54 dBm): 0x01 ['0xa5', '0x24', '0x48', '0x2d', '0x80', '0x1', '0x9c', '0x44', '0x11', '0x0'] ['0x1', '0xff', '0xff', '0xff', '0xff', '0x36', '0x0'] OrderedDict()
```

The `FUNC=0x09` and `TYPE=0x09` corresponds to the Pure CO2 sensors with power failure detection, with 8-bit resolution and up to 2000ppm.  The device is quite cool as it reflects the recommended CO2 thresholds over a LED (green if OK, yellow likely to ventilate the room, and red to definitely ventilate), and it has a message whenever a digital input changes (when power is lost), so it might be used as well to detect power failures in the room.

The `shortcut` identifier is `CO2` (data from `DB1.7-DB1.0`), and the failure detection identifier is `LRNB` (from `DB0.2`).


## Putting everything together

The `CO2` profile was not implemented in the [enocean application](https://github.com/kipe/enocean), I had to fork it and implement it on my own (just copy and pasting a XML block and filling the details), the link to the updated file is [here](https://github.com/alignan/enocean/blob/master/enocean/protocol/EEP.xml).

```xml
<profiles func="0x09" description="Gas Sensor">
      <profile description="Pure CO2 Sensor with Power Failure Detection" type="0x09">
        <data>
          <value shortcut="CO2" description="Pure CO2 sensor with 8 bit resolution and 0–2000ppm" offset="16" size="8" unit="ppm">
            <range>
              <min>0</min>
              <max>255</max>
            </range>
            <scale>
              <min>0</min>
              <max>2000</max>
            </scale>
          </value>
          <enum shortcut="PFD" description="Power Failure Detection" offset="28" size="1">
            <item description="Power failure not detected" value="0" />
            <item description="Power failure detected" value="1" />
          </enum>
        </data>
      </profile>
    </profiles>
```

The implementation itself was quite simple once I was able to verify the data frames from the radio, however rather than inspecting packets for its `TYPE` and `FUNC` fields (my preferred approach), I had to create a device mapping based on location as these fields were only populated by the library whenever the `learn` bit was set - perhaps something related to the protocol itself, but I wanted to implement this in a single lazy Sunday evening.

```python
ENOCEAN_DEVICES = {
    '01:80:F5:BC': 'main_bedroom_temperature',
    '01:9C:44:11': 'living_room_CO2'
}
```

Here's the looper:

```python
# endless loop receiving radio packets
    while communicator.is_alive():
        try:
            # Loop to empty the queue...
            packet = communicator.receive.get(block=True, timeout=1)

            meas = {}

            if packet.packet_type == PACKET.RADIO:
                if packet.rorg == RORG.BS4 and packet.sender_hex in ENOCEAN_DEVICES:

                    # a hack as the rorg_type and rorg_func fields are only populated when the learn bit is set,
                    # perhaps something I don't understand about the protocol... I'm taking the lazy approach...
                    if ENOCEAN_DEVICES[packet.sender_hex] == 'main_bedroom_temperature':
                    # if packet.rorg_type == 0x05 and packet.rorg_func == 0x02:
                        packet.select_eep(0x02, 0x05)
                        packet.parse_eep()
                        meas[ENOCEAN_DEVICES[packet.sender_hex]] = round(packet.parsed['TMP']['value'], 2)
                    if ENOCEAN_DEVICES[packet.sender_hex] == 'living_room_CO2':
                    # if packet.rorg_type == 0x09 and packet.rorg_func == 0x09:
                        packet.select_eep(0x09, 0x09)
                        packet.parse_eep()
                        meas[ENOCEAN_DEVICES[packet.sender_hex]] = round(packet.parsed['CO2']['value'], 2)
            if meas:
                publish_to_database(meas)

        except queue.Empty:
            continue
        except KeyboardInterrupt:
            break
        except Exception:
            traceback.print_exc(file=sys.stdout)
            break

        time.sleep(0.1)
```

As a next step I will replace the `/dev/ttyUSB0` with a proper `udev` rule.
