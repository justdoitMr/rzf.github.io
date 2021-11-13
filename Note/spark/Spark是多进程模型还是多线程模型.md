## Spark是多进程模型还是多线程模型

Apache  Spark的高性能一定程度上取决于它采用的异步并发模型（这里指server/driver端采用的模型），这与Hadoop  2.0（包括YARN和MapReduce）是一致的。Hadoop  2.0自己实现了类似Actor的异步并发模型，实现方式是epoll+状态机，而Apache  Spark则直接采用了开源软件Akka，该软件实现了Actor模型，性能非常高。尽管二者在server端采用了一致的并发模型，但在任务级别（特指Spark任务和MapReduce任务）上却采用了不同的并行机制：Hadoop  MapReduce采用了多进程模型，而Spark采用了多线程模型。

 注意，本文的多进程和多线程，指的是同一个节点上多个任务的运行模式。无论是MapReduce和Spark，整体上看，都是多进程：MapReduce应用程序是由多个独立的Task进程组成的；Spark应用程序的运行环境是由多个独立的Executor进程构建的临时资源池构成的。

 多进程模型便于细粒度控制每个任务占用的资源，但会消耗较多的启动时间，不适合运行低延迟类型的作业，这是MapReduce广为诟病的原因之一。而多线程模型则相反，该模型使得Spark很适合运行低延迟类型的作业。总之，Spark同节点上的任务以多线程的方式运行在一个JVM进程中，可带来以下好处：

 1）任务启动速度快，与之相反的是MapReduce Task进程的慢启动速度，通常需要1s左右；

 2）同节点上所有任务运行在一个进程中，有利于共享内存。这非常适合内存密集型任务，尤其对于那些需要加载大量词典的应用程序，可大大节省内存。

 3）同节点上所有任务可运行在一个JVM进程(Executor)中，且Executor所占资源可连续被多批任务使用，不会在运行部分任务后释放掉，这避免了每个任务重复申请资源带来的时间开销，对于任务数目非常多的应用，可大大降低运行时间。与之对比的是MapReduce中的Task：每个Task单独申请资源，用完后马上释放，不能被其他任务重用，尽管1.0支持JVM重用在一定程度上弥补了该问题，但2.0尚未支持该功能。

 尽管Spark的过线程模型带来了很多好处，但同样存在不足，主要有：

 1）由于同节点上所有任务运行在一个进程中，因此，会出现严重的资源争用，难以细粒度控制每个任务占用资源。与之相反的是MapReduce，它允许用户单独为Map  Task和Reduce Task设置不同的资源，进而细粒度控制任务占用资源量，有利于大作业的正常平稳运行。

 下面简要介绍MapReduce的多进程模型和Spark的多线程模型。

### MapReduce多进程模型

1. 每个Task运行在一个独立的JVM进程中；
2. 可单独为不同类型的Task设置不同的资源量，目前支持内存和CPU两种资源；
3. 每个Task运行完后，将释放所占用的资源，这些资源不能被其他Task复用，即使是同一个作业相同类型的Task。也就是说，每个Task都要经历“申请资源—> 运行Task –> 释放资源”的过程。

### Spark多线程模型

1. 每个节点上可以运行一个或多个Executor服务，Executor是一个进程；
2. 每个Executor配有一定数量的slot，表示该Executor中可以同时运行多少个ShuffleMapTask或者ReduceTask；
3. 每个Executor单独运行在一个JVM进程中，每个Task则是运行在Executor中的一个线程；
4. 同一个Executor内部的Task可共享内存，比如通过函数SparkContext#broadcast广播的文件或者数据结构只会在每个Executor中加载一次，而不会像MapReduce那样，每个Task加载一次；
5. Executor一旦启动后，将一直运行，且它的资源可以一直被Task复用，直到Spark程序运行完成后才释放退出。

总体上看，Spark采用的是经典的scheduler/workers模式，每个Spark应用程序运行的第一步是构建一个可重用的资源池，然后在这个资源池里运行所有的ShuffleMapTask和ReduceTask（注意，尽管Spark编程方式十分灵活，不再局限于编写Mapper和Reducer，但是在Spark引擎内部只用两类Task便可表示出一个复杂的应用程序，即ShuffleMapTask和ReduceTask），而MapReduce应用程序则不同，它不会构建一个可重用的资源池，而是让每个Task动态申请资源，且运行完后马上释放资源。