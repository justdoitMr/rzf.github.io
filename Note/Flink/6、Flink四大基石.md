## Flink四大基石
<!-- TOC -->

- [Flink四大基石](#flink四大基石)

<!-- /TOC -->
Flink之所以能这么流行，离不开它最重要的四个基石：

- Checkpoint：实现分布式一致性快照，提供精确性一致性语义，
- State：存储各个算子的中间计算结果，加速计算
- Time：三种时间语义
- Window：实现流式计算

![1622103858340](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/140526-259786.png)

**Checkpoint**

- 这是Flink最重要的一个特性。
- Flink基于Chandy-Lamport算法实现了一个**分布式的一致性的快照，从而提供了一致性的语义**。
- Chandy-Lamport算法实际上在1985年的时候已经被提出来，但并没有被很广泛的应用，而Flink则把这个算法发扬光大了。
- Spark最近在实现Continue streaming，Continue streaming的目的是为了降低处理的延时，其也需要提供这种一致性的语义，最终也采用了Chandy-Lamport这个算法，说明Chandy-Lamport算法在业界得到了一定的肯定。`https://zhuanlan.zhihu.com/p/53482103`

**State** 

- 提供了一致性的语义之后，Flink为了让用户在编程时能够更轻松、更容易地去管理状态，还提供了一套非常简单明了的State API，包括ValueState、ListState、MapState，BroadcastState。
- 保存各个算子的中间计算状态，可以加速计算，不需要重新开始计算。

所谓状态就是在流式计算过程中将算子的中间结果保存在内存或者文件系统中，等下一个事件进入算子后可以从之前的状态中获取中间结果，计算当前的结果，从而无须每次都基于全部的原始数据来统计结果，极大的提升了系统性能，状态化意味着应用可以维护随着时间推移已经产生的数据聚合

![1621660055700](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165454-678728.png)

**Time**

- 除此之外，Flink还实现了Watermark的机制，能够支持基于**事件的时间**的处理，能够**容忍迟到/乱序**的数据。

**Window**

- 另外流计算中一般在对流数据进行操作之前都会先进行开窗，即基于一个什么样的窗口上做这个计算。Flink提供了开箱即用的各种窗口，比如滑动窗口、滚动窗口、会话窗口以及非常灵活的自定义的窗口。