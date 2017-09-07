# Windbg-Cheat-Sheet
A practical guide to analyze memory dumps of .Net applications by using Windbg.


* [Environment](#environment)
* [Memory Leak](#memory-leak)
  * [General Heap Check](#user-content-1-general-heap-check)
  * [Check Finalizer Queue and Finalizer Thread](#user-content-2-check-finalizer-queue-and-finalizer-thread)
* [Deadlocks](#deadlock)
* [Command List](#command-list)
* [Articles](#articles)

### Environment

- Install SOS.dll(Son of strike) to same path with WinDBG.
- Install psscor4
- Set symbol path 
  - srv*d:\dumps\symbols*http://msdl.microsoft.com/download/symbols
- Run command of .loadby sos clr to load sos.



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


### Deadlock
------




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
### Articles
------

- [Guides from Tess Fernandez](https://blogs.msdn.microsoft.com/tess/2007/06/08/blog-post-index/)


