---
layout: post
title: Reading TwinCAT variables dynamically using C# (.NET 7)
---

I have been working with TwinCAT 3 communication using Javascript, Typescript and C#.

Lately I have been doing a project using .NET 7 and ASP.NET Core. In many use cases the frontend needs some data from PLC and then backend should provide it as json, nothing else (the data would go as-is from PLC to frontend).

Traditionally this causes some extra work as you need to provide the PLC structs also in C# code to be able to serialize it to json. In Javascript using [ads-client](https://github.com/jisotalo/ads-client/) it's easy: call `ReadSymbol()`, serialize data and that's it.

After making some research on latest [TwinCAT.Ads.dll](https://www.nuget.org/packages/Beckhoff.TwinCAT.Ads) (v. 6.x) and doing some testing, it seemed to be possible. It's possible to read any PLC value, get it as [`dynamic`](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/using-type-dynamic) type and the serialize it. And vice-versa.

<video autoplay muted loop src="https://user-images.githubusercontent.com/13457157/226168174-486438d3-615e-4db5-91c9-712a63a12b4d.mp4"></video>

## Example project

There is an example project available at [https://github.com/jisotalo/reading-twincat-variables-dynamically-using-csharp](https://github.com/jisotalo/reading-twincat-variables-dynamically-using-csharp)

## Data types at PLC
Let's assume we have following types in the PLC.
```
TYPE ST_Structure :
STRUCT
	TextValue : STRING(50) := 'some string data';
	RealValue : REAL := 20.23;
	DateValue : DT := DT#2023-3-18-19:33:05;
	SubStruct : ST_SubStructure;
END_STRUCT
END_TYPE
```

```
TYPE ST_SubStructure :
STRUCT
	SubTextValue	: STRING := 'This is substruct';
	SubArrayValue	: ARRAY[0..4] OF INT := [1, 2, 3, 4, 5];
END_STRUCT
END_TYPE
```
An instance of `ST_Structure` is created under GVL `GVL_Test`.
```
//GVL_Test
{attribute 'qualified_only'}
VAR_GLOBAL
	StructValue : ST_Structure;
END_VAR
```
Therefore the instance path is `GVL_Test.StructValue`.

## Reading data using Javascript

Just an example how it's done using [ads-client](https://github.com/jisotalo/ads-client/).

Note that you don't need to specify the data structure.
```js
try {
  const res = await client.readSymbol('GVL_Test.StructValue');
  console.log('Value read:', res.value);

} catch (err) {
  console.log('Reading failed:', err);
}
```

## Reading data using C# and TwinCAT.Ads.dll 4.x


Traditionally data transfer requires that the PLC data structure is specified in the C# code in a way that the data would be marshalled correctly. Also the byte alignments (pack mode) need to match in both PLC and C#. 

Frist we need to add `{attribute 'pack_mode' := '1'}` above struct definitions:
```
{attribute 'pack_mode' := '1'}
TYPE ST_SubStructure :
//...
```

Then in C# code we need to define data types in C# code following way:
```C#
[StructLayout(LayoutKind.Sequential, Pack = 1)]
public class ST_Structure
{
  [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 51)]
  public string TextValue;
  public float RealValue;
  public uint DateValue;
  public ST_SubStructure SubStruct;
}

[StructLayout(LayoutKind.Sequential, Pack = 1)]
public class ST_SubStructure
{
  [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 81)]
  public string SubTextValue;
  [MarshalAs(UnmanagedType.ByValArray, SizeConst = 5)]
  public ushort[] SubArrayValue;
}
```

Now it's possible to read data and cast it to defined type. Note that the 4.x version does not have async methods so you need to wrap it with `Task.Run()` if needed.

```C#
uint handle = client.CreateVariableHandle("GVL_Test.StructValue");
var data = (ST_Structure)client.ReadAny(handle, typeof(ST_Structure));
client.DeleteVariableHandle(handle);

Console.WriteLine(JsonConvert.SerializeObject(data, Formatting.Indented));
```

## Reading data using C# and TwinCAT.Ads.dll 6.x

The 6.x version seems to be a full rewrite or atleast it is dramatically evolved. There are now async methods available, so you it's easier to work with.

### Reading in a traditional way

It's of course still possible to read as before, but the data type can be provided as T and await can be used, which is nice.

```C#
var handle = await client.CreateVariableHandleAsync("GVL_Test.StructValue", CancellationToken.None);
var data = await client.ReadAnyAsync<ST_Structure>(handle.Handle, CancellationToken.None);
await client.DeleteVariableHandleAsync(handle.Handle, CancellationToken.None);

Console.WriteLine(JsonConvert.SerializeObject(data, Formatting.Indented));
```


### Reading dynamically (hard-coded variable names)

The documentation is quite awful about the new features. I found some clues from the following link:
- https://infosys.beckhoff.com/english.php?content=../content/1033/tc3_ads.net/9407528715.html&id=2830513048322293723

Based on the link, it should be possible to do some magic first and then access data using dot notation, such as `Symbols.GVL_Test.StructValue`, without providing any data types manually.

The final solution was the following. First we create a symbol loader, read all symbols and then access value by hard-coded name.

```C#
//Creating a dynamic symbol loader
IDynamicSymbolLoader loader = (IDynamicSymbolLoader)SymbolLoaderFactory.Create(client, SymbolLoaderSettings.DefaultDynamic);

//Loading symbols - note that this is quite expensive operation
ResultDynamicSymbols resultSymbols = await loader.GetDynamicSymbolsAsync(CancellationToken.None);
dynamic symbols = resultSymbols.Symbols;

//Accessing GVL_Test.StructValue and reading its data (note, no data type defined anywhere!)
var data = await symbols.GVL_Test.StructValue.ReadValueAsync(CancellationToken.None);

Console.WriteLine(JsonConvert.SerializeObject(data, Formatting.Indented));
```

The cons of this is that the variable instance path is hard-coded (`GVL_Test.StructValue`). You can't use a variable as instance path. For example, it would be impossible to create a method `ReadValue(string plcAddress)` or similar. **And naturally you lose all type error checking / strong typing!**


### Reading dynamically (dynamic variable names)

Having hard-coded variable names is not an option for me. It makes the project very hard to maintain.

The Beckhoff documentation didn't have any solution. All I found was how to do this to any other types than struct:
* https://infosys.beckhoff.com/english.php?content=../content/1033/tc3_ads.net/9407527691.html&id=3569771638928570568

Finally, after lot's of testing, the solution was found. The symbols need to be read using `IAdsSymbolLoader` instead of `IDynamicSymbolLoader`. Then symbols are read using `GetSymbolsAsync()`.

When accessing the symbol, it needs to be casted to `DynamicSymbol` which allows dynamic data reading.

```C#
//Creating a symbol loader
SymbolLoaderSettings settings = new SymbolLoaderSettings(SymbolsLoadMode.DynamicTree);
IAdsSymbolLoader dynLoader = (IAdsSymbolLoader)SymbolLoaderFactory.Create(client, settings);

//Loading symbols - note that this is quite expensive operation
var symbols = (await dynLoader.GetSymbolsAsync(CancellationToken.None)).Symbols;

//Accessing GVL_Test.StructValue symbol, casting it to DynamicSymbol and reading its data (note, no data type defined anywhere!)
var data = await (symbols["GVL_Test.StructValue"] as DynamicSymbol).ReadValueAsync(CancellationToken.None);

Console.WriteLine(JsonConvert.SerializeObject(data.Value, Formatting.Indented));
```

Thanks to this, we can create for example a simple `ReadValue(string plcAddress)` helper method:

```C#
async Task<dynamic> ReadValue(string plcAddress) 
{
  var symbol = await (symbols[plcAddress] as DynamicSymbol).ReadValueAsync(CancellationToken.None);
  return symbol.Value;
}
```

And for example a web API to read anything:
```C#
//read-value-by-name?variableName=GVL_Test.StructValue
app.MapGet("/read-value-by-name", async (string variableName) =>
{
  var data = await ReadValue(variableName);
  return Results.Text(JsonConvert.SerializeObject(data), "application/json");
});
```

You can also use device notifications with dynamic types:
```C#
var symbol = (symbols["GVL_Test.StructValue"] as DynamicSymbol);

symbol.NotificationSettings = new NotificationSettings(AdsTransMode.OnChange, 1000, 0);
symbol.ValueChanged += new EventHandler<ValueChangedEventArgs>((sender, e) =>
{
  Console.WriteLine($"Symbol {e.Symbol.InstancePath} changed: {JsonConvert.SerializeObject(e.Value)}");
});
```

### Writing data

Writing data is also possible using `WriteValueAsync`.

Simple example of a struct variable:
```C#
var symbol = (symbols["GVL_Test.StructValue.TextValue"] as DynamicSymbol);
await symbol.WriteValueAsync("a new string value", CancellationToken.None);
```

## Conclusion

Handling dynamic data is trickier with C# when comparing to Javascript world. However this opens a lot of possibilities. 

Personally I will still use strict typing in many cases. But sometimes, like with web APIs or simple projects, using dynamic types might be a good and fast solution.

