# JVM-GC-Handbook

---------------------
从GC roots标记可达对象：
https://www.cnblogs.com/kabi/p/6531375.html
执行方法的输入参数和局部变量
活动的线程
静态变量
JNI引用对象

---------------------
标记：找出所有活着的对象
stop the world：既不取决于堆的大小，也不取决于对象的总数大小，而是取决于活着对象的数量。
移除未使用对象：清除、压缩、复制。

---------------------
Java8默认使用的GC类型
Java8 默认使用：-XX:+UseParalleGC

Java9 默认使用：G1

---------------------
GC触发条件：GCLocker Initiated GC、Allocation Failure、Metadata GC Threshold

---------------------
打印GC信息使用到的参数：

-XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps

-----------------------
Serial GC：

串行垃圾收集器：在Young Generation使用mark-copy，在Old Generation使用mark-sweep-compact，并且都是单线程收集器；都会触发stop-the-world。  
只需要单个参数设置
  java -XX:+UseSerialGC com.mypackages.MyExecutableClass
  垃圾收集器使用：young 是 mark-copy；tenured是mark-sweep-compact
  会引发stop-the-world
  建议：几百兆和单个cpu的环境下使用
-----------------------
Minor GC 日志：
2015-05-26T14:45:37.987-02001:151.1262:[GC3(Allocation Failure4) 151.126: [DefNew5:629119K->69888K6(629120K)7, 0.0584157 secs]1619346K->1273247K8(2027264K)9,0.0585007 secs10][Times: user=0.06 sys=0.00, real=0.06 secs]11

1：GC事件启动的时间
2：GC事件相对于JVM启动的时间，以秒为测量单位
3：区分是Minor GC 或 Full GC的标志；这里表示使用的是Minor GC
4：触发GC时间的原因；这里表示在young中没有合适的内存空间保存新对象
5：垃圾收集器的名称；DefNew表示新生代使用Serial串行GC垃圾收集器
6：年轻代收集前后对象空间占有大小
7：年轻代的总大小
8：整个堆收集前和收集后的对象占用空间大小
9：整个堆的大小
10：以秒为单位的GC持续时间
11：-user：垃圾收集器线程占用的cpu时间
       -sys：花费在系统调用上的时间或等待系统事件的时间
       -real：应用暂定时间，这里是serial gc，是单线程的，所以real=user+sys
结论：老年代的大小=堆大小-年轻代占有大小； gc日志不直接展示老年代大小

2020-12-13T15:23:15.044-0800: 0.738: [GC (Allocation Failure) 2020-12-13T15:23:15.044-0800: 0.738: [DefNew: 59008K->5901K(66368K), 0.0107606 secs] 59008K->5901K(213824K), 0.0107985 secs] [Times: user=0.01 sys=0.01, real=0.01 secs] 

---------------------------
Full GC日志：
2015-05-26T14:45:59.690-02001: 172.8292:[GC (Allocation Failure) 172.829:[DefNew: 629120K->629120K(629120K), 0.0000372 secs3]172.829:[Tenured4: 1203359K->755802K 5(1398144K) 6,0.1855567 secs7] 1832479K->755802K8(2027264K)9,[Metaspace: 6741K->6741K(1056768K)]10 [Times: user=0.18 sys=0.00, real=0.18 secs]11

1：执行GC的时间戳
2：相对于JVM启动时间的秒数
3：与之前的Minor收集相似
4：Tenured：老年代的垃圾收集器，Tenured表示单线程、stop-the-world、mark-sweep-compact垃圾收集器被使用
5：垃圾收集器执行完成前后对象占用大小
6：老年代总容量
7：老年代收集所用时间
8：整合堆在新生代和老年代垃圾收集器执行前后对象占用大小
9：JVM堆总的可用大小
10：Metaspace大小
11：-user：CPU占用时间
       -sys：调用系统或等待系统事件消耗的时间
       -real：应用暂停时间

2020-12-13T15:23:15.418-0800: 1.112: [Full GC (Metadata GC Threshold) 2020-12-13T15:23:15.418-0800: 1.112: [Tenured: 3224K->5678K(147456K), 0.0209193 secs] 16062K->5678K(213824K), [Metaspace: 20577K->20577K(1067008K)], 0.0209826 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 

结论：Full GC会将年轻代、老年代和Metaspace都会收集

Parallel GC

并行垃圾收集器：在young generation使用mark-copy，在Old Generation使用mark-sweep-compact；且在Young Generation和Old Generation 都会stop-the-world；收集器都使用多线程进行标记-复制和标记-压缩。

-XX:ParallelGCThreads=0
          —指定并行垃圾收集器执行的线程数量；默认为所在机器的逻辑核数；当逻辑核数超过8时计算公式为：
         ParallelGCThreads = 8 + ((N - 8) * 5/8)：N为逻辑核数
获取mac的逻辑核数命令：sysctl hw.logicalcpu
https://www.cnblogs.com/dengq/p/13687534.html
linux查看物理CPU个数、核数、逻辑CPU个数
cat /proc/cpuinfo| grep "cpu cores”

主要目标：并行垃圾收集器主要解决的是吞吐量问题，延迟时间任然会存在问题，应为会stop-the-world。
--------------------------------
Minor GC

发生在Young Generation
2015-05-26T14:27:40.915-02001: 116.1152:[GC3(Allocation Failure4)[PSYoungGen5: 2694440K->1305132K6(2796544K)7]9556775K->8438926K8(11185152K)9, 0.2406675 secs10][Times: user=1.77 sys=0.01, real=0.24 secs]11

1：GC发生时间
2：GC发生时间（相对于JVM启动时间）
3：区分Minor GC 和 Full GC的标志
4：触发GC的原因；这里的Allocation Failure表示在Young Generation没有足够的空间对数据进行分配
5：垃圾收集器的名称；这里的PSYoungGen代表使用并行的mark-copy和stop-the-world收集器清理Young Generation
6：GC执行前后数据占用空间大小
7：总的Young Generation空间划分大小
8：GC执行前后数据占用空间大小
9：堆的总大小
10：GC执行持续的时间（单位为s）
11：
       -user：GC占用CPU时间
       -sys：GC花费在系统调用或等待系统事件的时间
       -real：停止应用的时间；这个时间应该接近于（user + sys）/ 垃圾收集使用的线程数
                   -有些活动不是并行化的，real时间会比按公式计算的值大

---------------
Full GC

2015-05-26T14:27:41.155-02001:116.3562:[Full GC3 (Ergonomics4)[PSYoungGen: 1305132K->0K(2796544K)]5[ParOldGen6:7133794K->6597672K 7(8388608K)8] 8438926K->6597672K9(11185152K)10, [Metaspace: 6745K->6745K(1056768K)] 11, 0.9158801 secs12, [Times: user=4.49 sys=0.64, real=0.92 secs]13

1：GC时间戳
2：GC执行时间，以秒为单位，相对于距离JVM启动的时间
3：GC类型标志：Full GC 表示Young Generation 和 Old Generation都以Full GC清理
4：GC触发原因：ergonomics表示JVM内部自动决定现在是最佳的GC时机
5：年轻代收集器类型：PSYoungGen表示并行的mark-copy且stop-the-world收集器用于收集Young Generation，通常年轻代数据的占有量会减少到0K
6：老年代收集器类型：ParOldGen表示并行的mark-sweep-compact且stop-the-world垃圾收集器被使用在老年代
7：GC执行前后老年代数据占用空间的大小
8：老年代总的空间大小（数据占用的空间+数据未占用的空间）
9：GC执行前后堆的数据占用空间大小
10：堆总的空间大小
11：Metaspace收集：大多数情况下不会进行收集，因为空间足够大
12：GC花费的时间
13：-user：GC线程占用CPU的时间
        -sys：GC调用系统的时间或等待系统事件的时间
        -real：应用挂起的时间；应用挂起的时间应接近于（user + sys）/ 线程数
                   -因为有些活动是非并行化的，所以实际时间会超出逻辑时间
Concurrent Mark and Sweep

优点：大量的工作听过并发线程处理，不需要stop-the-world
缺点：老年代碎片多且在某些情况下暂停时间不可预测，特别是大型堆栈。
启用参数：
  -XX:+UseConcMarkSweepGC
主要目的是：
      减少GC时间，因为要抢占多大多数的cpu时间或全部cpu时间，所以应用的吞吐量相对于在Parallel GC会小。

并发标记清除算法：在Young Generation使用的是并行mark-copy且stop-the-world算法；在Old Generation使用的是大多数的并发mark-sweep算法。为了防止长时间的暂停在Old Generation：第一、不压缩而是使用free-list管理空闲空间；第二、并发执行mark-sweep，不会stop-the-world，但也是会与应用竞争CPU时间；
默认执行并发垃圾收集的线程数为 （ 并行垃圾收集线程数 + 3 ）/ 4

并发线程数量
-XX:+ConcGCThreads=

启用并发标记清除算法
-XX:+UseConcMarkSweepGC
查看线程的核数命令：
java -XX:+PrintFlagsFinal -version | grep ParallelGCThreads

2015-05-26T16:23:07.219-02001: 64.3222:[GC3(Allocation Failure4) 64.322: [ParNew5: 613404K->68068K6(613440K) 7, 0.1020465 secs8] 10885349K->10880154K 9(12514816K)10, 0.1021309 secs11][Times: user=0.78 sys=0.01, real=0.11 secs]12

1：GC触发时间戳
2：GC触发时间，相对于JVM启动的时间，单位是秒
3：GC区分标志，区分是Minor GC和Full GC，这里是Minor GC
4：GC触发原因，这里是Young Generation空间不足
5：区分使用的收集器，ParNew表示在Young Generation使用的是mark-copy且stop-the-world的收集器，与Old Generation中的Concurrent Mark & Sweep 收集器一同工作
6：Young Generation在GC前后的对象占用空间大小
7：Young Generation总的空间大小
8：没有最终清理垃圾对象的GC持续时间
9：堆收集前后对象数据占用的空间大小
10：堆的总的空间大小
11：在Young Generation执行mark-copy收集器的持续时间，单位为秒，包括与ConcurrentMarkSweep通信的时间、足够老的对象提升到老年代的时间、一些最终清理的时间。
12：-user：GC占用CPU时间
        -sys：GC系统调用的时间或等待系统事件的时间
        -real：应用挂起时间，实际值会超出（user time + sys time）/垃圾收集使用到的线程数，会超出是因为有些操作是非并行化的。

Full GC

2015-05-26T16:23:07.321-0200: 64.425: [GC (CMS Initial Mark) [1 CMS-initial-mark: 10812086K(11901376K)] 10887844K(12514816K), 0.0001997 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
2015-05-26T16:23:07.321-0200: 64.425: [CMS-concurrent-mark-start]
2015-05-26T16:23:07.357-0200: 64.460: [CMS-concurrent-mark: 0.035/0.035 secs] [Times: user=0.07 sys=0.00, real=0.03 secs]
2015-05-26T16:23:07.357-0200: 64.460: [CMS-concurrent-preclean-start]
2015-05-26T16:23:07.373-0200: 64.476: [CMS-concurrent-preclean: 0.016/0.016 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]
2015-05-26T16:23:07.373-0200: 64.476: [CMS-concurrent-abortable-preclean-start]
2015-05-26T16:23:08.446-0200: 65.550: [CMS-concurrent-abortable-preclean: 0.167/1.074 secs] [Times: user=0.20 sys=0.00, real=1.07 secs]
2015-05-26T16:23:08.447-0200: 65.550: [GC (CMS Final Remark) [YG occupancy: 387920 K (613440 K)]65.550: [Rescan (parallel) , 0.0085125 secs]65.559: [weak refs processing, 0.0000243 secs]65.559: [class unloading, 0.0013120 secs]65.560: [scrub symbol table, 0.0008345 secs]65.561: [scrub string table, 0.0001759 secs][1 CMS-remark: 10812086K(11901376K)] 11200006K(12514816K), 0.0110730 secs] [Times: user=0.06 sys=0.00, real=0.01 secs]
2015-05-26T16:23:08.458-0200: 65.561: [CMS-concurrent-sweep-start]
2015-05-26T16:23:08.485-0200: 65.588: [CMS-concurrent-sweep: 0.027/0.027 secs] [Times: user=0.03 sys=0.00, real=0.03 secs]
2015-05-26T16:23:08.485-0200: 65.589: [CMS-concurrent-reset-start]
2015-05-26T16:23:08.497-0200: 65.601: [CMS-concurrent-reset: 0.012/0.012 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]
注意：年轻代Minor GC 可以与老年代并发标记清除收集器一同工作
CMS的Full GC包括7个阶段，1-5是标记阶段，会有两次stop-the-world（Initial Mark 和 Final Remark）
Phase 1: Initial Mark
初始标记：会stop-the-world，因为要标记Old Generation当中GC Roots和对年轻代对象的引用对象；GC Root直接引用的对象不多，所以很快。这是CMS期间的两大停摆事件之一。 此阶段的目标是标记Old Generation中的所有对象，这些对象要么是直接的GC ROOT，要么是从年轻一代中的某些活动对象引用的。 后者很重要，因为Old Generation是单独收集的。

2015-05-26T16:23:07.321-0200: 64.421: [GC (CMS Initial Mark2[1 CMS-initial-mark: 10812086K3(11901376K)4] 10887844K5(12514816K)6, 0.0001997 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]7

1：GC触发的时钟时间和相对于JVM启动的时间
2：区分GC所处阶段，CMS Initial Mark表示初始标记，标记所有Old Generation中的GC Roots
3：老年代对象数据占用的空间大小
4：老年代总的空间大小
5：堆已使用的空间大小
6：堆总的空间大小
7：初始标记的持续时间
        -user：占用CPU的时间长短
        -sys：花费在系统调用或等待系统事件的时间长短
        -real：应用挂起的时间长短

2020-12-14T09:13:05.211-0800: 1.425: [GC (CMS Initial Mark) [1 CMS-initial-mark: 4983K(118784K)] 12084K(210944K), 0.0006673 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

Phase 2: Concurrent Mark
并发标记：在第一阶段初始标记的基础上，从GC Roots遍历所有Old Generation的对象并且标记所有存活的对象，是并发标记，不会stop-the-world。不是所有在老年代中的存活的对象都会被标记，因为在GC在执行期间会改变引用，被改变对象引用区域叫做Card，对象被标记为dirty。由第一阶段标记过的对象出发，所有可达的对象都在本阶段标记。


2020-12-14T09:13:05.211-0800: 1.426: [CMS-concurrent-mark-start]
020-12-14T09:13:05.216-0800: 1.430: [CMS-concurrent-mark: 0.004/0.004 secs] [Times: user=0.03 sys=0.00, real=0.00 secs]

2015-05-26T16:23:07.321-0200: 64.425: [CMS-concurrent-mark-start]
2015-05-26T16:23:07.357-0200: 64.460: [CMS-concurrent-mark1: 035/0.035 secs2] [Times: user=0.07 sys=0.00, real=0.03 secs]3

1：CMS-concurrent-mark：收集器执行阶段，从GC Roots遍历所有老年代对象并标记存活对象（不是所有的存活对象都会别标记，因为没有stop-the-world，应用任然可以改变对象引用）
2：并发标记占用0.35的CPU时间，0.035s的挂钟时间（从事件从开始运行到结束，时钟走过的时间，这其中包含了进程在阻塞和等待状态的时间 （挂钟时间 ＝ 阻塞时间 ＋ 就绪时间 ＋运行时间））
3：Times在并发标记中基本上无意义，因为它是从CMS-concurrent-mark-start开始，并且包含有除并发标记以外的时间消耗。

Phase 3: Concurrent Preclean
https://blog.csdn.net/enemyisgodlike/article/details/106960687

不会stop-the-world，对card区域的dirty对象进行处理，并将从dirty对象可达的对象进行mark；除此之外还会做必要的对象清除和为Final Remark做准备。在本阶段，会查找前一阶段执行过程中,从新生代晋升或新分配或被更新的对象。通过并发地重新扫描这些对象，预清理阶段可以减少下一个stop-the-world 重新标记阶段的工作量。




2020-12-14T09:13:07.918-0800: 4.132: [CMS-concurrent-preclean-start]
2020-12-14T09:13:07.919-0800: 4.133: [CMS-concurrent-preclean: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2015-05-26T16:23:07.357-0200: 64.460: [CMS-concurrent-preclean-start]
2015-05-26T16:23:07.373-0200: 64.476: [CMS-concurrent-preclean1: 0.016/0.016 secs2] [Times: user=0.02 sys=0.00, real=0.02 secs]3
1：标识concurrent-preclean阶段
2：并发预清理占用0.016的CPU时间，0.016s的挂钟时间（从事件从开始运行到结束，时钟走过的时间，这其中包含了进程在阻塞和等待状态的时间 （挂钟时间 ＝ 阻塞时间 ＋ 就绪时间 ＋运行时间））
3：无意义，因为是从并发标记阶段开始计时的。

Phase 4: Concurrent Abortable Preclean.

-XX:CMSScheduleRemarkEdenSizeThreshold=2097152Bytes(2M)：当Eden

不会stop-the-world，尽可能与stop-the-world的Final Remark独立开，因为这个阶段需要重复地做一些事情直到达到某个条件停止，执行时间取决于多种因素（迭代次数、做的有效工作量的数量、挂钟时间等）；增加这一阶段是为了让我们可以控制这个阶段的结束时机，比如扫描多长时间（默认5秒）或者Eden区使用占比达到期望比例（默认50%）就结束本阶段。

2020-12-14T09:13:07.919-0800: 4.133: [CMS-concurrent-abortable-preclean-start]
2020-12-14T09:13:08.067-0800: 4.281: [CMS-concurrent-abortable-preclean: 0.148/0.148 secs] [Times: user=0.30 sys=0.02, real=0.15 secs]

2015-05-26T16:23:07.373-0200: 64.476: [CMS-concurrent-abortable-preclean-start]
2015-05-26T16:23:08.446-0200: 65.550: [CMS-concurrent-abortable-preclean1: 0.167/1.074 secs2] [Times: user=0.20 sys=0.00, real=1.07 secs]3
1：区分并发标记清除垃圾收集执行阶段，这里是concurrent-abortable-preclean阶段
2：并发可中止的预清理阶段，占用0.167的CPU时间，1.074s的挂钟时间（从事件从开始运行到结束，时钟走过的时间，这其中包含了进程在阻塞和等待状态的时间 （挂钟时间 ＝ 阻塞时间 ＋ 就绪时间 ＋运行时间））
3：无意义，因为是从并发标记阶段开始计时的。

Phase 5: Final Remark
第二个会stop-the-world的阶段，最后标记在老年代所有存活的对象；为了避免多个stop-the-world的收集器连续执行，CMS尽可能在Young Generation为0k的时候执行Final Remark；

2020-12-14T09:13:05.423-0800: 1.638: [GC (CMS Final Remark) [YG occupancy: 47292 K (92160 K)]2020-12-14T09:13:05.423-0800: 1.638: [Rescan (parallel) , 0.0098615 secs]2020-12-14T09:13:05.433-0800: 1.648: [weak refs processing, 0.0000618 secs]2020-12-14T09:13:05.433-0800: 1.648: [class unloading, 0.0023545 secs]2020-12-14T09:13:05.436-0800: 1.650: [scrub symbol table, 0.0024580 secs]2020-12-14T09:13:05.438-0800: 1.653: [scrub string table, 0.0003786 secs][1 CMS-remark: 4983K(118784K)] 52275K(210944K), 0.0155274 secs] [Times: user=0.10 sys=0.00, real=0.01 secs] 


2015-05-26T16:23:08.447-0200: 65.5501: [GC (CMS Final Remark2) [YG occupancy: 387920 K (613440 K)3]65.550: [Rescan (parallel) , 0.0085125 secs]465.559: [weak refs processing, 0.0000243 secs]65.5595: [class unloading, 0.0013120 secs]65.5606: [scrub string table, 0.0001759 secs7][1 CMS-remark: 10812086K(11901376K)8] 11200006K(12514816K) 9, 0.0110730 secs10] [[Times: user=0.06 sys=0.00, real=0.01 secs]11

1：GC事件触发时间
2：GC事件类型：CMS Final Remark 这里表示CMS的最终标记过程，会标记在Old Generation所有存活的对象（包括在前几个标记阶段创建或修改的对象引用）
3：YG occupancy：当前年轻代对象数据占用空间大小和总的空间大小
4：Rescan(parallel)：当应用挂起时，rescan并行地扫描所有的老年代中存活的对象，这里表明是并行执行并且花费了0.0085125秒
5：weak refs processing：最终标记阶段的子阶段是处理弱引用，花费的时间以及相对于JVM启动时间的时间戳
6：class unloading：卸载未使用的类，带有花费的时间和相对于JVM启动时间的时间戳
7：scrub symbol table：清理符号表以及其花费的时间
      -scrub string table：清理字符表（类级别的元数据和内部化字符串）及其花费的时间
8：CMS-remark：老年代存储对象占用空间大小和老年代当前的总共大小
9：这一GC阶段前后堆的对象数据占用空间大小以及堆的总空间大小
10：这一GC阶段的花费的时间
11： -user：cup占用时间   -sys：花费在系统调用或等待系统事件的时间

注意：到此，总共的CMS的五个标记阶段已完成

Phase 6: Concurrent Sweep
不会stop-the-world，这一阶段的主要目的是清除垃圾对象并回收垃圾对象占用的内存空间以备以后使用。


2015-05-26T16:23:08.458-0200: 65.561: [CMS-concurrent-sweep-start] 2015-05-26T16:23:08.485-0200: 65.588: [CMS-concurrent-sweep1: 0.027/0.027 secs2] [[Times: user=0.03 sys=0.00, real=0.03 secs] 3

1：CMS-concurrent-sweep：清理未标记的对象并回收内存空间
2：对应的cpu占用时间和这一阶段的挂钟时间
3：无意义，记录的是从CMS标记开始至今的时间，远超过这一阶段花费的时间。

Phase 7: Concurrent Reset
并发重置，重置CMS算法内部的数据结构，为下一CMS周期做准备
2020-12-14T09:13:08.096-0800: 4.310: [CMS-concurrent-reset-start]
2020-12-14T09:13:08.096-0800: 4.310: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

2015-05-26T16:23:08.485-0200: 65.589: [CMS-concurrent-reset-start] 2015-05-26T16:23:08.497-0200: 65.601: [CMS-concurrent-reset1: 0.012/0.012 secs2] [[Times: user=0.01 sys=0.00, real=0.01 secs]3

1：区分CMS阶段的标志
2：这一阶段占用CPU的时间和挂钟时间
3：无意义，记录的是从CMS标记开始至今的时间，远超过这一阶段花费的时间。


G1 – Garbage First

G1在 JDK7u4以上都可以使用，在JDK9开始，G1为默认的垃圾收集器，以替代CMS。

Garbage-First Garbage Collector
https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html

Collection Set：CSet是指一组可被回收的regions的集合。CSet涵盖的regions会在GC的过程中被异动到另一个regions。CSet中的regions可以来自Eden、Survivor、old。CSet会占用不到整个堆空间的1%

Remember Set：记录当前regions中的那些对象被其他regions进行了引用。价值在于垃圾收集器不需要扫描整个堆去确定哪些对象被其他区域引用了。比如我们只想收集Young regions，就可以通过RSet将本region有哪些对象被其他Old regions引用了，这样进行GC Roots可达性分析的时候就可以忽略了。

G1的主要目的是让stop-the-world可配置，可预测。G1是一个软实时垃圾收集器，可以设置指定但性能目标（比如指定stop-the-world在一定的毫秒范围内，不超过一定的毫秒时间），然后G1垃圾收集器会尽最大努力实现这一目标，不保证一定实现这一目标，如果那样的话就是硬实时。

G1将堆分割成多个较小的堆区域（通常是2048个），年轻代和老年代不是必须是连续的。每块区域可以使Eden region，可以使Survivor region，也可以是Old region。逻辑上所有的Eden region加上Survivor region是Young Generation，所有的Old region是Old Generation。


G1不用一次处理一大块儿堆，而是逐步处理(一次只需考虑集合集（区域的子集）)；每次暂停都会收集Young regions，还包含一些Old regions。

G1在并发阶段会预估每个区域存活数据量，用于构建collection set，首先收集垃圾对象最多的区域，因此得名G1。

启用G1的参数是
-XX:+UseG1GC

Evacuation Pause: Fully Young

在应用程序开始阶段，G1没有来自尚未执行的并发阶段的信息，因此以完全年轻的模式运行。当年轻代填满时，应用程序将停止并将Young regions的存活数据复制到Survivor regions，或全部free regions成为Survivor regions。

复制的过程叫做Evacuation（疏散）；疏散暂停的完整日志非常多。
2020-12-14T16:34:08.044-0800: 0.717: [GC pause (G1 Evacuation Pause) (young), 0.0037985 secs]
   [Parallel Time: 2.5 ms, GC Workers: 10]
      [GC Worker Start (ms): Min: 716.8, Avg: 716.9, Max: 717.0, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 0.1, Avg: 0.2, Max: 1.0, Diff: 0.9, Sum: 2.2]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.2, Diff: 0.2, Sum: 0.2]
         [Processed Buffers: Min: 0, Avg: 0.1, Max: 1, Diff: 1, Sum: 1]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.3, Diff: 0.3, Sum: 0.5]
      [Object Copy (ms): Min: 1.4, Avg: 2.0, Max: 2.2, Diff: 0.8, Sum: 20.2]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
         [Termination Attempts: Min: 1, Avg: 17.3, Max: 25, Diff: 24, Sum: 173]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
      [GC Worker Total (ms): Min: 2.3, Avg: 2.3, Max: 2.4, Diff: 0.1, Sum: 23.3]
      [GC Worker End (ms): Min: 719.2, Avg: 719.2, Max: 719.2, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms]
   [Other: 1.2 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 1.0 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 34.0M(34.0M)->0.0B(124.0M) Survivors: 3072.0K->5120.0K Heap: 37.7M(216.0M)->7776.5K(216.0M)]
 [Times: user=0.03 sys=0.00, real=0.00 secs]

0.134: [GC pause (G1 Evacuation Pause) (young), 0.0144119 secs]1
    [Parallel Time: 13.9 ms, GC Workers: 8]2
        …3
    [Code Root Fixup: 0.0 ms]4
    [Code Root Purge: 0.0 ms]5
    [Clear CT: 0.1 ms]
    [Other: 0.4 ms]6
        …7
    [Eden: 24.0M(24.0M)->0.0B(13.0M) 8Survivors: 0.0B->3072.0K 9Heap: 24.0M(256.0M)->21.9M(256.0M)]10
    [Times: user=0.04 sys=0.04, real=0.02 secs] 11

1：G1暂停清理Young regions，相对于JVM启动时间该事件的启动时间0.134s，挂钟时间为0.0144119s
2：下列活动消耗的时间这里是13.9ms，垃圾收集的线程个数为8个
3：
4：释放用于管理并行活动的数据结构，一般总是接近于0，这里是顺序完成的。
5：清理更多的数据结构，顺序执行。
6：其他各种活动，其中许多活动也是并行的。
7：
8：疏散暂停前后Eden区域数据空间占有量。
9：疏散暂停前后Survivor regions数据占用空间的大小
10：疏散暂停前后堆内对象数据的占用空间大小，整个堆的大小
11：-user：GC线程占用的CPU时间
       -sys：花在系统调用和等待系统事件的时间
       -real：应用挂起的时间；理想情况下接近于  （user time + system time）/  the number of GC threads
             - 一些活动是非并行化的，往往实际值大于逻辑计算的值

大部分工作是由多个专用的GC工作线程完成的。

[Parallel Time: 13.9 ms, GC Workers: 8]1
    [GC Worker Start (ms)2: Min: 134.0, Avg: 134.1, Max: 134.1, Diff: 0.1]
    [Ext Root Scanning (ms)3: Min: 0.1, Avg: 0.2, Max: 0.3, Diff: 0.2, Sum: 1.2]
    [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
        [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
    [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
    [Code Root Scanning (ms)4: Min: 0.0, Avg: 0.0, Max: 0.2, Diff: 0.2, Sum: 0.2]
    [Object Copy (ms)5: Min: 10.8, Avg: 12.1, Max: 12.6, Diff: 1.9, Sum: 96.5]
    [Termination (ms)6: Min: 0.8, Avg: 1.5, Max: 2.8, Diff: 1.9, Sum: 12.2]
        [Termination Attempts7: Min: 173, Avg: 293.2, Max: 362, Diff: 189, Sum: 2346]
    [GC Worker Other (ms)8: Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
    GC Worker Total (ms)9: Min: 13.7, Avg: 13.8, Max: 13.8, Diff: 0.1, Sum: 110.2]
    [GC Worker End (ms)10: Min: 147.8, Avg: 147.8, Max: 147.8, Diff: 0.0]


1：表示下列活动使用并行线程挂钟时间为13.9ms，GC工作线程数量为8个。
2：GC Worker Start （ms）：GC工作线程的开始时间（JVM启动到GC活动开始的时间差），这里包括最小值，平均值，最大值和差异值（差异值=最大值-最小值，差异值很大说明使用过多的线程数量被使用或存在进程在窃取垃圾收集器的CPU时间）
3：Ext Root Scanning (ms)：扫描外部（非堆）根对象（例如：classloaders、JNI references、JVM system roots等）；显示的是运行时间，“Sum”是CPU time
4：Code Root Scanning （ms）：扫描来自实际代码的根对象（例如局部变量），显示运行时间（最小值、平均值、差异值、CPU时间）；
5：Object Copy （ms）：疏散暂停期间，将被收集区域将存活对象复制出去所花费的时间（最小值、平均值、最大值、差异值、CPU时间）
6：Termination（ms）：当一个垃圾收集线程完成任务时，会进入终止例行程序，与其他工作线程同步并尝试窃取其他工作线程未完成的任务；时间表示第一个进入终止例行程序直到实际终止花费的时间。min表示什么时候进入终止例行程序，max表示什么时候真正的终止。
7：Termination Attempts：如果工作线程成功窃取了任务，那么终止尝试会重新进入终止例行程序尝试终止或窃取任务，每次进入终止尝试窃取任务或终止工作线程的例行程序，终止尝试的次数都会增长。
8：GC Worker Other （ms）：工作线程完成其他任务花费的时间（不包括上面的）；
9：GC Worker Total（ms）：工作线程总共运行的时间
10：GC Worker End（ms）：工作线程结束的时间戳，通常会大致相等，否则说明线程数量过多或在工作线程执行的过程中有干扰

疏散暂停过程中的一些杂项任务：
[Other: 0.4 ms]1
    [Choose CSet5: 0.0 ms]
    [Ref Proc: 0.2 ms]2
    [Ref Enq: 0.0 ms]3
    [Redirty Cards: 0.1 ms 6]
    [Humongous Register: 0.0 ms 7]
    [Humongous Reclaim: 0.0 ms 8]
    [Free CSet: 0.0 ms]4

1：Other：表示疏散暂停过程中的一些杂项任务，大多数也是并行的
2：Ref Proc：全称是reference process；清理非强引用（弱、软、虚，final，JNI引用），将这些引用排列到reference队列中
3：Ref Enq：全称是references enqueue；将剩余的非强引用排列到ReferenceQueue当中花费的时间
4：Free CSet：返回CSet中释放的空间所花费的时间，其中也会清理CSet中的RSet

5：Choose CSet：选择CSet，因为年轻代的所有区域都被收集，所以CSet不需要选择，消耗时间都是0ms；Choose CSet任务一般都是在mixed gc的过程中触发
6:重新脏化卡片，排队引用可能更新RSet，所有需要对关联的Card重新脏化（Redirty Cards）
7、8：Humongous Register、Reclaim：主要是对巨型对象回收的信息，Young GC阶段会对RSet中的被引用的短命巨型对象进行回收，巨型对象会直接回收不需要复制转移（因为转移的代价非常大）

Concurrent Marking

G1并发标记默认在堆的占用量达到整个堆的45%的时候触发，设置参数为-XX:InitiatingHeapOccupancyPercent=45

G1并发标记与CMS中的并发标记类似，G1并发标记由多个阶段组成，有些标记阶段可以完全并发，有些阶段需要stop-the-world

G1许多概念是建立在CMS支出上的，G1 Concurrent Marking 使用SATB（Snapshot-At-The-Begining）方式在并G1发标记周期的开始就进行标记所有的存活对象，即使标记的同时被标记的存活对象会转变成垃圾对象；为每个region建立存活对象统计信息，方便后面的过程快速选择CSet，这些信息用于Old Generation执行GC时使用，并发标记可完全并发执行！！！ （It can happen fully concurrently, if the marking determines that a region contains only garbage, or during a stop-the-world evacuation pause for Old regions that contain both garbage and live objects.）

Phase 1:Initial-mark
阶段一（初始标记）：此阶段将从GC roots直接可达的所有对象进行了标记，在CMS中需要独立的stop-the-world，在G1初始标记阶段依赖的是Evacuation Pause（疏散暂停），所以开销很小，疏散暂停期间如果执行了并发标记的初始标记阶段会在日志打印的第一行加上"（initial-mark）”的内容。
1.631: [GC pause (G1 Evacuation Pause) (young) (initial-mark), 0.0062656 secs]
Phase 2:Root Region Scan
阶段二（根区域扫描）：这一阶段标记从Root regions可达的存活对象，所谓Root regions是那些非空的并且在并发标记过程中间需要回收的区域。因为并发标记的过程中操作对象会造成麻烦，根区域扫描必须在下一次Evacuation Pause（疏散暂停）开始之前完成。如果转疏散停阶段需要提前开始，会向根扫描过程发一个终止信号，然后等根扫描阶段结束开始疏散停止。根区域是Survivor regions：在Young Generation中在下一次疏散暂停需要回收的区域。
1.362: [GC concurrent-root-region-scan-start]
1.364: [GC concurrent-root-region-scan-end, 0.0028513 secs]
Phase 3：Concurrent Mark
阶段三（并发标记）：这一阶段与CMS的并发标记阶段类似，遍历对象图中的所有对象，并在指定的bitmap（位图）中标记访问过的对象，为了符合STAB，G1 GC 要求应用程序的所有并发更新对象图的线程保留之前的引用以便后续跟踪。

为了达到这种目的，G1使用Pre-Write机制。在G1的并发标记阶段处在活动状态时，每当某对象的字段引用发生改变，都会在log buffers中记录，并发标记线程负责处理。
1.364: [GC concurrent-mark-start]
1.645: [GC concurrent-mark-end, 0.2803470 secs]
Phase 4：Remark
阶段四（重新标记）：会stop-the-world，类似CMS也会stop-the-world完成标记过程。对于G1来说，这一阶段会短暂地停止应用程序线程来停止并发更新日志的流入，并处理少量的更新日志，并标记在并发标记活动初始化时仍处于活动状态的未标记对象。重新标记阶段还会做一些额外的清理，例如引用清理和类卸载。
1.645: [GC remark 1.645: [Finalize Marking, 0.0009461 secs] 1.646: [GC ref-proc, 0.0000417 secs] 1.646: [Unloading, 0.0011301 secs], 0.0074056 secs]
[Times: user=0.01 sys=0.00, real=0.01 secs]
Phase 5：Cleanup
阶段5（清除）：这是最后一个为接下来的疏散做准备的阶段，计算堆中所有的存活对象并根据期望的GC效率队这些regions进行排序。为下一次并发标价做整理工作，维护并发标记的内部状态。

最后，在清除这一阶段回收全部的不包含存活对象的regions。这个阶段一些部分是并发的（例如：空regions回收和大部分的存活对象计算），但也要短暂地stop-the-world让应用线程不会进行干扰。

1、stop-the-world Cleanup的日志格式如下：
1.652: [GC cleanup 1213M->1213M(1885M), 0.0030492 secs]
[Times: user=0.01 sys=0.00, real=0.00 secs]
2、如果发现heap regions只包含垃圾对象，那么Cleanup的日志如下：
1.872: [GC cleanup 1357M->173M(1996M), 0.0015664 secs]
[Times: user=0.01 sys=0.00, real=0.01 secs]
1.874: [GC concurrent-cleanup-start]
1.876: [GC concurrent-cleanup-end, 0.0014846 secs]


Evacuation Pause: Mixed

混合式的疏散暂停

在G1并发标记完成之后，G1将调度一个混合收集器，这个混合收集器不仅从年轻区域中取出垃圾对象，还将一堆老年区域放到CSet当中。混合疏散暂停阶段并不总是在并发标记阶段后执行（有许多规则和试探会影响到这一点，例如：如果能够释放掉Old regions的很大一部分区域，那么就没必要进行混合疏散暂停阶段），因此通常会在Concurrent Marking （并发标记阶段）结束和Evacuation Pause：Mixed（混合疏散暂停阶段）之前出现许多完全young的Evacuation Pause。

1、全young的疏散暂停log


2、混合疏散暂停log


要添加到CSet中Old regions的数量和添加他们的顺序也是基于一定的规则（其中包括应用指定的软实时性能目标时间，并发标记期间收集标记存活对象的数量和收集垃圾对象的效率以及其他许多JVM配置参数）；混合疏散暂停收集的过程与fully-young疏散暂停收集过程大致相同。

Remembered sets允许不同的regions独立收集。例如，在收集regions A、B、C的时候，我们必须知道region D、E是否对region A、B、C有引用，以确定region A、B、C存活对象数据暂用空间的大小。但是遍历整合堆图需要花费大量的时间，并且会阻碍增量收集。就像其他GC算法收集Young Generation需要Card-Table，G1就需要Remember sets（RSets）。


如上图所示，每个region都需要一个RSet列出其他region指向当前region的引用对象。这些引用对象被视为当前region额外的GC ROOTS。

请注意：在Concurrent Marking阶段，Old regions中被确定为垃圾的region将被忽略，被Old regions中的垃圾region引用的对象不会被视为GC ROOTS，仍会被收集。

接下来的过程和其他垃圾收集器相同：确定哪些是活对象、哪些是垃圾对象

接下来活对象被移动到Survivor regions，并在必要的时候创建新对象，现在空的regions被释放空间可以再次用于存储新对象。（free是因为是copy过程，copy结束后还会存在之前的对象，所以要free（释放））


为了维护RSets，在应用运行期间，每当对字段进行写操作的时候就会出现Post-Write Barrier（写后屏障）。如果写字段的操作产生的新对象引用是跨region的（即：从一个region指向另一个region），则目标region（也就是被指向的region）会在当前region中的RSet中添加对应的记录。为了减少写屏障的引入带来的开销，将card放入RSet的过程是异步的并且做了许多优化。简单来说，Write Barrier（写屏障）将dirty-card放到local buffer（本地缓冲区），由专门的GC线程将dirty-card挑选出并将信息传播到被引用region的RSet中。

与fully-young Evacuation Pause的log相对比，mixed 有些差别。
[Update RS (ms)1: Min: 0.7, Avg: 0.8, Max: 0.9, Diff: 0.2, Sum: 6.1]
[Processed Buffers2: Min: 0, Avg: 2.2, Max: 5, Diff: 5, Sum: 18]
[Scan RS (ms)3: Min: 0.0, Avg: 0.1, Max: 0.2, Diff: 0.2, Sum: 0.8]
[Clear CT: 0.2 ms]4
[Redirty Cards: 0.1 ms]5

1：Update RS （ms）：因为RSets的处理是并发的，要确保local-buffer中的dirty-cards在region的垃圾收集之前将dirty-card信息传播到region的RSet当中。如果dirty-cards数量很多，那么并发GC线程无法处理任务。例如：可能因为输入修改的字段过多或CPU资源不足。
2：Prcessed Buffers：每个线程处理了多少缓冲区
3：Scan RS （ms）：扫描RSets中引用对象所花费的时间
4：Clear CT：这时清理card-table中的cards，清理只移除dirty状态的card，dirty表示字段已更新，信息已经被RSets记录。
5：Redirty Cards：这时将card-table中适当位置标记为dirty。Appropriate locations are defined by the mutations to the heap that GC does itself, e.g. while enqueuing references.

结论：由于存在写屏障和更活跃的后台线程，G1的吞吐量开销非常大，因此应用程序受到吞吐量限制或应用程序消耗100%的CPU，并且不太关心单个暂停的持续时间，那么CMS或Parallel可能是佳的选择。


GC Tuning: Basics
GC调优过程：
      1、列出性能目标
      2、运行测试代码
      3、测量结果
      4、将结果与性能目标相对比
      5、如果没达到性能目标，则重新修改执行测试。

性能目标分类：
      Latency、Throughput、Capacity（基础设施成本限制：例如设备资源都是有限的，不可能任意添加）

【JVM】吞吐量与延迟关系：堆内存增大，gc一次能处理的垃圾对象增大，吞吐量增大，但gc一次的时间会延长，导致后面排队的线程等待时间延长。内存堆减小，gc一次时间缩短，排队等待的线程等待时间缩短，延迟减小，吞吐量减小。
https://www.cnblogs.com/itplay/p/11381986.html
https://www.cnblogs.com/itplay/

GC性能调优目标限制：确保GC暂停持续时间不超过期望阈值、确保应用程序挂起时间不超过期望阈值、在满足延迟和吞吐量在合理范围内的情况下减少基础设施的开销。

性能调优对比：
-Xmx120M -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps 

开始：2020-12-15T17:23:29.706-0800: 0.213: [GC (Allocation Failure) 2020-12-15T17:23:29.706-0800: 0.213: [ParNew: 32661K->4096K(36864K), 0.0090053 secs] 32661K->26499K(118784K), 0.0090600 secs] [Times: user=0.04 sys=0.01, real=0.01 secs] 

结束：2020-12-15T17:33:28.123-0800: 598.627: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 


GC Tuning: In Practice
JVM 优化经验总结
https://developer.ibm.com/zh/articles/j-lo-jvm-optimize-experience/
High Allocation Rate（分配速率，单位是MB/sec）

过高的分配速率可能意味着应用程序的性能出现了问题。当在JVM上运行时，过高的分配速率会带来很大的开销，从而暴露问题。

如何测量分配速率？
测量分配速率指定XX:+PrintGCDetails -XX:+PrintGCTimeStamps 两个参数打印JVM的log日志。
0.291: [GC (Allocation Failure) [PSYoungGen: 33280K->5088K(38400K)] 33280K->24360K(125952K), 0.0365286 secs] [Times: user=0.11 sys=0.02, real=0.04 secs] 
0.446: [GC (Allocation Failure) [PSYoungGen: 38368K->5120K(71680K)] 57640K->46240K(159232K), 0.0456796 secs] [Times: user=0.15 sys=0.02, real=0.04 secs] 
0.829: [GC (Allocation Failure) [PSYoungGen: 71680K->5120K(71680K)] 112800K->81912K(159232K), 0.0861795 secs] [Times: user=0.23 sys=0.03, real=0.09 secs]
从上面的GC日志中，我们可以将计算分配速率：
            分配速率 = 上一次GC过Young Generation之后Young Generation剩余的对象数据占用空间大小 - 下一次收集开始前Young Generation中对象数据占用空间的大小

使用上面的log信息我们可以得出以下结论：
JVM启动291ms后，创建了33280K对象，第一个Minor GC事件清理了Young Generation，Minor GC过后Young Generation还剩下5088k的对象
JVM启动446ms后，Young Generation的对象数据占用空间已经增长到38368k大小，触发了下一个Minor GC，Young Generation在Minor GC后剩下5120k的对象
JVM启动829ms后，Young Generation的对象数据占用空间已经增长到71680k大小，触发了下一个Minor GC，Young Generation在Minor GC后剩下5120k的对象
计算得：

根据上表所示，我们得出结论，在测量期间软件的分配速率为161MB/sec

为什么要关心分配速率？
在测量完分配率之后，我们可以增加或减少GC的频率来调整GC的吞吐量。可以注意到只有Young Generation中的Minor GC会受到影响，GC暂停清理老年代的频率和持续时间都不直接受到分配率的影响，而是受到提升率的影响。

由于新对象是在Eden中进行的，我们可以直接查看Eden大小对分配率的影响。因此我们可以假设增大Eden的大小将减小较小GC暂停的频率，从而允许应用程序维持更快的分配率。假设我们增大Eden空间将减少Minor GC暂停的频率，从而允许应用程序维持更快的分配率。

通过-XX:NewSize -XX:MaxNewSize & -XX:SurvivorRatio 参数，我们可以分别指定Young Generation的初始大小、Young Generation最大大小，以及Survivor初始大小。
  -jdk8中默认的初始化SurvivorRatio=8，表示的是Eden占8/10，Young Generation中的Survivor中的from和to都是1/10。
      -最小SurvivorRatio设置的是3，
      -TargetSurvivorRatio默认是50，期望Survivor中存有存活对象的大小，是Survivor整体存活对象提升到Old Generation
事实却是如此
100M大小的Eden降低分配速率到100MB/sec
1GB大小的Eden增大分配速率到200MB/sec
Young Generation中Minor GC的频率降低，那么stop-the-world的频率就会降低，应用程序就会做更多的有效工作，应用程序的有效工作可能会创建更多的对象在Eden中，从而支持更高的分配速率。

分配速率会影响Minor GC stop-the-world的频率大小，要看整体的作用还要在应用程序正常操作时计算Major GC的频次和吞吐量（这就不是MB/sec）。

关于JVM参数的理解
https://blog.csdn.net/flyfhj/article/details/86630105
http://www.360doc.com/content/11/1010/15/7656248_154913380.shtml
https://docs.oracle.com/cd/E19159-01/819-3681/abeil/index.html
https://www.cnblogs.com/hellxz/p/10841550.html

Give me an Example
https://github.com/gvsmirnov/java-perv/blob/master/labs-8/src/main/java/ru/gvsmirnov/perv/labs/gc/Boxing.java
假设这个应用程序与外部传感器一起工作，应用程序在一个专用线程中不断地传感器的值（在例子中为随机值）：
public class BoxingFailure {
  private static volatile Double sensorValue;

  private static void readSensor() {
    while(true) sensorValue = Math.random();
  }

  private static void processSensorValue(Double value) {
    if(value != null) {
      //...
    }
  }
}
正如类的名字所示，这里的问题是装箱，可能为了进行null检查，将sensorValue字段设置为Double，这个例子是一个常见模式，基于最近的值进行处理操作。在现实的应用程序中，获取这个值要比获取一个随机值的开销大。这个例子中一个线程不断地生成新值，另一个是计算线程负责使用生成的随机值，避免了大的开销。

上面的应用程序会出现GC跟不上分配速率的问题。下面会进行讨论解决。

我们的JVMs会受到影响吗？
首先，我们应该只担心应用程序的吞吐量是否开始下降。由于应用程序创建的对象过多，几乎立即就会被Minor GC，因此Minor GC stop-the-world的频率就会增大，这在足够多的对象数据创建的情况下，Minor GC频率过大会对应用程序吞吐量产生严重的影响。

当你遇到这样的问题时，你将会得到一个日志文件（应用程序指定JVM参数为-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xmx32m）：
2.808: [GC (Allocation Failure) [PSYoungGen: 9760K->32K(10240K)], 0.0003076 secs]
2.819: [GC (Allocation Failure) [PSYoungGen: 9760K->32K(10240K)], 0.0003079 secs]
2.830: [GC (Allocation Failure) [PSYoungGen: 9760K->32K(10240K)], 0.0002968 secs]
2.842: [GC (Allocation Failure) [PSYoungGen: 9760K->32K(10240K)], 0.0003374 secs]
2.853: [GC (Allocation Failure) [PSYoungGen: 9760K->32K(10240K)], 0.0004672 secs]
2.864: [GC (Allocation Failure) [PSYoungGen: 9760K->32K(10240K)], 0.0003371 secs]
2.875: [GC (Allocation Failure) [PSYoungGen: 9760K->32K(10240K)], 0.0003214 secs]
2.886: [GC (Allocation Failure) [PSYoungGen: 9760K->32K(10240K)], 0.0003374 secs]
2.896: [GC (Allocation Failure) [PSYoungGen: 9760K->32K(10240K)], 0.0003588 secs]
这里我们应该注意Minor GC的频率，根据GC log我们可以看出，Minor GC频率很大（说明应用程序正在创建大量的对象数据，Minor GC后Young Generation的占用率仍然很低，没有进行完整的Minor GC），这说明Minor GC堆程序的吞吐量有很大的影响。

有什么解决方案？

在某些情况下，降低高频次的Minor GC对应用程序吞吐量的影响就是增加Young Generation空间。这样做本身不会降低分配率，但是会导致Minor GC频率降低，当每次只有少数会进入Survivor时，增加Young Generation空间的方法会起到作用。因为Minor GC stop-the-world的持续时间受到存活对象数量的影响，这里不会明显延长Minor GC stop-the-world的持续时间。

当我们使用-Xmx64m参数运行相同的应用程序时，增加了heap大小和Young Generation的大小，
2.808: [GC (Allocation Failure) [PSYoungGen: 20512K->32K(20992K)], 0.0003748 secs]
2.831: [GC (Allocation Failure) [PSYoungGen: 20512K->32K(20992K)], 0.0004538 secs]
2.855: [GC (Allocation Failure) [PSYoungGen: 20512K->32K(20992K)], 0.0003355 secs]
2.879: [GC (Allocation Failure) [PSYoungGen: 20512K->32K(20992K)], 0.0005592 secs]

然而，仅仅向heap增加更多的内存并不总是可行的解决方案，通过前面我们可以分析出大部分对象是哪里产生的，在本实例中，99%是使用readSensor方法创建的Double对象。简单的优化方案是，对象可以用基本类型的double替换，null可以用基本类型Double.Nan（0.0d / 0.0 Not-a-Number）,因为原始类型的值不是对象，所以不会产生垃圾，也就没有什么需要收集的，对象当中基本类型的double字段会直接覆盖，而不是在堆上分配新对象。

在本示例应用程序中，简单的更改将完全消除GC暂停。在某些情况下，JVM会非常聪明，可以使用逃逸分析技术（escape analysis technique）自行删除过多的创建对象内存大小的分配。长话短说，JIT编译器可能证明创建的对象永远不会“逃逸”它创建的作用域。在本示例应用程序中，使用基本类型double的这种方式，不需要在heap上分配内存和产生垃圾，所以JIT Compiler只是做了：减少对象空间的分配。

什么是逃逸？
       逃逸是指某个方法之内创建的对象，出了在方法体内引用外，还被方法体之外的其他变量引用到，这样带来的后果是在该方法执行完毕之后，该方法创建的被方法外引用的对象无法被GC回收。正常情况下是方法执行完毕之后方法体内创建的对象由于无法回收就造成了对象逃逸。

是否开启逃逸分析指定参数为“-XX:+DoEscapeAnalysis”。默认值是true，开启逃逸分析。
https://blog.csdn.net/baichoufei90/article/details/85180478

如何做Benchmark（基准测试）
要学一下怎么对java代码进行基准测试
https://github.com/openjdk/jmh
https://zhuanlan.zhihu.com/p/59277333

Premature Promotion（过早的晋升）

在解释过早地晋升的概念之前，我们先熟悉下晋升率（promotion rate）：晋升率是在单位时间内将从Young Generation的对象数据传播到Old Generation对象的数量来衡量的，晋升率通常以（MB/sec）为单位。

将在Young Generation长期存活的对象晋升到Old Generation是我们期望的行为。我们假设一种情况，不仅长期存活的对象存在Old Generation，没有在Young Generation没有达到预期寿命的对象也晋升到了老年代当中（这种成为过早提升）。这时，清理短寿命的对象成了Minor GC的工作，Minor GC不适合频繁执行，这样会导致更长的暂停时间，这将极大地影响应用程序的吞吐量。

如何测量晋升率。
指定JVM启动参数：-XX:+PrintGCDetails -XX:+PrintGCTimeStamps，并打印log记录Minor GC的信息。如下所示：
0.291: [GC (Allocation Failure) [PSYoungGen: 33280K->5088K(38400K)] 33280K->24360K(125952K), 0.0365286 secs] [Times: user=0.11 sys=0.02, real=0.04 secs] 
0.446: [GC (Allocation Failure) [PSYoungGen: 38368K->5120K(71680K)] 57640K->46240K(159232K), 0.0456796 secs] [Times: user=0.15 sys=0.02, real=0.04 secs] 
0.829: [GC (Allocation Failure) [PSYoungGen: 71680K->5120K(71680K)] 112800K->81912K(159232K), 0.0861795 secs] [Times: user=0.23 sys=0.03, real=0.09 secs]
从log日志中我们可以提取的信息：Young Generation和heap在Minor GC前后的存活对象数据占用空间大小，这样我们就可以计算出Old Generation的大小，将日志中的信息表示为：

从上面可以看出，平均晋升率为92MB/sec，峰值为140.95MB/sec。（第一次 promoted = heap存活对象的大小 - young存活对象的大小；第N次为 promoted = 第N次的old generation存活大小 - 第N-1次的Old Generation存活对象的大小）

注意你只能从Minor GC中提取信息，Full GC不会暴露提升率，因为Full GC中还包含Major GC的内容。

我们应该关心什么？
分配率影响Minor GC的频率，晋升率影响Major GC的频率。从Young Generation提升到Old Generation的对象越多，填满Old Generation的速度越快，大的晋升率意味着Major GC的频率会增加。

Full GC需要更长的时间，因为需要与更多的对象交互，并执行额外的复杂活动（如碎片清理）。

例子：
https://github.com/gvsmirnov/java-perv/blob/master/labs-8/src/main/java/ru/gvsmirnov/perv/labs/gc/PrematurePromotion.java
        让我们看一个过早晋升的应用程序。这个应用程序获取数据块，积累数据块，达到足够的数量时，一次性整批处理。

public class PrematurePromotion {

   private static final Collection<byte[]> accumulatedChunks = new ArrayList<>();

   private static void onNewChunk(byte[] bytes) {
       accumulatedChunks.add(bytes);

       if(accumulatedChunks.size() > MAX_CHUNKS) {
           processBatch(accumulatedChunks);
           accumulatedChunks.clear();
       }
   }
}
运行上面所示的应用程序，JVM会受到怎样的影响呢？
过早提升会出现下面的特征：
在短时间内频繁地触发Full GC
Old Generation存活的对象数据占用空间一直保持较小的水平（通常低于Old Generation的10-20%）
让我们指定JVM参数-Xmx24m -XX:NewSize=16m -XX:MaxTenuringThreshold=1进行演示。
2.176: [Full GC (Ergonomics) [PSYoungGen: 9216K->0K(10752K)] [ParOldGen: 10020K->9042K(12288K)] 19236K->9042K(23040K), 0.0036840 secs]
2.394: [Full GC (Ergonomics) [PSYoungGen: 9216K->0K(10752K)] [ParOldGen: 9042K->8064K(12288K)] 18258K->8064K(23040K), 0.0032855 secs]
2.611: [Full GC (Ergonomics) [PSYoungGen: 9216K->0K(10752K)] [ParOldGen: 8064K->7085K(12288K)] 17280K->7085K(23040K), 0.0031675 secs]
2.817: [Full GC (Ergonomics) [PSYoungGen: 9216K->0K(10752K)] [ParOldGen: 7085K->6107K(12288K)] 16301K->6107K(23040K), 0.0030652 secs]
咋一看过早晋升不是问题所在，Old Generation中存活对象占用空间大小在每次Full GC的时候都在减少。但是会存在一种情况是，每次Full GC只提升了很少一部分对象或根本没有提升对象，这样的情况我们看Full GC log是看不出来的。

当Young Generation大量对象提升到Old Generation中时，一些存在Old Generation的对象被收集。表面上给人一种错误的感觉是Old Generation中的对象占用量在减少，而实际上，频繁大量的对象在不断提升，从而触发Full GC的频次也增大了。

解决方案是什么？
简而言之，我们需要增加Young Generation存放空间，从而使Young Generation能够存放更多的数据。第一种方法是调整JVM参数（-Xmx64m -XX:NewSize=32m）增大年轻代的空间大小，这就会大大降低Full GC的频次，同时几乎不会影响Minor GC持续的时间。

2.251: [GC (Allocation Failure) [PSYoungGen: 28672K->3872K(28672K)] 37126K->12358K(61440K), 0.0008543 secs]
2.776: [GC (Allocation Failure) [PSYoungGen: 28448K->4096K(28672K)] 36934K->16974K(61440K), 0.0033022 secs]

第二种方案是简单地减少批次大小，这也会得到类似的结果。选择正确的解决方案在很大程度上取决于应用程序中实际的发生情况。在某些逻辑不允许减少批次大小。增加可用内存或重新分配优化Young Generation。

如果两者都不是可行的选择，优化数据结构可以消耗更少的内存，本质上目标都是相同的：让填充到Young Generation中的数据大小合适。

Weak, Soft and Phantom References（软、弱、虚、引用对象）

Java内存回收机制--Java引用的种类（强引用、弱引用、软引用、虚引用）
https://blog.csdn.net/daijin888888/article/details/49949283
https://blog.csdn.net/u014086926/article/details/52106589
https://mrdear.cn/posts/java_reference.html
影响GC的另一类问题是非强引用（软弱虚引用对象）的使用有关，虽然在许多情况下可能有助于避免不必要的OutOfMemoryError，但是大量使用这样的引用会影响垃圾收集的行为方式，影响应用程序的性能。

我们应该关心什么？
在使用弱引用时，应该知道弱引用进行垃圾收集的方式。每当GC发现一个对象是弱可达对象时（也就是说对该对象的最后一个引用是弱引用），该弱可达对象就会放到相应的ReferenceQueue上成为符合结束条件的对象。然后轮询这个引用队列并执行相关的清理活动。 A typical example for such cleanup would be the removal of the now missing key from the cache.

解决方式是，你仍然可以创建新的强引用对原来的对象。因此在最终完成并回收该对象之前，GC必须再次检查是否真的可以这样做。这样非强引用就不会在额外的GC周期中回收。

弱引用实际上比想想的常见的多。许多缓存解决方案使用弱引用构建实现，所以即使你没有在代码中直接创建任何对象，应用程序当中仍然有很大的肯能存在大量的非强引用对象。

要记住：在使用软引用对象时，软引用的收集急切程度远低于弱引用，收集软引用的时间点并没有指定，而是取决于JVM的实现。通常，软引用的收集仅作为内存耗尽的最后一搏。这意味着存在软引用收集将面临着Full GC的频次更多、时间更长（因为在Old Generation存有更多的对象）。

虚引用对象：使用虚引用对象必须手动进行内存管理（将这些虚引用对象标记为可回收的引用对象）。如果不手动标记的话会造成很大的麻烦。

为了确保可回收对象保持原样，可能不会检索虚引用的引用对象，虚引用的get方法总是返回null；

大部分开发人员忽略了javadoc中的一段内容：
       与软引用和弱引用不同，虚引用进入队列时不会被垃圾收集器自动清理。通过虚应用可达的对象保持直到他们变得不可达或所有这样的引用被清除。

也就是说，我们必须手动调用清除虚引用的应用对象，否则就会面临OutOfMemoryError的风险。虚引用存在的原因首先是允许通过通常的方法知道一个对象什么时候变得实际不可达。不像软引用对象和虚引用对象你不能复活一个虚引用对象。

示例：
https://github.com/gvsmirnov/java-perv/blob/master/labs-8/src/main/java/ru/gvsmirnov/perv/labs/gc/WeakReferences.java
下面是一个示例，该应用程序分配了许多对象，这些对象在Minor GC执行时成功回收。修改租用阈值改变提升率，我们可以指定JVM启动参数-Xmx24m -XX:NewSize=16m -XX:MaxTenuringThreshold=1，GC log如下所示：
2.330: [GC (Allocation Failure)  20933K->8229K(22528K), 0.0033848 secs]
2.335: [GC (Allocation Failure)  20517K->7813K(22528K), 0.0022426 secs]
2.339: [GC (Allocation Failure)  20101K->7429K(22528K), 0.0010920 secs]
2.341: [GC (Allocation Failure)  19717K->9157K(22528K), 0.0056285 secs]
2.348: [GC (Allocation Failure)  21445K->8997K(22528K), 0.0041313 secs]
2.354: [GC (Allocation Failure)  21285K->8581K(22528K), 0.0033737 secs]
2.359: [GC (Allocation Failure)  20869K->8197K(22528K), 0.0023407 secs]
2.362: [GC (Allocation Failure)  20485K->7845K(22528K), 0.0011553 secs]
2.365: [GC (Allocation Failure)  20133K->9501K(22528K), 0.0060705 secs]
2.371: [Full GC (Ergonomics)  9501K->2987K(22528K), 0.0171452 secs]
在上面指定的JVM参数的基础上执行应用程序，Full GC频率较低。但是如果JVM加上参数(-Dweak.refs=true)开启创建这些创建对象的弱引用，情况可能发生很大的变化。开启创建对象的弱引用的原因有很多，从使用对象做弱HashMap的key开始到对象空间分配配置结束。

在任何情况下使用弱引用会导致以下问题：
2.059: [Full GC (Ergonomics)  20365K->19611K(22528K), 0.0654090 secs]
2.125: [Full GC (Ergonomics)  20365K->19711K(22528K), 0.0707499 secs]
2.196: [Full GC (Ergonomics)  20365K->19798K(22528K), 0.0717052 secs]
2.268: [Full GC (Ergonomics)  20365K->19873K(22528K), 0.0686290 secs]
2.337: [Full GC (Ergonomics)  20365K->19939K(22528K), 0.0702009 secs]
2.407: [Full GC (Ergonomics)  20365K->19995K(22528K), 0.0694095 secs]
正如上面的log所示，现在有许多Full GC，并且比没设置（-Dweak.refs=true）开启弱引用时的Full GC时间多了一个数量级。又是一个过早提升的例子，但是这次难度比较大，当然根本原因是使用到了开启了弱引用，在没有开启弱引用的时候，应用程序创建的对象在Young Generation 进行Minor GC的时候已经被收集，不会进入到Old Generation。但是开启弱引用之后，原本会被Minor GC收集的对象会经理多次Minor GC并提升到了Old Generation以便进行适当的清理。像以前一样一个简单的解决方案是通过指定JVM参数（-Xmx64m -XX:NewSize=32m）来增加年轻代的大小。

2.328: [GC (Allocation Failure)  38940K->13596K(61440K), 0.0012818 secs]
2.332: [GC (Allocation Failure)  38172K->14812K(61440K), 0.0060333 secs]
2.341: [GC (Allocation Failure)  39388K->13948K(61440K), 0.0029427 secs]
2.347: [GC (Allocation Failure)  38524K->15228K(61440K), 0.0101199 secs]
2.361: [GC (Allocation Failure)  39804K->14428K(61440K), 0.0040940 secs]
2.368: [GC (Allocation Failure)  39004K->13532K(61440K), 0.0012451 secs]
如上所示（-Xmx64m -XX:NewSize=32m）指定增大了Old Generation 的空间大小之后又开始在Minor GC进行了回收。

如果在下一个应用程序当中使用软引用，情况会更糟，直到应用程序面临OutOfMemoryError的时候，软引用可达对象才会被回收，在
下面的引用程序gc日志中可以看出，用软引用替换弱引用会立即出现更多的Full GC。
https://github.com/gvsmirnov/java-perv/blob/master/labs-8/src/main/java/ru/gvsmirnov/perv/labs/gc/SoftReferences.java
2.162: [Full GC (Ergonomics)  31561K->12865K(61440K), 0.0181392 secs]
2.184: [GC (Allocation Failure)  37441K->17585K(61440K), 0.0024479 secs]
2.189: [GC (Allocation Failure)  42161K->27033K(61440K), 0.0061485 secs]
2.195: [Full GC (Ergonomics)  27033K->14385K(61440K), 0.0228773 secs]
2.221: [GC (Allocation Failure)  38961K->20633K(61440K), 0.0030729 secs]
2.227: [GC (Allocation Failure)  45209K->31609K(61440K), 0.0069772 secs]
2.234: [Full GC (Ergonomics)  31609K->15905K(61440K), 0.0257689 secs]
这里重中之重是第三个应用程序中看到的虚引用。使用之前相同的参数运行应用程序情况下与弱引用下的结果非常相似。事实上Full GC的数量要小得多，因为终止化方式不同。

添加(-Dno.ref.clearing=true)禁止虚引用的参数，很快会给我们这样的结果：
4.180: [Full GC (Ergonomics)  57343K->57087K(61440K), 0.0879851 secs]
4.269: [Full GC (Ergonomics)  57089K->57088K(61440K), 0.0973912 secs]
4.366: [Full GC (Ergonomics)  57091K->57089K(61440K), 0.0948099 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
使用虚引用时，必须非常谨慎，需要及时地虚引用可达的对象。如果不这样就会导致OutOfMemoryError错误。在处理引用队列的线程中出现的意外异常，会导致应用程序无法使用。

JVM受到了怎样的影响？

指定JVM参数-XX:+PrintReferenceGC查看不同引用类型对象对GC的影响。如果我们将此参数添加到WeakReference应用程序的例子中，会得到下面的log：
2.173: [Full GC (Ergonomics) 2.234: [SoftReference, 0 refs, 0.0000151 secs]2.234: [WeakReference, 2648 refs, 0.0001714 secs]2.234: [FinalReference, 1 refs, 0.0000037 secs]2.234: [PhantomReference, 0 refs, 0 refs, 0.0000039 secs]2.234: [JNI Weak Reference, 0.0000027 secs][PSYoungGen: 9216K->8676K(10752K)] [ParOldGen: 12115K->12115K(12288K)] 21331K->20792K(23040K), [Metaspace: 3725K->3725K(1056768K)], 0.0766685 secs] [Times: user=0.49 sys=0.01, real=0.08 secs] 
2.250: [Full GC (Ergonomics) 2.307: [SoftReference, 0 refs, 0.0000173 secs]2.307: [WeakReference, 2298 refs, 0.0001535 secs]2.307: [FinalReference, 3 refs, 0.0000043 secs]2.307: [PhantomReference, 0 refs, 0 refs, 0.0000042 secs]2.307: [JNI Weak Reference, 0.0000029 secs][PSYoungGen: 9215K->8747K(10752K)] [ParOldGen: 12115K->12115K(12288K)] 21331K->20863K(23040K), [Metaspace: 3725K->3725K(1056768K)], 0.0734832 secs] [Times: user=0.52 sys=0.01, real=0.07 secs] 
2.323: [Full GC (Ergonomics) 2.383: [SoftReference, 0 refs, 0.0000161 secs]2.383: [WeakReference, 1981 refs, 0.0001292 secs]2.383: [FinalReference, 16 refs, 0.0000049 secs]2.383: [PhantomReference, 0 refs, 0 refs, 0.0000040 secs]2.383: [JNI Weak Reference, 0.0000027 secs][PSYoungGen: 9216K->8809K(10752K)] [ParOldGen: 12115K->12115K(12288K)] 21331K->20925K(23040K), [Metaspace: 3725K->3725K(1056768K)], 0.0738414 secs] [Times: user=0.52 sys=0.01, real=0.08 secs]
像以前一样，只有分析GC的吞吐量和延迟时间才能看出各种引用类型对象对GC的影响。通常情况下GC对非强引用的处理非常少，在许多情况下是0。但是如果引用程序花费大量的时间清除引用或大量的非强引用被清理，那么就需要进一步的分析。

解决方案有哪些？
当你的应用程序实际上过度地使用了（软、弱、虚引用），通常是是更改引用程序的内在逻辑，没有通用的方式，只能根据实际的业务逻辑去修改。另外还有一些通用的解决方案：
弱引用：如果是由特定的内存池对象增加引起的，那么增加这个特定的内存池（可能包括整个heap），可以帮助你解决问题。如上面的示例部分所示增加heap和Young Generation的大小可以缓解这种问题。
虚引用：确保你使用过虚引用对象之后手动清理这些虚引用，很容易忽略某些角落里的虚引用、清理线程无法跟上虚引用对象填充队列的速度，这样会给GC造成很大的压力，肯能到最后造成OutOfMemoryError。
软引用：当软引用造成JVM出现问题时，解决问题的唯一方式是更改应用程序的内在逻辑。

其他例子：

本章将介绍一些不同寻常的问题。
RMI & GC

当你应用程序通过RMI发布消息或消费服务时，JVM会定期启动Full GC，确保本地未使用的对象也不会占用内存空间，即使没有在代码逻辑当中添加任何基于JMI的内容，第三方库或工具类仍然可以打开RMI端点（常见的就是JMX），如果远程连接到它，它就会使用RMI在底层发布数据。

RMI造成的问题通常是Old Generation剩有大量空间，但是会触发Full GC，造成stop-the-world。

删除远程引用对象的行为是通过System.gc()触发的，调用System.gc()在sun.rmi.transport.ObjectTable类中调用“sun.misc.GC.requestLatency(long)”方法，不需要设置当前线程类上下文类加载器（ClassLoader = Thread.currentThread().getContextClassLoader(); TCCL），会创建守护线程进行定期的垃圾回收。

Tomcat中JreMemoryLeakPreventionListener的类将“sun.misc.GC.requestLatency(long)”参数设置成了Long.valueOf(Long.MAX_VALUE - 1)。

深入理解JVM-类加载器深入解析
https://www.cnblogs.com/luozhiyun/p/10506220.html

对于RMI需要定期Full GC的优化，不是必须的，要禁用这种周期性的GC运行，你可以设置以下内容：
java -Dsun.rmi.dgc.server.gcInterval=9223372036854775807L -Dsun.rmi.dgc.client.gcInterval=9223372036854775807L com.yourcompany.YourApplication
这会将System.gc()运行后的周期设置为Long.MAX_VALUE，这样设置大多数来说，这就等价于永远不运行。

RMI导致的频繁地定期Full GC另一种解决方案是通过JVM参数指定“-XX:+DisableExplicitGC”来禁用对System.gc()的显示调用（不推荐，会造成不可知的影响，因为应用程序代码逻辑使用到了，你不确定强行一刀切会造成什么样的影响）

JVMTI tagging & GC

当应用程序与JAVA代理（-javaagent）一起运行时，代理旧有可能使用JVMTI标记堆中的对象。代理可以出于各种原因使用JVMTI进行对象标记，但是如果标记大量的对象，那么GC的性能就会收到影响，从而影响到应用程序的性能。

这个问题存在于本地代码中，其中JvmtiTagMap::do_weak_oops在每个垃圾收集事件期间遍历所有JVMTI标记的对象，并对所有JVMTI标记的对象执行开销不大的操作。更糟糕的是这个操作是顺序执行的不是并行的。

当存在大量的JVMTI标记对象时，这意味着GC过程大部分时间是在单线程中执行的，潜在地增加了一个数量级的GC暂停时间。

要检查指定代理是否会导致延长GC暂停时间，你需要打开诊断选项-XX:+UnlockDiagnosticVMOptions -XX:+TraceJVMTIObjectTagging，可以侦测并估计tag map消耗了多少本机内存，以及遍历这些对象花费了多长时间。

通常你也做不了什么，只能给代理商提建议，修复这种问题。

Humongous Allocations

每当应用程序使用G1收集算法时，GC时对大对象的分配可能会影响应用程序的性能（强调：大对象分配是指大于region大小的50%对象的分配）。

频繁地大对象分配会引发一下GC性能问题：
如果region当中包含大对象，剩余regiong空间将不再被使用，如果对象只是刚刚超过region的50%多一点，那么就会造成很大的空间浪费。
对于大对象的收集，G1并不像常规那样处理，JAVA1.8u40之前，大对象的回收只能在Full GC事件期间完成，Hotspot JVM 最新版本会在清除阶段标记周期结束时释放大区域，对于轿新的JVM来说，问题的影响已经显著下降。
要检查应用是否存在大对象空间需要制定JVM参数：
java -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintReferenceGC -XX:+UseG1GC -XX:+PrintAdaptiveSizePolicy -Xmx128m MyClass
G1出现大对象的日志：
 0.106: [G1Ergonomics (Concurrent Cycles) request concurrent cycle initiation, reason: occupancy higher than threshold, occupancy: 60817408 bytes, allocation request: 1048592 bytes, threshold: 60397965 bytes (45.00 %), source: concurrent humongous allocation]
 0.106: [G1Ergonomics (Concurrent Cycles) request concurrent cycle initiation, reason: requested by GC cause, GC cause: G1 Humongous Allocation]
 0.106: [G1Ergonomics (Concurrent Cycles) initiate concurrent cycle, reason: concurrent cycle initiation requested]
 0.106: [GC pause (G1 Humongous Allocation) (young) (initial-mark) 0.106: [G1Ergonomics (CSet Construction) start choosing CSet, _pending_cards: 0, predicted base time: 10.00 ms, remaining time: 190.00 ms, target pause time: 200.00 ms]
你现在有证据证明应用程序确实在分配大对象GC pause (G1 Humongous Allocation)，大对象请求空间为1048592字节（1M多点），region默认大小是2M（2097152 Bytes），计算得到：16 Bytes = 2097152 * 0.5 - 1048592 ； 大对象只超了16 Bytes

解决方案：
指定JVM参数-XX:G1HeapRegionSize=XX，增大region的空间来覆盖大小，指定范围在1~32M之间
有副作用，增大region的大小，将减少整个region的个数，修改之后需要查看是否提高了应用程序的吞吐量和降低了应用程序的延迟
另一种解决方案是修改应用程序，尽量避免大对象的分配，但是操作起来工作量就比较大

结论：
      JVM之上可能存在运行多个应用程序，加上GC调整的参数上百个，GC性能优化的方面就很多。

JVM调优工具：
JDK Mission Control

https://www.cnblogs.com/duanxz/p/5500494.html
java.lang.Object.finalize()：GC自动收集的是内存中的资源，但是对象持有的其他资源（如：文件和网络连接）GC无法收集，因此
需要在应用程序当中手动调用finalize()方法关闭。
当可终止对象不可达，对该对象的引用对象将放入一个队列当中，并标记该对象，在GC运行期间被视为存活对象。
通过队列一个接一个地删除可终止队列中的对象，并调用这些对象的finalize()方法
在调用finalize()方法之后，对象不会被立即释放，finalize()方法会将被引用对象放到某个位置（例如，public static 的字段），以便复活该对象，使该对象可以再次被引用。因此调用finalize()方法之后，GC必须重新确定该对象是不可达的，然后才进行收集
注意：即使恢复了被引用对象，finalize()方法也不会再次被调用
调用finalize()的方法的对象通常会至少存在一个额外的GC周期(如果他们的生命周期很长，那么至少多存在一个额外的Full GC)
注意：JVM存在没清理完所有垃圾对象的情况下退出，因此，可终结的对象可能永远不会被收集。
在这种情况下，网络连接之类的资源会被操作系统关闭并回收；但是删除文件的finalize()方法不会被操作系统调用。
为了在JVM退出的时候执行某些操作，JAVA提供了Runtime.getRuntime().addShutdownHook(Thread);该方法可以在JVM退出时安全地执行任意方法，传入参数为线程。

The finalize() method is an instance method, and finalizers act on instances. There is no equivalent mechanism for finalizing a class.
finalize()是一个不接收任何参数的方法，每个类只能有一个finalize()方法。
finalize()方法可以抛出任何异常，但是被GC调用的时候异常或错误信息会被忽略。
进程与线程之间的关系？
线程是轻量级的执行单元，比进程要小，但能够执行任意java代码，每个线程都是操作系统的一个可执行单元，但属于一个进程，进程的地址空间被包含在进程内的线程所共享，这意味着，每个线程可以被独立调用，每个线程有自己独立的栈和程序计数器。但是要和进程中的其他线程共享堆和对象。

JAVA程序启动和原始应用线程执行过程：
程序员执行Main()方法
JVM启动（应用程序在JVM上下文中运行）
JVM加载类，发现应用程序的入口点（main()）被运行
假设启动类通过了类加载检查，就会启动一个用于执行程序的专用线程主线程
JVM字节码解释器在主线程上启动
主线程的字节码解释器读取到main()方法的字节码，然后开始执行每一个字节码

每个Java程序同时这样开始：
每个JAVA程序都作为托管模型的一部分开始，每个线程有一个解释器。
JVM拥有控制应用程序线程的能力
线程机制：允许应用程序新建的线程与（主线程、JVM以各种目的启动的线程）并发执行。每次调用Thread::start()方法时，调用权都是委托给了操作系统，新的OS线程exec()是JVM字节码编译器的副本，解释器开始执行线程run()方法中的内容。这意味着应用程序线程有访问操作系统调度器控制的CPU。（操作系统调度器是操作系统的内置部分负责管理处理器的时间片）。线程的执行的时间不允许超过操作系统调度器给该线程分配的CPU时间片。

线程的生命周期：
NEW（创建）：线程已经被创建，但是线程的start()方法还没有被调用，所有线程都以这种状态启动。
RUNNABLE（就绪状态、可运行状态）：线程正在运行，允许操作系统调度器调度该线程。
BLOCKED（阻塞）：线程阻塞，

什么时候触发Full GC：
程序调用了System.gc()
调优方法：1、尽量使用；2、指定命令-XX：+DisableExplicitGC禁用RMI调用System.gc
老年代空间不足
原因：1、从年轻代晋升到老年代的对象大于老年代剩余可用空间；2、创建的对象或数组过大老年代存放不下；
java.lang.OutOfMemoryError: Java heap space
调优方法：
尽量在Minor GC阶段被回收
尽量不要创建过大的对象或数组
PermGen/Metaspace空间不足
该区域存放内容：class信息、常量、静态变量等信息
导致空间不足的原因：应用程序加载的类、反射的类和调用的方法较多，PermGen/MetaSpace空间会被占满
1.7：java.lang.OutOfMemoryError: PermGen space
1.7 ：设置参数：-XX:PermSize=xxM -XX:MaxPermSize=xxM
1.8：-XX:MetaspaceSize=xxM（默认是21M）   -XX:MaxMetaspaceSize=XXM(默认没有限制)
-XX:MinMetaspaceFreeRatio=N （在Metaspace GC过后会进行计算当前Metaspace的剩余可用空间大小，默认是40%，如果空闲空间小于当前整个MetaSpace整个空间大小的40%，就会扩大MetaSpace的空间大小）
-XX:MaxMetaspaceExpansion （默认单位为Bytes）扩展幅度：默认大约为5M多点。   -XX:MinMetaspaceExpansion:（默认单位为Bytes）最小扩展幅度是0.3M
CMS收集老年代时可能会出现promotion failed和concurrent mode failure
https://my.oschina.net/hosee/blog/674181
 promotion failed是进行Minor GC时，将对象晋升到老年代时，老年代放不下造成的，多数情况下老年代空间够大，但是碎片较多。
106.641: [GC 106.641: [ParNew (promotion failed): 14784K->14784K(14784K), 0.0370328 secs]106.678: [CMS106.715: [CMS-concurrent-mark: 0.065/0.103 secs] [Times: user=0.17 sys=0.00, real=0.11 secs]
(concurrent mode failure): 41568K->27787K(49152K), 0.2128504 secs] 52402K->27787K(63936K), [CMS Perm : 2086K->2086K(12288K)], 0.2499776 secs] [Times: user=0.28 sys=0.00, real=0.25 secs]
解决方案：增大Survivor空间大小、增大对象在Survivor存放寿命阈值或
concurrent mode failure
CMS的并发收集就不起作用了，就会使用Serial Old进行收集，会stop-the-world
0.195: [GC 0.195: [ParNew: 2986K->2986K(8128K), 0.0000083 secs]0.195: [CMS0.212: [CMS-concurrent-preclean: 0.011/0.031 secs] [Times: user=0.03 sys=0.02, real=0.03 secs]
(concurrent mode failure): 56046K->138K(57344K), 0.0271519 secs] 59032K->138K(65472K), [CMS Perm : 2079K->2078K(12288K)], 0.0273119 secs] [Times: user=0.03 sys=0.00, real=0.03 secs]
在老年代进行CMS GC的时候，有对象晋升到老年代，老年代存放不下，或者大对象存放老年代，老年代存放不下，导致的。
解决方案：-XX:UseCMSCompactAtFullCollection  -XX:CMSFullGCBeforeCompaction=0 CMSInitiatingOccupancyFraction=-1
-XX:UseCMSCompactAtFullCollection在FullGC的时候对老年代进行标记压缩，较少碎片，该参数默认是开启，检查下，防止未开启。
-XX:CMSFullGCsBeforeCompaction=0，Full GC进行多少次标记清除之后，进行一次标记压缩，默认是0，表示每次Full GC都会进行标记清除和标记压缩。
CMSInitiatingOccupancyFraction=-1，次参数设置CMS触发的阈值，默认是-1，根据jdk源码的逻辑计算我们可以知道是92%
https://blog.csdn.net/insomsia/article/details/91802923
<0时的计算公式：老年代阈值 = ((100 - MinHeapFreeRatio) + (double)( tr * MinHeapFreeRatio)/100)/100
>=0时,老年代的阈值CMSInitiatingOccupancyFraction/100









