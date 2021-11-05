## Flink四大基石

Flink之所以能这么流行，离不开它最重要的四个基石：Checkpoint、State、Time、Window。

![1622103858340](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/140526-259786.png)

**Checkpoint**

- 这是Flink最重要的一个特性。
- Flink基于Chandy-Lamport算法实现了一个分布式的一致性的快照，从而提供了一致性的语义。
- Chandy-Lamport算法实际上在1985年的时候已经被提出来，但并没有被很广泛的应用，而Flink则把这个算法发扬光大了。
- Spark最近在实现Continue streaming，Continue streaming的目的是为了降低处理的延时，其也需要提供这种一致性的语义，最终也采用了Chandy-Lamport这个算法，说明Chandy-Lamport算法在业界得到了一定的肯定。`https://zhuanlan.zhihu.com/p/53482103`

**State** 

- 提供了一致性的语义之后，Flink为了让用户在编程时能够更轻松、更容易地去管理状态，还提供了一套非常简单明了的State API，包括ValueState、ListState、MapState，BroadcastState。

**Time**

- 除此之外，Flink还实现了Watermark的机制，能够支持基于事件的时间的处理，能够容忍迟到/乱序的数据。

**Window**

- 另外流计算中一般在对流数据进行操作之前都会先进行开窗，即基于一个什么样的窗口上做这个计算。Flink提供了开箱即用的各种窗口，比如滑动窗口、滚动窗口、会话窗口以及非常灵活的自定义的窗口。