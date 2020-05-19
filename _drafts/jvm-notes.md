# OpenJDK 学习笔记

#### [JEP 304: Garbage Collector Interface](https://openjdk.java.net/jeps/304)


* The heap, a subclass of CollectedHeap
* The barrier set, a subclass of BarrierSet, which implements the various barriers for the runtime
* An implementation of CollectorPolicy
* An implementation of GCInterpreterSupport, which implements the various barriers for a GC for the interpreter* (using assembler instructions)
* An implementation of GCC1Support, which implements the various barriers for a GC for the C1 compiler
* An implementation of GCC2Support, which implements the various barriers for a GC for the C2 compiler
* Initialization of eventual GC specific arguments
* Setup of a MemoryService, the related memory pools, memory managers, etc.


#### [JEP 363: Remove the Concurrent Mark Sweep (CMS) Garbage Collector](https://openjdk.java.net/jeps/363)

[JEP 291: Deprecate the Concurrent Mark Sweep (CMS) Garbage Collector](https://openjdk.java.net/jeps/291) 弃用 CMS

#### [JEP 189: Shenandoah: A Low-Pause-Time Garbage Collector](https://openjdk.java.net/jeps/189)

wiki: [Shenandoah GC](https://wiki.openjdk.java.net/display/shenandoah/Main)

论文: [Shenandoah: An open-source concurrent compacting garbage collector for OpenJDK](https://www.researchgate.net/publication/306112816_Shenandoah_An_open-source_concurrent_compacting_garbage_collector_for_OpenJDK)

The phases above do roughly this:

1. Init Mark initiates the concurrent marking. It prepares the heap and application threads for concurrent mark, and then scans the root set. **This is the first pause in the cycle**, and the most dominant consumer is the root set scan. Therefore, its duration is dependent on the root set size.
2. Concurrent Marking walks over the heap, and traces reachable objects. This phase runs alongside the application, and its duration is dependent on the number of live objects and the structure of object graph in the heap. Since the application is free to allocate new data during this phase, the heap occupancy goes up during concurrent marking.
3. Final Mark finishes the concurrent marking by draining all pending marking/update queues and re-scanning the root set. It also initializes evacuation by figuring out the regions to be evacuated (collection set), pre-evacuating some roots, and generally prepares runtime for the next phase. Part of this work can be done concurrently during Concurrent precleaning phase. **This is the second pause in the cycle**, and the most dominant time consumers here are draining the queues and scanning the root set. 
4. Concurrent Cleanup reclaims immediate garbage regions – that is, the regions where no live objects are present, as detected after the concurrent mark.
5. Concurrent Evacuation copies the objects out of collection set to other regions. *This is the major difference against other OpenJDK GCs.* This phase is again running along with application, and so application is free to allocate. Its duration is dependent on the size of chosen collection set for the cycle.
6. Init Update Refs initializes the update references phase. It does almost nothing except making sure all GC and applications threads have finished evacuation, and then preparing GC for next phase. **This is the third pause in the cycle**, the shortest of them all.
7. Concurrent Update References walks over the heap, and updates the references to objects that were moved during concurrent evacuation. *This is the major difference against other OpenJDK GCs.* Its duration is dependent on number of objects in heap, but not the object graph structure, because it scans the heap linearly. This phase runs concurrently with the application.
8. Final Update Refs finishes the update references phase by re-updating the existing root set. It also recycles the regions from the collection set, because now heap does not have references to (stale) objects to them. **This is the last pause in the cycle**, and its duration is dependent on the size of root set.
9. Concurrent Cleanup reclaims the collection set regions, which now have no references to.

#### [JEP 333: ZGC: A Scalable Low-Latency Garbage Collector ](https://openjdk.java.net/jeps/333)

depends [JEP 312: Thread-Local Handshakes](https://openjdk.java.net/jeps/312)

[wiki](https://wiki.openjdk.java.net/display/zgc/Main)

**Goals**

1. GC pause times should not exceed 10ms
2. Handle heaps ranging from relatively small (a few hundreds of megabytes) to very large (many terabytes) in size
3. No more than 15% application throughput reduction compared to using G1
4. Lay a foundation for future GC features and optimizations leveraging colored pointers and load barriers
5. Initially supported platform: Linux/x64

At a glance, ZGC is a concurrent, single-generation, region-based, NUMA-aware, compacting collector. Stop-the-world phases are limited to root scanning, so GC pause times do not increase with the size of the heap or the live set.

**Limitations**

The initial experimental version of ZGC will not have support for class unloading. The ClassUnloading and ClassUnloadingWithConcurrentMark options will be disabled by default. Enabling them will have no effect.

Also, ZGC will initially not have support for JVMCI (i.e. Graal). An error message will be printed if the EnableJVMCI option is enabled.

These limitations will be addressed at a later stage in this project.

#### [JEP 318: Epsilon: A No-Op Garbage Collector](https://openjdk.java.net/jeps/318)

Develop a GC that handles memory allocation but does not implement any actual memory reclamation mechanism. Once the available Java heap is exhausted, the JVM will shut down.

**Goals**

Provide a completely passive GC implementation with a bounded allocation limit and the lowest latency overhead possible, at the expense of memory footprint and memory throughput. A successful implementation is an isolated code change, does not touch other GCs, and makes minimal changes in the rest of JVM.

#### [JEP 366: Deprecate the ParallelScavenge + SerialOld GC Combination](http://openjdk.java.net/jeps/366)

Deprecate the combination of the Parallel Scavenge and Serial Old garbage collection algorithms.

**Non-Goals**

It is not a goal to remove this GC combination.

It is not a goal to deprecate any other GC combinations.


This combination is unusual since it pairs the parallel young generation and serial old generation GC algorithms. We think this combination is only useful for deployments with a very large young generation and a very small old generation. 

The only advantage of this combination compared to using a parallel GC algorithm for both the young and old generations is slightly lower total memory usage. We believe that this small memory footprint advantage (at most ~3% of the Java heap size) is not enough to outweigh the costs of maintaining this GC combination.

The only way to select the parallel young and old generation GC algorithms without a deprecation warning will to specify only -XX:+UseParallelGC on the command line.

#### JVMCI

#### 其它

适用于 CLion 的  [CMakeLists.txt](https://github.com/ojdkbuild/ojdkbuild/blob/master/src/java-12-openjdk/CMakeLists.txt)