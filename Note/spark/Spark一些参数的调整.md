# Spark一些参数的调整

## 通用参数设置规范

![1652422840274](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202205/13/142041-951333.png)

![1652422860905](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202205/13/142101-932790.png)

## 针对dorado

特殊case说明：当任务存在扫event_log表时需注意，若对event_log表进行了过滤，且过滤比很高，如下图的case，input为74T，但shuffle write仅为3.5G，那么建议提高单partition的读取数据量，将参数set spark.sql.files.maxPartitionBytes=536870912提高10倍至5368709120；

![1653397283437](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202205/24/210124-155443.png)

> spark.sql.files.maxPartitionBytes每一个partitioin默认读取数据的最大数量，默认128m。

## 参数说明

目前测试：在不手动添加任何参数、平均时长在30min以内、单个shuffle 量在500G以下的任务可以使用该模版，但实际任务情况还需跟踪观察。

### 内存资源分配

-设置spark任务的优先级，数值设置的越大，优先级就越高，在同一时间提交的作业具有更高的优先级执行，获取更多的资源。
set spark.yarn.priority=9

 driver内存设置：set spark.driver.memory=15g;

driver核心数设置：set spark.driver.cores=3;

driver堆外内存的大小：set spark.driver.memoryOverhead=4096;spark内存=jvm内存+堆外内存

executor内存设置：set spark.executor.memory=5G;

> 1. 该参数用来设置每个Executor进程的内存。Executor内存的大小，很多时候直接决定了Spark作业的性能，而且跟常见的JVM OOM异常，也有直接的关联。
> 2.  每个Executor进程的内存设置4G-8G较为合适。但是这只是一个参考值，具体的设置还是得根据不同部门的资源队列来定。可以看看自己团队的资源队列的最大内存限制是多少，num-executors乘以executor-memory，就代表了你的Spark作业申请到的总内存量（也就是所有Executor进程的内存总和），这个量是不能超过队列的最大内存量的。
> 3. 此外，如果你是跟团队里其他人共享这个资源队列，那么申请的总内存量最好不要超过资源队列最大总内存的1/3-1/2，避免你自己的Spark作业占用了队列所有的资源，导致别的同学的作业无法运行。

executor堆外内存设置：set spark.executor.memoryOverhead=1024;

executor的核心数设置：set spark.executor.cores=2;

> 1. 该参数用于设置每个Executor进程的CPU core数量。这个参数决定了每个Executor进程并行执行task线程的能力。因为每个CPU core同一时间只能执行一个task线程，因此每个Executor进程的CPU core数量越多，越能够快速地执行完分配给自己的所有task线程。
> 2.  Executor的CPU core数量设置为2-4个较为合适。同样得根据不同部门的资源队列来定，可以看看自己的资源队列的最大CPU core限制是多少，再依据设置的Executor数量，来决定每个Executor进程可以分配到几个CPU core。同样建议，如果是跟他人共享这个队列，那么num-executors * executor-cores不要超过队列总CPU core的1/3-1/2左右比较合适，也是避免影响其他同学的作业运行。

set spark.vcore.boost.ratio=2;

### 关于内存资源说明

~~~java
spark资源参数调节说明：

spark执行的时候，可以通过 --executor-memory 来设置executor执行时所需的memory。但如果设置的过大，程序是会报错的。

两个相关的参数：

**yarn.scheduler.maximum-allocation-mb**

这个参数表示每个container能够申请到的最大内存，一般是集群统一配置。Spark中的executor进程是跑在container中，所以container的最大内存会直接影响到executor的最大可用内存。当你设置一个比较大的内存时，日志中会报错，同时会打印这个参数的值。

**spark.yarn.executor.memoryOverhead**

executor执行的时候，用的内存可能会超过executor-memoy，所以会为executor额外预留一部分内存。spark.yarn.executor.memoryOverhead代表了这部分内存。这个参数如果没有设置，会有一个自动计算公式(位于ClientArguments.scala中)，代码如下：

![1652423869855](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202205/13/143751-358249.png)

其中，MEMORY_OVERHEAD_FACTOR默认为0.1，executorMemory为设置的executor-memory, MEMORY_OVERHEAD_MIN默认为384m。参数MEMORY_OVERHEAD_FACTOR和MEMORY_OVERHEAD_MIN一般不能直接修改，是Spark代码中直接写死的。

**executor-memory计算**：

计算公式：val executorMem = args.executorMemory + executorMemoryOverhead

假设executor为X（整数，单位为M），即

1. 如果没有设置spark.yarn.executor.memoryOverhead,

executorMem= X+max(X*0.1,384)

2. 如果设置了spark.yarn.executor.memoryOverhead（整数，单位是M）

executorMem=X +spark.yarn.executor.memoryOverhead 

但是需要满足：executorMem< yarn.scheduler.maximum-allocation-mb  
~~~

### 动态executor申请

> num-executors
>
> 该参数用来设置Spark作业总共要用多少Executor进程来执行。Driver在想YARN集群管理器申请资源时，YARN集群管理器会尽可能按照你的设置来在集群的各个工作节点上，启动相应数量的Executor进程。

动态executor申请：`spark.dynamicAllocation.enabled`这个参数需要设置为true表示开启动态资源分配

> 是否开启动态资源分配，默认开启，同时强烈建议用户不要关闭。当开启动态资源分配时，Spark可以根据当前作业的负载动态申请和释放资源。

表示executors设置的初始值： `spark.dynamicAllocation.initialExecutors `

set spark.dynamicAllocation.minExecutors=10; 最少分配多少个`executors`

> 和上面相反，此参数限定了某一时刻executor的最小个数。平台默认是3，即在任何时刻，作业都会保持至少有3个及以上的executor存活，保证任务可以迅速调度。

set spark.dynamicAllocation.maxExecutors=300; 最多分配多少个`executors`

> 开启动态资源分配后，最多可以申请的executor个数。平台默认设置为1000.当在Spark UI中观察到task较多时，可以调大此参数，保证task能够并发执行完成，缩短作业执行时间。

spark.dynamicAllocation.executorIdleTimeout

> 当某个executor空闲超过这个设定值，就会被kill，默认60s

spark.dynamicAllocation.cachedExecutorIdleTimeout

> 当某个缓存数据的executor空闲时间超过这个设定值，就会被kill，默认infinity

spark.dynamicAllocation.schedulerBacklogTimeout

> 任务队列非空，资源不够，申请executor的时间间隔，默认1s

### shuffle partition并行度

spark.sql.shuffle.partitions: 在有Join或者聚合等需要shuffle的操作时，从mapper端写出的partition个数，默认设置2000。

~~~sql
select a,avg(c) from table group by a
~~~

不考虑优化行为，如果一个map端的task中包含有3000个a，根据spark.sql.shuffle.partitions=2000，会将计算结果分成2000份partition，写到磁盘，启动2000个reducer，每个reducer从每个mapper端拉取对应索引的partition。

当作业数据较多时，适当调大该值，当作业数据较少时，适当调小以节省资源。

- spark.sql.adaptive.enabled:

是否开启调整partition功能，如果开启，spark.sql.shuffle.partitions设置的partition可能会合并到一个reducer中运行。默认开启，可以更好的利用当个executor的性能，还能缓解小文件的问题。

- spark.sql.adaptive.shuffle.targetPostShuffleInputSize:

~~~sql
 set spark.sql.adaptive.shuffle.targetPostShuffleInputSize=536870912;
~~~

和spark.sql.adaptive.enabled配合使用，当开启调整partition功能后，当mapper端两个partition的数据合并后数据量小于targetPostShuffleInputSize时，spark会将两个partition进行合并到一个reducer端进行处理。平台默认是64M，用户可根据自身作业的情况调整该值。当调大该值时，一个reducer端task处理的数据量变大，最终产出的数据，存到HDFS上文件也变大；当调小该值时，相反。

- set spark.sql.adaptive.minNumPostShufflePartitions=10;

当开启调整partition功能后，有时会导致很多分区被合并，为了防止分区过少，可以设置该参数，防止分区过少影响性能。

- set spark.sql.adaptive.maxNumPostShufflePartitions=1000;

最多多少个文件，超过这个文件数量就会进行合并操作。

### 开启parquet切分

set spark.sql.parquet.adaptiveFileSplit=true;

初始task调节，合并小文件

~~~sql
set spark.sql.files.maxPartitionBytes=536870912;
//合并文件的最大值
~~~

### shuffle落地到hdfs

~~~JAVA
//设置shuffle执行过程中数据落hdfs
set spark.shuffle.hdfs.enable=true
//默认值：3
//参数说明：shuffle read任务从shuffle write任务那里节点正在拉自己的数据，如果网络由于异常拉失败而失败，它将自动重试。 此参数表示可以重试的最大次数。 如果在指定的次数内进行或不成功，则可能导致作业失败。
set spark.shuffle.io.maxRatries=1
//设置重新尝试之间的时间间隔
set spark.shuffle.io.retryWait=0s
~~~

### 自动数据倾斜处理

~~~java
//设置spark sql中的join方式为hash join
set spark.sql.adaptive.hashjoin=true
 
 //开启自动数据倾斜处理
set spark.sql.adaptive.skewedJoin.enabled=true
 
//spark.sql.adaptive.skewedPartitionFactor 默认为10 当一个partition的size大小大于 该值乘以所有parititon大小的中位数且大于spark.sql.adaptive.skewedPartitionSizeThreshold，或者parition的条数大于该值乘以所有parititon条数的中位数且 大于 spark.sql.adaptive.skewedPartitionRowCountThreshold， 才会被当做倾斜的partition进行相应的处理
set spark.sql.adaptive.skewedPartitionFactor=3
// 控制处理一个倾斜 Partition 的 Task 个数上限，默认值为 5
set spark.sql.adaptive.skewedPartitionMaxSplits=20
//设置自动进行聚合操作
set spark.sql.adaptive.skewedJoinWithAgg.enabled=true
//
set spark.sql.adaptive.multipleSkewedJoin.enabled=true
set spark.shuffle.highlyCompressedMapStatusThreshold=20000
~~~

## 小型作业

目前测试：在不手动添加任何参数、平均时长在30min以内、单个shuffle 量在500G以下的任务可以使用该模版，但实际任务情况还需跟踪观察。

### 基础资源

1. set spark.driver.memory=15g; driver内存设置
2.  set spark.driver.cores=3;driver核心数设置
3. set spark.driver.memoryOverhead=4096;driver堆外内存的大小，spark内存=jvm内存+堆外内存
4. set spark.executor.memory=5G;executor内存设置
5.  set spark.executor.memoryOverhead=1024;executor堆外内存设置
6. set spark.executor.cores=2;executor的核心数设置
7. set spark.vcore.boost.ratio=2;

### 动态executor申请：

`spark.dynamicAllocation.enabled`这个参数需要设置为true表示开启动态资源分配

1.  `spark.dynamicAllocation.initialExecutors ``表示executors设置的初始值`
2. set spark.dynamicAllocation.minExecutors=10; 最少分配多少个`executors`
3. set spark.dynamicAllocation.maxExecutors=300; 最多分配多少个`executors`

### shuffle partition并行度

1. set spark.sql.adaptive.minNumPostShufflePartitions=10;
2. set spark.sql.adaptive.maxNumPostShufflePartitions=1000;
3. set spark.sql.adaptive.shuffle.targetPostShuffleInputSize=536870912;

### 开启parquet切分

1.  set spark.sql.parquet.adaptiveFileSplit=true;

初始task调节，合并小文件

> set spark.sql.files.maxPartitionBytes=536870912;

## 中型作业

目前测试：在不手动添加任何参数、平均时长在90min以内、单个shuffle 量在2T以下的任务可以使用该模版，但实际任务情况还需跟踪观察。

### 基础资源

1. set spark.driver.memory=25g;

2. set spark.driver.cores=4;

3. set spark.driver.memoryOverhead=5120;

4. set spark.executor.memory=10G;

5. set spark.executor.memoryOverhead=4096;

6. set spark.executor.cores=3;

7. set spark.vcore.boost.ratio=1;

### 动态executor申请

1. set spark.dynamicAllocation.minExecutors=10;

2. set spark.dynamicAllocation.maxExecutors=600;

### shuffle partition并行度

1.  set spark.sql.adaptive.minNumPostShufflePartitions=10;

2.  set spark.sql.adaptive.maxNumPostShufflePartitions=1000;

3. set spark.sql.adaptive.shuffle.targetPostShuffleInputSize= 536870912;

### 开启parquet切分，初始task调节，合并小文件

1. set spark.sql.parquet.adaptiveFileSplit=true;

2. set spark.sql.files.maxPartitionBytes=536870912;

### 推测

> `spark.speculation`：是否开启spark推测任务执行，默认是false。
>
> `spark.speculation.interval`：多长时间检查一次任务执行缓慢，是否需要推测执行，100ms。
>
> `spark.speculation.multiplier`：一个任务的执行速度比中值慢多少倍，默认是1.5
>
> `spark.speculation.quantile`为特定阶段启用推测之前必须完成的部分任务。默认0.75
>
> `spark.speculation.minTaskRuntime`：在考虑推测之前，任务运行的最短时间。这可以用来避免启动非常短的任务的推测性副本。默认100ms

1. set spark.speculation.multiplier=2.5;

2. set spark.speculation.quantile=0.8;

### shuffle 落地hdfs

1. set spark.shuffle.hdfs.enabled=true;

2. set spark.shuffle.io.maxRetries=1;

3. set spark.shuffle.io.retryWait=0s;

## 大型作业

目前测试：在不手动添加任何参数、平均时长在120min以内、单个shuffle 量在10T以下的任务可以使用该模版，但实际任务情况还需跟踪观察。

\1. --基础资源

\2. set spark.driver.memory=25g;

\3. set spark.driver.cores=4;

\4. set spark.driver.memoryOverhead=5120;

\5. set spark.executor.memory=15G;

\6. set spark.executor.memoryOverhead=3072;

\7. set spark.executor.cores=3;

\8. set spark.vcore.boost.ratio=1;

\9. --动态executor申请

\10. set spark.dynamicAllocation.minExecutors=10;

\11. set spark.dynamicAllocation.maxExecutors=900;

\12. --ae

\13. set spark.sql.adaptive.minNumPostShufflePartitions=10;

\14. set spark.sql.adaptive.maxNumPostShufflePartitions=3000;

\15. set spark.sql.adaptive.shuffle.targetPostShuffleInputSize= 536870912;

\16. --shuffle 落地hdfs

\17. set spark.shuffle.hdfs.enabled=true;

\18. set spark.shuffle.io.maxRetries=1;

\19. set spark.shuffle.io.retryWait=0s;

\20. --开启parquet切分，合并小文件

set spark.sql.parquet.adaptiveFileSplit=true;

\22. set spark.sql.files.maxPartitionBytes=536870912;

\23. --推测

\24. set spark.speculation.multiplier=2.5;

\25. set spark.speculation.quantile=0.9;

## 超大型作业

目前测试：在不手动添加任何参数、平均时长大于120min、单个shuffle 量在10T以上的任务可以使用该模版，但实际任务情况还需跟踪观察。

\1. --基础资源

\2. set spark.driver.memory=30g;

\3. set spark.driver.cores=4;

\4. set spark.driver.memoryOverhead=5120;

\5. set spark.executor.memory=20G;

\6. set spark.executor.memoryOverhead= 5120;

\7. set spark.executor.cores=5;

\8. set spark.vcore.boost.ratio=1;

\9. --动态executor申请

\10. set spark.dynamicAllocation.minExecutors=10;

\11. set spark.dynamicAllocation.maxExecutors=1500;

\12. --ae

\13. set spark.sql.adaptive.minNumPostShufflePartitions=10;

\14. set spark.sql.adaptive.maxNumPostShufflePartitions=7000;

\15. set spark.sql.adaptive.shuffle.targetPostShuffleInputSize= 536870912;

\16. --开启parquet切分,合并小文件

\17. set spark.sql.parquet.adaptiveFileSplit=true;

\18. set spark.sql.files.maxPartitionBytes=536870912;

\19. -- shuffle 落地 hdfs

\20. set spark.shuffle.hdfs.enabled=true;

\21. set spark.shuffle.io.maxRetries=1;

\22. set spark.shuffle.io.retryWait=0s;

\23. --推测

\24. set spark.speculation.multiplier=2.5;

\25. set spark.speculation.quantile=0.9;

## 备用参数

### 预留

**任务的优先级**

> set spark.yarn.priority = 9

 **hash join**

> set spark.sql.adaptive.hashJoin.enabled=true;
>
> set spark.sql.adaptiveHashJoinThreshold=52428800;- 设置hash join吃的阈值。

**输出文件合并**

>  set spark.merge.files.byBytes.enabled=true;
>
> set spark.merge.files.byBytes.repartitionNumber=100;
>
> set spark.merge.files.byBytes.fileBytes=134217728;
>
> set spark.merge.files.byBytes.compressionRatio=3;

**skew_join 解析绕过tqs**

> set tqs.analysis.skip.hint=true;

**初始task上限**

> set spark.sql.files.openCostInBytes=4194304;
>
> set spark.datasource.splits.max=20000;

**broadcast时间**

> broadcastTimeout 失败时间
>
> set spark.sql.broadcastTimeout = 3600;

**防止get json报错**

> set spark.sql.mergeGetMapValue.enabled=true;

**数据倾斜**：OptimizeSkewedJoin

> set spark.sql.adaptive.allowBroadcastExchange.enabled=true;

需要配合 'spark.sql.adaptive.enabled'使用，那么在shuffle过程中会动态处理join方式。

> When true and 'spark.sql.adaptive.enabled' is true, Spark dynamically 
> handles skew in shuffled join (sort-merge and shuffled hash) by 
> splitting (and replicating if needed) skewed partitions.
>
> //设置spark sql中的join方式为hash join
>
> set spark.sql.adaptive.hashJoin.enabled=false;
>
>  //开启自动数据倾斜处理
>
> set spark.sql.adaptive.skewedJoin.enabled=true;
>
> set`spark.sql.adaptive.skewJoin.skewedPartitionFactor`=3;//spark.sql.adaptive.skewedPartitionFactor 默认为5，当一个partition的size大小大于 该值乘以所有parititon大小的中位数且大于spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes，那么被认为发生数据倾斜。或者parition的条数大于该值乘以所有parititon条数的中位数且 大于 spark.sql.adaptive.skewedPartitionRowCountThreshold， 才会被当做倾斜的partition进行相应的处理
>
> `spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes`：默认是256M，一个分区被认为是数据倾斜，如果分区是以字节为单位，并且大于大于这个值，并且也大于'spark.sql.adaptive.skewJoin.skewedPartitionFactor' 这个值乘分区大小的中位数，那么就是数据倾斜的分区，
>
> // 控制处理一个倾斜 Partition 的 Task 个数上限，默认值为 5
> set spark.sql.adaptive.skewedPartitionMaxSplits=20
>
> //设置自动进行聚合操作
> set spark.sql.adaptive.skewedJoinWithAgg.enabled=true
>
> set spark.sql.adaptive.multipleSkewedJoin.enabled=true
> set spark.shuffle.highlyCompressedMapStatusThreshold=20000
>
> //设置广播表的大下，当小于这个值的时候，会广播
>
> `spark.sql.adaptive.autoBroadcastJoinThreshold`，默认是10m

**并发读文件**

\32. set spark.sql.concurrentFileScan.enabled=true;

\33. --filter按比例读取文件

\34. set spark.sql.files.tableSizeFactor={table_name}:{filter 比例};

\35. set spark.sql.files.tableSizeFactor=dm_content.tcs_task_dict:10;

\36. --AM failed 时长

\37. set spark.yarn.am.waitTime=200s;

\38. --shuffle service 超时设置

\39. set spark.shuffle.registration.timeout=12000;

\40. set spark.shuffle.registration.maxAttempts=5;

\41. --parquet index 特性生效，in 条件的个数

\42. set spark.sql.parquet.pushdown.inFilterThreshold=30; 

\43.  

\44. --设置engine

\45. set tqs.query.engine.type=sparkcli;

\46.  

\47. --hive metastore 超时

\48. spark.hadoop.hive.metastore.client.socket.timeout=600

\49.  

\50. --manta备用

\51. spark.sql.adaptive.maxNumPostShufflePartitions 5000

\52. spark.executor.memoryOverhead 8000

\53. spark.sql.adaptive.shuffle.targetPostShuffleInputSize 536870912


##  dorado默认参数

贴这个是为了防止冗余写。

\1. //基础资源

\2. spark.driver.memory        12G

\3. spark.driver.memoryOverhead        6144

\4. spark.executor.memory        8G

\5. spark.executor.memoryOverhead        6144

\6. spark.executor.cores        4

\7. //动态资源

\8. spark.dynamicAllocation.enabled        true

\9. spark.dynamicAllocation.executorIdleTimeout        120s

\10. spark.dynamicAllocation.initialExecutors        5

\11. spark.dynamicAllocation.maxExecutors        900

\12. spark.dynamicAllocation.minExecutors        5

\13.  

\14. //jvm相关

\15. spark.executor.extraJavaOptions        -XX:+UseG1GC -XX:G1HeapRegionSize=4m -XX:-UseGCOverheadLimit -verbose:gc -XX:+PrintAdaptiveSizePolicy -XX:+UnlockDiagnosticVMOptions -XX:+G1SummarizeConcMark -Xss4m -XX:MaxDirectMemorySize=4096m

\16.  

\17. //shuffle相关，spark自带

\18. spark.shuffle.consolidateFiles        true

\19. spark.shuffle.hdfs.rootDir        hdfs://haruna/spark_topi/shuffle/hdfs_dancenn

\20. spark.shuffle.highlyCompressedMapStatusThreshold        2000

\21. spark.shuffle.service.enabled        true

\22. spark.shuffle.statistics.verbose        true

\23. spark.speculation        true

\24. //ae相关

\25. spark.sql.adaptive.enabled        true

\26. spark.sql.adaptive.join.enabled        true

\27. spark.sql.adaptive.maxNumPostShufflePartitions        500

\28. spark.sql.adaptive.minNumPostShufflePartitions        1

\29. spark.sql.adaptive.shuffle.targetPostShuffleInputSize        67108864 //64MB

\30. spark.sql.adaptive.shuffle.targetPostShuffleRowCount        20000000

\31. spark.sql.adaptive.skewedJoin.enabled        false

\32. spark.sql.adaptive.skewedPartitionFactor        10

\33. spark.sql.adaptive.skewedPartitionRowCountThreshold        10000000

\34. spark.sql.adaptive.skewedPartitionSizeThreshold        67108864

\35. spark.sql.adaptiveBroadcastJoinThreshold        20971520

\36. spark.sql.autoBroadcastJoinThreshold        20971520

\37. //初始文件合并

\38. spark.sql.files.maxPartitionBytes        1073741824//1GB

\39. spark.sql.files.openCostInBytes        16777216

\40. //其他

\41. spark.sql.hive.caseSensitiveInferenceMode        NEVER_INFER

\42. spark.sql.hive.convertMetastoreOrc        true

\43. spark.sql.hive.convertMetastoreParquet        true

\44. spark.sql.parquet.adaptiveFileSplit        false

\45. spark.sql.parquet.compression.codec        gzip

\46. spark.sql.parquet.filterPushdown        true

\47. spark.sql.parquet.pushdown.inFilterThreshold        20

\48. spark.sql.skip.adjust.partitioned.file        true

\49. spark.sql.sources.bucketing.enabled        false

\50. //spark容错以及yarn优先级

\51. spark.task.maxFailures        8

\52. park.yarn.maxAppAttempts        1

\53. spark.yarn.priority        1

##  风神默认

\1. spark.datasource.splits.max        50000

\2.  

\3. spark.driver.memory        12G

\4. spark.driver.memoryOverhead        6144

\5. spark.executor.memory        5g

\6. spark.executor.memoryOverhead        3072

\7.  

\8. spark.dynamicAllocation.enabled        true

\9. spark.dynamicAllocation.executorIdleTimeout        30s

\10. spark.dynamicAllocation.initialExecutors        5

\11. spark.dynamicAllocation.maxExecutors        300

\12. spark.dynamicAllocation.minExecutors        5

\13.  

\14. spark.executor.extraJavaOptions        -XX:+UseG1GC -XX:G1HeapRegionSize=4m -XX:-UseGCOverheadLimit -verbose:gc -XX:+PrintAdaptiveSizePolicy -XX:+UnlockDiagnosticVMOptions -XX:+G1SummarizeConcMark -Xss4m -XX:MaxDirectMemorySize=2560m -XX:+ExitOnOutOfMemoryError

\15.  

\16. spark.memory.fraction        0.7

\17. spark.network.timeout        60s

\18. spark.olap.collect.app.enable        true

\19. spark.olap.metrics.httpserver        data.olap.spark_metrics

\20. spark.olap.metrics.running.info        true

\21.  

\22. spark.shuffle.highlyCompressedMapStatusThreshold        2000

\23. spark.shuffle.service.enabled        true

\24. spark.shuffle.statistics.verbose        true

\25. spark.speculation        true

\26.  

\27. spark.sql.adaptive.enabled        true

\28. spark.sql.adaptive.hashJoin.enabled        true

\29. spark.sql.adaptive.join.enabled        true

\30. spark.sql.adaptive.maxNumPostShufflePartitions        2000

\31. spark.sql.adaptive.minNumPostShufflePartitions        1

\32. spark.sql.adaptive.shuffle.targetPostShuffleInputSize        52428800 //50MB

\33. spark.sql.adaptive.shuffle.targetPostShuffleRowCount        5000000

\34. spark.sql.adaptive.skewedJoin.enabled        true

\35. spark.sql.adaptive.skewedPartitionFactor        3

\36. spark.sql.adaptive.skewedPartitionMaxSplits        6

\37. spark.sql.adaptive.skewedPartitionRowCountThreshold        5000000

\38. spark.sql.adaptive.skewedPartitionSizeThreshold        52428800

\39. spark.sql.adaptiveBroadcastJoinThreshold        20971520

\40. spark.sql.adaptiveHashJoinThreshold        52428800

\41. spark.sql.autoBroadcastJoinThreshold        20971520

\42. spark.sql.broadcastTimeout        3000

\43.  

\44. spark.sql.parquet.adaptiveFileSplit        true

\45. spark.sql.files.maxPartitionBytes        268435456

\46. spark.sql.files.openCostInBytes        4194304

\47.  

\48. spark.sql.hive.caseSensitiveInferenceMode        NEVER_INFER

\49. spark.sql.hive.convertMetastoreOrc        true

\50. spark.sql.hive.convertMetastoreParquet        true

\51. spark.sql.optimizer.metadataOnly        true

\52. spark.sql.or.filter.optimize        false

\53. spark.sql.orc.adaptiveFileSplit        true

\54. spark.sql.execution.engine        sparkCli

\55. spark.sql.parquet.compression.codec        gzip

\56. spark.sql.parquet.filterPushdown        true

\57. spark.sql.partitionFileFormatConsistentCheck.enabled        true

\58. spark.sql.resolveAmbiguous        true

\59. spark.sql.resolveAmbiguousInGroupBy.enabled        false

\60. spark.sql.skip.adjust.partitioned.file        true

\61. spark.sql.sources.bucketing.enabled        false

\62.  

\63. spark.vcore.boost        2

\64. spark.vcore.boost.ratio        2