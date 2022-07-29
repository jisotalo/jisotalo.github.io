---
layout: post
title: Codesys network variable list (NVL) generator
---

Due to recent development of [`codesys-client`](https://github.com/jisotalo/codesys-client) Node.js library, I had a need to simplify process to crete network variable list (NVL) declaration files (`.GVL`).

The `codesys-client` is a Node.js library to read/write data to Codesys PLCs. It can be found from https://github.com/jisotalo/codesys-client

I created a simple html page that can be used to create a file that can then be imported to Codesys for receiving data.

The tool can be found online from https://jisotalo.github.io/others/nvl-file-generator.html

![image](https://user-images.githubusercontent.com/13457157/181707384-d64eda07-782b-40ce-a4d1-48c1ef16aa9d.png)

Pressing the button creates a `NVL_ReceiveTest.GVL` file that will be downloaded using browser download window.

```xml
<GVL>
<Declarations><![CDATA[{attribute 'qualified_only'}
VAR_GLOBAL
  DataToReceive : ST_Data;
END_VAR]]></Declarations>
<NetvarSettings Protocol="UDP">
  <ListIdentifier>100</ListIdentifier>
  <Pack>True</Pack>
  <Checksum>False</Checksum>
  <Acknowledge>False</Acknowledge>
  <CyclicTransmission>False</CyclicTransmission>
  <TransmissionOnChange>False</TransmissionOnChange>
  <TransmissionOnEvent>False</TransmissionOnEvent>
  <Interval>T#100ms</Interval>
  <MinGap>T#50ms</MinGap>
  <EventVariable></EventVariable>
  <ProtocolSettings>
    <ProtocolSetting Name="Broadcast Adr." Value="255.255.255.255" />
    <ProtocolSetting Name="Port" Value="12023" />
  </ProtocolSettings>
</NetvarSettings>
</GVL>
``