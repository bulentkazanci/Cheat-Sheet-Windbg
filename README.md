# Windbg-Cheat-Sheet
A practical guide to analyze memory dumps of .Net applications by using Windbg.


* [Environment](#environment)
* [Dump Generation](#dump-generation)
  * [Manual Dump Generation](#1-manual-dump-generation)
  * [Automatic Dump Generation](#2-automatic-dump-generation)
* [Memory Leak](#memory-leak)
  * [General Heap Check](#user-content-1-general-heap-check)
  * [Check Finalizer Queue and Finalizer Thread](#user-content-2-check-finalizer-queue-and-finalizer-thread)
* [High CPU Usage]()
* [Deadlocks](#deadlock)
* [Static Class,Field etc. Access](#static-field-access)
* [Command List](#command-list)
* [Articles](#articles)

### Environment

- Install SOS.dll(Son of strike) to same path with WinDBG.
- Install psscor4
- Set symbol path 
  - srv*d:\dumps\symbols*http://msdl.microsoft.com/download/symbols
- Run command of .loadby sos clr to load sos.


### Dump Generation
------

#### 1. Manual Dump Generation

There are multiple manners to generate dumps ranging from task manager and debug diagnostic. I prefer to use procdump for manual generation. To generate dump manually, follow instructions below.

- Download procdump from this [link](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump).
- Run command prompt as administrator and swtich the path of procdump. Following is an example from my local.
```
cd \BrkDev\Procdump
```
- Procdump provides a variety of parameters which change characteristic of generated dump. List of parameters can be seen in the link above. I will use following command to get full memory dump of all process memory.
```
procdump -ma [process identifier] [folder path]
```


#### 2. Automatic Dump Generation

Sometimes it is not possible to find a change to collect dump before application crashes. Therefore, Windows Error Reporting can be configures so that dumps can be collected in a situation of crash; however, applications that handles their own custom crash reporting are not supported by this feature. Ultimately, in order to collect dumps automatically, follow the steps below.

- Open Registery Editor(Regedit).
- Go to following record.
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps
```
- If the record does not exist, create it. Otherwise, check the values of following parameters.

| Value           |           Description                          | Type         | Default Value           |
| -------------   |------------------------------------------------|-----------   |-------------------------|
| Dump Folder     | The path where dump files will be stored.      |REG_EXPAND_SZ	|%LOCALAPPDATA%\CrashDumps|
| DumpCount       | Max. number of the dump files in folder.       |REG_DWORD     |    10                   |
| DumpType        | Type of dump file. 2 is Full dump.             |REG_DWORD     |    1                    |
| CustomDumpFlags | Custom dump options.                           |REG_DWORD     |MiniDumpWithDataSegs, MiniDumpWithUnloadedModules, MiniDumpWithProcessThreadData.|

- Locate and delete following registry entries.
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AeDebug\Debugger
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\DbgManagedDebugger
HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\AeDebug\Debugger
HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\.NETFramework\DbgManagedDebugger
```


### Memory Leak
------

#### 1. General Heap Check

- Verify dump file. Make sure that heap is not corrupted.

```
!verifyheap
```

- Look at commited memory with following command. The best command to look at memory as whole.

```
!address -summary
```

Sections in result
  - Image: memory used for executables and dlls.
  - Stack: stack space for threads.
  - https://blogs.msdn.microsoft.com/webtopics/2010/04/02/address-summary-explained/



- Following command shows gc heap and loader heap usages. Look at the amount of space that heaps allocate.
(Reminder: gcheap is collected by gc but loader heap is not. Loader heap is for static objects.) Make sure that both values are under certain size.

```
!eeheap
```

- Run following command to see full list of objects in memory(name, count, size etc.)

```
!dumpheap -stat
```

- If there is an object looking suspicious(having more size or count) dump that object.

```
!dumpheap -type [class name space]
```

- Pick the one allocationg more space than others and go to its details.

```
!do [address]
```



#### 2. Check Finalizer Queue and Finalizer Thread

- list all the objects in memory

```
!dumpheap -stat
```

- Take the suspicious one. following command only gives addresses of objects of that type.

```
!dumpheap -type [type of object] -short
```

- To see what keeps the reference to those objects run following command. the example below is valid for bytearray. Find the one(above) waiting for being finalized.

```
.foreach(bytearr {!dumpheap -type System.Byte[] -short}){!gcroot bytearr; .echo - - - - - }
```

- check the finalizer queue with following command. 

```
!finalizequeue
```

- If there are many objects waiting for finalizing analyze the finalizer thread stack. Switch the finalizer thread.

```
!clrstack
```

### High CPU Usage
------

- Check uptime of threads
```
!runaway
```
- Get list of an arbitrary number of threads on top of the list

- Switch thread contexes one by one

- Check their clr stack

- If all of them waits for the same method. Analyze the method.



### Deadlock
------



### Static Field Access
------
 - List application domains.
```
!dumpdomain
```
- Delve into selected module
```
!dumpmodule  -mt [module identifier]
```
- Dump class
```
!dumpclass [class identifier]
```
- Get value of static field
```
!do [field identifier]
```


### Command List
------

- Evaluates given expression.

```
?
Example usage: `?e186fa28`
```

- Gives list of threads.
```
!threads
```
- Switches thread context to specified id's context.
```
~[id]s
~5s
```
- Gives detailed information about that thread pool.
```
!threadpool
```
- Check this if you look for high cpu usage. You can see which thread works for how long.
```
!runaway
```
- Lists all .net call stacks. Lists contexts of all threads.
```
!eestack
```
- Lists exceptions of current thread.
```
!printexception
```
- Shows usage statistics of heaps. Heap count equals to core counts.
```
!heapstat
```
- If you do not dispose object, it is alive during 2 GC process. If you dispose, it is alive during 1 GC process.
```
!fq(finalizer queue)
```
- Kernel time: cpu operations like siscall, interrupt, tcp call etc.
  User time: cpu operations like read, multiplication.
```
.time
```
- First command related to memory.
```
!address -summary
```
- Gives process info like computer name. Process environment block.
```
!peb
```
- Gives managed heap usage.(.net heap)
  Size of heaps should be nearly equal to each other.
```
!eeheap -gc
```
- Lists all objects on heap.
```
!dumpheap -stat
```
- Gives address list of specified type of objects.
```
!dumpheap -mt [address]
```
- Lists the objects refers to specified address.
  If it says that there is no unique root found, the object will be collected in next gc cycle.
```
!gcroot [address]
```
- Gives details of method.
```
!dumpmd
```
- Gives call stack of current thread.
```
!clrstack
```
- Shows thread locks if there is.
```
!syncblk
```
- Lists modules loaded by application.
```
lm
```
- Makes windbg more talkative - shows detailed messages
```
.srcnoisy 3
```
### Articles
------

- [Guides from Tess Fernandez](https://blogs.msdn.microsoft.com/tess/2007/06/08/blog-post-index/)


