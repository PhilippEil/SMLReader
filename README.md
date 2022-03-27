# MeterToMQTT

An ESP8266 based sensor to MQTT gateway.
This gateway can not only read your smart electricity meter with SML, it can also read analog and digital sensors. Which are for example attached to water meters or gas meters.

## About
It is based on the SMLReader repository of [mruettgers](https://github.com/mruettgers/SMLReader).

The aim of this project is to read the meter readings of modern energy meters and make them available via MQTT.

The software was primarily developed and tested for the EMH ED300L electricity meter, but should also work with other energy meters that have an optical interface and communicate via the SML protocol.

SMLReader publishes the metrics read from the meter's optical unit to an MQTT broker configured via the provided web interface.

If you like this project, you might consider to [support me](#donate).

### Screenshots
![WiFi and MQTT setup](doc/screenshots/screenshot_setup.png)
![Grafana](doc/screenshots/screenshot_grafana.png)
```bash
MB-Monty ➜  ~  mosquitto_sub -h 10.4.32.103 -v -t smartmeter/mains/#
smartmeter/mains/info Hello from 00C7551E, running SMLReader version 2.1.5.
smartmeter/mains/sensor/1/obis/1-0:1.8.0/255/value 3546245.9
smartmeter/mains/sensor/1/obis/1-0:2.8.0/255/value 13.2
smartmeter/mains/sensor/1/obis/1-0:1.8.1/255/value 0.0
smartmeter/mains/sensor/1/obis/1-0:2.8.1/255/value 13.2
smartmeter/mains/sensor/1/obis/1-0:1.8.2/255/value 3546245.9
smartmeter/mains/sensor/1/obis/1-0:2.8.2/255/value 0.0
smartmeter/mains/sensor/1/obis/1-0:16.7.0/255/value 451.2
```

## Getting started
First you need to configure the software, for help visit the [wiki](https://github.com/PhilippEil/MeterToMQTT/wiki) 

Next the software must first somehow get onto the device. 
This can be done in several ways.

### Uploading a precompiled binary

A precompiled binary for the Wemos D1 and configured to have a sensor attached to pin D2 can be downloaded from the releases page on GitHub:  

[https://github.com/mruettgers/SMLReader/releases](https://github.com/mruettgers/SMLReader/releases)

#### Flashing

This precompiled binary can be flashed to the device with the help of [esptool.py](https://github.com/espressif/esptool).
Other tools should also work, as long as you have configured them to match the specs of your ESP8266.

```bash
esptool.py --port /dev/ttyUSB0 write_flash -fm dout 0x00000 path/to/smlreader.bin
```

As a docker enthusiast I prefer to utilize a dockerized version of esptool.py by bind-mounting the current working directory to `/src` and assuming that `smlreader.bin` is existing in the current working directory:
```bash
docker run -it --device /dev/ttyUSB0 -v $(pwd):/src --rm mruettgers/esptool --port /dev/ttyUSB0 write_flash -fm dout 0x00000 /src/smlreader.bin
```

The device path and the path to the binary have to be adjusted to fit your needs.


##### Example

```bash
nb-ubuntu ➜  ~/Downloads  ls SMLReader_D1mini_v2.1.3.bin                                                                                                         
SMLReader_D1mini_v2.1.3.bin

nb-ubuntu ➜  ~/Downloads  docker run -it --device=/dev/ttyUSB0 -v $(pwd):/src --rm mruettgers/esptool --port /dev/ttyUSB0 write_flash -fm dout 0x00000 /src/SMLReader_D1mini_v2.1.3.bin
esptool.py v2.8
Serial port /dev/ttyUSB0
Connecting....
Detecting chip type... ESP8266
Chip is ESP8266EX
Features: WiFi
Crystal is 26MHz
MAC: d8:f1:5b:07:0d:eb
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
Auto-detected Flash size: 4MB
Flash params set to 0x0340
Compressed 378528 bytes to 266738...
Wrote 378528 bytes (266738 compressed) at 0x00000000 in 23.6 seconds (effective 128.1 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

---

### Using your IDE for building and flashing

You should be able to use your preferred IDE to build and flash SMLReader if you take care of the dependencies and the build flags configured in the `platform.io` file.
I strongly recommend using PlatformIO as it takes care of that itself.

#### Building

Building SMLReader in PlatformIO is straight forward and can be done by executing the build task matching your environment (i.e. `d1_mini`).

In case you get the following error, it is time to install a Git client (https://git-scm.com/downloads) and to make sure that the path to `git` is covered by the PATH variable and `git` is thus executable from everywhere.

```
UserSideException: Please install Git client from https://git-scm.com/downloads:
  File "/home/monty/.platformio/penv/lib/python3.8/site-packages/platformio/builder/main.py", line 168:
    env.SConscript("$BUILD_SCRIPT")
```

![Building with PlatformIO](doc/screenshots/screenshot_platformio_build.png)


#### Flashing

![Flashing with PlatformIO](doc/screenshots/screenshot_platformio_upload.png)

---


### Running

WiFi and MQTT are configured via the web interface provided by [IotWebConf](https://github.com/prampec/IotWebConf) and which can be reached after joining the WiFi network named SMLReader and heading to http://192.168.4.1.   
If the device has already been configured, the web interface can be reached via the IP address obtained from your local network's DHCP server.
To login provide the user `admin` and the configured AP password.

*Attention: You have to change the AP Password (empty by default), otherwise SMLReader won't work.*

If everything is configured properly and running with a sensor in place, SMLReader will publish the metrics and values received from the meter to the configured MQTT broker:

```
MB-Monty ➜  ~  mosquitto_sub -h 10.4.32.103 -v -t smartmeter/mains/#
smartmeter/mains/info Hello from 00C7551E, running SMLReader version 2.1.5.
smartmeter/mains/sensor/1/obis/1-0:1.8.0/255/value 3546245.9
smartmeter/mains/sensor/1/obis/1-0:2.8.0/255/value 13.2
smartmeter/mains/sensor/1/obis/1-0:1.8.1/255/value 0.0
smartmeter/mains/sensor/1/obis/1-0:2.8.1/255/value 13.2
smartmeter/mains/sensor/1/obis/1-0:1.8.2/255/value 3546245.9
smartmeter/mains/sensor/1/obis/1-0:2.8.2/255/value 0.0
smartmeter/mains/sensor/1/obis/1-0:16.7.0/255/value 451.2

smartmeter/mains/sensor/2/obis/1-0:1.8.0/255/value 3546245.9
smartmeter/mains/sensor/2/obis/1-0:2.8.0/255/value 13.2
smartmeter/mains/sensor/2/obis/1-0:1.8.1/255/value 0.0
smartmeter/mains/sensor/2/obis/1-0:2.8.1/255/value 13.2
smartmeter/mains/sensor/2/obis/1-0:1.8.2/255/value 3546245.9
smartmeter/mains/sensor/2/obis/1-0:2.8.2/255/value 0.0
smartmeter/mains/sensor/2/obis/1-0:16.7.0/255/value 451.2

smartmeter/mains/sensor/3/obis/1-0:1.8.0/255/value 3546245.9
smartmeter/mains/sensor/3/obis/1-0:2.8.0/255/value 13.2
smartmeter/mains/sensor/3/obis/1-0:1.8.1/255/value 0.0
smartmeter/mains/sensor/3/obis/1-0:2.8.1/255/value 13.2
smartmeter/mains/sensor/3/obis/1-0:1.8.2/255/value 3546245.9
smartmeter/mains/sensor/3/obis/1-0:2.8.2/255/value 0.0
smartmeter/mains/sensor/3/obis/1-0:16.7.0/255/value 451.2
```

---


### Debugging

Serial logging can be enabled by setting `SERIAL_DEBUG=true` in the `platformio.ini` file before building.
To increase the log level and to get the raw SML data, also set `SERIAL_DEBUG_VERBOSE=true`.

#### Serial port monitor

A serial port monitor can be attached using the corresponding function of your IDE or by invoking a terminal client like `miniterm` which comes shipped with `python-serial`.
The serial port settings are `115200,8,N,1`.

##### miniterm.py

```bash
miniterm /dev/ttyUSB0 115200
```

###### Example

```bash
nb-ubuntu ➜  ~/Downloads  miniterm /dev/ttyUSB0 115200
--- Miniterm on /dev/ttyUSB0  115200,8,N,1 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
Config size: 685
Wrong config version.
AP password was not set in configuration
State changing from: 0 to 1
Setting up AP: SMLReader
With default password: <hidden>
AP IP address: 192.168.4.1
State changed from: 0 to 1
...
```

Or the dockerized way:

```bash
docker run -it --device /dev/ttyUSB0 -v $(pwd):/src --rm mruettgers/esptool ash -c "miniterm.py /dev/ttyUSB0 115200"
```



##### PlatformIO

![PlatformIO Monitor](doc/screenshots/screenshot_platformio_upload_and_monitor.png)



---

## Acknowledgements

* [mruettgers/SMLReader](https://github.com/mruettgers/SMLReader)

### Third party libraries
* [ESPSoftwareSerial](https://github.com/plerup/espsoftwareserial)
* [IotWebConf](https://github.com/prampec/IotWebConf)
* [MicroDebug](https://github.com/rlogiacco/MicroDebug)
* [ESPAsyncTCP](https://github.com/me-no-dev/ESPAsyncTCP)
* [Pangolin MQTT Client](https://github.com/philbowles/PangolinMQTT)
* [libSML](https://github.com/volkszaehler/libsml)
* [JLed](https://github.com/jandelgado/jled)

### Links

* https://www.msxfaq.de/sonst/bastelbude/smartmeter_d0_sml_protokoll.htm
* https://www.photovoltaikforum.com/thread/78798-sml-pr%C3%BCfsummenberechnung-ich-verzweifle-an-crc/ (Helped me to finally solve some weird CRC calculation issues before switching over to libSML)
* http://www.stefan-weigert.de/php_loader/sml.php (Sources of the 3D-printed housing of the reading head)
* https://github.com/gemu2015/Sonoff-Tasmota (Gemu's excellent Tasmota version with SML support)

## Donate

### Paypal

mruettgers:
[![Paypal](https://www.paypalobjects.com/en_US/DK/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=GK95YZCEGJT84)

## License

Distributed under the GPL v3 license.  
See [LICENSE](LICENSE) for more information.
