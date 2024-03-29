---
title: 详解CMS垃圾回收机制
date: 2016-04-23 22:31:15
tags:
    - JAVA
    - CMS
comments: true
categories:
    - 技术
typora-root-url: ../../source
typora-copy-images-to: ../../source/img
---
## 什么是CMS？
Concurrent Mark Sweep。
看名字就知道，CMS是一款并发、使用标记-清除算法的gc。
CMS是针对老年代进行回收的GC。

<!-- more -->
## CMS有什么用？
CMS以获取最小停顿时间为目的。
在一些对响应时间有很高要求的应用或网站中，用户程序不能有长时间的停顿，CMS 可以用于此场景。
## CMS如何执行？
总体来说CMS的执行过程可以分为以下几个阶段：  
### 初始标记 
初始标记阶段需要STW(Stop The World)。
该阶段进行可达性分析，标记GC ROOT能直接关联到的对象。
注意是**直接关联**,间接关联的对象在下一阶段标记。
### 并发标记
此阶段是和用户线程并发执行的过程。
该阶段进行GC ROOT TRACING，在第一个阶段被暂停的线程重新开始运行。
由前阶段标记过的对象出发，所有可到达的对象都在本阶段中标记。
### 并发预清理
并发预处理阶段做的工作还是标记，与3.4的重标记功能相似。
既然相似为什么要有这一步？
前面我们讲过，CMS是以获取最短停顿时间为目的的GC。
重标记需要STW，因此重标记的工作尽可能多的在并发阶段完成来减少STW的时间。
此阶段标记**从新生代晋升的对象、新分配到老年代的对象**以及在**并发阶段被修改了的对象**。
*****
此阶段比较复杂，从大家容易忽略或者说不理解的地方抛出一个问题大家思考下：
  ● **如何确定老年代的对象是活着的？**
答案很简单，通过GC ROOT TRACING可到达的对象就是活着的。
继续延伸，如果存在以下场景怎么办：
{% asset_img new_reference_old.png new reference old %}   


老年代进行GC时如何确保上图中Current Obj标记为活着的？
（确认新生代的对象是活着的也存在相同问题，大家可以思考下，文章后面会给出答案）
 答案是必须扫描新生代来确保。**这也是为什么CMS虽然是老年代的gc，但仍要扫描新生代的原因**。(注意初始标记也会扫描新生代)
在CMS日志中我们可以清楚地看到扫描日志：
>[GC[YG occupancy: 820 K (6528 K)]
>[Rescan (parallel) , 0.0024157 secs]
>[weak refs processing, 0.0000143 secs]
>[scrub string table, 0.0000258 secs] 
>[1 CMS-remark: 479379K(515960K)] 480200K(522488K), 0.0025249 secs] 
>[Times: user=0.01 sys=0.00, real=0.00 secs]
>Rescan阶段(remark阶段的一个子阶段)会扫描新生代和老年代中的对象。在日志中可以看到此阶段标识为Rescan (parallel)，说明此阶段是并行进行的。

**重点来了：全量的扫描新生代和老年代会不会很慢？**肯定会。
CMS号称是停顿时间最短的GC，如此长的停顿时间肯定是不能接受的。
如何解决呢？
大家可以先思考下。

 

**必须要有一个能够快速识别新生代和老年代活着的对象的机制。**
**先说新生代。**
新生代垃圾回收完剩下的对象全是活着的，并且活着的对象很少。
**如果在扫描新生代前进行一次Minor GC，情况是不是就变得好很多？**
CMS 有两个参数：**CMSScheduleRemarkEdenSizeThreshold**、**CMSScheduleRemarkEdenPenetration**，默认值分别是2M、50%。两个参数组合起来的意思是预清理后，eden空间使用超过2M时启动可中断的并发预清理（CMS-concurrent-abortable-preclean），直到eden空间使用率达到50%时中断，进入remark阶段。
如果能在可中止的预清理阶段发生一次Minor GC,那就万事大吉、天下太平了。 
这里有一个小问题,**可终止的预清理要执行多长时间来保证发生一次Minor GC?**
答案是没法保证。道理很简单，因为垃圾回收是JVM自动调度的,什么时候进行GC我们控制不了。
但此阶段总有一个执行时间吧？是的。
CMS提供了一个参数**CMSMaxAbortablePrecleanTime** ，默认为5S。
只要到了5S，不管发没发生Minor GC，有没有到CMSScheduleRemardEdenPenetration都会中止此阶段，进入remark。
如果在5S内还是没有执行Minor GC怎么办？
CMS提供CMSScavengeBeforeRemark参数，使remark前强制进行一次Minor GC。
这样做利弊都有。好的一面是减少了remark阶段的停顿时间;坏的一面是Minor GC后紧跟着一个remark pause。如此一来，停顿时间也比较久。
CMS日志如下：
>7688.150: [CMS-concurrent-preclean-start]
>7688.186: [CMS-concurrent-preclean: 0.034/0.035 secs]
>7688.186: [CMS-concurrent-abortable-preclean-start]
>7688.465: [GC 7688.465: [ParNew: 1040940K->1464K(1044544K), 0.0165840 secs] 1343593K->304365K(2093120K), 
>0.0167509 secs]7690.093: [CMS-concurrent-abortable-preclean: 1.012/1.907 secs]  7690.095: [GC[YG occupancy: 522484 K (1044544 K)]
>7690.095: [Rescan (parallel) , 0.3665541 secs]7690.462: [weak refs processing, 0.0003850 secs] [1 CMS-remark: 302901K(1048576K)] 825385K(2093120K), 0.3670690 secs]

**7688.186启动了可终止的预清理，在随后的三秒内启动了Minor GC，然后进入了Remark阶段.**
实际上为了减少remark阶段的STW时间，预清理阶段会尽可能多做一些事情来减少remark停顿时间。
remark的rescan阶段是多线程的，为了便于多线程扫描新生代，**预清理阶段会将新生代分块**。
每个块中存放着多个对象，这样remark阶段就不需要从头开始识别每个对象的起始位置。
多个线程的职责就很明确了，把分块分配给多个线程，很快就扫描完。
遗憾的是，这种办法仍然是建立在发生了Minor GC的条件下。
如果没有发生Minor GC，top（下一个可以分配的地址空间）以下的所有空间被认为是一个块(这个块包含了新生代大部分内容)。
这种块对于remark阶段并不会起到多少作用，因此并行效率也会降低。
*****
**ok，新生代的机制讲完了，下面讲讲老年代**。
老年代的机制与一个叫**CARD TABLE**的东西（这个东西其实就是个数组,数组中每个位置存的是一个byte）密不可分。
CMS将老年代的空间分成大小为512bytes的块，card table中的每个元素对应着一个块。
并发标记时，如果某个对象的引用发生了变化，就标记该对象所在的块为** dirty card**。
并发预清理阶段就会重新扫描该块，将该对象引用的对象标识为可达。
举个例子：
并发标记时对象的状态：
{% asset_img concurrent_mark.png concurrent mark %}


**但随后current obj的引用发生了变化：**
{% asset_img referrence_changed.png referrence changed %} 

current obj所在的块被标记为了dirty card。
随后到了pre-cleaning阶段，还记得该阶段的任务之一就是标记这些在并发标记阶段被修改了的对象么？之后那些通过current obj变得可达的对象也被标记了，变成下面这样：
{% asset_img dirty_card.png dirty card %}

同时dirty card标志也被清除。
老年代的机制就是这样。
**不过card table还有其他作用**。
还记得前面提到的那个问题么？进行Minor GC时,如果有老年代引用新生代，怎么识别？
(有研究表明，在所有的引用中，老年代引用新生代这种场景不足1%.原因大家可以自己分析下)
当有老年代引用新生代，对应的card table被标识为相应的值（card table中是一个byte，有八位，约定好每一位的含义就可区分哪个是引用新生代，哪个是并发标记阶段修改过的）。
**所以，Minor GC通过扫描card table就可以很快的识别老年代引用新生代**。
这里提一下，hotspot 虚拟机使用字节码解释器、JIT编译器、 write barrier维护 card table。
当字节码解释器或者JIT编译器更新了引用，就会触发write barrier操作card table.
再提一下，由于card table的存在，当老年代空间很大时会发生什么？（这里大家可以自由发挥想象）
至此，预清理阶段的工作讲完。
### 重标记(STW)
暂停所有用户线程，重新扫描堆中的对象，进行可达性分析,标记活着的对象。
有了前面的基础，这个阶段的工作量被大大减轻，停顿时间因此也会减少。
注意这个阶段是多线程的。
### 并发清理。
用户线程被重新激活，同时清理那些无效的对象。
### 重置
CMS清除内部状态，为下次回收做准备。 
*****
CMS执行过程讲完了，重点讲解了并发预清理时的操作及CMS几个关键参数。你们可以消化一下，消化完了可以休息一下，因为事情还没结束。
## CMS有什么问题
every coin has two sides ------高中英语作文我经常用的一句话。
在我看来，**CMS这三个字母就隐含了问题所在。并发+标记-清除算法 是问题的来源。** 
### 并发
#### 抢占CPU
并发意味着多线程抢占CPU资源，即GC线程与用户线程抢占CPU。这可能会造成用户线程执行效率下降。
CMS默认的回收线程数是**(CPU个数+3)/4**。这个公式的意思是当CPU大于4个时,保证回收线程占用至少25%的CPU资源，这样用户线程占用75%的CPU，这是可以接受的。
但是，如果CPU资源很少，比如只有两个的时候怎么办？按照上面的公式，CMS会启动1个GC线程。相当于GC线程占用了50%的CPU资源，这就可能导致用户程序的执行速度忽然降低了50%，50%已经是很明显的降低了。
这种场景怎么处理呢？
我给的答案是可以不用考虑这种场景。现在的PC机中都至少有双核处理器，更别说大型的服务器了。
CMS的解决方案是提供了一个 incremental mode（增量模式）。
在这种模式下，进行并发标记、清理时让GC线程、用户线程交替运行，尽量减少GC线程独占CPU资源的时间。
这会造成GC时间更长，但对用户线程造成的影响就会少一些。
但实践证明，这种模式下CMS的表现很一般，并没有什么大的优化。
i-CMS已经被声明为“deprecated”，不再提倡使用。
(https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector)

#### 浮动垃圾
并发清理阶段用户线程还在运行，这段时间就可能产生新的垃圾，新的垃圾在此次GC无法清除，只能等到下次清理。这些垃圾有个专业名词：浮动垃圾。
由于垃圾回收阶段用户线程仍在执行，必需预留出内存空间给用户线程使用。因此不能像其他回收器那样，等到老年代满了再进行GC。
CMS 提供了CMSInitiatingOccupancyFraction参数来设置老年代空间使用百分比,达到百分比就进行垃圾回收。
这个参数默认是92%，参数选择需要看具体的应用场景。
设置的太小会导致频繁的CMS GC，产生大量的停顿；反过来想，设置的太高会发生什么？
假设现在设置为99%，还剩1%的空间可以使用。
在并发清理阶段，若用户线程需要使用的空间大于1%，就会产生Concurrent  Mode Failure错误，意思就是说并发模式失败。
这时，虚拟机就会启动备案：使用Serial Old收集器重新对老年代进行垃圾回收.如此一来，停顿时间变得更长。
所以CMSInitiatingOccupancyFraction的设置要具体问题具体分析。
网上有一些设置此参数的公式，个人认为不是很严谨(原因就是CMS另外一个问题导致的),因此不写出来以免大家疑惑。

其实CMS有动态检查机制。
CMS会根据历史记录，预测老年代还需要多久填满及进行一次回收所需要的时间。
在老年代空间用完之前，CMS可以根据自己的预测自动执行垃圾回收。
这个特性可以使用参数UseCMSInitiatingOccupancyOnly来关闭。

这里提个问题，如果让你设计，**如何预测什么时候开始自动执行**？

### 标记-清除 
4.3 前两个问题是由并发引起的，接下来要说的问题就是由标记-清除算法引起的。
使用标记-清除算法可能造成大量的空间碎片。空间碎片过多，就会给大对象分配带来麻烦。
往往老年代还有很大剩余空间，但无法找到足够大的连续空间来分配当前对象,不得不触发一次Full GC。
CMS的解决方案是使用UseCMSCompactAtFullCollection参数(默认开启)，在顶不住要进行Full GC时开启内存碎片整理。
这个过程需要STW，碎片问题解决了,但停顿时间又变长了。
虚拟机还提供了另外一个参数CMSFullGCsBeforeCompaction，用于设置执行多少次不压缩的Full GC后，跟着来一次带压缩的（默认为0，每次进入Full GC时都进行碎片整理）。
延伸一个“foreground collector”的东西给大家，这个玩意在Java8中也声明为deprecated。(https://bugs.openjdk.java.net/browse/JDK-8027132)
CMS存在的问题已经讲清楚，大家消化下。
*****
至此，CMS相关内容已经讲完。

## 总结
**CMS采用了多种方式尽可能降低GC的暂停时间,减少用户程序停顿**。
**停顿时间降低的同时牺牲了CPU吞吐量** 。
**这是在停顿时间和性能间做出的取舍，可以简单理解为"空间(性能)"换时间**。

文中提到的几个问题大家可以把自己当成设计者来思考。

## 参考资料
<https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector>
<https://blogs.oracle.com/jonthecollector/entry/did_you_know>
<http://dept.cs.williams.edu/~freund/cs434/hotspot-gc.pdf>
<https://plumbr.eu/handbook/garbage-collection-algorithms-implementations>
<https://blogs.msdn.microsoft.com/abhinaba/2009/03/02/back-to-basics-generational-garbage-collection/>
<https://bugs.openjdk.java.net/browse/JDK-8027132>
《深入理解Java虚拟机 JVM高级特性与最佳实践》

