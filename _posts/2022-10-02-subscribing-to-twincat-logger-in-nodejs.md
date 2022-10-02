---
layout: post
title: Subscribing to TwinCAT Logger in Node.js
---

TwinCAT has its own system logger running in ADS port 100. It logs different system messages for displaying in Visual Studio, Windows event list or in a MessageBox popup. 

Log lines can also be added from PLC manually, see https://stackoverflow.com/questions/51700348/is-there-a-way-to-print-to-output-console-twincat3

![image](https://user-images.githubusercontent.com/13457157/193469388-25d2f37a-2f33-4850-a7fe-b5a2d749630f.png)

But how to subscribe to these logs in Node.js?

## Finding how it works

I usually use two different methods to research on stuff like this: Wireshark or [TwinCAT ADS monitor](https://www.beckhoff.com/en-en/products/automation/twincat/tfxxxx-twincat-3-functions/tf6xxx-tc3-connectivity/tf6010.html). The latter is faster to use if you know that ADS commands are the way to go.

When TwinCAT system is restarted, few log lines are added. Therefore I started monitoring ADS traffic and restarted local TC to see what happens. After that when looking through the requests, it's not hard to find correct ones like this:
![image](https://user-images.githubusercontent.com/13457157/193469703-779cdea7-475c-4d7e-a97e-fdc5cce52958.png)

So now we know that port number 100 sends device notifications when new log lines are available. By looking [reserved ADS ports](https://github.com/jisotalo/ads-server/blob/master/src/ads-commons.ts#L114), we can see that port 100 is for Logger. Note that this is different than EventLogger, which we will probably handle in the future posts.

Now we know that where the log lines come from. But how to subscribe to those values? For that we need to know `index group` and `index offset`. And because Visual Studio (TwinCAT XAE) is showing those log lines, it most probably subscribes to device notifications.

So starting traffic recording using ADS monitor and then opening TwinCAT XAE should show some device notification registering. Bingo.

![image](https://user-images.githubusercontent.com/13457157/193469965-c1f8de2f-9c57-44d0-9b4b-5dbc8592a420.png)

So to listen for TwinCAT Logger, one should add device notification request to following
* ADS port: 100
* Index group: 1
* Index offset: 0xffff
* Data length: 1024
* Transmission mode: cyclic

## Unpacking received data

To see how it works, let's create a simple test using [ads-client library](https://github.com/jisotalo/ads-client/).

```js
//test-1.js
const ads = require('ads-client');

const client = new ads.Client({
  targetAmsNetId: 'localhost',
  targetAdsPort: 100,
  bareClient: true
});

client.connect()
  .then(async res => {
    console.log(`Connected to the ${res.targetAmsNetId}`);
    console.log(`Router assigned us AmsNetId ${res.localAmsNetId} and port ${res.localAdsPort}`);

    await client.subscribeRaw(1, 0xffff, 1024, data => {
      console.log(`Received data: ${data.value.toString()}`);
    }, 0, false); //Note: cyclic and 0ms cycle time

    console.log('Subscribed');
  })
  .catch(err => {
    console.log('Something failed:', err);
  })

``` 

By starting it using `node test-1.js` and then restarting TwinCAT, we can see that data is incoming! Naturally it's not too clear as I'm just converting the whole data packet to string (to see quickly if things match up).

```
Connected to the 127.0.0.1.1.1
Router assigned us AmsNetId 192.168.5.131.1.1 and port 33784
Subscribed
Received data:  ��Ӎ��☺☺►'TwinCAT SystemMTwinCAT System Restart initiated from AmsNetId: 192.168.5.131.1.1 port 33195.
Received data: �u�Ӎ��☺☺►'TwinCAT System2Saving configuration of COM server 
TcEventLogger !
Received data:  ��Ӎ��☺☺►'TwinCAT System(Shutting down COM Server TcEventLogger !
```

## Unpacking the received logger entry data

The raw logger entry packet looks like following:
![image](https://user-images.githubusercontent.com/13457157/193470413-16ccc2d9-f4a4-4ce9-88a1-ed3bd1f8ca76.png)

With experience from other ADS libraries, I suspected that the timestamp might be in Windows FILETIME format (starts from year 1601), which was the case. The string parts are easy to see and looking through few packets it was clear that the packet also contains sender ADS port and log message level.

![image](https://user-images.githubusercontent.com/13457157/193470742-f178cb09-7ed2-4731-86ca-5e7b828d489d.png)


The log level is is a masked value (same style as in [ADSLOGSTR](https://infosys.beckhoff.com/english.php?content=../content/1033/tcplclib_tc2_system/31033611.html&id=9189897725322916238)) and the values can be found from Tc2System PLC libary.

![image](https://user-images.githubusercontent.com/13457157/193470617-d8c7c16c-db6d-4a46-ad89-08b8ed1833dd.png)

There are some bytes that I couldn't find out.

## Creating a Node.js code to unpack the packet

Here is the final code for subscribing to TwinCAT logger and displaying each log line as object.

https://gist.github.com/jisotalo/80b01d590666dc19b228c732353820a1

```js
/*
Example how to subscribe and handle TwinCAT Logger data
https://jisotalo.github.io

Copyright (c) Jussi Isotalo <j.isotalo91@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
*/
const ads = require('ads-client');
const long = require('long'); //Required to handle FILETIME

const client = new ads.Client({
  targetAmsNetId: 'localhost',
  targetAdsPort: 100,
  bareClient: true
});

/**
 * Converts msgType to array of message types
 * @param {number} msgTypeMask 
 * @returns 
 */
const convertMsgType = (msgTypeMask) => {
  let strs = [];

  if ((msgTypeMask & 0x1) == 0x1)
    strs.push('hint')
  if ((msgTypeMask & 0x2) == 0x2)
    strs.push('warning')
  if ((msgTypeMask & 0x4) == 0x4)
    strs.push('error')
  if ((msgTypeMask & 0x10) == 0x10)
    strs.push('log')
  if ((msgTypeMask & 0x20) == 0x20)
    strs.push('msgbox')
  if ((msgTypeMask & 0x40) == 0x40)
    strs.push('resource')
  if ((msgTypeMask & 0x80) == 0x80)
    strs.push('string')

  return strs;
}

/**
 * Unpacks received TwinCAT Logger entry to object
 * @param {Buffer} data 
 * @returns 
 */
const unpackTwinCatLoggerEntry = (data) => {
  let pos = 0;
  const row = {};
  const _unknown = {};
  
  //0..7 - timestamp
  row.timestamp = new Date(new long(data.readUInt32LE(pos), data.readUInt32LE(pos + 4)).div(10000).sub(11644473600000).toNumber());
  pos += 8;

  //8 - message type
  row.msgTypeMask = data.readUint8(pos);
  row.msgTypes = convertMsgType(row.msgTypeMask);
  pos += 1;

  //9 - unknown byte
  _unknown.byte_9 = data.readUint8(pos);
  pos += 1;

  //10 - unknown byte
  _unknown.byte_10 = data.readUint8(pos);
  pos += 1;

  //11 - unknown byte
  _unknown.byte_11 = data.readUint8(pos);
  pos += 1;

  //12..13 - sender ADS port
  row.senderAdsPort = data.readUint16LE(pos);
  pos += 2;

  //14 - unknown byte
  _unknown.byte_14 = data.readUint8(pos);
  pos += 1;

  //15 - unknown byte
  _unknown.byte_15 = data.readUint8(pos);
  pos += 1;

  //16..n - sender and payload string
  //There are also few bytes of unknown data between sender and msg, not handled here
  let str = data.slice(pos).toString().slice(0, -1);

  //Sender is from start of string until \0
  row.sender = str.substr(0, str.indexOf("\0"));

  //Message is from end until \0
  row.msg = str.substr(str.lastIndexOf("\0") + 1);

  //Uncomment to add unknown bytes to object
  //row._unknown = _unknown;

  //Uncomment to add raw data to object
  //row._raw = data;

  return row;
}

//Following can be used to test
//unpackTwinCatLoggerEntry(Buffer.from('50d8be9f72d6d80101000000102700005477696e4341542053797374656d00002000000054635254696d652053657276657220737461727465643a2054635254696d652e00', 'hex'));

client.connect()
  .then(async res => {
    console.log(`Connected to the ${res.targetAmsNetId}`);
    console.log(`Router assigned us AmsNetId ${res.localAmsNetId} and port ${res.localAdsPort}`);

    await client.subscribeRaw(1, 0xffff, 1024, data => {
      const entry = unpackTwinCatLoggerEntry(data.value);

      console.log(entry);
      
    }, 0, false); //Note: cyclic and 0ms cycle time

    console.log('Subscribed');    
  })
  .catch(err => {
    console.log('Something failed:', err)
  })
```

When the code is running and TwinCAT is restarted, log lines can be seen:

```js
Connected to the 127.0.0.1.1.1
Router assigned us AmsNetId 192.168.5.131.1.1 and port 33805
Subscribed
{
  timestamp: 2022-10-02T18:51:05.496Z,
  msgTypeMask: 1,
  msgTypes: [ 'hint' ],
  senderAdsPort: 10000,
  sender: 'TwinCAT System',
  msg: 'TwinCAT System Restart initiated from AmsNetId: 192.168.5.131.1.1 port 33195.'
}
{
  timestamp: 2022-10-02T18:51:05.525Z,
  msgTypeMask: 1,
  msgTypes: [ 'hint' ],
  senderAdsPort: 10000,
  sender: 'TwinCAT System',
  msg: 'Saving configuration of COM server TcEventLogger !'
}
{
  timestamp: 2022-10-02T18:51:06.252Z,
  msgTypeMask: 1,
  msgTypes: [ 'hint' ],
  senderAdsPort: 10000,
  sender: 'TwinCAT System',
  msg: 'Shutting down COM Server TcEventLogger !'
}
```

And now if PLC code sends messages to Logger, the Node.js app can also see them:
```
IF SendLog THEN
    ADSLOGSTR(
        msgCtrlMask := ADSLOG_MSGTYPE_HINT OR ADSLOG_MSGTYPE_LOG, 
        msgFmtStr   := 'Test message from PLC to Node.js', 
        strArg      := ''
    );      
    SendLog := FALSE;
END_IF
```
```js
Connected to the 127.0.0.1.1.1
Router assigned us AmsNetId 192.168.5.131.1.1 and port 33831
Subscribed
{
  timestamp: 2022-10-02T18:56:40.433Z,
  msgTypeMask: 17,
  msgTypes: [ 'hint', 'log' ],
  senderAdsPort: 352,
  sender: 'PlcTask1',
  msg: 'Test message from PLC to Node.js'
}
```

## What next

Perhaps next we should see how to send messages to TwinCAT Logger from Node.js? 

Other thing to do is to see how TwinCAT EventLogger can be used from Node.js