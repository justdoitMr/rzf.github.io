## 时间语义
<!-- TOC -->

- [时间语义](#时间语义)
  - [Flink 中的时间语义](#flink-中的时间语义)
  - [EventTime 的引入](#eventtime-的引入)
  - [Watermark](#watermark)
    - [水位线（Watermark）](#水位线watermark)
    - [基本概念](#基本概念)
    - [watermark详解](#watermark详解)
    - [watermark的特点](#watermark的特点)
    - [watermark的传递](#watermark的传递)
    - [Watermark 的引入](#watermark-的引入)
      - [Assigner with periodic watermarks](#assigner-with-periodic-watermarks)
      - [Assigner with punctuated watermarks](#assigner-with-punctuated-watermarks)
    - [watermark的设定](#watermark的设定)
    - [并行度问题](#并行度问题)
    - [案例](#案例)
    - [源码说明](#源码说明)
    - [数据丢失问题](#数据丢失问题)
  - [小结](#小结)

<!-- /TOC -->

### Flink 中的时间语义 

- watermark。
- `.allowedLateness(Time.seconds(1))`允许迟到一段时间
- 使用侧输出流进行输出。

> 不做设置的话默认是处理时间语义。

在 Flink 的流式处理中，会涉及到时间的不同概念，如下图所示： 

![1622880880702](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/161443-627644.png)

![1614644740170](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/17/105437-808486.png)

- Event Time：是事件创建的时间。它通常由事件中的时间戳描述，例如采集的日志数据中，每一条日志都会记录自己的生成时间， Flink 通过**时间戳分配器访问事件时间戳**。 指定算子根据数据自身包含的信息决定当前时间。每个事件时间都带有一个时间戳，而**系统的逻辑时间是由水位线来定义**。 
- Ingestion Time：是数据进入 Flink 的时间。指定每个接收的记录都把在数据源算子的处理时间作为事件时间的时间戳，并自动生成水位线。  
- Processing Time：是每一个执行基于时间操作的算子的**本地系统时间**，与机器相关，**默认的时间属性就是 Processing Time**。 指定算子根据处理机器的系统时钟决定数据流当前的时间。 处理时间窗口基于机器时间触发，它可以涵盖触发时间点之前到达算子的任意元素 

例如，一条日志进入 Flink 的时间为 2017-11-12 10:00:00.123，到达 Window 的系统时间为 2017-11-12 10:00:01.234，日志的内容如下： 

```java
2017-11-02 18:37:15.624 INFO Fail over to rm2
```

对于业务来说，要统计 1min 内的故障日志个数，哪个时间是最有意义的？ ——eventTime，因为我们要根据日志的生成时间进行统计。 

### EventTime 的引入 

在 Flink 的流式处理中，绝大部分的业务都会使用 eventTime，一般只在eventTime 无法使用时，才会被迫使用 ProcessingTime 或者 IngestionTime。

如果要使用 EventTime，那么需要引入 EventTime 的时间属性，引入方式如下所示：  

```java
public class CountWindow_eventtime {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        设置时间语义,事件时间语义
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.execute();
    }
}
```

### Watermark 

水位线用于告知算子不必再等那些时间戳小于或等于水位线的事件 。水位线的等待是什么意思，就是直接把时间调的慢多少分钟，这样就相当于等待了若干分钟。

![1622882370631](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/163935-652122.png)

**一个 watermark 本质上就代表了这个 watermark 所包含的 timestamp 数值，表示以后到来的数据已经再也没有小于或等于这个时间的了。**

#### 水位线（Watermark）

- 怎样避免乱序数据带来计算不正确？
- 遇到一个时间戳达到了窗口关闭时间，不应该立刻触发窗口计算，而是等待一段时间，等迟到的数据来了再关闭窗口
- Watermark是一种衡量Event Time进展的机制，可以设定延迟触发
- **Watermark是用于处理乱序事件**的，而正确的处理乱序事件，通常用Watermark机制结合window来实现；
- 数据流中的Watermark用于表示timestamp小于Watermark的数据，都已经到达了，因此，window的执行也是由Watermark触发的。
- watermark用来让程序自己平衡延迟和结果正确性

**作用**

![1622882548416](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/164231-609979.png)

而watermark的时间是根据事件事件来计算的，这样就可以根据事件时间来触发窗口的计算。

#### Timestamp 分配和 Watermark 生成

Flink 支持两种 watermark 生成方式。第一种是在 SourceFunction 中产生，相当于把整个的 timestamp 分配和 watermark 生成的逻辑放在流处理应用的源头。我们可以在 SourceFunction 里面通过这两个方法产生 watermark：

- 通过 collectWithTimestamp 方法发送一条数据，其中第一个参数就是我们要发送的数据，第二个参数就是这个数据所对应的时间戳；也可以调用 emitWatermark 去产生一条 watermark，表示接下来不会再有时间戳小于等于这个数值记录。
- 另外，有时候我们不想在 SourceFunction 里生成 timestamp 或者 watermark，或者说使用的 SourceFunction 本身不支持，我们还可以在使用 DataStream API 的时候指定，调用的 DataStream.assignTimestampsAndWatermarks 这个方法，能够接收不同的 timestamp 和 watermark 的生成器。

![1638454301295](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/221148-991928.png)

两者的区别主要有三个方面，首先定期生成是现实时间驱动的，这里的“定期生成”主要是指 watermark（因为 timestamp 是每一条数据都需要有的），即定期会调用生成逻辑去产生一个 watermark。而根据特殊记录生成是数据驱动的，即是否生成 watermark 不是由现实时间来决定，而是当看到一些特殊的记录就表示接下来可能不会有符合条件的数据再发过来了，这个时候相当于每一次分配 Timestamp 之后都会调用用户实现的 watermark 生成方法，用户需要在生成方法中去实现 watermark 的生成逻辑。

大家要注意的是就是我们在分配 timestamp 和生成 watermark 的过程，虽然在 SourceFunction 和 DataStream 中都可以指定，但是还是建议生成的工作越靠近 DataSource 越好。这样会方便让程序逻辑里面更多的 operator 去判断某些数据是否乱序。**Flink 内部提供了很好的机制去保证这些 timestamp 和 watermark 被正确地传递到下游的节点**。

**总体上而言生成器可以分为两类：**第一类是定期生成器；第二类是根据一些在流处理数据流中遇到的一些特殊记录生成的。

#### 基本概念

我们知道，流处理从事件产生，到流经 source，再到 operator，中间是有一个过程和时间的，虽然大部分情况下，流到 operator 的数据都是按照事件产生的时间顺序来的，但是也不排除由于网络、分布式等原因，导致乱序的产生，所谓乱序，就是指 Flink 接收到的事件的先后顺序不是严格按照事件的 Event Time 顺序排的。 

![1614655834182](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/19/092749-699206.png)

在这里设置的水位线相当于**时间调慢了一段时间**，对于乱序到来的数据，比如对于事件时间是5的数据，设置延迟是3秒，那么事件时间是5的数据到来是只相当于当前进展到5-3=2秒，事件时间是6秒的数据到来时候只相当于当前进展到3秒。也就是对每一条到来的数据，统一延迟3秒钟。当事件时间是8的时间到来时候，8-3=5，此时说明前面0-5秒的数据已经全部到来，可以关闭窗口，因为这里设置窗口的大小是5。

watermark是用来处理乱序事件的，waterMark一般设置一个相对较小的统一的延迟时间。

那么此时出现一个问题，一旦出现乱序，如果只根据 eventTime 决定 window 的运行，我们不能明确数据是否全部到位，但又不能无限期的等下去，此时必须要有个机制来保证一个特定的时间后，必须触发 window 去进行计算了，这个特别的机制，就是 Watermark。 

- Watermark 是一种衡量 Event Time 进展的机制 
- Watermark 是用于处理乱序事件的，而正确的处理乱序事件，通常用Watermark 机制结合 window 来实现。 
- 数据流中的 Watermark 用于表示 timestamp 小于 Watermark 的数据，都已经到达了（如何理解这句话，比如说watermark允许延迟的时间是3，那么事件时间为5的数据到达时候，那么当前的watermarks是2，说明事件时间为2的数据以及之前的数据已经到齐了），因此， window 的执行也是由 Watermark 触发的。 
- **Watermark 可以理解成一个延迟触发机制，我们可以设置 Watermark 的延时时长 t，每次系统会校验已经到达的数据中最大的 maxEventTime，然后认定 eventTime小于 maxEventTime - t 的所有数据都已经到达，如果有窗口的停止时间等于maxEventTime – t，那么这个窗口被触发执行。** 

有序流的 Watermarker 如下图所示：（ Watermark 设置为 0） 

![1614656042731](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/214126-9523.png)

乱序流的 Watermarker 如下图所示：（ Watermark 设置为 2） 

![1614656067281](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/113429-644739.png)

当 Flink 接收到数据时， 会按照一定的规则去生成 Watermark，这条 Watermark就等于当前所有到达数据中的 **maxEventTime - 延迟时长**，也就是说， **Watermark 是基于数据携带的时间戳生成的，一旦 Watermark 比当前未触发的窗口的停止时间要晚，那么就会触发相应窗口的执行**。由于 event time 是由数据携带的，因此，如果运行过程中无法获取新的数据，那么没有被触发的窗口将永远都不被触发。 

上图中，我们设置的允许最大延迟到达时间为 2s，所以时间戳为 7s 的事件对应的 Watermark 是 5s，时间戳为 12s 的事件的 Watermark 是 10s，如果我们的窗口 1是 1s~5s，窗口 2 是 6s~10s，那么时间戳为 7s 的事件到达时的 Watermarker 恰好触发窗口 1，时间戳为 12s 的事件到达时的 Watermark 恰好触发窗口 2。

**Watermark 就是触发前一窗口的“关窗时间”，一旦触发关门那么以当前时刻为准在窗口范围内的所有所有数据都会收入窗中。**
只要没有达到水位那么不管现实中的时间推进了多久都不会触发关窗。 

![1622889433596](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/183717-675917.png)

watermark是一直单调递增的序列，当waternark值大于等于窗口结束时间时候，会触发计算。

#### watermark详解

![1623129086219](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/131127-526004.png)![1623129595750](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/131957-442663.png)

**如果数据延迟太严重，比如上面案例中，A事件在事件D触发窗口之前还没有到来，那么就只能丢失A数据了，所以说watermark机制不能完全避免数据的不丢失，只能说是避免，但是如果数据延迟太严重，那么解决的方案是使用测输出流配合watermark机制。**

#### watermark的特点

![1614659939633](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/101819-143027.png)

watermark=2插入数据流，说明事件时间为2的数据已经全部到达，接下来、有事件时间为5的数据带来，所以插入watermark=5，也就是说事件时间为5以及之前的数据已经全部到达，但是后来又有事件时间为3的数据到达，但是现在watermark是5，说明事件时间是3的数据已经迟到。watermark的时间是根据数据的事件时间产生的。

- watermark是一条特殊的数据记录，本质上可以看作是一个时间戳。

```java
@PublicEvolving
public final class Watermark extends StreamElement {

	/** The watermark that signifies end-of-event-time. */
	public static final Watermark MAX_WATERMARK = new Watermark(Long.MAX_VALUE);

	// ------------------------------------------------------------------------

	/** The timestamp of the watermark in milliseconds. */
	private final long timestamp;  //watermark其实就是一个带着时间戳的数据，可以直接插入数据流中，代表时间的推移

	/**
	 * Creates a new watermark with the given timestamp in milliseconds.
	 */
	public Watermark(long timestamp) {
		this.timestamp = timestamp;
	}

	/**
	 * Returns the timestamp associated with this {@link Watermark} in milliseconds.
	 */
	public long getTimestamp() {
		return timestamp;
	}

	// ------------------------------------------------------------------------

	@Override
	public boolean equals(Object o) {
		return this == o ||
				o != null && o.getClass() == Watermark.class && ((Watermark) o).timestamp == this.timestamp;
	}

	@Override
	public int hashCode() {
		return (int) (timestamp ^ (timestamp >>> 32));
	}

	@Override
	public String toString() {
		return "Watermark @ " + timestamp;
	}
}
```

- watermark必须单调递增，以确保任务的事件时间时钟在向前推进，而不是在后退
- watermark与事件事件相关。

watermart就代表一个含义，其之前的数据已经全部到齐了，所以watermark的设置要保证其前面的数据全部到齐，假如说当前设置的延迟时间戳是s秒，**延迟时间的设置主要看最大的迟到程度**，每次使用当前到来的数据的时间戳减去延迟时间进行比较，如果结果小于零，或者小于当前的watermark的值，那么watermark还使用之前的值，否则人进行更新操作，那么窗口什么时候关闭呢？一直更新watermark直到其值等于窗口大小就关闭一次窗口操作，根据watermark的值判断是否需要关闭窗口。如果窗口关闭之后，还有迟到的数据，这个时候可以输出到测输出流中。

watermark表示的数据到齐与否是左闭右开的，设置延迟的时间应该根据最大的迟到时间差来。watermark的设置是当前数据的事件时间-设置的延迟时间

下面的图，同时可以存在多个桶，然后依次把数据分配到桶中，触发桶的执行是根据watermark进行的，当事件时间为8的数据到达后，8-3=5，此时说明前5秒的数据已经全部到达，注意这里不包含事件时间为5秒的数据，此时第一个桶，也就是1234所在的桶被触发执行。

时间语义的三种保证：

- watermark。
- `.allowedLateness(Time.seconds(1))`允许迟到一段时间
- 使用侧输出流进行输出。

#### watermark的传递

上面考虑的是一条流，相当于一个分区，对于多个任务之间，水位线要以**最小的**那个值为准

![1614661794820](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/135527-780703.png)

![1638454379583](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/221302-166640.png)

具体的传播策略基本上遵循这三点。

- **首先**，watermark 会以广播的形式在算子之间进行传播。比如说上游的算子连接了三个下游的任务，它会把自己当前的收到的 watermark 以广播的形式传到下游。
- **第二**，如果在程序里面收到了一个 Long.MAX_VALUE 这个数值的 watermark，就表示对应的那一条流的一个部分不会再有数据发过来了，它相当于就是一个终止的标志。
- **第三**，对于单流而言，这个策略比较好理解，而**对于有多个输入的算子，watermark 的计算就有讲究了，一个原则是：****单输入取其大，多输入取小**。

举个例子，假设这边蓝色的块代表一个算子的一个任务，然后它有三个输入，分别是 W1、W2、W3，这三个输入可以理解成任何输入，这三个输入可能是属于同一个流，也可能是属于不同的流。然后在计算 watermark 的时候，**对于单个输入而言是取他们的最大值，因为我们都知道 watermark 应该遵循一个单调递增的一个原则。对于多输入，它要统计整个算子任务的 watermark 时，就会取这三个计算出来的 watermark 的最小值。即一个多个输入的任务，它的 watermark 受制于最慢的那条输入流。**

这一点类似于木桶效应，整个木桶中装的水会受制于最矮的那块板。

watermark 在传播的时候有一个特点是，它的传播是幂等的。多次收到相同的 watermark，甚至收到之前的 watermark 都不会对最后的数值产生影响，因为对于单个输入永远是取最大的，而对于整个任务永远是取一个最小的。

同时我们可以注意到这种设计其实有一个局限，具体体现在它没有区分你这个输入是一条流多个 partition 还是来自于不同的逻辑上的流的 JOIN。对于同一个流的不同 partition，我们对他做这种强制的时钟同步是没有问题的，因为一开始就把一条流拆散成不同的部分，但每一个部分之间共享相同的时钟。

但是如果算子的任务是在做类似于 JOIN 操作，那么要求两个输入的时钟强制同步其实没有什么道理的，因为完全有可能是把一条离现在时间很近的数据流和一个离当前时间很远的数据流进行 JOIN，这个时候对于快的那条流，因为它要等慢的那条流，所以说它可能就要在状态中去缓存非常多的数据，这对于整个集群来说是一个很大的性能开销。

#### ProcessFunction

简单介绍 ProcessFunction，因为 **watermark 在任务里的处理逻辑分为内部逻辑和外部逻辑**。外部逻辑其实就是通过 ProcessFunction 来体现的，如果你需要使用 Flink 提供的时间相关的 API 的话就只能写在 ProcessFunction 里。

ProcessFunction 和时间相关的功能主要有三点：

- **第一点**，根据你当前系统使用的时间语义不同，你可以去获取当前你正在处理这条记录的 Record Timestamp，或者当前的 Processing Time。
- **第二点**，它可以获取当前算子的时间，可以把它理解成当前的 watermark。
- **第三点**，为了在 ProcessFunction 中去实现一些相对复杂的功能，允许注册一些 timer（定时器）。比如说在 watermark 达到某一个时间点的时候就触发定时器，所有的这些回调逻辑也都是由用户来提供，涉及到如下三个方法，registerEventTimeTimer、registerProcessingTimeTimer 和 onTimer。在 onTimer 方法中就需要去实现自己的回调逻辑，当条件满足时回调逻辑就会被触发。

一个简单的应用是，我们在做一些时间相关的处理的时候，可能需要缓存一部分数据，但这些数据不能一直去缓存下去，所以需要有一些过期的机制，我们可以通过 timer 去设定这么一个时间，指定某一些数据可能在将来的某一个时间点过期，从而把它从状态里删除掉。所有的这些和时间相关的逻辑在 Flink 内部都是由自己的 Time Service（时间服务）完成的。

#### Watermark 处理

![1638454623115](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/221709-520418.png)

一个算子的实例在收到 watermark 的时候，**首先要更新当前的算子时间**，这样的话在 ProcessFunction 里方法查询这个算子时间的时候，就能获取到最新的时间。**第二步它会遍历计时器队列**，这个计时器队列就是我们刚刚说到的 timer，你可以同时注册很多 timer，Flink 会把这些 Timer 按照触发时间放到一个优先队列中。**第三步 Flink 得到一个时间之后就会遍历计时器的队列，然后逐一触发用户的回调逻辑**。 通过这种方式，Flink 的某一个任务就会将当前的 watermark 发送到下游的其他任务实例上，从而完成整个 watermark 的传播，从而形成一个闭环。

#### Watermark 的引入 

![1614683588914](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/191311-558140.png)

Event Time的使用一定要指定数据源中的时间戳调用assignTimestampAndWatermarks方法，传入一个BoundedOutOfOrdernessTimestampExtractor，就可以指定watermark，在底层提取时间要求的是毫秒数。

watermark 的引入很简单，对于乱序数据，最常见的引用方式如下： 

```java
//                这种方式是周期性生成watermart,也可以随机生成
//BoundedOutOfOrdernessTimestampExtractor这个类的作用是提取时间戳
                .assignTimestampsAndWatermarks(new 
                   //设置watermark的延迟时间是2秒，表示最大的乱序程度                            BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(2)) {

//            提取每一个数据的时间戳
            @Override
            public long extractTimestamp(SensorReading element) {
//                返回的时间戳是毫秒数
                return element.getTempStamp()*1000l;
            }
        });

//                如果数据已经知道有序，没有乱序发生，就不用设置延迟，不用设置延迟时间，不需要延迟触发，可以只指定时间戳就行了
//                .assignTimestampsAndWatermarks(new AscendingTimestampExtractor<SensorReading>() {
//                    @Override
//                    public long extractAscendingTimestamp(SensorReading element) {
//                        return element.getTempStamp()*1000l;
//                    }
//                })
```

**时间戳接口**

```java
public interface AssignerWithPeriodicWatermarks<T> extends TimestampAssigner<T>{}
public interface AssignerWithPunctuatedWatermarks<T> extends TimestampAssigner<T> {}

//TimestampAssigner接口中必须指明如何提取时间戳
public interface TimestampAssigner<T> extends Function {

	/**
	 * Assigns a timestamp to an element, in milliseconds since the Epoch.
	 *
	 * <p>The method is passed the previously assigned timestamp of the element.
	 * That previous timestamp may have been assigned from a previous assigner,
	 * by ingestion time. If the element did not carry a timestamp before, this value is
	 * {@code Long.MIN_VALUE}.
	 *
	 * @param element The element that the timestamp will be assigned to.
	 * @param previousElementTimestamp The previous internal timestamp of the element,
	 *                                 or a negative value, if no timestamp has been assigned yet.
	 * @return The new timestamp.
	 */
	long extractTimestamp(T element, long previousElementTimestamp);
}


```

![1614662515762](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/215326-650305.png)

```java
//BoundedOutOfOrdernessTimestampExtractor其实就是一个周期性生成时间戳的类
public abstract class BoundedOutOfOrdernessTimestampExtractor<T> implements AssignerWithPeriodicWatermarks<T> {

	private static final long serialVersionUID = 1L;

	/** The current maximum timestamp seen so far. */
  当前最大的时间戳
	private long currentMaxTimestamp;

	/** The timestamp of the last emitted watermark. */
  上一次发出的时间戳
	private long lastEmittedWatermark = Long.MIN_VALUE;

	/**
	 * The (fixed) interval between the maximum seen timestamp seen in the records
	 * and that of the watermark to be emitted.
	 */
  最大的延迟时间，也就是waternark的延迟时间 
	private final long maxOutOfOrderness;

	public BoundedOutOfOrdernessTimestampExtractor(Time maxOutOfOrderness) {
		if (maxOutOfOrderness.toMilliseconds() < 0) {
			throw new RuntimeException("Tried to set the maximum allowed " +
				"lateness to " + maxOutOfOrderness + ". This parameter cannot be negative.");
		}
    设置最大延迟时间
		this.maxOutOfOrderness = maxOutOfOrderness.toMilliseconds();
    最大的时间戳，加上延迟时间是因为后面还要减去最大的时间戳保证数据不会溢出
		this.currentMaxTimestamp = Long.MIN_VALUE + this.maxOutOfOrderness;
	}

	public long getMaxOutOfOrdernessInMillis() {
		return maxOutOfOrderness;
	}

	/**
	 * Extracts the timestamp from the given element.
	 *
	 * @param element The element that the timestamp is extracted from.
	 * @return The new timestamp.
	 */
	public abstract long extractTimestamp(T element);

	@Override
	public final Watermark getCurrentWatermark() {
		// this guarantees that the watermark never goes backwards.
    潜在的watermark=最大的时间戳减去最大的延迟时间
		long potentialWM = currentMaxTimestamp - maxOutOfOrderness;
    判断潜在的watermark是否大于上一次发出去的watermark，因为watermark是递增的一个时间
		if (potentialWM >= lastEmittedWatermark) {
      更新watermark时间
			lastEmittedWatermark = potentialWM;
		}
		return new Watermark(lastEmittedWatermark);
	}
//提取时间戳的方法
	@Override
	public final long extractTimestamp(T element, long previousElementTimestamp) {
		long timestamp = extractTimestamp(element);
		if (timestamp > currentMaxTimestamp) {
			currentMaxTimestamp = timestamp;
		}
		return timestamp;
	}
}
AscendingTimestampExtractor
相当于仅仅延迟一毫秒时间
@Override
	public final Watermark getCurrentWatermark() {
		return new Watermark(currentTimestamp == Long.MIN_VALUE ? Long.MIN_VALUE : currentTimestamp - 1);
	}
```

`AscendingTimestampExtractor`也是周期性生成

![1614662990403](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/133541-295517.png)

`AssignerWithPunctuatedWatermarks`没有被实现

![1614663059985](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/133103-949255.png)

Event Time 的使用一定要指定数据源中的时间戳。否则程序无法知道事件的事件时间是什么(数据源里的数据没有时间戳的话，就只能使用 Processing Time 了)。 

我们看到上面的例子中创建了一个看起来有点复杂的类，这个类实现的其实就是分配时间戳的接口。 Flink 暴露了 TimestampAssigner 接口供我们实现，使我们可以自定义如何从事件数据中抽取时间戳。

```java
StreamExecutionEnvironment env =
StreamExecutionEnvironment.getExecutionEnvironment();
// 设置事件时间语义
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
DataStream<SensorReading> dataStream = env.addSource(new SensorSource())
.assignTimestampsAndWatermarks(new MyAssigner())
```

MyAssigner 有两种类型，都是继承 TimestampAssigner 

- AssignerWithPeriodicWatermarks
  - 周期性的生成watermark：系统会周期性的将watermark插入到流中
  - 默认周期是200毫秒，可以使用ExecutionConfig.setAutoWatermarkInterval()方法进行设置
  - 升序和前面乱序的处理BoundedOutOfOrdernessTimestampExtractor，都是基于周期性watermark的。
- AssignerWithPunctuatedWatermarks
  - 没有时间周期规律，可打断的生成watermark

以上两个接口都继承自 TimestampAssigner。 

##### Assigner with periodic watermarks 

周期性的生成 watermark：系统会周期性的将 watermark 插入到流中(水位线也是一种特殊的事件!)。默认周期是 200 毫秒。可以使用ExecutionConfig.setAutoWatermarkInterval()方法进行设置。 

```java
// 每隔 5 秒产生一个 watermark
env.getConfig.setAutoWatermarkInterval(5000);
```

产生 watermark 的逻辑：每隔 5 秒钟， Flink 会调用AssignerWithPeriodicWatermarks 的 getCurrentWatermark()方法。如果方法返回一个 时间戳大于之前水位的时间戳，新的 watermark 会被插入到流中。这个检查保证了水位线是单调递增的。如果方法返回的时间戳小于等于之前水位的时间戳，则不会产生新的 watermark。 

例子，自定义一个周期性的时间戳抽取： 

```java
// 自定义周期性时间戳分配器
public static class MyPeriodicAssigner implements
        AssignerWithPeriodicWatermarks<SensorReading>{
    private Long bound = 60 * 1000L; // 延迟一分钟
    private Long maxTs = Long.MIN_VALUE; // 当前最大时间戳
    @Nullable
    @Override
    public Watermark getCurrentWatermark() {
        return new Watermark(maxTs - bound);
    } @
            Override
    public long extractTimestamp(SensorReading element, long previousElementTimestamp)
    {
        maxTs = Math.max(maxTs, element.getTimestamp());
        return element.getTimestamp();
    }
}
```

**一种简单的特殊情况是， 如果我们事先得知数据流的时间戳是单调递增的，也就是说没有乱序， 那我们可以使用 AscendingTimestampExtractor， 这个类会直接使用数据中的事件时间戳生成 watermark。**

```java
DataStream<SensorReading> dataStream = …
        dataStream.assignTimestampsAndWatermarks(
        new AscendingTimestampExtractor<SensorReading>() {
@Override
public long extractAscendingTimestamp(SensorReading element) {
        return element.getTimestamp() * 1000;
        }
});
```

而对于乱序数据流， 如果我们能大致估算出数据流中的事件的最大延迟时间，就可以使用如下代码： 

```java
DataStream<SensorReading> dataStream = …
dataStream.assignTimestampsAndWatermarks(
  //       水位线BoundedOutOfOrdernessTimestampExtractor参数是最大的延迟时间
new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(1)) {
@Override
public long extractTimestamp(SensorReading element) {
return element.getTimestamp() * 1000L;
}
});
```

##### Assigner with punctuated watermarks 

间断式地生成 watermark。和周期性生成的方式不同，**这种方式不是固定时间的，而是可以根据需要对每条数据进行筛选和处理**。 直接上代码来举个例子， 我们只给sensor_1 的传感器的数据流插入 watermark： 

```java
public static class MyPunctuatedAssigner implements
        AssignerWithPunctuatedWatermarks<SensorReading>{
    private Long bound = 60 * 1000L; // 延迟一分钟
    @Nullable
    @Override
    public Watermark checkAndGetNextWatermark(SensorReading lastElement, long
            extractedTimestamp) {
        if(lastElement.getId().equals("sensor_1"))
            return new Watermark(extractedTimestamp - bound);
        else
            return null;
    }
    @Override
    public long extractTimestamp(SensorReading element, long previousElementTimestamp)
    {
        return element.getTimestamp();
    }
}
```

**对于迟到的数据，写入到测输出流中**

```java
public class CountWindow_eventtime_ {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        //env.setParallelism(1);

//        设置时间语义,事件时间语义
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.getConfig().setAutoWatermarkInterval(100);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//       水位线BoundedOutOfOrdernessTimestampExtractor参数是最大的延迟时间
        map
//                这种方式是周期性生成watermart,也可以随机生成
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(2)) {

//            提取每一个数据的时间戳
            @Override
            public long extractTimestamp(SensorReading element) {
//                返回的时间戳是毫秒数
                return element.getTempStamp()*1000l;
            }
        });


        OutputTag<SensorReading> late = new OutputTag<>("late");
//        基于事件时间的开窗工作
//        统计3秒内温度的最小值
        SingleOutputStreamOperator<SensorReading> minTemp = map.keyBy("id")
                .timeWindow(Time.seconds(2))//2秒滚动一个窗口
                .allowedLateness(Time.seconds(5))//允许迟到的时间
                .sideOutputLateData(late)//迟到的数据，放进测输出流中
                .minBy("temperature");

        minTemp.print();

//        输出测输出流中的数据
        minTemp.getSideOutput(late).print("late");

        env.execute();
    }

}
```

#### watermark的设定

- **在Flink中，watermark由应用程序开发人员生成，这通常需要对相应的领域有一定的了解**。
- 如果watermark设置的延迟太久，收到结果的速度可能就会很慢，解决办法是在水位线到达之前输出一个近似结果
- 而如果watermark到达得太早，则可能收到错误结果，不过Flink处理迟到数据的机制可以解决这个问题

> 总的来说，延迟时间太长，收到数据的速度会很慢，如果延迟时间太短，可能发生数据的丢失。

**周期性生成中周期的设定**

```java
@PublicEvolving
	public void setStreamTimeCharacteristic(TimeCharacteristic characteristic) {
		this.timeCharacteristic = Preconditions.checkNotNull(characteristic);
		if (characteristic == TimeCharacteristic.ProcessingTime) {
      //如果是处理时间，设置时间是0
			getConfig().setAutoWatermarkInterval(0);
		} else {
      //事件时间设置的周期是200毫秒
			getConfig().setAutoWatermarkInterval(200);
		}
	}

//初始窗口的设定
@Override
	public Collection<TimeWindow> assignWindows(Object element, long timestamp, WindowAssignerContext context) {
		if (timestamp > Long.MIN_VALUE) {
			// Long.MIN_VALUE is currently assigned when no timestamp is present
			long start = TimeWindow.getWindowStartWithOffset(timestamp, offset, size);
			return Collections.singletonList(new TimeWindow(start, start + size));
		} else {
			throw new RuntimeException("Record has Long.MIN_VALUE timestamp (= no timestamp marker). " +
					"Is the time characteristic set to 'ProcessingTime', or did you forget to call " +
					"'DataStream.assignTimestampsAndWatermarks(...)'?");
		}
	}

/**
	 * Method to get the window start for a timestamp.
	 *
	 * @param timestamp epoch millisecond to get the window start.
	 * @param offset The offset which window start would be shifted by.
	 * @param windowSize The size of the generated windows.
	 * @return window start
	 */
offset是一个偏移量，也就是起始点的位置时间发生一点偏移
	public static long getWindowStartWithOffset(long timestamp, long offset, long windowSize) {
		return timestamp - (timestamp - offset + windowSize) % windowSize;
	}
也就是使用初始的时间戳-初试时间戳对窗口的大小取模，第一个窗口确定，后面所有窗口都确定

//在watermark之后设置数据可以迟到多长时间
.allowedLateness(Time.seconds(1))
  
//三条线保证数据不会丢失
   .keyBy(ApacheLogEvent::getUrl)//按照url进行分组操作
//                窗口的大小是5，窗口滑动的步长是10
                .timeWindow(Time.minutes(5), Time.minutes(10))
//                在窗口中设置允许迟到数据,上面设置watermark可以等待一秒，在窗口这里在允许等待一分钟
                .allowedLateness(Time.seconds(1))
//                在这里设置侧输出流
                .sideOutputLateData(outputTag)
```

#### 并行度问题

假如说输入的数据流有多条，也就是有多个source，那么多条数据流的watermark之间不会相互影响，但是如果只有一条输入流，但是每一个算子的并行度不同，那么数据在多个任务之间的传输，watermark会遵循木桶原则，也就是根据最小的waternark值设置自己的watermark的值。

![1614737979924](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/101942-178134.png)

在map和kb之间，map的并行度是4，map之后的每一个任务都会广播自己的watermark值给后边的kb任务，kb任务会选取所有的watermark的最小值来设置自己的watermark值。

```java
public class CountWindow_eventtime_ {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1，并行度不是1的情况
        //env.setParallelism(1);

//        设置时间语义,事件时间语义
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.getConfig().setAutoWatermarkInterval(100);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//       水位线BoundedOutOfOrdernessTimestampExtractor参数是最大的延迟时间
        map
//                这种方式是周期性生成watermart,也可以随机生成
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(2)) {

//            提取每一个数据的时间戳
            @Override
            public long extractTimestamp(SensorReading element) {
//                返回的时间戳是毫秒数
                return element.getTempStamp()*1000l;
            }
        });


        OutputTag<SensorReading> late = new OutputTag<>("late");
//        基于事件时间的开窗工作
//        统计3秒内温度的最小值
        SingleOutputStreamOperator<SensorReading> minTemp = map.keyBy("id")
                .timeWindow(Time.seconds(2))//2秒滚动一个窗口
                .allowedLateness(Time.seconds(5))//允许迟到的时间
                .sideOutputLateData(late)//迟到的数据，放进测输出流中
                .minBy("temperature");

        minTemp.print();

//        输出测输出流中的数据
        minTemp.getSideOutput(late).print("late");

        env.execute();
    }

}
```

#### 案例

![1623131796430](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/144830-386195.png)

```java
public class Test20 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
//        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        DataStreamSource<Order> orderDataStreamSource = env.addSource(new SourceFunction<Order>() {
            private boolean flag=true;
            @Override
            public void run(SourceContext<Order> ctx) throws Exception {
                Random random = new Random();
                while (flag){
//                    产生订单的id
                    String id = UUID.randomUUID().toString();
//                    产生用户id
                    int userId = random.nextInt(2);
                    int money = random.nextInt(101);//0--100
//                    产生随机的延迟时间
                    long evenTime=System.currentTimeMillis()-random.nextInt(5)* 1000;
                    ctx.collect(new Order(id,userId,money,evenTime));
                    Thread.sleep(2000);
                }
            }
            @Override
            public void cancel() {
                flag=false;
            }
        });

//        基于事件时间的窗口计算+watermark
//        求每一个用户订单的总金额，最近5秒钟
//        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
//        在新版本中默认使用的是事件时间
//        设置水位线
        SingleOutputStreamOperator<Order> orderSingleOutputStreamOperator = orderDataStreamSource.assignTimestampsAndWatermarks(WatermarkStrategy.
                <Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))//设置延迟时间，也即是最大的乱序事件
                .withTimestampAssigner((order, timestamp) -> order.evevTime));//设置事件时间


        SingleOutputStreamOperator<Order> money = orderSingleOutputStreamOperator.keyBy(order -> order.getOrderId())
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .sum("money");

        money.print();
        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class Order {
        private String orderId;
        private Integer userId;
        private Integer money;
        private Long evevTime;
    }

}
```

#### 源码说明

```java
SingleOutputStreamOperator<Order> orderSingleOutputStreamOperator = orderDataStreamSource.assignTimestampsAndWatermarks(WatermarkStrategy.
                <Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))//设置延迟时间，也即是最大的乱序事件
                .withTimestampAssigner((order, timestamp) -> order.evevTime));//设置事件时间

static <T> WatermarkStrategy<T> forBoundedOutOfOrderness(Duration maxOutOfOrderness) {
		return (ctx) -> new BoundedOutOfOrdernessWatermarks<>(maxOutOfOrderness);
	}

public BoundedOutOfOrdernessWatermarks(Duration maxOutOfOrderness) {
		checkNotNull(maxOutOfOrderness, "maxOutOfOrderness");
		checkArgument(!maxOutOfOrderness.isNegative(), "maxOutOfOrderness cannot be negative");

		this.outOfOrdernessMillis = maxOutOfOrderness.toMillis();

		// start so that our lowest watermark would be Long.MIN_VALUE.
  //时间戳的初始值
		this.maxTimestamp = Long.MIN_VALUE + outOfOrdernessMillis + 1;
	}

@Override
	public void onEvent(T event, long eventTimestamp, WatermarkOutput output) {
   //获取当前进来的数据的最大时间戳
		maxTimestamp = Math.max(maxTimestamp, eventTimestamp);
	}

@Override
	public void onPeriodicEmit(WatermarkOutput output) {
    //当前最大时间戳-允许延迟的时间
		output.emitWatermark(new Watermark(maxTimestamp - outOfOrdernessMillis - 1));
	}
```

#### 数据丢失问题

![1623132921218](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/215931-144166.png)

在这个例子中，在D数据到来时候，会触发计算，所以会导致数据A丢失问题，所以需要引入侧输出 流机制。

![1623133116787](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/144837-194982.png)

```java
public class Test21 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
//        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        DataStreamSource<Order> orderDataStreamSource = env.addSource(new SourceFunction<Order>() {
            private boolean flag=true;
            @Override
            public void run(SourceContext<Order> ctx) throws Exception {
                Random random = new Random();
                while (flag){
//                    产生订单的id
                    String id = UUID.randomUUID().toString();
//                    产生用户id
                    int userId = random.nextInt(2);
                    int money = random.nextInt(101);//0--100
//                    产生随机的延迟时间,有可能导致数据丢失
                    long evenTime=System.currentTimeMillis()-random.nextInt(5)* 1000;
                    ctx.collect(new Order(id,userId,money,evenTime));
                    Thread.sleep(2000);
                }
            }
            @Override
            public void cancel() {
                flag=false;
            }
        });

//        用来存放迟到的数据
        OutputTag<Order> tag = new OutputTag<>("data", TypeInformation.of(Order.class));

//        基于事件时间的窗口计算+watermark
//        求每一个用户订单的总金额，最近5秒钟
//        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
//        在新版本中默认使用的是事件时间
//        设置水位线
        SingleOutputStreamOperator<Order> orderSingleOutputStreamOperator = orderDataStreamSource.assignTimestampsAndWatermarks(WatermarkStrategy.
                <Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))//设置延迟时间，也即是最大的乱序事件
                .withTimestampAssigner((order, timestamp) -> order.evevTime));//设置事件时间


        SingleOutputStreamOperator<Order> money = orderSingleOutputStreamOperator.keyBy(order -> order.getOrderId())
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .allowedLateness(Time.seconds(3))//允许延迟
                .sideOutputLateData(tag)//延迟的数据存放的位置
                .sum("money");

//        打印正常到来的数据
        money.print();
//        打印侧输出流中的结果
        System.out.println(money.getSideOutput(tag));
        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class Order {
        private String orderId;
        private Integer userId;
        private Integer money;
        private Long evevTime;
    }
}
```

### 小结

总的来说，使用watermark机制并不能保证数据完全不丢失，对于延迟很高的数据，依然会发生数据的丢失，那么此时可以允许发生数据的延迟，设定一个延迟的时间，然后使用测输出流来收集迟到的数据。

- watermark。
- `.allowedLateness(Time.seconds(1))`允许迟到一段时间，使用侧输出流进行输出。

