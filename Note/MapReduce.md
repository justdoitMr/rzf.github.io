# MapReduce原理

[TOC]

![1611807458144](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/185545-451300.png)

mapreducer一个作业的计算流程，适合迭代式计算，不适合并行计算。

## MapReducer概念

### 什么是mapreducer。

Mapreduce是一个分布式运算程序的编程框架，是用户开发“基于hadoop的数据分析应用”的核心框架；

Mapreduce核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个hadoop集群上。

- MapReduce处理过程分为两个阶段：Map和Reduce。
  - Map负责把一个任务分解成多个任务；
  - Reduce负责把分解后多任务处理的结果汇总。

### 为什么要MapReduce？

1. 海量数据在单机上处理因为硬件资源限制，无法胜任

2. 而一旦将单机版程序扩展到集群来分布式运行，将极大增加程序的复杂度和开发难度

3. 引入mapreduce框架后，开发人员可以将绝大部分工作集中在业务逻辑的开发上，而将分布式计算中的复杂性交由框架来处理。

4. mapreduce分布式方案考虑的问题
   1. 运算逻辑要不要先分后合？ 
   2. 程序如何分配运算任务（**切片**）？
   3. 两阶段的程序如何启动？如何协调？
   4. 整个程序运行过程中的监控？容错？重试？

分布式方案需要考虑很多问题，但是我们可以将分布式程序中的公共功能封装成框架，让开发人员将精力集中于业务逻辑上。而mapreduce就是这样一个分布式程序的通用框架。

### MapReduce优缺点

- **优点：**

  1. MapReduce 易于编程

     它简单的实现一些接口，就可以完成一个分布式程序，这个分布式程序可以分布到大量廉价的PC机器上运行。也就是说你写一个分布式程序，跟写一个简单的串行程序是一模一样的。就是因为这个特点使得MapReduce编程变得非常流行。

  2. 良好的扩展性

     当你的计算资源不能得到满足的时候，你可以通过简单的增加机器来扩展它的计算能力。

  3. 高容错性

     MapReduce设计的初衷就是使程序能够部署在廉价的PC机器上，这就要求它具有很高的容错性。比如其中一台机器挂了，它可以把上面的计算任务转移到另外一个节点上运行，不至于这个任务运行失败，而且这个过程不需要人工参与，而完全是由Hadoop内部完成的。

  4. 适合PB级以上海量数据的离线处理

     可以实现上千台服务器集群并发工作，提供数据处理能力。

- **缺点：**

1. 不擅长实时计算

   MapReduce无法像MySQL一样，在毫秒或者秒级内返回结果。

2. 不擅长流式计算

   流式计算的输入数据是动态的，而MapReduce的输入数据集是静态的，不能动态变化。这是因为MapReduce自身的设计特点决定了数据源必须是静态的.

3. 不擅长DAG（有向图）计算

   多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下，MapReduce并不是不能做，而是使用后，每个MapReduce作业的输出结果都会写入到磁盘，会造成大量的磁盘IO，导致性能非常的低下。

### mapreducer工作机制

![1610589656746](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/14/100427-221494.png)

1. 分布式的运算程序往往需要分成至少2个阶段。

2. 第一个阶段的MapTask并发实例，完全并行运行，互不相干。

3. 第二个阶段的ReduceTask并发实例互不相干，但是他们的数据依赖于上一个阶段的所有MapTask并发实例的输出，map阶段输出的分区个数决定下一个阶段reduce的个数。

4. MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序，串行运行。

总结：分析WordCount数据流走向深入理解MapReduce核心思想。

### MapReduce进程

- 一个完整的mapreduce程序在分布式运行时有三类实例进程：
  - MrAppMaster：负责整个程序的过程调度及状态协调，简单来说就是负责整个作业的执行情况。
  - MapTask：负责map阶段的整个数据处理流程
  - ReduceTask：负责reduce阶段的整个数据处理流程

### MapReduce编程规范

**用户编写的程序分成三个部分：Mapper，Reducer，Driver(提交运行mr程序的客户端)**

1. Mapper阶段
   1. 用户自定义的Mapper要继承自己的父类
   2. Mapper的输入数据是KV对的形式（KV的类型可自定义）
   3. Mapper中的业务逻辑写在map()方法中
   4. Mapper的输出数据是KV对的形式（KV的类型可自定义）
   5. map()方法（maptask进程）对每一个<K,V>调用一次

2. Reducer阶段
   1. 用户自定义的Reducer要继承自己的父类
   2. Reducer的输入数据类型对应Mapper的输出数据类型，也是KV
   3. Reducer的业务逻辑写在reduce()方法中
   4. Reducetask进程对每一组相同k的<k,v>组调用一次reduce()方法

3. Driver阶段

整个程序需要一个Drvier来进行提交，提交的是一个描述了各种必要信息的job对象

**MR程序的运行流程**

![1610591087175](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/14/102447-793004.png)

1. 在MapReduce程序读取文件的输入目录上存放相应的文件。

2. 客户端程序在submit()方法执行前，获取待处理的数据信息，然后根据集群中参数的配置形成一个任务分配规划。

3. 客户端提交job.split、jar包、job.xml等文件给yarn，yarn中的resourcemanager启动MRAppMaster。

4. MRAppMaster启动后根据本次job的描述信息，计算出需要的maptask实例数量，然后向集群申请机器启动相应数量的maptask进程。

5. maptask利用客户指定的inputformat来读取数据，形成输入KV对。

6. maptask将输入KV对传递给客户定义的map()方法，做逻辑运算

7. map()运算完毕后将KV对收集到maptask缓存。

8. maptask缓存中的KV对按照K分区排序后不断写到磁盘文件

9. MRAppMaster监控到所有maptask进程任务完成之后，会根据客户指定的参数启动相应数量的reducetask进程，并告知reducetask进程要处理的数据分区。

10. Reducetask进程启动之后，根据MRAppMaster告知的待处理数据所在位置，从若干台maptask运行所在机器上获取到若干个maptask输出结果文件，并在本地进行重新归并排序，然后按照相同key的KV为一个组，调用客户定义的reduce()方法进行逻辑运算。

11. Reducetask运算完毕后，调用客户指定的outputformat将结果数据输出到外部存储。

### 单词统计程序案例

**代码实现**

**Map程序**

~~~ java
/**
 * keyin:输入数据的key
 * value:输入数据的value
 * keyout:输出数据的类型
 * valueout:输出数据的value类型
 */
public class WordCountMapper extends Mapper<LongWritable, Text,Text, IntWritable> {
    IntWritable v = new IntWritable();
    Text k = new Text();
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
//      rzf rzf
//        获取一行数据，然后转换为string类型
        String str=value.toString();
//        按照空格对字符串进行切割
        String [] words=str.split(" ");
//        循环把所有单词全部输出
        for (String word:words){

//            设置键值2，是text类型
            k.set(word);
//            设置value值

            v.set(1);
            context.write(k,v);

        }
    }
}
~~~

**reducer程序**

~~~ java
/**
 * keyin valuein :分别代表map阶段输出的key和value
 * keyout valueout:分别代表输出
 *
 */
public class WordCountReducer extends Reducer<Text,IntWritable,Text,IntWritable> {
    IntWritable v=new IntWritable();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
//        rzf 1
//        rzf 1
        int sum=0;
//        累加求和
        for (IntWritable value:values){
            sum+= value.get();
        }
        v.set(sum);
//        写出
        context.write(key,v);
    }
}

~~~

**配置提交程序**

~~~ java
public class WordCountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
//        获取job对象
        Job job = Job.getInstance(conf);

//        设置jar包的存储位置
        job.setJarByClass(WordCountDriver.class);

//        关联map和reducer类
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);

//        设置mapper阶段数据的key value类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

//        设置最终数据输出的key value类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

//          设置输入路径和输出路径
        FileInputFormat.setInputPaths(job,new Path(args[0]));
        FileOutputFormat.setOutputPath(job,new Path(args[1]));
//        提交job
//        job.submit();
        boolean flag=job.waitForCompletion(true);
    }
}
~~~

### 在集群上测试

用maven打jar包，需要添加的打包插件依赖

注意：标记红颜色的部分需要替换为自己工程主类

~~~ java
<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.3.2</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-assembly-plugin </artifactId>
				<configuration>
					<descriptorRefs>
						<descriptorRef>jar-with-dependencies</descriptorRef>
					</descriptorRefs>
					<archive>
						<manifest>
							<mainClass>com.atguigu.mr.WordcountDriver</mainClass>
						</manifest>
					</archive>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
~~~

注意：如果工程上显示红叉。在项目上右键->maven->update project即可。

1. 将程序打成jar包，然后拷贝到Hadoop集群中

步骤详情：右键->Run as->maven install。等待编译完成就会在项目的target文件夹中生成jar包。如果看不到。在项目上右键-》Refresh，即可看到。修改不带依赖的jar包名称为wc.jar，并拷贝该jar包到Hadoop集群。

![1610626175345](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/14/200935-815987.png)

2. 启动Hadoop集群

3. 执行WordCount程序

~~~ java
[rzf@hadoop01 hadoop-2.7.2]$ hadoop jar /opt/module/wc.jar(jar包的路径) qq.rzf.WordCountDriver（主函数的全类名） /rzf.txt（文件输入路径） /user/output（文件输出路径）
//注意：文件的路径都是hdfs文件系统的路径
~~~

## Hadoop序列化

### Writable序列化

序列化就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储（持久化）和网络传输。 

反序列化就是将收到字节序列（或其他数据传输协议）或者是硬盘的持久化数据，转换成内存中的对象。

Java的序列化是一个重量级（实现的功能比较多）序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，header，继承体系等），不便于在网络中高效传输。所以，hadoop自己开发了一套序列化机制（Writable），精简、高效。

- 为什么要序列化？

  一般来说，“活的”对象只生存在内存里，关机断电就没有了。而且“活的”对象只能由本地的进程使用，不能被发送到网络上的另外一台计算机。 然而序列化可以存储“活的”对象，可以将“活的”对象发送到远程计算机。

Hadoop序列化的特点

1. 紧凑，高效使用存储空间。
2. 快速，读写数据额外开销小。
3. 可扩展，随着通信协议的升级可以升级。
4. 互操作，支持多种语言交互。

### 常用数据序列化类型

| Java类型 | HadoopWritable类型 |
| -------- | ------------------ |
| boolean  | BooleanWritable    |
| byte     | ByteWritable       |
| int      | IntWritable        |
| float    | FloatWritable      |
| long     | LongWritable       |
| double   | DoubleWritable     |
| string   | Text               |
| map      | MapWritable        |
| array    | ArrayWritable      |

### 自定义bean对象实现序列化接口（Writable）

在企业开发中往往常用的基本序列化类型不能满足所有需求，比如在Hadoop框架内部传递一个bean对象，那么该对象就需要实现序列化接口。

- 自定义bean对象要想序列化传输，必须实现序列化接口，需要注意以下7项。

1. 必须实现Writable接口
2. 反序列化时，需要反射调用空参构造函数，所以必须有**空参构造函数**

~~~ java
public FlowBean() {
	super();
}
~~~

3. 重写序列化方法

~~~ java
@Override
public void write(DataOutput out) throws IOException {
	out.writeLong(upFlow);
	out.writeLong(downFlow);
	out.writeLong(sumFlow);
}
~~~

4. 重写反序列化方法

~~~ java
@Override
public void readFields(DataInput in) throws IOException {
	upFlow = in.readLong();
	downFlow = in.readLong();
	sumFlow = in.readLong();
}
~~~

5. **注意反序列化的顺序和序列化的顺序完全一致**，序列化的管道可以想象为一个队列。

6. 要想把结果显示在文件中，需要重写toString()，且用”\t”分开，方便后续用

7. 如果需要将自定义的bean放在key中传输，则还需要实现comparable接口，因为mapreduce框中的shuffle过程一定会对key进行排序

~~~ java
@Override
public int compareTo(FlowBean o) {
	// 倒序排列，从大到小
	return this.sumFlow > o.getSumFlow() ? -1 : 1;
}
~~~

> 之所以需要序列化，就是因为map阶段和reducer阶段在不同的服务器中，所以中间需要传输bean对象，所以需要先将对象进行序列化操作。

### 序列化案例演示

**需求**

统计每一个手机号耗费的总上行流量、下行流量、总流量

**输出数据格式**

~~~ java
7 	13560436666	120.196.100.99		1116		 954			200
id	手机号码		网络ip			上行流量  下行流量     网络状态码
~~~

**输出数据格式**

~~~ java
13560436666 		1116		      954 			2070
手机号码		    上行流量        下行流量		总流量
~~~

**过程分析**

![1610709538094](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/20/092830-692571.png)

**编写流量统计的Bean对象**

~~~ java
// 1 实现writable接口
public class FlowBean implements Writable {

    private long upFlow;
    private long downFlow;
    private long sumFlow;

    //2  反序列化时，需要反射调用空参构造函数，所以必须有
    public FlowBean() {
        super();
    }

    public FlowBean(long upFlow, long downFlow) {
        super();
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        this.sumFlow = upFlow + downFlow;
    }

    //3  写序列化方法
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(upFlow);
        out.writeLong(downFlow);
        out.writeLong(sumFlow);
    }

    //4 反序列化方法
    //5 反序列化方法读顺序必须和写序列化方法的写顺序必须一致
    @Override
    public void readFields(DataInput in) throws IOException {
        this.upFlow  = in.readLong();
        this.downFlow = in.readLong();
        this.sumFlow = in.readLong();
    }
    public void set(long upFlow1,long downFlow1){
        upFlow=upFlow1;
        downFlow=downFlow1;
        sumFlow=upFlow1+downFlow1;
    }

    // 6 编写toString方法，方便后续打印到文本
    @Override
    public String toString() {
        return upFlow + "\t" + downFlow + "\t" + sumFlow;
    }

    public long getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(long upFlow) {
        this.upFlow = upFlow;
    }

    public long getDownFlow() {
        return downFlow;
    }

    public void setDownFlow(long downFlow) {
        this.downFlow = downFlow;
    }

    public long getSumFlow() {
        return sumFlow;
    }

    public void setSumFlow(long sumFlow) {
        this.sumFlow = sumFlow;
    }
}
~~~

**编写mapper类**

~~~ java
public class MapperFlowBean extends Mapper {
    @Override
    protected void map(Object key, Object value, Context context) throws IOException, InterruptedException {
        //    value值
        FlowBean v=new FlowBean();
        //    key值
        Text k=new Text();

//8 	15910133277	192.168.100.5	www.hao123.com	3156	2936	200
//            获取一行，转换为string类型
            String line=value.toString();
//        切分电话号码,以/t的方式进行切分
            String fields[]=line.split("\t");

//        封装bean
//        设置手机号为键值
            k.set(fields[1]);
            long upFlow=Long.parseLong(fields[fields.length-3]);
            long downFlow=Long.parseLong(fields[fields.length-2]);
            v.setUpFlow(upFlow);
            v.setDownFlow(downFlow);
//        写出
        context.write(k,v);
    }
}
~~~

**编写reducer类**

~~~ java
public class ReducerFlowBean extends Reducer<Text,FlowBean,Text,FlowBean> {
    FlowBean v=new FlowBean();
    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException {
//   15910133277	3156	2936
//        累加求和
        long sum_upFlow=0;
        long sum_downFlow=0;
        for (FlowBean flowBean:values) {
            sum_upFlow+=flowBean.getUpFlow();
            sum_downFlow+=flowBean.getDownFlow();

        }
        v.set(sum_upFlow,sum_downFlow);
        context.write(key,v);
    }
}
~~~

**编写driver类**

~~~ java
public class FlowSumDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        args =new String[]{"E:\\file\\input\\phone_data.txt","E:\\file\\output"};

        Configuration conf = new Configuration();
//        1获取job对象
        Job job = Job.getInstance(conf);

//        2设置jar的路径
        job.setJarByClass(FlowSumDriver.class);

//        3关联mapper和reducer
        job.setMapperClass(MapperFlowBean.class);
        job.setReducerClass(ReducerFlowBean.class);

//        4设置mapper输出的key value类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);

//        5设置最终输出的类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

//        6设置输入输出的路径
        FileInputFormat.setInputPaths(job,new Path(args[0]));
        FileOutputFormat.setOutputPath(job,new Path(args[1]));

//        7提交job
        boolean result = job.waitForCompletion(true);
        System.out.println(result);
    }
}
~~~

## MapReduce框架原理

**输入类继承关系**

![1621483399717](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/20/120322-333613.png)

![1621483430884](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621483430884.png)

### InputFormat数据输入

#### 切片与MapTask并行度决定机制。

1. 问题引出：

   MapTask的并行度决定Map阶段的任务处理并发度，进而影响到整个Job的处理速度。

   思考：1G的数据，启动8个MapTask，可以提高集群的并发处理能力。那么1K的数据，也启动8个MapTask，会提高集群性能吗？MapTask并行任务是否越多越好呢？哪些因素影响了MapTask并行度？

2. MapTask并行度决定机制

   数据块：Block是HDFS物理上把数据分成一块一块。

   数据切片：数据切片只是在**逻辑上**对输入进行分片，并不会在磁盘上将其切分成片进行存储。

   ![1610713064253](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/19/200515-905618.png)

   > 注意：切片是针对单个文件进行的，不会把多个文件呢合并到一起进行切片。

3. maptask的并行度决定map阶段的任务处理并发度，进而影响到整个job的处理速度。那么，mapTask并行任务是否越多越好呢？ 

   **MapTask并行度决定机制**

   一个job的map阶段MapTask并行度（个数），由客户端提交job时的切片个数决定，并不是越多越好，如果切片很少，但是mapTask很多的情况下，启动mapTask就很浪费时间，启动mapTask的时间都超过程序的运行时间了，所以并不是mapTask越多越好。

#### Job提交流程源码和切片源码详解

**图解**

![1610844815012](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/19/200516-947861.png)

**代码说明**

~~~ java
waitForCompletion()//提交作业的入口
//下面的方法都是在submit()方法里面进行的
submit();

// 1建立连接
	connect();	
		// 1）创建提交Job的代理
		new Cluster(getConfiguration());
			// （1）判断是本地yarn还是远程
			initialize(jobTrackAddr, conf); 

// 2 提交job
submitter.submitJobInternal(Job.this, cluster)
	// 1）创建给集群提交数据的Stag路径
	Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);

	// 2）获取jobid ，并创建Job路径
	JobID jobId = submitClient.getNewJobID();

	// 3）拷贝jar包到集群
copyAndConfigureFiles(job, submitJobDir);	
	rUploader.uploadFiles(job, jobSubmitDir);

// 4）计算切片，生成切片规划文件
writeSplits(job, submitJobDir);
		maps = writeNewSplits(job, jobSubmitDir);
		input.getSplits(job);

// 5）向Stag路径写XML配置文件
writeConf(conf, submitJobFile);
	conf.writeXml(out);

// 6）提交Job,返回提交状态
status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());
~~~

**作业提交流程图**

![1621478221736](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621478221736.png)

**详细源码调试**

~~~ java
//作业提交流程waitForCompletion()方法，提交作业
//1，执行waitForCompletion()方法，提交作业
boolean flag=job.waitForCompletion(true);
//2,执行submit()提交操作
public boolean waitForCompletion(boolean verbose
                                   ) throws IOException, InterruptedException,
                                            ClassNotFoundException {
    if (state == JobState.DEFINE) {
      submit();//进入这个方法
    }
    if (verbose) {
      monitorAndPrintJob();
    } else {
      // get the completion poll interval from the client.
      int completionPollIntervalMillis = 
        Job.getCompletionPollInterval(cluster.getConf());
      while (!isComplete()) {
        try {
          Thread.sleep(completionPollIntervalMillis);
        } catch (InterruptedException ie) {
        }
      }
    }
    return isSuccessful();
  }

//进入submit()提交方法
public void submit() 
         throws IOException, InterruptedException, ClassNotFoundException {
    ensureState(JobState.DEFINE);//判断作业当前的额状态
    setUseNewAPI();//设置新旧api
    connect();//进入建立集群的方法
    final JobSubmitter submitter = 
        getJobSubmitter(cluster.getFileSystem(), cluster.getClient());
    status = ugi.doAs(new PrivilegedExceptionAction<JobStatus>() {
      public JobStatus run() throws IOException, InterruptedException, 
      ClassNotFoundException {
        return submitter.submitJobInternal(Job.this, cluster);
      }
    });
    state = JobState.RUNNING;
    LOG.info("The url to track the job: " + getTrackingURL());
   }
//进入connect（）方法
private synchronized void connect()
          throws IOException, InterruptedException, ClassNotFoundException {
    if (cluster == null) {
      //在这里链接集群
      cluster = 
        ugi.doAs(new PrivilegedExceptionAction<Cluster>() {
                   public Cluster run()
                          throws IOException, InterruptedException, 
                                 ClassNotFoundException {
                     return new Cluster(getConfiguration());//进入建立集群的方法
                   }
                 });
    }
  }
//进入建立集群的方法
 public Cluster(InetSocketAddress jobTrackAddr, Configuration conf) 
      throws IOException {
    this.conf = conf;
    this.ugi = UserGroupInformation.getCurrentUser();
    initialize(jobTrackAddr, conf);//对创建的集群进行初始化操作
  }
//进入初始化集群的方法
private void initialize(InetSocketAddress jobTrackAddr, Configuration conf)
      throws IOException {

    synchronized (frameworkLoader) {
      for (ClientProtocolProvider provider : frameworkLoader) {
        LOG.debug("Trying ClientProtocolProvider : "
            + provider.getClass().getName());
        ClientProtocol clientProtocol = null; 
        try {
          if (jobTrackAddr == null) {
            clientProtocol = provider.create(conf);
          } else {
            clientProtocol = provider.create(jobTrackAddr, conf);
          }

          if (clientProtocol != null) {
            clientProtocolProvider = provider;
            client = clientProtocol;
            LOG.debug("Picked " + provider.getClass().getName()
                + " as the ClientProtocolProvider");
            break;
          }
          else {
            LOG.debug("Cannot pick " + provider.getClass().getName()
                + " as the ClientProtocolProvider - returned null protocol");
          }
        } 
        catch (Exception e) {
          LOG.info("Failed to use " + provider.getClass().getName()
              + " due to error: ", e);
        }
      }
    }

    if (null == clientProtocolProvider || null == client) {
      throw new IOException(
          "Cannot initialize Cluster. Please check your configuration for "
              + MRConfig.FRAMEWORK_NAME
              + " and the correspond server addresses.");
    }
  }
//执行完毕初始化工作后，会重新回到创建集群的位置
~~~

初始化中会创建localRunner，如果是在集群中，那么就会创建yarnRunner。

![1621475432607](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/20/095034-127971.png)

最终会返回下面的位置

![1621475614713](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/20/095336-446641.png)

![1621475735555](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621475735555.png)

~~~ java
JobStatus submitJobInternal(Job job, Cluster cluster) 
  throws ClassNotFoundException, InterruptedException, IOException {

    //validate the jobs output specs 
    checkSpecs(job);//校验输出路径是否正确

    Configuration conf = job.getConfiguration();//获取作业的配置信息
    addMRFrameworkToDistributedCache(conf);

  //创建临时工作目录。也就是三个文件的提交路径
    Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);
    //configure the command line options correctly on the submitting dfs
    InetAddress ip = InetAddress.getLocalHost();
    if (ip != null) {
      submitHostAddress = ip.getHostAddress();
      submitHostName = ip.getHostName();
      conf.set(MRJobConfig.JOB_SUBMITHOST,submitHostName);
      conf.set(MRJobConfig.JOB_SUBMITHOSTADDR,submitHostAddress);
    }
  //获取jobid号码
    JobID jobId = submitClient.getNewJobID();
  //设置job的id
    job.setJobID(jobId);
    Path submitJobDir = new Path(jobStagingArea, jobId.toString());
    JobStatus status = null;
    try {
      conf.set(MRJobConfig.USER_NAME,
          UserGroupInformation.getCurrentUser().getShortUserName());
      conf.set("hadoop.http.filter.initializers", 
          "org.apache.hadoop.yarn.server.webproxy.amfilter.AmFilterInitializer");
      conf.set(MRJobConfig.MAPREDUCE_JOB_DIR, submitJobDir.toString());
      LOG.debug("Configuring job " + jobId + " with " + submitJobDir 
          + " as the submit dir");
      // get delegation token for the dir
      TokenCache.obtainTokensForNamenodes(job.getCredentials(),
          new Path[] { submitJobDir }, conf);
      
      populateTokenCache(conf, job.getCredentials());

      // generate a secret to authenticate shuffle transfers
      if (TokenCache.getShuffleSecretKey(job.getCredentials()) == null) {
        KeyGenerator keyGen;
        try {
          keyGen = KeyGenerator.getInstance(SHUFFLE_KEYGEN_ALGORITHM);
          keyGen.init(SHUFFLE_KEY_LENGTH);
        } catch (NoSuchAlgorithmException e) {
          throw new IOException("Error generating shuffle secret key", e);
        }
        SecretKey shuffleKey = keyGen.generateKey();
        TokenCache.setShuffleSecretKey(shuffleKey.getEncoded(),
            job.getCredentials());
      }
      if (CryptoUtils.isEncryptedSpillEnabled(conf)) {
        conf.setInt(MRJobConfig.MR_AM_MAX_ATTEMPTS, 1);
        LOG.warn("Max job attempts set to 1 since encrypted intermediate" +
                "data spill is enabled");
      }
//提交文件信息，也就是文件的额切片信息
      copyAndConfigureFiles(job, submitJobDir);

      Path submitJobFile = JobSubmissionFiles.getJobConfPath(submitJobDir);
      
      // Create the splits for the job
      LOG.debug("Creating splits at " + jtFs.makeQualified(submitJobDir));
      int maps = writeSplits(job, submitJobDir);
      conf.setInt(MRJobConfig.NUM_MAPS, maps);
      LOG.info("number of splits:" + maps);

      // write "queue admins of the queue to which job is being submitted"
      // to job file.
      String queue = conf.get(MRJobConfig.QUEUE_NAME,
          JobConf.DEFAULT_QUEUE_NAME);
      AccessControlList acl = submitClient.getQueueAdmins(queue);
      conf.set(toFullPropertyName(queue,
          QueueACL.ADMINISTER_JOBS.getAclName()), acl.getAclString());

      // removing jobtoken referrals before copying the jobconf to HDFS
      // as the tasks don't need this setting, actually they may break
      // because of it if present as the referral will point to a
      // different job.
      TokenCache.cleanUpTokenReferral(conf);

      if (conf.getBoolean(
          MRJobConfig.JOB_TOKEN_TRACKING_IDS_ENABLED,
          MRJobConfig.DEFAULT_JOB_TOKEN_TRACKING_IDS_ENABLED)) {
        // Add HDFS tracking ids
        ArrayList<String> trackingIds = new ArrayList<String>();
        for (Token<? extends TokenIdentifier> t :
            job.getCredentials().getAllTokens()) {
          trackingIds.add(t.decodeIdentifier().getTrackingId());
        }
        conf.setStrings(MRJobConfig.JOB_TOKEN_TRACKING_IDS,
            trackingIds.toArray(new String[trackingIds.size()]));
      }

      // Set reservation info if it exists
      ReservationId reservationId = job.getReservationId();
      if (reservationId != null) {
        conf.set(MRJobConfig.RESERVATION_ID, reservationId.toString());
      }

      // Write job file to submit dir
      //提交一些配置文件,也就是xml文件
      writeConf(conf, submitJobFile);
      
      //
      // Now, actually submit the job (using the submit name)
      //
      printTokens(jobId, job.getCredentials());
      status = submitClient.submitJob(
          jobId, submitJobDir.toString(), job.getCredentials());
      if (status != null) {
        return status;
      } else {
        throw new IOException("Could not launch job");
      }
    } finally {
      if (status == null) {
        LOG.info("Cleaning up the staging area " + submitJobDir);
        if (jtFs != null && submitJobDir != null)
          jtFs.delete(submitJobDir, true);

      }
    }
  }
//提交过程到此位置已经结束
~~~

#### **FileInputFormat切片机制**

切片在本地运行模式中是32m，在hadoop版本一中是64m，在版本2中是128m。

**处理流程**

FileInputFormat源码解析(input.getSplits(job))

1. 找到你数据存储的目录。
2. 开始遍历处理（规划切片）目录下的每一个文件(**注意是以文件为单位进行遍历的**)
3. 遍历第一个文件ss.txt
   1. 获取文件大小fs.sizeOf(ss.txt);
   2. 计算切片大小computeSliteSize(Math.max(minSize,Math.max(maxSize,blocksize)))=blocksize=128M
   3. 默认情况下，切片大小=blocksize
   4. 开始切，形成第1个切片：ss.txt—0:128M 第2个切片ss.txt—128:256M 第3个切片ss.txt—256M:300M（**每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，不大于1.1倍就划分一块切片**）
   5. 将切片信息写到一个切片规划文件中
   6. 整个切片的核心过程在==getSplit()==方法中完成。
   7. 数据切片只是在逻辑上对输入数据进行分片，并不会再磁盘上将其切分成分片进行存储。InputSplit只记录了分片的元数据信息，比如起始位置、长度以及所在的节点列表等。
   8. 注意：block是HDFS上物理上存储的存储的数据，切片是对数据逻辑上的划分。

4. 提交切片规划文件到yarn上，yarn上的MrAppMaster就可以根据切片规划文件计算开启maptask个数。

**FileInputFormat中默认的切片机制：**

1. 简单地按照文件的内容长度进行切片
2. 切片大小，默认等于block大小
3. **切片时不考虑数据集整体，而是逐个针对每一个文件单独切片**

~~~ java
//例子：
file1.txt    320M
file2.txt    10M
//经过FileInputFormat的切片机制运算后，形成的切片信息如下：
file1.txt.split1--  0~128
file1.txt.split2--  128~256
file1.txt.split3--  256~320
file2.txt.split1--  0~10M
~~~

**FileInputFormat切片大小的参数配置**

通过分析源码，在FileInputFormat中，计算切片大小的逻辑：Math.max(minSize, Math.min(maxSize, blockSize));  

切片主要由这几个值来运算决定

- mapreduce.input.fileinputformat.split.minsize=1 默认值为1

- mapreduce.input.fileinputformat.split.maxsize= Long.MAXValue 默认值Long.MAXValue

因此，默认情况下，切片大小=blocksize。

**maxsize（切片最大值）：参数如果调得比blocksize小，则会让切片变小，而且就等于配置的这个参数的值。**

**minsize （切片最小值）：参数调的比blockSize大，则可以让切片变得比blocksize还大。**

1. 获取切片信息API

~~~ java
// 根据文件类型获取切片信息
FileSplit inputSplit = (FileSplit) context.getInputSplit();
// 获取切片的文件名称
String name = inputSplit.getPath().getName();
~~~

**切片机制源码详解**

是FileInputFormat()里面的默认切片机制。

```java
//获取切片
 int maps = writeSplits(job, submitJobDir);
//切片方法
private int writeSplits(org.apache.hadoop.mapreduce.JobContext job,
      Path jobSubmitDir) throws IOException,
      InterruptedException, ClassNotFoundException {
    JobConf jConf = (JobConf)job.getConfiguration();
    int maps;
    if (jConf.getUseNewMapper()) {
      maps = writeNewSplits(job, jobSubmitDir);
    } else {
      maps = writeOldSplits(jConf, jobSubmitDir);
    }
    return maps;
  }

private <T extends InputSplit>
  int writeNewSplits(JobContext job, Path jobSubmitDir) throws IOException,
      InterruptedException, ClassNotFoundException {
    Configuration conf = job.getConfiguration();
    InputFormat<?, ?> input =
      ReflectionUtils.newInstance(job.getInputFormatClass(), conf);
//真正进行切片的方法
    List<InputSplit> splits = input.getSplits(job);
    T[] array = (T[]) splits.toArray(new InputSplit[splits.size()]);

    // sort the splits into order based on size, so that the biggest
    // go first
    Arrays.sort(array, new SplitComparator());
    JobSplitWriter.createSplitFiles(jobSubmitDir, conf, 
        jobSubmitDir.getFileSystem(conf), array);
    return array.length;
  }

//下面是对文件进行切片的方法
public List<InputSplit> getSplits(JobContext job) throws IOException {
    StopWatch sw = new StopWatch().start();
  //minSize是1
  //maxSize默认是long的最大值
    long minSize = Math.max(getFormatMinSplitSize(), getMinSplitSize(job));
    long maxSize = getMaxSplitSize(job);

    // generate splits
    List<InputSplit> splits = new ArrayList<InputSplit>();
    List<FileStatus> files = listStatus(job);
  //遍历所有的文件，这里可以看出来是按照文件进行一个一个的切分，不会把多个文件进行合并切分
    for (FileStatus file: files) {
      Path path = file.getPath();
      long length = file.getLen();
      if (length != 0) {
        BlockLocation[] blkLocations;
        if (file instanceof LocatedFileStatus) {
          blkLocations = ((LocatedFileStatus) file).getBlockLocations();
        } else {
          FileSystem fs = path.getFileSystem(job.getConfiguration());
          blkLocations = fs.getFileBlockLocations(file, 0, length);
        }
        //判断文件是否可以进行切分
        if (isSplitable(job, path)) {
          //获取文件块的大小，默认是128m，老版本是64m,本地模式是32m
          long blockSize = file.getBlockSize();
          //计算切片的大小
          long splitSize = computeSplitSize(blockSize, minSize, maxSize);

          long bytesRemaining = length;
          //判断剩余块的大小是否是切片大小的1.1倍
          while (((double) bytesRemaining)/splitSize > SPLIT_SLOP) {
            int blkIndex = getBlockIndex(blkLocations, length-bytesRemaining);
            //如果大于1.1倍，那么就重新增加一个切片
            splits.add(makeSplit(path, length-bytesRemaining, splitSize,
                        blkLocations[blkIndex].getHosts(),
                        blkLocations[blkIndex].getCachedHosts()));
            bytesRemaining -= splitSize;
          }

          if (bytesRemaining != 0) {
            int blkIndex = getBlockIndex(blkLocations, length-bytesRemaining);
            splits.add(makeSplit(path, length-bytesRemaining, bytesRemaining,
                       blkLocations[blkIndex].getHosts(),
                       blkLocations[blkIndex].getCachedHosts()));
          }
        } else { // not splitable
          splits.add(makeSplit(path, 0, length, blkLocations[0].getHosts(),
                      blkLocations[0].getCachedHosts()));
        }
      } else { 
        //Create empty hosts array for zero length files
        splits.add(makeSplit(path, 0, length, new String[0]));
      }
    }
    // Save the number of input files for metrics/loadgen
    job.getConfiguration().setLong(NUM_INPUT_FILES, files.size());
    sw.stop();
    if (LOG.isDebugEnabled()) {
      LOG.debug("Total # of splits generated by getSplits: " + splits.size()
          + ", TimeTaken: " + sw.now(TimeUnit.MILLISECONDS));
    }
    return splits;
  }

//计算切片大小的方法
 protected long computeSplitSize(long blockSize, long minSize,
                                  long maxSize) {
    return Math.max(minSize, Math.min(maxSize, blockSize));
  }
//默认块的大小等于切片的大小
```

> 注意：存储是物理上的存储，按照128m进行存储，但是切片是逻辑上的切片操作。129如果是存储，那么存储两块，但是如果是切片的话，就会切成1片。

#### CombineTextInputFormat切片机制

1. 框架默认的TextInputFormat切片机制是对任务按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个MapTask，这样如果有大量小文件，就会产生大量的MapTask，处理效率极其低下。

2. 优化策略：

   - 最好的办法，在数据处理系统的最前端（**预处理/采集**），将小文件先合并成大文件，再上传到HDFS做后续分析。
   - 补救措施：如果已经是大量小文件在HDFS中了，可以使用另一种InputFormat来做切片（CombineTextInputFormat），它的切片逻辑跟TextFileInputFormat不同：**它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个maptask。**

3. 应用场景：

   **CombineTextInputFormat**用于小文件过多的场景，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个MapTask处理。

   > 注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

4. 优先满足最小切片大小，不超过最大切片大小

~~~ java
//设置切片允许的最大和最小值
CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m
CombineTextInputFormat.setMinInputSplitSize(job, 2097152);// 2m
//举例：0.5m+1m+0.3m+5m=2m + 4.8m=2m + 4m + 0.8m
//在驱动类中添加下面内容
// 9 如果不设置InputFormat,它默认用的是TextInputFormat.class
job.setInputFormatClass(CombineTextInputFormat.class)
CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m
CombineTextInputFormat.setMinInputSplitSize(job, 2097152);// 2m
//可以通过控制最大的切片大小控制切片的个数
~~~

5. 生成切片过程包括：**虚拟存储过程和切片过程二部分。**

   - 虚拟存储过程：

   将输入目录下所有文件大小，依次和设置的setMaxInputSplitSize值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值2倍，此时将文件均分成2个虚拟存储块（防止出现太小切片）。

   - 例如setMaxInputSplitSize值为4M，输入文件大小为8.02M，则先逻辑上分成一个4M。剩余的大小为4.02M，如果按照4M逻辑划分，就会出现0.02M的小的虚拟存储文件，所以将剩余的4.02M文件切分成（2.01M和2.01M）两个文件。

   - 切片过程：
     - 判断虚拟存储的文件大小是否大于setMaxInputSplitSize值，大于等于则单独形成一个切片。
     - 如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。
     - 测试举例：有4个小文件大小分别为1.7M、5.1M、3.4M以及6.8M这四个小文件，则虚拟存储之后形成6个文件块，大小分别为：


~~~ java
1.7M，（2.55M、2.55M），3.4M以及（3.4M、3.4M）

最终会形成3个切片，大小分别为：

（1.7+2.55）M，（2.55+3.4）M，（3.4+3.4）M
~~~

![1610931516712](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/18/085839-539791.png)

   

#### FileInputFormat实现类

1. 思考：在运行MapReduce程序时，输入的文件格式包括：基于行的日志文件、二进制格式文件、数据库表等。那么，针对不同的数据类型，MapReduce是如何读取这些数据的呢？

2. FileInputFormat常见的接口实现类包括：

   TextInputFormat(按照文件的大小进行切分，如果有很多小文件，那么就会产生很多碎片，key是long类型，行偏移量，value是text类型，是一行的内容)、

   KeyValueTextInputFormat（按照文件大小对文件进行切片，切完后第一列是key，剩下的内容是value）、NLineInputFormat（按照行数对文件进行切片,key是long类型，value是text类型）、CombineTextInputFormat（切片与设置的文件大小有关，key是long类型,value是text类型）和自定义InputFormat（跟默认切片一样）等。

**继承关系**

![1611623437846](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/26/091039-31518.png)

##### TextInputFormat

TextInputFormat是默认的FileInputFormat实现类。按行读取每条记录。**键是存储该行在整个文件中的起始字节偏移量， LongWritable类型。值是这行的内容，不包括任何行终止符（换行符和回车符），Text类型。**

~~~ java
//以下是一个示例，比如，一个分片包含了如下4条文本记录。
Rich learning form
Intelligent learning engine
Learning more convenient
From the real demand for more close to the enterprise
//每条记录表示为以下键/值对：
(0,Rich learning form)//偏移量代表每一行字符的个数
(19,Intelligent learning engine)
(47,Learning more convenient)
(72,From the real demand for more close to the enterprise)
~~~

##### KeyValueTextInputFormat

每一行均为一条记录，被分隔符分割为key，value。可以通过在驱动类中设置conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR, "\t");来设定分隔符。默认分隔符是tab（\t）。

~~~ java
//以下是一个示例，输入是一个包含4条记录的分片。其中——>表示一个（水平方向的）制表符。
line1 ——>Rich learning form
line2 ——>Intelligent learning engine
line3 ——>Learning more convenient
line4 ——>From the real demand for more close to the enterprise
//每条记录表示为以下键/值对：
(line1,Rich learning form)
(line2,Intelligent learning engine)
(line3,Learning more convenient)
(line4,From the real demand for more close to the enterprise)
//此时的键是每行排在制表符之前的Text序列。
~~~

###### KeyValueTextInputFormat案例演示

**需求**

统计输入文件中每一行的第一个单词相同的行数。

**输入**

~~~ java
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
~~~

**输出**

~~~ java
banzhang	2
xihuan	2
~~~

**过程说明**

![1610935994483](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/18/101316-511704.png)

**代码实现**

**mapper**

~~~ java
public class KVTextMapper extends Mapper<Text,Text,Text, IntWritable> {
    IntWritable v=new IntWritable();
    @Override
    protected void map(Text key, Text value, Context context) throws IOException, InterruptedException {
//         封装对象

//        写出
        context.write(key,v);
    }
}
~~~

**reducer**

~~~ java
public class KVTextReducer extends Reducer<Text, IntWritable,Text,IntWritable> {

    IntWritable v=new IntWritable();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
//        累加求和
        int sum=0;
        for (IntWritable value:values){
            sum+=value.get();
        }
        v.set(sum);
        context.write(key,v);
    }
}
~~~

**driver**

~~~ java
public class KVTDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
       args=new String[]{"E:\\file\\input","E:\\file\\input1"};
        Configuration conf = new Configuration();
//        设置切割方式,设置以空格切分
        conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR," ");


        Job job = Job.getInstance(conf);

        job.setJarByClass(KVTDriver.class);

        job.setMapperClass(KVTextMapper.class);
        job.setReducerClass(KVTextReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        job.setInputFormatClass(KeyValueTextInputFormat.class);

        FileInputFormat.setInputPaths(job,new Path(args[0]));
        FileOutputFormat.setOutputPath(job,new Path(args[1]));

        job.waitForCompletion(true);

    }
}
~~~

##### NLineInputFormat

如果使用NlineInputFormat，代表每个map进程处理的InputSplit不再按Block块去划分，而是按NlineInputFormat指定的行数N来划分。即输入文件的总行数/N=切片数，如果不整除，切片数=商+1

~~~ java 
//以下是一个示例，仍然以上面的4行输入为例。
Rich learning form
Intelligent learning engine
Learning more convenient
From the real demand for more close to the enterprise
//例如，如果N是2，则每个输入分片包含两行。开启2个MapTask。
(0,Rich learning form)
(19,Intelligent learning engine)
//另一个 mapper 则收到后两行：
(47,Learning more convenient)
(72,From the real demand for more close to the enterprise)
//这里的键和值与TextInputFormat生成的一样。
~~~

###### 案例演示

对每个单词进行个数统计，要求根据每个输入文件的行数来规定输出多少个切片。此案例要求每三行放入一个切片中。

**输入**

~~~ java
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang banzhang ni hao
xihuan hadoop banzhang
~~~

**过程**

![1610941408841](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/18/114330-986751.png)

**mapper**

~~~ java
public class NLineMapper extends Mapper<LongWritable, Text,Text, LongWritable> {
    LongWritable v=new LongWritable(1);
    Text k=new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
//        获取一行
        String line=value.toString();

//        对数据进行切割
        String []words=line.split(" ");

//        循环遍历写出
        for(String word:words){
            k.set(word);
            context.write(k,v);
        }
    }
}
~~~

**reducer**

~~~ java
public class NLineReducer extends Reducer<Text, LongWritable,Text,LongWritable> {
    LongWritable v=new LongWritable();

    @Override
    protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {
        int sum=0;
        for (LongWritable value:values){
            sum += value.get();

        }
        v.set(sum);
        context.write(key,v);
    }
}
~~~

**driver**

~~~ java
public class NLineDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
        args = new String[] {"E:\\file\\input\\nl.txt", "E:\\file\\output2" };

        // 1 获取job对象
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration);

        // 7设置每个切片InputSplit中划分三条记录
        NLineInputFormat.setNumLinesPerSplit(job, 3);

        // 8使用NLineInputFormat处理记录数
        job.setInputFormatClass(NLineInputFormat.class);

        // 2设置jar包位置，关联mapper和reducer
        job.setJarByClass(NLineDriver.class);
        job.setMapperClass(NLineMapper.class);
        job.setReducerClass(NLineReducer.class);

        // 3设置map输出kv类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        // 4设置最终输出kv类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);

        // 5设置输入输出数据路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 6提交job
        job.waitForCompletion(true);
    }
}
~~~

##### 自定义InputFormat

- 在企业开发中，Hadoop框架自带的InputFormat类型不能满足所有应用场景，需要自定义InputFormat来解决实际问题。

- 自定义InputFormat步骤如下：
  - 自定义一个类**继承FileInputFormat。**
  - 改写RecordReader，实现一次读取一个完整文件封装为KV。
  - 在输出时使用SequenceFileOutPutFormat（以文件名称为key，文件的内容为value存储）输出合并文件。

**InputFormat源码**

~~~ java
@InterfaceAudience.Public
@InterfaceStability.Stable
public abstract class InputFormat<K, V> {

  public abstract 
    List<InputSplit> getSplits(JobContext context
                               ) throws IOException, InterruptedException;
  
 
  public abstract 
    RecordReader<K,V> createRecordReader(InputSplit split,
                                         TaskAttemptContext context
                                        ) throws IOException, 
                                                 InterruptedException;
}
~~~

只需要实现类中的两个抽象方法即可。

###### 案例演示

无论HDFS还是MapReduce，在处理小文件时效率都非常低，但又难免面临处理大量小文件的场景，此时，就需要有相应解决方案。可以自定义InputFormat实现小文件的合并。

**需求**

将多个小文件合并成一个SequenceFile文件（SequenceFile文件是Hadoop用来存储二进制形式的key-value对的文件格式），SequenceFile里面存储着多个文件，存储的形式为文件路径+名称为key，文件内容为value。

~~~ java
//1、自定义一个类继承FileInputFormat
//（1）重写isSplitable()方法，返回false不可切割
//（2）重写createRecordReader()，创建自定义的RecordReader对象，并初始化
//2、改写RecordReader，实现一次读取一个完整文件封装为KV
//（1）采用IO流一次读取一个文件输出到value中，因为设置了不可切片，最终把所有文件都封装到了value中
//（2）获取文件路径信息+名称，并设置key
//3、设置Driver
// （1）设置输入的inputFormat
job.setInputFormatClass(WholeFileInputformat.class);
// （2）设置输出的outputFormat
job.setOutputFormatClass(SequenceFileOutputFormat.class);
//案例地址
https://github.com/justdoitMr/BigData_doc/tree/master/codes/hadoop/FileInputFormat/src/com/qq/rzf/FileInputFormat
~~~

上面合并小文件要和CombineTextInputFormat合并小文件区分开，自定义输入合并小文件最终是把多个文件归并为一个文件，对外表现的是一个文件，文件里面存储的是小文件的名字和文件对应的内容，而CombineTextInputFormat合并小文件，对外表现是一个文件，但是在内部还是相互独立的小文件。

**代码演示**

**RecordReader源码**

~~~ java
@InterfaceAudience.Public
@InterfaceStability.Stable
public abstract class RecordReader<KEYIN, VALUEIN> implements Closeable {

  /**
   * Called once at initialization.
   * @param split the split that defines the range of records to read
   * @param context the information about the task
   * @throws IOException
   * @throws InterruptedException
   */
  public abstract void initialize(InputSplit split,
                                  TaskAttemptContext context
                                  ) throws IOException, InterruptedException;

  /**
   * Read the next key, value pair.
   * @return true if a key/value pair was read
   * @throws IOException
   * @throws InterruptedException
   */
  public abstract 
  boolean nextKeyValue() throws IOException, InterruptedException;

  /**
   * Get the current key
   * @return the current key or null if there is no current key
   * @throws IOException
   * @throws InterruptedException
   */
  public abstract
  KEYIN getCurrentKey() throws IOException, InterruptedException;
  
  /**
   * Get the current value.
   * @return the object that was read
   * @throws IOException
   * @throws InterruptedException
   */
  public abstract 
  VALUEIN getCurrentValue() throws IOException, InterruptedException;
  
  /**
   * The current progress of the record reader through its data.
   * @return a number between 0.0 and 1.0 that is the fraction of the data read
   * @throws IOException
   * @throws InterruptedException
   */
  public abstract float getProgress() throws IOException, InterruptedException;
  
  /**
   * Close the record reader.
   */
  public abstract void close() throws IOException;
}
~~~

**自定义WhoFileInputFormat**

~~~ java
public class WhoFileInputFormat extends FileInputFormat<Text,BytesWritable> {

    @Override
    public RecordReader<Text, BytesWritable> createRecordReader(InputSplit inputSplit, TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
        WhoRecordReader recordReader = new WhoRecordReader();
//        初始化工作
        recordReader.initialize(inputSplit,taskAttemptContext);

        return recordReader;
    }
}
~~~

**改写RecordReader**

~~~ java
class WhoRecordReader extends RecordReader<Text,BytesWritable> {
    FileSplit fileSplit;
    Configuration conf;
    Text k=new Text();
    BytesWritable v=new BytesWritable();
    boolean isProcess=true;

    @Override
    public void initialize(InputSplit inputSplit, TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {

//        获取文件切片信息
        this.fileSplit= (FileSplit) inputSplit;
//        获取上下文信息
        conf=taskAttemptContext.getConfiguration();

    }

    @Override
    public boolean nextKeyValue() throws IOException, InterruptedException {
        if(isProcess){
            //        核心业务逻辑

            byte []buffer=new byte[(int)fileSplit.getLength()];
//        获取切片的路径
            Path path = fileSplit.getPath();
//        通过切片信息获取文件系统
            FileSystem fileSystem = path.getFileSystem(conf);

//        获取输入流
            FSDataInputStream fis = fileSystem.open(path);

//        拷贝数据到缓冲区
            IOUtils.readFully(fis,buffer,0,buffer.length);

//        将缓冲区中的数据读入v中
            v.set(buffer,0,buffer.length);

//        设置k值，文件的路径和名称
            k.set(path.toString());

//        关闭资源
            IOUtils.closeStream(fis);
            isProcess=false;
            return true;
        }

        return false;
    }

    @Override
    public Text getCurrentKey() throws IOException, InterruptedException {
        return k;
    }

    @Override
    public BytesWritable getCurrentValue() throws IOException, InterruptedException {
        return v;
    }

    @Override
    public float getProgress() throws IOException, InterruptedException {
        return 0;
    }

    @Override
    public void close() throws IOException {

    }
}
~~~

**mapper**

~~~ java
public class SequenceFileMapper extends Mapper<Text, BytesWritable,Text,BytesWritable> {

    @Override
    protected void map(Text key, BytesWritable value, Context context) throws IOException, InterruptedException {

//        直接写出文件即可
        context.write(key,value);
    }
}
~~~

**reducer**

~~~ java
public class SequenceReducer extends Reducer<Text, BytesWritable,Text,BytesWritable> {
    @Override
    protected void reduce(Text key, Iterable<BytesWritable> values, Context context) throws IOException, InterruptedException {
        for (BytesWritable value:values){
//            循环写出到一个文件中去
            context.write(key,value);
        }
    }
}
~~~

**driver**

~~~ java
public class SequenceDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
        args = new String[] { "e:/input/inputinputformat", "e:/output1" };

        // 1 获取job对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        // 2 设置jar包存储位置、关联自定义的mapper和reducer
        job.setJarByClass(SequenceDriver.class);
        job.setMapperClass(SequenceFileMapper.class);
        job.setReducerClass(SequenceReducer.class);

        // 7设置输入的inputFormat
        job.setInputFormatClass(WhoFileInputFormat.class);

        // 8设置输出的outputFormat
        job.setOutputFormatClass(SequenceFileOutputFormat.class);

// 3 设置map输出端的kv类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(BytesWritable.class);

        // 4 设置最终输出端的kv类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(BytesWritable.class);

        // 5 设置输入输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 6 提交job
        boolean result = job.waitForCompletion(true);
        System.exit(result ? 0 : 1);

    }
}
~~~



### MapReduce工作流程

**Mapper过程**

![1611276152425](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/22/084234-963207.png)

在submit()之前会形成切片信息

**Reducer过程**

![1611276890921](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/22/085453-977813.png)

上面的流程是整个MapReduce最全工作流程，但是Shuffle过程只是从第7步开始到第16步结束，具体Shuffle过程详解，如下：

1）MapTask收集我们的map()方法输出的kv对，放到内存缓冲区中（缓冲区其实就是一个队列，在写入的时候，缓冲区左侧写入的是元数据，右侧写入的是kv对）

2）从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件

3）多个溢出文件会被合并成大的溢出文件（其实就是序列化到磁盘中）

4）在溢出过程及合并的过程中，都要调用Partitioner进行分区和针对key进行排序（这里针对的是每一个分区内部排序）

5）ReduceTask根据自己的分区号，去各个MapTask机器上取相应的结果分区数据

6）ReduceTask会取到同一个分区的来自不同MapTask的结果文件，ReduceTask会将这些文件再进行合并（归并排序）

7）合并成大文件后，Shuffle的过程也就结束了，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法，有多少个分区，就会产生多少个reducer task,一个reducer task专门处理某一个分区的内容 ）

3．注意

Shuffle中的缓冲区大小会影响到MapReduce程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快。

缓冲区的大小可以通过参数调整，参数：io.sort.mb默认100M。

4．源码解析流程

![1611277898123](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/22/091140-261686.png)

### Shuffle机制

**shuffle过程**

![1611277994869](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/185614-855115.png)

- Mapreduce确保每个reducer的输入都是按键排序的。系统执行排序的过程（即将map输出作为输入传给reducer）称为shuffle，Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle。

1. mapreducer工作流

   1. MapTask工作机制

   ​	（1）Read阶段：Map Task通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。

   ​	（2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。

   ​	（3）Collect阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。

   ​	（4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

   ​	溢写阶段详情：

   ​	步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。

   ​	步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。

   ​	步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当期内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。

   ​	（5）Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

   ​	当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。

   ​	在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认100）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。

   ​	让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。

#### partition分区

- 问题引出：要求将统计结果按照条件输出到不同文件中（分区）。比如：将统计结果按照手机归属地不同省份输出到不同文件中（分区）

#### 默认分区

**源码解读**

~~~ java
public class HashPartitioner<K, V> extends Partitioner<K, V> {
  /** Use {@link Object#hashCode()} to partition. */
  public int getPartition(K key, V value, int numReduceTasks) {
    return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
    //numReduceTasks 表示reducerTask的个数，Integer.MAX_VALUE全是1，与任何数字与操作还是本身
  }
}
//默认分区是根据key的hashCode对reduceTasks个数取模得到的。用户没法控制哪个key存储到哪个分区
~~~

#### 自定义Partitioner步骤:

1. 自定义类继承Partitioner，重写getPartition()方法

~~~ java
public class CustomPartitioner extends Partitioner<Text, FlowBean> {
 	@Override
	public int getPartition(Text key, FlowBean value, int numPartitions) {
          // 控制分区代码逻辑
    … …
		return partition;
	}
}
~~~

2. 在Job驱动中，设置自定义Partitioner 

~~~ java
job.setPartitionerClass(CustomPartitioner.class);
~~~

3. 自定义Partition后，要根据自定义Partitioner的逻辑设置相应数量的ReduceTask

~~~ java
job.setNumReduceTasks(5);
~~~

4. 案例分析

~~~ java
例如：假设自定义分区数为5，则
job.setNumReduceTasks(1);会正常运行，只不过会产生一个输出文件
job.setNumReduceTasks(2);会报错
job.setNumReduceTasks(6);大于5，程序会正常运行，会产生空文件

~~~

#### 分区实战

**需求**

将统计结果按照手机归属地不同省份输出到不同文件中（分区）

**结果**

手机号136、137、138、139开头都分别放到一个独立的4个文件中，其他开头的放到一个文件中。最后一共产生5个文件

1. 自定义类继承Partitioner，重新getPartition()方法

~~~ java
public class ProvincePartitioner extends Partitioner<Text,FlowBean> {
    @Override
    public int getPartition(Text text, FlowBean flowBean, int i) {

//        key是手机号
//        bean是流量信息
//        首先获取手机号码的前三位
        String prePhoneNum = text.toString().substring(0,3);
//        表示有4个分区号码
        int partition=4;
//        partition必须从0开始
        if("136".equals(prePhoneNum)){
            partition=0;
        }else if("137".equals(prePhoneNum)){
            partition=1;
        }else if("138".equals(prePhoneNum)){
            partition=2;
        }else {
            partition=3;
        }
//        切记partition需要返回
        return partition;
    }
}

~~~

  		2. 在job驱动中，设置自定义partitioner： 

~~~ java
//        关联自定义的partition操作
        job.setPartitionerClass(ProvincePartitioner.class);
~~~

 	3. 自定义partition后，要根据自定义partitioner的逻辑设置相应数量的reduce task

~~~ java
//        还需要设置reducerTask的个数
        job.setNumReduceTasks(5);
//注意：
如果reduceTask的数量> getPartition的结果数，则会多产生几个空的输出文件part-r-000xx；
如果1<reduceTask的数量<getPartition的结果数，则有一部分分区数据无处安放，会Exception；
如果reduceTask的数量=1，则不管mapTask端输出多少个分区文件，最终结果都交给这一个reduceTask，最终也就只会产生一个结果文件 part-r-00000；
~~~

#### **排序**

- 排序是MapReduce框架中最重要的操作之一。Map Task和Reduce Task均会对数据（按照key）进行排序。该操作属于Hadoop的默认行为。任何应用程序中的数据均会被排序，而不管逻辑上是否需要。默认排序是按照字典顺序排序，且实现该排序的方法是快速排序

- 对于Map Task，它会将处理的结果暂时放到一个缓冲区中，当缓冲区使用率达到一定阈值后，再对缓冲区中的数据进行一次快速排序，并将这些有序数据写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行归并排序，以将这些文件合并成一个大的有序文件。

- 对于Reduce Task，它从每个Map Task上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则放到磁盘上，否则放到内存中。如果磁盘上文件数目达到一定阈值，则进行一次合并以生成一个更大文件；如果内存中文件大小或者数目超过一定阈值，则进行一次合并后将数据写到磁盘上。当所有数据拷贝完毕后，Reduce Task统一对内存和磁盘上的所有数据进行一次合并。

1. 每个阶段的默认排序

   - 部分排序：

   MapReduce根据输入记录的键对数据集排序。保证输出的每个文件内部排序。

   - 全排序：

   如何用Hadoop产生一个全局排序的文件？最简单的方法是使用**一个分区**。但该方法在处理大型文件时效率极低，因为一台机器必须处理所有输出文件，从而完全丧失了MapReduce所提供的并行架构。

   替代方案：首先创建一系列排好序的文件；其次，串联这些文件；最后，生成一个全局排序的文件。主要思路是使用一个分区来描述输出的全局排序。例如：可以为上述文件创建3个分区，在第一分区中，记录的单词首字母a-g，第二分区记录单词首字母h-n, 第三分区记录单词首字母o-z。

   - 辅助排序：（GroupingComparator分组）

   Mapreduce框架在记录到达reducer之前按键对记录排序，但键所对应的值并没有被排序。甚至在不同的执行轮次中，这些值的排序也不固定，因为它们来自不同的map任务且这些map任务在不同轮次中完成时间各不相同。一般来说，大多数MapReduce程序会避免让reduce函数依赖于值的排序。但是，有时也需要通过特定的方法对键进行排序和分组等以实现对值的排序。

#### WritableComparable排序案例实操（全排序）

**需求**

根据手机流量案例，把总的流量从大到小排序输出

**期望输出结果**

~~~ java
13509468723	7335	110349	117684
13736230513	2481	24681	27162
13956435636	132		1512	1644
~~~

**步骤**

![1611381088435](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/23/135130-669128.png)

**FlowBean实现WritableComparable接口重写排序方法**

~~~ java
public class FlowBean implements WritableComparable<FlowBean> {
    private long upFlow;
    private long downFlow;
    private long sumFlow;

    public FlowBean(){
        super();
    }

    public FlowBean(long upFlow, long downFlow) {
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        sumFlow=upFlow+downFlow;
    }

    @Override
    public int compareTo(FlowBean o) {
        if(this.getSumFlow()>o.getSumFlow())
        {
            return -1;
        }else if(this.getSumFlow()<o.getSumFlow()){
            return 1;
        }else {
            return 0;
        }
    }
//序列化方法
    @Override
    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeLong(upFlow);
        dataOutput.writeLong(downFlow);
        dataOutput.writeLong(sumFlow);

    }
//反序列化方法
    @Override
    public void readFields(DataInput dataInput) throws IOException {
        upFlow=dataInput.readLong();
        downFlow=dataInput.readLong();
        sumFlow=dataInput.readLong();

    }

    @Override
    public String toString() {
        return upFlow +"\t" + downFlow +"\t" + sumFlow ;
    }

    public long getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(long upFlow) {
        this.upFlow = upFlow;
    }

    public long getDownFlow() {
        return downFlow;
    }

    public void setDownFlow(long downFlow) {
        this.downFlow = downFlow;
    }

    public long getSumFlow() {
        return sumFlow;
    }

    public void setSumFlow(long sumFlow) {
        this.sumFlow = sumFlow;
    }
}
~~~

**Mapper**

~~~ java
public class FlowCountSortMapper extends Mapper<LongWritable, Text,FlowBean,Text> {

    FlowBean k=new FlowBean();
    Text v=new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
//        13509468723	7335	110349	117684
        String line=value.toString();
        String words[]=line.split("\t");

        String phoneNum=words[0];

        long uf=Long.parseLong(words[1]);
        long df=Long.parseLong(words[2]);
        long sf=Long.parseLong(words[3]);

        k.setUpFlow(uf);
        k.setDownFlow(df);
        k.setSumFlow(sf);

        v.set(phoneNum);
        context.write(k,v);
    }
}
~~~

**Reducer**

~~~ java
public class FlowCountSortReducer extends Reducer<FlowBean, Text,Text,FlowBean> {

    @Override
    protected void reduce(FlowBean key, Iterable<Text> values, Context context) throws IOException, InterruptedException {

//        直接遍历数据写出即可
        for(Text value:values){
            context.write(value, key);
        }
    }
}
~~~

**dr**iver

~~~ java
public class FlowCountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
        args = new String[]{"e:/output1","e:/output2"};

        // 1 获取配置信息，或者job对象实例
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration);

        // 2 指定本程序的jar包所在的本地路径
        job.setJarByClass(FlowCountDriver.class);

        // 3 指定本业务job要使用的mapper/Reducer业务类
        job.setMapperClass(FlowCountSortMapper.class);
        job.setReducerClass(FlowCountSortReducer.class);

        // 4 指定mapper输出数据的kv类型
        job.setMapOutputKeyClass(FlowBean.class);
        job.setMapOutputValueClass(Text.class);

        // 5 指定最终输出的数据的kv类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        // 6 指定job的输入原始文件所在目录
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
        boolean result = job.waitForCompletion(true);
        System.exit(result ? 0 : 1);

    }
}
~~~

- 自定义排序WritableComparable

~~~ java
//bean对象实现WritableComparable接口重写compareTo方法，就可以实现排序
@Override
public int compareTo(FlowBean o) {
	// 倒序排列，从大到小
	return this.sumFlow > o.getSumFlow() ? -1 : 1;
}
~~~

#### WritableComparable排序案例实操（区内排序）

**需求**

要求每个省份手机号输出的文件中按照总流量内部排序。

**过程**

![1611381649104](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/185622-33383.png)

**在全局排序的基础上添加分区代码**

~~~ java
public class ProvincePartition extends Partitioner<FlowBean, Text> {

    @Override
    public int getPartition(FlowBean flowBean, Text text, int i) {
        //        key是手机号
//        bean是流量信息
//        首先获取手机号码的前三位
        String prePhoneNum = text.toString().substring(0,3);
//        表示有4个分区号码
        int partition=4;
//        partition必须从0开始
        if("136".equals(prePhoneNum)){
            partition=0;
        }else if("137".equals(prePhoneNum)){
            partition=1;
        }else if("138".equals(prePhoneNum)){
            partition=2;
        }else {
            partition=3;
        }
//        切记partition需要返回
        return partition;
    }
}
~~~

**在driver类中进行设置**

~~~ java
//        关联自定义分区
        job.setPartitionerClass(ProvincePartition.class);
        job.setNumReduceTasks(5);

~~~

#### **Combiner合并**

- Combiner是MR程序中Mapper和Reducer之外的一种组件。

- Combiner组件的父类就是Reducer。

- Combiner和Reducer的区别在于运行的位置
  - Combiner是在每一个MapTask所在的节点运行;
  - Reducer是接收全局所有Mapper的输出结果；

- Combiner的意义就是对每一个MapTask的输出进行局部汇总，以减小网络传输量。

- Combiner能够应用的前提是不能影响最终的业务逻辑，而且，Combiner的输出kv应该跟Reducer的输入kv类型要对应起来。

![1611537608868](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/25/092009-564121.png)

1.  自定义Combiner实现步骤
   1. 自定义一个Combiner继承Reducer，重写Reduce方法

~~~ java
public class WordcountCombiner extends Reducer<Text, IntWritable, Text, IntWritable>{
	
	IntWritable v = new IntWritable();
	
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,
			Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
		
		int sum = 0;
		
		// 1 汇总操作
		for (IntWritable value : values) {
			sum += value.get();
		}
		
		v.set(sum);
		
		// 2 写出
		context.write(key, v);
	}
}
~~~

 	2. 在Job驱动类中设置：

~~~ java
job.setCombinerClass(WordcountCombiner.class);
~~~

##### Combiner单词合并案例

**需求**

统计过程中对每一个MapTask的输出进行局部汇总，以减小网络传输量即采用Combiner功能。

**期望数据输出**

期望：Combine输入数据多，输出时经过合并，输出数据降低。

![1611537702960](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/25/092144-808154.png)

**案例实现一**

在原来wordCount案例的基础上，增加一个WordcountCombiner类继承Reducer

~~~ java
public class WordCountCombiner extends Reducer<Text, IntWritable,Text,IntWritable> {
    IntWritable v=new IntWritable();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
//        累加求和
        int sum=0;
        for (IntWritable value:values){
            sum +=value.get();
        }
        v.set(sum);
        context.write(key,v);
    }
}
//在driver类中设置
//        设置combiner类
        job.setCombinerClass(WordCountCombiner.class);
~~~

**第二种方式实现**

将WordcountReducer作为Combiner在WordcountDriver驱动类中指定

~~~ java
// 指定需要使用Combiner，以及用哪个类作为Combiner的逻辑
job.setCombinerClass(WordcountReducer.class);
~~~

#### **GroupingComparator分组（辅助**排序）

分组排序是在把数据送到reducer端之前对其进行排序，对Reduce阶段的数据根据某一个或几个字段进行分组。

分组排序步骤：

1. 自定义类继承WritableComparator

2. 重写compare()方法

~~~ java
@Override
public int compare(WritableComparable a, WritableComparable b) {
		// 比较的业务逻辑
		… …

		return result;
}
~~~

3. 创建一个构造将比较对象的类传给父类

~~~ java
protected OrderGroupingComparator() {
		super(OrderBean.class, true);
}
~~~

##### GroupingComparator分组案例实操

**需求**

有如下订单，要求输出每一个订单编号中最贵的商品

| 订单id  | 商品id | 成交金额 |
| ------- | ------ | -------- |
| 0000001 | Pdt_01 | 222.8    |
| Pdt_02  | 33.8   |          |
| 0000002 | Pdt_03 | 522.8    |
| Pdt_04  | 122.4  |          |
| Pdt_05  | 722.4  |          |
| 0000003 | Pdt_06 | 232.8    |
| Pdt_02  | 33.8   |          |

**输出**

~~~ java
1	222.8
2	722.4
3	232.8
~~~

**需求分析**

- 利用“订单id和成交金额”作为key，可以将Map阶段读取到的所有订单数据按照id升序排序，如果id相同再按照金额降序排序，发送到Reduce。
- 在Reduce端利用groupingComparator将订单id相同的kv聚合成组，然后取第一个即是该订单中最贵商品，如图4-18所示。

![1611541018592](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/25/101659-985739.png)

> 注意：如果我们没有重写分组排序，按照key进行排序后直接输出第一个对象的话，是不可以的，因为对于reducer端会把所有的key相同的对象分到一个reducer中，但是我们这里是把一个bean对象作为key传递，每一个bean对象都不同，所以最后会把所有的记录全部输出。

**OrderGroupComparitor**

~~~ java
public class OrderGroupComparitor extends WritableComparator{

    protected OrderGroupComparitor(){
        super(OrderBean.class, true);
    }


    @Override
    public int compare(WritableComparable a, WritableComparable b) {
        OrderBean bean1=(OrderBean)a;
        OrderBean bean2=(OrderBean)b;
        int result=0;
//        这里根据id进行排序是因为相同id需要分为一组
        if(bean1.getPrice()>bean2.getId()){
            result=1;
        }else if(bean1.getId() < bean2.getId()){
            result=-1;
        }else {
//            两个对象的id 相同
            result=0;
        }
        return result;
    }
}
//在driver类下面添加
      // 8 设置reduce端的分组
        job.setGroupingComparatorClass(OrderGroupComparitor.class);
~~~

### MapTask工作机制

**图解**

![1611563565463](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/25/163247-224920.png)





- Read阶段：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。

- Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。

- Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。

- Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

**溢写阶段详情：**

1. 步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。

2. 步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。

3. 步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。

- Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。

在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。

让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。

### **ReduceTask工作机制**

**图解**

![1611563755167](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/25/163556-313976.png)

- Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

- Merge阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。

- Sort阶段：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。

- Reduce阶段：reduce()函数将计算结果写到HDFS上

**设置ReduceTask并行度（个数）**

ReduceTask的并行度同样影响整个Job的执行并发度和执行效率，但与MapTask的并发数由切片数决定不同，ReduceTask数量的决定是可以直接手动设置：

~~~ java
// 默认值是1，手动设置为4
job.setNumReduceTasks(4);
~~~

注意事项
1. ReduceTask=0，表示没有Reduce阶段，输出文件个数和Map个数一致。
2. ReduceTask默认值就是1，所以输出文件个数为一个。
3. 如果数据分布不均匀，就有可能在Reduce阶段产生**数据倾斜**
4. ReduceTask数量并不是任意设置，还要考虑业务逻辑需求，有些情况下，需要计算全局汇总结果，就只能有1个ReduceTask。
5. 具体多少个ReduceTask，需要根据集群性能而定。
6. 如果分区数不是1，但是ReduceTask为1，是否执行分区过程。答案是：不执行分区过程。因为在MapTask的源码中，执行分区的前提是先判断ReduceNum个数是否大于1。不大于1肯定不执行。

> 有分区的话，分区个数要和reducer个数保持一致。

### OutputFormat数据输出

OutputFormat是MapReduce输出的基类，所有实现MapReduce输出都实现了 OutputFormat接口。下面我们介绍几种常见的OutputFormat实现类。

![1611622659683](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/26/085743-524890.png)

1. 文本输出TextOutputFormat

   默认的输出格式是TextOutputFormat，它把每条记录写为文本行。它的键和值可以是任意类型，因为TextOutputFormat调用toString()方法把它们转换为字符串。

2. SequenceFileOutputFormat

    将SequenceFileOutputFormat输出作为后续 MapReduce任务的输入，这便是一种好的输出格式，因为它的格式紧凑，很容易被压缩。

3. 自定义OutputFormat

   根据用户需求，自定义实现输出。

#### 自定义OutputFormat

 1. 使用场景：

    为了实现控制最终文件的输出路径和输出格式，可以自定义OutputFormat。例如：要在一个MapReduce程序中根据数据的不同输出两类结果到不同目录，这类灵活的输出需求可以通过自定义OutputFormat来实现。

2. 自定义OutputFormat步骤
   1. 自定义一个类继承FileOutputFormat。
   2. 改写RecordWriter，具体改写输出数据的方法write()。

#### 自定义OutputFormat案例实操

**需求**

过滤输入的log日志，包含atguigu的网站输出到e:/atguigu.log，不包含atguigu的网站输出到e:/other.log。

**过程**

![1611622859480](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/185635-655515.png)

**Mapper**

~~~ java
public class FilterMapper extends Mapper<LongWritable, Text,Text, NullWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

//        在这里直接把文本写出即可
        context.write(value,NullWritable.get());
    }
}
~~~

**Reducer**

~~~ java
public class FilterReducer extends Reducer<Text, NullWritable,Text,NullWritable> {
    @Override
    protected void reduce(Text key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
//        防止有重复数据，所以使用循环写出
        for(NullWritable value:values){
            context.write(key,NullWritable.get());
        }
    }
}
~~~

**FilterOutputFormat**

~~~ java
public class FilterOutputFormat extends FileOutputFormat<Text,NullWritable> {

    @Override
    public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
        return new FRecordWriter(taskAttemptContext);
    }
}

class FRecordWriter extends RecordWriter<Text,NullWritable>{
    FSDataOutputStream dataOutputStream ;
    FSDataOutputStream dataOutputStream1;

    public FRecordWriter(TaskAttemptContext taskAttemptContext) throws IOException {
        //        首先获取文件系统,使用配置信息通过反射的方式获取
        FileSystem fileSystem = FileSystem.get(taskAttemptContext.getConfiguration());

//        创建输出流
         dataOutputStream = fileSystem.create(new Path("E:\\file\\output3"));
         dataOutputStream1 = fileSystem.create(new Path("E:\\file\\output4"));

    }

    @Override
    public void write(Text text, NullWritable nullWritable) throws IOException, InterruptedException {
//        写具体的业务逻辑
        if(text.toString().contains("atguigu")){
            dataOutputStream.write(text.toString().getBytes());
        }else {
            dataOutputStream1.write(text.toString().getBytes());
        }
    }

    @Override
    public void close(TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
//        关闭资源
        IOUtils.closeStream(dataOutputStream);
        IOUtils.closeStream(dataOutputStream1);
    }
}
~~~

**driver**

~~~ java
public class FilterDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
        args = new String[] { "e:/input/inputoutputformat", "e:/output2" };

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        job.setJarByClass(FilterDriver.class);
        job.setMapperClass(FilterMapper.class);
        job.setReducerClass(FilterReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(NullWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        // 要将自定义的输出格式组件设置到job中
        job.setOutputFormatClass(FilterOutputFormat.class);

        FileInputFormat.setInputPaths(job, new Path(args[0]));
        // 虽然我们自定义了outputformat，但是因为我们的outputformat继承自fileoutputformat
        // 而fileoutputformat要输出一个_SUCCESS文件，所以，在这还得指定一个输出目录
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        boolean result = job.waitForCompletion(true);
        System.exit(result ? 0 : 1);

    }
}
~~~

### Join的多种应用

#### Reduce Join

**工作原理**

Map端的主要工作：为来自不同表或文件的key/value对，打标签以区别不同来源的记录。然后用连接字段作为key，其余部分和新加的标志作为value，最后进行输出。

Reduce端的主要工作：在Reduce端以连接字段作为key的分组已经完成，我们只需要在每一个分组当中将那些来源于不同文件的记录(在Map阶段已经打标志)分开，最后进行合并就ok了。

#### Reduce Join案例实操

**订单数据表**

| id   | pid  | amount |
| ---- | ---- | ------ |
| 1001 | 01   | 1      |
| 1002 | 02   | 2      |
| 1003 | 03   | 3      |
| 1004 | 01   | 4      |
| 1005 | 02   | 5      |
| 1006 | 03   | 6      |

**商品信息表**

| pid  | pname |
| ---- | ----- |
| 01   | 小米  |
| 02   | 华为  |
| 03   | 格力  |

将商品信息表中数据根据商品pid合并到订单数据表中。

| id   | pname | amount |
| ---- | ----- | ------ |
| 1001 | 小米  | 1      |
| 1004 | 小米  | 4      |
| 1002 | 华为  | 2      |
| 1005 | 华为  | 5      |
| 1003 | 格力  | 3      |
| 1006 | 格力  | 6      |

**需求分析**

通过将关联条件作为Map输出的key，将两表满足Join条件的数据并携带数据所来源的文件信息，发往同一个ReduceTask，在Reduce中进行数据的串联，如图4-20所示。

![1611637338006](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/26/130219-278454.png)

**bean类**

~~~ java
public class TableBean implements Writable {

    private String id;//订单的id
    private String pid;//产品的id
    private int amount;//产品数量
    private String name;//产品名称
    private String flag;//标记是产品表还是订单表

    public TableBean(){
        super();
    }

    public TableBean(String id, String pid, int amount, String name, String flag) {
        this.id = id;
        this.pid = pid;
        this.amount = amount;
        this.name = name;
        this.flag = flag;
    }

    @Override
    public void write(DataOutput dataOutput) throws IOException {

        dataOutput.writeUTF(id);
        dataOutput.writeUTF(pid);
        dataOutput.writeInt(amount);
        dataOutput.writeUTF(name);
        dataOutput.writeUTF(flag);

    }

    @Override
    public void readFields(DataInput dataInput) throws IOException {
        id=dataInput.readUTF();
        pid=dataInput.readUTF();
        amount=dataInput.readInt();
        name=dataInput.readUTF();
        flag=dataInput.readUTF();

    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getPid() {
        return pid;
    }

    public void setPid(String pid) {
        this.pid = pid;
    }

    public int getAmount() {
        return amount;
    }

    public void setAmount(int amount) {
        this.amount = amount;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getFlag() {
        return flag;
    }

    public void setFlag(String flag) {
        this.flag = flag;
    }

    @Override
    public String toString() {
        return id +"\t"+ amount+"\t"+name ;
    }
}
~~~

**Mapper**

~~~ java
public class TableMapper extends Mapper<LongWritable, Text,Text,TableBean> {
    String name;
    TableBean bean=new TableBean();
    Text k=new Text();


    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
//        获取文件的名称
        FileSplit inputSplit = (FileSplit) context.getInputSplit();
        name = inputSplit.getPath().getName();
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
//        获取一行
        String line = value.toString();

        if(line.startsWith("order")){
            String []fields=line.split("\t");
            bean.setId(fields[0]);
            bean.setPid(fields[1]);
            bean.setAmount(Integer.parseInt(fields[2]));
            bean.setName("");
            bean.setFlag("order");
            k.set(fields[1]);
        }else {
            String []fields=line.split("\t");
            bean.setId("");
            bean.setPid(fields[0]);
            bean.setAmount(0);
            bean.setName(fields[1]);
            bean.setFlag("pd");
            k.set(fields[0]);
        }
        context.write(k,bean);
    }
}
~~~

**Reducer**

~~~ java
public class TableReducer extends Reducer<Text,TableBean,TableBean, NullWritable> {
    @Override
    protected void reduce(Text key, Iterable<TableBean> values, Context context) throws IOException, InterruptedException {

//        存储订单信息表
        ArrayList <TableBean>orderList=new ArrayList();
//        存储产品信息
        TableBean tableBean = new TableBean();
        for (TableBean value:values) {
            if("order".equals(value.getFlag()))
            {
                //            为什么要在这里创建一个对象，使用拷贝的形式？
//            因为value里面持有的是TableBean的引用，而不是真正的一个对象
                TableBean temp = new TableBean();
                try {
//                value里面只是对象的引用
//                使用拷贝的话，就是把value引用指向的对象的数据拷贝到新对象里面，然后在进行添加
                    BeanUtils.copyProperties(temp,value);
                    orderList.add(temp);
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                } catch (InvocationTargetException e) {
                    e.printStackTrace();
                }
            }else {
                try {
//                    只有一个产品表
                    BeanUtils.copyProperties(tableBean,value);
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                } catch (InvocationTargetException e) {
                    e.printStackTrace();
                }
            }

//            遍历订单表，设置名字
            for (TableBean v:orderList) {
                v.setName(tableBean.getName());
                context.write(v,NullWritable.get());
            }
        }

    }
}
~~~

**driver**

~~~ java
// 0 根据自己电脑路径重新配置
        args = new String[]{"E:\\file\\in", "E:\\file/output3"};

// 1 获取配置信息，或者job对象实例
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration);

        // 2 指定本程序的jar包所在的本地路径
        job.setJarByClass(TableDriver.class);

        // 3 指定本业务job要使用的Mapper/Reducer业务类
        job.setMapperClass(TableMapper.class);
        job.setReducerClass(TableReducer.class);

        // 4 指定Mapper输出数据的kv类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(TableBean.class);

        // 5 指定最终输出的数据的kv类型
        job.setOutputKeyClass(TableBean.class);
        job.setOutputValueClass(NullWritable.class);

        // 6 指定job的输入原始文件所在目录
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
        boolean result = job.waitForCompletion(true);
        System.exit(result ? 0 : 1);
~~~

#### Map Join

**使用场景**

Map Join适用于一张表十分小、一张表很大的场景。

**优点**

思考：在Reduce端处理过多的表，非常容易产生数据倾斜。怎么办？

在Map端缓存多张表，提前处理业务逻辑，这样增加Map端业务，减少Reduce端数据的压力，尽可能的减少数据倾斜。

**具体办法：采用DistributedCache**

在Mapper的setup阶段，将文件读取到缓存集合中。

- 在驱动函数中加载缓存。

- // 缓存普通文件到Task运行节点。

~~~ java
job.addCacheFile(new URI("file://e:/cache/pd.txt"));
~~~

#### Map Join案例实操

**需求**

订单表

| id   | pid  | amount |
| ---- | ---- | ------ |
| 1001 | 01   | 1      |
| 1002 | 02   | 2      |
| 1003 | 03   | 3      |
| 1004 | 01   | 4      |
| 1005 | 02   | 5      |
| 1006 | 03   | 6      |

**商品表**

| pid  | pname |
| ---- | ----- |
| 01   | 小米  |
| 02   | 华为  |
| 03   | 格力  |

**结果**

| id   | pname | amount |
| ---- | ----- | ------ |
| 1001 | 小米  | 1      |
| 1004 | 小米  | 4      |
| 1002 | 华为  | 2      |
| 1005 | 华为  | 5      |
| 1003 | 格力  | 3      |
| 1006 | 格力  | 6      |

**分析**

![1611637719288](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/185648-771221.png)

**在驱动中添加缓存文件**

~~~ java
public class DistributeCacheDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException, URISyntaxException {
        // 0 根据自己电脑路径重新配置
        args = new String[]{"e:/input/inputtable2", "e:/output1"};

// 1 获取job信息
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration);

        // 2 设置加载jar包路径
        job.setJarByClass(DistributedCacheDriver.class);

        // 3 关联map
        job.setMapperClass(DistributedCacheDriver.class);

// 4 设置最终输出数据类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        // 5 设置输入输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 6 加载缓存数据,也就是把缓存数据加载到集群上
        job.addCacheFile(new URI("file:///e:/input/inputcache/pd.txt"));

        // 7 Map端Join的逻辑不需要Reduce阶段，设置reduceTask数量为0
        job.setNumReduceTasks(0);

        // 8 提交
        boolean result = job.waitForCompletion(true);
        System.exit(result ? 0 : 1);

    }
}
~~~

**Mapper**

~~~ java
public class DistributedCacheDriver extends Mapper<LongWritable, Text,Text, NullWritable> {
    Map <String,String>hashmap=new HashMap<>();
    Text k=new Text();

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
//        获取缓存文件的路径
        URI[] cacheFiles = context.getCacheFiles();
        String path = cacheFiles[0].getPath().toString();


//        用来缓存小表,第二个参数是编码方式
        BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(path), "UTF-8"));
        String line;
        while (StringUtils.isNotEmpty(line=reader.readLine())){
//            读取一行并且判断是否是空
//            切割数据
            String[] split = line.split("\t");
//            [0]:是产品的pid [1]:是产品对应的名
            hashmap.put(split[0],split[1]);
        }

//        关闭资源
        IOUtils.closeStream(reader);
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {


//       获取一行
        String line=value.toString();
//        切割数据
        String[] words = line.split("\t");
//        获取产品名字
        String pid=words[1];
        String name = hashmap.get(pid);
//        拼接数据
        line=line+"\t"+name;
        k.set(line);
        context.write(k,NullWritable.get());
    }
} 
~~~

### 计数器的应用

Hadoop为每个作业维护若干内置计数器，以描述多项指标。例如，某些计数器记录已处理的字节数和记录数，使用户可监控已处理的输入数据量和已产生的输出数据量。

**计数器的API**

~~~ java
//1 采用枚举的方式统计计数
enum MyCounter{MALFORORMED,NORMAL}
//对枚举定义的自定义计数器加1
context.getCounter(MyCounter.MALFORORMED).increment(1);
//2 采用计数器组、计数器名称的方式统计
context.getCounter("counterGroup", "counter").increment(1);
//组名和计数器名称随便起，但最好有意义。
//计数结果在程序运行后的控制台上查看。
~~~

### 数据清洗ETL

在运行核心业务MapReduce程序之前，往往要先对数据进行清洗，清理掉不符合用户要求的数据。清理的过程往往只需要运行Mapper程序，不需要运行Reduce程序。

#### 数据清洗案例实操-简单解析版

**需求**

去除日志中字段长度小于等于11的日志。

**期望输出**

每行字段长度都大于11。

**分分析**

需要在Map阶段对输入的数据根据规则进行过滤清洗。

**Mapper类**

~~~ java
public class MapperLog extends Mapper<LongWritable,Text, Text, NullWritable> {

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
//        获取一行数据
        String line=value.toString();

        boolean result=parselog(line,context);

        if(!result){
            return;
        }

        context.write(value,NullWritable.get());
    }

    private boolean parselog(String line, Context context) {

//        判断字符串长度是否大于11
//        以空格切分字符串
        String[] fields = line.split(" ");
//        判断长度
        if(fields.length > 11){
//            获取一个计数器，判断true的次数
//            参数表示在map阶段，判断true的次数
            context.getCounter("map","true").increment(1);
            return true;
        }else {
            context.getCounter("map","false").increment(1);
            return false;
        }
    }
}
~~~

**driver类**

~~~ java
public class DriverLog {
    public static void main(String[] args) throws Exception {

// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
        args = new String[] { "E:\\file\\input\\log.txt", "E:\\file\\output5" };

        // 1 获取job信息
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        // 2 加载jar包
        job.setJarByClass(DriverLog.class);

        // 3 关联map
        job.setMapperClass(MapperLog.class);

        // 4 设置最终输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        // 设置reducetask个数为0
        job.setNumReduceTasks(0);

        // 5 设置输入和输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 6 提交
        job.waitForCompletion(true);
    }
}
~~~

### MapReduce开发总结

切片大小：集群是128m，本地是32m。129m切1片。

1. 输入数据接口：InputFormat
   1. 默认使用的实现类是：TextInputFormat ，切片默认按照块大小进行切片，对单个文件进行切片，key是偏移量,v是每一行的内容。
   2. TextInputFormat的功能逻辑是：一次读一行文本，然后将该行的起始偏移量作为key，行内容作为value返回。
   3. KeyValueTextInputFormat每一行均为一条记录，被分隔符分割为key，value。默认分隔符是tab（\t），切片是按照块大小切片。
   4. NlineInputFormat按照指定的行数N来划分切片，改变的是切片，没有改变key,value。
   5. CombineTextInputFormat可以把多个小文件合并成一个切片处理，提高处理效率，改变的是切片，设置最大值，小于最大值的话会合并到一起。
   6. 用户还可以自定义InputFormat，没有改变切片，改变key,value,key是文件的名称或者路径，value是文件的内容。
2. 逻辑处理接口：Mapper 
   1. 用户根据业务需求实现其中三个方法：map()（实现业务逻辑），   setup()(初始化时候用)   cleanup () （关闭资源使用）。
3. Partitioner分区
   1. 有默认实现 HashPartitioner，逻辑是根据key的哈希值和numReduces来返回一个分区号；key.hashCode()&Integer.MAXVALUE % numReduces
   2. 如果业务上有特别的需求，可以自定义分区，可以分多个区，分区号从0开始累计。
4. Comparable排序
   1. 当我们用自定义的对象作为key来输出时，就必须要实现WritableComparable接口，重写其中的compareTo()方法。
   2. 部分排序：对最终输出的每一个文件进行内部排序，也就是分区内部有序。
   3. 全排序：对所有数据进行排序，通常只有一个Reduce，最后输出只有一个文件，并且有序。
   4. 二次排序：排序的条件有两个。
5. Combiner合并
   1. Combiner合并可以提高程序执行效率，减少IO传输。但是使用时必须不能影响原有的业务处理结果。
6. Reduce端分组：GroupingComparator，对最终结果进行分组。
   1.  在Reduce端对key进行分组。应用于：在接收的key为bean对象时，想让一个或几个字段相同（全部字段比较不相同）的key进入到同一个reduce方法时，可以采用分组排序。
7. 逻辑处理接口：Reducer
   1. 用户根据业务需求实现其中三个方法：reduce()   setup()（初始化）   cleanup () 
8. 输出数据接口：OutputFormat
   1. 默认实现类是TextOutputFormat，功能逻辑是：将每一个KV对，向目标文本文件输出一行。
   2. 将SequenceFileOutputFormat输出作为后续 MapReduce任务的输入，这便是一种好的输出格式，因为它的格式紧凑，很容易被压缩。作为下一级mapreducer的输入文件。
   3. 用户还可以自定义OutputFormat。
9. 下图中画红框的部分都是可以自定义的部分

![1611715562244](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/27/104605-746806.png)

**reducer中自定义部分**

![1611715629687](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/27/104712-477610.png)



## MapReduce数据压缩

### 压缩概述

压缩技术能够有效减少底层存储系统（HDFS）读写字节数。压缩提高了网络带宽和磁盘空间的效率。在Hadood下，尤其是数据规模很大和工作负载密集的情况下，使用数据压缩显得非常重要。在这种情况下，I/O操作和网络数据传输要花大量的时间。还有，Shuffle与Merge过程同样也面临着巨大的I/O压力。

鉴于磁盘I/O和网络带宽是Hadoop的宝贵资源，数据压缩对于节省资源、最小化磁盘I/O和网络传输非常有帮助。不过，尽管压缩与解压操作的CPU开销不高，其性能的提升和资源的节省并非没有代价。

如果磁盘I/O和网络带宽影响了MapReduce作业性能，在任意MapReduce阶段启用压缩都可以改善端到端处理时间并减少I/O和网络流量。

压缩**mapreduce的一种优化策略：通过压缩编码对mapper或者reducer的输出进行压缩，以减少磁盘IO，**提高MR程序运行速度（但相应增加了cpu运算负担）

注意：压缩特性运用得当能提高性能，但运用不当也可能降低性能

基本原则：

（1）运算密集型的job，少用压缩

（2）IO密集型的job，多用压缩

### **MR支持的压缩编码**

| 压缩格式 | hadoop自带？ | 算法    | 文件扩展名 | 是否可切分 | 换成压缩格式后，原来的程序是否需要修改 |
| -------- | ------------ | ------- | ---------- | ---------- | -------------------------------------- |
| DEFLATE  | 是，直接使用 | DEFLATE | .deflate   | 否         | 和文本处理一样，不需要修改             |
| Gzip     | 是，直接使用 | DEFLATE | .gz        | 否         | 和文本处理一样，不需要修改             |
| bzip2    | 是，直接使用 | bzip2   | .bz2       | 是         | 和文本处理一样，不需要修改             |
| LZO      | 否，需要安装 | LZO     | .lzo       | 是         | 需要建索引，还需要指定输入格式         |
| Snappy   | 否，需要安装 | Snappy  | .snappy    | 否         | 和文本处理一样，不需要修改             |

- 为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器，如下表所示。

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

### 压缩方式选择

1. Gzip压缩

   - 优点：压缩率比较高，而且压缩/解压速度也比较快；Hadoop本身支持，在应用中处理Gzip格式的文件就和直接处理文本一样；大部分Linux系统都自带Gzip命令，使用方便。

   - 缺点：**不支持Split。**

   应用场景：当每个文件压缩之后在130M以内的（1个块大小内），都可以考虑用Gzip压缩格式。例如说一天或者一个小时的日志压缩成一个Gzip文件

2. bzip2压缩

   - 优点：支持Split；具有很高的压缩率，比Gzip压缩率都高；Hadoop本身自带，使用方便。

   - 缺点：压缩/解压速度慢。

   应用场景：适合对速度要求不高，但需要较高的压缩率的时候；或者输出之后的数据比较大，处理之后的数据需要压缩存档减少磁盘空间并且以后数据用得比较少的情况；或者对单个很大的文本文件想压缩减少存储空间，同时又需要支持Split，而且兼容之前的应用程序的情况。

3. Lzo压缩

   - 优点：压缩/解压速度也比较快，合理的压缩率；支持Split，是Hadoop中最流行的压缩格式；可以在Linux系统下安装lzop命令，使用方便

   - 缺点：压缩率比Gzip要低一些；Hadoop本身不支持，需要安装；在应用中对Lzo格式的文件需要做一些特殊处理（为了支持Split需要建索引，还需要指定InputFormat为Lzo格式）。

   应用场景：一个很大的文本文件，压缩之后还大于200M以上的可以考虑，而且单个文件越大，Lzo优点越越明显。

4. Snappy压缩

   - 优点：高速压缩速度和合理的压缩率。

   - 缺点：不支持Split；压缩率比Gzip要低；Hadoop本身不支持，需要安装。

   应用场景：当MapReduce作业的Map输出的数据比较大的时候，作为Map到Reduce的中间数据的压缩格式；或者作为一个MapReduce作业的输出和另外一个MapReduce作业的输入。

### **压缩位置选择**

- 压缩可以在MapReduce作用的任意阶段启用

![1611726422736](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/27/134704-366393.png)



### 压缩参数配置

要在Hadoop中启用压缩，可以配置如下参数：

| 参数                                                         | 默认值                                                       | 阶段        | 建议                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ----------- | --------------------------------------------- |
| io.compression.codecs      （在core-site.xml中配置）         | org.apache.hadoop.io.compress.DefaultCodec,   org.apache.hadoop.io.compress.GzipCodec,   org.apache.hadoop.io.compress.BZip2Codec | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器  |
| mapreduce.map.output.compress（在mapred-site.xml中配置）     | false                                                        | mapper输出  | 这个参数设为true启用压缩                      |
| mapreduce.map.output.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec                   | mapper输出  | 企业多使用LZO或Snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress（在mapred-site.xml中配置） | false                                                        | reducer输出 | 这个参数设为true启用压缩                      |
| mapreduce.output.fileoutputformat.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.   DefaultCodec                | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2       |
| mapreduce.output.fileoutputformat.compress.type（在mapred-site.xml中配置） | RECORD                                                       | reducer输出 | SequenceFile输出使用的压缩类型：NONE和BLOCK   |

### 数据压缩案例



CompressionCodec有两个方法可以用于轻松地压缩或解压缩数据。

- 要想对正在被写入一个输出流的数据进行压缩，我们可以使用createOutputStream(OutputStreamout)方法创建一个CompressionOutputStream，将其以压缩格式写入底层的流。
- 相反，要想对从输入流读取而来的数据进行解压缩，则调用createInputStream(InputStreamin)函数，从而获得一个CompressionInputStream，从而从底层的流读取未压缩的数据。

**测试压缩方式**

| DEFLATE | org.apache.hadoop.io.compress.DefaultCodec |
| ------- | ------------------------------------------ |
| gzip    | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2   | org.apache.hadoop.io.compress.BZip2Codec   |

**代码说明**

~~~ java
public class TestCompress {
    public static void main(String[] args) throws IOException, ClassNotFoundException {

//        对文件进行压缩
//        第一个参数是压缩的文件，第二个是压缩的格式
        compress("E:\\file\\input\\log.txt","org.apache.hadoop.io.compress.BZip2Codec");

//        加压缩操作
        deCompress("E:\\file\\input\\log.txt.bz2");
    }

    private static void deCompress(String s) throws IOException {
//        1 压缩方式检查
        CompressionCodecFactory compressionCodecFactory = new CompressionCodecFactory(new Configuration());
//        获取压缩方式
        CompressionCodec codec = compressionCodecFactory.getCodec(new Path(s));

        if (codec == null) {
            System.out.println("不支持的压缩");
            return;
        }

//       2 获取解压文件的输入流
        FileInputStream fileInputStream = new FileInputStream(new File(s));

        CompressionInputStream inputStream = codec.createInputStream(fileInputStream);

//        3 创建普通文件的输出流
        FileOutputStream fileOutputStream = new FileOutputStream(new File(s + "decode"));

//        4 拷贝输出
        IOUtils.copyBytes(inputStream, fileOutputStream, 1024 * 1024 * 5, false);

//       5 关闭资源
        IOUtils.closeStream(fileOutputStream);
        IOUtils.closeStream(inputStream);
        IOUtils.closeStream(fileInputStream);
    }


    private static void compress(String s, String s1) throws IOException, ClassNotFoundException {

//        1 获取普通文件的输入流
        FileInputStream fileInputStream = new FileInputStream(new File(s));
//        通过反射获取压缩类的对象
        Class codeClass = Class.forName(s1);

//        通过反射方法创建一个压缩类的对象
        CompressionCodec code = (CompressionCodec) ReflectionUtils.newInstance(codeClass, new Configuration());


//        2 获取输出流,文件需要添加上文件的压缩类型
        FileOutputStream fileOutputStream = new FileOutputStream(new File(s + code.getDefaultExtension()));

//        创建一个具有真正意义上的压缩输出流,普通的输出流放到可以压缩文件的输出流中
        CompressionOutputStream outputStream = code.createOutputStream(fileOutputStream);

//        3 流的对拷贝,从输入流的对象拷贝到可以压缩的输出流，boolean表示最后是否关闭输入和输出流
        IOUtils.copyBytes(fileInputStream, outputStream, 1024 * 1024 * 5, false);

//        关闭流
        IOUtils.closeStream(outputStream);
        IOUtils.closeStream(fileOutputStream);
        IOUtils.closeStream(fileInputStream);
    }
}
~~~

#### Map输出端采用压缩

即使你的MapReduce的输入输出文件都是未压缩的文件，你仍然可以对Map任务的中间结果输出做压缩，因为它要写在硬盘并且通过网络传输到Reduce节点，对其压缩可以提高很多性能，这些工作只要设置两个属性即可，我们来看下代码怎么设置。

~~~ java
//给大家提供的Hadoop源码支持的压缩格式有：BZip2Codec 、DefaultCodec
//在驱动文件中添加以下代码
// 开启map端输出压缩
	configuration.setBoolean("mapreduce.map.output.compress", true);
// 设置map端输出压缩方式
configuration.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class, CompressionCodec.class);
~~~

#### Reduce输出端采用压缩

~~~ java
//在驱动文件中添加如下代码
// 设置reduce端输出压缩开启
		FileOutputFormat.setCompressOutput(job, true);
		
		// 设置压缩的方式
	    FileOutputFormat.setOutputCompressorClass(job, BZip2Codec.class); 
~~~





