---
layout: post
title: Old Onda v820w tablet - A new life (Installing Remix OS)
---

In 2015, I bought a cheap Chinese Android / Windows tablet Onda v820w. It had Windows 8.1 and Android 4 installed (Dual-OS). I tought that it could be useful at university (really wasn't) and it didn't cost much (~99 euros).


<https://www.onda-tablet.com/onda-v820w-dual-os-tablet-32gb.html>

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/49d46646-f00a-434c-826a-5ebbb4b2807a)

Later, I upgraded Windows 8.1 to Windows 10. It was possible by removing Android installation to get more space. I remember that it wasn't an easy task, and eventually I even created a step-by-step guide to some forum. The post was quite popular, but sadly, I don't remember the site and haven't found it anymore.

After that, the Onda worked quite well as a small Windows 10 tablet for a while.

## Year 2024

During cleanup of some drawers, I found the tablet. It didn't boot up anymore to Windows for some reason. It displayed a text `preparing automatic repair` and then Windows requested a recovery media.

That sounded like a challenge - I wanted to get the tablet running. I didn't really need it, but at least throwing it to recycling with my personal files in didn't sound good.

And that's how the next evenings were lost...

## Finding information

After looking around in the internet, the biggest issue was old non-working links. As the tablet is quite ancient already, 95% of the links weren't working. Best discussion forum I found was [4pda.to](https://4pda.to), which is a Russian PDA community (luckily Google Translate exists).

**Some links that might be helpful**
* <https://4pda.to/forum/index.php?showtopic=636940> (good discussion, use Google Translate)
* <https://disk.yandex.ru/d/clIpHcR-k7Zdj> (Remix OS image)
* <https://disk.yandex.ru/d/QAtiqrDtkBL9c> (Windows 10 image)
* <https://cloud.mail.ru/public/LN81/hwP8W29Ax> (files, drivers)
* <https://drive.google.com/drive/folders/1lhb29j5tnwyvyZS5KUzEuxNVYBKxaBoa> (files, drivers)
* <https://download.chainfire.eu/696/SuperSU/UPDATE-SuperSU-v2.46.zip?retrieve_file=1> (SuperSU for rooting)
* <https://4pda.to/forum/index.php?showtopic=636940&st=2220#entry44472766> TWRP
* <https://github.com/tmikkonen/sailfish-onda-v820w> (Sailfish OS guide)
* <https://talk.maemo.org/showthread.php?t=96708> (Sailfish OS discussion)
* <https://kezhan.info/2015/04/13/Onda-v820w-flash-os-Booting-to-droidbootimg-failed>
* <https://forum.chuwi.com/t/i-want-to-use-dnx-fastboot-mode-but-the-driver-is-not-installed-on-windows-10/18717>
* <https://groups.google.com/g/remix-os-for-pc/c/b0ij-vCPiQI>
* <https://softwarepen.wordpress.com/2016/04/18/installing-play-store-on-remix-os/>
* <https://groups.google.com/g/remix-os-for-pc/c/29JcJTfbsgk>
* <https://xdaforums.com/t/gapps-10-04-2015-4-4-4-pa-google-apps-plus-all-roms.3074533/>
* <https://drive.google.com/drive/u/3/folders/1gOX-rQYtySNIT10-vSpOPvyXRWedZK29> **(files from this guide)**

## Sailfish OS

I found [a repository by tmikkonen](https://github.com/tmikkonen/sailfish-onda-v820w), that guides how to install Sailfish OS to Onda v820w. It sounded interesting and I had nothing to lose.

The installation was smooth and it really worked! Sailfish OS was now running on the tablet. Everything seem to operate well, including wifi, touch screen and hardware. 

However, updating the OS to newer versions failed, as old links weren't working anymore. The old version of the Sailfish OS was too old, and the only available web browser was extremely awful to use. Also, the browser didn't work very well (bad Javascript support).

## Remix OS (Android 4.4.4 custom ROM)

I found an Remix OS image for exactly this tablet version (Onda v820w **v3**). However at first I had lots of issues to get it working. No matter what I tried, I couldn't run the flashing process. Flashing with **Manufacturing Flash Tool** always ended in fastboot errors:

* `Booting to droidboot.img failed`
* `Process TIMEOUT Failed to execute "fastboot  -s **serial_hidden** -S 100M  oem start_partitioning"`

I started to Google around again. I needed to finalize this, it was a mission.

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/10a670a6-fccf-47ba-b84a-e134fb737d71)


After finding the following two links, I managed to flash the operating system. The issue I had was missing Android drivers, even I had installed them. I had to install them **twice**. When installing the driver for the first time, I managed to start the flashing process. But then I had `TIMEOUT` issues and noticed, that the driver was again faulty. Second install fixed it.

* <https://kezhan.info/2015/04/13/Onda-v820w-flash-os-Booting-to-droidbootimg-failed>
* <https://forum.chuwi.com/t/i-want-to-use-dnx-fastboot-mode-but-the-driver-is-not-installed-on-windows-10/18717>

### The flashing process step-by-step

Maybe someone else wants to do this too? I have uploaded the files needed to Google Drive: https://drive.google.com/drive/folders/1gOX-rQYtySNIT10-vSpOPvyXRWedZK29

Note: This is for Onda v820w v3. Not sure about other versions, but the 4pda.to discussion had some links.

1. Clear all data from the tablet (eMMC)
    * **Probably not needed, but might be a good starting point**
    * Connect USB keyboard to the tablet (you need micro-USB OTG dongle)
    * Power up the tablet and hit ESC repeatedly to get to the boot menu
    * Select **SCU** to get to the BIOS

    ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/21c73fb6-2436-447d-a1ab-08d680784389)
    
    * Go to **Advanced -> LPSS & SCC Configuration**
    * Enabled *eMMC Secure Erase** 
    
    ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/114787c5-3cbd-42d8-acbe-c85786558525)

    * Accept changes and reboot. It will take a while.
    * Now the tablet is empty and it should boot directly to UEFI shell (as there are no operating systems)

2. Install following drivers and applications to your PC
    * [Intel_Android_Driver_v1.9.0](https://drive.google.com/file/d/12hRPizZ-eGdOIGFTRMwwJgNcm_MIllno/view?usp=drive_link)
    * [iSocUSB-Driver-Setup-1.2.0.exe](https://drive.google.com/file/d/1tecb2g3HjsgS5oiVZ-lYW_7WbiRldyqo/view?usp=drive_link)
    * [ManufacturingFlashTool_Setup_6.0.43.7z](https://drive.google.com/file/d/1G9aCw1QbsuKsAg5jgOHiq9_pbGLUIPaH/view?usp=drive_link)

3. Download the [RemixOS zip](https://drive.google.com/file/d/1bveLOoR2HBLE5X10CBcxQuoYCWObMcMJ/view?usp=drive_link) and unzip it
4. Copy `CUSTOM_CONFIG.INI` from the RemixOS zip to the Manufacturing Flash Tool install location (usually `C:\Program Files (x86)\Intel\Manufacturing Flash Tool`)
5. Open **Manufacturing Flash Tool** and select **File -> Settings** and set the following

  ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/bf207005-3bef-4005-b35e-27fdf1024116)

6. Connect the tablet to the PC using a USB cable. Open **Device manager** and check if the connected device is displayed correctly. If you see **Intel Android AD** with a exclamation mark / warning, you need to update its driver.
    * If so, then download [usb_driver.zip](https://drive.google.com/file/d/1XGu54iKDiiOk_POURUci9yohkFM2D8j5/view?usp=drive_link), unzip it, select **update driver** under the device in device manager and select downloaded file `android_winusb.inf`. Then select `Android ADB interface`.

      |Driver status||
      |-|-|
      ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/f8b50e68-a68c-426e-b102-571adb3a0f55) | WRONG!
      ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/c239c242-9d99-4428-b635-ddb99812af78) | OK!


  ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/2e9e3924-04bd-4d75-a37a-29057b1807c0)

6. Now in flash tool, click **File -> Open** and select `flash.xml` file from extracted Remix OS archive. The application starts to monitor for connected Android devices.
7. Shutdown the tablet. Press **volume+**, **volume-** and **power button** at the same time, until the tablet boots into DnX/fastboot mode

  ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/0a8d0857-e0a7-4fe2-9c19-846e4f14e024)

8. Connect the tablet to the PC with USB cable. The flash tool should start flashing.
    * NOTE: If you get errors, open **device manager** and check the step 7 again (yes, again) 
    * My issue was that I needed to install the driver again at this point! 
9. If everything goes well, the flashing takes a few minutes and then you have a fresh (ancient) Android installed

  ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/061b33f3-67e5-49ee-b92e-b45dd9753ad5)

## Rooting the Android on Remix OS

1. Enable USB debugging from Android
2. Connect tablet to the PC
3. Download and copy [UPDATE-SuperSU-v2.46.zip](https://drive.google.com/file/d/1pnPDJXMUPmUcadM7rHpmcHkt-7Et-E3h/view?usp=drive_link) file to tablet memory
4. Download [twrp.zip](https://drive.google.com/file/d/1nMKzRKxzIzT3bJJtrUZERw_RaS8o-LPe/view?usp=drive_link), unzip it and run `recovery.bat`
5. Tablet should now reboot to Droidboot. Select `Recovery`

  ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/3bc691d5-1714-42d7-ab14-bf63ef2f8fd0)

7. Russian TWRP open. Press back button until you are in following menu

  ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/e0af1df7-66bb-4b6b-9c16-9f73d5661061)

8. Select install (upmost). Find the `UPDATE-SuperSU-v2.46.zip` and select it.

    ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/43e1199c-c0ab-4ecf-be3d-dc8e89401ee3)

9. Pull the slider to the right to install
10. Reboot. Device should now be rooted.
 
## Installing Play Store

*You need to root first.*

I managed to install working Play Store by doing the following. Not sure what step is needed and what not, but finally it works.

1. Install [open_gapps-x86-4.4-nano-20220503.zip](https://drive.google.com/file/d/1Fg9njA_0fU_zIBZ7ZCc3KTch7xpfY-kU/view?usp=drive_link) using TWRP
    * See previous chapter, just select different zip
2. Install [GMSActivator.apk](https://drive.google.com/file/d/1Cetg7FOz45IAho4y1nZlLAN2tBfgpQTt/view?usp=drive_link), run it and press **Install Google Services**
    * Some sources say you need to wait long (1 hour?)
    * It never finalized for me but I think Google Service was installed
3. Reboot device
4. Install [Lucky Patcher](https://www.luckypatchers.com/download/)
5. Open Lucky Patcher, select **Toolbox -> Path to Android**
6. Select all patches and apply them. Reboot.
7. Open Lucky Patcher, select **Toolbox -> Install Modded Google Play Store**
8. Install **Modded Google Play 9.8.07**

  ![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/e0cc5afe-70e6-43f9-9fbb-d7fb679738ac)

9. Reboot when requested
10. Test Play Store

If it doesn't work yet, try again.

I installed latest possible Chrome from Play Store successfully.

## Installing Firefox to the Remix OS

If the Play Store isn't working, you can install Firefox manually. The default browser is so old that it's not working very well.

The latest supported Firefox version for this Remix OS (Android 4.4.4) is 
**68.11.0**. You can download it from the following link (NOTE: it has to be **x86** version);

<https://www.apkmirror.com/apk/mozilla/firefox/firefox-68-11-0-release/firefox-browser-fast-private-68-11-0-3-android-apk-download/>

## Conclusion

Now the 9 years old tablet works again and I can use it to monitor electricity prices and weather!

![image](https://github.com/jisotalo/jisotalo.github.io/assets/13457157/60ae29d1-2cf7-4940-9492-b38e7fcf50ae)