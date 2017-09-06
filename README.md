# Windbg-Cheat-Sheet
A practical guide to analyze memory dumps of .Net applications by using Windbg.



* [Memory Leak](#memory-leak)
* [Deadlocks](#deadlock)
* [Command List](#command-list)


### Memory Leak
------

#### 1. Check Finalizer Queue and Finalizer Thread

- list all the objects in memory

`!dumpheap -stat`

- Take the suspicious one. following command only gives addresses of objects of that type.

`!dumpheap -type [type of object] -short`

- To see what keeps the reference to those objects run following command. the example below is valid for bytearray. Find the one(above) waiting for being finalized.

`.foreach(bytearr {!dumpheap -type System.Byte[] -short}){!gcroot bytearr; .echo - - - - - }`

- check the finalizer queue with following command. 

`!finalizequeue`

- If there are many objects waiting for finalizing analyze the finalizer thread stack. Switch the finalizer thread.

`!clrstack`

2. 


### Deadlock
------




### Command List
------

- Evaluates given expression.

`?`

Example usage: `?e186fa28`

- 



