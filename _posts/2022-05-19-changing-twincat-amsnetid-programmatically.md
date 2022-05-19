---
layout: post
title: Changing TwinCAT AmsNetId programmatically
---

TwinCAT local AmsNetId can be changed using `Router->Change AmsNetId` from tray icon menu.

![image](https://user-images.githubusercontent.com/13457157/169378130-4b236698-eec8-444c-8e3c-e61a77efc4f0.png)

But what if you want to change the AmsNetId manually from your own program or script? Obviously the tray menu does something under the hood.

## Current AmsNetId location
I already knew that the Windows registery has the AmsNetId as a value under `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Beckhoff\TwinCAT3\System`. The address is as a hexadecimal binary, like in the following figure `c0 a8 05 83 01 01` is `192.168.5.131.1.1`. 

![image](https://user-images.githubusercontent.com/13457157/169379242-c7a20b61-54d1-48ed-9b61-34f31f26026b.png)

I first tried to change it directly editing it in regedit but it just didn't work out. 
## Finding how it is done

As the current address is in the registery, I assumed that the tray icon does something to registery too.

I used a great tool RegistryChangesView by Nir Sofer from https://www.nirsoft.net/utils/registry_changes_view.html - it lists all registery changes that have occured during two snapshots.

By first taking a snapshot of registery with old AmsNetId and then again after changing it to `192.168.5.131.1.2` for testing, we can see clearly what happens:
![image](https://user-images.githubusercontent.com/13457157/169381905-314dcc62-0bca-4c59-bd38-dfa6e130bbf3.png)

A new registery value with name `RequestedAmsNetId` was added to `HKEY_LOCAL_MACHINE\Software\WOW6432Node\Beckhoff\TwinCAT3\System`. So the way to change AmsNetId is not to edit existing one - instead a new temporary value needs to be added!

## Solution

### Using .reg file
Using simple .reg file the AmsNetId can be changed. So create a file like `change.reg` with contents similar to the following (just change the values the one you like).

Note that a reboot is required to really change it.

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\Software\WOW6432Node\Beckhoff\TwinCAT3\System]
"RequestedAmsNetId"=hex(3):C0,A8,05,83,01,02
```

### Using C# 

A simple code using C# under .NET Framework can also do the trick. Note that the application needs to be run as an administrator.

```cs
//Using Microsoft.Win32
var key = Registry.LocalMachine.OpenSubKey(@"Software\WOW6432Node\Beckhoff\TwinCAT3\System", true);
key.SetValue("RequestedAmsNetId", new byte[] { 192, 168, 5, 131, 1, 2 }, RegistryValueKind.Binary);
```
