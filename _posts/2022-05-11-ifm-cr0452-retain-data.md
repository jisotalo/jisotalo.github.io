---
layout: post
title: ifm CR0452 RETAIN data problem
---

In one project I have been working with [ifm CR0452 R360/BasicDisplay XL HMI](https://www.ifm.com/fi/en/product/CR0452) screen. It's quite nice small, and not too costly, HMI for mobile machines 
and it can be programmed using CODESYS 2.

![image](https://user-images.githubusercontent.com/13457157/167919044-92c529c1-8666-4c67-be3f-e234bc03d5aa.png)

The program had lot's of settings saved into RETAIN memory. Previous programmer had made a `PROGRAM` that checks if each setting variable is in allowed range. 
The program took the active value as input and then set the checked value to output. So no `IN_OUT` was used.

The problem was that some settings were randomly lost. It's not very nice to setup the system every time mobile machine has been powered on.

At first, I got some ideas that might cause the problem in this case
* The data isn't saved automatically
* The data is saved too often and memory has been corrupted
* Defining data inside `STRUCT` isn't correctly supported
* PLC code has a bug that causes data to lose
* Something else???

## Documentation

Even though ifm products have been proven quite nice to work with, the documentation isn't the best. 
And you can't find anything useful from Google unlike with TwinCAT or basic CODESYS, as it's not so popular platform. Or at least the community is not that open.

The programming documentation for this screen is [here](https://www.ifm.com/mounting/7391002UK.pdf) as PDF format.

## Possible problems

### The data isn't saved automatically

In some system, like TwinCAT 3 (PERSISTENT), remainent data needs to be saved manually using a function block.

By looking the PDF, it seems that the CR0452 loads and saves retain data automatically. So the problem isn't most probably related to that.
![image](https://user-images.githubusercontent.com/13457157/167922415-b1122275-f958-48e1-9a86-afd4141549d2.png)

### The data is saved too often and memory has been corrupted

Some other previous programmer had used RETAIN data to save also a heartbeat signal that changes all the time. 
If the data is saved automatically, could it destroy the memory? For example, Arduino EEPROM doesn't allow many save cycles until it is destroyed.

By looking at the PDF, it seems that retain data is saved into FRAM memory. 
From Google I found that it shouldnt't have that kind of problem and it should have [billions of write-cycles](https://www.symmetryelectronics.com/blog/the-lifetime-of-fram-memory-is-essentially-unlimited/).

![image](https://user-images.githubusercontent.com/13457157/167921667-e7cd4941-d69f-4234-bd9d-1bc18bf1f736.png)

### Defining data inside `STRUCT` isn't correctly supported

As I mostly work with quite modern systems (TwinCAT 3, C#, etc..), it's normal to use quite complex structures. However, with older systems (CODESYS 2..) I wouldn't be so sure about the complex datatype support.

However, I couldn't find any reason why the structs couldn't work in retain memory and as the data wasn't lost always, so I just needed to hope it isn't the reason.
*(Please, let me use structs instead of single variables)*

### PLC code has a bug that causes data to lose

As the CR0452 doesn't support breakpoints (or at least I couldn't get them working), the debugging is god damn hard.

However, what if the PLC code was the reason? As I mentioned, no IN_OUT was used in valid parameter range checking.

I couldn't find any reason for the problem from the code. Perhaps the program should still be changed so that it won't create new copies of variables, just edit them directly.

### Something else???

Finally I got a stupid idea: What if the variables weren't yet loaded when the first PLC cycle runs? I added a timer that didn't let the main PLC code run for the first few seconds.

**That worked!**

![image](https://user-images.githubusercontent.com/13457157/167924801-465109db-946a-4ce6-a10f-22ecca3a2cd7.png)

## Conclusion

So the problem was combination of the PLC code and the CR0452 system.

It seems that the RETAIN variables __might not be yet loaded during the first PLC cycle__ in some ifm products.

As the parameter check program was run during the first PLC cycle, it took uninitialized retain variables as inputs and the wrote them back, which caused original data to be lost.




