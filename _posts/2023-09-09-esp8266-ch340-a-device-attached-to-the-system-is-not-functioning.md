---
layout: post
title: Fixing ESP8266/CH340 `a device attached to the system is not functioning` error
---

I have my own ESP8266-based solution that handles for example

* measuring temperatures using ds18b20 sensor,
* measuring temperatures using thermocouples (SPI bus),
* measuring temperature, barometric pressure and RH (I2C bus) and
* reading values from Victron solar systems. 

Data is sent using MQTT or HTTP to configured endpoint. It also has a web-based configuration panel. It's built with [PlatformIO](https://platformio.org/) which is super useful extension for vscode.

## CH340 and Windows 11 issue
Lately I updated my laptop to a newer one and it has Windows 11.

I installed vscode, PlatformIO and built the existing project succesfully. After connecting the ESP8266 to the compressor using USB, Windows automatically installed drivers for it (or actually for the USB/serial adapter CH340).

When trying to upload the project to the ESP8266 the process fails to an error:

```
serial.serialutil.SerialException: Cannot configure port, something went wrong. Original message: PermissionError(13, 'A device attached to the system is not functioning.', None, 31)
```

No matter what I do, the error still persist. When trying the same ESP and project with older PC, everything works fine.

## The fix

After searching around, I found a StackOverflow question with similar problem. The user had issues with some driver version. 

[https://stackoverflow.com/questions/76158517/serial-to-usb-cable-ch340-new-driver-not-functioning-with-c-sharp-applicatio](https://stackoverflow.com/questions/76158517/serial-to-usb-cable-ch340-new-driver-not-functioning-with-c-sharp-applicatio)

There [was an useful answer](https://stackoverflow.com/a/76416083/8140625) that indeed mentions Windows 11 and driver problems. The solution was to install older drivers for CH340.

The drivers Windows 11 had automatically installed were quite recent, from 2023. I searched around and [downloaded the drivers from 2011](https://www.arduined.eu/ch340g-converter-windows-7-driver-download/), installed them and everything started working!


![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/e611bcc8-4a52-4cca-bcbc-4d99bbd30a90)

The drivers are also available here on my Github repository:

[https://jisotalo.github.io/others/CH340-drivers-3.3.2011.zip](https://jisotalo.github.io/others/CH340-drivers-3.3.2011.zip)