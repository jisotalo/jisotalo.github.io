---
layout: post
title: Running TwinCAT 3 projects locally with Usermode Runtime
---

I have always disliked the feature of the TwinCAT 3 that you need to run it in your development PC kernel level, just like on a real PLC hardware.
Other systems, such as TwinCAT 2, Codesys 2 and Codesys 3, run the PLC software in separate soft-PLC application on Windows.

As the TC3 is running in kernel, having different catastrophic faults (divide by zero, null pointer, etc..)  can lead to the blue screen of death (BSOD) in Windows. This causes unnecessary reboots and loss of data, in addition to the wasted working hours. Not to mention forever-loops and other "user-errors" that can happen during development.

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/7e1e0b2f-5eb6-4c3b-97e7-50d6ef2edac5)

Other issues I remember facing:
* adminstrator privileges are required
* some BIOS setting changes are required
* HyperV isn't supported
* some processors aren't supported
* running virtual machines isn't supported (such as VirtualBox or WSL 2)

## Common TwinCAT 3 start-up errors

When installing TwinCAT and trying to run it locally, you might end up struggling with some of the following errors:

* `TwinCAT System' (10000): Sending ams command >> Init4\RTime: Start Interrupt: Ticker started >> AdsError: 4132 (0x1024, RTIME: incompatible software detected) << failed!`

* `'TCRTIME' (200): start of real-time avoided by "HyperV"`

* `Setting TwinCat in Run Mode inside HyperV (virtual machine) is not possible`

* `Intel VT-x is not supported or not enabled in the bios`

* `'TwinCAT System' (10000): Sending ams command >> Init4\RTime: Start Interrupt: Ticker started >> AdsWarning 4120 (0x1018, RTIME: enter real-time mode fails: Intel VT-x extension not enabled (in Bios)!) << failed!`

* `'TwinCAT System' (10000): Sending ams command >> Init4\RTime: Start Interrupt: Ticker started >> AdsError: 4136 (0x1028, RTIME: enter real-time mode fails: AMD-V extension not enabled (in Bios)!)`

* `'TwinCAT System' (10000): Sending ams command >> Init4\RTime: Start Interrupt: Ticker started >> AdsWarning: 4115 (0x1013, RTIME: system clock setup fails. Hint: On Windows8 system and above execute win8settick.bat in TwinCAT\3.1\System as administrator and reboot.) << failed!`
  * Note: This is usually fixed easily by running one .bat file - read the answer here: [https://stackoverflow.com/a/54266445](https://stackoverflow.com/a/54266445)

## Traditional fixes

If you want to get TwinCAT running in real-time mode (kernel), there are quite many different fixes to do it. Usually it's totally doable but you need to tweak different settings, such as BIOS and virtualization.

Some cases it's not possible to change those settings or you might have a processor that still has issues - personally I have had problems with Intel i7 processors where nothing worked. Or maybe you just don't want to do any of these tweaks.

Usually final solution is to isolate one of the processor cores to TwinCAT use only. This usually helps - naturally the core is no longer available for other use.

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/2aa08e65-d650-4fc9-a764-6bad389b7126)

If you want to fix the issue, try to following links for starters (not in particular order). 
* https://stackoverflow.com/questions/54264180/twincat3-adswarning-4115-system-clock-setup-fail
* https://stackoverflow.com/questions/66698779/twincat-activate-configuration-error-0x1028-unable-to-activate
* https://stackoverflow.com/questions/67481562/cant-set-any-isolated-cores-in-local-virtual-twincat-plc
* https://stackoverflow.com/questions/59463238/twincat-realtime-startup-of-isolated-cpu-fails

However, if you want to try something else, check out the next chapter.

## Running PLC code in TwinCAT Usermode (soft-PLC / non-real-time)

Since TwinCAT 3.1.4024.17 (?), there has been a new feature available as beta version in TwinCAT installation: **[TwinCAT 3 Usermode Runtime](https://www.beckhoff.com/fi-fi/products/automation/twincat/tc1xxx-twincat-3-base/tc1700.html)**. It's a TwinCAT 3 runtime running in Windows terminal as a normal application. You can start it with a batch file and close it when needed - by just closing the terminal.

Direct quote from Beckhoff website:
```
The TwinCAT 3 Usermode Runtime provides a way to run the applications programmed in TwinCAT without real-time properties in the user mode of the operating system. The TwinCAT 3 Usermode Runtime can be used free of license costs purely for engineering purposes and only requires (trial) licenses of the TwinCAT products used.

This variant makes it possible to run customer programs, so that in the case of engineering systems in particular, which permit neither installation nor operation of the real-time runtime, the test run can take place for development purposes.
```
You can run TwinCAT PLC application on your PC without doing any BIOS/virtualization fixes and without worrying the faulty PLC code would crash your PC!

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/4e13938a-86e2-4f93-a57b-e54690145d39)

It seems Beckhoff has created a separate TwinCAT OS that is now available to be run under windows. Most probably it's related to TwinCAT/BSD and TwinCAT-OS. In the directory `C:\TwinCAT\3.1\Runtimes\bin\Build_4024.50`  you can see different libraries, such as `TcOsSys.dll` (most probably is the magic here) and `TcSystemServiceUm.exe` (which starts the TwinCAT).

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/c37853c5-dc33-46c7-94db-3db6230ca70b)

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/ac7c04f2-7669-48e6-bd6f-59ae6b428518)

### How to start the Usermode Runtime

1. Open `C:\TwinCAT\3.1\Runtimes`
2. (Take a backup-copy of `UmRT_Default` folder just in-case)
3. Open `C:\TwinCAT\3.1\Runtimes\UmRT_Default`
4. Click `Start.bat`. The TwinCAT should start in config mode
5. The AmsNetId can be seen in terminal output (and of course changed by editing `Start.bat`)


![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/eea9be04-2aec-4e9b-bb19-7b8830940738)

**Example terminal output:**
```
C:\TwinCAT\3.1\Runtimes\UmRT_Default>C:\TwinCAT\3.1\Runtimes\bin\Build_4024.50\TcSystemServiceUm.exe -i 192.168.4.1.1.1 -n UmRT_Default -c ".\3.1"
TcSysSrvUm: started
heap memory allocated 0000020A057C0000 size=8000000
TcSysSrvUm state: Config
AmsNetId: 192.168.4.1.1.1
TcSysSrvUm state: Config

Press 'c' for Reconfig TwinCAT System.
Press 'r' for Restart TwinCAT system.
Press 's' to view current state.
Press 'x' to exit TwinCAT system service.
```

### Downloading and running PLC software in Usermode Runtime

This is very straightforward. Just open your PLC project and select `UmRT_Default` as target. It should be visible when the runtime is started using `Start.bat` and it's still open.

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/d95dd5f3-fd89-47ad-8061-f2f16fc8b655)

When the system is set to run mode, the terminal displays `TcSysSrvUm state: Run`. In the figure below you can also see that errors by system and `ADSLOGSTR()` are displayed in terminal:

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/5a3d4ab3-4485-47c8-8a26-de72a2653816)

The `Boot` directory can be found under `C:\TwinCAT\3.1\Runtimes\UmRT_Default\3.1\Boot` which contains PLC binaries and persistent data (just like in normal PLC). So if you have any problems with the project, you can always delete all contents to start over.

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/3445a001-5edc-44df-b90e-a6b171fa2a60)

## Conclusion

I have been using the Usermode for around last 6 months as my day job PC has issues running in kernel mode. 

It has been a pleasure, even when the product is still in beta. If I mess up something in code, only the Usermode application will crash/hang instead of whole PC. And it's much faster to start!

Some issues I have noticed:
* TwinCAT displays PLC CPU usage as 0% or 99% even though it's working normally (so 99% doesn't mean PLC code is running a forever loop)
* `MEMCPY()` is not working with overlapping memory areas
  * Yes - it shouldn't work in other system either but it still works in my experience
  * Old codebase I have been working with has had lot's of this kinds of operations that caused problems with Usermode Runtime
  * Converting `MEMCPY()` calls to use `MEMMOVE()` instead fixed the issues
* The Usermode Runtime sometimes crashes during PLC program download by just disappearing
  * Adding `pause` to end of `Start.bat` helps so the window doesn't just disappear
  * Probably still some Beta issues
* Page faults, null pointers and divide-by-zero problems sometimes crash the PLC system
  * The window just disappears
  * Adding `pause` to end of `Start.bat` helps so the window doesn't just disappear
  * Probably still some Beta issues
* Some memory full errors during software download
  * Clean & rebuild usually helps
  * If not then final solution has been deleting the `C:\TwinCAT\3.1\Runtimes\UmRT_Default\3.1\Boot` directory

I really like this new Usermode Runtime and I hope it will get developed further. Looking forward for TwinCAT 3.1.4026 release! Maybe some day this is the default instead of kernel mode.

![smoke](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/6caa9986-31c4-46f3-983c-ef7b1fc3cd8e)
