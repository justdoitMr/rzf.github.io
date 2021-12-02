## Flink CEP



### 什么是复杂事件处理 CEP？

CEP 的意思是复杂事件处理，例如：起床-->洗漱-->吃饭-->上班等一系列串联起来的事件流形成的模式称为 CEP。如果发现某一次起床后没有刷牙洗脸亦或是吃饭就直接上班，就可以把这种非正常的事件流匹配出来进行分析，看看今天是不是起晚了。

简而言之，**就是由一个或多个由简单事件构成的事件流通过一定的规则匹配，然后输出用户想得到的数据，满足规则的复杂事件。**

特征：

- 目标：从有序的简单事件流中发现一些高阶特征
- 输入：一个或多个由简单事件构成的事件流
- 处理：识别简单事件之间的内在联系，多个符合一定规则的简单事件构成复杂事件
- 输出：满足规则的复杂事件

如下图，从一条数据流中匹配出复杂的特征

![1638451143517](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/211904-59374.png)

CEP 用于分析低延迟、频繁产生的不同来源的事件流。CEP 可以帮助在复杂的、不相关的事件流中找出有意义的模式和复杂的关系，以接近实时或准实时的获得通知并阻止一些行为。

CEP 支持在流上进行模式匹配，根据模式的条件不同，分为连续的条件或不连续的条件；模式的条件允许有时间的限制，当在条件范围内没有达到满足的条件时，会导致模式匹配超时。

看起来很简单，但是它有很多不同的功能：

- 输入的流数据，尽快产生结果
- 在 2 个 event 流上，基于时间进行聚合类的计算
- 提供实时/准实时的警告和通知
- 在多样的数据源中产生关联并分析模式
- 高吞吐、低延迟的处理

![1638449092915](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/204453-123647.png)

- **第一个是异常行为检测的例子：**假设车辆维修的场景中，当一辆车出现故障时，这辆车会被送往维修点维修，然后被重新投放到市场运行。如果这辆车被投放到市场之后还未被使用就又被报障了，那么就有可能之前的维修是无效的。
- **第二个是策略营销的例子：**假设打车的场景中，用户在 APP 上规划了一个行程订单，如果这个行程在下单之后超过一定的时间还没有被司机接单的话，那么就需要将这个订单输出到下游做相关的策略调整。 
- **第三个是运维监控的例子：**通常运维会监控服务器的 CPU、网络 IO 等指标超过阈值时产生相应的告警。但是在实际使用中，后台服务的重启、网络抖动等情况都会造成瞬间的流量毛刺，对非关键链路可以忽略这些毛刺而只对频繁发生的异常进行告警以减少误报。

### Flink CEP应用场景

- **风险控制：**对用户异常行为模式进行实时检测，当一个用户发生了不该发生的行为，判定这个用户是不是有违规操作的嫌疑。
- **策略营销：**用预先定义好的规则对用户的行为轨迹进行实时跟踪，对行为轨迹匹配预定义规则的用户实时发送相应策略的推广。
- **运维监控：**灵活配置多指标、多依赖来实现更复杂的监控模式。

### Flink CEP原理

![1638449634278](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/205355-460139.png)

Flink CEP 内部是用 NFA（非确定有限自动机）来实现的，由点和边组成的一个状态图，以一个初始状态作为起点，经过一系列的中间状态，达到终态。点分为起始状态、中间状态、最终状态三种，边分为 take、ignore、proceed 三种。 

- **take**：必须存在一个条件判断，当到来的消息满足 take 边条件判断时，把这个消息放入结果集，将状态转移到下一状态。
- **ignore**：当消息到来时，可以忽略这个消息，将状态自旋在当前不变，是一个自己到自己的状态转移。 
- **proceed**：又叫做状态的空转移，当前状态可以不依赖于消息到来而直接转移到下一状态。举个例子，当用户购买商品时，如果购买前有一个咨询客服的行为，需要把咨询客服行为和购买行为两个消息一起放到结果集中向下游输出；如果购买前没有咨询客服的行为，只需把购买行为放到结果集中向下游输出就可以了。 也就是说，如果有咨询客服的行为，就存在咨询客服状态的上的消息保存，如果没有咨询客服的行为，就不存在咨询客服状态的上的消息保存，咨询客服状态是由一条 proceed 边和下游的购买状态相连。

下面以一个打车的例子来展示状态是如何流转的，规则见下图所示。

![1638449810680](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/205650-891334.png)

以乘客制定行程作为开始，匹配乘客的下单事件，如果这个订单超时还没有被司机接单的话，就把行程事件和下单事件作为结果集往下游输出。

假如消息到来顺序为：行程-->其他-->下单-->其他。

状态流转如下：

1. 开始时状态处于**行程状态**，即等待用户制定行程。 

![1638449849681](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/205729-834941.png)

2. 当收到行程事件时，**匹配行程状态的条件，把行程事件放到结果集中，通过 take 边将状态往下转移到下单状态。** 

![1638449881863](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/205802-725099.png)

3. 由于下单状态上有一条 ignore 边，所以可以忽略收到的其他事件，直到收到下单事件时将其匹配，放入结果集中，并且将当前状态往下转移到超时未接单状态。这时候结果集当中有两个事件：**制定行程事件和下单事件**。  

![1638449949695](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/205909-389159.png)

![1638449965069](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1638449965069.png)

4. 超时未接单状态时，如果来了一些其他事件，同样可以被 ignore 边忽略，直到超时事件的触发，将状态往下转移到最终状态，这时候整个模式匹配成功，最终将结果集中的制定行程事件和下单事件输出到下游。

![1638450024226](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/210024-988896.png)

上面是一个匹配成功的例子，如果是不成功的例子会怎么样？

假如当状态处于超时未接单状态时，收到了一个接单事件，那么就不符合超时未被接单的触发条件，此时整个模式匹配失败，之前放入结果集中的行程事件和下单事件会被清理。

![1638450091486](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/210131-163472.png)

### Flink CEP程序开发

Flink 为 CEP 提供了专门的 Flink CEP library，它包含如下组件：

- Event Stream
- pattern 定义
- pattern 检测
- 生成 Alert

![1638451232865](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/212033-31204.png)

首先，开发人员要在 DataStream 流上定义出模式条件，之后 Flink CEP 引擎进行模式检测，必要时生成告警。

#### Flink CEP 程序结构

主要分为两部分：**定义事件模式和匹配结果处理。**

**官方案例**

~~~ java
DataStream<Event> input = ...
Pattern<Event, ?> pattern = Pattern.<Event>begin("start").where(
        new SimpleCondition<Event>() {
            @Override
            public boolean filter(Event event) {
                return event.getId() == 42;
            }
        }
    ).next("middle").subtype(SubEvent.class).where(
        new SimpleCondition<SubEvent>() {
            @Override
            public boolean filter(SubEvent subEvent) {
                return subEvent.getVolume() >= 10.0;
            }
        }
    ).followedBy("end").where(
         new SimpleCondition<Event>() {
            @Override
            public boolean filter(Event event) {
                return event.getName().equals("end");
            }
         }
    );

PatternStream<Event> patternStream = CEP.pattern(input, pattern);

DataStream<Alert> result = patternStream.select(
    new PatternProcessFunction<Event, Alert>() {
        @Override
        public void select(
                Map<String, List<Event>> pattern,
                Context ctx,
                Collector<Alert> out) throws Exception {
            out.collect(createAlertFrom(pattern));
        }
    });
~~~

**程序结构分为三部分：**首先需要定义一个模式(**Pattern**)，即第 2 行代码所示，接着把定义好的模式绑定在 DataStream 上（**第 25 行**），最后就可以在具有 CEP 功能的 DataStream 上将匹配的结果进行处理（**第 27 行**）。下面对关键部分做详细讲解：

- **定义模式：**上面示例中，分为了三步，首先匹配一个 ID 为 42 的事件，接着匹配一个体积大于等于 10 的事件，最后等待收到一个 name 等于 end 的事件。 
- **匹配结果输出：**此部分，需要重点注意 select 函数（第 30 行，注：本文基于 Flink 1.7 版本）里边的 Map 类型的 pattern 参数，Key 是一个 pattern 的 name，它的取值是模式定义中的 Begin 节点 start，或者是接下来 next 里面的 middle，或者是第三个步骤的 end。后面的 map 中的 value 是每一步发生的匹配事件。因在每一步中是可以使用循环属性的，可以匹配发生多次，所以 map 中的 value 是匹配发生多次的所有事件的一个集合。

#### Flink CEP构成

![1638450305574](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/210506-889877.png)

上图中，蓝色方框代表的是一个个**单独的模式**；浅黄色的椭圆代表的是这个模式上可以**添加的属性**，包括模式可以发生的循环次数，或者这个模式是贪婪的还是可选的；橘色的椭圆代表的是**模式间的关系**，定义了多个模式之间是怎么样串联起来的。通过定义模式，添加相应的属性，将多个模式串联起来三步，就可以构成了一个完整的 Flink CEP 程序。

简单来说三部分：

- 模式
- 模式上的属性
- 模式与模式之间的关系

#### 定义模式

~~~ java
pattern.next("start").where(
        new SimpleCondition<Event>() {
            @Override
            public boolean filter(Event event) {
                return event.getId() == 42;
            }
        }
)
~~~

定义模式主要有如下 5 个部分组成：

- **pattern**：前一个模式
- **next/followedBy/...**：开始一个新的模式
- **start**：模式名称
- **where**：模式的内容
- **filter**：核心处理逻辑

#### 模式的属性

接下来介绍一下怎样设置模式的属性。模式的属性主要分为**循环属性和可选属性**。

- 循环属性可以定义模式匹配发生固定次数（**times**），匹配发生一次以上（**oneOrMore**），匹配发生多次以上(**timesOrMore**)。
- 可选属性可以设置模式是贪婪的（**greedy**），即匹配最长的串，或设置为可选的（**optional**），有则匹配，无则忽略。

#### 模式的有效期

由**于模式的匹配事件存放在状态中进行管理，所以需要设置一个全局的有效期（within）**。若不指定有效期，匹配事件会一直保存在状态中不会被清除。至于有效期能开多大，要依据具体使用场景和数据量来衡量，关键要看匹配的事件有多少，随着匹配的事件增多，新到达的消息遍历之前的匹配事件会增加 CPU、内存的消耗，并且随着状态变大，数据倾斜也会越来越严重。

#### **模式间的联系**

**主要分为三种：**

- 严格连续性（next/notNext）

- 宽松连续性（followedBy/notFollowedBy）

- 和非确定宽松连续性（followedByAny）。

三种模式匹配的差别见下表所示：

![1638450592904](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/210953-243279.png)

总结如下：

- **严格连续性**：需要消息的顺序到达与模式完全一致。
- **宽松连续性**：允许忽略不匹配的事件。
- **非确定宽松连性**：不仅可以忽略不匹配的事件，也可以忽略已经匹配的事件。

#### 多模式组合

除了前面提到的模式定义和模式间的联系，还可以把相连的多个模式组合在一起看成一个模式组，类似于视图，可以在这个模式视图上进行相关操作。

![1638450636868](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/211040-978538.png)

上图这个例子里面，首先匹配了一个登录事件，然后接下来匹配浏览，下单，购买这三个事件反复发生三次的用户。 

如果没有模式组的话，代码里面浏览，下单，购买要写三次。有了模式组，只需把浏览，下单，购买这三个事件当做一个模式组，把相应的属性加上 times(3) 就可以了。

#### 处理结果

**处理匹配的结果主要有四个接口：**

- PatternFlatSelectFunction

- PatternSelectFunction

- PatternFlatTimeoutFunction 

- PatternTimeoutFunction。

从名字上可以看出，输出可以分为两类：select 和 flatSelect 指定输出一条还是多条，timeoutFunction 和不带 timeout 的 Function 指定可不可以对超时事件进行旁路输出。 

下图是输出的综合示例代码：

![1638450770237](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/02/211425-891126.png)

### 使用案例

#### 以登录事件流为例

~~~scala
case class LoginEvent(userId: String, ip: String, eventType: String, eventTime: String)

val env = StreamExecutionEnvironment.getExecutionEnvironment
 env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
 env.setParallelism(1)

val loginEventStream = env.fromCollection(List(
  LoginEvent("1", "192.168.0.1", "fail", "1558430842"),
  LoginEvent("1", "192.168.0.2", "fail", "1558430843"),
  LoginEvent("1", "192.168.0.3", "fail", "1558430844"),
  LoginEvent("2", "192.168.10.10", "success", "1558430845")
 )).assignAscendingTimestamps(_.eventTime.toLong)
~~~

#### Pattern API

每个 Pattern 都应该包含几个步骤，或者叫做 state。从一个 state 到另一个 state，通常我们需要定义一些条件

每个 Pattern 都应该包含几个步骤，或者叫做 state。从一个 state 到另一个 state，通常我们需要定义一些条件，例如下列的代码：

```scala
val loginFailPattern = Pattern.begin[LoginEvent]("begin")//名字
 .where(_.eventType.equals("fail"))//过滤条件
 .next("next")
 .where(_.eventType.equals("fail"))
 .within(Time.seconds(10)
```

每个 state 都应该有一个标示：例如`.begin[LoginEvent]("begin")`中的 `"begin"`

每个 state 都需要有一个唯一的名字，而且需要一个 filter 来过滤条件，这个过滤条件定义事件需要符合的条件，例如:

```scala
.where(_.eventType.equals("fail"))//定义事件的类型为fail进行过滤
```

我们也可以通过 subtype 来限制 event 的子类型：

```scala
start.subtype(SubEvent.class).where(...);
```

事实上，你可以多次调用 subtype 和 where 方法；而且如果 where 条件是不相关的，你可以通过 or 来指定一个单独的 filter 函数：

```scala
pattern.where(...).or(...);
```

之后，我们可以在此条件基础上，通过 next 或者 followedBy 方法切换到下一个state，next 的意思是说上一步符合条件的元素之后紧挨着的元素；而 followedBy 并不要求一定是挨着的元素。这两者分别称为严格近邻和非严格近邻。

```scala
val strictNext = start.next("middle")
val nonStrictNext = start.followedBy("middle")
```

最后，我们可以将所有的 Pattern 的条件限定在一定的时间范围内：

```scala
next.within(Time.seconds(10))
```

这个时间可以是 Processing Time，也可以是 Event Time。

**Pattern 检测**

通过一个 input DataStream 以及刚刚我们定义的 Pattern，我们可以创建一个PatternStream：

```
val input = ...
val pattern = ...
val patternStream = CEP.pattern(input, pattern)
val patternStream = CEP.pattern(loginEventStream.keyBy(_.userId), loginFailPattern)
```

一旦获得 PatternStream，我们就可以通过 select 或 flatSelect，从一个 Map 序列找到我们需要的警告信息。

**select**

select 方法需要实现一个 PatternSelectFunction，通过 select 方法来输出需要的警告。它接受一个 Map 对，包含 string/event，其中 key 为 state 的名字，event 则为真实的 Event。

```scala
val loginFailDataStream = patternStream
 .select((pattern: Map[String, Iterable[LoginEvent]]) => {
 val first = pattern.getOrElse("begin", null).iterator.next()
 val second = pattern.getOrElse("next", null).iterator.next()
 Warning(first.userId, first.eventTime, second.eventTime, "warning")
 })
```

其返回值仅为 1 条记录。

**flatSelect**

通过实现 PatternFlatSelectFunction，实现与 select 相似的功能。唯一的区别就是 flatSelect 方法可以返回多条记录，它通过一个 Collector[OUT]类型的参数来将要输出的数据传递到下游。

**超时事件的处理**

通过 within 方法，我们的 parttern 规则将匹配的事件限定在一定的窗口范围内。当有超过窗口时间之后到达的 event，我们可以通过在 select 或 flatSelect 中，实现PatternTimeoutFunction 和 PatternFlatTimeoutFunction 来处理这种情况。

```scala
val patternStream: PatternStream[Event] = CEP.pattern(input, pattern)
val outputTag = OutputTag[String]("side-output")
val result: SingleOutputStreamOperator[ComplexEvent] = patternStream.select
(outputTag){
 (pattern: Map[String, Iterable[Event]], timestamp: Long) => TimeoutEvent
()
} {
 pattern: Map[String, Iterable[Event]] => ComplexEvent()
}
val timeoutResult: DataStream<TimeoutEvent> = result.getSideOutput(outputTag)
```