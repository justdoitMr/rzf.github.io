## ProcessFunction API（ 底层 API） 

我们之前学习的转换算子是无法访问事件的时间戳信息和水位线信息的。而这在一些应用场景下，极为重要。例如 MapFunction 这样的 map 转换算子就无法访问时间戳或者当前事件的事件时间。 

基于此， DataStream API 提供了一系列的 Low-Level 转换算子。可以访问时间戳、 watermark 以及注册定时事件。还可以输出特定的一些事件，例如超时事件等。Process Function 用来构建事件驱动的应用以及实现自定义的业务逻辑(使用之前的window 函数和转换算子无法实现)。例如， Flink SQL 就是使用 Process Function 实现的。 

**Flink 提供了 8 个 Process Function： **

- ProcessFunction
- KeyedProcessFunction
- CoProcessFunction
- ProcessJoinFunction
- BroadcastProcessFunction
- KeyedBroadcastProcessFunction
- ProcessWindowFunction
- ProcessAllWindowFunction 

**继承关系**

![1615952074611](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/17/113437-161134.png)

### KeyedProcessFunction 

这里我们重点介绍 KeyedProcessFunction。
KeyedProcessFunction 用来操作 KeyedStream。 KeyedProcessFunction 会处理流的每一个元素，输出为 0 个、 1 个或者多个元素。所有的 Process Function 都继承自RichFunction 接口，所以都有 open()、close()和 getRuntimeContext()等方法。而KeyedProcessFunction<K, I, O>还额外提供了两个方法: 

- processElement(I value, Context ctx, Collector<O> out), 流中的每一个元素都会调用这个方法，调用结果将会放在 Collector 数据类型中输出。 Context 可以访问元素的时间戳，元素的 key，以及 TimerService 时间服务。 Context 还可以将结果输出到别的流(side outputs)。 
- onTimer(long timestamp, OnTimerContext ctx, Collector<O> out) 是一个回调函数。当之前注册的定时器触发时调用。参数 timestamp 为定时器所设定的触发的时间戳。 Collector 为输出结果的集合。 OnTimerContext 和processElement 的 Context 参数一样，提供了上下文的一些信息，例如定时器
  触发的时间信息(事件时间或者处理时间)。 

**案例**

```java
public class ProcessFunTest {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        //        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

//        先分组。然后自定义处理
//        sdss.keyBy("id").process(new MyProcess()).print();


        env.execute();
    }

    /**
     * 实现自定义的处理类
     */
    public static class MyProcess extends KeyedProcessFunction<Tuple,SensorReading,Integer>{

        ValueState<Long> tsTimer;

        @Override
        public void open(Configuration parameters) throws Exception {
            tsTimer=getRuntimeContext().getState(new ValueStateDescriptor<Long>("ts-timer",Long.class));
        }

      //处理每一个元素
        @Override
        public void processElement(SensorReading value, Context ctx, Collector<Integer> out) throws Exception {

//            输出id的长度
            out.collect(value.getId().length());

//            从context获取时间戳
            ctx.timestamp();
//            获取当前数据的键
            ctx.getCurrentKey();
//            测输出流的输出
//            ctx.output();
//          获取定时服务,注册处理时间定时器
//            ctx.timerService().registerProcessingTimeTimer();
//            事件时间定时器,根据当前的时间戳注册
            ctx.timerService().registerEventTimeTimer(value.getTempStamp()+10*1000);
//            更新时间,也就是保存时间戳
            tsTimer.update(value.getTempStamp()+10*1000);
//            删除时间定时器

//            ctx.timerService().deleteEventTimeTimer(1000);
//            如果现在想清空事件，可以直接使用时间戳
            ctx.timerService().deleteEventTimeTimer(tsTimer.value());
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<Integer> out) throws Exception {

//            在这里写定时器时间到的时候做的触发操作
            System.out.println(timestamp+"定时器时间到");

        }

        @Override
        public void close() throws Exception {
            tsTimer.clear();
        }
    }
}
```

### TimerService 和 定时器（ Timers） 

Context 和 OnTimerContext 所持有的 TimerService 对象拥有以下方法: 

- long currentProcessingTime()：返回当前处理时间 
- long currentWatermark()：返回当前 watermark 的时间戳 
- void registerProcessingTimeTimer(long timestamp) 会注册当前 key 的processing time 的定时器。当 processing time 到达定时时间时，触发 timer。 
- void registerEventTimeTimer(long timestamp) 会注册当前 key 的 event time 定时器。当水位线大于等于定时器注册的时间时，触发定时器执行回调函数。 
- void deleteProcessingTimeTimer(long timestamp) 删除之前注册处理时间定时器。如果没有这个时间戳的定时器，则不执行。 
- void deleteEventTimeTimer(long timestamp) 删除之前注册的事件时间定时器，如果没有此时间戳的定时器，则不执行。

当定时器 timer 触发时， 会执行回调函数 onTimer()。 注意定时器 timer 只能在keyed streams 上面使用。 

下面举个例子说明 KeyedProcessFunction 如何操作 KeyedStream。

- 需求：监控温度传感器的温度值，如果温度值在 10 秒钟之内(processing time)连续上升， 则报警。 

```java
public class ProcessFunTestCase {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        //        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

        map.keyBy("id")
                .process(new TempIncrease(10l))
                .print();



        env.execute();
    }

    /**
     * 实现自定义处理函数，检测一段时间内温度连续上升，输出报警
     */
    public static class TempIncrease extends KeyedProcessFunction<String,SensorReading,String>{

//        定义时间间隔，当前统计时间的间隔
        private Integer interval;

        public TempIncrease(Integer interval) {
            this.interval = interval;
        }

//        定义状态，保存上一次的温度值，定义定时器时间戳
        private ValueState<Double> lastTempState;
        private ValueState<Long> timertsState;

        @Override
        public void open(Configuration parameters) throws Exception {
            lastTempState=getRuntimeContext().getState(new ValueStateDescriptor<Double>("last-time-state",Double.class,Double.MIN_VALUE));
            timertsState=getRuntimeContext().getState(new ValueStateDescriptor<Long>("timer-ts",Long.class));

        }

        @Override
        public void processElement(SensorReading value, Context ctx, Collector<String> out) throws Exception {
//            获取状态
            Double lastTemp = lastTempState.value();
//            获取时间戳
            Long timer = timertsState.value();
            lastTempState.update(value.getTemperature());

//            如果温度上升并且没有定时器，就注册10秒后的定时器开始等待
            if(value.getTemperature()>lastTemp && timer == null){
//                计算定时器的时间戳
                long ts=ctx.timerService().currentProcessingTime()+interval*1000l;
//                        注册处理时间的定时器
                ctx.timerService().registerProcessingTimeTimer(ts);
//                更新状态
                timertsState.update(ts);
            }
//            如果温度下降，就删除定时器
            else if(value.getTemperature() < lastTemp && timer != null){
                ctx.timerService().deleteProcessingTimeTimer(timer);
                timertsState.clear();
            }

//            更新温度的状态
            lastTempState.update(value.getTemperature());
        }

//        真正定时器的触发，调用的是ontime方法

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
//            定时器触发，输出报警信息
            out.collect("传感器"+ctx.getCurrentKey()+"发出报警信息"+interval+"秒上升");
            timertsState.clear();
        }

        @Override
        public void close() throws Exception {
            lastTempState.clear();
        }
    }

}
```

### 侧输出流（ SideOutput） 

大部分的 DataStream API 的算子的输出是单一输出，也就是某种数据类型的流。除了 split 算子，可以将一条流分成多条流，这些流的数据类型也都相同。 process function 的 side outputs 功能可以产生多条流，并且这些流的数据类型可以不一样。一个 side output 可以定义为 OutputTag[X]对象， X 是输出流的数据类型。 process function 可以通过 Context 对象发射一个事件到一个或者多个 side outputs。

下面是一个示例程序，用来监控传感器温度值，将温度值低于 30 度的数据输出到 side output。 

```java
public class ProcessFunTestsideoutput {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        //        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });
        //        定义一个outputTag,用来输出低温输出流
        OutputTag<SensorReading> outputTag = new OutputTag<SensorReading>("low-stream") {
        };

        SingleOutputStreamOperator<SensorReading> highStream = map.process(new ProcessFunction<SensorReading, SensorReading>() {


            @Override
            public void processElement(SensorReading value, Context ctx, Collector<SensorReading> out) throws Exception {

//                如果当前是高温，那么就从主流输出，否则就从测输出流输出
                if(value.getTemperature() > 30){
                    out.collect(value);
                }else {
                    ctx.output(outputTag,value);
                }
            }
        });

        highStream.print("high-stream");


//        获取测输出流的输出结果
        highStream.getSideOutput(outputTag).print("low-stream");


        env.execute();
    }

}
```

### CoProcessFunction 

对于两条输入流， DataStream API 提供了 CoProcessFunction 这样的 low-level操作。 CoProcessFunction 提供了操作每一个输入流的方法: processElement1()和processElement2()。
类似于 ProcessFunction，这两种方法都通过 Context 对象来调用。这个 Context对象可以访问事件数据，定时器时间戳， TimerService，以及 side outputs。CoProcessFunction 也提供了 onTimer()回调函数。 

