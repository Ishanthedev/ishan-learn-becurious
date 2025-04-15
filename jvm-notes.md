
## Java notes 
**Creation of jar**

```
jar -cvf <JARname>,jar *.class
```

**Some common error in Java compile**
```
<javaname>.java:line: error: package
<packageName>does not exist
```
```
<JavaName>.java:line: error: cannot find symbol
<className>"
```
```
"java.lang.NoClassDefFoundError:<className>"
"Java.lang.ClassnotfoundException:<ClassName>"
```

## Exception in jVM
Exception stacktrace:
**Exception stacktrace contains the following information:**

1.Error message
2.java classes or native code, involved
3.Code line number of the java(not compiled)code
Line numbers are of not complied code(source) allowed to track down the code
execution method flow for troubleshooting

## JVM
* The Java virtual machine is an abstraction computing machine that provides runtime environment to the java code or application.

* The java code is first complied into bytes code to generate a class file.This class file is then interpreted by the java virtual machine for underlying platform.

* When you download java and get java runtime, environment(JRE)
The JRE consists of the java virtual machine(JVM), java platform
core classes, and the supporting java platform libraries, all
three are required to run java application on your computer

**How to get details of the memory area?**
There are many memory area in JVM, runtime process that are not managed by garbage collection Metaspace, compiler, Memory Tracking, thread stack, etc.

To get the summary of spaces please run the java process with an option
```
-XX:nativeMemoryTracking=summary or
-XX:nativeMemoryTracking=detail to understand the space that are used
```

```
java -XX:nativeMemoryTracking=summary -cp <classpath> <Entrypoint>
jcmd <PID> VM.native_memory summary
```

## Common Memory error:
```
java.lang.OutofMemoryError: Java heap space
Cause: Heap space is no sufficient. GC throws this error
```
**Action:** Check if the load on the process is high, then try increasing the memory heap size(-Xms) and relaunch the process if the process is still throwing OOM error, after increasing the heap check the memory leak by analyzing heap dump(look for any open source/known issues)
```
Java.lang.OutofMemoryError:GC Overhead limit exceeded
```
**Cause:** This means, GC is running all the time, but is very slow in collecting object

**Actions:** Check for any caches data structures in the process and see if we can flush them to disk or persist them
```
Java.lang.OutofMemoryError:Unable to create new native thread
```
**Cause:** This is not memory error, but it is related to the process not being able to create a thread
when you receive this error, all the process running on the linux server will start failing to create thread 
basically a thread leak

**Action:** Using thread dump we need to identify the process that is not feeing up the thread and fix it.
```
OutofMemoryError:kill process or sacrifice child
```
**Cause:** Operating system has a job which monitors the memory allocation of all the process this job monitors the low memory situation and kills the rouge process which is consuming more memory resources.

**Action:** Ensure system as sufficient memory

## Garbage collector
Garbage collection is the process of looking
1. At the heap memory
2. Identifying which object are in use and which are not
3. Deleting the unused object.

* An in use object or referenced object means that some part of the your program still maintains a pointer to that job an unused object or unreferenced object, is no longer referenced by any part of your program.
so the memory used by an unreferenced object can be reclaimed.

* GC frees the developer from the burden of manual memory management.

* In the language like C and C++ there are API's to allocate and deallocate memory malloc(),calloc() etc..
 if you forget to deallocate it will result memory leak

**Memory management Strategies:**
**Mark and sweep:**
1.Simplest GC algorithm
2.Divides memory into two regions: used an unused
3.Marking phase identifies used memory
4.sweeping phase reclaims unused memory
**Copying:**
1. Dividies memory into 2 survivors space regions: from space and "to space"
2. New OBjects are allocated in from spaces
3. When from space is full, live objects are copied to "to space"

**Mark and compact:**
1. Similar to mark and sweep
2. After matching, live object are compacted into one regions Generational.
3. Divides memory into young and old generation
4. Young generation has a higher turnover rate
5. Old generation is larger and is cleaned only when Major GC runs

**GC and memory Generation**
1. All GC algorith can be configured by passing runtime option to the java process
2. What is stop the world?
3. Minor/Major GC
4. Full GC

**Amazon EMR moved to parallel GC from 5.36** 
Garbage collector:
**Serial collector:**
The serial collector uses a single thread to perform all the garbage collection work No thread overhead. It is the best suited to single processor.

**Parallel Collector:**
The Parallel collector is also known as throughput collector, its a generation collector similar to the serial collector, (*multiple threads)

**Garbage-First(G1)  Garbage collector**
Designed to scale from small to large multiple processor machines with a large amount of memory.

**The Z Garbage collector**
ZGC provides max pause times under a millisecond, but at the cost of throughput it is intended for application that is required low latency

**G1GC:**
G1 is a generational, incremental, parallel mostly concurrent, stop the work and evacuating gargage collector

**G1GC features:**
1. More predictable pauses
2. Work with large heaps(Due to incremental nature)
3. Reasonable worst case response times
Recommended use case
1. Application that requires large heaps with limited GC
heap sizes large than 6GB and stable and predicable pause time below 0.5 seconds
**G1GC logs:**
Full GC(very time consuming)>> caused by too high heap occupancy in the old generation
How to detect?
``(-verbose -xx:+ UseG1GC -XX:+ PrintGCDetails)``
Limit FUll GC
``-xx:G1HeapRegionSize(Increase region size, for example if too many humongous object)``
increase the size of Java help
Increase ``-xx:COncGCThreads``
Force G1 to start making earlier. ``-XX:G1ReservePdercent``

**G1GC tuning**
latency -- Pause includes by the JVM as it performs garbage collection
```
-Xmn,-XX:NewRatio(don't limit eden)
-Xms -Xmx(equal)
```
Throughput --The percentage of the wall clock time the JCM has available
for executing the application
``-XX: MaxGCPauseMillis``(Increase)
Footprint --the allocation heap.
``-XX:GCTimeRatio``(Decrease)

You might want to use **jstack:**
Your app is **frozen or unresponsive**
It's **running slower than usual**
You suspect there might be a deadlock (when threads get stuck waiting for each other)
To use jstack, you simply run it with the process ID of your Java application:
**jstack process-id**

What you'll find is a treasure trove of information:
A list of all the threads running in your application
What each thread is doing at that moment
Any locks or synchronization issues
Potential deadlocks

**A thread dump**
A light weight process. Each and every line of code is Java is executed in some thread.

Thread dumps:
Crucial to understand whats going on in the JVM, it provides details about which line of code is getting executed.
By reading a thread dump we will almost get a fair idea on what is happening in the process
Command used to generate the thread dump:
```
Kill -3 <PID>
Jstack <pip>
```

**Important Thead and its state:**

| State          | Description                                                                                   |
|----------------|----------------------------------------------------------------------------------------------|
| Runnable       | A thread executing in the Java Virtual Machine is in this state.                             |
| Blocked        | A thread that is blocked waiting for a monitor lock is in this state.                        |
| Waiting        | A thread that is waiting indefinitely for another thread to perform a particular action is in this state. |
| Timed_Waiting  | A thread is waiting to perform an action for up to a specified waiting time in this state.   |


Use this tool for quick thread analysis https://jstack.review/#tda_1_dump

## Heap dump consist of
1. All objects, classes, fileds, primitive values and references
2. All classes classes loaders, name super static fields
3. garbage collection Roots: object defined to be reachable by the JVM
4. Thread stacks and local Variable: The call stacks of the threads at the moment of teh snapshot and per frame information about local objects(thread dump)

**How to capture heapdump using jmap:**
```
jmap -dump:format=b,file=filename.hprof <pid>
```

Force option(if pid does not respond)
eclipse mat

**Java FLight recorder**

Using the command line add the following flag to your Java application
java -xx:+unlockCommercialFeatures -XX:+FlightRecorder -
XX:StartFlightRecording=duration=60s,filename=myrecording.jfr


using Diagnostic command
```
jcmd <process Id> Jfr.start duration=60s filename=myrecording.jfr
```

Java Mission console(JMC)
Compile(javac):
helloworld.java > helloWorld.class(bytecode)
Decompile/Disassemble(javap):
javap <classname.class>
javap -c <classname.class>
javap -cp <classpath> <fully qualified classname>

Useful to check if a class is a part of jar to verifying the signature of the method
```
facing java.lang.NoSuchmethodError
```
**How to setup gc.log**
In the ```/ect/hadoop/conf/yarn-evn.sh```
```
"$YARN_OPTS -XX:+PrintGCDetails -XX:+PrintGCDataStamps -Xloggc:/var/log/hadoop-yarn/gc-rm.log"
```