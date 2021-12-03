---
layout: post
title: ESP8266 - Updating time from NTP server
---

I have been working on an ESP8266 project lately. The idea is to create a system that reads temperature/other sensors and then sends the data using MQTT or REST.
The system is fully configurable with a web browser. I'm not too good with C++ so.. not too easy :D

Naturally, I needed to get the active timestamp that is updated fron NTP and works fully automatically (time goes by and it also syncs automatically every now and then). 
However there were lot's of different guides and libraries for the job. Finally found the simplest solution
that has been working without any hassle. And the best: No additional libraries are needed.

Source is mainly from https://github.com/esp8266/Arduino/blob/master/libraries/esp8266/examples/NTP-TZ-DST/NTP-TZ-DST.ino There are many other examples too.
## Get active timestamp
```c++
#include <TZ.h>
#include <time.h>       // time() ctime()

//call configTime somewhere in the beginning
//  1st parameter is the time zone (here UTC0)
//  2st parameter is NTP server address
//See TZ.h file for other timezones
configTime(TZ_Etc_UCT, "pool.ntp.org"));

//Later: Get active epoch timestamp (seconds since 1970)
//Note that if the NTP hasn't updated yet, it starts from 0
uint32_t epochTime = time(nullptr);
```
## Get a notification when NTP updates time
Default NTP sync interval is 1 hour.
```c++
#include <TZ.h>
#include <time.h>       // time() ctime()

//call configTime somewhere in the beginning
//  1st parameter is the time zone (here UTC0)
//  2st parameter is NTP server address
//See TZ.h file for other timezones
configTime(TZ_Etc_UCT, "pool.ntp.org"));

//Set callback to time update (lambda function here)
settimeofday_cb([](bool ntp) {
  if(ntp) {
    Serial.println("Time updated from NTP server");
  } else {
    Serial.println("Time updated manually");
  }
});

//Later: Get active epoch timestamp (seconds since 1970)
//Note that if the NTP hasn't updated yet, it starts from 0
uint32_t epochTime = time(nullptr);
```

To change sync interval to something else, like 15 seconds, add the following function to your code somewhere.

```c++
uint32_t sntp_update_delay_MS_rfc_not_less_than_15000() { return 15000; } //ms -> 15 seconds
```

## Creating a timestamp string
I have created timestamp strings in the following way. Note that the timestamp won't take 72 bytes but if the char[] is smaller the sprintf() gives a warning 
as the output _might_ be that long.

The following creates a string like `10.11.2021 - 15:30:45`.
```c++
time_t epochTime = time(nullptr);

//Creating time struct
auto time = localtime(&epochTime);


static char DateAndTimeString[72];
sprintf(DateAndTimeString, "%02d.%02d.%04d %02d:%02d:%02d", time->tm_mday,
        time->tm_mon + 1, time->tm_year + 1900, time->tm_hour, time->tm_min,
        time->tm_sec);
```

See more examples from https://github.com/esp8266/Arduino/blob/master/libraries/esp8266/examples/NTP-TZ-DST/NTP-TZ-DST.ino
