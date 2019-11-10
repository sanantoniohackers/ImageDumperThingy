# ImageDumperThingy
This script was written to beat any driver where the source has been obfuscated. All this does is pulls it from memory so you can view it statically rather than a bunch of garbage. Sometimes it can dump it almost perfectly, sometimes you have to fix the image base, and sometimes you have to fix the IAT and what not so your mileage may very if you use this. You can also do this yourself with a few commands, but this just makes it easier. The results whether you use this or those commands will still be the same.
For example, aksfridge is an obfuscated driver and when viewed from IDA, it comes out as garbage and seems like a nightmare:    

![aksfridge](https://github.com/ch3rn0byl/Driver-Puller-Thingy/blob/master/Images/wtf.PNG)

# How to use it?
WinDbg SHOULD load the JavaScript dll by default. You can check it with the following command:
```
2: kd> .scriptproviders
Available Script Providers:
    NatVis (extension '.NatVis')
    JavaScript (extension '.js')
```
If JavaScript is shown, you're good to go otherwise you can do the following command to load it:
```
2: kd> .load jsprovider.dll
```
Next is to load the script:
```
2: kd> .scriptload C:\Users\ch3rn\Desktop\DumpModule.js
JavaScript script successfully loaded from 'C:\Users\ch3rn\Desktop\DumpModule.js'
```
Once loaded, you can kinda get the picture what's going on:
```
2: kd> dx -r1 -v Debugger.State.Scripts.DumpModule.Contents
Debugger.State.Scripts.DumpModule.Contents                 : [object Object]
    host             : [object Object]
    WriteToFile     
    GetAmountOfModules
    DumpDriver      
    ToDisplayString  [ToDisplayString([FormatSpecifier]) - Method which converts the object to its display string representation according to an optional format specifier]
```
This just shows you the functions that are in this script. You can either run it or save it to a variable so you can call it later on rather than some long ass command:
```
2: kd> dx @$myscript = Debugger.State.Scripts.DumpModule.Contents
@$myscript = Debugger.State.Scripts.DumpModule.Contents                 : [object Object]
    host             : [object Object]
```

Now actually running it!
```
2: kd> dx @$myscript.DumpDriver("aksfridge")
>>> There are 193 loaded modules!
>>> Searching for aksfridge...found it!
>>> Module Name: aksfridge.sys
>>> Module Base Address: 0xfffff800576b0000
>>> Module Size: 0x26000
>>> Dumping aksfridge from memory starting at fffff800576b0000...done!
>>> Writing aksfridge to disk...done!
>>> Done!

@$myscript.DumpDriver("aksfridge")
```

Once this is done, there would be a new filename written to disk. In this case, it will be aksfridge_dumped_by_windbg.sys. If it was an obfuscated driver, you can now view it much easier in Ida and actually get an idea of what's going on. 

---

Then when I put the dumped binary into Ida, I get this:    

![output](https://github.com/ch3rn0byl/Driver-Puller-Thingy/blob/master/Images/fml.PNG)

---

These two files compared is slightly different in size but that's okay because it is still functional:    

![comparison](https://github.com/ch3rn0byl/Driver-Puller-Thingy/blob/master/Images/comp.PNG)

The size is calculated by the highest PointerToRawData plus the SizeOfRawData. Some drivers, like this one, will be different in size because it will have data in IMAGE_DIRECTORY_ENTRY_SECURITY that will definitely add to the size of the PE. This will not impact what you see :)

---

The output from the debugger is the same as what's shown in Ida in several places as you can see in the image below:    

![sick](https://github.com/ch3rn0byl/Driver-Puller-Thingy/blob/master/Images/sick.png)

The ImageBase can be fixed up that way it doesn't look gross inside IDA, but that's okay. I can always fix this to do that, or you can fix the ImageBase yourself when you get to it.

---

# ToDos:
  1. Not sure, but this can always get worked on more to become better
