# 垃圾回收器
> Note: 本文的内容都是来源于文中链接的文章，结合自己的理解所写，仅作自己参考使用。英语能力有限，有些概念可能理解有误，请勿轻信。
 
> 调优：https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/
> basic: https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html


## Serial Garbage Collector
串行收集器使用单个线程来执行垃圾回收。进行回收时，应用程序将会暂停。

## Parallel Garbage Collector
并行收集器与串行收集器的主要区别在于使用多个线程来加速垃圾回收。进行回收时，应用程序将会暂停。
并行收集器也被称为吞吐量收集器。
并行收集器默认会启用并行压缩，会执行整个堆压缩。

通过命令行选项 `-XX:+UseParallelGC` 启用并行收集器。


## Mostly Concurrent Collector
串行收集器和并行收集器在执行垃圾回收时，应用程序将会暂停，而并发收集器在应用程序运行时也能执行垃圾回收过程中的大部分（Mostly）操作。

并发收集器会使用应用程序的处理器资源来缩短主要收集的暂停时间。最明显的开销是在回收器的并发操作期间使用一个或多个处理器资源（此时应用程序可能仍在运行，因此应用程序的吞吐量将受到影响）。
以下两种 `CMS` 与 `G1` 都属于并发收集器，但这两者的实现有很大的区别。

### CMS Garbage Collector
> https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mark_sweep_cms_collector

`CMS` 使用标记清除算法来执行垃圾回收，不会执行压缩。

以下列举 `CMS` 执行回收的两个主要特性:

#### 暂停  
`CMS` 收集器在并发收集周期内暂停应用程序两次:
1. 第一次暂停发生在将对象标记为活动对象时。第一次停顿称为 *initial mark pause* （初始标记停顿）。
2. 第二次暂停出现在并发跟踪阶段（见下文）的末尾，并且发现了并发跟踪遗漏的对象。这些对象是在 `CMS` 收集器完成对对象的跟踪之后，由于应用程序线程对对象的引用进行了更新，因此产生了一些对象现在应当被标记但却没有被标记的的情况。这第二次停顿称为 *remark pause* （再标记停顿）。 

年轻代收集和老年代收集的暂停是独立发生的。它们不会同时发生，但可能快速连续发生。

#### 并发阶段
应用程序在并发阶段不会暂停。

##### concurrent tracing phase
可访问对象图的并发跟踪发生在 *initial mark pause* 和 *remark pause* 之间。
在此并发跟踪阶段，一个或多个并发垃圾收集器线程可能正在使用本来应用程序可以使用的处理器资源。因此，即使应用程序线程没有暂停，应用程序在这个阶段和其他并发阶段的吞吐量也会相应下降。

##### concurrent sweeping phase 
在 *remark pause* 之后，并发扫描阶段收集标识为不可达的对象（垃圾）。一旦收集周期完成，`CMS` 收集器将等待，几乎不消耗任何计算资源，直到下一个收集周期开始。

值得注意的是：`CMS` 并不会压缩空间，

### G1 Garbage Collector
> https://www.oracle.com/java/technologies/javase/hotspot-garbage-collection.html
> https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html
> https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html

`G1`（Garbage First）收集器适用于具有大内存的多核处理器计算机，该收集器延迟低，并且能压缩空间减少碎片化。

与 `CMS` 一样，`G1` 也会在应用程序运行时执行一部分的回收操作。

`G1` 将堆划分为一组大小相等的堆区域，每个区域都是一个连续的虚拟内存范围。因此内存越大，对 `G1` 而言越好。

`G1` 在并发全局标记阶段确定整个堆中对象的活跃度。标记阶段完成后，`G1` 就知道了哪些区域的可回收的垃圾会更多。`G1` 将会集中在这些可能充满可回收对象（即垃圾）的堆区域进行收集和压缩活动，这就是为什么被称为垃圾优先的原因。


#### 分代
从逻辑上讲，`G1`是存在世代的概念。

其中一组空区域将会被指定为逻辑上的年轻代。当年轻代满员时，这组区域将被执行垃圾收集也就是 `young collection` (有时也叫 `young gc`)。
在某些情况下，年轻代之外的区域（老年代）可以同时执行垃圾收集。这被称为 *mixed collection* （`mixed gc`）。

`G1` 将活动对象从堆的一个或多个区域复制到堆上选定的、最初为空的区域，根据存活对象的年龄，可以将该对象复制到幸存区，并在此过程中释放内存。这些操作被称为 `evacuation`（疏散）。
疏散在多处理器上并行执行（并不是并发，会暂停），以减少暂停时间并提高吞吐量。

#### 暂停
`G1` 会暂停应用程序以将活动对象复制到新区域。

这些暂停可以是仅收集年轻代的 `young gc` 暂停，也可以是年轻代和老年代的 `mixed gc` 暂停。
与 `CMS` 一样，在应用程序停止时，有一个*remark pause* 以完成标记。 `G1` 将 *initial mark* 作为疏散暂停的一部分进行。
`G1` 在收集的末尾有一个清理阶段，该阶段部分操作是 `STW`（stop-the-world），部分操作是并发的。清理阶段的 `STW` 部分标识空区域，并确定作为下一个收集候选项的区域。

`G1` 使用暂停预测模型来满足用户定义的暂停时间目标，并根据指定的暂停时间目标选择要收集的区域数量。

`G1` 相对于 `CMS` 有几个好处：
1. `G1` 执行垃圾回收会进行压缩减少碎片化，而 `CMS` 不会。
2. `G1` 提供比 `CMS` 收集器更可预测的垃圾回收暂停时间，并允许用户指定所需的暂停时间目标。
