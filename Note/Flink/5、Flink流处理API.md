## Flink流处理API
<!-- TOC -->

- [Flink流处理API](#flink流处理api)
  - [概述](#概述)
  - [DataSet介绍](#dataset介绍)
  - [如何写Flink程序](#如何写flink程序)
    - [流处理wordcount案例](#流处理wordcount案例)
  - [Environment](#environment)
      - [getExecutionEnvironment](#getexecutionenvironment)
      - [createLocalEnvironment](#createlocalenvironment)
      - [createRemoteEnvironment](#createremoteenvironment)
  - [Source](#source)
      - [从集合读取数据](#从集合读取数据)
      - [从文件中读取数据](#从文件中读取数据)
      - [以 kafka 消息队列的数据作为来源](#以-kafka-消息队列的数据作为来源)
      - [socket数据源](#socket数据源)
      - [自定义 Source](#自定义-source)
      - [Mysql数据源](#mysql数据源)
  - [Transform （转换）](#transform-转换)
    - [算子分类](#算子分类)
    - [基本转换](#基本转换)
      - [map()](#map)
      - [**flatMap**](#flatmap)
      - [**Filter**](#filter)
      - [sum()](#sum)
      - [reduce()](#reduce)
      - [综合案例](#综合案例)
    - [基于 KeyedStream 的转换](#基于-keyedstream-的转换)
      - [KeyBy](#keyby)
      - [滚动聚合算子（Rolling Aggregation）](#滚动聚合算子rolling-aggregation)
      - [Reduce](#reduce-1)
      - [Split 和 Select](#split-和-select)
      - [拆分和选择](#拆分和选择)
      - [Connect 和 CoMap](#connect-和-comap)
      - [Union](#union)
    - [重分区](#重分区)
      - [rebalance重平衡分区](#rebalance重平衡分区)
      - [其他分区](#其他分区)
  - [并行度](#并行度)
  - [支持的数据类型](#支持的数据类型)
    - [基础数据类型](#基础数据类型)
    - [Java 和 Scala 元组（Tuples）](#java-和-scala-元组tuples)
    - [Scala 样例类（case classes）](#scala-样例类case-classes)
    - [Java 简单对象（POJOs）](#java-简单对象pojos)
    - [其它（Arrays, Lists, Maps, Enums, 等等）](#其它arrays-lists-maps-enums-等等)
    - [为数据类型创建类型信息](#为数据类型创建类型信息)
    - [定义键值和引用字段](#定义键值和引用字段)
  - [实现 UDF 函数——更细粒度的控制流](#实现-udf-函数更细粒度的控制流)
    - [函数类（Function Classes）](#函数类function-classes)
    - [匿名函数（Lambda Functions）](#匿名函数lambda-functions)
    - [富函数（Rich Functions）](#富函数rich-functions)
  - [Sink](#sink)
    - [基于控制台的输出](#基于控制台的输出)
    - [Kafka](#kafka)
    - [Rides](#rides)
    - [Elasticsearch](#elasticsearch)
    - [JDBC 自定义 sink](#jdbc-自定义-sink)
    - [Connectors](#connectors)
      - [jdbc Connectors](#jdbc-connectors)
      - [Kafka Producer](#kafka-producer)

<!-- /TOC -->

### 概述

Flink中的流一般分为**有边界的流和无边界的流**，有边界的流就是批处理。

**API层次结构**

![1621574171278](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/135540-53159.png)

- Table Api一般是面向对象进行编程。
- DataStream是面向流和批的处理，**DataSet api是面向批处理的，现在已经被淘汰**。

入门案例使用DataSet面向批处理写，后面使用流处理和批处理一体的DataStream

**语法说明**

![1621574518154](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/132214-880288.png)

**Flink编程模型**

![1621574558280](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/132241-692408.png)

- Data Source是加载数据
- Transformations是处理数据做转换
- Data Sink是输出结果

**编程模型说明**

![1621574665781](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/132428-919301.png)

### DataSet介绍

**批处理执行环境类ExecutionEnvironment**

```java
@Public
public class ExecutionEnvironment {}
//ExecutionEnvironment是一个工具类，
//是操作DataSet批处理的一个工具类
```

**DataSet版本wordcount**

```java
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.*;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

public class Test01 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

//        2 加载数据
        DataSet<String> dataSource = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        3 处理数据
        FlatMapOperator<String, String> flatmapData = dataSource.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
//               value表示每一行数据。现在对每一行数据进行切割
                String[] s = value.split(" ");
//                遍历切割好的单词，返回单词表
                for (String str : s) {
                    out.collect(str);
                }
            }
        });
//        转换结构
        MapOperator<String, Tuple2<String, Integer>> map = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return new Tuple2<>(value, 2);
            }
        });

//        统计单词
        UnsortedGrouping<Tuple2<String, Integer>> grouped = map.groupBy(0);

//        对最终的结果进行聚合操作
        AggregateOperator<Tuple2<String, Integer>> sum = grouped.sum(1);

//        4 输出结果
        sum.print();

    }
}
```

**DataSet的api继承结构**

![1621576716798](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/135839-951899.png)

### 如何写Flink程序

下面的程序全部是基于DataStream的API。

构建一个典型的 Flink 流式应用需要以下几步: 

1. 设置执行环境。
2. 从数据源中读取一 条或多条流。**source**
3. 通过一 系列流式转换来实现应用逻辑。**transform**
4. 选择性地将结果输出到 一个或多个数据汇中。**sink**
5. 执行程序。 

**DataStream流处理类图**

![1614430440646](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/205401-717813.png)

**继承结构图**

![1614474745877](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/28/091227-963313.png)

**流处理Api的分类,Flink程序的构成**

![1614392944297](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/213317-534420.png)

**Flink程序的执行**

- Flink 程序都是通过**延迟计算** (**lazily execute**) 的方式执行(Spark程序也是懒加载方式执行)。也就是说，那些创建数据流和转换操作的 API 调用不会立即触发数据处理，而只会在执行环境中构建一个执行计划（就是最初的数据流图）。计划中包含了从环境创建的流式数据源以及应用于这些数据源之上的一系列转换。只有在调用 execute ()方法时，系统才会触发程序执行。在spark中式遇到一个行动算子会触发作业的执行。
- 构建完的计划会被转成 JobGraph 并提交至 JobManager 执行。根据执行环境类型的不同，系统可能需要将 JobGraph 发送到作为本地线程启动的JobManager 上(本地执行环境) ，也可能会将其发送到远程 JobManager 上。如果是后者，除 JobGraph 之外，我们还要同时提供包含应用所需全部类和依赖的 JAR 包。 

#### 流处理wordcount案例

```java
public class Test02 {

    /**
     * DatasTREAM既支持流处理，也支持批处理，如何区分流处理和批处理？
     * @param args
     */
    public static void main(String[] args) throws Exception {

//        1 获取执行环境,获取的是流处理的env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(2);

//        如果想做批处理，可以进行下面的设置,RuntimeExecutionMode是一个枚举类，里面有三个值
        env.setRuntimeMode(RuntimeExecutionMode.BATCH);//使用批处理
//        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);//根据数据源自动进行选择
//        env.setRuntimeMode(RuntimeExecutionMode.STREAMING);//流处理

//      读取数据
        DataStreamSource<String> data = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        操作数据
        SingleOutputStreamOperator<String> flatmapData = data.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str:s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return  Tuple2.of(value,1);
            }
        });

//        数据聚合操作
//        KeyedStream<Tuple2<String, Integer>, Tuple> sum = mapData.keyBy(0);

//        第一个参数表示输入的类型，第二个参数表示提取key 的类型
        KeyedStream<Tuple2<String, Integer>, String> sum = mapData.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });

//        SingleOutputStreamOperator<Tuple2<String, Integer>> sum1 = sum.sum(1);

        sum.print();
//        启动程序并且等待结束
        env.execute();
//        在后续的开发中，把一切的数据源都看做是流数据

    }
}
```

> DataStream是流处理和批处理一体的api，原来的dataSet批处理api已经废弃，可以通过控制流处理api的参数进行批处理操作。

### Environment 

##### getExecutionEnvironment 

创建一个执行环境，表示当前执行程序的上下文。 **如果程序是独立调用的，则此方法返回本地执行环境；如果从命令行客户端调用程序以提交到集群，则此方法返回此集群的执行环境**，也就是说， getExecutionEnvironment 会根据查询运行的方式决定返回什么样的运行环境，是最常用的一种创建执行环境的方式。 

**批处理执行环境ExecutionEnvironment**

```java
public class ExecutionEnvironment {}
```

![1614393681958](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/104122-239154.png)

**流处理执行环境StreamExecutionEnvironment**

```java
@Public
public class StreamExecutionEnvironment {}
//StreamExecutionEnvironment是流处理的执行环境
```

![1614393750522](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/104443-618574.png)

**案例**

```java
public class EnvTest {

    public static void main(String[] args) {
//        获取批处理执行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
//        获取流处理的执行环境
        StreamExecutionEnvironment senv = StreamExecutionEnvironment.getExecutionEnvironment();
    }
}
```

##### createLocalEnvironment 

返回本地流式执行环境，需要在调用时指定默认的并行度。 

```java
public class CLENV {

    public static void main(String[] args) {

//        设置并行度是2，获取的是流处理的本地执行环境
        LocalStreamEnvironment lsenv = StreamExecutionEnvironment.createLocalEnvironment(2);
    }
}
```

##### createRemoteEnvironment 

返回集群执行环境，远程流式执行环境，将 Jar 提交到远程服务器。需要在调用时指定 JobManager的 IP 和端口号，并指定要在集群中运行的 Jar 包。 

```java
public class RMENV {

    public static void main(String[] args) {

//        创建集群环境，设置主机，端口号和并行度，获取的是流处理的远程执行环境
        StreamExecutionEnvironment rmenv = StreamExecutionEnvironment.createRemoteEnvironment("hadoop100", 888, 2);
    }
}
```

从上面我们可以看到，本地环境类，集群环境类都是StreamExecutionEnvironment的子类。

### Source 

##### 从集合读取数据 

```java
public class ReadFromCollection {

    public static void main(String[] args) throws Exception{

//        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
//        设置全局的并行度 ,设置1的话就是单线程执行
        env.setParallelism(1);

//        从集合中读取数据
        DataStream<SensorReading> ds = env.fromCollection(Arrays.asList(
                new SensorReading("sensor_1", 1547718199L, 35.8),
                new SensorReading("sensor_6", 1547718201L, 15.4),
                new SensorReading("sensor_7", 1547718202L, 6.7),
                new SensorReading("sensor_10", 1547718205L, 38.1)
        ));

//        从元素中读取数据
        DataStreamSource<Integer> ids = env.fromElements(1, 2, 3, 4, 5);

//        打印输出得到的结果
        ds.print("data");


//        如果在这里设置并行度是1，那么就会按照顺序输出，这里设置的并行度只是设置当前任务的并行度，也可以设置全局的并行度
//        ids.print("int").setParallelism(1);
        ids.print("int");
//        执行任务
        env.execute("作业一");
    }
}
```

**案例2**

```java
public class Test03 {

    public static void main(String[] args) throws Exception {

//        TODO env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
//        如果没有做设置的话，默认进行的是流处理

//        TODO source
        /**
         * 基于集合的source
         */
//        fromElements():可变参数
        DataStreamSource<String> ds = env.fromElements("spark", "hadoop", "flink");

//        基于集合创建source
//        fromCollection()：参数是一个集合
        DataStreamSource<String> dsc = env.fromCollection(Arrays.asList("spark", "hadoop", "flink"));

//         基于1-100生成一个集合
        DataStreamSource<Long> dsg = env.generateSequence(1, 100);
        DataStreamSource<Long> dsg1 = env.fromSequence(1, 100);

//        TODO sink
        ds.print();
        dsc.print();
        dsg.print();
        dsg1.print();

//        TODO execute()
        env.execute();
    }
}
```

##### 从文件中读取数据

```java
public class ReadFromFile {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        打印输出
        sdss.print();

//        执行程序
        env.execute();
    }
}
```

**案例二**

```java
public class Test04 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于文件或者文件夹读取
          */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<String> dsf = env.readTextFile("D:\\soft\\idea\\work\\work08\\hmlen\\src\\main\\resources\\words.txt");
//        读取文件夹下面的所有文件
        DataStream<String> dsFile = env.readTextFile("D:\\soft\\idea\\work");
        env.execute();

    }
}
```

##### 以 kafka 消息队列的数据作为来源 

需要引入 kafka 连接器的依赖 

```java
<dependency>
    <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-kafka-0.11_2.12
        </artifactId>
    <version>1.10.1</version>- //flink的版本
</dependency>
// 0.11是kafka的版本，2.12是scala的版本
```

**案例**

```java
public class ReadFromKafka {

    public static void main(String[] args) throws Exception {

        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();


        // kafka 配置项
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "consumer-group");
        properties.setProperty("key.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset", "latest");

//        添加数据源

        DataStreamSource<String> sdss = env.addSource(new FlinkKafkaConsumer011<String>("sensor", new SimpleStringSchema(), properties));
//sensor代表kafka组
        
        sdss.print();

//        执行程序
        env.execute();

    }
}

//案例er
public class Test14 {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);

//        添加kafka数据源
//        准备配置文件
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");//生产者组的ip
        props.setProperty("group.id", "flink");//消费者组
        props.setProperty("auto.offset.reset","latest");//设置offset,如果有offset记录，那么就从offset记录开始消费，没有的话从最新的消息消费
        props.setProperty("flink.partition-discovery.interval-millis","5000");//会开启一个后台线程每隔5s检测一下Kafka的分区情况，动态分区的检测
        props.setProperty("enable.auto.commit", "true");//自动提交
        props.setProperty("auto.commit.interval.ms", "2000");//自动提交时间间隔，会提交到默认的主题
//        创建kafka source
        FlinkKafkaConsumer<String> flink_kafka = new FlinkKafkaConsumer<>("flink_kafka", new SimpleStringSchema(), props);
//      使用source
        DataStreamSource<String> source = env.addSource(flink_kafka);

        env.execute();
    }
}
```

##### socket数据源

- 启动socket：nc -lk port

```java
public class Test05 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于socket读取文件
          */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        基于socket读取数据
        DataStreamSource<String> socket = env.socketTextStream("DESKTOP-56SGKD3", 9999);

        SingleOutputStreamOperator<String> words = socket.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str : s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapS = words.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return new Tuple2<>(value, 1);
            }
        });

        KeyedStream<Tuple2<String, Integer>, Tuple> keyByDtat = mapS.keyBy(1);

        keyByDtat.print();

        env.execute();
    }
}
public class Test02 {

    /**
     * DatasTREAM既支持流处理，也支持批处理，如何区分流处理和批处理？
     * @param args
     */
    public static void main(String[] args) throws Exception {

//        1 获取执行环境,获取的是流处理的env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

//        如果想做批处理，可以进行下面的设置,RuntimeExecutionMode是一个枚举类，里面有三个值
        env.setRuntimeMode(RuntimeExecutionMode.BATCH);//使用批处理
//        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);//根据数据源自动进行选择
//        env.setRuntimeMode(RuntimeExecutionMode.STREAMING);//流处理

//      读取数据
        DataStreamSource<String> data = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        操作数据
        SingleOutputStreamOperator<String> flatmapData = data.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str:s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return  Tuple2.of(value,1);
            }
        });

//        数据聚合操作
//        KeyedStream<Tuple2<String, Integer>, Tuple> sum = mapData.keyBy(0);

//        第一个参数表示输入的类型，第二个参数表示提取key 的类型,根据返回的key对数据进行分组
        KeyedStream<Tuple2<String, Integer>, String> sum = mapData.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });

//        SingleOutputStreamOperator<Tuple2<String, Integer>> sum1 = sum.sum(1);

        sum.print();
//        启动程序并且等待结束
        env.execute();
//        在后续的开发中，把一切的数据源都看做是流数据

    }
}
```

##### 自定义 Source 

可以添加多种数据源

![1621927806658](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/153010-218291.png)

除了以上的 source 数据来源，我们还可以自定义 source。需要做的，只是传入一个 SourceFunction 就可以。具体调用如下： 

```java
//我们可以添加自己定义的数据源
DataStream<SensorReading> dataStream = env.addSource( new MySensor());
```

**自定义数据源**

![1621752852591](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/152941-131078.png)

**代码演示**

```java
//自定义数据源，需要去实现SourceFunction接口，接口中的泛型表示产生的数据类型
//此接口有两个方法，
public static class MySeneorFun implements SourceFunction<SensorReading>{
//定义一个标志位，控制数据的产生
        private boolean running=true;


        public void run(SourceContext<SensorReading> ctx) throws Exception {

//            定义随机数发生器
            Random random = new Random();

//            设置10个传感器温度的初始值
            HashMap<String, Double> temp = new HashMap<String,Double>();

            for (int i = 0; i <10; i++) {
//                nextGaussian按照高斯分布生成随机数，也就是正泰分布
                temp.put("sensor_"+(i+1),random.nextGaussian()*20);
            }

//            产生数据
            while (running){
//                使用collect方法生成我们的额数据
                for (String id:temp.keySet()) {
//                    使得温度分布更加随机
                    Double ranTem=temp.get(id)+random.nextGaussian();
                    temp.put(id,ranTem);
                    ctx.collect(new SensorReading(id,System.currentTimeMillis(),temp.get(id)));
                }
//              控制输出的频率
                Thread.sleep(3);
            }

        }

        public void cancel() {

            running=false;
        }
    }
```

**案例**

```java
public class MyUDF {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        //env.setParallelism(1);

//        自定义从源中读取数据
//        只要自定义的source执行起来之后，就会调用run方法
        DataStreamSource<SensorReading> sdss = env.addSource(new MySeneorFun());

//        打印输出
        sdss.print();

//        执行程序
        env.execute();

    }

    public static class MySeneorFun implements SourceFunction<SensorReading>{
//定义一个标志位，控制数据的产生
        private boolean running=true;

        public void run(SourceContext<SensorReading> ctx) throws Exception {

//            定义随机数发生器
            Random random = new Random();

//            设置10个传感器温度的初始值
            HashMap<String, Double> temp = new HashMap<String,Double>();

            for (int i = 0; i <10; i++) {
//                nextGaussian按照高斯分布生成随机数，也就是正泰分布
                temp.put("sensor_"+(i+1),random.nextGaussian()*20);
            }

//            产生数据
            while (running){
//                使用collect方法生成我们的额数据
                for (String id:temp.keySet()) {
//                    使得温度分布更加随机
                    Double ranTem=temp.get(id)+random.nextGaussian();
                    temp.put(id,ranTem);
                    ctx.collect(new SensorReading(id,System.currentTimeMillis(),temp.get(id)));
                }
//              控制输出的频率
                Thread.sleep(3);
            }

        }

        public void cancel() {

            running=false;
        }
    }
}
```

**案例二**

```java
public class Test06 {
    /**
     * Author itcast
     * Desc
     * 需求
     * 每隔1秒随机生成一条订单信息(订单ID、用户ID、订单金额、时间戳)
     * 要求:
     * - 随机生成订单ID(UUID)
     * - 随机生成用户ID(0-2)
     * - 随机生成订单金额(0-100)
     * - 时间戳为当前系统时间
     * <p>
     * API
     * 一般用于学习测试,模拟生成一些数据
     * Flink还提供了数据源接口,我们实现该接口就可以实现自定义数据源，不同的接口有不同的功能，分类如下：
     * SourceFunction:非并行数据源(并行度只能=1)
     * RichSourceFunction:多功能非并行数据源(并行度只能=1)
     * ParallelSourceFunction:并行数据源(并行度能够>=1)
     * RichParallelSourceFunction:多功能并行数据源(并行度能够>=1)--后续学习的Kafka数据源使用的就是该接口
     */


    public static void main(String[] args) throws Exception {
        /**
         * 基于socket读取文件
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);
        //2.Source
        DataStream<Order> orderDS = env
                .addSource(new MyOrderSource())//添加自定义的数据源
                .setParallelism(2);//可以自己设置并行度

        //3.Transformation

        //4.Sink
        orderDS.print();//每一次打印两条数据，因为并行度是2
        //5.execute
        env.execute();
    }
//    Data注释可以直接使用get set方法
    @Data
    @AllArgsConstructor //添加全参构造器
    @NoArgsConstructor //添加空参数构造器
    public static class Order {
        private String id;
        private Integer userId;
        private Integer money;
        private Long createTime;
    }

    public static class MyOrderSource extends RichParallelSourceFunction<Order> {
        private Boolean flag = true;

//        执行并且生成数据
        @Override
        public void run(SourceContext<Order> ctx) throws Exception {
            Random random = new Random();
            while (flag) {
                Thread.sleep(1000);
//                订单id
                String id = UUID.randomUUID().toString();
//                用户id
                int userId = random.nextInt(3);
                int money = random.nextInt(101);
                long createTime = System.currentTimeMillis();
                ctx.collect(new Order(id, userId, money, createTime));
            }
        }

        //取消任务/执行cancle命令的时候执行
        @Override
        public void cancel() {
            flag = false;
        }
    }
}
```

**lombok的使用**

添加依赖

```java
 <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.2</version>
    <scope>provided</scope>
</dependency>
```

安装插件

![1621754220104](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/30/193319-447764.png)

**SourceFunction接口**

```java
public interface SourceFunction<T> extends Function, Serializable {

	
  //用来产生我们的数据的方法，我们可以在这里写产生数据的方法
	void run(SourceContext<T> ctx) throws Exception;

  //控制产生数据的方法
	void cancel();

	@Public // Interface might be extended in the future with additional methods.
	interface SourceContext<T> {

		void collect(T element);

		@PublicEvolving
		void collectWithTimestamp(T element, long timestamp);

		@PublicEvolving
		void emitWatermark(Watermark mark);

		@PublicEvolving
		void markAsTemporarilyIdle();

		Object getCheckpointLock();

		void close();
	}
}

```

**类图**

![1621754486366](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/152129-113162.png)

自定义source的时候有多重source，可以根据自己的选择继承使用。

##### Mysql数据源

- 需求:

实际开发中,经常会实时接收一些数据,要和MySQL中存储的一些规则进行匹配,那么这时候就可以使用Flink自定义数据源从MySQL中读取数据

那么现在先完成一个简单的需求:

从MySQL中实时加载数据

要求MySQL中的数据有变化,也能被实时加载出来

```java
public class Test07 {
    /**
     * Desc
     * 需求:
     * 实际开发中,经常会实时接收一些数据,要和MySQL中存储的一些规则进行匹配,那么这时候就可以使用Flink自定义数据源从MySQL中读取数据
     * 那么现在先完成一个简单的需求:
     * 从MySQL中实时加载数据
     * 要求MySQL中的数据有变化,也能被实时加载出来
     */


    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        自定义数据源，并且设置并行度是1
        DataStreamSource<Student> data = env.addSource(new MySqlSource()).setParallelism(1);

        data.print();

        env.execute();

    }


    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Student{
        private Integer id;
        private String name;
        private Integer age;
    }

    public static class MySqlSource extends RichParallelSourceFunction<Student>{

        private boolean flag=true;
        private Connection connection=null;
        private PreparedStatement ps=null;
        private ResultSet rs=null;

        @Override
        public void open(Configuration parameters) throws Exception {
//            这个方法只执行一次，适合开启资源，比如这里打开数据库连接
//            /获取数据库连接地址
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
            String sql="select * from t_student";
//            执行sql语句
            ps = connection.prepareStatement(sql);
        }

        @Override
        public void run(SourceContext<Student> ctx) throws Exception {
            while (flag){
//                每5秒钟进行一次查询
                rs=ps.executeQuery();
                while (rs.next()){
                    int id = rs.getInt("id");
                    String name = rs.getString("name");
                    int age = rs.getInt("age");
                    ctx.collect(new Student(id,name,age));
                }
                TimeUnit.SECONDS.sleep(5);
            }

        }

        @Override
        public void cancel() {
//            接受到cancel命令时候，取消数据的生成
            flag=false;
        }

        @Override
        public void close() throws Exception {
//            关闭资源
            if(connection != null)
                connection.close();
            if(ps != null)
                ps.close();
            if(rs != null){
                rs.close();
            }
        }
    }
}
```

**测试数据**

```java
CREATE TABLE `t_student` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(255) DEFAULT NULL,
    `age` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

INSERT INTO `t_student` VALUES ('1', 'jack', '18');
INSERT INTO `t_student` VALUES ('2', 'tom', '19');
INSERT INTO `t_student` VALUES ('3', 'rose', '20');
INSERT INTO `t_student` VALUES ('4', 'tom', '19');
INSERT INTO `t_student` VALUES ('5', 'jack', '18');
INSERT INTO `t_student` VALUES ('6', 'rose', '20');
```

### Transform （转换）

流式转换以一个或多个数据流为输入，井将它们转换成一个或多个输出流。完成一个 DataStream API 程序在本质上可以归结为:**通过组合不同的转换来创建一个满足应用逻辑的 Dataflow 图**。 

大多数流式转换都是基于**用户自定义函**数来完成的。这些函数封装了用户应用逻辑，指定了输入流的元素将如何转换为输出流的元素。函数可以通过实现某个特定转换的接口类来定义 ，函数接口规定了用户需要实现的转换方法

#### 算子分类

DataStream API的转换分为四类: 

1. 作用于**单个事件**的基本转换。 
2. 针对**相同键值事件**的 KeyedStream 转换。
3. 将多条**数据流合并为一 条或将一条数据流拆分成多条**流的转换。
4. 对流中的事件进行**重新组织**的分发转换 。

![1621852286978](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/183132-651854.png)

**操作概览**

![1621852330281](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/183220-69874.png)

整体来说，流式数据上的操作可以分为四类。

- 第一类是对于单条记录的操作，比如筛除掉不符合要求的记录（Filter 操作），或者将每条记录都做一个转换（Map 操作）
- 第二类是对多条记录的操作。比如说统计一个小时内的订单总成交量，就需要将一个小时内的所有订单记录的成交量加到一起。为了支持这种类型的操作，就得通过 Window 将需要的记录关联到一起进行处理
- 第三类是对多个流进行操作并转换为单个流。例如，多个流可以通过 Union、Join 或 Connect 等操作合到一起。这些操作合并的逻辑不同，但是它们最终都会产生了一个新的统一的流，从而可以进行一些跨流的操作。
- 最后， DataStream 还支持与合并对称的拆分操作，即把一个流按一定规则拆分为多个流（Split 操作），每个流是之前流的一个子集，这样我们就可以对不同的流作不同的处理。

#### 基本转换

基本转换会单独处理每个事件 ，这意味着每条输出记录都由单条输入记录所生成。常见的基本转换函数有:简单的值转换，记录拆分或过滤等 ，map()，flatMap()，Filter()，这三个是简单的转换算子

##### map()

通过调用 DataStream.map() 方法可以指定 map 转换产生 一 个新的DataStream。该转换将每个到来的事件传给一个用户自定义的映射器( user defined mapper) ，后者针对每个输入只会返回 一个(可能类型发生改变的)输出事件。图所示的 map 转换会将每个方形输入转换为圆形。 

![1614424509615](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/213417-791672.png)

**MapFunction接口**

```java
//mapFunction是一个接口，只需要实现其中的方法就可以
//MapFunction 的两个类型参数分别是输入事件和输出事件的类型 ，它们可以通过 MapFuncti on 接口来指定。 i亥接口的 map() 方怯将每个输入事件转换为一个输出事件:
public interface MapFunction<T, O> extends Function, Serializable {
    O map(T var1) throws Exception;
}
//map()方法是针对每一个元素进行的操作
```

**案例**

```java
public class TransformBase01 {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");


//        对读进来的数据做基本的转换
//     1， map():把传进来的数据转换为长度输出
//        MapFunction的泛型，String表示输入时候的泛型，Integer表示输出时候的泛型
        DataStream<Integer>mapStream=sdss.map(new MapFunction<String, Integer>() {
//            使用一个匿名类
//            重写map方法进行转换
            public Integer map(String s) throws Exception {
//                在这里做具体的转换,对输出的每一条数据都做转换
                return s.length();
            }
        });

//        打印输出
        //sdss.print();

//        mapStream.print();
//        执行程序
        env.execute();

    }
}
```

##### **flatMap**

- flatMap 转换类似于 map ，但它可以对每个输入事件产生零个、 一个或多个输出事件 。事实上， flatMap 转换可以看做是 filter 和 map 的泛化，它能够实现后两者的操作。 flatMap 转换会针对每个到来事件应用一个函数 。 
- 对应 的FlatMapFunction 定义了 flatMap() 方法，你可以在其中通过向 Collector 对象传递数据的方式返回零个、一个或多个事件作为结果 :。
- flatMap:将集合中的每个元素变成一个或多个元素,并返回扁平化之后的结果

![1621852739453](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/183901-464396.png)

**FlatMapFunction 接口**

```java
@FunctionalInterface
@Public
// T:输入元素类型
// O:输入元素类型
// Collector：用于返回多个结果
public interface FlatMapFunction<T, O> extends Function, Serializable {
    void flatMap(T var1, Collector<O> var2) throws Exception;
}
//对每一个集合进行扁平化处理后返回结果
```

**案例**

```java
public class TransformBase01 {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");


//        对读进来的数据做基本的转换
//      2 flatMap()转换
//                按照逗号对字符串进行切分,输入是String类型，输出是String类型
        DataStream <String>mapFlatStream=sdss.flatMap(new FlatMapFunction<String, String>() {
//            collector是用来输出数据
            public void flatMap(String s, Collector<String> collector) throws Exception {
                String[] split = s.split(",");
                for (String value:split) {
                    collector.collect(value);
                }
            }
        });

//        打印输出
        //sdss.print();

//        mapStream.print();
//            mapFlatStream.print();
        mapFliter.print();
//        执行程序
        env.execute();

    }
}
```

##### **Filter**

- filter 转换利用一个作用在流中每条输入事件上的布尔条件来决定事件的去留:如果返回值为 true ，那么它会保留输入事件并将其转发到输出，否则它会把事件丢弃。通过调用 DataStream.filter() 方住可以指定 filter 转换产生一个数据类型不变的 DataStream。 
- 可以利用 FilterFunction 接口或 Lambda 函数来实现定义布尔条件的函数。FilterFunction 接口的类型为输入流的类型，它的 filter() 方法会接收一个输入事件，返回一个布尔值: 对数据做过滤操作。

**FilterFunction接口**

```java
@FunctionalInterface
@Public
//T:输入元素的类型
public interface FilterFunction<T> extends Function, Serializable {
    boolean filter(T var1) throws Exception;
}
//满足过滤条件返回true，就保留数据
```

下面filter 操作仅保留了白色方块。 

![1614424678962](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/220449-508125.png)

```java
public class TransformBase01 {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        对读进来的数据做基本的转换
//        3 filter()过滤操作
        DataStream <String>mapFliter=sdss.filter(new FilterFunction<String>() {
            public boolean filter(String s) throws Exception {
//                返回false说明过滤掉，返回true说明保留
                return s.startsWith("sensor_1");
            }
        });

//        打印输出
        //sdss.print();

//        mapStream.print();
//            mapFlatStream.print();
        mapFliter.print();
//        执行程序
        env.execute();

    }
}
```

##### sum()

sum:按照指定的字段对集合中的元素进行求和

```java
public class Test02 {

    /**
     * DatasTREAM既支持流处理，也支持批处理，如何区分流处理和批处理？
     * @param args
     */
    public static void main(String[] args) throws Exception {

//        1 获取执行环境,获取的是流处理的env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

//        如果想做批处理，可以进行下面的设置,RuntimeExecutionMode是一个枚举类，里面有三个值
        env.setRuntimeMode(RuntimeExecutionMode.BATCH);//使用批处理
//        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);//根据数据源自动进行选择
//        env.setRuntimeMode(RuntimeExecutionMode.STREAMING);//流处理

//      读取数据
        DataStreamSource<String> data = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        操作数据
        SingleOutputStreamOperator<String> flatmapData = data.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str:s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return  Tuple2.of(value,1);
            }
        });

//        数据聚合操作
//        KeyedStream<Tuple2<String, Integer>, Tuple> sum = mapData.keyBy(0);

//        第一个参数表示输入的类型，第二个参数表示提取key 的类型,根据返回的key对数据进行分组
        KeyedStream<Tuple2<String, Integer>, String> sum = mapData.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });
    //      sum()函数做聚合操作
        SingleOutputStreamOperator<Tuple2<String, Integer>> sum1 = sum.sum(1);

        sum.print();
//        启动程序并且等待结束
        env.execute();
//        在后续的开发中，把一切的数据源都看做是流数据
    }
}
```

##### reduce()

reduce:对集合中的元素进行聚合

![1621853250817](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/185045-98488.png)

**案例**

```java
public class Test02 {

    /**
     * DatasTREAM既支持流处理，也支持批处理，如何区分流处理和批处理？
     * @param args
     */
    public static void main(String[] args) throws Exception {

//        1 获取执行环境,获取的是流处理的env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

//        如果想做批处理，可以进行下面的设置,RuntimeExecutionMode是一个枚举类，里面有三个值
        env.setRuntimeMode(RuntimeExecutionMode.BATCH);//使用批处理
//        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);//根据数据源自动进行选择
//        env.setRuntimeMode(RuntimeExecutionMode.STREAMING);//流处理

//      读取数据
        DataStreamSource<String> data = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        操作数据
        SingleOutputStreamOperator<String> flatmapData = data.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                String[] s = value.split(" ");
                for (String str:s) {
                    out.collect(str);
                }
            }
        });

        SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = flatmapData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return  Tuple2.of(value,1);
            }
        });

//        数据聚合操作
//        KeyedStream<Tuple2<String, Integer>, Tuple> sum = mapData.keyBy(0);

//        第一个参数表示输入的类型，第二个参数表示提取key 的类型,根据返回的key对数据进行分组
        KeyedStream<Tuple2<String, Integer>, String> sum = mapData.keyBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });
//          sum()函数做聚合操作
//        SingleOutputStreamOperator<Tuple2<String, Integer>> sum1 = sum.sum(1);


//        使用reduce对最后的结果进行聚合操作
        SingleOutputStreamOperator<Tuple2<String, Integer>> res = sum.reduce(new ReduceFunction<Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                return Tuple2.of(value1.f0,value1.f1+value2.f1);
            }
        });
        res.print();
//        启动程序并且等待结束
        env.execute();
//        在后续的开发中，把一切的数据源都看做是流数据

    }
}
```

##### 综合案例

```java
public class Test08 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

//        2 加载数据
        DataSet<String> dataSource = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        3 处理数据
        FlatMapOperator<String, String> flatmapData = dataSource.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
//               value表示每一行数据。现在对每一行数据进行切割
                String[] s = value.split(" ");
//                遍历切割好的单词，返回单词表
                for (String str : s) {
                    out.collect(str);
                }
            }
        });
//        对数据进行过滤操作
        FilterOperator<String> filterData = flatmapData.filter(new FilterFunction<String>() {
            @Override
            public boolean filter(String value) throws Exception {
                return !value.equals("TMD");//如果是TMD那么就返回false，表示过滤掉
            }
        });


//        转换结构
        MapOperator<String, Tuple2<String, Integer>> map = filterData.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return new Tuple2<>(value, 1);
            }
        });

//        统计单词
        UnsortedGrouping<Tuple2<String, Integer>> groupData = map.groupBy(new KeySelector<Tuple2<String, Integer>, String>() {
            @Override
            public String getKey(Tuple2<String, Integer> value) throws Exception {
                return value.f0;
            }
        });

//        对最终的结果进行聚合操作
//        AggregateOperator<Tuple2<String, Integer>> res = groupData.sum(1);

//        也可以使用reduce进行聚合操作
        ReduceOperator<Tuple2<String, Integer>> result = groupData.reduce(new ReduceFunction<Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                return Tuple2.of(value1.f0, value1.f1 + value2.f1);
            }
        });

//        4 输出结果
        result.print();

    }
}
```

#### 基于 KeyedStream 的转换 

- 很多应用需要将事件按照某个属性**分组**后再进行处理。作为 DataStream API中 一类特殊的 DataStream ， KeyedStream 抽象可以从逻辑上将事件按照键值分配到多条独立的子流 中 ，其继承于DataStream。

![1614435119343](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/221221-910361.png)

```java
//可以看到，KeyedStream还是继承于DataStream
public class KeyedStream<T, KEY> extends DataStream<T> { }
```

- **在流中没有直接的聚合操作，必须先进行分组操作，然后在做聚合操作。**
- 作用于 KeyedStream 的状态化转换可以对当前处理事件的键值所对应上下文中的状态进行读写。这意味着所有键值相同的事件可以访问相同的状态，因此它们可以被一并处理 。 
- KeyedStream 也支持使用你之见看到过的 map 、f1atMap 和 filter 等转换进行处理 

##### KeyBy 

- keyBy 转换通过指定键值的方式将一个 DataStream 转化为 KeyedStream。**流中的事件会根据各自键值被分到不同的分区**，这样一来，有着相同键值的事件一定会在后续算子的同一个任务上处理。虽然键值不同的事件也可能会在同一个任务上处理，但任务函数所能访问的键值分区状态始终会被约束在当前事件键值的范围内。 
- keyBy () 方法接收一个用来指定分区键值(可以是多个)的参数，返回 一个KeyedStream。 
- 流处理中没有groupBy,而是keyBy

**KeySelector接口**

```java
@Public
@FunctionalInterface
public interface KeySelector<IN, KEY> extends Function, Serializable {
//返回分区的id
	KEY getKey(IN value) throws Exception;
}
```

如下图将黑色时间划分到一个分区中，其他时间划分到另一个分区中。

![1614425447441](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/220624-639789.png)

DataStream → KeyedStream：逻辑地将一个流拆分成不相交的分区，**每个分区包含具有相同 key 的元素，在内部以 hash 的形式实现的**。一个分区内也可能有不同的key,因为是按照hash进行映射的。

**代码说明**

```java
public class TransformBase_keyby {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(4);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        转换为sensorreading类型
        DataStream<SensorReading> map = sdss.map(new MapFunction<String, SensorReading>() {
            public SensorReading map(String s) throws Exception {

                String[] s1 = s.split(",");

                return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
            }
        });

//        按照第一个位置处的元素进行分区，在这里是按照id对数据进行分区操作
        KeyedStream<SensorReading, Tuple> resgb = map.keyBy("id");
        resgb.writeAsText("D:\\soft\\idea\\work\\work08\\data2");


//        执行程序
        env.execute();
    }
}
//会按照指定的id进行分组操作
```

##### 滚动聚合算子（Rolling Aggregation） 

滚动聚合转换作用于 KeyedStream 上，它将生成一个包含聚合结果(例如求和、最小值、最大值等)的 DataStream。滚动聚合算子会对每一个遇到过的键值保存一个聚合结果。每当有新事件到来，该算子都会更新相应的聚合结果， 并将其 以事件的形式发送出 去。 滚动聚合虽然不需要用户自定义函数，但需要接收一个用于指定聚合目标字段的参数。 

这下面这些算子可以针对 KeyedStream 的每一个支流做滚动聚合。

- sum()：滚动计算输入流中指定宇段的和。 
- max()：滚动计算输入流中指定字段的最大值。 
- min()：滚动计算输入流中指定字段的最小值。 
- minBy()：滚动计算输入流中迄今为止最小值，返回该值所在事件 。 
- maxBy()：滚动计算输入流中迄今为止最大值，返回该值所在事件。 

> 无法将多个滚动聚合方法组合使用， 每次只能计算一个。 

**案例**

```java
public class Transform_Rolling_Aggregation {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        转换为sensorreading类型
//        DataStream<SensorReading> map = sdss.map(new MapFunction<String, SensorReading>() {
//            public SensorReading map(String s) throws Exception {
//
//                String[] s1 = s.split(",");
//
//                return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
//            }
//        });

//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
        });

//       按照bean的字段进行分组
//        public class KeyedStream<T, KEY> extends DataStream<T>：<T, KEY>T表示输入类型，KEY表示分组字段的类型
//        但是返回的key类型是tuple类型，这里和源码有关
//        public KeyedStream<T, Tuple> keyBy(String... fields) {
//            return keyBy(new Keys.ExpressionKeys<>(fields, getType()));
//        }
        KeyedStream<SensorReading, Tuple> keyedStream = map.keyBy("id");
//        传入的匿名函数，最后返回String类型
//        KeyedStream<SensorReading, String> sensorReadingObjectKeyedStream = map.keyBy(data ->
//            data.getId()
//        );
//        分组聚合，取当前最大的温度值，按照字段，取某一个字段的最大值
//        SingleOutputStreamOperator<SensorReading> resultStream = keyedStream.max("temperature");
//        滚动聚合，获取某一个最大值
        SingleOutputStreamOperator<SensorReading> resultStream = keyedStream.maxBy("temperature");

//        打印输出
       // keyedStream.print();

        resultStream.print();
//        执行程序
        env.execute();
    }
}
//输出结果
SensorReading{id='sensor_1', tempStamp=1547718199, temperature=35.8}
SensorReading{id='sensor_6', tempStamp=1547718201, temperature=15.4}
SensorReading{id='sensor_7', tempStamp=1547718202, temperature=6.7}
SensorReading{id='sensor_10', tempStamp=1547718205, temperature=38.1}
SensorReading{id='sensor_1', tempStamp=1547718198, temperature=36.8}
SensorReading{id='sensor_1', tempStamp=1547718198, temperature=36.8}
SensorReading{id='sensor_1', tempStamp=1547718209, temperature=39.8}
//可以看到，时间戳没有进行更新
```

> 滚动聚合算子会为每个处理过的键值维持一个状态。由干这些状态不会被自动清理，所以该算子只能用于键值域有限的流。 

##### Reduce 

reduce 转换是滚动聚合转换的泛化。它将一个 ReduceFunction 应用在一个 KeyedStream 上，每个到 来事件都会和 reduce 结果进行一次组合，从而产生 一个新的 DataStream  ，reduce 转换不会改变数据类型， 因此输出流的类型会 永远和输入流保持一致。 

KeyedStream → DataStream：一个分组数据流的聚合操作，合并当前的元素和上次聚合的结果，产生一个新的值，返回的流中包含每一次聚合的结果，而不是只返回最后一次聚合的最终结果。 

**ReduceFunction接口**

```JAVA
@Public
@FunctionalInterface
public interface ReduceFunction<T> extends Function, Serializable {
//只有一个泛型类型，也就是说数据类型不能够变，和滚动聚合一样
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
  //value1是之前聚合出来的状态，value2是当前的最新的值
	T reduce(T value1, T value2) throws Exception;
}
```

**案例**

```java
public class Transform_Rolling_Reducer {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

//        转换为sensorreading类型
//        DataStream<SensorReading> map = sdss.map(new MapFunction<String, SensorReading>() {
//            public SensorReading map(String s) throws Exception {
//
//                String[] s1 = s.split(",");
//
//                return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
//            }
//        });

//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));
        });
        KeyedStream<SensorReading, Tuple> keyedStream = map.keyBy("id");

//        reduce聚合操作，取最大的温度值和最新的时间戳
       DataStream resultStream= keyedStream.reduce(new ReduceFunction<SensorReading>() {
            @Override
            public SensorReading reduce(SensorReading value1, SensorReading value2) throws Exception {

                return new SensorReading(value1.getId(),value2.getTempStamp(),Math.max(value1.getTemperature(),value2.getTemperature()));

            }
        });
       resultStream.print();



//        执行程序
        env.execute();
    }
}
//输出结果
SensorReading{id='sensor_1', tempStamp=1547718199, temperature=35.8}
SensorReading{id='sensor_6', tempStamp=1547718201, temperature=15.4}
SensorReading{id='sensor_7', tempStamp=1547718202, temperature=6.7}
SensorReading{id='sensor_10', tempStamp=1547718205, temperature=38.1}
SensorReading{id='sensor_1', tempStamp=1547718198, temperature=36.8}
SensorReading{id='sensor_1', tempStamp=1547718183, temperature=36.8}
SensorReading{id='sensor_1', tempStamp=1547718209, temperature=39.8}
//和上面的区别是时间戳进行更新操作
```

map,flatMap,filter,reducer都是操作一条数据流

下面的算子可以操作多条数据流=>多流转换算子

##### Split 和 Select 

split 转换是 union 转换的逆操作。它将输入流分割成**两条或多条类型和输入流相同**的输出流。每一个到来的事件都可以被发往零个、 一个或多个输出流。因此， split 也可以用来过滤或复制事件。 

DataStream.split() 方法接收一个 OutputSelector ，它用来定义如何将数据流的元素分配到不同的命名输出 ( named outpu t) 中。 OutputSelector 中定 义的 select() 方也会在每个输入事件到来时被调用，并随即返回 一个 java.lang.lterable[String] 对象 。 针对某记录所返回的一 系列 String 值指定了该记录需要被发往哪些输出流。 

DataStream.split() 方法会返回 一个 SplitStream 对象，它提供的 select()方法可以让我们通过指定输出名称的方式从 SplitStream 中选择一条或多条流 。 

按照一定的特征，把数据做一个划分，split和select要配合使用。

**Split**

![1614471868219](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/30/194818-219338.png)

DataStream → SplitStream：根据某些特征把一个 DataStream 拆分成两个或者多个 DataStream。 

**Select**

![1614471898093](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/30/194819-379399.png)

SplitStream→ DataStream：从一个 SplitStream 中获取一个或者多个DataStream。 

**OutputSelector接口**

```java
@PublicEvolving
public interface OutputSelector<OUT> extends Serializable {
	/**
	 * Method for selecting output names for the emitted objects when using the
	 * {@link SingleOutputStreamOperator#split} method. The values will be
	 * emitted only to output names which are contained in the returned
	 * iterable.
	 *
	 * @param value
	 *            Output object for which the output selection should be made.
	 */
	Iterable<String> select(OUT value);
}
```

**需求**

需求： 传感器数据按照温度高低（以 30 度为界），拆分成两个流。 

```java
public class TransformBase_multTrans {

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

//        分流操作，按照温度30为界限，分成两条流
//        参数表示选择器
//        上层api已经被弃用，所以要使用底层的api
        SplitStream<SensorReading> split = map.split(new OutputSelector<SensorReading>() {
//            方法返回一个可以迭代类型的集合
            @Override
            public Iterable<String> select(SensorReading value) {
//                给每一条记录打标签
                return (value.getTemperature()>30)? Collections.singletonList("high"):Collections.singletonList("low");
            }
        });

//        按照标签筛选
//        DataStream<SensorReading> high = split.select("low");
//        筛选多个标签
        DataStream<SensorReading> high = split.select("low","high");

        high.print();

//        执行程序
        env.execute();
    }
}
```

##### 拆分和选择

- Split就是将一个流分成多个流
- Select就是获取分流后对应的数据

> 注意：split函数已过期并移除

- Side Outputs：可以使用process方法对流中数据进行处理，并针对不同的处理结果将数据收集到不同的OutputTag中

 对流中的数据按照奇数和偶数进行分流，并获取分流后的数据

```java
public class Test09 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //2.Source
        DataStreamSource<Integer> data = env.fromElements(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        OutputTag<Integer> oddTag = new OutputTag<Integer>("奇数", TypeInformation.of(Integer.class));
        OutputTag<Integer> even = new OutputTag<Integer>("偶数",TypeInformation.of(Integer.class));

        SingleOutputStreamOperator<Integer> res = data.process(new ProcessFunction<Integer, Integer>() {
            @Override
            public void processElement(Integer value, Context ctx, Collector<Integer> out) throws Exception {
//                out收集完毕的数据还是放在一起，ctx可以将数据放在不同的outputtag
                if(value %2 == 0){
                    ctx.output(even,value);
                }else {
                    ctx.output(oddTag,value);
                }
            }
        });
//        获取数据
        DataStream<Integer> oddData = res.getSideOutput(oddTag);
        DataStream<Integer> evenData = res.getSideOutput(even);
        oddData.print("奇数");
        evenData.print("偶数");
        env.execute();
    }
}
```



##### Connect 和 CoMap 

**connect**

DataStream.connect() 方法接收一个 DataStream 并返回 一个 ConnectedStream 对象，该对象表示两个联结起来 (connected) 的流:，ConnectedStreams 对象提供了 map() 和 flatMap ()方法，它们分别接收一个CoMapFunction 和一个 CoFlatMapFunction 作为参数，两个函数都是以两条输入流的类型外加输出流的类型作为其类型参数，它们为两条输入流定义了各自的处理方怯。 mapl() 和 flatMapl() 用来处理第一条输入流的事件， map2() 和 flatMap2() 用来处理第二条输入流的事件: 

![1614473695309](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/28/214907-132600.png)

DataStream,DataStream → ConnectedStreams：连接两个保持他们类型的数据流，两个数据流被 Connect 之后，只是被放在了一个同一个流中，内部依然保持各自的数据和形式不发生任何变化，两个流相互独立。 

**CoMap,CoFlatMap**

![1614473760250](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/144255-313871.png)

ConnectedStreams → DataStream：作用于 ConnectedStreams 上，功能与 map和 flatMap 一样，对 ConnectedStreams 中的每一个 Stream 分别进行 map 和 flatMap处理。 

**CoMapFunction接口**

```java
@Public
public interface CoMapFunction<IN1, IN2, OUT> extends Function, Serializable {

	/**
	 * This method is called for each element in the first of the connected streams.
	 *
	 * @param value The stream element
	 * @return The resulting element
	 * @throws Exception The function may throw exceptions which cause the streaming program
	 *                   to fail and go into recovery.
	 */
  //第一条流的逻辑
	OUT map1(IN1 value) throws Exception;

	/**
	 * This method is called for each element in the second of the connected streams.
	 *
	 * @param value The stream element
	 * @return The resulting element
	 * @throws Exception The function may throw exceptions which cause the streaming program
	 *                   to fail and go into recovery.
	 */
  //第二条流的逻辑
	OUT map2(IN2 value) throws Exception;
}
//上面两条流的逻辑相互独立
```

**合流**

```java
public class TransformBase_multTrans_connect {

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

//        分流操作，按照温度30为界限，分成两条流
//        参数表示选择器
//        上层api已经被弃用，所以要使用底层的api
        SplitStream<SensorReading> split = map.split(new OutputSelector<SensorReading>() {
//            方法返回一个可以迭代类型的集合
            @Override
            public Iterable<String> select(SensorReading value) {
//                给每一条记录打标签
                return (value.getTemperature()>30)? Collections.singletonList("high"):Collections.singletonList("low");
            }
        });

//        按照标签筛选
//        DataStream<SensorReading> high = split.select("low");
//        筛选多个标签
        DataStream<SensorReading> high =split.select("high");
        DataStream<SensorReading> low =split.select("low");

//        使用connect，将高温流转换为元祖类型，与低温流合并之后，输出状态信息
        SingleOutputStreamOperator<Tuple2<String, Double>> dataStream = high.map(new MapFunction<SensorReading, Tuple2<String, Double>>() {
            @Override
            public Tuple2<String, Double> map(SensorReading value) throws Exception {
                return new Tuple2<>(value.getId(), value.getTemperature());
            }
        });

//        接下来做链接操作,基于dataStream流和高温流做链接操作
//        public <R> ConnectedStreams<T, R> connect(DataStream<R> dataStream)
//        T是调用的流的类型，R是被调用流的类型
        ConnectedStreams<Tuple2<String, Double>, SensorReading> connectStreams = dataStream.connect(low);

//        三个参数代表的含义
//        Tuple2<String, Double>:高温流的输入类型
//        SensorReading:低温流的输入类型
//        Object：最终输出的流的类型
        SingleOutputStreamOperator<Object> result = connectStreams.map(new CoMapFunction<Tuple2<String, Double>, SensorReading, Object>() {

//            因为返回类型是object,所以两个方法返回的类型要兼容

            @Override
            public Object map1(Tuple2<String, Double> value) throws Exception {

//                对高温流，返回一个三元组
//                元祖里面的元素叫做f0,f1,f2
                return new Tuple3<>(value.f0, value.f1, "high tem warning");
            }

            @Override
            public Object map2(SensorReading value) throws Exception {
//                对于低温流，返回正常信息
                return new Tuple2<>(value.getId(), "normal");
            }
        });

//        high.print();
            result.print();

//        执行程序
        env.execute();
    }
}
//输出结果
(sensor_1,35.8,high tem warning)
(sensor_6,normal)
(sensor_1,36.8,high tem warning)
(sensor_7,normal)
(sensor_1,36.2,high tem warning)
(sensor_10,normal)
(sensor_1,39.8,high tem warning)

```

上面的缺点是不可以链接多条流操作。

> CoMapFunction 函数无法选择从哪条流读取数据 和 CoFlatMapFunction 内方法的调用顺序无法控制 。一旦对应流中有事件到来，系统就需要调用相应的方泣。 

##### Union 

- DataStream.union() 方法可以合并两条或多条类型相同的 DataStream ，生成一个新的类型相同的 DataStream。这样后续的转换操作就可以对所有输入流中的元素统一处理。 
- union可以进行多条流的合并，**但是多条流的数据类型必须一样**，也就是union只可以合并同类型的流，connect只能合并两条流，但是流的类型可以不一样。也就是说connent可以合并一样类型的流和不一样类型的流。
- 可以发现，到目前为止，所有的转换全部是基于DataStream流进行的，因为基础的数据结构是DataStream类型。

![1614474499237](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/28/210810-646817.png)

union 执行过程中，来自两条流的事件会以 FIFO (先进先出)的方式合井，其顺序无法得到任何保证。此外， union **算子不会对数据进行去重**，每个输入消息都会被发往下游算子。 

DataStream → DataStream：对两个或者两个以上的 DataStream 进行 union 操作，产生一个包含所有 DataStream 元素的新 DataStream。 

```java
public class TransformBase_multTrans_connect {

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

//        分流操作，按照温度30为界限，分成两条流
//        参数表示选择器
//        上层api已经被弃用，所以要使用底层的api
        SplitStream<SensorReading> split = map.split(new OutputSelector<SensorReading>() {
//            方法返回一个可以迭代类型的集合
            @Override
            public Iterable<String> select(SensorReading value) {
//                给每一条记录打标签
                return (value.getTemperature()>30)? Collections.singletonList("high"):Collections.singletonList("low");
            }
        });

//        按照标签筛选
//        DataStream<SensorReading> high = split.select("low");
//        筛选多个标签
        DataStream<SensorReading> high =split.select("high");
        DataStream<SensorReading> low =split.select("low");

//        使用connect，将高温流转换为元祖类型，与低温流合并之后，输出状态信息
        SingleOutputStreamOperator<Tuple2<String, Double>> dataStream = high.map(new MapFunction<SensorReading, Tuple2<String, Double>>() {
            @Override
            public Tuple2<String, Double> map(SensorReading value) throws Exception {
                return new Tuple2<>(value.getId(), value.getTemperature());
            }
        });

//        接下来做链接操作,基于dataStream流和高温流做链接操作
//        public <R> ConnectedStreams<T, R> connect(DataStream<R> dataStream)
//        T是调用的流的类型，R是被调用流的类型
        ConnectedStreams<Tuple2<String, Double>, SensorReading> connectStreams = dataStream.connect(low);

//        三个参数代表的含义
//        Tuple2<String, Double>:高温流的输入类型
//        SensorReading:低温流的输入类型
//        Object：最终输出的流的类型
        SingleOutputStreamOperator<Object> result = connectStreams.map(new CoMapFunction<Tuple2<String, Double>, SensorReading, Object>() {

//            因为返回类型是object,所以两个方法返回的类型要兼容

            @Override
            public Object map1(Tuple2<String, Double> value) throws Exception {

//                对高温流，返回一个三元组
//                元祖里面的元素叫做f0,f1,f2
                return new Tuple3<>(value.f0, value.f1, "high tem warning");
            }

            @Override
            public Object map2(SensorReading value) throws Exception {
//                对于低温流，返回正常信息
                return new Tuple2<>(value.getId(), "normal");
            }
        });

//        union可以连接多条流，但是流的类型必须一样，从底层源码可以看到public final DataStream<T> union(DataStream<T>... streams)
//        从源码可以看到，泛型都是T类型,union还可以联合多条流
        DataStream<SensorReading> unionStream = high.union(low);
        unionStream.print();

//        high.print();
//            result.print();

//        执行程序
        env.execute();
    }
}
```

union和connect的区别

1. Union 之前两个流的类型必须是一样， Connect 可以不一样，在之后的 coMap中再去调整成为一样的 
2. Connect 只能操作两个流， Union 可以操作多个 
3. 两个DataStream经过connect之后被转化为ConnectedStreams，ConnectedStreams会对两个流的数据应用不同的处理方法，且双流之间可以共享状态。

![1621845625755](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/144750-807202.png)

#### 重分区

类似于Spark中的repartition,但是功能更强大,可以直接解决数据倾斜

Flink也有数据倾斜的时候，比如当前有数据量大概10亿条数据需要处理，在处理过程中可能会发生如图所示的状况，出现了数据倾斜，其他3台机器执行完毕也要等待机器1执行完毕后才算整体将任务完成；

![1621850834398](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/144750-273094.png)

所以在实际的工作中，出现这种情况比较好的解决方案就是rebalance(内部使用round robin方法将数据均匀打散)

![1621850867161](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/144749-230543.png)

##### rebalance重平衡分区

```java
public class Test10 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //2.Source
        DataStreamSource<Long> source = env.fromSequence(0, 100);
        env.setParallelism(2);

//        下面操作相当于将数据进行重新随机的分配，有可能发生数据的倾斜
        SingleOutputStreamOperator<Long> filterData = source.filter(new FilterFunction<Long>() {
            @Override
            public boolean filter(Long value) throws Exception {
                return value > 10;
            }
        });

        SingleOutputStreamOperator<Tuple2<Integer, Integer>> mapData = filterData
                .rebalance()
                .map(new RichMapFunction<Long, Tuple2<Integer, Integer>>() {

            @Override
            public Tuple2<Integer, Integer> map(Long value) throws Exception {
                int indexOfThisSubtask = getRuntimeContext().getIndexOfThisSubtask();//获取子任务id，也就是分区编号
                return Tuple2.of(indexOfThisSubtask, 1);
            }
        });

//        查看每一个分区中有多少个元素
        SingleOutputStreamOperator<Tuple2<Integer, Integer>> countData = mapData.keyBy(item -> item.f0).sum(1);

        countData.print();

        env.execute();
    }
}
```

##### 其他分区

![1621925336453](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/144857-364414.png)

最后一个可以理解为自定义分区。

**案例演示**

```java
public class Test11 {

    public static void main(String[] args) throws Exception {

//        DataSet api使用
//        1 准备环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

//        2 加载数据
        DataSet<String> linesDS = env.readTextFile("D:\\soft\\idea\\work\\work08\\src\\main\\resources\\words.txt");
//        3 处理数据
        FlatMapOperator<String, Tuple2<String, Integer>> tupleDS = linesDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });

        //3.Transformation
//        DataStream<Tuple2<String, Integer>> result1 = tupleDS.global();数据发送到第一个task
//        DataStream<Tuple2<String, Integer>> result2 = tupleDS.broadcast();把tupleDS中的数据广播到下游所有的task
//        DataStream<Tuple2<String, Integer>> result3 = tupleDS.forward();上下游并行度一样的时候，一个对一个的发送
//        DataStream<Tuple2<String, Integer>> result4 = tupleDS.shuffle();打乱随机轮流的发送
//        PartitionOperator<Tuple2<String, Integer>> result5 = tupleDS.rebalance();轮流分配，重平衡
//        DataStream<Tuple2<String, Integer>> result6 = tupleDS.rescale();本地轮流分配
//        下面是自定义分区器
        PartitionOperator<Tuple2<String, Integer>> result7 = tupleDS.partitionCustom(new Partitioner<String>() {
            @Override
            public int partition(String key, int numPartitions) {
                return key.equals("hello") ? 0 : 1;
            }
        }, t -> t.f0);

        //4.sink
        //result1.print();
        //result2.print();
        //result3.print();
        //result4.print();
        //result5.print();
        //result6.print();
        result7.print();

        //5.execute
        env.execute();

    }
}
```

### 并行度

每个算子都会产生一个或多个并行任务。每个任务负责处理算子的部分输入流。算子井行化任务的数目称为该算子的井行度。它决定了算子处理的井行化程度以及能够处理的数据规模。 

算子的并行度可以在执行环境级别或单个算子级别进行控制。默认情况下，应用内所有算子的并行度都会被设置为应用执行环境的井行度。而环境的并行度(即所有算子的默认井行度) 则会根据应用启动时所处的上下文自动初始化。如果应用是在一个本地执行环境中运行， 并行度会设置为 CPU 的线程数目。如果应用是提交到 Flink 集群运行，那么除非提交客户端明确指定 ,否则环境的并行度将设置为集群的默认并行度.

一般情况下，最好将算子并行度设置为随环境默认并行度变化的值。这样就可以通过提交客户端来轻易调整井行度，从而实现应用的扩缩容 。 你可以按照下面的示例来访问环境的默认并行度 : 

```java
//设置环境的并行度
env.setParallelism(1);
```

你可以通过显式指定的方式来覆盖算子的默认并行度。下面的示例中，数据源算子会以环境默认并行度执行， map 转换的任务数是数据糠的两倍，数据汇操作固定以两个并行任务执行: 

```java
val env = StreamExecutionEnvironment.getExecutionEnvironment
// 获取默认并行度
val defaultP = env.getParallelism
//数据源以默认并行度运行
val result: = env.addSource(new CustomSource)
//设置 map 的并行度为默认并行度的两倍
.map(new MyMapper).setParallelism(defaultP * 2)
// print 数据汇的并行度固定为 2
.print().setParallelism(2)
```







### 支持的数据类型

- Flink 流应用程序处理的是以**数据对象**表示的事件流。 所以在 Flink 内部， 我们需要能够处理这些对象。它们需要被序列化和反序列化，以便通过网络传送它们；或者从状态后端、检查点和保存点读取它们。为了有效地做到这一点， Flink 需要明确知道应用程序所处理的数据类型。 Flink 使用类型信息的概念来表示数据类型，并为每个数据类型生成特定的序列化器、反序列化器和比较器。 
- Flink 还具有一个类型提取系统，该系统分析函数的输入和返回类型，以自动获取类型信息，从而获得序列化器和反序列化器。但是，在某些情况下，例如 lambda函数或泛型类型，需要显式地提供类型信息，才能使应用程序正常工作或提高其性能 
- Flink 支持 Java 和 Scala 中所有常见数据类型。使用最广泛的类型有以下几种。 

#### 基础数据类型

Flink 支持所有的 Java 和 Scala 基础数据类型， Int, Double, Long, String, … 包括包装类

#### Java 和 Scala 元组（Tuples） 

java中的元组

![1614486838998](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/28/123400-213651.png)

#### Scala 样例类（case classes） 

```java
 case class Person(name: String, age: Int)
        val persons: DataStream[Person] = env.fromElements(
            Person("Adam", 17),
            Person("Sarah", 23) )
        persons.filter(p => p.age > 18)
```

#### Java 简单对象（POJOs） 

```java
public class Person {
  //注意：属性需要声明为public类型
            public String name;
            public int age;
  //还必须有一个空参数的构造方法
            public Person() {}
            public Person(String name, int age) {
                this.name = name;
                this.age = age;
            }
        }
        DataStream<Person> persons = env.fromElements(
                new Person("Alex", 42),
                new Person("Wendy", 23));
```

#### 其它（Arrays, Lists, Maps, Enums, 等等） 

Flink 对 Java 和 Scala 中的一些特殊目的的类型也都是支持的，比如 Java 的ArrayList， HashMap， Enum 等等。

#### 为数据类型创建类型信息

Flink 类型系统的核心类是 Typelnformation ，它为系统生成序列化器和比
较器提供了必要的信息。 

```java
@Public
public abstract class TypeInformation<T> implements Serializable {}
```

#### 定义键值和引用字段

1. 根据字段的位置引用
2. 使用字段表达式
3. 使用键值选择器

**使用键值选择器**

```java
//        使用键值选择器，输入类型是sensorreading类型，输出的键值是元祖类型
        KeyedStream<SensorReading, List<String>> id = map.keyBy(new KeySelector<SensorReading, List<String>>() {
            @Override
            public List<String> getKey(SensorReading value) throws Exception {
                return Collections.singletonList(value.getId());
            }
        });
```

### 实现 UDF 函数——更细粒度的控制流 

#### 函数类（Function Classes） 

Flink 暴露了所有 udf 函数的接口(实现方式为接口或者抽象类)。例如MapFunction, FilterFunction, ProcessFunction 等等。 

下面例子实现了 FilterFunction 接口： 

```java
DataStream<String> flinkTweets = tweets.filter(new FlinkFilter());
public static class FlinkFilter implements FilterFunction<String> {
    @Override
    public boolean filter(String value) throws Exception {
        return value.contains("flink");
    }
}
```

还可以将函数实现成匿名类 

```java
DataStream<String> flinkTweets = tweets.filter(new FilterFunction<String>() {
@Override
public boolean filter(String value) throws Exception {
	return value.contains("flink");
	}
});
```

我们 filter 的字符串"flink"还可以当作参数传进去。 

```java
    DataStream<String> tweets = env.readTextFile("INPUT_FILE ");
    DataStream<String> flinkTweets = tweets.filter(new KeyWordFilter("flink"));

public static class KeyWordFilter implements FilterFunction<String> {
    private String keyWord;

    KeyWordFilter(String keyWord) {
        this.keyWord = keyWord;
    }

    @Override
    public boolean filter(String value) throws Exception {
        return value.contains(this.keyWord);
    }
}
```

#### 匿名函数（Lambda Functions） 

```java
DataStream<String> tweets = env.readTextFile("INPUT_FILE");
DataStream<String> flinkTweets = tweets.filter( tweet -> tweet.contains("flink") );
```

#### 富函数（Rich Functions） 

“富函数”是 DataStream API 提供的一个函数类的接口， 所有 Flink 函数类都有其 Rich 版本。 它与常规函数的不同在于，可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能，富函数的命名规则是以 Rich 开头，后面跟着普通转换函数的名字  

- RichMapFunction
- RichFlatMapFunction
- RichFilterFunction 

Rich Function 有一个生命周期的概念。 典型的生命周期方法有： 

- open()方法是 rich function 的初始化方法，当一个算子例如 map 或者 filter被调用之前 open()会被调用。，open() :是富函数中的初始化方法。它在每个任务首次调用转换方能(如fi lter 或 m ap) 前调用 一 次。 open() 通常用于那些只需进行一 次的设置工作。 
- close () 作为函数的终止方洁，会在每个任务最后一次调用转换方法后调用一次。它通常用于清理和释放资源。  
- getRuntimeContext()方法提供了函数的 RuntimeContext 的一些信息，例如函数执行的并行度，任务的名字，以及 state 状态 

```java
public class Transform_Rolling_RechFunction {

    public static void main(String[] args) throws Exception {

        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是4,open和close会输出四次，因为有4个分区，每一个分区会对应的有一个MyRichFunction类的实例对象
        env.setParallelism(4);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");
//        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line -> {
            String[] s1 = line.split(",");

            return new SensorReading(s1[0], new Long(s1[1]), new Double(s1[2]));
        });

//        DataStream<Tuple2<String, Integer>> resultStream = map.map(new MyMapper());
        DataStream<Tuple2<String, Integer>> resultStream = map.map(new MyRichFunction());

        resultStream.print();

//        执行程序
        env.execute();


    }

    public static class MyMapper implements MapFunction<SensorReading, Tuple2<String, Integer>> {

//        每一个数据到来后，都会调用map方法

        @Override
        public Tuple2<String, Integer> map(SensorReading value) throws Exception {
            return new Tuple2<>(value.getId(), value.getId().length());
        }
    }

    //    实现自定义的富函数类
    public static class MyRichFunction extends RichMapFunction<SensorReading, Tuple2<String, Integer>> {

        @Override
        public void open(Configuration parameters) throws Exception {
//            一般做一些初始化工作,定义状态，获取建立数据库的链接
            System.out.println("open method");

        }

        @Override
        public Tuple2<String, Integer> map(SensorReading value) throws Exception {
            return new Tuple2<>(value.getId(), value.getId().length());
        }

        @Override
        public void close() throws Exception {
//            一般是关闭链接，清空状态的收尾工作
            System.out.println("close");
        }
    }
}
```

37

### Sink 

Flink 没有类似于 spark 中 foreach 方法，让用户进行迭代的操作。虽有对外的输出操作都要利用 Sink 完成。最后通过类似如下方式完成整个任务最终输出操作。 

```java
stream.addSink(new MySink(xxxx))
```

官方提供了一部分的框架的 sink。除此以外，需要用户自定义实现 sink。 

**官方提供的连接器**

![1614559803465](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/01/085237-536905.png)

**第三方提供的连接器**

![1614559978906](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/151149-524971.png)

#### 基于控制台的输出

- ds.print 直接输出到控制台
- ds.printToErr() 直接输出到控制台,用红色
- ds.writeAsText("本地/HDFS的path",WriteMode.OVERWRITE).setParallelism(1)

 **注意:**

- 在输出到path的时候,可以在前面设置并行度,如果
  - 并行度>1,则path为目录
  - 并行度=1,则path为文件名

**案例**

```java
public class Test12 {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);

        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);
        DataStreamSource<String> dataSource = env.readTextFile("D:\\soft\\idea\\work\\work08\\hmlen\\src\\main\\resources\\words.txt");
        dataSource.print("输出标示");
        dataSource.print();
        dataSource.printToErr();//控制台上红颜色输出标示
        env.execute();
    }
}
```

#### Kafka 

**添加依赖包**

```java
<dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka-0.11_2.12</artifactId>
            <version>1.10.1</version>
</dependency>
```

**案例**

```java
public class Transform_sink {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");


//        使用lambda表达式
        DataStream<String> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2])).toString();
        });

//        new一个kafka的生产者
        map.addSink(new FlinkKafkaProducer011<String>("localhost:9092","sinktest",new SimpleStringSchema()));

//        执行程序
        env.execute();
    }
}
```

#### Rides

**依赖**

```java
<dependency>
<groupId>org.apache.bahir</groupId>
<artifactId>flink-connector-redis_2.11</artifactId>
<version>1.0</version>
</dependency>
```

**案例**

使用建造者模式

```java
public class SinkTestRides {

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


//        定义jedis链接配置

        FlinkJedisPoolConfig config = new FlinkJedisPoolConfig.Builder().setHost("localhost").setPort(6379).build();
//        创建一个连接器
        map.addSink(new RedisSink<>(config,new MyRidesMapper()));
//        执行程序
        env.execute();

    }

    public static class MyRidesMapper implements RedisMapper<SensorReading> {

//定义数据保存到rides的命令，以哈希表形式存储,hset sensor_temp id temputer
//        SENSOR_TEMP:表名
        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.HSET,"SENSOR_TEMP");
        }

        @Override
        public String getKeyFromData(SensorReading data) {
//写入数据库的键
            return data.getId();
        }

        @Override
        public String getValueFromData(SensorReading data) {
            
//            写入数据库的值
            double temperature = data.getTemperature();
            return temperature+"";
        }
    }
}
```

#### Elasticsearch 

**依赖**

```java
<dependency>
<groupId>org.apache.flink</groupId>
<artifactId>flink-connector-elasticsearch6_2.12</artifactId>
<version>1.10.1</version>
</dependency>
```

**案例**

```java
public class SinkTestES {

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

//        定义es的链接配置
        ArrayList<HttpHost> host=new ArrayList();
        host.add(new HttpHost("localhost",9200));

        map.addSink(new ElasticsearchSink.Builder<SensorReading>(host,new MyEsSinkFun()).build());


//        执行程序
        env.execute();

    }

    public static class MyEsSinkFun implements ElasticsearchSinkFunction<SensorReading>{

        @Override
        public void process(SensorReading sensorReading, RuntimeContext runtimeContext, RequestIndexer requestIndexer) {
//            实现写入数据的source
            HashMap datasource = new HashMap<String,String>();
            datasource.put("id",sensorReading.getId());
            datasource.put("temp",sensorReading.getTemperature());
            datasource.put("ts",sensorReading.getTempStamp());

//            创建请求作为向es发起的写入命令
            IndexRequest source = Requests.indexRequest().index("sensor").type("readingdata").source(datasource);

//            发送请求，使用indeser发发送
            requestIndexer.add(source);

        }
    }
}
```

#### JDBC 自定义 sink 

可以定义多种sink进行输出

![1621927643858](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/152727-856427.png)

**引入依赖**

```java
<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<version>5.1.44</version>
</dependency>
```

**自定义sink输出**

```java
//    实现MyJDBCFun
    public static class MyJDBCFun extends RichSinkFunction<SensorReading>{
        Connection connection=null;

//        定义预编译器
    PreparedStatement insertStatement=null;
    PreparedStatement updatestat=null;

    @Override
    public void open(Configuration parameters) throws Exception {
//        声明周期方法，创建一个链接
//        获取一个链接
        connection= DriverManager.getConnection("jdbc:mysql://localhost:3306/test","root","root");
//        初始化
        insertStatement=connection.prepareStatement("insert into temp_sensor(id,temp) values(?,?)");
        updatestat=connection.prepareCall("update temp_sensor set temp=? where id=?");

    }

    @Override
    public void invoke(SensorReading value, Context context) throws Exception {
//        这个方法每来一条数据就会创建一个链接，所以这样很消耗资源，所以我们把这个链接放在open方法中
//        每来一条语句，就调用链接执行sql,设置占位符
        updatestat.setDouble(1,value.getTemperature());
        updatestat.setString(2,value.getId());
//        执行语句
        updatestat.execute();
//        如果没有更新成功，就执行插入操作
        if(updatestat.getUpdateCount() ==0){
            insertStatement.setString(1,value.getId());
            insertStatement.setDouble(2,value.getTemperature());
            insertStatement.execute();
        }
    }

    @Override
    public void close() throws Exception {
//        关闭所有的资源
        insertStatement.close();
        updatestat.close();
        connection.close();
    }
}
```

**案例**

```java
public class SinkTestJDBC {

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

        map.addSink(new MyJDBCFun());
//        执行程序
        env.execute();

    }
}
```

#### Connectors

##### jdbc Connectors

```java
public class Test13 {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Stu> dataSource = env.fromElements(new Stu(2, "xiaorui", 25));

        dataSource.addSink(JdbcSink.sink("insert into t_student values (null,?,?)",
                (ps,value)->
                {
                    ps.setString(1,value.getName());
                    ps.setInt(2,value.getAge());
                }, new JdbcConnectionOptions.JdbcConnectionOptionsBuilder()
                .withUrl("jdbc:mysql://localhost:3306/test")
                .withUsername("root")
                .withPassword("root")
                .withDriverName("com.mysql.jdbc.Driver")
                .build()));

        env.execute();

    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
class Stu{
    private int id;
    private String name;
    private int age;
}
```

##### Kafka Producer

Flink 里已经提供了一些绑定的 Connector，例如 kafka source 和 sink，Es sink 等。读写kafka、es、rabbitMQ 时可以直接使用相应 connector 的 api 即可，虽然该部分是 Flink 项目源代码里的一部分，但是真正意义上不算作 Flink 引擎相关逻辑，并且该部分没有打包在二进制的发布包里面。所以在提交 Job 时候需要注意， job 代码
jar 包中一定要将相应的 connetor 相关类打包进去，否则在提交作业时就会失败，提示找不到相应的类，或初始化某些类异常。

**添加依赖**

```java
 <dependency>
           <groupId>org.apache.flink</groupId>
           <artifactId>flink-connector-kafka_2.11</artifactId>
           <version>1.12.3</version>
</dependency>
```

**参数设置**

以下参数都必须/建议设置上

1. 订阅的主题
2. 反序列化规则
3. 消费者属性-集群地址
4. 消费者属性-消费者组id(如果不设置,会有默认的,但是默认的不方便管理)
5. 消费者属性-offset重置规则,如earliest/latest...
6. 动态分区检测(当kafka的分区数变化/增加时,Flink能够检测到!)
7. 如果没有设置Checkpoint,那么可以设置自动提交offset,后续学习了Checkpoint会把offset随着做Checkpoint的时候提交到Checkpoint和默认主题中

**参数设置**

![1621941296269](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/191501-976107.png)

![1621941335200](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/191548-220370.png)

![1621941395341](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/191636-455476.png)

实际的生产环境中可能有这样一些需求，比如：

- 场景一：有一个 Flink 作业需要将五份数据聚合到一起，五份数据对应五个 kafka topic，随着业务增长，新增一类数据，同时新增了一个 kafka topic，如何在不重启作业的情况下作业自动感知新的 topic。
- 场景二：作业从一个固定的 kafka topic 读数据，开始该 topic 有 10 个 partition，但随着业务的增长数据量变大，需要对 kafka partition 个数进行扩容，由 10 个扩容到 20。该情况下如何在不重启作业情况下动态感知新扩容的 partition？

针对上面的两种场景，首先需要在构建 FlinkKafkaConsumer 时的 properties 中设置 flink.partition-discovery.interval-millis 参数为非负值，表示开启动态发现的开关，以及设置的时间间隔。此时 FlinkKafkaConsumer 内部会启动一个单独的线程定期去 kafka 获取最新的 meta 信息。

针对场景一，还需在构建 FlinkKafkaConsumer 时，topic 的描述可以传一个正则表达式描述的 pattern。每次获取最新 kafka meta 时获取正则匹配的最新 topic 列表。

针对场景二，设置前面的动态发现参数，在定期获取 kafka 最新 meta 信息时会匹配新的 partition。为了保证数据的正确性，新发现的 partition 从最早的位置开始读取。

![1621943840230](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/144354-168624.png)

> 注意:
>
> 开启 checkpoint 时 offset 是 Flink 通过状态 state 管理和恢复的，并不是从 kafka 的 offset 位置恢复。在 checkpoint 机制下，作业从最近一次checkpoint 恢复，本身是会回放部分历史数据，导致部分数据重复消费，Flink 引擎仅保证计算状态的精准一次，要想做到端到端精准一次需要依赖一些幂等的存储系统或者事务操作。

**案例**

```java
public class Test15 {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);

        //        添加kafka数据源
//        准备配置文件
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");//生产者组的ip
        props.setProperty("group.id", "flink");//消费者组
        props.setProperty("auto.offset.reset","latest");//设置offset,如果有offset记录，那么就从offset记录开始消费，没有的话从最新的消息消费
        props.setProperty("flink.partition-discovery.interval-millis","5000");//会开启一个后台线程每隔5s检测一下Kafka的分区情况，动态分区的检测
        props.setProperty("enable.auto.commit", "true");//自动提交
        props.setProperty("auto.commit.interval.ms", "2000");//自动提交时间间隔，会提交到默认的主题
//        创建kafka source
        FlinkKafkaConsumer<String> flink_kafka = new FlinkKafkaConsumer<>("flink_kafka", new SimpleStringSchema(), props);
//      使用source
        DataStreamSource<String> source = env.addSource(flink_kafka);
//        在这里可以基于kafka的数据源source做数据的清洗工作，然后在输出处理过后的数据

//          下面添加输出数据源
//        准备配置文件
        Properties properties = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");//生产者组的ip
//        创建kafka消费者
        FlinkKafkaProducer<String> flink_kafka_sink = new FlinkKafkaProducer<>("flink_kafka", new SimpleStringSchema(), properties);
//      添加kafka sink
        DataStreamSink<String> stringDataStreamSink = source.addSink(flink_kafka_sink);
      
        env.execute();

    }
}
```

