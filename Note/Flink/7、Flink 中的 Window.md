## Flink 中的 Window 
<!-- TOC -->

- [Flink 中的 Window](#flink-中的-window)
  - [为什么需要Window](#为什么需要window)
  - [Window的分类](#window的分类)
    - [按照time和count分类](#按照time和count分类)
    - [按照slide和size分类](#按照slide和size分类)
  - [Window Api](#window-api)
    - [总体概况](#总体概况)
    - [API类图](#api类图)
    - [窗口的创建](#窗口的创建)
      - [窗口分配器（window assigner）](#窗口分配器window-assigner)
      - [创建不同类型的窗口](#创建不同类型的窗口)
      - [窗口函数（window function）](#窗口函数window-function)
        - [增量聚合函数](#增量聚合函数)
        - [全窗口函数](#全窗口函数)
        - [计数窗口](#计数窗口)
      - [其他API](#其他api)
      - [案例](#案例)
      - [Window API总览](#window-api总览)

<!-- /TOC -->


- 窗口是流式应用中一类十分常见的操作。它们可以在无限数据流上基于**有界区间**实现聚合等转换。通常情况下，这些区间都是基于时间逻辑定义的。窗口算子提供了一种基于有限大小的桶对事件进行分组，并对这些桶中的有限内容进行计算的方法。 
- streaming 流式计算是一种被设计用于处理无限数据集的数据处理引擎，而无限数据集是指一种不断增长的本质上无限的数据集，而**window是一种切割无限数据为有限块进行处理的手段**。Window 是无限数据流处理的核心，Window将一个无限的stream 拆分成有限大小的” buckets”桶，我们可以在这些桶上做计算操作。 
- **窗口算子可用在键值分区或非键值分区的数据流上。用于键值分区窗口的算子可以并行计算，而非键值分区窗口只能单线程处理。**

### 为什么需要Window

在流处理应用中，数据是连续不断的，有时我们需要做一些聚合类的处理，例如：在过去的1分钟内有多少用户点击了我们的网页。在这种情况下，我们必须定义一个窗口(window)，用来收集最近1分钟内的数据，并对这个窗口内的数据进行计算。

![1614567122089](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/105209-808469.png)

- 一般真实的流都是无界的，怎样处理无界的数据？
- 可以把无限的数据流进行切分，得到有限的数据集进行处理——也就是得到有界流


**窗口（window）就是将无限流切割为有限流的一种方式，它会将流数据分发到有限大小的桶（bucket）中进行分析。**

### Window的分类

#### 按照time和count分类

**time-window**:时间窗口:根据时间划分窗口,如:每xx分钟统计最近xx分钟的数据

**count-window**:数量窗口:根据数量划分窗口,如:每xx个数据统计最近xx个数据

**图示**

![1622104270812](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/27/163118-133038.png)

#### 按照slide和size分类

窗口有两个重要的属性: **窗口大小size和滑动间隔slide**,根据它们的大小关系可分为:

**滚动窗口（Tumbling Windows）**

滚动窗口:**size=slide**,如:每隔10s统计最近10s的数据

![1622104549235](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/27/163552-363085.png)

![1614568758764](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/111920-559392.png)

- 将数据依据固定的窗口长度对数据进行切分，窗口大小固定。
- 时间对齐，窗口长度固定，没有重叠。
- 在时间节点上的数据可以自定义开闭区间。
- 只需要定义窗口的大小即可。
- DataStream API针对**事件时间和处理时间**的滚动窗口分别提供了对应的分配器TumblingEventTimeWindows 和 TumblingProcessingTimeWindows 
- 滚动窗口分配器只接受一个参数，以时间单元表示窗口的大小，他可以利用分配器的of(Time size)进行指定。

> 特点：时间对齐，窗口长度固定，数据没有重叠。 

**滑动窗口（Sliding Windows）**

滑动窗口:**size>slide**,如:每隔5s统计最近10s的数据

![1622104608127](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/27/163650-745134.png)

例如，你有 10 分钟的窗口和 5 分钟的滑动，那么每个窗口中 5 分钟的窗口里包含着上个 10 分钟产生的数据，如下图所示 

![1614568849756](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/112106-853588.png)

> 注意:当size<slide的时候,如每隔15s统计最近10s的数据,那么中间5s的数据会丢失,所有开发中不用。

- 滑动窗口是固定窗口的更广义的一种形式，滑动窗口由固定的窗口长度和滑动间隔组成。
- 窗口长度固定，可以有数据的重叠。
- 可以自定义滑动的间隔。
- 滑动窗口分配器将元素分配给大小固定且按指定滑动间隔移动的窗口。 
- 对于滑动窗口而言，你需要指定窗口大小以及用于定义新窗口开始频率的滑动间隔。如果滑动间隔小于窗口大小，则窗口会出现重叠，此时元素会被分配给多个窗口:如果滑动间隔大于窗口大小，则一些元素可能不会分配给任何窗口，因此可能会被直接丢弃。 
- Data Stream API 提供了针对事件时间和处理时间的分配器以及相关的简写方怯 

> 特点：时间对齐，窗口长度固定， 可以有数据重叠 

**会话窗口（Session Windows）**

![1614568939743](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/27/162854-431280.png)

- 由一系列事件组合一个指定时间长度的timeout间隙组成，也就是一段时间没有接收到新数据就会生成新的窗口
- 特点：**时间无对齐**。
- 可以指定回话间隔，也就是多长时间没有产生新的数据就开辟一个窗口
- 会话窗口将元素放入长度可变且不重叠的窗口中。会话窗口的边界由非活动间隔，即持续没有收到记录的时间间隔来定义 。
- session 窗口分配器通过 session 活动来对元素进行分组， session 窗口跟滚动窗口和滑动窗口相比，不会有重叠和固定的开始时间和结束时间的情况，相反，当它在一个固定的时间周期内不再收到元素，即非活动间隔产生，那这个窗口就会关闭。一个 session 窗口通过一个 session 间隔来配置，这个 session 间隔定义了非活跃周期的长度，当这个非活跃周期产生，那么当前的 session 将关闭并且后续的元素将被分配到新的 session 窗口中去。 

> 特点：时间对齐，窗口长度固定， 可以有重叠 

**综上，窗口的划分如下**

- 时间窗口（Time Window）:按照**时间**来划分数据

  根据数据的移动规则进行划分

  - 滚动时间窗口，用的较多
  - 滑动时间窗口，用的较多
  - 会话窗口

- 计数窗口（Count Window）：按照数据的个数划分数据

  根据数据的移动规则进行划分

  - 滚动计数窗口，用的较少
  - 滑动计数窗口，用的较少

### Window Api

#### 总体概况

![1614587298004](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/162821-962794.png)

首先按照key进行分组，然后按照窗口对分组进行聚合操作。

Non-Keyed-Windows是直接对窗口中的数据进行聚合操作。

#### API类图

**api**

```java
@PublicEvolving
public abstract class Window {

	/**
	 * Gets the largest timestamp that still belongs to this window.
	 *
	 * @return The largest timestamp that still belongs to this window.
	 */
	public abstract long maxTimestamp();
}
```

**继承结构**

![1614600461734](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/200745-895551.png)

**类图**

![1615716866515](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/14/181429-750953.png)

#### 窗口的创建

**新建窗口需要两个组件**

1. 一 个用于决定输入流中的元素该如何划分给窗口的分配器( window assigner) 。窗口分配器会产生一个**WindowedStream** (如果用在非键值分区的 DataStream 上则是 **AIIWindowedStream**) 。 
2. 一 个作用于 WindowedStream (或 AIIWindowedStream ) 上，用于处理分配到窗口中元素的**窗口函数**。 

![1622876156662](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/145557-227196.png)

- 窗口分配器——window()方法
- 我们可以用.window()来定义一个窗口，然后基于这个window去做一些聚合或者其它处理操作。注意window ()方法必须在keyBy之后才能用，windowAll可以作用与非键值事件的数据。
- Flink提供了更加简单的.timeWindow和.countWindow方法，用于定义**时间窗口和计数窗口**。

![1614573542742](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/145605-88098.png)

##### 窗口分配器（window assigner）

- window()方法接收的输入参数是一个WindowAssigner
- WindowAssigner负责将每条输入的数据分发到正确的window中
- Flink提供了通用的WindowAssigner
  - 滚动窗口（tumbling window）
  - 滑动窗口（sliding window）
  - 会话窗口（session window）
  - 全局窗口（global window）

**分配器继承结构**

![1615717395768](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/14/182317-714546.png)

##### 创建不同类型的窗口

根据**参数的个**数来区分是**滑动窗口还是滚动窗口**。

- 滚动时间窗口（tumblingtime window）

Flink 默认的时间窗口根据 Processing Time 进行窗口的划分，将 Flink 获取到的数据根据进入 Flink 的时间划分到不同的窗口中 ，时间间隔可以通过 Time.milliseconds(x)， Time.seconds(x)， Time.minutes(x)等其中的一个来指定。 

```java
//里面的参数是窗口分配器 
.window(TumblingProcessingTimeWindows.of(Time.seconds(15)));
.window(TumblingEventTimeWindows.of(Time.seconds(1)))
  //下面这种写法是上面两种方法的简写，通过参数的个数确定是滑动窗口还是滚动窗口
 .timeWindow(Time.seconds(15));
```

> 滚动窗口默认窗口大小size和sliding式相同的。

- 滑动时间窗口（sliding time window）

滑动窗口和滚动窗口的函数名是完全一致的，只是在传参数时需要传入两个参数，一个是 window_size，一个是 sliding_size。 时间间隔可以通过 Time.milliseconds(x)， Time.seconds(x)，Time.minutes(x)等其中的一个来指定 

```java
//处理时间滑动窗口分配器，注意有两个参数
.window(SlidingProcessingTimeWindows.of(Time.hours(1), Time.seconds(15)))
//事件时间滑动窗口，注意两个参数
.window(SlidingEventTimeWindows.of(Time.hours(1), Time.seconds(15)))
//下面是简写，按照参数个数判断是时间窗口还是滑动窗口
.timeWindow(Time.seconds(12),Time.seconds(2));
```
> 滑动窗口，窗口的大小size大于滑动的距离sliding。
> 
> 窗口的大小size小于滑动的距离sliding时候会发生数据的丢失，一般情况下不适用。

- 会话窗口（session window）

```JAVA
//事件时间会话窗口分配器
.window(EventTimeSessionWindows.withGap(Time.seconds(25)));
//处理时间会话窗口分配器
 .window(ProcessingTimeSessionWindows.withGap(Time.seconds(12)))
```
> 会话窗口只需要添加一个时间间隔即可，表示多长时间没有数据就开启一个新的窗口。

**计数窗口**

CountWindow根据**窗口中相同key元素的数量**来触发执行，执行时只计算元素数量达到窗口大小的key对应的结果。

> 注意： CountWindow的window_size指的是相同 Key 的元素的个数，不是输入的所有元素的总数。 

- 滚动计数窗口（tumblingcount window）

默认的 CountWindow 是一个滚动窗口，只需要指定窗口大小即可，当元素数量达到窗口大小时，就会触发窗口的执行。 

```java
countWindow(10);
```
> 滚动计数窗口中，窗口的大小默认和滚动事件个数式一致的。

- 滑动计数窗口（sliding count window）

滑动窗口和滚动窗口的函数名是完全一致的，只是在传参数时需要传入两个参数，**一个是 window_size，一个是 sliding_size**。下面代码中的 sliding_size 设置为了 2，也就是说，每收到两个相同 key 的数据
就计算一次，每一次计算的 window范围是 10 个元素 

```java
countWindow(10,2);
```

> 同样，窗口的大小要大于滑动的事件个数，否则会发生数据的丢失。


##### 窗口函数（window function）

**类图**

![1614607444753](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/220406-129546.png)

- window function定义了要对窗口中收集的数据做的计算操作。
- 窗口函数定义了针对窗口内元素的计算逻辑。 

**函数分类**：两类

- **增量聚合函数**（incremental aggregation functions）：类似于流处理的过程，每来一条数据，就在以前数据的基础上做聚合操作。
  - **它的应用场景是窗口内以状态形式存储某个值且需要根据每个加入窗口的元素对该值进行更新**。此类函数通常会十分节省空间且最终会将聚合值作为单个结果发送出去。因为最终只存储的式一个结果状态。 
  - 每条数据到来就进行计算，保持一个简单的状态。
  - ReduceFunction, AggregateFunction
- 全窗口函数（full window functions）
  - 它会接收集窗口内的所有元素，并在执行计算时对它们进行遍历。虽然全量窗口函数通常需要占用更多空间，但它和增量聚合函数相比，支持更复杂的逻辑。 
  - 先把窗口所有数据收集起来，等到计算的时候会遍历所有数据。
  - ProcessWindowFunction，WindowFunction

###### 增量聚合函数

**AggregateFunction接口**

```java
@PublicEvolving
public interface AggregateFunction<IN, ACC, OUT> extends Function, Serializable {
// in输入类型，acc:累加操作，out:输出操作
	/**
	 * Creates a new accumulator, starting a new aggregate.
	 *
	 * <p>The new accumulator is typically meaningless unless a value is added
	 * via {@link #add(Object, Object)}.
	 *
	 * <p>The accumulator is the state of a running aggregation. When a program has multiple
	 * aggregates in progress (such as per key and window), the state (per key and window)
	 * is the size of the accumulator.
	 *
	 * @return A new accumulator, corresponding to an empty aggregate.
	 */
  ////创建一个累加器来启动聚合
	ACC createAccumulator();

	/**
	 * Adds the given input value to the given accumulator, returning the
	 * new accumulator value.
	 *
	 * <p>For efficiency, the input accumulator may be modified and returned.
	 *
	 * @param value The value to add
	 * @param accumulator The accumulator to add the value to
	 */
  //向 累加器中添加一个输入元素并返回累加器
	ACC add(IN value, ACC accumulator);

	/**
	 * Gets the result of the aggregation from the accumulator.
	 *
	 * @param accumulator The accumulator of the aggregation
	 * @return The final aggregation result.
	 */
  //根据累加器计算并返回结果
	OUT getResult(ACC accumulator);

	/**
	 * Merges two accumulators, returning an accumulator with the merged state.
	 *
	 * <p>This function may reuse any of the given accumulators as the target for the merge
	 * and return that. The assumption is that the given accumulators will not be used any
	 * more after having been passed to this function.
	 *
	 * @param a An accumulator to merge
	 * @param b Another accumulator to merge
	 *
	 * @return The accumulator with the merged state
	 */
  // 合并两个累加器并返回合并结果
	ACC merge(ACC a, ACC b);
}
//i去接口定 义 了输 入 类型 IN ， 累 加 器类型 ACC 以 及结果类型 OUT。它和ReduceFunction 不同的是中间数据类型 以及结果类型不再依赖输入类型。
//ReduceFunction 接收两个同类型的值井将它们组合生成一个类型不变的值。当被用在窗口化数据流上时， ReduceFunction 会对分配给窗口的元素进行增量聚合。窗口只需要存储当前聚合结果，一个和 ReduceFunction 的输入及输出类型都相同的值。每当收到一个新元素，算子都会以该元素和从窗口状态取出的当前聚合值为参数调用ReduceFunction ， 随后会用 ReduceFunction 的结果替换窗口状态。
@Public
@FunctionalInterface
public interface ReduceFunction<T> extends Function, Serializable {

	/**
	 * The core method of ReduceFunction, combining two values into one value of the same type.
	 * The reduce function is consecutively applied to all values of a group until only a single value remains.
	 *
	 * @param value1 The first value to combine.
	 * @param value2 The second value to combine.
	 * @return The combined value of both input values.
	 *
	 * @throws Exception This method may throw exceptions. Throwing an exception will cause the operation
	 *                   to fail and may trigger recovery.
	 */
	T reduce(T value1, T value2) throws Exception;
}
```

**案例**

```java
public class TimeWindow {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//        开窗测试
//        先做分组操作,也可以直接使用timewindow开窗口
        //                countWindow(10,2);

        //.window(EventTimeSessionWindows.withGap(Time.seconds(25)));
//                .timeWindow(Time.seconds(15));
//                window(TumblingProcessingTimeWindows.of(Time.seconds(15)));
//          做增量聚合操作
       /* map.keyBy("id").timeWindow(Time.seconds(15)).reduce(new ReduceFunction<SensorReading>() {
            @Override
            public SensorReading reduce(SensorReading value1, SensorReading value2) throws Exception {

            }
        });*/
//          使用AggregateFunction函数,实现count()计数功能
        DataStream<Integer> result = map.keyBy("id").timeWindow(Time.seconds(15)).aggregate(new AggregateFunction<SensorReading, Integer, Integer>() {
//      SensorReading:输入类型
//            integer:输出类型
//            Integer：输出类型

            @Override
            public Integer createAccumulator() {
//                创建一个累加器
                return 0;
            }

            @Override
            public Integer add(SensorReading value, Integer accumulator) {
//                做累加操作
                return accumulator + 1;
            }

            @Override
            public Integer getResult(Integer accumulator) {
//                获取结果
                return accumulator;
            }

            @Override
            public Integer merge(Integer a, Integer b) {
//                做分区合并操作
                return a + b;
            }
        });

        result.print();
//        map.print();
        env.execute();
    }
}
```

###### 全窗口函数

ReduceFunction 和 AggregateFunction 都是对分配到窗口的事件进行增量计算。 然而有些时候我们需要访问窗口内的所有元素来执行一些更加复杂的计算，例如计算窗口内数据的中值或出现频率最高的值。对于此类应用，ReduceFunction 和 AggregateFunction 都不适合。 FlinkDataStream API 提供的 ProcessWindowFunction 可以对窗口内容执行任意计算。 

**WindowFunction接口**

```java
@Public
public interface WindowFunction<IN, OUT, KEY, W extends Window> extends Function, Serializable {
//in:输入 out:输出，key:键的类型，w:当前的window类型
  //key:默认是元祖类型
	/**
	 * Evaluates the window and outputs none or several elements.
	 *
	 * @param key The key for which this window is evaluated.
	 * @param window The window that is being evaluated.
	 * @param input The elements in the window being evaluated.
	 * @param out A collector for emitting elements.
	 * 
	 * @throws Exception The function may throw exceptions to fail the program and trigger recovery. 
	 */
	void apply(KEY key, W window, Iterable<IN> input, Collector<OUT> out) throws Exception;
}
//已经被ProcessWindowFunction取代
```

**ProcessWindowFunction接口**

```java
/**
 * Base abstract class for functions that are evaluated over keyed (grouped) windows using a context
 * for retrieving extra information.
 *
 * @param <IN> The type of the input value.
 * @param <OUT> The type of the output value.
 * @param <KEY> The type of the key.
 * @param <W> The type of {@code Window} that this window function can be applied on.
 */
@PublicEvolving
public abstract class ProcessWindowFunction<IN, OUT, KEY, W extends Window> extends AbstractRichFunction {

	private static final long serialVersionUID = 1L;

	/**
	 * Evaluates the window and outputs none or several elements.
	 *
	 * @param key The key for which this window is evaluated.
	 * @param context The context in which the window is being evaluated.
	 * @param elements The elements in the window being evaluated.
	 * @param out A collector for emitting elements.
	 *
	 * @throws Exception The function may throw exceptions to fail the program and trigger recovery.
	 */
  // 对窗口执行计算
	public abstract void process(KEY key, Context context, Iterable<IN> elements, Collector<OUT> out) throws Exception;

	/**
	 * Deletes any state in the {@code Context} when the Window is purged.
	 *
	 * @param context The context to which the window is being evaluated
	 * @throws Exception The function may throw exceptions to fail the program and trigger recovery.
	 */
  // 在窗口清除时删除自定义的单个窗口状态
	public void clear(Context context) throws Exception {}

	/**
	 * The context holding window metadata.
	 */
  // 保存窗口无数据的上下文
	public abstract class Context implements java.io.Serializable {
		/**
		 * Returns the window that is being evaluated.
		 */
    // 返回窗口的元数据
		public abstract W window();

		/** Returns the current processing time. */
		public abstract long currentProcessingTime();

		/** Returns the current event-time watermark. */
    // 返回当前处理时间
		public abstract long currentWatermark();

		/**
		 * State accessor for per-key and per-window state.
		 *
		 * <p><b>NOTE:</b>If you use per-window state you have to ensure that you clean it up
		 * by implementing {@link ProcessWindowFunction#clear(Context)}.
		 */
		public abstract KeyedStateStore windowState();

		/**
		 * State accessor for per-key global state.
		 */
		public abstract KeyedStateStore globalState();

		/**
		 * Emits a record to the side output identified by the {@link OutputTag}.
		 *
		 * @param outputTag the {@code OutputTag} that identifies the side output to emit to.
		 * @param value The record to emit.
		 */
		public abstract <X> void output(OutputTag<X> outputTag, X value);
	}
}
```

**案例**

```java
public class TimeWindow_fullwin {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });
//        全窗口
        SingleOutputStreamOperator<Tuple3<String,Long,Integer>> id = map.keyBy("id")
          //时间窗口
                .timeWindow(Time.seconds(15))
                .process(new ProcessWindowFunction<SensorReading, Object, Tuple, TimeWindow>() {
                    /**
                     *
                     * @param tuple
                     * @param context 上下文，里面包含window
                     * @param elements
                     * @param out
                     * @throws Exception
                     */
                    @Override
                    public void process(Tuple tuple, Context context, Iterable<SensorReading> elements, Collector<Object> out) throws Exception {
                      //处理逻辑
                    }
                })
                .apply(new WindowFunction<SensorReading, Tuple3<String,Long,Integer>, Tuple, TimeWindow>() {

                    /**
                     *和增量函数的区别就是，此函数使用Iterable吧数据全部拿来，然后在做处理，而增量是拿到一个数据，就处理一个数据
                     * @param tuple 当前的键
                     * @param window window类型
                     * @param input 当前的所有输入的数据
                     * @param out 输出数据，没有返回值类型，使用out输出
                     * @throws Exception
                     */
                    @Override
                    public void apply(Tuple tuple, TimeWindow window, Iterable<SensorReading> input, Collector<Tuple3<String,Long,Integer>> out) throws Exception {
                        String id=tuple.getField(0);//获取id，根据位置获取
                        Long windowAnd=window.getEnd();
                        int size = IteratorUtils.toList(input.iterator()).size();
                        out.collect(new Tuple3<>(id,windowAnd,size));
                    }
                });
        id.print();


        env.execute();
    }
}

//apply是windowFunction中的方法
//process是ProcessWindowFunction中的方法
```

###### 计数窗口

**接口**

```java
@PublicEvolving
public interface AggregateFunction<IN, ACC, OUT> extends Function, Serializable {

	/**
	 * Creates a new accumulator, starting a new aggregate.
	 *
	 * <p>The new accumulator is typically meaningless unless a value is added
	 * via {@link #add(Object, Object)}.
	 *
	 * <p>The accumulator is the state of a running aggregation. When a program has multiple
	 * aggregates in progress (such as per key and window), the state (per key and window)
	 * is the size of the accumulator.
	 *
	 * @return A new accumulator, corresponding to an empty aggregate.
	 */
	ACC createAccumulator();

	/**
	 * Adds the given input value to the given accumulator, returning the
	 * new accumulator value.
	 *
	 * <p>For efficiency, the input accumulator may be modified and returned.
	 *
	 * @param value The value to add
	 * @param accumulator The accumulator to add the value to
	 */
	ACC add(IN value, ACC accumulator);

	/**
	 * Gets the result of the aggregation from the accumulator.
	 *
	 * @param accumulator The accumulator of the aggregation
	 * @return The final aggregation result.
	 */
	OUT getResult(ACC accumulator);

	/**
	 * Merges two accumulators, returning an accumulator with the merged state.
	 *
	 * <p>This function may reuse any of the given accumulators as the target for the merge
	 * and return that. The assumption is that the given accumulators will not be used any
	 * more after having been passed to this function.
	 *
	 * @param a An accumulator to merge
	 * @param b Another accumulator to merge
	 *
	 * @return The accumulator with the merged state
	 */
	ACC merge(ACC a, ACC b);
}
```

**案例**

```java
public class CountWindow {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//        开计数窗口测试
        SingleOutputStreamOperator<Double> result = map.keyBy("id")
                //统计5个数的平均值，每两个数滑动一次
                .countWindow(5, 2)
                .aggregate(new MyAvgTemp());
        result.print();


        env.execute();
    }

    public static class MyAvgTemp implements AggregateFunction<SensorReading, Tuple2<Double,Integer>,Double>{

        @Override
        public Tuple2<Double, Integer> createAccumulator() {
//            创建一个tuple2(),作为中间计算使用
            return new Tuple2<>(0.0,0);
        }

        @Override
        public Tuple2<Double, Integer> add(SensorReading value, Tuple2<Double, Integer> accumulator) {
//            加上当前记录的温度
            return new Tuple2<>(accumulator.f0+value.getTemperature(),accumulator.f1+1);
        }

        @Override
        public Double getResult(Tuple2<Double, Integer> accumulator) {
            return accumulator.f0/accumulator.f1;
        }

        @Override
        public Tuple2<Double, Integer> merge(Tuple2<Double, Integer> a, Tuple2<Double, Integer> b) {
            return new Tuple2<>(a.f0+b.f0,a.f1+b.f1);
        }
    }
}

```

##### 其他API

1. .trigger()——触发器：定义window什么时候关闭，触发计算并输出结果
2. .evictor()——移除器：定义移除某些数据的逻辑
3. .allowedLateness()——允许处理迟到的数据
4. .sideOutputLateData()——将迟到的数据放入侧输出流
5. .getSideOutput()——获取侧输出流

##### 案例

**基于时间的滚动和滑动窗口**

![1622878592124](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622878592124.png)

**代码演示**

```java
//nc -lk 9999
public class Test17 {
    /*
    1,5
    2,5
    3,5
    4,5
     */
    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        SingleOutputStreamOperator<CarInfo> mapData = socket.map(new MapFunction<String, CarInfo>() {
            @Override
            public CarInfo map(String value) throws Exception {

                String[] s = value.split(" ");
                return new CarInfo(s[0], Integer.parseInt(s[1]));
            }
        });

//        每5秒钟统计一次，统计最近5秒内，各个红绿灯路口通过的汽车的数量，基于时间的滚动窗口
//        每5秒钟统计一次，统计最近10秒内，各个红绿灯路口通过的汽车的数量，基于时间的滑动窗口

//        需求中要的是各个路口/红绿灯的结果，所以需要先进行分组
        KeyedStream<CarInfo, String> keyedData = mapData.keyBy(item -> item.sensorId);

//        创建滑动窗口
        SingleOutputStreamOperator<CarInfo> res1 = keyedData.window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
                .sum("count");//做聚合操作

        SingleOutputStreamOperator<CarInfo> res2 = keyedData.window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))
                .sum("count");


        res1.print();


        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class CarInfo{
        private String sensorId;//信号灯的id
        private Integer count;//通过该信号灯的车的数量
    }
}
```

**基于数量的滚动和滑动窗口**

![1622878655029](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/144629-538798.png)

**代码演示**

```java
public class Test18 {
    /*
    1,5
    2,5
    3,5
    4,5
     */
    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        SingleOutputStreamOperator<CarInfo> mapData = socket.map(new MapFunction<String, CarInfo>() {
            @Override
            public CarInfo map(String value) throws Exception {

                String[] s = value.split(" ");
                return new CarInfo(s[0], Integer.parseInt(s[1]));
            }
        });

//        每5秒钟统计一次，统计最近5秒内，各个红绿灯路口通过的汽车的数量，基于时间的滚动窗口
//        每5秒钟统计一次，统计最近10秒内，各个红绿灯路口通过的汽车的数量，基于时间的滑动窗口

//        需求中要的是各个路口/红绿灯的结果，所以需要先进行分组
        KeyedStream<CarInfo, String> keyedData = mapData.keyBy(item -> item.sensorId);

        SingleOutputStreamOperator<CarInfo> res1 = keyedData.countWindow(5)
                .sum("count");

        SingleOutputStreamOperator<CarInfo> res2 = keyedData.countWindow(5, 3)
                .sum("count");

        res1.print();


        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class CarInfo{
        private String sensorId;//信号灯的id
        private Integer count;//通过该信号灯的车的数量
    }
}
```

**会话窗口**

![1622879583581](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/155446-57266.png)

**代码演示**

```java
public class Test19 {
   
    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
        DataStreamSource<String> socket = env.socketTextStream("hadoop100", 9999);

        SingleOutputStreamOperator<CarInfo> mapData = socket.map(new MapFunction<String, CarInfo>() {
            @Override
            public CarInfo map(String value) throws Exception {

                String[] s = value.split(" ");
                return new CarInfo(s[0], Integer.parseInt(s[1]));
            }
        });

//        每5秒钟统计一次，统计最近5秒内，各个红绿灯路口通过的汽车的数量，基于时间的滚动窗口
//        每5秒钟统计一次，统计最近10秒内，各个红绿灯路口通过的汽车的数量，基于时间的滑动窗口

//        需求中要的是各个路口/红绿灯的结果，所以需要先进行分组
        KeyedStream<CarInfo, String> keyedData = mapData.keyBy(item -> item.sensorId);

        SingleOutputStreamOperator<CarInfo> res = keyedData.window(ProcessingTimeSessionWindows.withGap(Time.seconds(10)))
                .sum("count");

        res.print();
        env.execute();
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class CarInfo{
        private String sensorId;//信号灯的id
        private Integer count;//通过该信号灯的车的数量
    }
}
```

##### Window API总览

![1614587298004](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/162821-962794.png)