
<!-- TOC -->

- [Flink状态管理](#flink状态管理)
    - [无状态计算](#无状态计算)
    - [有状态计算](#有状态计算)
    - [Flink中的状态分类](#flink中的状态分类)
    - [Flink中的状态](#flink中的状态)
    - [算子状态（Operator State）](#算子状态operator-state)
      - [算子状态数据结构](#算子状态数据结构)
    - [键控状态（Keyed State）](#键控状态keyed-state)
      - [键控状态数据结构](#键控状态数据结构)
      - [键控状态的使用](#键控状态的使用)
      - [练习](#练习)
    - [状态后端（State Backends）](#状态后端state-backends)
      - [状态后端分类](#状态后端分类)
      - [状态后端配置](#状态后端配置)
    - [案例说明](#案例说明)

<!-- /TOC -->

## Flink状态管理

![20211116093249](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211116093249.png)

目前Flink已经可以做到状态的自动管理。

#### 无状态计算

![1623139219723](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/144846-62949.png)

#### 有状态计算

![1623139250027](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/16/090016-926526.png)

#### Flink中的状态分类

- State
  - ManagerState-开发中推荐使用,Flink自动管理和优化，支持多种数据结构和类型
    - KeyState：只能在keyedStream上面使用，支持多种数据结构
    - operatorState：可以在所有数据流中使用，支持liststate结构
  - Raw State，完全由用户自己管理，只支持byte[]数组，只能在自定义operator中使用。
    - operatorState

![1623139879988](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/161125-490915.png)

开发中大多使用自动管理机制。

![1623140791468](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/144931-463035.png)

#### Flink中的状态

![1614671349533](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/101838-752218.png)

- **由一个任务维护，并且用来计算某个结果的所有数据，都属于这个任务的状态**
- 可以认为状态就是一个**本地变量**，可以被任务的业务逻辑访问
- Flink会进行状态管理，包括状态一致性、故障处理以及高效存储和访问，以便开发人员可以专注于应用程序的逻辑
- Flink中的状态是和任务绑定在一起的，可以认为是任务的一个**本地变量**。
- 在Flink中，**状态始终与特定算子相关联**
- 为了使运行时的Flink了解算子的状态，算子需要预先注册其状态

**有两种类型的状态：**

- 算子状态（Operator State）
  - 算子状态的作用范围限定为算子任务
- 键控状态（Keyed State）
  - 根据输入数据流中定义的键（key）来维护和访问，也就是说只有当前key对应的数据才可以访问当前的状态

#### 算子状态（Operator State）

![1614673703139](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/101844-455784.png)

- 算子状态的作用范围限定为算子任务，由同一并行任务所处理的所有数据都可以访问到相同的状态
- 状态对于同一子任务而言是共享的
- 算子状态不能由相同或不同算子的另一个子任务访问

##### 算子状态数据结构

- 列表状态（List state）
  - 将状态表示为一组数据的列表
- 联合列表状态（Union list state）
  - 也将状态表示为数据的列表。它与常规列表状态的区别在于，在发生故障时，或者从保存点（savepoint）启动应用程序时如何恢复
- 广播状态（Broadcast state）
  - 如果一个算子有多项任务，而它的每项任务状态又都相同，那么这种特殊情况最适合应用广播状态。

**案例**

```java
public class StateTest {

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

//        定义一个有状态的map操作，统计当前有多少个温度值
//        统计当前分区数据的个数
        SingleOutputStreamOperator<Integer> result = map.map(new MyMapMapper());

        result.print();


        env.execute();
    }

    public static class MyMapMapper implements MapFunction<SensorReading,Integer>, ListCheckpointed<Integer> {

//        定义一个本地变量，作为算子状态
        private Integer count=0;

        @Override
        public Integer map(SensorReading value) throws Exception {
            return count++;
        }

//        对状态做快照
        @Override
        public List<Integer> snapshotState(long checkpointId, long timestamp) throws Exception {
            return Collections.singletonList(count);
        }

//        恢复快照
        @Override
        public void restoreState(List<Integer> state) throws Exception {
            for (Integer num:state) {
//                恢复count的值
                count+=num;
            }
        }
    }
}

```

#### 键控状态（Keyed State）

![1614735541965](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/17/111456-631534.png)

1. 键控状态是根据输入数据流中定义的键（key）来维护和访问的，根据键分组后，每一组键都有对应的状态。
2. Flink为每个key维护一个状态实例，并将具有相同键的所有数据，都分区到同一个算子任务中，这个任务会维护和处理这个key对应的状态
3. 当任务处理一条数据时，它会自动将状态的访问范围限定为当前数据的key

##### 键控状态数据结构

- 值状态（Valuestate）
  - 将状态表示为单个的值
- 列表状态（List state）
  - 将状态表示为一组数据的列表
- 映射状态（Mapstate）
  - 将状态表示为一组Key-Value对
- 聚合状态（Reducing state & Aggregating State）
  - 将状态表示为一个用于聚合操作的列表

##### 键控状态的使用

声明一个键控状态

![1614736400065](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/16/090525-748769.png)

读取状态

![1614736439159](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/16/090527-536107.png)

对状态赋值

![1614736465644](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/16/090526-942864.png)

**案例**

```java
public class KeyedState {

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

        SingleOutputStreamOperator<Integer> result = map
                .keyBy("id")
                .map(new MyKeyCountMapper());
        result.print();


        env.execute();
    }

    public static class MyKeyCountMapper extends RichMapFunction<SensorReading,Integer>{

//        new ValueStateDescriptor<Integer>("keycount",Integer.class):参数表示状态的名字和状态的类型
//        在环境所需要的实例全部都创建出来后才可以拿到运行时上下文环境，必须在open()后才可以做初始化操作
        private ValueState<Integer>keyCountState;

//        其他类型的状态声明
        private ListState<String> listState;

//        map状态
        private MapState<String,Double> mapState;

        @Override
        public void open(Configuration parameters) throws Exception {
            keyCountState=getRuntimeContext().getState(new ValueStateDescriptor<Integer>("keycount",Integer.class,0));
//            这里没有对count进行初始化，报错
            //keyCountState=getRuntimeContext().getState(new ValueStateDescriptor<Integer>("keycount",Integer.class));

//            初始化列表状态
            listState=getRuntimeContext().getListState(new ListStateDescriptor<String>("list_state",String.class));

            mapState=getRuntimeContext().getMapState((new MapStateDescriptor<String, Double>("map-state",String.class,Double.class)));
        }

        @Override
        public Integer map(SensorReading value) throws Exception {

            //            存入一个状态
            mapState.put("1",32.5);
//            向map状态中获取一个状态
            Double aDouble = mapState.get("1");

            //            获取list状态的值
            Iterable<String> strings = listState.get();
            for (String str:strings) {
                System.out.println(str);

            }

//            所有的状态都有clear()方法，清空所有的状态
            mapState.clear();
//            向列表中添加一个新的状态
            listState.add("state-1");

//            对单个状态变量的值做操作
            Integer count = keyCountState.value();
            count++;
            keyCountState.update(count);
            return count;

        }
    }
}

```

##### 练习

```java
public class ApplicationCase {

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

//        定义flatmap检测温度的跳变，输出报警信息
//        注意，这里对数据进行了分组，相同组中的数据温度差值大于10才会报警，不同组之间相差10不会报警
        SingleOutputStreamOperator<Tuple3<String, Double, Double>> result = map
                .keyBy("id")
                .flatMap(new MychangeFlatMap(10.0));

        result.print();

        env.execute();
    }

    public static class MychangeFlatMap extends RichFlatMapFunction<SensorReading, Tuple3<String,Double,Double>>
    {
        private Double threshold;

//        定义一个状态，保存上一次的温度值
        private ValueState<Double> lastTemp;

        public MychangeFlatMap(Double threshold) {
            this.threshold = threshold;
        }

//        在open方法中对状态变量做初始化


        @Override
        public void open(Configuration parameters) throws Exception {
            lastTemp=getRuntimeContext().getState(new ValueStateDescriptor<Double>("temp",Double.class,1.0));

        }

        @Override
        public void flatMap(SensorReading value, Collector<Tuple3<String, Double, Double>> out) throws Exception {

//            获取状态，也就是上一次的温度值
            Double value1 = lastTemp.value();
            if(lastTemp!=null){
                Double diff=Math.abs(value.getTemperature()-value1);
                if(diff >= threshold){
                    out.collect(new Tuple3<String,Double,Double>(value.getId(),value1,value.getTemperature()));

                }

            }
                //            更新状态信息
            lastTemp.update(value.getTemperature());
        }

//        最后还要做清理工作

        @Override
        public void close() throws Exception {
            lastTemp.clear();
        }
    }
}
```

#### 状态后端（State Backends）

![1623389350664](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/132913-888517.png)

- 每传入一条数据，有状态的算子任务都会读取和更新状态
- 由于有效的状态访问对于处理数据的低延迟至关重要，因此每个并行任务都会在本地维护其状态，以确保快速的状态访问
- 状态的存储、访问以及维护，由一个可插入的组件决定，这个组件就叫做状态后端（state backend）
- 状态后端主要负责两件事：**本地的状态管理（也就是内存中的状态管理），以及将检查点（checkpoint）状态写入远程存储。容错性的保证，备份状态**。

##### 状态后端分类

**MemoryStateBackend**

![1623389433434](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/133035-909589.png)

- 内存级的状态后端，会将键控状态作为内存中的对象进行管理，**将它们存储在TaskManager的JVM堆上，而将checkpoint存储在JobManager的内存中**
- 特点：快速、低延迟，但不稳定

**FsStateBackend**

![1623389471575](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/133113-305891.png)

- 将checkpoint存到远程的持久化文件系统（FileSystem）上，而对于本地状态，跟MemoryStateBackend一样，也会存在TaskManager的JVM堆上
- 同时拥有内存级的本地访问速度，和更好的容错保证

**RocksDBStateBackend**

![1623391739132](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/140902-198202.png)

将所有状态序列化后，存入本地的RocksDB中存储。

##### 状态后端配置

```java
public class StateBackend {

    public static void main(String[] args) throws Exception {
        //        创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        状态后端的配置
//        内存状态后端
        env.setStateBackend(new MemoryStateBackend());
//        文件系统状态后端
        env.setStateBackend(new FsStateBackend("path"));
//        RocksDBStateBackend需要引入依赖
        env.setStateBackend(new RocksDBStateBackend("path"));

//        顺序输出，设置并行度是1
        env.setParallelism(1);

//        从文件中读取
        DataStreamSource<String> sdss = env.readTextFile("D:\\soft\\idea\\work\\work08\\data\\file.txt");

        //        使用lambda表达式
        DataStream<SensorReading> map = sdss.map(line->{
            String[] s1 = line.split(",");

            return new SensorReading(s1[0],new Long(s1[1]),new Double(s1[2]));

        });

        map.print();

        env.execute();
    }

}

```

**引入依赖**

```java
 <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-statebackend-rocksdb_2.12</artifactId>scala版本号
            <version>1.10.1</version>flink版本号
        </dependency>
```

#### 案例说明

![1623143547852](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/16/092147-275587.png)

**代码演示**

```java
public class Test22 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Tuple2<String, Integer>> source = env.fromElements(
                Tuple2.of("beijing", 2),
                Tuple2.of("shanghai", 5),
                Tuple2.of("shenzhen", 8),
                Tuple2.of("guangzhou", 1),
                Tuple2.of("xian", 19),
                Tuple2.of("dalian", 9),
                Tuple2.of("tianjin", 18)
        );

//        求value的最大值
//        source.keyBy(t -> t.f0)
//                .maxBy(1)
//                .print();

        SingleOutputStreamOperator<Tuple3<String, Integer, Integer>> map = source.map(new RichMapFunction<Tuple2<String, Integer>, Tuple3<String, Integer, Integer>>() {
//            定义一个状态，存放最大值
            private ValueState<Integer> max;
//            对状态进行初始化工作,open()方法只会调用一次
            @Override
            public void open(Configuration parameters) throws Exception {
                ValueStateDescriptor<Integer> integerValueStateDescriptor = new ValueStateDescriptor<Integer>("max",Integer.class);
                max=getRuntimeContext().getState(integerValueStateDescriptor);
            }

//            使用状态
            @Override
            public Tuple3<String, Integer, Integer> map(Tuple2<String, Integer> value) throws Exception {

//                获取当前的值
                Integer currentValue = value.f1;
//                获取状态
                Integer state = max.value();

                if(state == null || currentValue > state){
                    max.update(currentValue);
                    return Tuple3.of(value.f0,currentValue,currentValue);
                }
                return Tuple3.of(value.f0,currentValue,currentValue);
            }
        });

        map.print();


        env.execute();
    }
}
```

**案例二**

![1623388204192](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/11/131025-894336.png)

```java
public class Test23 {

    public static void main(String[] args) throws Exception {
        /**
         * 基于事件的滚动和滑动窗口
         */

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
//        设置并行度为1，方便观察
        env.setParallelism(1);
        DataStreamSource dataStreamSource = env.addSource(new MyKakfa());

        dataStreamSource.print();


        env.execute();
    }

     static class MyKakfa extends RichParallelSourceFunction implements CheckpointedFunction {

//        因为并行度设置为1，所以只有一个分区
        private ListState<Long> offsetState=null;
//        如果有多个分区，那么List状态里面存储的是分区对应的状态
//        private ListState<Map<partition,Offset>> offsetState=null;
        private Long offset=0l;
        private boolean flag=true;


//          持久化state
//        该方法会定时执行将state存放到checkPoint（磁盘目录）中
        @Override
        public void snapshotState(FunctionSnapshotContext context) throws Exception {
//            清理内存中的数据，并且存放到checkPoint中
            offsetState.clear();
            offsetState.add(offset);

        }
        //        初始化
        @Override
        public void initializeState(FunctionInitializationContext context) throws Exception {

            ListStateDescriptor<Long> descriptor = new ListStateDescriptor<>("offsetState", Long.class);
            ListState<Long> listState = context.getOperatorStateStore().getListState(descriptor);
            offsetState=listState;
        }

        @Override
        public void run(SourceContext ctx) throws Exception {

           while (flag){
               Iterator<Long> iterator = offsetState.get().iterator();
               if(iterator.hasNext()){
                   offset=iterator.next();
               }
               offset+=1;
               int subtaskId=getRuntimeContext().getIndexOfThisSubtask();
               ctx.collect("subtaskId="+subtaskId+"   offset="+offset);
               Thread.sleep(2000);
           }
        }

        @Override
        public void cancel() {
            flag=false;
        }
    }
}
```

