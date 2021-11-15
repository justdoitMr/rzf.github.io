
<!-- TOC -->

- [什么是状态一致性？](#什么是状态一致性)
- [分类：](#分类)
- [面试题：](#面试题)
  - [flink是如何保证状态一致性的？（内部保证）](#flink是如何保证状态一致性的内部保证)
- [Exactly-once两阶段提交步骤](#exactly-once两阶段提交步骤)
- [Flink+Kafka实现端到端的精确一致性](#flinkkafka实现端到端的精确一致性)

<!-- /TOC -->

## 什么是状态一致性？

- 有状态的流处理，内部每个算子任务都可以有自己的状态。
- **对于流处理内部来说，所谓的状态一致性，其实就是我们所说的计算结果要保证准确。**
- 一条数据不应该丢失，也不应该重复计算。
- 在遇到故障的时候可以状态恢复，恢复以后重新计算，计算结果也是完全正确的。

## 分类：

- AT-MOST-ONCE(最多一次):
  
  当任务故障时，最简单的做法是什么都不干，既不恢复丢失的状态，也不重播丢失的数据。At-most-once 语义的含义是最多处理一次事件。这种情况会发生数据的丢失。
- AT-LEAST-ONCE(至少一次):

  在大多数的真实应用场景，我们希望不丢失事件。这种类型的保障称为 at-least-once，意思是所有的事件都得到了处理，而一些事件还可能被处理多次。这种情况不会丢失事件，但是数据会重复处理。
- EXACTLY-ONCE(精确一次):
  
  恰好处理一次是最严格的保证，也是最难实现的。恰好处理一次语义不仅仅意味着没有事件丢失，还意味着针对每一个数据，内部状态仅仅更新一次。

## 面试题：

### flink是如何保证状态一致性的？（内部保证）

- flink使用了一种轻量级的快照机制--**检查点**（checkpoint）来保证exactly-once语义。
- **所有任务的状态，在某个时间点的一份快照。这个时间点，所有任务都恰好处理完一个相同的数据输入。**
- 应用状态的一致检查点，是 Flink 故障恢复机制的核心

**端到端（end-to-end）状态一致性**

- 在flink的流处理器内部由**流处理器**实现一致性，**一致性除了需要在flink内部保证还需要保证数据源和输出端的一致性。**
- **端到端的一致性，意味着整个数据流，每个组件都有自己的一致性。**
- **一致性的级别取决于所有组件中一致性最弱的组件。**

**端到端的保证：**

- **内部保证-- - checkpoint**
- **source端- -可重设数据的读取位置**
- **sink端一-从故障恢复时，数据不会重复写入外部系统**

```text
两种写入方式：
幂等写入：一个操作多次执行，但是只改一次结果，重复的不起作用。
事务写入：构建的事务对应着checkpoint,等到checkpoint真正完成的时候，才把对应的结果写入sink中。
```

**两种事务写入的实现方式:**

```text
1. 预写日志( Write- Ahead-Log , WAL )
2. 把结果数据先当成状态保存，然后在收到checkpoint完成的通知时,一次性写入sink系统
3. 简单易于实现，由于数据提前在状态后端中做了缓存，所以无论什么sink系统，都能用这种方式一批搞定
4. DataStream API提供了-个模板类: GenericWriteAheadSink, 来实现这种事务性sink

两阶段提交(Two- Phase-Commit,2PC )
1. 对于每个checkpoint, sink任务会启动一个事务，并将接下来所有接收的数据添加到事务里
2. 然后将这些数据写入外部sink系统，但不提交它们--这时只是“预提交”)
3. 当它收到checkpoint完成的通知时，它才正式提交事务，实现结果的真正写入

这种方式真正实现了exactly-once，它需要一个提供事 务支持的外部sink系统。

Flink 提供了TwoPhaseCommitSinkFunction接口。
```

> 2PC对外部sink系统的要求:

- 外部sink系统必须提供事务支持，或者sink任务必须能够模拟外部系统上的事务。
- 在checkpoint的间隔期间里，必须能够开启一个事务并接受数据写入。
- 在收到checkpoint完成的通知之前，事务必须是“等待提交”的状态。在故障恢复的情况下，这可能需要-些时间。如果这个时候sink系统关闭事务(例如超时了) ，那么未提交的数据就会丢失
- sink任务必须能够在进程失败后恢复事务。
- 提交事务必须是幂等操作。

## Exactly-once两阶段提交步骤

- 第一条数据来了之后，开启一个kafka的事务(transaction) ，正常写入kafka分区日志但标记为未提交,这就是“预提交”
- jobmanager 触发checkpoint操作，barrier 从source开始向下传递，遇到barrier的算子将状态存入状态后端,并通知jobmanager
- sink 连接器收到barrier,保存当前状态，存入checkpoint,通知jobmanager,并开启下一阶段的事务，用于提交下个检查点的数据:
- jobmanager收到所有任务的通知，发出确认信息，表示checkpoint完成
- sink任务收到jobmanager的确认信息，正式提交这段时间的数据
- 外部kafka关闭事务，提交的数据可以正常消费了。

## Flink+Kafka实现端到端的精确一致性

我们知道，端到端的状态一致性的实现，需要每一个组件都实现，对于 Flink + Kafka 的数据管道系统（Kafka 进、Kafka 出）而言，各组件怎样保证 exactly-once 语义呢？

    内部 —— 利用 checkpoint 机制，把状态存盘，发生故障的时候可以恢复， 保证内部的状态一致性
    source —— kafka consumer 作为 source，可以将偏移量保存下来，如果后续任务出现了故障，恢复的时候可以由连接器重置偏移量，重新消费数据，保证一致性
    sink —— kafka producer 作为 sink，采用两阶段提交 sink，需要实现一个 TwoPhaseCommitSinkFunction

我们知道 Flink 由 JobManager 协调各个 TaskManager 进行 checkpoint 存储， checkpoint 保存在 StateBackend 中，默认StateBackend 是内存级的，也可以改为文件级的进行持久化保存。

![20211115093522](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115093522.png)

当 checkpoint 启动时，JobManager 会将检查点分界线（barrier）注入数据流； barrier 会在算子间传递下去。

![20211115093547](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115093547.png)

每个算子会对当前的状态做个快照，保存到状态后端。对于 source 任务而言，就会把当前的 offset 作为状态保存起来。下次从 checkpoint 恢复时，source 任务可以重新提交偏移量，从上次保存的位置开始重新消费数据。

![20211115093620](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115093620.png)

每个内部的 transform 任务遇到 barrier 时，都会把状态存到 checkpoint 里。

sink 任务首先把数据写入外部 kafka，这些数据都属于预提交的事务（还不能被消费）；当遇到barrier 时，把状态保存到状态后端，并开启新的预提交事务。

![20211115093701](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115093701.png)

当所有算子任务的快照完成，也就是这次的 checkpoint 完成时，JobManager 会向所有任务发通知，确认这次 checkpoint 完成。

当 sink 任务收到确认通知，就会正式提交之前的事务，kafka 中未确认的数据就改为“已确认”，数据就真正可以被消费了。

![20211115093806](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115093806.png)

所以我们看到，执行过程实际上是一个两段式提交，每个算子执行完成，会进行“预提交”，直到执行完 sink 操作，会发起“确认提交”，如果执行失败，预提交会放弃掉。

具体的两阶段提交步骤总结如下：

1. 第一条数据来了之后，开启一个 kafka 的事务（transaction），正常写入 kafka 分区日志但标记为未提交，这就是“预提交”
2. jobmanager 触发 checkpoint 操作，barrier 从 source 开始向下传递，遇到 barrier 的算子将状态存入状态后端，并通知 jobmanager
3. sink 连接器收到 barrier，保存当前状态，存入 checkpoint，通知 jobmanager，并开启下一阶段的事务，用于提交下个检查点的数据
4. jobmanager 收到所有任务的通知，发出确认信息，表示 checkpoint 完成
5. sink 任务收到 jobmanager 的确认信息，正式提交这段时间的数据外部 kafka 关闭事务，提交的数据可以正常消费了。

所以我们也可以看到，如果宕机需要通过 StateBackend 进行恢复，只能恢复所有确认提交的操作。