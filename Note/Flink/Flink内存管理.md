<!-- TOC -->

- [Flink内存管理](#flink内存管理)
  - [JVM自动内存管理优缺点](#jvm自动内存管理优缺点)
  - [自主内存管理](#自主内存管理)
    - [JVM内存管理的不足](#jvm内存管理的不足)
    - [自主内存管理](#自主内存管理-1)
    - [堆外内存 VS 堆内内存](#堆外内存-vs-堆内内存)
  - [内存模型](#内存模型)
    - [内存布局](#内存布局)
    - [内存计算](#内存计算)
  - [内存数据结构](#内存数据结构)
    - [**内存段**](#内存段)
    - [**内存页**](#内存页)
    - [**Buffer**](#buffer)
    - [**Buffer资源池**](#buffer资源池)
  - [**内存管理器**](#内存管理器)
    - [**批处理中**](#批处理中)
    - [**在流计算中**](#在流计算中)
  - [**网络缓冲器**](#网络缓冲器)
  - [**总结**](#总结)

<!-- /TOC -->


## Flink内存管理

### JVM自动内存管理优缺点

优点：JVM降低了程序员对内存管理的门槛，JVM可以对代码进行深度优化

缺点：但同时也使得程序员把管理内存的主动权交给了JVM

### 自主内存管理

**Flink选择自主内存管理**，即回收部分JVM进程内存管理的主动权，原因：JVM内存管理在大数据场景下有诸多问题。

#### JVM内存管理的不足

**有效数据密度低**

Java对象在内存中的存储主要包含：对象头、实例数据、对齐填充部分。

![1637582576160](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/22/200257-289206.png)

例如，boolean占1byte，但是在内存中数据存储不是连续的，而是按照8byte的整数倍进行存储的，就会进行填充，造成数据密度低。

**垃圾回收**

在大数据场景下，内存不可控，如果出现需要消耗大量内存进行计算的场景，当产生海量Java对象，一旦出现Full GC，GC耗时可能甚至达到分钟级，直接影响集群的心跳等。

**OOM问题影响稳定性**

例如，OutOfMemoryException在分布式框架中经常会遇到，分布式框架的健壮性和稳定性都会收到影响。

缓存未命中问题

CPU的L1、L2、L3的多级缓存理论基础来自于程序局部性原理：

- 空间局部性：被CPU访问的数据，CPU短期内还要访问

- 时间局部性：被CPU访问的数据附近的数据，CPU短期内还要访问

但是，Java堆对象的存储并不连续。CPU空转是计算机性能之殇。

#### 自主内存管理

在Flink中Java对象的有效信息被序列化，在内存中连续存储，保存在预分配的内存块上，内存块叫作**MemorySegment**，即内存分配的最小单元。很多运算可以直接操作序列化的二进制数据，而不需要反序列化。

MemorySegment可以在堆上：Java byte数组；也可以在堆外：ByteBuffer。

#### 堆外内存 VS 堆内内存

**堆外内存的优势：**

- 避免GC和内存溢出

- 高效的IO操作。堆外内存写磁盘IO或网络IO是zero-copy(零拷贝)

- 堆外内存是进程间共享的。JVM进程崩溃不会丢失数据，可以用来故障恢复

**堆外内存的劣势：**

- 堆上内存的使用、监控、调试简单。

- 短生命周期的MemorySegment的分配，堆内内存开销更小。

- Flink的部分操作在堆外内存比堆内内存慢。

> Flink在计算中采用了DBMS的Sort和Join算法，直接操作二进制数据，避免反复序列化。

### 内存模型

####  内存布局

TaskManager是Flink中执行计算的核心组件，使用了大量堆外内存。TM的简化和详细内存结构如下，

![1637661982188](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/23/180622-273484.png)

**详细内存划分**

![1637662025531](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/23/180712-35979.png)

**组成关系说明**

![1637662066891](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/23/180748-594417.png)

**内存参数及配置**

![1637662161057](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/23/180921-755196.png)

**总内存配置**

![1637662222267](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/23/181022-3286.png)

JVM内存参数控制如下：

1. JVM堆上内存，使用-Xmx和-Xms参数进行控制
2. JVM直接内存，使用-XX：MaxDirectMemorySize进行控制。对于托管内存，使用Unsafe.allocateMemory()申请，不受该参数控制。
3. JVM MetaSpace使用-XX：MaxMetaspaceSize进行控制。

#### 内存计算

在容器环境下，内存计算是在ResourceManager中进行的。计算好的参数使用-D 参数交给Java进程。（JobManager 是 Flink 集群的控制单元。它由三种不同的组件组成：**ResourceManager、Dispatcher和每个正在运行作业的JobMaster**）

- Flink会根据默认值或其他配置参数自动调整剩余内存部分的大小。 

- 通过配置进程总内存可以指定由Flink JVM 进程使用的总内存大小。对于容器化部署模式（Containerized Deployment），这相当于申请的容器（Container）大小。

- 还可以通过设置 Flink总内存的特定内部组成部分的方式来进行内存配置。 

以上三种方式中，用户需要至少选择其中一种进行配置（本地运行除外），否则Flink将无法启动。

1. 不建议同时设置进程总内存和 Flink总内存。这可能会造成内存配置冲突，从而导致部署失败。

2. 通常情况下，不建议对框架堆内存和框架堆外内存进行调整。

3. 如果只配置了进程总内存，则从进程总内存扣除JVM元空间和JVM执行开销，剩余内存作为Flink总内存

4. 如果已经明确设置了任务堆内存和托管内存，建议不要再设置进程总内存或 Flink总内存，否则可能会造成内存配置冲突。

5. 如果手动设置了托管内存，则使用其值，否则使用默认分配系数*Flink总内存

6. 如果手动设置了网络缓冲内存，则使用其值，否则使用默认分配系数*Flink总内存

7. 如果配置了Flink总内存，而没有配置Task堆上内存和托管内存，则从Flink总内存中划分网络缓冲内存和托管内存，剩下的作为Task内存。

它们都可以通过指定在总内存中所占比例的方式进行配置，同时受限于相应的的最大/最小值范围。

- JVM开销：可以配置占用进程总内存的固定比例
- 网络内存：可以配置占用Flink总内存的固定比例（仅针对TaskManager）

这些内存部分的大小必须在相应的最大值、最小值范围内，否则Flink将无法启动。最大值、最小值具有默认值，也可以通过相应的配置参数进行设置。

如果没有明确指定内存部分的大小，Flink会根据总内存和占比计算出该内存部分的大小。计算得到的内存大小将受限于相应的最大值、最小值范围。

如果配置了总内存和其他内存部分的大小，那么Flink也有可能会忽略给定的占比。这种情况下，受限的等比内存部分的实际大小是总内存减去其他所有内存部分后剩余的部分。这样推导得出的内存大小必须符合最大值、最小值范围，否则 Flink将无法启动。 
例1，

进程总内存 = 1000Mb

JVM 开销最小值 = 64Mb

JVM 开销最大值 = 128Mb

JVM 开销占比 = 0.1

那么 JVM 开销的实际大小将会是 1000Mb x 0.1 = 100Mb，在 64-128Mb的范围内。

例2，

进程总内存 = 1000Mb

JVM 开销最小值 = 128Mb

JVM 开销最大值 = 256Mb

JVM 开销占比 = 0.1

那么 JVM 开销的实际大小将会是 128Mb，因为根据总内存和占比计算得到的内存大小 100Mb小于最小值。

例3，

进程总内存 = 1000Mb

任务堆内存 = 100Mb

JVM 开销最小值 = 64Mb

JVM 开销最大值 = 256Mb

JVM 开销占比 = 0.1

那么，JVM 开销的实际大小将会是1000 – 100 = 900Mb，如果配置了总内存和其他内存部分的大小，那么 Flink也有可能会忽略给定的占比。

### 内存数据结构

Flink的内存管理像OS管理内存，划分为段和页。

#### **内存段**

内存段，即**MemorySegment**，是Flink内存抽象的最小分配单元。其实就是一个内存块，默认32KB。

MemorySegment可以在堆上：Java byte数组；也可以在堆外：基于Netty的ByteBuffer。

对于Java基本类型，MemorySegment可以直接读写二进制数据，对于其他类型，读取byte[]后反序列化，修改后序列化到MemorySegment。

![1637662619406](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/23/181700-175179.png)

HeapMemorySegment分配堆上内存，HybridMemorySegment分配堆外内存，实际上后来Flink用HybridMemorySegment分配堆外堆内内存。这设计JIT的编译优化。如果同时使用两个类，运行时每一次都要去查询函数表，确定调用哪个子类中的方法，无法提前优化。如果只是用一个实现子类，自动识别方法的调用都可以被虚化和内联，性能差在2.7倍左右。HybridMemorySegment使用Unsafe提供的一系列方法同时操作堆上和堆外内存。

#### **内存页**

MemorySegment的抽象粒度面向的是OS的内存管理，这种抽象对于上层的读写显然过于繁琐，Flink又抽象了一层，即内存页。内存页是MemorySegment之上的数据访问视图，数据读取抽象为DataInputView，数据写抽象为DataOutputView。

对于内存的读写是非常底层的行为，对于上层应用（DataStream作业）而言，涉及向MemorySegment写入、读取二进制的地方都使用到了DataInputView、DataOutputView，而不是直接使用MemorySegment。

#### **Buffer**

Task算子处理完数据后，将结果交给下游的时候，使用的抽象或者说内存对象是Buffer。其实现类是NetworkBuffer。一个NetworkBuffer包装了一个MemorySegment。

![1637662759741](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/23/181921-449970.png)

NetworkBuffer底层是MemorySegment。Buffer的申请释放由Flink自行管理，Flink引入了引用计数的概念，当有新的Buffer消费者，引用加一，当消费完，引用减1，最终当引用数变为0，就可以将Buffer释放了。

AbstractReferenceCountedByteBuf是Netty中的抽象类。通过实现该类，Flink拥有了引用计数控制Netty申请到的Buffer的内存的能力。

#### **Buffer资源池**

Buffer资源池：BufferPool，用来管理Buffer，包含Buffer的申请、释放、销毁、可用Buffer的通知。实现类是LocalBufferPool，每个Task拥有自己的LocalBufferPool。

为了方便对BufferPool的管理，设计了BufferPoolFactory，唯一实现类是NetworkBufferPool。每个TaskManager只有一个NetworkBufferPool。同一个TaskManager共享NetworkBufferPool。

### **内存管理器**

MemoryManager是Flink管理托管内存的组件，只使用堆外内存。在批处理中用在排序、Hash表和中间结果缓存，在流计算中作为RocksDBStateBackend的内存。Flink 1.10以后MemoryManager的管理范围缩小到Task的Slot级别。

MemoryManager通过MemoryPool管理所有MemorySegemnt，不需要Network Buffer那一层抽象。

#### **批处理中**

在批处理中，MemoryManager使用Unsafe申请堆外内存，包装为ByteBuffer后再包装为MemorySegment。

#### **在流计算中**

MemoryManager控制RocksDB的内存使用量，从TM的内存配置中计算而来。RocksDB自己来负责运行过程中的内存的申请和释放。

**释放**

- 内存使用完毕
- Task停止（正常或异常）执行

### **网络缓冲器**

当结果分区（ResultPartition）开始写出数据的时候，需要向LocalBufferPool申请Buffer资源，使用BufferBuilder写入MemorySegment。BufferBuilder在上游Task中，用来向申请到的MemorySegment写入数据。与BufferBuilder相对的是BufferConsumer，BufferConsumer位于下游Task中，负责从MemorySegment中读取数据。一个BufferBuilder对应一个BufferConsumer。

### **总结**

大数据场景下，使用Java的内存管理会带来一系列问题，Flink从一开始就选择自主管理内存。

Flink对内存进行了一系列抽象，内存段MemorySegment是最小的内存分配单位，对应内存块。对于跨内存段的访问，Flink抽象了DataInputView和DataOutputView，可以理解为内存页。

在计算层面，Flink内存管理器提供内存的释放和申请。在数据传输层面，Flink抽象了网络内存缓存Buffer。