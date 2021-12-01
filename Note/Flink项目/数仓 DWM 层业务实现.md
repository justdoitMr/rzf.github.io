

## DWS 层与 DWM 层的设计

### 设计思路

我们在之前通过分流等手段，把数据分拆成了独立的 Kafka Topic。那么接下来如何处理数据，就要思考一下我们到底要通过实时计算出哪些指标项。

因为实时计算与离线不同，实时计算的开发和运维成本都是非常高的，要结合实际情况考虑是否有必要象离线数仓一样，建一个大而全的中间层。

如果没有必要大而全，这时候就需要大体规划一下要实时计算出的指标需求了。把这些指标以主题宽表的形式输出就是我们的 DWS 层。

### 需求梳理

![20211201142629](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211201142629.png)
![20211201142647](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211201142647.png)

为什么在离线数仓中没有DWT层，因为DWT层存放的是历史的聚集的结果，实时数仓中不需要这一层，可以实时计算。

当然实际需求还会有更多，这里主要以为可视化大屏为目的进行实时计算的处理。

DWM 层的定位是什么，DWM 层主要服务 DWS，因为部分需求直接从 DWD 层到DWS 层中间会有一定的计算量，而且这部分计算的结果很有可能被多个 DWS 层主题复用，

所以部分 DWD 成会形成一层 DWM，我们这里主要涉及业务。

- 访问 UV 计算
- 跳出明细计算
- 订单宽表
- 支付宽表

## DWM 层-访客 UV 计算


### 需求分析与思路

UV，全称是 Unique Visitor，即独立访客，对于实时计算中，也可以称为 DAU(Daily Active User)，即**每日活跃用户**，因为实时计算中的 UV 通常是指当日的访客数。

那么如何从用户行为日志中识别出当日的访客，那么有两点：

1. 其一，是识别出该访客打开的第一个页面，表示这个访客开始进入我们的应用

那具体在这里我们根据哪一个字段进行判断呢？

![20211201144511](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211201144511.png)

我们根据日志中的last_page_id判断，如果该字段是null，那么说明是今天第一次登录该页面，否则不空，也就是说不是第一次。这个字段代表上一跳的地址，为null说明没有上一跳。

2. 其二，由于访客可以在一天中多次进入应用，所以我们要在一天的范围内进行去重

因为我们要统计日活跃量，而每一个访客可以多次重复登录，所以需要进行去重操作。我们可以使用Flink中的keyStated，一个mid对应于一个状态。key1-->state,状态中可以存储**年月日**时间。

### 代码实现

#### 从 Kafka 的 dwd_page_log 主题接收数据

```java
 //TODO 2.读取Kafka dwd_page_log 主题的数据
        String groupId = "unique_visit_app";//消费者组
        String sourceTopic = "dwd_page_log";//dwd层数据源，是kafka中的一个主题
        String sinkTopic = "dwm_unique_visit";//存放在kafka中的dwm_unique_visit主题中
//        读取数据
        DataStreamSource<String> kafkaDS = env.addSource(MyKafkaUtils.getKafkaConsumer(sourceTopic, groupId));

        //TODO 3.将每行数据转换为JSON对象
        SingleOutputStreamOperator<JSONObject> jsonObjDS = kafkaDS.map(JSON::parseObject);
```

#### 核心的过滤代码实现

- 首先用 keyby 按照 mid 进行分组，每组表示当前设备的访问情况
- 分组后使用 keystate 状态，记录用户进入时间，实现 RichFilterFunction 完成过滤
- 重写 open 方法用来初始化状态
- 重写 filter 方法进行过滤
  - 可以直接筛掉 last_page_id 不为空的字段，因为只要有上一页，说明这条不是这个用户进入的首个页面。
  - 状态用来记录用户的进入时间，只要这个 lastVisitDate 是今天，就说明用户今天已经访问过了所以筛除掉。如果为空或者不是今天，说明今天还没访问过，则保留。
  - 因为状态值主要用于筛选是否今天来过，所以这个记录过了今天基本上没有用了，这里 enableTimeToLive 设定了 1 天的过期时间，避免状态过大。

```java
       //TODO 4.过滤数据  状态编程  只保留每个mid每天第一次登陆的数据
//        首先进行过滤分组，这里过滤掉的是mid为null的数据，也就是不合法的数据
        KeyedStream<JSONObject, String> keyedStream = jsonObjDS.keyBy(jsonObj -> jsonObj.getJSONObject("common").getString("mid"));

//        因为这里需要用到状态编程，所以使用富函数，普通的Filter不可以使用状态
        /**
         * 在这里我们选用什么状态呢》valueState即可，因为我们存储的是一个时间
         */
        SingleOutputStreamOperator<JSONObject> uvDS = keyedStream.filter(new RichFilterFunction<JSONObject>() {

//            时间存储围殴String了欸行
            private ValueState<String> dateState;
//            因为数据里面只有时间戳，所以我们需要进行转换
            private SimpleDateFormat simpleDateFormat;

            /**
             * 初始化
             * @param parameters
             * @throws Exception
             */
            @Override
            public void open(Configuration parameters) throws Exception {
                ValueStateDescriptor<String> valueStateDescriptor = new ValueStateDescriptor<>("date-state", String.class);


                /**
                 * Flink中的状态可以设置一个超时时间
                 */
                //设置状态的超时时间以及更新时间的方式
                StateTtlConfig stateTtlConfig = new StateTtlConfig
                        .Builder(Time.hours(24))
                        .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
                        .build();

//                设置状态过时时间
                valueStateDescriptor.enableTimeToLive(stateTtlConfig);

//              访问时间状态
                dateState = getRuntimeContext().getState(valueStateDescriptor);

                simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
            }

            @Override
            public boolean filter(JSONObject value) throws Exception {

                //取出上一条页面信息
                String lastPageId = value.getJSONObject("page").getString("last_page_id");

                //判断上一条页面是否为Null
                if (lastPageId == null || lastPageId.length() <= 0) {

                    //取出状态数据
                    String lastDate = dateState.value();

                    //取出今天的日期，也就是数据中的ts字段
                    String curDate = simpleDateFormat.format(value.getLong("ts"));

                    //判断两个日期是否相同
                    if (!curDate.equals(lastDate)) {
//                        如果不相同，那么就保留当前数据，更新状态
//                        注意，这里的状态更新，是一条数据对应一个状态，在这之前已经按照Mid进行分组了
                        dateState.update(curDate);
                        return true;
                    }
//                    else {
//                        return false;
//                    }
                }
//                上一条不是null，直接返回false，过滤掉即可
                return false;

            }
        });
```

#### 将过滤处理后的 UV写入到 Kafka的 dwm_unique_visit

```java
        //TODO 5.将数据写入Kafka
        uvDS.print();
        uvDS.map(JSONAware::toJSONString)//将json转换为String类型
                .addSink(MyKafkaUtils.getKafkaProducer(sinkTopic));
```

## DWM 层-跳出明细计算

### 需求分析与思路

#### 什么是跳出

跳出就是用户成功访问了网站的一个页面后就退出，不在继续访问网站的其它页面。而跳出率就是用跳出次数除以访问次数。

使用会话窗口的方案解决，在没有会话id的时候，如何确定这个数据是同一次会话中访问的呢？ 什么时候使用**会话窗口**，会话窗口之间的间隔时间我们自己可以确定，在我我们需要使用会话id去计算某一些指标的时候，但是这个时候没有会话id，那么我们就可以使用会话窗口解决。如果两次会话之间相隔时间较长，那么就认为是一次新的会话。

但是会话窗口可能导致丢失数据，比如数据A和数据B之间的间隔不足10秒，并且不是同一条数据，那么针对开的10秒会话窗口，两条数据分到一个窗口中，会发生丢失，针对这种情况，我们使用CEP。

关注跳出率，可以看出引流过来的访客是否能很快的被吸引，渠道引流过来的用户之间的质量对比，对于应用优化前后跳出率的对比也能看出优化改进的成果。

#### 计算跳出行为的思路

首先要识别哪些是跳出行为，要把这些跳出的访客最后一个访问的页面识别出来。那么要抓住几个特征：

该页面是用户近期访问的第一个页面，这个可以通过该页面是否有上一个页面（last_page_id）来判断，如果这个表示为空，就说明这是这个访客这次访问的第一个页面。

首次访问之后很长一段时间（自己设定），用户没继续再有其他页面的访问。

这第一个特征的识别很简单，保留 last_page_id 为空的就可以了。但是第二个访问的判断，其实有点麻烦，首先这不是用一条数据就能得出结论的，需要组合判断，要用一条存在的数据和不存在的数据进行组合判断。而且要通过一个不存在的数据求得一条存在的数据。更麻烦的他并不是永远不存在，而是在一定时间范围内不存在。那么如何识别有一定失效的组合行为呢？

最简单的办法就是 Flink 自带的 CEP 技术。这个 CEP 非常适合通过多条数据组合来识别某个事件。

用户跳出事件，本质上就是一个条件事件加一个超时事件的组合。

**CEP编程三步骤**

1. 定义模式序列
2. 将模式序列应用到流上
3. 提取匹配上的超时事件

cep可以处理乱序数据