---
layout: post
title: TwinCAT and Codesys - Function block FB_init array initialization
---

I have been using IEC 61131-3 function block `FB_init` methods quite a lot for different library blocks. They allow almost constructor-like initialization possibilities for function blocks. For example it's quite useful to pass buffer size to a block using FB_init and then initialize a dynamic array using `__NEW` operator.

```
DataLogger  : FB_Logger(BufferSize := 2000);
StateLogger : FB_Logger(BufferSize := 500);
```

```
METHOD FB_init : BOOL
VAR_INPUT
	bInitRetains : BOOL; // if TRUE, the retain variables are initialized (warm start / cold start)
	bInCopyCode : BOOL;  // if TRUE, the instance afterwards gets moved into the copy code (online change)
	BufferSize	: UDINT;
END_VAR
//Code:
//Note this is not a fully working correct example
Buffer := __NEW(BYTE, BufferSize);
```

However I just had a situation that I needed to have array of function block instances that all have a FB_init parameter. 

The [Beckhoff Information System had an example](https://infosys.beckhoff.com/english.php?content=../content/1033/tc3_plc_intro/5094414603.html&id=6967794353598129051) of an array with different parameters, so I knew it was possible (phew).
```
PROGRAM MAIN
VAR
    fbSample  : FB_Sample(nInitParam := 1) := (nInput := 2, nMyProperty := 3);
    aSample   : ARRAY[1..2] OF FB_Sample[(nInitParam := 4), (nInitParam := 7)]
                            := [(nInput := 5, nMyProperty := 6), (nInput := 8, nMyProperty := 9)];
END_VAR
```

However in my case the array size was based on a CONSTANT, the initial values were all the same and updating them manually would be too time-consuming. Finally I found that it's possible, like the following (all instances get 123 to FB_init):
```
Example : ARRAY[0..ARRAY_MAXIMUM] OF FB_Block(123);
```

Here are the different possibilities for initialization.

```
//FB_Block
METHOD FB_init : BOOL
VAR_INPUT
	bInitRetains : BOOL; // if TRUE, the retain variables are initialized (warm start / cold start)
	bInCopyCode : BOOL;  // if TRUE, the instance afterwards gets moved into the copy code (online change)
	MyParam	: INT;
END_VAR
//Code:
Prm := MyParam;
```

```
{attribute 'qualified_only'}
VAR_GLOBAL
	//Single block instance
	SingleFB_1 : FB_Block(MyParam := 1);
	SingleFB_2 : FB_Block(1);
	
	//Array of function block instances with different FB_init parameters
	ArrayFB_1	: ARRAY[0..2] OF FB_Block[(MyParam := 1), (MyParam := 2), (MyParam := 3)];
	ArrayFB_2	: ARRAY[0..2] OF FB_Block[(1), (2), (3)];
	
	//Array of function block instances with same FB_init parameter
	ArrayFB_3	: ARRAY[0..2] OF FB_Block(MyParam := 1);
	ArrayFB_4	: ARRAY[0..2] OF FB_Block(1);
END_VAR
```

Results:
![image](https://user-images.githubusercontent.com/13457157/202921567-042fc20a-b2fd-4ebb-82c7-44d679e89a35.png)

