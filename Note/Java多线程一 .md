# 多线程

[TOC]

## 程序，进程，线程的理解

### **进程**

- 程序由指令和数据组成，但这些指令要运行，数据要读写，就必须将指令加载至 CPU，数据加载至内存。在指令运行过程中还需要用到磁盘、网络等设备。进程就是用来加载指令、管理内存、管理 `IO` 的
- 当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程。
- 进程就可以视为程序的一个实例。大部分程序可以同时运行多个实例进程（例如记事本、画图、浏览器等），也有的程序只能启动一个实例进程（例如网易云音乐、360 安全卫士等）
- 程序的一次执行过程，或是正在运行的一个程序。 说明：**进程作为资源分配的单位，系统在运行时会为每个进程分配不同的内存区域**，**进程是一个动态的。**
- 一个`java`程序就是一个进程。

### **线程**

- 一个进程之内可以分为一到多个线程。
- 一个线程就是一个指令流，可以把线程看作是一组指令的集合。将指令流中的一条条指令以一定的顺序交给 `CPU` 执行
- `Java` 中，线程作为最小调度单位，**进程作为资源分配的最小单位**。 在` windows `中进程是不活动的，只是作
  为线程的容器，而线程是作为**调度的基本单位**，负责运行程序。
- 进程可进一步细化为线程，是一个程序内部的一条执行路径。 说明：**线程作为CPU调度和执行的单位，每个线程拥独立的运行栈和程序计数器(`pc`)，线程切换的开销小。**
- **每一个线程都拥有计数器，`java`虚拟机栈，本地方法栈。**

### **程序**

程序(`programm`) 概念：是为完成特定任务、用某种语言编写的一组指令的集合。即指一段静态的代码,程序是静态的，而进程是动态的。

**图示理解**

![1608032529430](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/15/194211-295050.png)

- 从`jvm`角度考虑

![1608032587778](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/15/194308-722759.png)

==进程可以细化为多个线程。每个线程，拥有自己独立的：栈、程序计数器，多个线程，共享同一个进程中的结构：方法区、堆。==

### **进程和程序的对比**

- 进程基本上相互独立的，而线程存在于进程内，是进程的一个子集。
- 进程拥有共享的资源，如内存空间等，供其内部的线程共享
- 进程间通信较为复杂
  - 同一台计算机的进程通信称为 `IPC（Inter-process communication）`
  - 不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，例如` HTTP`
- 线程通信相对简单，因为它们共享进程内的内存，一个例子是多个线程可以访问同一个共享变量
- 线程更轻量，线程上下文切换成本一般上要比进程上下文切换低

## 并行与并发

### 单`cpu`与多`cpu`的理解

- 单核`CPU`，其实是一种假的多线程，因为在一个时间单元内，也只能执行一个线程的任务。例如：虽然有多车道，但是收费站只有一个工作人员在收费，只有收了费才能通过，那么`CPU`就好比收费人员。如果某个人不想交钱，那么收费人员可以把他“挂起”,但是因为`CPU`时间单元特别短，因此感觉不出来。
- 如果是多核的话，才能更好的发挥多线程的效率。（现在的服务器都是多核的）
- 一个`Java`应用程序`java.exe`，其实至少三个线程：**`main()`主线程，`gc()`垃圾回收线程，异常处理线程。当然如果发生异常，会影响主线程。**

### 并发

单核` cpu` 下，线程实际还是 串行执行 的。操作系统中有一个组件叫做任务调度器，将` cpu `的时间片（`windows`
下时间片最小约为 15 毫秒）分给不同的程序使用，只是由于` cpu `在线程间（时间片很短）的切换非常快，人类感觉是 同时运行的 。总结为一句话就是： **微观串行，宏观并行 ，**

一般会将这种 线程轮流使用` CPU` 的做法称为并发，` concurrent`。

| cpu  | 时间片1 | 时间片2 | 时间片3 | 时间片4 |
| ---- | ------- | ------- | ------- | ------- |
| 核   | 线程1   | 线程2   | 线程3   | 线程4   |

**图解**

![1608038646230](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/15/212407-899688.png)

### 并行

多核` cpu`下(也就是一个`cpu`中有多个核，核心数越多，可以提高程序的吞吐量)，每个 核（`core`） 都可以调度运行线程，这时候线程可以是并行的。

| cpu  | 时间片1 | 时间片2 | 时间片3 | 时间片4 |
| ---- | ------- | ------- | ------- | ------- |
| 核1  | 线程1   | 线程1   | 线程3   | 线程3   |
| 核2  | 线程2   | 线程4   | 线程2   | 线程4   |

**图解**

![1608038923948](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/15/212845-8130.png)

### 小结

- 并发（`concurrent`）是同一时间应对（`dealing with`）多件事情的能力
- 并行（`parallel`）是同一时间动手做（`doing`）多件事情的能力

## 同步和异步

**以调用的角度来讲**

- 需要等待结果返回，才能继续运行就是同步
- 不需要等待结果返回，就能继续运行就是异步

**设计**

- 多线程可以让方法执行变为异步的（即不要巴巴干等着）比如说读取磁盘文件时，假设读取操作花费了 5 秒钟，如果没有线程调度机制，这 5 秒`cpu` 什么都做不了，其它代码都得暂停...

**案例**

- 充分利用多核 `cpu` 的优势，提高运行效率。想象下面的场景，执行 3 个计算，最后将计算结果汇总。

~~~ java
计算 1 花费 10 ms
计算 2 花费 11 ms
计算 3 花费 9 ms
汇总需要 1 ms
~~~

- 如果是串行执行，那么总共花费的时间是 10 + 11 + 9 + 1 = `31ms`

- 但如果是四核 `cpu`，各个核心分别使用线程 1 执行计算 1，线程 2 执行计算 2，线程 3 执行计算 3，那么 3 个线程是并行的，花费时间只取决于最长的那个线程运行的时间，即 `11ms `最后加上汇总时间只会花费 `12ms`

> 注意
> 需要在多核 `cpu` 才能提高效率，单核仍然时是轮流执行

**结论**

- 单核` cpu `下，多线程不能实际提高程序运行效率，**只是为了能够在不同的任务之间切换，不同线程轮流使用`cpu `，不至于一个线程总占用`cpu`，别的线程没法干活**
- 多核 `cpu `可以并行跑多个线程，但能否提高程序运行效率还是要分情况的
  - 有些任务，经过精心设计，将任务拆分，并行执行，当然可以提高程序的运行效率。但不是所有计算任务都能拆分（参考后文的【阿姆达尔定律】）
  - 也不是所有任务都需要拆分，任务的目的如果不同，谈拆分和效率没啥意义
- `IO `操作不占用` cpu`，只是我们一般拷贝文件使用的是【阻塞` IO`】，这时相当于线程虽然不用`cpu`，但需要一直等待` IO `结束，没能充分利用线程。所以才有后面的【非阻塞 `IO`】和【异步 `IO`】优化

## 思维导图总结

![1610274193742](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/10/182315-326361.png)

## Java线程

### 创建线程的方法

#### 方式一：继承于`Thread`类

1. 创建一个继承于`Thread`类的子类
2. 重写`Thread`类的`run()` --> 将此线程执行的操作声明在`run()`中
3. 创建`Thread`类的子类的对象
4. 通过此对象调用`start()`

> 主方法也对应一个`main()`线程，这是一个`java`程序启动默认的线程

- 代码测试

~~~ java
public class TestVolatile {
    public static void main(String[] args) {
//        4 创建对象
        MyThread myThread=new MyThread();
//        5 启动线程
        myThread.start();
        //不可以对同一个线程实例启动两次
        // myThread.start(); 错误

//        /主线程打印奇数
        for(int j=0;j<1000;j++){
            if(j %2 != 0){
                System.out.println(Thread.currentThread().getName()+":"+j);
            }
        }
    }
}

//1 创建类继承thread类
class MyThread extends Thread{
//    2 重写run()方法
    @Override
    public void run() {
        super.run();
//        3 具体的操作
        for(int i=0;i<1000;i++){
            if(i%2==0)
            System.out.println(Thread.currentThread().getName()+":"+i);
        }
    }
}
//源码解读
public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);//在这里把当前线程添加进去等待调度执行

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
~~~

Thread继承与Runnable接口。

![1612182587865](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/202949-652462.png)

- 测试二,使用匿名内部类的方法

~~~ java
public class Test01 {
    public static void main(String[] args) {
//        使用匿名内部类创建一个线程
        Thread thread=new Thread(){
//            此线程的执行和main线程的执行是不一样的
//            run()内写逻辑代码
            @Override
            public void run() {
                super.run();
                System.out.println(Thread.currentThread().getName()+"  running!");
            }
        };
//        最好给线程起一个名字
        thread.setName("t1");
        thread.start();

        System.out.println(Thread.currentThread().getName()+"  running !");
    }
}
//输出
main  running !
t1  running!
~~~

#### 方式二：使用 Runnable 配合 Thread

创建多线程的方式二：实现`Runnable`接口

1. 创建一个实现了`Runnable`接口的类
2. 实现类去实现`Runnable`中的抽象方法：run()
3. 创建实现类的对象
4. 将此对象作为参数传递到`Thread`类的构造器中，创建`Thread`类的对象
5. 通过`Thread`类的对象调用`start()`

**查看Runnable接口的源码**

~~~ java
@FunctionalInterface
public interface Runnable {
    public abstract void run();//只有一个run()抽象方法
}
~~~

> 在`jdk`中，只要接口中只有一个抽象方法，就可以使用`lambda`表达式进行简化。也可以从`@FunctionalInterface`注解进行判断，只要带有此注解的接口，就可以使用`lambda`表达式进行简化。

**代码演示**

```java
public class Test02 {
    public static void main(String[] args) {
//        1 实现接口
        Runnable runnable=new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()+"  running !");
            }
        };
//        2 创建线程，把runnable作为对象进行传递
        Thread thread=new Thread(runnable);
//        设置线程的名字
        thread.setName("t1");
//        3 启动线程
        thread.start();

//        主线程
        System.out.println(Thread.currentThread().getName()+"  running");
    }
}
//输出
main  running
t1  running !
```

**使用`lambda`表达式**

~~~ java
public class Test02 {
    public static void main(String[] args) {
//        1 实现接口,使用lambda表达式创建对象
        Runnable runnable =() ->{
//            下面直接写方法体
            System.out.println(Thread.currentThread().getName()+"  running !");
        };
//        2 把runnable作为对象进行传递
        Thread thread=new Thread(runnable);
//        设置线程的名字
        thread.setName("t1");
//        3 启动接口
        thread.start();

//        主线程
        System.out.println(Thread.currentThread().getName()+"  running");
    }
}
~~~

- 比较创建线程的两种方式。
  - 开发中：优先选择：实现`Runnable`接口的方式，原因：实现的方式没有类的单继承性的局限性，实现的方式更适合来处理多个线程有共享数据的情况。 

#### 原理之 Thread 与 Runnable 的关系

**从源码的角度看看`Runnalle`的工作原理**

~~~ java
Thread thread=new Thread(runnable);//构造函数

//Thread类的构造函数
public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
//init()方法
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;

        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
        g.checkAccess();

        /*
         * Do we have the required permissions?
         */
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        g.addUnstarted();

        this.group = g;
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;//在这里，我们发现把target对象给了Thread对象的一个属性
        setPriority(priority);
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;

        /* Set thread ID */
        tid = nextThreadID();
    }

//我们发现在Thread类的run()方法中使用了target对象，也就是说Runnable底层实际还是调用的是Thread类的run()方法，如果runnable自己有run方法，那么就会采用runnable的run()方法
 @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
~~~

**源码角度理解`Thread`方法**

~~~ java
Thread thread=new Thread(){
//            此线程的执行和main线程的执行是不一样的
//            run()内写逻辑代码
            @Override
            public void run() {
                super.run();
                System.out.println(Thread.currentThread().getName()+"  running!");
            }
        };,
//我们继承Thread类，然后重写父类中的run()方法，那么调用的自然是子类的run()方法，不难理解
~~~

- 总结一句话就是，不管使用的是哪一种方式创建对象，在底层调用的都是`run()`方法。
- 使用`Thread`是把线程和任务合并在了一起，`Runnable`是把线程和任务分开了，用` Runnable` 更容易与线程池等高级` API `配合，用` Runnable `让任务类脱离了` Thread `继承体系，更灵活。通过查看源码可以发现，方法二其实到底还是通过方法一执行的！
- 不推荐直接操作`Thread`对象。

#### 方法三，FutureTask 配合 Thread

**先来看看FutureTask的继承关系**

![1608080880307](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/090801-486054.png)

> `FutureTask`实现了`RunnableFeature`接口，`RunnableFeature`接口继承`Runnable`接口和`Future`接口。
>
> `Future`接口是用来返回任务的执行结果的，而`Runnable`只有一个抽象方法，不可以在两个线程之间相互传递结果。
>
> Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果
>
> runnable并没有定义其他的方法，所以不容易在两个线程之间传递结果。

**使用FutureTask创建线程**

`FutureTask `能够接收` Callable` 类型的参数，用来处理有返回结果的情况

~~~ java
//和Runnable接口很相似，区别是此接口中call方法可以有返回值和抛出异常
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
~~~

**代码演示**

~~~ java
public class Test03 {
    public static void main(String[] args) {
        //1 创建对象
        FutureTask <Integer>task=new FutureTask<Integer>(new Callable() {
            @Override
            public Object call() throws Exception {
                System.out.println(Thread.currentThread().getName()+"  running !");
                Thread.sleep(1000);
              //在这里可以返回结果
                return 100;
            }
        });
    //2 创建Thread对象
//        因为FutureTask实现了Runnable接口，所以可以传递给Thread()
        Thread thread=new Thread(task);
        thread.setName("t1");
//        3 启动线程
        thread.start();
//        4 获取任务的返回结果
        try {
            Integer integer=task.get();
            System.out.println("integer :"+integer);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

				task.get()//获取返回的结果
//        主线程
        System.out.println(Thread.currentThread().getName()+"  running");
    }
}

~~~

- `Future`提供了三种功能：   　　

1. 判断任务是否完成；   　　
2. 能够中断任务；   　　
3. 能够获取任务执行结果。

> 多个线程的执行顺序是有底层的调度器进行调度的，无法干预。

### 查看进程的方法

#### windows

~~~ java
//任务管理器可以查看进程和线程数，也可以用来杀死进程
tasklist 查看进程
taskkill 杀死进程
tasklist | findstr java //通过筛选查看java进程
taskkill /F /PID 号码 //强制杀死某一个进程
jps //是jdk自带的查看进程的命令
~~~

#### linux

~~~ java
ps -fe //查看所有进程
ps -fT -p 进程号 //查看某个进程（PID）的所有线程
kill  进程号 //杀死进程的pid
top //按大写 H 切换是否显示线程,top命令可以动态展示进程的信息
top -H（表示查看某一个线程） -p（表示进程的id）// 查看某个进程（PID）的所有线程
//使用grep和管道运算符
ps -fe  | grep java(表示关键字)
~~~

#### Java

~~~ java
javac // 编译
java //运行java程序
jps 命令查看所有 Java 进程
jstack 进程pid // 查看某个 Java 进程（PID）的所有线程状态
jconsole //来查看某个 Java 进程中线程的运行情况（图形界面）
~~~

#### 查看进程和线程的工具

在cmd窗口中输入jconsole。

### 线程运行的原理

#### jvm的栈与栈帧

拟虚拟机栈描述的是`Java`方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧(`stack  frame`)用于存储**局部变量表、操作数栈、动态链接、方法出口等信息**，是属于线程的私有的。当`java`中使用多线程时，每个线程都会维护它自己的栈帧！每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法。

#### 线程的运行原理

**栈与栈帧的演示**

~~~ java
public class TestStack {
    public static void main(String[] args) {
        test01(5);

    }
    public static void test01(int x){
        int y=x+1;
        Object o=test02();
        System.out.println("o ="+o);
    }
    public static Object test02(){
        Object o=new Object();
        return o;
    }
}
~~~

1. 当我们启动一个`java`程序的时候，就相当于启动一个`jvm`进程，`jvm`进程会分配到运行时数据区的内存资源，`cpu`等资源，首先会在栈中存放主线程的栈帧，压入栈底部，同时我们也可以看到`main`线程对应的参数,每一个方法的参数和局部变量信息存放在其对应栈帧中的局部变量表中。

![1608084042999](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/113634-678149.png)

2. 接下来我们进入`test01()`方法的内部，那么在`java`栈中会压入一个栈帧存放`test01()`方法的运行信息，同时把`test01()`方法中的参数和局部变量全部存储到`test01()`方法对应的局部变量表中。

![1608084325812](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/100527-364772.png)

3. 现在`test02()`方法也被压入栈帧当中。

![1608084445126](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/100726-333007.png)

3. 当我们的``test02()`方法执行完毕之后，那么在栈中，`test02()`对应的栈帧也就出栈了，内存被释放。`test01()`方法在调用`test02()`方法时候，会记录一下调用地址，当`test02()`方法执行完毕之后，会返回结果到调用处重新接着`test01()`方法继续执行。

![1608084593146](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/100954-369261.png)

4. 最后当我们的main()函数也执行完毕之后，那么所有的栈帧都会从栈中抛出，`java`程序运行结束，

![1608084809439](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/101331-930962.png)

- 总结来说就是，每一个栈帧，就对应这一个方法的调用。

如果这块想要深入理解，可以去看看这篇文章，是关于`jvm`的。

**上面的程序的执行原理**

![1619351859743](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/25/195741-680119.png)

#### 线程的运行原理（多线程）

**代码**

~~~ java
public class TestStack {
    public static void main(String[] args) {

        Thread thread=new Thread("t1"){
            @Override
            public void run() {
                super.run();
                test01(10);
            }
        };
//        启动子线程
        thread.start();
//        启动主线程
        test01(5);

    }
    public static void test01(int x){
        int y=x+1;
        Object o=test02();
        System.out.println("o ="+o);
    }
    public static Object test02(){
        Object o=new Object();
        return o;
    }
}
~~~

1. 同时启动两个线程，查看我们的`main`线程

![1608086425405](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/104026-46782.png)

2. 查看我们的`t1`线程

![1608086472809](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/104113-899332.png)

3. 执行我们的`main`线程，此时`main`线程已经执行完毕`test02()`方法。

![1608086631860](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/113631-968022.png)

4. 查看我们的`t1`线程执行的位置，通过查看，我们发现`t1`进程还没有开始执行。

![1608086931753](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/113753-727620.png)

5. 执行完毕我们的`t1`线程

![1608087051481](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/105052-311195.png)

6. 现在再次查看我们的`main`线程，发现`main`线程还没有执行完毕，被阻塞在输出的位置。

![1608087148872](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/105230-615332.png)

7. 执行完毕`main`方法后，所有线程都运行完毕，没有线程可执行

![1608087227764](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/105348-741793.png)

> 上面在打断点调试的时候，断电的类型要选择`Thread`类型的。

**小结**

通过追踪的方式，我们发现，线程之间的执行都是相互不会影响的，各自都有自己的运行时数据区，相当于每一个线程都有自己的小黑屋，独立运行，互相不会影响，至少到目前这里，我们认为是这样，后面那可就不一定了。

#### 线程的上下文切换（Thread Context Switch）

因为以下一些原因导致 `cpu` 不再执行当前的线程，转而执行另一个线程的代码

**被动切换**

- 线程的` cpu` 时间片用完
- 垃圾回收
- 有更高优先级的线程需要运行

**主动切换**

- 线程自己调用了 `sleep、yield、wait、join、park、synchronized、lock `等方法

  当 `Context Switch `发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，`Java `中对应的概念就是程序计数器（`Program Counter Register`），它的作用是记住下一条` jvm `指令的执行地址，是线程私有的。

- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等
- `Context Switch `频繁发生会影响性能,所以说并不是线程数量越多越好，当线程的数量超过`cpu`的核心数量，那么就需要频繁的在各个线程之间进行切换，很明显会影响吞吐量。

### 常见方法

![1608547620381](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/21/184701-371738.png)

![1608547650272](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/21/184732-165742.png)

![1608547676902](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/21/184758-574487.png)



#### Start()与Run()

`start()`表示启动一个线程，而`run()`表示一个线程启动后要执行的代码。

**为什么不可以直接调用`run()`方法呢？下面我们通过代码解释**

- 代码展示

~~~ java
public class Test04 {
    public static void main(String[] args) {
        Thread t1=new Thread("t1"){
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()+"  running");
            }
        };
        t1.run();//在这里我们调用的是run()方法
        System.out.println(Thread.currentThread().getName()+"  do other thing");
    }
}
//输出结果
main  running
main  do other thing
~~~

通过上面的输出我们发现，虽然新创建了一个线程去执行别的操作，但是最后打印出执行操作的是`main`线程，并不是我们新创建的线程`t1`。下面我们调用`start`启动线程。

- 调用`start`启动线程

~~~ java
public class Test04 {
    public static void main(String[] args) {
        Thread t1=new Thread("t1"){
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()+"  running");
            }
        };
        t1.start();//在这里我们使用start()方法启动线程
        System.out.println(Thread.currentThread().getName()+"  do other thing");
    }
}
//输出结果
main  do other thing
t1  running
~~~

输出结果展示`jvm`重新给我们新启动了一个线程`t1`去执行我们的代码，而不再是主线程去执行。

**解释**：如果直接调用`run`方法去启动线程，其实`jvm`是没有给我们新创建线程，而是使用的是主线程去执行我们的所有操作，但是如果使用的是`start`方法启动，那么`jvm`会为我们新创建一个线程去执行其他的代码。

#### 获取线程的状态

~~~ java
public class Test04 {
    public static void main(String[] args) {
        Thread t1=new Thread("t1"){
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()+"  running");
            }
        };
        System.out.println(t1.getState());;
        t1.start();
//        t1.start();一个线程启动后，不可以多次启动
        System.out.println(t1.getState());
        System.out.println(Thread.currentThread().getName()+"  do other thing");
    }
}
//输出结果
NEW //表示线程是新创建的
RUNNABLE //表示线程处于运行状态
main  do other thing
t1  running
~~~

####  sleep 与 yield

**Sleep**

调用 sleep 会让当前线程从 Running 进入 Timed Waiting 状态（阻塞）
2. 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException
3. 睡眠结束后的线程未必会立刻得到执行，也就是线程从**阻塞—>就绪状态。**
4. 建议用 `TimeUnit` 的 sleep 代替 Thread 的 sleep 来获得更好的可读性

~~~ java
 TimeUnit.SECONDS.sleep(1000);
~~~

5. Sleep方法写在哪一个线程里面，就阻塞哪一个线程。

**代码测试**

~~~ java
public class Test05 {
    public static void main(String[] args) {
        Thread t1=new Thread(){
            @Override
            public void run() {
                try {
//                    参数代表是毫秒数，1s=1000ms
                    Thread.sleep(2000);
                }catch (Exception e){
                    e.printStackTrace();
                }

            }
        };
        t1.setName("t1");
        t1.start();
        System.out.println("t1 statu:{}"+t1.getState());
    }
}
//输出
t1 statu:{}RUNNABLE
~~~

**yield**

1. 调用 yield 会让当前线程从 Running 进入 Runnable 就绪状态，然后调度执行其它线程，也就是使线程从**运行—>就绪。**
2. 具体的实现依赖于操作系统的任务调度器，可能当前没有就绪的线程，那么此时调度器还会把资源分配给当前线程使用。

#### 线程优先级

- 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它
- 如果` cpu `比较忙，那么优先级高的线程会获得更多的时间片，但` cpu `闲时，优先级几乎没作用

**代码演示**

~~~ java
Runnable task1 = () -> {
        int count = 0;
        for (; ; ) {
            System.out.println("---->1 " + count++);
        }
    };
    Runnable task2 = () -> {
        int count = 0;
        for (; ; ) {
            // Thread.yield();
            System.out.println(" ---->2 " + count++);
        }
    };
    Thread t1 = new Thread(task1, "t1");
    Thread t2 = new Thread(task2, "t2");
// t1.setPriority(Thread.MIN_PRIORITY);
// t2.setPriority(Thread.MAX_PRIORITY);
    t1.start();
    t2.start();
~~~

虽然可以设置线程的优先级，但是最终线程的运行状况还是由我们的任务调度器决定的，这两个方法最终仅仅是给任务调度器一个提示而已，我们是无法干预的。

### 案例展示

#### 使用Sleep来限制对cpu的使用

- 在没有利用 `cpu` 来计算时，不要让 while(true) 空转浪费` cpu`，这时可以使用 yield 或 sleep 来让出 `cpu `的使用权给其他程序

~~~ java
public class TestCpu {
    public static void main(String[] args)  {
        new Thread(() ->{
            while (true){
                try {
                   // Thread.sleep(1000);//程序不会休眠
                }catch (Exception e){
                    e.printStackTrace();
                }

            }
        }).start();
    }
}
~~~

**`cpu`占用率**：此时发现占用率很高，第一个`java`程序就是我们运行这的程序。但是如果添加上sleep语句，`cpu`的利用率很快就可以降下来。

![1608108862899](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/165423-409788.png)

- 可以用 wait 或 条件变量达到类似的效果
- 不同的是，后两种都需要加锁，并且需要相应的唤醒操作，一般适用于要进行同步的场景
- sleep 适用于无需锁同步的场景

### Join()方法

**为什么会需要join方法**，先来看看下面的程序输出什么

~~~ java
public class Test06 {
    static int i=0;
    public static void main(String[] args) {
        test01();

    }
    public static void test01(){
        System.out.println(Thread.currentThread().getName()+"  开始：");
        Thread thread=new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"  开始:");
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"  结束：");
            i=10;
        });
        thread.setName("t1");
        thread.start();
        System.out.println(Thread.currentThread().getName()+"  结果是：{}"+i);
    }
}
//输出
main  开始：
main  结果是：{}0
t1  开始:
t1  结束：
~~~

- 分析
  - 因为主线程和线程`t1 `是并行执行的，`t1 `线程需要 1 秒之后才能算出 r=10
  - 而主线程一开始就要打印 r 的结果，所以只能打印出 r=0
- 解决方法
  - 用 sleep 行不行？为什么？不可以，因为我们的子线程的运行是不可预测的，也就是我们不知道从运行开始到结束花费多少时间，也就是我们的主线程等待的时间不好计算。
  - 用 join，加在` t1.start()` 之后即可，join()方法是等待一个线程结束，哪一个线程对象调用此方法，join()就等待哪一个线程。

**代码测试**

~~~ java
public class Test06 {
    static int i=0;
    public static void main(String[] args) throws InterruptedException {
        test01();

    }
    public static void test01() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"  开始：");
        Thread thread=new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"  开始:");
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"  结束：");
            i=10;
        });
        thread.setName("t1");
        thread.start();
//        我们的主线程要等待thread线程运行结束，所在在thread启动之后，再用t1线程调用join()方法，也就是说明让thread线程抢占资源先运行，
        thread.join();//当前值main线程执行了此语句，也就是主线程等待thread线程执行中止后才从thread.join()返回
        System.out.println(Thread.currentThread().getName()+"  结果是：{}"+i);
    }
}
//输出结果
main  开始：
t1  开始:
t1  结束：
main  结果是：{}10//等待t1线程结束后才输出
~~~

**图解**

![1608184506504](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/17/135508-560676.png)

#### 同步

- 需要等待结果返回，才能继续运行就是同步
- 不需要等待结果返回，就能继续运行就是异步

**代码说明**

~~~ java
public class Test07 {
        public static int r1=0;
        public static int r2=0;
    public static void main(String[] args) throws InterruptedException {
        Test01();

    }
    public static void Test01() throws InterruptedException {
        Thread thread=new Thread(() ->{
            try {
                Thread.sleep(1);
              r1=10;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            
        });
        Thread thread1=new Thread(()->{
            try {
                Thread.sleep(2);
              r2=20;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            
        });
        long start=System.currentTimeMillis();
        thread.start();;
        thread1.start();
        thread.join();
        thread1.join();
        long end=System.currentTimeMillis();
        System.out.println(end-start);
        System.out.println(r1);
        System.out.println(r2);

    }
}
//输出
10
  20
  2
~~~

- 分析如下
  - 第一个 join：等待 t1 时, t2 并没有停止, 而在运行
  - 第二个 join：1s 后, 执行到此, t2 也运行了 1s, 因此也只需再等待 1s
  - 如果颠倒两个 join 呢？输出结果一致

**图解等待过程**

![1608185219386](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/17/140702-314322.png)

#### 有时效的join

**代码说明**

~~~ java
//等够时间 
public static void test02() throws InterruptedException {
        Thread thread=new Thread(()->{
           try {
               //子线程在2秒后会执行结束
               Thread.sleep(2);
               r1=10;
           }catch (Exception e){
               e.printStackTrace();
           }
        });
        long start=System.currentTimeMillis();
        thread.start();
        //而在主线程这里，只等待了1.5秒，也就是说没有等待子线程执行结束
        thread.join(3000);
        long end=System.currentTimeMillis();
        System.out.println(r1);
        System.out.println(end-start);
    }
//没有等够时间
 public static void test02() throws InterruptedException {
        Thread thread=new Thread(()->{
           try {
               //子线程在2秒后会执行结束
               Thread.sleep(2);
               r1=10;
           }catch (Exception e){
               e.printStackTrace();
           }
        });
        long start=System.currentTimeMillis();
        thread.start();
        //而在主线程这里，值等待了1.5秒，也就是说没有等待子线程执行结束
        thread.join(1500);
        long end=System.currentTimeMillis();
        System.out.println(r1);
        System.out.println(end-start);
    }

~~~



###  interrupt 方法详解

**打断 sleep，wait，join 的线程**

- 这几个方法都会让线程进入阻塞状态
- 打断 `sleep `的线程, 会清空打断状态，也就是会设置打断状态为`false`，一般情况下，对于正常运行的程序，被其他线程打断后，其打断状态是真，以 sleep 为例。

**代码说明**

~~~ java
public class Test08 {
    public static void main(String[] args) throws InterruptedException {
        Thread thread=new Thread(()->{
            System.out.println("Sleeping....");
            try {
                Thread.sleep(5000);//wait join
            }catch (Exception e){
                e.printStackTrace();
            }
        });
        thread.setName("t1");
        thread.start();

//        让主线程来打断子线程
//        这里让主线程休眠1秒是为了先让子线程进入休眠，然后再去打断
        Thread.sleep(1);
        System.out.println("interupt......");
        thread.interrupt();
        System.out.println("打断标记："+thread.isInterrupted());
    }
}
//输出结果
Sleeping....
interupt......
打断标记：false
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at rzf.qq.com.MyThread.Test08.lambda$main$0(Test08.java:8)
	at java.lang.Thread.run(Thread.java:748)
~~~

**打断正常运行的程序**

**打断正常运行的线程, 不会清空打断状态**，打断正常运行的程序，程序不会立马进入阻塞状态，相反还会继续执行，我们可以根据打断状态标记做一些其他的工作，然后在结束线程的执行。

**代码说明**

~~~ java
public class Test09 {
    public static void main(String[] args) throws InterruptedException {
        Thread thread=new Thread(()->{
           while (true){
               boolean interrupt=Thread.currentThread().isInterrupted();
               if(interrupt){
                   System.out.println(Thread.currentThread().getName()+"已经被打断");
                   break;
               }
           }
        },"t1");
        thread.start();
//        主线程休眠两秒再去做打断工作
        Thread.sleep(2);
        System.out.println(Thread.currentThread().getName()+"开始执行打断操作");
        thread.interrupt();
    }
}
~~~

#### 打断park()线程

打断 `park` 线程, 不会清空打断状态，也就是打断之后，状态是`true`。

**代码说明**

~~~ java
public class Test11 {
    public static void main(String[] args) throws InterruptedException {
            test01();
    }

    public static void test01() throws InterruptedException {
        Thread thread=new Thread(()->{
            System.out.println("parking.....");
            LockSupport.park();
            System.out.println("unparking.....");
            System.out.println("打断状态："+Thread.currentThread().isInterrupted());
        });

        thread.start();

//        主线程休息两秒后去打断子线程
        Thread.sleep(2);
        thread.interrupt();
    }
}
//interrupt（）方法可以打断正处在park()模式的线程，如果不打断，park()类型的线程将会处于阻塞状态
//打断park线程后，线程状态标记会设置为true,如果再次调用park（）线程，那么将不会启动，也就是说在状态标记为true的时候，park()会失效
~~~

**修改状态标记**

> 可以使用 `Thread.interrupted()` 清除打断状态

~~~ java
public class Test11 {
    public static void main(String[] args) throws InterruptedException {
            test01();
    }

    public static void test01() throws InterruptedException {
        Thread thread=new Thread(()->{
            System.out.println("parking.....");
            LockSupport.park();
            System.out.println("unparking.....");
            System.out.println("打断状态："+Thread.currentThread().isInterrupted());//true
            Thread.interrupted();//在这里重新修改状态标记
            LockSupport.park();//重新生效
           
            System.out.println("再次调用park线程");
        });

        thread.start();

//        主线程休息两秒后去打断子线程
        Thread.sleep(2);
        thread.interrupt();
    }
}

~~~

### 不推荐使用的方法

还有一些不推荐使用的方法，这些方法已过时，容易破坏同步代码块，造成线程死锁

| 方法名    | static | 说明                   |
| --------- | ------ | ---------------------- |
| stop()    |        | 停止线程运行           |
| suspend() |        | 挂起或者暂停线程的运行 |
| resume()  |        | 恢复线程的运行         |



### 多线程编程模式

#### 两阶段终止模式

**图解两阶段终止模式**

![1608189958073](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/17/152559-693949.png)

**代码实现**

~~~ java
public c  lass Test10 {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination t1=new TwoPhaseTermination();
        t1.start();

//        主线程休眠3.5秒
        Thread.sleep(3500);
//        主线程去打断子线程的执行
        t1.stop();

    }
}

class TwoPhaseTermination{
//    创建一个监控线程
    private Thread monitor;
//    启动监控线程
    public void start(){
        monitor=new Thread(()->{
//            在这里时刻监控当前线程是否被打断
            while (true){
                boolean interrupt=Thread.currentThread().isInterrupted();
                if(interrupt){
                    System.out.println("程序已经被打断........");
                    break;
                }
//                如果没有被打断，就执行监控操作
                try {
//                    在下面这两条语句都有可能被打断
                    Thread.sleep(1000);//这种情况如果被打断，打断标记将会被设置为false，这里打断会抛出异常
//                    下面语句如果被打断，那么是正常被打断，她的的打断标记会设置为true,可以正常退出
                    System.out.println("执行监控记录......");
                } catch (InterruptedException e) {
                    e.printStackTrace();
//                  重新设置打断标记
                    Thread.currentThread().interrupt();//在这里把打断标记重新设置为true
                }
            }
        });
        monitor.start();
    }

//    提供停止监控线程
    public void stop(){
//        对线程进行打断
        monitor.interrupt();

    }
}
~~~

### 主线程与守护线程

默认情况下，`Java` 进程需要等待所有线程都运行结束，才会结束。有一种特殊的线程叫做守护线程，只要其它非守护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束。

**代码说明**

~~~ java
log.debug("开始运行...");
Thread t1 = new Thread(() -> {
 log.debug("开始运行...");
 sleep(2);
 log.debug("运行结束...");
}, "daemon");
// 设置该线程为守护线程
t1.setDaemon(true);
t1.start();
sleep(1);
log.debug("运行结束...");
~~~

> 注意：
>
> - 垃圾回收器线程就是一种守护线程
> - `Tomcat `中的 `Acceptor `和 `Poller` 线程都是守护线程，所以 `Tomcat `接收到 `shutdown `命令后，不会等
>   待它们处理完当前请求
> - 守护线程需要在线程启动之前进行设置，不可以在线程启动之后设置

**守护进程和守护线程的区别**   

- 无论是进程还是线程， 都遵循： 守护xxx 会等待主xxx 运行完毕后被销毁
  - 守护进程 :只会守护到主进程的代码结束
  - 守护线程 :会守护所有其他**非守护线程**的结束

**运行完毕并非终止运行**

- 对主进程来说， 运行完毕指的是主进程代码运行完毕
- 对主线程来说， 运行完毕指的是主线程所在的进程内所有非守护线程统统运行完毕， 主线程才算运行完毕

**守护进程**

- 主进程和子进程互不干扰
- 主进程执行完毕之后程序不会结束,会等待所有的子进程结束之后才结束

是一个子进程,守护的是主进程

结束条件 : 主进程的代码结束,守护进程也结束

> - 主进程的代码结束,守护进程结束
> - 主进程要回收守护进程(子进程)的资源
> - 主进程等待其他所有子进程结束
> - 主进程回收所有子进程的资源

**守护线程**

- 主线程会等待子线程的结束而结束
- 守护线程会随着主线程的结束而结束
- 守护线程会守护主线程和所有的子线程

**守护线程问题**

- 主线程需不需要回收子线程的资源
  - 不需要,线程资源属于进程,所以进程结束了,线程的资源自然就被回收了
- 主线程为什么要等待子线程结束之后才结束
  - 主线程结束意味着进程进程,进程结束,所有的子线程都会结束，要想让子线程能够顺利执行完,主线程只能等
- 守护线程到底是怎么结束的
  - 主线程结束了,主进程也结束,守护线程被主进程的结束结束掉了



### 线程的状态转换

> 从操作系统层面理解线程的转换

![1608265195740](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/18/121957-958348.png)

- 【初始状态】仅是在语言层面创建了线程对象，还未与操作系统线程关联
- 【可运行状态】（就绪状态）指该线程已经被创建（与操作系统线程关联），可以由` CPU `调度执行
- 【运行状态】指获取了 `CPU `时间片运行中的状态
  - 当 `CPU` 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换
- 【阻塞状态】
  - 如果调用了阻塞 `API`，如 `BIO `读写文件，这时该线程实际不会用到 `CPU`，会导致线程上下文切换，进入【阻塞状态】
  - 等 `BIO `操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】
  - 与【可运行状态】的区别是，对【阻塞状态】的线程来说只要它们一直不唤醒，调度器就一直不会考虑调度它们
- 【终止状态】表示线程已经执行完毕，生命周期已经结束，不会再转换为其它状态

**从JAVA API层面讨论进程的状态**

根据 `Thread.State` 枚举，分为六种状态

![1608266077895](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/18/123439-360586.png)

- `NEW` 线程刚被创建，但是还没有调用 `start() `方法
- `RUNNABLE` 当调用了 `start() `方法之后，注意，`Java API `层面的 `RUNNABLE `状态涵盖了 操作系统 层面的
  【可运行状态，就绪状态】、【运行状态】和【阻塞状态】（由于 `BIO `导致的线程阻塞，在 `Java` 里无法区分，仍然认为是可运行）
- `BLOCKED` ， `WAITING` ， `TIMED_WAITING `都是` Java API` 层面对【阻塞状态】的细分，后面会在状态转换一节详述。
- `TERMINATED` 当线程代码运行结束

**代码说明**

~~~ java
public class TestState {
    public static void main(String[] args) {
//        对t1线程并不调用start方法，所以状态是new
        Thread t1=new Thread("t1"){
            @Override
            public void run() {
                System.out.println("running....");
            }
        };
//      t2线程已经启动，所以处于可运行状态
        Thread t2=new Thread("t2"){
            @Override
            public void run() {
                while (true){//这里的运行表示是java中的可运行状态，有三种
                }
            }
        };
        t2.start();

//        t3会早于主线程结束，所以t3会处于终止状态
        Thread t3=new Thread("t3"){
            @Override
            public void run() {
                System.out.println("running.....");
            }
        };
        t3.start();

//        t4处于休眠状态，所以会打印time_waiting,有时限的等待
        Thread t4=new Thread("t4"){
            @Override
            public void run() {
                synchronized (TestState.class){
                    try {
                        Thread.sleep(1000000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        t4.start();

//        t5会一直处于waiting状态
        Thread t5=new Thread("t5"){
            @Override
            public void run() {
                try {
//                    在这里t5会一直等待t2,但是t2是处于死循环，所以t5会一直等待
                    t2.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        t5.start();

//        因为前面t4已经对类进行加锁，所以t6再也拿不到所，也就会进入block阶段
        Thread t6=new Thread("t6"){
            @Override
            public void run() {
                synchronized (TestState.class){
                    try {
                        Thread.sleep(1000000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        t6.start();
        System.out.println("t1 state:"+t1.getState());
        System.out.println("t2 state:"+t2.getState());
        System.out.println("t2 state:"+t3.getState());
        System.out.println("t2 state:"+t4.getState());
        System.out.println("t2 state:"+t5.getState());
        System.out.println("t2 state:"+t6.getState());

    }
}

//输出
running.....
t1 state:NEW
t2 state:RUNNABLE
t2 state:TERMINATED
t2 state:TIMED_WAITING
t2 state:WAITING
t2 state:BLOCKED

Process finished with exit code -1
~~~

**线程状态变迁图小结**

![1608541402608](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/21/170325-178639.png)

- 当线程执行wait方法后会进入等待状态，进入等待状态的线程需要其他线程的唤醒才可以进入运行状态。
- 超时等待是在等待的基础上增加超时限制，也就是超过时间的限定时会自动进入运行状态。
- 当线程调用同步方法时，如果没有获取到锁，就会进入阻塞状态。

> 注意上面的图中,`java`将操作系统中**运行和就绪**两种状态合并为运行状态。
>
> 阻塞状态是线程阻塞在了`synchronized`关键字修饰的方法或者代码块时的状态，也就是线程没有获取到锁，而处于阻塞状态。

### 本章小结

**本章的重点在于掌握**

- 线程创建

- 线程重要` api`，如 `start`，`run`，`sleep`，`join`，`interrupt` 等

- 线程状态

- 应用方面

  - 异步调用：主线程执行期间，其它线程异步执行耗时操作
  - 提高效率：并行计算，缩短运算时间
  - 同步等待：join
  - 统筹规划：合理使用线程，得到最优效果


  - 原理方面

      - 线程运行流程：栈、栈帧、上下文切换、程序计数器
      - `Thread `两种创建方式 的源码

- 模式方面

     - 终止模式之两阶段终止

## 思维导图总结

![1610331048801](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/11/101050-510264.png)

## 共享模型 之管程(悲观锁)

可见性，原子性，有序性

> 1. **关键字`synchronized`可以修饰方法或者以同步代码块的形式来使用，他主要确保多个线程在同一个时刻。只有一个线程处于方法或者是代码块中，他保证线程对变量的访问的可见性和排他性（互斥访问）**
>
> 2. **关键字`volatile`可以用来修饰字段（成员变量），就是告知程序在任何对该变量的访问均需要从共享内存中获取，而对他的改变必须同步刷新回共享内存，他能够保证所有线程对变量的访问的可见性。**

### `java`中多线程操作共享变量的问题

两个线程对初始值为 0 的静态变量一个做自增，一个做自减，各做 5000 次，结果是 0 吗？

**代码说明**

~~~ java
public class Test14 {
    public static int i=0;
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(()->{
            for(int j=0;j<5000;j++){
                i++;
            }
        });

        Thread t2=new Thread(()->{
            for(int j=0;j<5000;j++){
                i--;
            }
        });
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}
//输出：正常输出应该是0.但是此程序输出的结果每一次都不是0，并且还不一样，这就是java并发操作共享变量带来的问题
~~~

#### 问题分析

以上的结果可能是正数、负数、零。为什么呢？因为 `Java` 中对静态变量的自增，自减并不是原子操作，要彻底理解，必须从字节码来进行分析

- 例如对于` i++ `而言（`i` 为静态变量），实际会产生如下的 `JVM `字节码指令：

~~~ java
getstatic i // 获取静态变量i的值
iconst_1 // 准备常量1
iadd // 自增
putstatic i // 将修改后的值存入静态变量i
~~~

- 而对应 i-- 也是类似：

~~~ java
getstatic i // 获取静态变量i的值
iconst_1 // 准备常量1
isub // 自减
putstatic i // 将修改后的值存入静态变量i
~~~

**而` Java` 的内存模型如下，完成静态变量的自增，自减需要在主存和工作内存中进行数据交换：**

![1608337257773](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/19/082058-805883.png)

**如果是单线程以上 8 行代码是顺序执行（不会交错）没有问题：**因为自始至终都是有一个线程来操作共享的变量。

![1608337288446](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/19/082129-772104.png)

**但多线程下这 8 行代码可能交错运行：**

- 出现负数的情况

![1608337365035](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/19/082246-94236.png)

- 出现正数的情况

![1608337405504](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/19/082326-535489.png)

#### 临界区 Critical Section

- 一个程序运行多个线程本身是没有问题的
- 问题出在多个线程访问**共享资源**
  - **多个线程读共享资源其实也没有问题**
  - **在多个线程对共享资源读写操作时发生指令交错，就会出现问题**
- 一段代码块内如果存在对**共享资源**的多线程读写操作，称这段代码块为**临界区**

**代码说明**

~~~ java
static int counter = 0;
static void increment()
// 临界区
{
 counter++;
}
static void decrement()
// 临界区
{
 counter--;
}
~~~

#### 竞态条件 Race Condition

多个线程在**临界区内执行**，由于代码的**执行序列不同而导致结果无法预测**，称之为发生了**竞态条件**

### synchronized解决方案

#### 应用之互斥

为了避免临界区的竞态条件发生，有多种手段可以达到目的。

- 阻塞式的解决方案：**`synchronized`，Lock**
- 非阻塞式的解决方案：**原子变量**

本次使用阻塞式的解决方案：`synchronized`，来解决上述问题，即俗称的【对象锁】，它采用**互斥**的方式让同一时刻至多只有一个线程能持有【对象锁】，其它线程再想获取这个【对象锁】时就会阻塞住。这样就能保证拥有锁的线程可以安全的执行临界区内的代码，不用担心线程上下文切换

> - 注意
>   - 虽然 `java `中**互斥和同步**都可以采用` synchronized `关键字来完成，但它们还是有区别的：
>     - 互斥是保证临界区的竞态条件发生，**同一时刻只能有一个线程执行临界区代码**
>     - 同步是由于线程**执行的先后、顺序不同、需要一个线程等待其它线程运行到某个点**

#### synchronized

**语法**

~~~ java
synchronized(对象) // 线程1， 线程2(blocked)阻塞状态
{
 //临界区，也就是需要受保护的代码
}
//对对象加锁，但是需要保证同一时刻只有一个线程对此对象进行操作
~~~

**加锁改进代码**

~~~ java
public class Test14 {
    public static int i=0;
//    在这里需要创建一个对象，应为使用锁需要一个共享的对象来让各个线程加锁访问
    static Object lock=new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(()->{
            for(int j=0;j<5000;j++){
//                现在可以对临界区的代码进行加锁
                synchronized (lock){
                    i++;
                }

            }
        });

        Thread t2=new Thread(()->{
            for(int j=0;j<5000;j++){
//                对临界区进行加锁
                synchronized (lock){//同样锁住的是lock对象
                    i--;
                }
            }
        });
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}
//现在无论运行多少次，结果都是0
~~~

#### synchronized锁原理理解

- `synchronized(对象) `中的对象，可以想象为一个房间（`room`），有唯一入口（门）房间只能一次进入一人
  进行计算，线程` t1`，`t2 `想象成两个人

  - 当线程 `t1 `执行到` synchronized(room) `时就好比 `t1 `进入了这个房间，并锁住了门拿走了钥匙，在门内执行`count++` 代码
  - 这时候如果` t2 `也运行到了` synchronized(room) `时，它发现门被锁住了，只能在门外等待，发生了上下文切换，阻塞住了
  - **这中间即使` t1` 的` cpu `时间片不幸用完，被踢出了门外（不要错误理解为锁住了对象就能一直执行下去哦），这时门还是锁住的，`t1` 仍拿着钥匙，`t2` 线程还在阻塞状态进不来，只有下次轮到` t1` 自己再次获得时间片时才能开门进入**
  - 当` t1` 执行完` synchronized{} `块内的代码，这时候才会从 `obj `房间出来并解开门上的锁，唤醒` t2 `线程把钥匙给他。`t2` 线程这时才可以进入 `obj `房间，锁住了门拿上钥匙，执行它的` count--` 代码

  **图解**

  ![1608338954219](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/19/084915-187749.png)

**图解上述代码执行过程**

![1608338992320](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/19/084954-983364.png)

#### 思考

`synchronized` 实际是用**对象锁**保证了临界区内代码的原子性(也就是说同一时间只能有一个线程去执行临界区的代码)，**临界区内的代码对外是不可分割的，不会被线程切换所打断。**

- 如果把 `synchronized(obj) `放在` for `循环的外面，如何理解？-- 原子性

  `i++`对应的字节码指令有4条指令，如果把锁放在循环里面，那么也就是保证`i++`对应的四条字节码指令具有原子性，不可分割型，必须一次执行完成，但是如果把锁放在循环的外面，一共有4*5000条指令，说明这些指令的执行不可分割。两种加锁的方式最终结果是一样的。

- 如果 `t1.synchronized(obj1)` 而 `t2.synchronized(obj2)` 会怎样运作？-- 锁对象

  如果想要保护临界资源，就要使多个线程锁住的是同一个对象。如果不同的线程锁的是不同的对象，那么就相当于给不同的房间加锁，失去了锁的意义。

- 如果 `t1.synchronized(obj) `而 `t2` 没有加会怎么样？如何理解？-- 锁对象

  不可以，如果`t2`没有对临界区进行加锁，那么`t2`线程在进行上下文切换的时候，也就不会去进行获取对象锁，自然还是可以执行临界区的代码，不能保证临界区资源的原子性。

#### 面向对象改进

~~~ java
public class Test14 {

    public static void main(String[] args) throws InterruptedException {
        Room room=new Room();
        Thread t1=new Thread(()->{
           room.increase();
        });

        Thread t2=new Thread(()->{
            room.deincrease();
        });
        t1.start();
        t2.start();
//        t1.join();
//        t2.join();
        System.out.println(room.getCount());
    }
}


//使用面向对象思想对上面的代码进行改进
class Room{
    private int count=0;
    public void increase(){
//        this表示锁主的是当前的对象
        synchronized (this){
            count++;
        }
    }
    public void deincrease(){
        synchronized (this){
            count--;
        }
    }
//    获取count值的方法
    public int getCount(){
//        要保证对象出去临界区资源后才可以获取值，所以也需要加锁
        synchronized (this){
            return count;
        }
    }
}
~~~

###  方法上的 synchronized

> `synchronized`锁只可以**锁对象**

~~~ java
class Test{
    //把关键字添加在成员方法上，也相当于给当前的对象添加锁，也就是this对象加锁
    public synchronized void test() {

    }
}
//等价于
class Test{
    public void test() {
        synchronized(this) {

        }
    }
}
~~~

**静态方法加锁**

~~~ java
class Test{
    public synchronized static void test() {
    }
}
//等价于
class Test{
    public static void test() {
        //静态方法加锁相当于锁主的是class对象，即类对象
        synchronized(Test.class) {

        }
    }
}
//静态方法加锁，因为静态方法是属于类的，所以相当于给类的class对象加锁
~~~

#### 不加 `synchronized `的方法

不加 `synchronzied` 的方法就好比不遵守规则的人，不去老实排队（好比翻窗户进去的）

#### 线程八锁

**如果线程锁主的是同一个对象，那么会有互斥的一种效果，但是如果锁住的是不同的对象，那么此时线程之间可能会并行或者并发执行。线程八锁实际上就是判断锁住的是否是同一个对象。**

**题目一**

~~~ java
public class Test15 {
    public static void main(String[] args) {
        Number number=new Number();
        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin");
            number.test01();
        },"t1").start();
        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin.......");
            number.test02();
        },"t2").start();


    }
}
class Number{
  //说明锁住的是this对象,因为是普通的方法
    public synchronized void test01(){
        System.out.println("1.........");
    }
    public synchronized void test02(){
        System.out.println("2..........");
    }
}
//输出结果，也有可能是线程t2先执行，然后t1执行，因为锁住的是同一个对象
t1begin
t2begin.......
1.........
2..........
~~~

**题目二**

~~~ java
public class Test15 {
    public static void main(String[] args) {
        Number n1=new Number();//同一把锁对象
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"begin");
                n1.test01();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin.......");
            n1.test02();
        },"t2").start();


    }
}
//锁都是添加在普通的方法上，相当于锁住的是this对象，哪一个线程抢到锁，那个线程先执行
class Number{
    public synchronized void test01() throws InterruptedException {
        Thread.sleep(2);//在这里先休眠2秒
        System.out.println("1.........");
    }
    public synchronized void test02(){
        System.out.println("2..........");
    }
}
//输出情况：
//可能是t1线程先休眠2秒，然后t2线程在打印结果
//也可能是t2线程先打印结果，然后t1线程休眠两秒
~~~

**题目三**

~~~ java
//添加一个没有加锁的方法
public class Test15 {
    public static void main(String[] args) {
        Number n1=new Number();//同一把锁
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"begin");
                n1.test01();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin.......");
            n1.test02();
        },"t2").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin.......");
            n1.test03();
        },"t3").start();


    }
}
//没有添加锁的方法可以和加锁的两个方法并发执行
class Number{
    public synchronized void test01() throws InterruptedException {
        Thread.sleep(2);
        System.out.println("1.........");
    }
    public synchronized void test02(){
        System.out.println("2..........");
    }
    public  void test03(){
        System.out.println("3..........");
    }
}
//此时打印结果可能有3中
3，2s后，1，2
3，2，2s后，1
2，3，2s后，1
//应为t3线程没有加锁，所以和t1,t2线程完全可以并发进行，不需要保证互斥的访问
~~~

**题目四**

~~~ java
public class Test15 {
    public static void main(String[] args) {
      //锁住的是不同的对象，所以线程之间不会互斥进行访问，线程之间可以并行执行
        Number n1=new Number();
        Number n2=new Number();
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"begin");
                n1.test01();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin.......");
            n2.test02();
        },"t2").start();
    }
}
class Number{
  //锁住的是this对象
    public synchronized void test01() throws InterruptedException {
        Thread.sleep(2);
        System.out.println("1.........");
    }
  //锁住的是this对象
    public synchronized void test02(){
        System.out.println("2..........");
    }
}
//输出结果，一定是先输出2，在输出1，
t1begin
t2begin.......
2..........
1.........
//上面的两个线程锁住的是不同的对象，没有互斥的效果，两个线程是并行执行
~~~

**题目五**

~~~ java
public class Test15 {
    public static void main(String[] args) {
      //两把不同的锁对象
        Number n1=new Number();
        Number n2=new Number();
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"begin");
                n1.test01();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin.......");
            n2.test02();
        },"t2").start();
    }
}
class Number{
  //锁住的是类对象，在这里方法是静态的，所以锁住的是class类对象
    public static synchronized void test01() throws InterruptedException {
        Thread.sleep(2);
        System.out.println("1.........");
    }
    public synchronized void test02(){
        System.out.println("2..........");
    }
}
//输出结果
t1begin
t2begin.......
2..........
1.........
//因为锁住的是不同的对象，所以没有互斥的关系，先输出2，后输出1
~~~

**题目六**

~~~ java
public class Test15 {
    public static void main(String[] args) {
      //注意这里只有一个对象，和第八题分开
        Number n1=new Number();
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"begin");
                n1.test01();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin.......");
            n1.test02();
        },"t2").start();
    }
}
class Number{
  //给类对象加锁
    public static synchronized void test01() throws InterruptedException {
        Thread.sleep(2);
        System.out.println("1.........");
    }
  //类对象加锁
    public static synchronized void test02(){
        System.out.println("2..........");
    }
}
//两个锁都是锁类对象，因为内存中只有一份类对象，所以两个线程之间有互斥的关系
//打印结果
2，2s后，1
2s后，1，2
//主要看调度器先调度哪一个线程
~~~

**题目七**

~~~ java
public class Test15 {
    public static void main(String[] args) {
      //两把对象锁
        Number n1=new Number();
        Number n2=new Number();
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"begin");
                n1.test01();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin.......");
            n2.test02();
        },"t2").start();
    }
}
class Number{
  //锁住的是类对象
    public static synchronized void test01() throws InterruptedException {
        Thread.sleep(2);
        System.out.println("1.........");
    }
  //锁住的是this对象
    public  synchronized void test02(){
        System.out.println("2..........");
    }
}
//锁住的是不同的对象，t1锁住的是类对象，t2锁住的是n2对象，所以输出结果是：2，1
~~~

**题目八**

~~~ java
public class Test15 {
    public static void main(String[] args) {
      //注意这里是2个对象，要和第六题分开
        Number n1=new Number();
        Number n2=new Number();
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"begin");
                n1.test01();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"begin.......");
            n2.test02();
        },"t2").start();
    }
}
class Number{
    public static synchronized void test01() throws InterruptedException {
        Thread.sleep(2);
        System.out.println("1.........");
    }
    public  static synchronized void test02(){
        System.out.println("2..........");
    }
}
//两个方法都是类方法，所以两个线程锁住的都是一个类对象，所以有互斥的关系
2，2s后，1
2s后，1，2
//主要看调度器先调度哪一个线程
~~~

### 变量的线程安全分析

#### 成员变量和静态变量是否线程安全？

- 如果它们没有共享，则线程安全
- 如果它们被共享了，根据它们的状态是否能够改变，又分两种情况
  - 如果只有读操作，则线程安全
  - 如果有读写操作，则这段代码是临界区，需要考虑线程安全

#### 局部变量是否线程安全

- 局部变量是线程安全的
- 但局部变量**引用的对象**则未必（涉及jvm中的逃逸分析）
  - 如果该对象没有逃离方法的作用访问，它是线程安全的
  - 如果该对象逃离方法的作用范围，需要考虑线程安全

#### 局部变量线程安全分析

**代码说明**

~~~ java
public static void test1() {
 int i = 10;//i是定义在方法内部的局部变量
 i++;
}
~~~

每个线程调用` test1() `方法时局部变量 i，会在每个线程的栈帧内存中被创建多份，因此不存在共享

**反编译结果分析**

~~~ java
//下面是test1()方法反编译后的结果，也就是方法的描述信息
flags: ACC_PUBLIC, ACC_STATIC
 Code:
 stack=1, locals=1, args_size=0
   0: bipush 10 //10做入操作数栈操作，操作数栈是栈帧中对应的操作数栈
   2: istore_0 //吧10存储到局部变量表中，局部变量表也在栈帧中
   3: iinc 0, 1 //取出局部变量表0位置处的值做自增操作
   6: return
 LineNumberTable:
   line 10: 0
   line 11: 3
   line 12: 6
 LocalVariableTable:
 Start Length Slot Name Signature
 3 4 0 i I
~~~

**图解**

![1608345276074](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1608345276074.png)

- 为什么对于非引用的局部变量没有线程安全问题呢？

  对于每一个`jvm`实例，里面的`java`虚拟机栈，程序计数器，本地方法栈，这三个内存结构都是线程私有的，也就是每一个线程对应一份，线程在执行代码的时候，会拷贝指令代码到自己的私有内存区域进行计算操作，各个线程相互独立，所以不会存在线程安全的问题。

**局部变量的引用**

**代码说明**

~~~ java
public class Test16 {
    public static final int THREAD_NUMBER=2;
    public static final int LOOP_NUM=200;
    public static void main(String[] args) {
        ThreadUnSafe threadUnSafe=new ThreadUnSafe();
        for(int i=0;i<THREAD_NUMBER;i++){
            new Thread(()->{
                threadUnSafe.method01(LOOP_NUM);
            },"thread"+(i+1)).start();
        }

    }
}
class ThreadUnSafe{
//    共享资源
    ArrayList <String>list=new ArrayList();

    public void method01(int loopNum){
        for(int i=0;i<loopNum;i++){
            method02();
            method03();
        }
    }
    private void method02(){
        list.add("1");
    }
    private void method03(){
        list.remove(0);
    }
}
//输出
Exception in thread "thread1" java.lang.IndexOutOfBoundsException: Index: 1, Size: 1
	at java.util.ArrayList.rangeCheck(ArrayList.java:659)
	at java.util.ArrayList.remove(ArrayList.java:498)
	at rzf.qq.com.MyThread.ThreadUnSafe.method03(Test16.java:32)
	at rzf.qq.com.MyThread.ThreadUnSafe.method01(Test16.java:25)
	at rzf.qq.com.MyThread.Test16.lambda$main$0(Test16.java:12)
	at java.lang.Thread.run(Thread.java:748)
~~~

- 分析
  - 无论哪个线程中的 `method2 `引用的都是同一个对象中的` list `成员变量
  - `method3` 与 `method2` 分析相同

**图解**

![1608346310807](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/19/105152-266155.png)

- 因为两个线程同时访问共享变量，如果对`list`列表的删除和添加元素不是互斥的话，可能线程1去添加删除一个元素，此时线程2又去删除一个元素，会产生数组越界。

**修改为局部变量**

**代码说明**

~~~ java
public class Test16 {
    public static final int THREAD_NUMBER=2;
    public static final int LOOP_NUM=200;

    public static void main(String[] args) {
        ThreadSafe threadSafe=new ThreadSafe();
        for(int i=0;i<THREAD_NUMBER;i++){
            new Thread(()->{
                threadSafe.method01(LOOP_NUM);
            },"thread"+(i+1)).start();
        }

    }
}
class ThreadSafe{
  //注意这里的final修饰
    public final void method01(int loopNum){
//        修改为局部变量
        ArrayList <String>list=new ArrayList<String>();
        for(int i=0;i<loopNum;i++){
            method02(list);
            method03(list);
        }
    }
  //注意下面两个方法是私有的
  //因为方法是私有的,这就可以保证其他地方不能调用下面的两个方法,所以传进来的参数不会暴漏给其他的方法
    private void method02(ArrayList <String>list){
        list.add("1");
    }
    private void method03(ArrayList <String>list){
        list.remove(0);
    }
}
//此时程序正常运行
~~~

- 分析
  - `list` 是局部变量，每个线程调用时会创建其不同实例，没有共享
  - 而 `method2 `的参数是从` method1 `中传递过来的，与` method1 `中引用同一个对象,并且在这里`method02`方法也是私有的方法,其他的方法不能调用`method02`方法,自然参数也就不会暴漏给其他的方法.
  - `method3 `的参数分析与` method2 `相同

**图示分析**

![1608350460983](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/19/120103-891245.png)

- 每一个线程内部都有自己的私有`list`对象,因此不会相互影响,所以不会造成线程安全问题.

**访问修饰符问题**

~~~ java
public class Test16 {
    public static final int THREAD_NUMBER=2;
    public static final int LOOP_NUM=200;

    public static void main(String[] args) {
        ThreadSafe threadSafe=new ThreadSafe();
        for(int i=0;i<THREAD_NUMBER;i++){
            new Thread(()->{
                threadSafe.method01(LOOP_NUM);
            },"thread"+(i+1)).start();
        }

    }
}
class ThreadSafe{
  //注意这里的final修饰
    public final void method01(int loopNum){
//        修改为局部变量
        ArrayList <String>list=new ArrayList<String>();
        for(int i=0;i<loopNum;i++){
            method02(list);
            method03(list);
        }
    }
 //注意方法的访问修饰是public
    public void method02(ArrayList <String>list){
        list.add("1");
    }
    public void method03(ArrayList <String>list){
        list.remove(0);
    }
}
//此时程序正常运行
~~~

- 也就是访问修饰符不会影响程序的并发安全问题.因为我们传入参数的时候,实际上传入的是对象的地址,访问的是同一个对象.

**子类继承父类线程是否安全**

**代码说明**

~~~ java
public class Test16 {
    public static final int THREAD_NUMBER=2;
    public static final int LOOP_NUM=200;

    public static void main(String[] args) {
        SubClass threadSafe=new SubClass();
        for(int i=0;i<THREAD_NUMBER;i++){
            new Thread(()->{
                threadSafe.method01(LOOP_NUM);
            },"thread"+(i+1)).start();
        }

    }
}

class ThreadSafe{

    public final void method01(int loopNum){
//        修改为局部变量
        ArrayList <String>list=new ArrayList<String>();
        for(int i=0;i<loopNum;i++){
            method02(list);
            method03(list);
        }
    }
  //注意下面两个方法的访问类型都是public ,也就是可以被子类重写
    public void method02(ArrayList <String>list){
        list.add("1");
    }
    public void method03(ArrayList <String>list){
        list.remove(0);
    }
}

class SubClass extends ThreadSafe{
//    对父类添加的方法进行重写，对共享资源进行操作
    public void method03(ArrayList <String>list){
        new Thread(()->{
            list.remove(0);
        }).start();

    }
}
~~~

- 此时不是线程安全的,因为在子线程中也对`list`进行访问操作,相当于访问了共享的资源,
- 但是如果吧`method02`和`method03`方法访问修饰修改为私有的,那么是线程安全的,也就是在一定程度上,如果吧方法的访问修饰改为私有的,那么子类就不可以重写私有方法,所以子类的覆盖的方法是另外一个方法.
- 另外在一些公共方法前面最好添加`final`关键字,也是防止子类对方法进行修改操作.

**小结**

- 把方法变为私有,或者添加`final`关键字,一定程度上可以保证线程安全问题.

> 从这个例子可以看出 private 或 final 提供【安全】的意义所在，请体会开闭原则中的【闭】

#### 常见的线程安全类

- `String`字符串是线程安全的.内部状态不可以改变保证线程安全
- `Integer`:包装类都是线程安全的，内部状态不可以改变保证线程安全
- `StringBuffer`:字符串拼接的线程安全类，使用synchronized关键字保证线程安全
- `Random`产生随机数的线程安全类
- `Vector`线程安全的`list`实现
- `Hashtable`线程安全的`map`实现
- `java.util.concurrent` 包下的类

**这里说它们是线程安全的是指，多个线程调用它们同一个实例的某个方法时，是线程安全的。也可以理解为:因为每一个实例的方法前面都有`synchronized`关键字**

~~~ java
 Hashtable table = new Hashtable();
    new Thread(()->{
        table.put("key", "value1");
    }).start();
    new Thread(()->{
        table.put("key", "value2");
    }).start();
~~~

- 它们的每个方法是原子的,也就是对于每一个方法,已经添加`synchronized`关键字,是原子操作.线程的上下文切换不会导致并发安全问题。
- 但注意它们多个方法的组合不是原子的，见后面分析

##### 线程安全类方法的组合

分析下面代码是否线程安全？

~~~ java
Hashtable table = new Hashtable();
// 线程1，线程2
if( table.get("key") == null) {
 table.put("key", value);
}
//不是线程安全的
~~~

**图解**

![1608355224927](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/19/132027-280933.png)

##### 不可变类线程安全性

String、Integer 等都是不可变类，因为其内部的状态不可以改变，因此它们的方法都是线程安全的
有同学或许有疑问，String 有 replace，substring 等方法【可以】改变值啊，那么这些方法又是如何保证线程安全的呢？

**代码说明**

~~~ java
public String substring(int beginIndex, int endIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        if (endIndex > value.length) {
            throw new StringIndexOutOfBoundsException(endIndex);
        }
        int subLen = endIndex - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
    }
~~~

- 我们以`substring`源码来说明,其实最后使用`new String(value, beginIndex, subLen);`返回的是一个新的字符串,并不会影响字符串的本身.所以是线程安全的.
- 在源码中把`Sring`设置为`final`类，目的就是遵循了设计模式中的开闭原则，防止继承`String`进而修改类源码.比如有些子类继承与String类，然后对其中的一些方法进行重写，覆盖原来的方法，这样就不能保证线程安全。
- 另外，如果变量使用final声明的话，仅仅能保证变量的引用不在改变，而不能保证对象本身内部的属性不能改变，所以使用final声明的对象仍然不能保证线程安全。

### 练习

#### 卖票问题.

测试下面代码是否存在线程安全问题，并尝试改正

~~~ java
import java.util.List;
import java.util.Random;
import java.util.Vector;

public class Exercise01 {
    public static void main(String[] args) throws InterruptedException {
        TicketWindow t=new TicketWindow(1000);

        List<Integer>list=new Vector<>();
//        所有的线程集合
        List<Thread>threadList=new Vector<>();

//        模拟同时有多少个人来买票
        for(int i=0;i<3000;i++){
            Thread thread=new Thread(()->{
//                随机数表示买票的多少
                int count=t.sell(randomAccount());
                try {
                    Thread.sleep(randomAccount());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
//                add方法也是线程安全的
                list.add(count);
            });
//            把所有线程添加到集合中，本来就是线程安全的方法
            threadList.add(thread);
            thread.start();
        }
        
//        在主线程中要等待所有子线程运行完毕
        for (Thread thread : threadList) {
            thread.join();
        }
        
        
//        下面是主线程的代码，但是主线程要等待上面的所有子线程运行结束才可以统计
//        统计余票的数量
        System.out.println("剩余的票数："+t.getCount());
//        输出卖出的票数
        System.out.println("卖出的票数："+list.stream().mapToInt(i->i).sum());
    }

    //    random为线程安全的类
    static Random random=new Random();
//    随机1-5
    public static int randomAccount(){
        return random.nextInt(5)+1;
    }
}



//售票窗口
class TicketWindow {

  //共享变量
    private int count;
    public TicketWindow(int count) {
        this.count = count;
    }
//    获取与，余票的数量
    public int getCount() {
        return count;
    }
//    对count进行读写，属于临界区，所以对此方法加锁，也就是对this对象加锁
//    下面对共享变量的访问是原子操作
    public synchronized int sell(int amount) {
        if (this.count >= amount) {
            this.count -= amount;
            return amount;
        } else {
            return 0;
        }
    }
}
~~~

#### 转账

多个需要保护的对象

~~~ java
public class ExerciseTransfer {
 public static void main(String[] args) throws InterruptedException {
   Account a = new Account(1000);
   Account b = new Account(1000);
   Thread t1 = new Thread(() -> {
   for (int i = 0; i < 1000; i++) {
   a.transfer(b, randomAmount());
   }
   }, "t1");
   Thread t2 = new Thread(() -> {
   for (int i = 0; i < 1000; i++) {
   b.transfer(a, randomAmount());
   }
   }, "t2");
   t1.start();
   t2.start();
   t1.join();
   t2.join();
   // 查看转账2000次后的总金额
   log.debug("total:{}",(a.getMoney() + b.getMoney()));
 }
 // Random 为线程安全
 static Random random = new Random();
 // 随机 1~100
 public static int randomAmount() {
 	return random.nextInt(100) +1;
 	}
}
class Account {
 private int money;
 public Account(int money) {
 		this.money = money;
 }
 public int getMoney() {
	 return money;
 }
 public void setMoney(int money) {
 		this.money = money;
 }
 public void transfer(Account target, int amount) {
     if (this.money > amount) {
     this.setMoney(this.getMoney() - amount);
     target.setMoney(target.getMoney() + amount);
 		}
	 }
}
//如果把锁加在this对象上面，那么锁住的只能是某一个对象的money变量，这里要锁住的是两个对象共享的变量，所以必须把锁添加在类上，如果没理解，请看线程8锁那一块
//也就是说如果加所加载方法上面，只能锁住this对象，不能锁住target对象
public synchronized void transfer(Account target, int amount) {
     if (this.money > amount) {
     this.setMoney(this.getMoney() - amount);
     target.setMoney(target.getMoney() + amount);
 	}
}
//要锁主类对象，也就是说this对象的money和target对象的money属性都需要被保护
public void transfer(Account target, int amount) {
  //在这里锁住的是类而第一种加锁的方式只能锁住this对象
  synchronized(Account.class){
     if (this.money > amount) {
     this.setMoney(this.getMoney() - amount);
     target.setMoney(target.getMoney() + amount);
 		}
  }
}

~~~

> 如果有多个需要保护的变量，需要锁住的是类对象

### Monitor的概念

**Java 对象头**
以 32 位虚拟机为例

integer:12字节=8+4

int:4字节

**普通对象**

![1608426353550](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/090554-816407.png)

其中`klass word1`存储的是对象的类型信息，是一个指针，指向的是方法区的类型信息。

**数组对象**

![1608426424780](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/090706-665405.png)

组的对象头还要额外添加数组的长度信息

**其中 Mark Word 结构为**

![1608426453307](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/27/163241-658799.png)

`hashcode`:哈希码，`age`:垃圾回收看的分代的年龄，`biased_lock`:偏向锁,01:加锁的状态，前面的30位代表的是锁对象的地址。`ptr_to_lock_record`表示锁的地址，后两位表示是那种锁。

**64为虚拟机的Mark Word**

![1608426690728](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/091132-375421.png)

#### Monitor锁

如下面的图所示，当开始执行synchronized代码块的时候，会将obj对象和操作系统的monitor对象进行关联，然后obj对象中的前30位记录monitor对象的地址，后两位修改为10，表示重量级锁，

![1620105087062](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/04/131128-459069.png)

- Monitor 被翻译为**监视器或管程**
- 每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的Mark Word 中就被设置指向 Monitor 对象的指针
- **Monitor 结构如下**

![1608427262276](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/092104-130949.png)

- 刚开始Monitor中的Owner为null.
- 当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2，Monitor中只能有一
  个 Owner，其中monitor是操作系统层面的对象，而obj是java层面的对象。
- 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入
  `EntryList BLOCKED`,**此队列中的线程是没有获取到锁的线程。**
- Thread-2 执行完同步代码块的内容，然后唤醒 `EntryList `中等待的线程来竞争锁，竞争的时是非公平的
- 图中 `WaitSet `中的 `Thread-0，Thread-1` 是之前获得过锁，但条件不满足进入` WAITING `状态的线程，后面讲`wait-notify `时会分析

**注意**

> `synchronized`必须是进入同一个对象的`monitor`才会有效，如果进入不同对象的`monitor`，那么无效。因为一个对象会关联一个`monitor`管程。
>
> 不添加`synchronized`锁的对象不会关联`monitor`对象，也不遵循以上的规则。

#### synchronized原理

**字节码角度理解锁**

**源码说明**

~~~ java
public class Test17 {
    static final Object lock = new Object();
    static int count = 0;

    public Test17() {
    }

    public static void main(String[] args) {
        synchronized(lock) {
            ++count;
        }
    }
}

~~~

**字节码文件**

~~~ java
public static void main(java.lang.String[]);
    descriptor: ([吗  Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: getstatic     #2                  // Field lock:Ljava/lang/Object;也就是拿到lock锁的引用对象，我们可以看到实际是object对象，synchronized的开始，拿到的是引用的地址
         3: dup //复制锁的指令
         4: astore_1//把复制的锁存储到局部变量表为1的slot槽位置，是为了以后解锁使用
         5: monitorenter //synchronized对应的指令，其实就是将锁对象的markw word设置为monitor指针，将lock与操作系统的monitor进行关联
         6: getstatic     #3                  // Field count:I //代表取出count
         9: iconst_1//准备常数1
        10: iadd//做累加操作
        11: putstatic     #3                  // Field count:I.重新写回count操作
        14: aload_1//加载局部变量表中1位置的临时锁对象的地址
        15: monitorexit//将对象头的mark word进行重置，加锁前mark word存储的是hash code等信息，但是加锁后存储的是monotor指针，所以要重置为加锁之前的信息，然后唤醒entry list中的一个阻塞线程
                  只是把monitor和obj对象头中的信息交换了一下，并没有丢失。
        16: goto          24//执行第24条指令，也就是返回
        
                  
        //如果锁中发生异常，就执行下面的代码，也可以正常释放锁
        19: astore_2//异常对象存储到局部变量表2的位置
        20: aload_1//重新加载锁的引用
        21: monitorexit//将对象头的mark word进行重置，加锁前mark word存储的是hash code等信息，但是加锁后存储的是monotor指针，所以要重置为加锁之前的信息，然后唤醒entry list中的一个阻塞线程
        22: aload_2//加载异常对象
        23: athrow//抛出异常
        24: return
      Exception table://异常表检测范围是6-16如果出现异常，就到19行去执行
                  //检测范围从19-22，如果出现异常，就到19行去处理
         from    to  target type
             6    16    19   any
            19    22    19   any
      LineNumberTable:
        line 8: 0
        line 9: 6
        line 10: 14
        line 12: 24
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      25     0  args   [Ljava/lang/String;
~~~

- 所以从字节码角度，不管我们加锁的代码块是否可以正常运行，底层都可以正常加锁和释放锁，不会出现加锁然后无法释放的情况。
- 总结来说，就是synchronized锁在底层真正使用的是monitor进行加锁，一个锁对象对应一个monitor对象。monitor锁是由操作系统进行提供的，成本很高，开销大。属于重量级锁。

#### synchronized原理进阶

重量级锁--monitor

轻量级锁--锁记录

故事角色

- 老王 - JVM
- 小南 - 线程
- 小女 - 线程
- 房间 - 对象
- 房间门上 - **防盗锁 - Monitor**
- 房间门上 - **小南书包 - 轻量级锁**
- 房间门上 - **刻上小南大名 - 偏向锁**，锁偏向于某一个线程使用
- 批量重刻名 - **一个类的偏向锁撤销到达 20 阈值**
- 不能刻名字 - **批量撤销该类对象的偏向锁，设置该类不可偏向**



小南要使用房间保证计算不被其它人干扰（原子性），最初，他用的是防盗锁，当上下文切换时，锁住门。这样，即使他离开了，别人也进不了门，他的工作就是安全的。

但是，很多情况下没人跟他来竞争房间的使用权。小女是要用房间，但使用的时间上是错开的，小南白天用，小女晚上用。每次上锁太麻烦了，有没有更简单的办法呢？

小南和小女商量了一下，约定不锁门了，而是谁用房间，谁把自己的书包挂在门口，但他们的书包样式都一样，因此每次进门前得翻翻书包，看课本是谁的，如果是自己的，那么就可以进门，这样省的上锁解锁了。万一书包不是自己的，那么就在门外等，并通知对方下次用锁门的方式。

后来，小女回老家了，很长一段时间都不会用这个房间。小南每次还是挂书包，翻书包，虽然比锁门省事了，但仍然觉得麻烦。于是，小南干脆在门上刻上了自己的名字：【小南专属房间，其它人勿用】，下次来用房间时，只要名字还在，那么说明没人打扰，还是可以安全地使用房间。如果这期间有其它人要用这个房间，那么由使用者将小南刻的名字擦掉，升级为挂书包的方式。

同学们都放假回老家了，小南就膨胀了，在 20 个房间刻上了自己的名字，想进哪个进哪个。后来他自己放假回老家了，这时小女回来了（她也要用这些房间），结果就是得一个个地擦掉小南刻的名字，升级为挂书包的方式。老王觉得这成本有点高，提出了一种批量重刻名的方法，他让小女不用挂书包了，可以直接在门上刻上自己的名字

后来，刻名的现象越来越频繁，老王受不了了：算了，这些房间都不能刻名了，只能挂书包

##### 轻量级锁

- **轻量级锁的使用场景：如果一个对象虽然有多线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化。**
- 轻量级锁对使用者是透明的，即语法仍然是` synchronized`，假设有两个方法同步块，利用同一个对象加锁
- 使用轻量级锁是jvm自动进行的，如果轻量级锁加锁失败了，jvm才会换为重量级锁。

**代码说明**

~~~ java
public class Test17 {

    static final Object lock=new Object();

    public static void main(String[] args) {
        

    }
    public static void method01(){
        synchronized (lock){
            method02();
        }
    }

    public static void method02(){
        synchronized (lock){
//            同步块B

        }
    }
}
~~~

- 创建锁记录（`lock record`)对象，其实这个锁记录就可以认为是monitor对象，每一个线程的栈帧都会包含一个**锁记录的结构**，内部可以存储锁定对象的`Mark Word`(也就是对象头中的`mark word`结构)。锁记录包含两部分：**1，对象引用存储的是锁对象的地址，2，lock record记录的是对象头的mark word信息。**，现在可以看到对象头中的锁标记是01，表示没有加锁的状态。

![1608430712495](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/101834-900964.png)

- 让锁记录中的`object reference`指向刚才的锁对象， 并且尝试用`cas`替换掉`Object`中的`mark word`，并且将`mark work`的值存入锁记录。对象头中的01表示为没有加锁，添加到锁记录中就变为00，表示轻量级锁。在这里是将对象头中的`hashcode`等信息和`lock recode`信息做一个交换，因为解锁的时候，还需要进行恢复对象的状态。obj中01表示没有加锁状态，而锁记录中00表示轻量级锁状态。

![1608431916759](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/103837-670129.png)

- 如果`cas`替换成功（也就是对象的状态是01的时候，就是没加锁时候，如果其他线程已经把01修改为00，那么这个时候加锁就会失败），对象头中的锁记录地址和状态00，表示由该线程对对象加锁

![1608432158497](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/104240-205756.png)

- 如果`cas`失败，有两种情况
  - **如果是其他线程已经持有该对象的轻量级锁，这个时候表示有竞争，进入锁膨胀的过程。**
  - 如果是自己执行了`synchronized`重入（也就是自己的线程又给同一个对象添加锁），那么需要添加一条`lock record`作为重入的计数，就是重新创建一个栈帧，把栈帧的锁记录重新做上面的一系列操作，比如`cas`操作,交换对象头和锁记录中的信息。重入实际就是判断同一个线程对锁对象添加了几次锁，直接可以根据`lock record`的数量就可以计算。但是这样加锁会失败，因为第一次已经把obj对象头中的10修改为00,表示已经加锁了，但是会产生一个新的锁记录，锁记录的对象头中记录的是null值，最后在解锁的时候，解一次锁，就会去掉一个`lock record`记录。锁记录个数代表加锁的次数。

![1608432573493](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/104936-686289.png)

- 当退出`synchronized`的代码块（解锁）时候，如果有`lock record`值为`null`的情况，表示有锁重入，这个时候重置锁记录，表示重入次数减一。

![1608433111541](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/105834-665733.png)

- 当退出`synchronized`代码块锁记录的值不是`null`这种情况时候，这个时候使用`cas`将`mark word`的值恢复为对象头原来的初始值。cas是一个原子操作，不可以打断。
  - 回恢复成功，那么就解锁成功。
  - 失败，那么说明轻量级锁进行了锁膨胀，或者是升级为重量级锁，进入重量级锁的解锁流程。

##### 锁膨胀

如果在尝试加轻量级锁的过程中，`CAS`操作无法成功，这个时候有一种情况就是有其他线程为锁对象已经添加上轻量级锁（也就是说明有线程在竞争），**这个时候需要进行锁的膨胀，将轻量级锁升级为重量级锁。**

**代码说明**

~~~ java
public class Test17 {

    static final Object lock=new Object();

    public static void main(String[] args) {


    }
    public static void method01(){
        synchronized (lock){
           //同步代码块
        }
    }
}
~~~

- 当Thread-1进行添加轻量级锁的时候，Thread-0已经对该对象添加了轻量级锁，可以看到obj对象中锁状态已经变为00，

![1608433998815](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/111321-848617.png)

- 这个时候Thread-1添加轻量级锁失败，需要进入锁膨胀的过程，因为线程1没有申请锁成功，必须申请重量级锁进入阻塞队列等待，所以要升级。这个时候是对obj对象升级为重量级锁，而不是申请新的锁对象。
  - 首先为object对象申请一个重量级的monitor锁，让object指向重量级锁的地址。因为Thread-1没有拿到锁，就需要进入阻塞状态，但是轻量级锁没有阻塞这种状态，所以要升级为重量级锁。
  - 然后自己进入monitor的`EntryList BLOCK`阻塞队列中。其中锁的类型也修改为重量级锁标志10.

![1608434455417](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/112059-331227.png)

- 当Thread-0退出同步代码块进行解锁的时候，使用cas将mark word恢复给对象头，如果失败，那么会进入重量级锁的解锁流程，也就是按照obj对象中monitor对象的地址找到monitor对象，设置owner为null,同时唤醒entryList 中的block线程。
- 切记：obj中记录的是monitor对象地址，而monitor对象中记录的是obj对象中对象头的信息。

##### 自旋优化

在**重量级锁进行竞争**的时候，还可以使用**自旋来进行优化**，如果当前线程自旋成功（即这时候持锁的线程已经退出同步代码块，释放了锁），什么叫自旋，也就是让将要进入阻塞队列的线程先不要进入阻塞队列，让他在原地进行几次循环等待，等待娶她线程让出锁，然后在获取锁，这样做可以减小上下文切换带来的压力，但是如果有很多线程都进入自旋的状态，那么很吃cpu的性能。这个时候当前线程就可以避免被阻塞（阻塞会发生上下文切换）。

**自旋重试成功的情况**

| 线程1（`cpu` 1上）       | 对象的Mark             | 线程2（`cpu2`上）                    |
| ------------------------ | ---------------------- | ------------------------------------ |
| -                        | 10（表示重量级锁）     | -                                    |
| 访问同步块（获取monitor) | 10(重置锁)重量锁指针   | -                                    |
| 成功（加锁）             | 10（重置锁）重量锁指针 | -                                    |
| 执行同步块               | 10（重置锁）重量指针锁 | -                                    |
| 执行同步块               | 10（重置锁）重量指针锁 | 访问同步块，获取monitor              |
| 执行同步块               | 10（重置锁）重量指针锁 | 自旋重试，没有获取锁，在这里循环重试 |
| 执行完毕                 | 10（重置锁）重量锁指针 | 自旋重试                             |
| 成功（解锁）             | 01（无锁）             | 自旋重试                             |
| -                        | 10（重置锁）重量锁指针 | 成功（加锁）                         |
| -                        | 10（重置锁）重量锁指针 | 执行同步块                           |
| ……                       |                        |                                      |

- 自旋优化，适合是多核`cpu`的。

**自旋重试失败的情况**

| 线程1（`cpu` 1上）       | 对象的Mark             | 线程2（`cpu2`上）                    |
| ------------------------ | ---------------------- | ------------------------------------ |
| -                        | 10（表示重量级锁）     | -                                    |
| 访问同步块（获取monitor) | 10(重置锁)重量锁指针   | -                                    |
| 成功（加锁）             | 10（重置锁）重量锁指针 | -                                    |
| 执行同步块               | 10（重置锁）重量指针锁 | -                                    |
| 执行同步块               | 10（重置锁）重量指针锁 | 访问同步块，获取monitor              |
| 执行同步块               | 10（重置锁）重量指针锁 | 自旋重试，没有获取锁，在这里循环重试 |
| 执行完毕                 | 10（重置锁）重量锁指针 | 自旋重试                             |
| 成功（解锁）             | 01（无锁）             | 自旋重试                             |
| -                        | 10（重置锁）重量锁指针 | 自旋成功                             |
| -                        | 10（重置锁）重量锁指针 | 阻塞                                 |
| ……                       |                        |                                      |

- 在jdk6之后，自旋锁是自适应的，比如对象刚刚一次自旋操作成功过，那么会认为这次自旋成功的可能性会高，就多旋转几次，反之，就少旋转几次甚至不旋转。
- 自旋会占用cpu的时间，单核的cpu会浪费性能，但是多核的cpu会发挥其优势。
- java 7之后不能控制是否开启自旋功能。

##### 偏向锁

轻量级锁在没有竞争的时候（也就是说只有当前一个线程），每一次仍然需要进行cas操作，开销依然很大。

jdk6中引入偏向锁进行优化，只有第一次使用cas将线程id(可以理解为线程的名字)设置到对象头的mark word头，之后发现这个**线程的id**是自己的就表示没有竞争，不用重新进行cas操作，以后只要不发生竞争，这个对象就归该线程所有。线程id一般是唯一的，这样可以避免每一次进行cas操作。

**代码说明**

~~~ java
static final Object obj = new Object();
public static void m1() {
 synchronized( obj ) {
 // 同步块 A
 m2();
 }
}
public static void m2() {
 synchronized( obj ) {
 // 同步块 B
 m3();
 }
}
public static void m3() {
 synchronized( obj ) {
// 同步块 C
 }
}
~~~

**轻量级锁锁重入**

可以看到，在m1()方法中获取到锁之后，在m2()中，还会去重新尝试获取锁，使用的是cas操作，同理，在m3()方法中还会使用cas重新获取锁。这样每次重新获取锁，然后添加锁记录会影响开销。每次尝试获取锁，都会添加锁记录，然后尝试使用cas去修改锁对象信息，

![1608508817117](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/21/080019-12287.png)

**偏向锁**

使用线程的id号。也就是把线程的id添加到monitor中。

![1608508883101](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/21/080124-149134.png)

###### 偏向状态

**64位虚拟机对象头**

- 10：表示是重量级锁。
- 00：表示是轻量级锁。
- 01：正常状态，没有添加锁。
- biased_lock:表示是否启用偏向锁。

![1608426690728](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/20/091132-375421.png)

一个对象创建时

- 如果开启偏向锁（默认开启），那么对象创建后，markword值为0x05,，也就是最后三位是101，这时他的thread,epoch,age都是0.
- 偏向锁开启默认是延迟的，不会再程序启动时立即生效，如果想避免延迟，可以添加vm参数：XX：BiasedLockingStartupDelay=0来禁用延迟。
- 如果没有开启偏向锁，那么对象创建后，markword值为0x01,，也就是最后三位是001，这时他的hashcode，age,都是0，第一次使用hashcode的时候才会赋值。
- 当给锁对象的对象头markword添加线程id后，当锁退出后，markword中存储的还是第一次解锁线程的线程id，除非是其他线程重新使用此对象锁，markdown中的线程id才会改变。
- **偏向锁的使用场景是当只有一个线程的时候，也就是没有竞争的时候。**当冲突很少的时候适合于偏向锁。

> 加锁的顺序：偏向锁--->轻量级锁--->重量级锁

######  撤销对象锁的偏向状态- 调用对象的hashCode()方法

当一个可偏向的对象获取对象的哈希码之后，会禁用偏向锁，因为哈希码占用了存储线程id的位置，也就是如果调用某个锁的hashcode()方法，那么此对象的偏向锁会被撤销。而轻量级锁的哈希码存储在所记录的栈帧中，而重量级锁的哈希码存储在monitor中。而偏向锁没有额外的存储位置。

###### 撤销-其他线程使用对象

==轻量级锁和偏向锁使用的前提是两个线程在访问对象锁的时间都是错开的。而重量级锁多个线程可以并发执行。==

当某个线程给锁对象添加偏向锁后，然后解锁，当再次有其他线程给对象添加锁的时候，那么这次对象的偏向锁状态会失效，升级为轻量级锁，也就是对象从可偏向状态转变为不可偏向状态。

~~~ java
private static void test2() throws InterruptedException {
 Dog d = new Dog();
 Thread t1 = new Thread(() -> {
 synchronized (d) {
 log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 synchronized (TestBiased.class) {
 TestBiased.class.notify();
 }
 // 如果不用 wait/notify 使用 join 必须打开下面的注释
 // 因为：t1 线程不能结束，否则底层线程可能被 jvm 重用作为 t2 线程，底层线程 id 是一样的
 /*try {
 System.in.read();
 } catch (IOException e) {
 e.printStackTrace();
 }*/
 }, "t1");
 t1.start();
 Thread t2 = new Thread(() -> {
 synchronized (TestBiased.class) {
 try {
 TestBiased.class.wait();
 } catch (InterruptedException e) {
 e.printStackTrace();
 }
 }
 log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
 synchronized (d) {
 log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 log.debug(ClassLayout.parseInstance(d).toPrintableSimple(true));
 }, "t2");
 t2.start();
}
//输出
[t1] - 00000000 00000000 00000000 00000000 00011111 01000001 00010000 00000101 
[t2] - 00000000 00000000 00000000 00000000 00011111 01000001 00010000 00000101 
[t2] - 00000000 00000000 00000000 00000000 00011111 10110101 11110000 01000000 
[t2] - 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000001 
~~~

###### 撤销-调用wait/notify

wait/notify只有**重量级锁**才有，当调用这两个方法时候，会把偏向锁和轻量级锁升级为重量级锁。

###### 批量重偏向

如果对象被多个线程访问，但是没有竞争，这个时候偏向线程t1的对象仍然有机会偏向t2,重偏向会重新设置对象的id。

当撤销偏向锁阈值超过20次后，jvm会这样觉得，我是不是偏向错了呢。于是在给对象加锁时候重新偏向至加锁线程。

~~~ java
private static void test3() throws InterruptedException {
 Vector<Dog> list = new Vector<>();
 Thread t1 = new Thread(() -> {
 for (int i = 0; i < 30; i++) {
 Dog d = new Dog();
 list.add(d);
 synchronized (d) {
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 }
 synchronized (list) {
 list.notify();
 } 
 }, "t1");
 t1.start();
 
 Thread t2 = new Thread(() -> {
 synchronized (list) {
 try {
 list.wait();
 } catch (InterruptedException e) {
 e.printStackTrace();
 }
 }
 log.debug("===============> ");
 for (int i = 0; i < 30; i++) {
 Dog d = list.get(i);
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 synchronized (d) {
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 }, "t2");
 t2.start();
}

~~~

###### 批量撤销

当撤销偏向锁的阈值达到40次之后，jvm也会这样觉得，自己确实偏向错了，根本就不该偏向，于是整个类的所有对象都会变为不可偏向的，新建的对象也是不可偏向的。

~~~ java
static Thread t1,t2,t3;
private static void test4() throws InterruptedException {
 Vector<Dog> list = new Vector<>();
 int loopNumber = 39;
 t1 = new Thread(() -> {
 for (int i = 0; i < loopNumber; i++) {
 Dog d = new Dog();
 list.add(d);
 synchronized (d) {
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 }
 LockSupport.unpark(t2);
 }, "t1");
 t1.start();
 t2 = new Thread(() -> {
 LockSupport.park();
 log.debug("===============> ");
 for (int i = 0; i < loopNumber; i++) {
 Dog d = list.get(i);
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 synchronized (d) {
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 LockSupport.unpark(t3);
 }, "t2");
t2.start();
 t3 = new Thread(() -> {
 LockSupport.park();
 log.debug("===============> ");
 for (int i = 0; i < loopNumber; i++) {
 Dog d = list.get(i);
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 synchronized (d) {
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 log.debug(i + "\t" + ClassLayout.parseInstance(d).toPrintableSimple(true));
 }
 }, "t3");
 t3.start();
 t3.join();
 log.debug(ClassLayout.parseInstance(new Dog()).toPrintableSimple(true));
}
~~~

###### 锁消除

~~~ java
@Fork(1)
@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations=3)
@Measurement(iterations=5)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class MyBenchmark {
 static int x = 0;
 @Benchmark
 public void a() throws Exception {
 x++;
 }
 @Benchmark
 public void b() throws Exception {
 Object o = new Object();
 synchronized (o) {
 x++;
    }
 }
}

java -jar benchmarks.jar
Benchmark Mode Samples Score Score error Units
c.i.MyBenchmark.a avgt 5 1.542 0.056 ns/op 
c.i.MyBenchmark.b avgt 5 1.518 0.091 ns/op 

//-XX:-EliminateLocks表示关闭锁消除
java -XX:-EliminateLocks -jar benchmarks.jar
Benchmark Mode Samples Score Score error Units
c.i.MyBenchmark.a avgt 5 1.507 0.108 ns/op 
c.i.MyBenchmark.b avgt 5 16.976 1.572 ns/op 
~~~

#### 锁小结

创建一个对象最初是无锁状态的，一个对象获取锁后升级为偏向锁，当出现锁竞争的时候，就升级为轻量级锁，轻量级锁然后在升级为重量级锁，但是在轻量级锁升级为重量级锁的过程中，有一个自旋优化的过程，这样可以减小开销。

轻量级锁，在当前线程A创建一个锁记录，然后尝试通过CAS把markword更新为指向线程A的锁记录的指针，如果成功了，那么markword最后两位就变成00(轻量级锁)，如果此时又来了一个B线程，那么会在B线程中创建一个锁记录，尝试CAS把markword更新为指向线程A的该锁记录的指针，如果失败的话，会查看markword的指针指向的是不是B线程中的某个栈帧(锁记录)，如果是，即A和B是同一个线程，也就是当前操作是重入锁操作，即在当前线程对某个对象重复加锁，这是允许的，也就是可以获取到锁了。如果markword记录的不是B线程中的某个栈帧(锁记录)，那么线程B就会尝试自旋，如果自选超过一定次数，就会升级成重量级锁（轻量级锁升级成重量级锁的第一种时机：自选次数超过一定次数），如果B线程在自选的过程中，又来了一个线程C来竞争该锁，那么此时直接轻量级锁膨胀成重量级锁（轻量级锁升级成重量级锁的第二种时机：有两个以上的线程在竞争同一个锁。注：A,B,C3线程>2个线程）

如果一开始是无锁状态，那么第一个线程获取索取锁的时候，判断是不是无锁状态，如果是无锁(001),就通过CAS将mark word里的部分地址记录为当前线程的ID，同时最后倒数第三的标志位置为1，即倒数三位的结果是(101)，表示当前为轻量级锁。下一个如果该线程再次获取该锁的时候，就直接判断mark word里记录的线程ID是不是我当前的线程ID，如果是的话，就成功获取到锁了，即不需再进行CAS操作，这就是相对轻量级锁来说，偏向锁的优势（只需进行第一次的CAS，而无需每次都进行CAS，当然这个理想过程是没有其他线程来竞争该锁）。如果中途有其他线程来竞争该锁，发现已经是101状态，那么就会查看偏向锁记录的线程是否还存活，如果未存活，即偏向锁的撤消，将markword记录的锁状态从101(偏向锁)置未001(无锁)，然后重新偏向当前竞争成功的线程，如果当前线程还是存活状态，那么就升级成轻量级锁。

### Wait/Notify

#### wait/notify原理

![1608513672927](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/21/092114-83848.png)

- `Owner` 线程发现条件不满足，调用 `wait `方法，即可进入 `WaitSet` 变为 `WAITING` 状态(这个状态是)

  `BLOCKED` （处于block的线程是正在等待锁的线程）和 `WAITING`（已经获取到锁，但是又放弃锁，进入`waitset`队列，原因是线程所需的条件没有得到满足） 的线程都处于阻塞状态，不占用 `CPU `时间片

- `BLOCKED` 线程会在 `Owner `线程释放锁时唤醒（**处于block的线程是没有获取锁的线程**），正在等待锁的线程。

- `WAITING` 线程会在 `Owner` 线程调用 `notify `或 `notifyAll `时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入`EntryList `重新竞争,而`block`在`owner`释放锁后，会唤醒block队列中的一个线程获取锁。

- waitSet和EntryList中的线程都处于阻塞状态，不会占用cpu的时间。

- waitSet的线程是必须是处于owner中的线程才有资格进入waitSet队列，进入这个队列可以调用wait()方法，处于owner中的线程才可以调用notify()方法唤醒waitSet中的线程。

**转换关系**

![1620196533576](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/05/143535-848163.png)

当多个线程同时请求某个对象监视器时，对象监视器会设置几种状态用来区分请求的线程：

- Contention List：所有请求锁的线程将被首先放置到该竞争队列；
- Entry List：Contention List中那些有资格成为候选人的线程被移到Entry List；
- Wait Set：那些调用wait方法被阻塞的线程被放置到Wait Set；
- OnDeck：任何时刻最多只能有一个线程正在竞争锁，该线程称为OnDeck；
- Owner：获得锁的线程称为Owner；
- !Owner：释放锁的线程。 

新请求锁的线程将首先被加入到Conetention List中，当某个拥有锁的线程（Owner状态）调用unlock之后，如果发现 EntryList为空则从Contention List中移动线程到EntryList

> wait和notify对应的是monitor里面处于阻塞的线程。

#### API介绍

- `obj.wait() `让进入` object `监视器的线程到 `waitSet `等待
- `obj.notify()` 在` object `上正在 `waitSet `等待的线程中挑一个唤醒
- `obj.notifyAll()` 让`object `上正在 `waitSet `等待的线程全部唤醒

它们都是线程之间进行协作的手段，都属于 `Object `对象的方法。**必须获得此对象的锁,成为`owner`之后**，才能调用这几个方法

**代码测试**

~~~ java
public class Test18 {
    public static Object object=new Object();
    public static void main(String[] args) throws InterruptedException {
        synchronized (object){
//            object先获取锁之后，才可以调用wait进入waitSet队列等待
            object.wait();
        }
    }
}
~~~

**`API`使用**

~~~ java
public class Test18 {
    public final static Object object=new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(()->{
            synchronized (object){
                System.out.println(Thread.currentThread().getName()+"  开始执行代码......");
                try {
//                    t1线程被阻塞，进入阻塞队列中，也就是waitSet队列
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"  执行其他的代码........");
            }
        },"t1");
        t1.start();

        Thread t2=new Thread(()->{
            synchronized (object){
                System.out.println(Thread.currentThread().getName()+"  开始执行代码......");
                try {
//                    t2线程被阻塞，进入阻塞队列中，也就是waitSet队列
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"  执行其他的代码........");
            }
        },"t2");
        t2.start();

//        主线程在2秒后执行
        Thread.sleep(2);
        System.out.println(Thread.currentThread().getName()+"  唤醒object对象上其他的锁.......");
//        此时主线程获得锁
        synchronized (object){
//            object.notify();//唤醒阻塞队列中的一个线程
            object.notifyAll();
        }
    }
}
//
t1  开始执行代码......
t2  开始执行代码......
main  唤醒object对象上其他的锁.......
t2  执行其他的代码........
t1  执行其他的代码........
~~~

**带时限的wait()**

~~~ java
public class Test18 {
    public final static Object object=new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(()->{
            synchronized (object){
                System.out.println(Thread.currentThread().getName()+"  开始执行代码......");
                try {
//                    t1线程被阻塞，进入阻塞队列中，也就是waitSet队列
                    object.wait(2000);//等待两秒钟自动唤醒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"  执行其他的代码........");
            }
        },"t1");
        t1.start();
    }
}
~~~

- **`wait() `方法会释放对象的锁**，进入 `WaitSet` 等待区，从而让其他线程就机会获取对象的锁。无限制等待，直到`notify` 为止
- `wait(long n)` 有时限的等待, 到` n `毫秒后结束等待，或是被` notify`

### 正确使用wait、notify方法

#### sleep(long n) 和 wait(long n) 的区别

1. `sleep `是` Thread `静态方法，而 `wait `是 `Object `的方法 ，所有的对象都有wait()方法。
2. `sleep `不需要强制和 `synchronized `配合使用，但 `wait `需要和` synchronized `一起用 ，因为wait首先需要获取对象的锁，草可以使用。而sleep()什么时候都可以使用。
3. **`sleep` 在睡眠的同时，不会释放对象锁的，其他线程不能获取锁，但是会释放cpu的使用权，但 `wait `在等待的时候会释放对象锁**，也就是调用wait()方法会释放对象锁。所以使用wait()效率会更高。
4. 它们状态 TIMED_WAITING是一样的.也就是说都是有时限的等待。

> 锁的对象尽量添加`final`关键字，保证对象的引用尽量不会改变

**代码说明**

~~~ java
public class Test19 {
    private final static Object lock=new Object();
    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            synchronized (lock){
                try {
                    System.out.println("t1 线程获得锁");
//                    Thread.sleep(20000);不会释放锁
                    lock.wait();//会释放锁
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        },"t1").start();
//        现在主线程也尝试获取锁
        Thread.sleep(2);
        synchronized (lock){
            System.out.println("主线程获取到了锁");
        }
    }
}
//输出
t1 线程获得锁
主线程获取到了锁
~~~

#### **step01**

~~~ java
public class Test20 {

    static final Object room=new Object();
    static boolean hasCigarette=false;//代表是否有烟
    static boolean hasTakeout=false;
    public static void main(String[] args) throws InterruptedException {
//小南的线程是最先执行的
        new Thread(()->{
            synchronized (room){
                System.out.println(Thread.currentThread().getName()+"  有没有烟："+hasCigarette);
                if(!hasCigarette){
                    System.out.println(Thread.currentThread().getName()+"  没有烟，先休息会");
                    try {
                        Thread.sleep(2);//不会释放锁，效率比较低
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName()+"  有没有烟："+hasCigarette);
                if(hasCigarette){
                    System.out.println(Thread.currentThread().getName()+"  开始去干活");
                }
            }
        },"小南").start();

        for(int i=0;i<5;i++){
            new Thread(()->{
                synchronized (room){
                    System.out.println(Thread.currentThread().getName()+"  开始干活去");
                }

            },"其他人").start();
        }

//        1秒后开始送烟
        Thread.sleep(1);
        new Thread(()->{//送烟并不需要添加锁
          synchronized(object){//在这里加锁也不能解决问题
             System.out.println(Thread.currentThread().getName()+"  烟到了");
                hasCigarette=true;
          }
        },"送烟").start();

    }
}
小南  有没有烟：false
小南  没有烟，先休息会
送烟  烟到了
小南  有没有烟：true
小南  开始去干活
其他人  开始干活去
其他人  开始干活去
其他人  开始干活去
其他人  开始干活去
其他人  开始干活去
~~~

- 不足之处
  - 小南的线程必须睡足2秒，就算是烟提前送到，小南也不能干活。
  - 在小南没有烟期间，因为使用的是`sleep`,所以小南线程并没有释放锁，所以其他人也只能等待，小南线程结束之后其他人才可以干活，效率很低。
  - 加了 `synchronized (room)` 后，就好比小南在里面反锁了门睡觉，烟根本没法送进门，`main `没加
    `synchronized `就好像` main `线程是翻窗户进来的
  - 解决方法，使用 `wait - notify`(主要是在等待的时候回释放锁) 机制

#### **step02**

~~~ java
public class Test20 {

    static final Object room = new Object();
    static boolean hasCigarette = false;//代表是否有烟
    static boolean hasTakeout = false;

    public static void main(String[] args) throws InterruptedException {
//小南的线程是最先执行的
        new Thread(() -> {
            synchronized (room) {
                System.out.println(Thread.currentThread().getName() + "  有没有烟：" + hasCigarette);
                if (!hasCigarette) {
                    System.out.println(Thread.currentThread().getName() + "  没有烟，先休息会");
                    try {
//                        Thread.sleep(2);
//                        wait方法抛出的异常在其他的方法调用interrupt打断时候抛出
                        room.wait();//会释放锁
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "  有没有烟：" + hasCigarette);
                if (hasCigarette) {
                    System.out.println(Thread.currentThread().getName() + "  开始去干活");
                }
            }
        }, "小南").start();

        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                synchronized (room) {
                    System.out.println(Thread.currentThread().getName() + "  开始干活去");
                }

            }, "其他人").start();
        }

//        1秒后开始送烟
        Thread.sleep(1);
        new Thread(() -> {
            synchronized (room){
                System.out.println(Thread.currentThread().getName() + "  烟到了");
                hasCigarette = true;
//              烟送到后，然后叫醒小南线程
                room.notify();
            }
        }, "送烟").start();

    }
小南  有没有烟：false
小南  没有烟，先休息会
其他人  开始干活去
其他人  开始干活去
其他人  开始干活去
其他人  开始干活去
其他人  开始干活去
送烟  烟到了
小南  有没有烟：true
小南  开始去干活
~~~

- 使用`wait`最大的好处就是可以释放锁，小南等待的同事其他线程可以获取锁。
- 解决了其它干活的线程阻塞的问题
- 但如果有其它线程也在等待条件呢？

#### **step03**

~~~ java
public class Test20 {

    static final Object room = new Object();
    static boolean hasCigarette = false;//代表是否有烟
    static boolean hasTakeout = false;

    public static void main(String[] args) throws InterruptedException {
//小南的线程是最先执行的
        new Thread(() -> {
            synchronized (room) {
                System.out.println(Thread.currentThread().getName() + "  有没有烟：" + hasCigarette);
                if (!hasCigarette) {
                    System.out.println(Thread.currentThread().getName() + "  没有烟，先休息会");
                    try {
//                        Thread.sleep(2);
//                        wait方法抛出的异常在其他的方法调用interrupt打断时候抛出
                        room.wait();//会释放锁
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "  有没有烟：" + hasCigarette);
                if (hasCigarette) {
                    System.out.println(Thread.currentThread().getName() + "  开始去干活");
                }
            }
        }, "小南").start();

        new Thread(() -> {
            synchronized (room) {
                System.out.println(Thread.currentThread().getName() + "  有没有外卖：" + hasTakeout);
                if (!hasTakeout) {
                    System.out.println(Thread.currentThread().getName() + "  没有外卖，先休息会");
                    try {
//                        Thread.sleep(2);
//                        wait方法抛出的异常在其他的方法调用interrupt打断时候抛出
                        room.wait();//会释放锁
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "  有没有外卖：" + hasTakeout);
                if (hasTakeout) {
                    System.out.println(Thread.currentThread().getName() + "  开始去干活");
                }
            }
        }, "小女").start();

//        1秒后开始外卖
        Thread.sleep(1);
        new Thread(() -> {
            synchronized (room){
                System.out.println(Thread.currentThread().getName() + "  外卖到了");
                hasTakeout = true;
                room.notify();//随机叫醒的
              //room.notifyAll();可以解决问题，但是又把全部线程都唤醒
            }
        }, "送外卖").start();

    }
}
小南  有没有烟：false
小南  没有烟，先休息会
小女  有没有外卖：false
小女  没有外卖，先休息会
送外卖  外卖到了
小南  有没有烟：false
//虽然外卖送到了，但是唤醒的却是小南线程，唤醒线程错误
~~~

- `notify` 只能随机唤醒一个` entryList `(`blocking`阻塞队列)中的线程，这时如果有其它线程也在等待，那么就可能唤醒不了正确的线程，称之为【**虚假唤醒**】
- 解决方法，改为`notifyAll`

#### **step04**

~~~ java
public class Test20 {

    static final Object room = new Object();
    static boolean hasCigarette = false;//代表是否有烟
    static boolean hasTakeout = false;

    public static void main(String[] args) throws InterruptedException {
//小南的线程是最先执行的
        new Thread(() -> {
            synchronized (room) {
                System.out.println(Thread.currentThread().getName() + "  有没有烟：" + hasCigarette);
                while (!hasCigarette) {//这里使用循环
                    System.out.println(Thread.currentThread().getName() + "  没有烟，先休息会");
                    try {
//                        Thread.sleep(2);
//                        wait方法抛出的异常在其他的方法调用interrupt打断时候抛出
                        room.wait();//会释放锁
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "  有没有烟：" + hasCigarette);
                if (hasCigarette) {
                    System.out.println(Thread.currentThread().getName() + "  开始去干活");
                }
            }
        }, "小南").start();

        new Thread(() -> {
            synchronized (room) {
                System.out.println(Thread.currentThread().getName() + "  有没有外卖：" + hasTakeout);
                if (!hasTakeout) {
                    System.out.println(Thread.currentThread().getName() + "  没有外卖，先休息会");
                    try {
//                        Thread.sleep(2);
//                        wait方法抛出的异常在其他的方法调用interrupt打断时候抛出
                        room.wait();//会释放锁
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "  有没有外卖：" + hasTakeout);
                if (hasTakeout) {
                    System.out.println(Thread.currentThread().getName() + "  开始去干活");
                }
            }
        }, "小女").start();

//        1秒后开始外卖
        Thread.sleep(1);
        new Thread(() -> {
            synchronized (room){
                System.out.println(Thread.currentThread().getName() + "  外卖到了");
                hasTakeout = true;
//              烟送到后，然后叫醒小南线程
//                room.notify();
              //这里唤醒了所有的线程，但是烟没有送到，所以小南哪里西药使用一个循环去等待
                room.notifyAll();
            }
        }, "送外卖").start();

    }
}
小南  有没有烟：false
小南  没有烟，先休息会
小女  有没有外卖：false
小女  没有外卖，先休息会
送外卖  外卖到了
小女  有没有外卖：true
小女  开始去干活
小南  没有烟，先休息会

Process finished with exit code -1
~~~

- 用 `notifyAll` 仅解决某个线程的唤醒问题，但使用 `if + wait `判断仅有一次机会，一旦条件不成立，就没有重新判断的机会了
- 解决方法，用` while + wait`，当条件不成立，再次 `wait`

**小结**

> 这不是操作系统中学习的信号量机制吗。

**使用wait()和notify()的正确方法**

~~~ java
synchronized(lock) {
   while(条件不成立) {
   		lock.wait();
   }
 // 干活
}
//另一个线程
synchronized(lock) {
  //为什么需要使用while()循环，因为这里直接唤醒所有的线程，可能存在虚假唤醒问题，也就是某些线程条件不满足，但是被唤醒，所以使用循环让其等待
 	lock.notifyAll();
}
~~~

### 模式

#### 同步模式之保护性暂停

##### 定义

即 `Guarded Suspension`，**用在一个线程等待另一个线程的执行结果**，生产者消费者模式。
**要点**

- 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个` GuardedObject`
- 如果有结果不断从一个线程到另一个线程那么可以使用消息队列（见生产者/消费者）
- `JDK` 中，`join `的实现、`Future` 的实现，采用的就是此模式
- 因为要等待另一方的结果，因此归类到同步模式

![1608520209448](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/21/111012-849220.png)

**代码说明**

~~~ java
public class Test21 {
    public static void main(String[] args) {
      //这个对象就是一把锁，两个线程共用锁
        GuardedObject g=new GuardedObject();
//        线程t1 t2是同时执行的
        new Thread(()->{
            try {
//                线程1在等待获取结果
                System.out.println(Thread.currentThread().getName()+"  正在开始获取结果");
                Object o=g.get();
                System.out.println(Thread.currentThread().getName()+"  获取的结果是:  "+o.toString());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"  产生数据结果");
            g.complete("hrefgvdcxsvdcxs");
            System.out.println(Thread.currentThread().getName()+"  产生结果完毕");
        },"t2").start();

    }
}


class GuardedObject{
//    代表将来的结果
    private Object redponse;

    /**
     * 获取结果
     * @return 返回获取的结果
     */
    public Object get() throws InterruptedException {
//        第一步：先要拿到锁对象
        synchronized (this){
            while (redponse == null){//防止进行虚假唤醒
                this.wait();//会释放锁
//                当有其他线程唤醒时，就退出循环
            }
        }
        return redponse;
    }

    public void complete(Object o){
        synchronized (this){
//            给结果变量进行赋值
            this.redponse=o;
//            赋值完成后唤醒等待的线程，this就是一把锁对象
            this.notifyAll();
        }
    }
}
t1  正在开始获取结果
t2  产生数据结果
t2  产生结果完毕
t1  获取的结果是:  hrefgvdcxsvdcxs
~~~

- 使用`join`的局限性在于必须等待另一个线程执行结束，比如`t2`线程执行完毕后必须等待`t1`线程也执行完毕，但是使用保护性暂停模式的话，`t2`线程执行完成后可以做其他的事情，不必等待`t1`线程也执行结束
- 使用`join`的话，等待结果的变量必须设置为全局的，但是使用保护性暂停模式的话，可以设置为局部的。

**改进**

上面不带参数的`get`方法的话，如果没有产生结果，那么就会陷入死等状态，可以设置一个最大的时间。一旦超过这个时间的话，没有等到结果，就退出。

![1620207284318](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/05/173446-946854.png)

#### join原理

**源码解读**

~~~ java
public final void join() throws InterruptedException {
        join(0);//无参数的join实际上调用参数为0的有参数方法
    }

//有参数的方法
 public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();//记录开始时间
        long now = 0;//记录经历的时间

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {//如果时间等于0
            while (isAlive()) {//判断线程是否存活
                wait(0);
            }
        } else {
            while (isAlive()) {//保护性暂停就是指的这里，如果条件不满足，就一直循环等待
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;//求一次经历时间
            }
        }
    }
//wait()调用的是object的本地方法
public final native void wait(long timeout) throws InterruptedException;
~~~

- `join`源码就是用的**保护性暂停模式**
- 保护性暂停模式是一个线程等待另一个线程的结果，join是一个线程等待另一个线程的结束。

#### 扩展-多任务版 GuardedObject

图中 Futures 就好比居民楼一层的信箱（每个信箱有房间编号），左侧的 t0，t2，t4 就好比等待邮件的居民，右侧的 t1，t3，t5 就好比邮递员，如果需要在多个类之间使用 GuardedObject 对象，作为参数传递不是很方便，因此设计一个用来解耦的中间类，这样不仅能够解耦【结果等待者】和【结果生产者】，还能够同时支持多个任务的管理

![1608551009532](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/21/194332-364539.png)

**代码**

不好理解

~~~ java
public class Test21 {
    public static void main(String[] args) throws InterruptedException {
//        产生三个居民进行收信
        for(int i=0;i<3;i++){
            new People().start();
        }
//        一秒钟后开始送信
        Thread.sleep(1);
        for (Integer id : Mailboxes.getIds()) {
            new Postman(id,"id+abcde").start();
        }

    }
}


class People extends Thread{
    @Override
    public void run() {
        GuardedObject g=new GuardedObject();
        try {
            System.out.println("开始收信："+g.getId());
            Object o=g.get(5000);
            System.out.println("信的内容："+o.toString());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Postman extends Thread{
    private int id;
    private String mail;

    public Postman(int id, String mail) {
        this.id = id;
        this.mail = mail;
    }

    @Override
    public void run() {
//        送信的逻辑,根据id号码获取信件的内容
        GuardedObject g=Mailboxes.getGuardedObject(id);
        System.out.println("开始送信："+id+" "+mail);
        g.complete(mail);

    }
}

class Mailboxes{
    private static Map<Integer,GuardedObject> box=new Hashtable<>();
    private static int id;
//    产生唯一的id
//多个线程会访问此方法，，所以需要保证唯一性
//    锁关键字添加在static方法上，相当于给类添加了锁
    public static synchronized int generateId(){
        return id++;
    }

//    产生GuardedObject对象
    public static GuardedObject createGuardedObject(){
        GuardedObject go=new GuardedObject();
        box.put(go.getId(),go);//box类是线程安全的
        return go;
    }

//    获取键值得集合
    public static Set<Integer> getIds(){
        return box.keySet();//box类是线程安全的
    }

//    根据id获取信件的内容
    public static GuardedObject getGuardedObject(Integer id){
//        在这里使用remove()意思是获取到一封信件后，应该把信件删除
        return box.remove(id);
    }

}

class GuardedObject{
    private int id;
//    代表将来的结果
    private Object redponse;

    public void setId(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    /**
     * 带时限的等待
     * @param timeOut 等待的时间
     * @return 返回获取的结果
     * @throws InterruptedException
     */
    public Object get(long timeOut) throws InterruptedException {
//        第一步：先要拿到锁对象
        synchronized (this){
//            记录一下开始的时间
            long begin=System.currentTimeMillis();
//            经历的时间
            long passTime=0;
            while (redponse == null){//防止进行虚假唤醒
//                判断是否超时
                if(passTime > timeOut){
                    System.out.println("产生超时");
                    break;
                }
//                timeOut-passTime防止线程被虚假唤醒
                this.wait((timeOut-passTime));//会释放锁
//                当有其他线程唤醒时，就退出循环

//                获取经历时间
                passTime=System.currentTimeMillis()-begin;
            }
        }
        return redponse;
    }
    /**
     * 获取结果
     * @return 返回获取的结果
     */
    public Object get() throws InterruptedException {
//        第一步：先要拿到锁对象
        synchronized (this){
            while (redponse == null){//防止进行虚假唤醒
                this.wait();//会释放锁
//                当有其他线程唤醒时，就退出循环
            }
        }
        return redponse;
    }

    public void complete(Object o){
        synchronized (this){
//            给结果变量进行赋值
            this.redponse=o;
//            赋值完成后唤醒等待的线程
            this.notifyAll();
        }
    }
}
~~~

上面这种模式，邮递员和居民是一对一的关系，而生产者消费者模式可以一对多，也就是一个生产者可以对应多个消费者。

#### 生产者消费者模式-异步模式

**定义**

- 要点
  - 与前面的保护性暂停中的 GuardObject 不同，不需要产生结果和消费结果的线程一一对应
  - 消费队列可以用来平衡生产和消费的线程资源
  - 生产者仅负责产生结果数据，不关心数据该如何处理，而消费者专心处理结果数据
  - 消息队列是有容量限制的，满时不会再加入数据，空时不会再消耗数据
  - JDK 中各种阻塞队列，采用的就是这种模式，当存放消息的队列满了之后，生产消息的进程就会被阻塞。

![1620209870960](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/05/181755-404546.png)

之所以叫做异步模式就是生产者生产的消息不需要被立即的消费掉，但是前面的保护性暂停模式，生产者生产的消息会被立即的消费掉，是同步的关系。保护性暂停是一一对应的关系。共同点都是在多个线程之间传递消息。

**生产者消费者模式**

~~~ java
public class Test {

    public static void main(String[] args) throws InterruptedException {

        MessageQueue messageQueue = new MessageQueue(2);

//        创建生产者线程
        for (int i = 0; i < 3; i++) {
            final int id=i;
            new Thread(()->{
                try {
                    messageQueue.put(new Message(id," message "+id));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            },"producer"+id).start();
        }



//        创建一个消费者线程
        new Thread(()->{
//            消费者消费消息
            try {
                while (true){
                    Thread.sleep(2000);
                    Message message = messageQueue.take();
                    System.out.println(Thread.currentThread().getName()+" "+message);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"reducer").start();

    }
}


/**
 * 是java线程之间通信的类
 */
class MessageQueue{

//    创建集合存放消息
    private LinkedList<Message>queue=new LinkedList();

//    队列的容量
    private int capcity;

    public MessageQueue(int capcity) {
        this.capcity = capcity;
    }

    //    获取消息的方法
    public Message take() throws InterruptedException {
//        首先检查队列是否空
//        这里的队列相当于一把锁
      synchronized (queue){
          while (queue.isEmpty()){
//          如果队列是空，那么就进入entryList后等待，直到队列不空为止
              queue.wait();
          }
          Message message=queue.removeFirst();
//          通知生产者线程
          queue.notifyAll();
//          返回队列头部的元素
          return message;
      }
    }




//    存放消息的方法
    public void put(Message message) throws InterruptedException {

        synchronized (queue){
//            检查队列是否是满的
            while (queue.size() == capcity){
//                如果满，就进入阻塞队列进行等待
                queue.wait();
            }
//            将新的消息添加到队列的尾部
            queue.add(message);
//            唤醒等待消息的线程
            queue.notifyAll();
        }

    }


}

//只有get()方法，说明对象的内部不可改变，只有构造的时候可以初始化
final class Message{
    private int id;
    private Object value;

    @Override
    public String toString() {
        return "Message{" +
                "id=" + id +
                ", value=" + value +
                '}';
    }

    public Message(int id, Object value) {
        this.id = id;
        this.value = value;
    }

    public int getId() {
        return id;
    }

    public Object getValue() {
        return value;
    }
}
~~~

### Park()和UnPark()

#### 基本使用

他们是`LockSupport()`中的方法

~~~ java
//暂停当前的线程,那个线程调用park，就暂停哪一个方法
LockSupport.park()//也就是把此语句写在某个线程里面
//恢复某一个线程的运行
LockSupport.unpark(需要恢复的线程对象)
~~~

> park线程对应的状态是wait状态，也就是无时限的等待。
>
> unpark既可以在park之前调用，也可以在park之后调用，unpark()就是用来恢复被暂停线程的运行。

**案例**

~~~ java
public class LockSupportTest {

    public static void main(String[] args) throws InterruptedException {


        Thread t1=new Thread(()->{
            int i=0;
            while (true){
                try {
                    System.out.println(Thread.currentThread().getName()+" "+i++);
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if(i == 10){
                    LockSupport.park();
                }
            }

        });
        t1.start();

        int j=0;
        while (true){
            try {
                j++;
                System.out.println(Thread.currentThread().getName()+" "+j);
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if(j == 20){
              //在这里唤醒被暂停的线程
                LockSupport.unpark(t1);
            }
        }
    }
}

~~~

#### 特点

与 `Object` 的 `wait & notify` 相比

- `wait，notify` 和` notifyAll `必须配合` Object Monitor` 一起使用（也就是必须先获取对象的`monitor`锁），而 `park，unpark `不必
- `park & unpark `是以**线程为单位**来【阻塞】和【唤醒】线程，而` notify `只能随机唤醒一个等待线程，`notifyAll`是唤醒所有等待线程，就不那么【精确】
- unpark()可以精确的唤醒某一个线程。
- `park & unpark` 可以先` unpark`，而 `wait & notify `不能先` notify`

#### park和unpark底层原理

每个线程都有自己的一个 `Parker` 对象（底层是用c代码实现的），由三部分组成` _counter` ， `_cond` 和 `_mutex `打个比喻

- 线程就像一个旅人，`Parker `就像他随身携带的背包，条件变量（`_cond`)就好比背包中的帐篷。`_counter `就好比背包中的备用干粮（0 为耗尽，1 为充足）
- 调用 `park` 就是要看需不需要停下来歇息
  - 如果备用干粮耗尽，那么钻进帐篷歇息
  - 如果备用干粮充足，那么不需停留，继续前进
- 调用` unpark`，就好比令干粮充足
  - 如果这时线程还在帐篷，就唤醒让他继续前进
  - 调用unpark()就是先让_counter变为1，也就是干粮变为充足，然后在叫醒线程接着执行。
  - 如果这时线程还在运行，此时调用unpark()操作，那么这个时候就是相当于补充干粮，也就是把_counter（）变为1。那么下次他调用 `park `时，仅是消耗掉备用干粮，不需停留继续前进
    - 因为背包空间有限，多次调用` unpark `仅会补充一份备用干粮(也就是多次调用`unpark`，只会有一次起作用)

**先调用park然后调用unpark的情况**

**调用park**

![1608600691080](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/22/093132-940076.png)

1. 当前线程调用` Unsafe.park()` 方法
2. 检查` _counter` ，本情况为 0，这时，获得` _mutex `互斥锁
3. 线程进入`_cond `条件变量阻塞
4. 设置 `_counter = 0`

**调用unpark**

![1608600774572](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/22/093257-831830.png)

1. 调用 `Unsafe.unpark(Thread_0) `方法，首先设置 `_counter `为 1
2. 唤醒` _cond `条件变量（可以认为是阻塞队列）中的 Thread_0，这种情况是线程在阻塞队列中时的情况。
3. `Thread_0` 恢复运行
4. 设置` _counter `为 0

**先调用unpark后调用park**

![1608600844863](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/22/093413-319505.png)

1. 调用 `Unsafe.unpark(Thread_0) `方法，先试设置` _counter `为 1，也就是先补充能量。
2. 当前线程调用` Unsafe.park() `方法
3. 检查` _counter` ，本情况为 1，这时线程无需阻塞，继续运行
4. 设置 `_counter `为 0

### 重新理解线程之间的转换

- new：初始状态，仅仅创建java线程对象，并没有和操作系统的线程对象关联起来。当调用start()就可以和操作系统线程关联，由底层调度执行。
- running:java层面包含三个，运行状态的线程，等待调度的就绪现场，被操作系统阻塞起来的线程。
- watting:当线程获取到锁之后，调用wait()方法后就会从运行状态转换为watting状态。无时限的一直等待。

![1608601426553](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/22/094348-548698.png)

**假设有线程T**

##### **情况 1 NEW --> RUNNABLE**

- 当调用` t.start()` 方法时，由 `NEW --> RUNNABLE`

##### **情况 2 RUNNABLE <--> WAITING**

`t `线程用 `synchronized(obj) `获取了对象锁后

- 调用 `obj.wait()` 方法时，`t` 线程从` RUNNABLE --> WAITING`
- 调用` obj.notify() `， `obj.notifyAll() `， `t.interrupt() `时，因为线程被这三个方法调用的时候，但是唤醒之后不一定就是`runnable`，首先会进入`entryList`队列，等待某个线程释放锁之后去和之前竞争锁的线程一起竞争锁，所以有两种情况。
  - 竞争锁成功，t 线程从` WAITING --> RUNNABLE`
  - 竞争锁失败，t 线程从 `WAITING --> BLOCKED2`，`block`状态是当线程无法获取锁的时候，会进入阻塞状态。block状态也就是线程没有竞争到sunchronized时候就会进入block状态。

**代码说明**

~~~ java
public class Test24 {

    public static Object object=new Object();
    public static void main(String[] args) throws InterruptedException {

        new Thread(()->{
            synchronized (object){
                System.out.println(Thread.currentThread().getName()+" 开始执行");
                try {
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"   执行其他代码");
            }
        },"t1").start();

        new Thread(()->{
            synchronized (object){
                System.out.println(Thread.currentThread().getName()+" 开始执行");
                try {
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"   执行其他代码");
            }
        },"t2").start();

//        主线程先休息2秒
        Thread.sleep(2);
        synchronized (object){
//            主线程唤醒全部子线程
            System.out.println(Thread.currentThread().getName()+"  唤醒全部线程");
            object.notifyAll();
        }

    }
}
~~~

##### **情况 3 RUNNABLE <--> WAITING**

- 当前线程调用` t.join()` (当前线程等待线程t执行完成)方法时，**当前线程**从` RUNNABLE --> WAITING`

  注意是当前线程在`t` 线程对象的监视器上等待，比如想让主线程等待t线程执行完在执行，那么就在主线程中调用t线程的join方法。想让某一个线程等待，就在某个线程中调用另一个线程的join方法。此时主线程就会从running进入waiting等待。

- **t 线程运行结束，或调用了当前线程的 interrupt() 时**，当前线程从` WAITING --> RUNNABLE`

##### **情况 4 RUNNABLE <--> WAITING**

- 当前线程调用 `LockSupport.park()` 方法会让当前线程从 `RUNNABLE --> WAITING`
- 调用 `LockSupport.unpark`(目标线程) 或调用了线程 的` interrupt()` ，会让目标线程从` WAITING -->
  RUNNABLE`

**有时限的超时等待**

##### **情况 5 RUNNABLE <--> TIMED_WAITING**

t 线程用` synchronized(obj)` 获取了对象锁后

- 调用 `obj.wait(long n) `方法时，t 线程从` RUNNABLE --> TIMED_WAITING`
- t 线程等待时间超过了 n 毫秒，或调用 `obj.notify() ， obj.notifyAll() ， t.interrupt() `时
  - 竞争锁成功，t 线程从` TIMED_WAITING --> RUNNABLE`
  - 竞争锁失败，t 线程从 `TIMED_WAITING --> BLOCKED`

##### **情况 6 RUNNABLE <--> TIMED_WAITING**

- 当前线程调用` t.join(long n) `方法时，当前线程从 `RUNNABLE --> TIMED_WAITING`
  - 注意是当前线程在t 线程对象的监视器上等待
  - 当前线程等待时间超过了 n 毫秒，或t 线程运行结束，或调用了当前线程的 `interrupt()` 时，当前线程从`TIMED_WAITING --> RUNNABLE`

##### **情况 7 RUNNABLE <--> TIMED_WAITING**

- 当前线程调用 `Thread.sleep(long n) `，当前线程从 `RUNNABLE --> TIMED_WAITING`
- 当前线程等待时间超过了 n 毫秒，当前线程从 `TIMED_WAITING --> RUNNABLE`

##### **情况 8 RUNNABLE <--> TIMED_WAITING**

- 当前线程调用 `LockSupport.parkNanos(long nanos) `或` LockSupport.parkUntil(long millis)` 时，当前线
  程从 RUNNABLE --> TIMED_WAITING
- 调用 LockSupport.unpark(目标线程) 或调用了线程 的 interrupt() ，或是等待超时，会让目标线程从
  TIMED_WAITING--> RUNNABLE

##### **情况 9 RUNNABLE <--> BLOCKED**

- t 线程用 synchronized(obj) 获取了对象锁时如果竞争失败，从 RUNNABLE --> BLOCKED
- 持 obj 锁线程的同步代码块执行完毕，会唤醒该对象上所有 BLOCKED （block线程在entryList队列中 ）的线程重新竞争（也就是处于EntryList队列上面的线程），如果其中 t 线程竞争成功，从 BLOCKED --> RUNNABLE ，其它失败的线程仍然 BLOCKED

##### **情况 10 RUNNABLE <--> TERMINATED**

- 当前线程所有代码运行完毕，进入 TERMINATED

### 多把锁

**先看一个场景**

> 多把不相干的锁
> 一间大屋子有两个功能：睡觉、学习，互不相干。
> 现在小南要学习，小女要睡觉，但如果只用一间屋子（一个对象锁）的话，那么并发度很低
> 解决方法是准备多个房间（多个对象锁）

**代码说明**

~~~ java
public class Test25 {
    public static void main(String[] args) {
        BigRoom b=new BigRoom();
        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"  开始休息");
            try {
                b.Sleep();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"  休息好了");
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"  开始学习");
            try {
                b.learning();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"  学习好了");
        },"t2").start();

    }
}

//锁住整个对象的话线程之间的并发度很低，某一时刻t1需要睡觉，但是锁住的是整个对象，而t2不能获取该锁，所以不可以学习
//但是这两个线程不是互斥关系
class BigRoom{

    public void Sleep() throws InterruptedException {
        synchronized (this){
//            锁住的是当前的对象
            System.out.println("休息两秒钟.....");
            Thread.sleep(2);
        }
    }

    public void learning() throws InterruptedException {
        synchronized (this){
            System.out.println("学习2秒钟.....");
            Thread.sleep(2);
        }
    }
}
~~~

#### 解决方法

可以把大房间分成两个小房间，一个用来学习，一个用来休息，也就是申请多把锁对象。

**代码说明**

~~~ java
public class Test25 {
    public static void main(String[] args) {
        BigRoom b=new BigRoom();
        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"  开始休息");
            try {
                b.Sleep();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"  休息好了");
        },"t1").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"  开始学习");
            try {
                b.learning();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"  学习好了");
        },"t2").start();

    }
}
//锁住整个对象的话线程之间的并发度很低，某一时刻t1需要睡觉，但是锁住的是整个对象，而t2不能获取该锁，所以不可以学习
//但是这两个线程不是互斥关系
class BigRoom{
//增加多把锁来提高程序的并发度，但是要保证多个线程之间的业务是没有关联的，也就是不是互斥关系
  //相当于把一个大房子分成两个小房子
    private final Object studyRoom=new Object();
    private final Object sleepRoom=new Object();

    public void Sleep() throws InterruptedException {
        synchronized (sleepRoom){
//            锁住的是当前的对象
            System.out.println("休息两秒钟.....");
            Thread.sleep(2);
        }
    }

    public void learning() throws InterruptedException {
        synchronized (studyRoom){
            System.out.println("学习2秒钟.....");
            Thread.sleep(2);
        }
    }
}
~~~

**将锁的粒度细分**

- 好处，是可以增强并发度
- 坏处，如果一个线程需要同时获得多把锁，就容易发生死锁

### 活跃性

活跃性保函三种情况：死锁，活锁，饥饿

#### **死锁**

- 有这样的情况：一个线程需要同时获取多把锁，这时就容易发生死锁
- `t1` 线程 获得` A`对象 锁，接下来想获取` B`对象 的锁` t2 `线程 获得` B`对象 锁，接下来想获取 `A`对象 的锁 例：

**代码说明**

~~~ java
public class Test26 {
    public static void main(String[] args) {
        test01();
    }

    public static void test01(){
//        申请两把锁
        Object lock01=new Object();
        Object lock02=new Object();
        Thread t1=new Thread(()->{
            synchronized (lock01){
                System.out.println(Thread.currentThread().getName()+"  获取了lock01");
                try {
                    Thread.sleep(2);
                    System.out.println(Thread.currentThread().getName()+"  想获取lock02");
                    synchronized (lock02){
                        System.out.println(Thread.currentThread().getName()+"  获取lock02成功");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        t1.setName("t1");

        Thread t2=new Thread(()->{
            synchronized (lock02){
                System.out.println(Thread.currentThread().getName()+"  获取了lock02");
                try {
                    Thread.sleep(1);
                    System.out.println(Thread.currentThread().getName()+"  想获取lock01");
                    synchronized (lock01){
                        System.out.println(Thread.currentThread().getName()+"  获取lock01成功");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        t2.setName("t2");

        t1.start();
        t2.start();


    }
}
//输出结果
t1  获取了lock01
t2  获取了lock02
t2  想获取lock01
t1  想获取lock02
//两个线程第二次获取锁都没有成功
~~~

#### **定位死锁**

检测死锁可以使用 jconsole工具，或者使用 jps 定位进程 id，再用 jstack 定位死锁：

~~~ java
cmd > jps
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
12320 Jps
22816 KotlinCompileDaemon
33200 TestDeadLock // JVM 进程
11508 Main
28468 Launcher

cmd > jstack 33200
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
2018-12-29 05:51:40
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.91-b14 mixed mode):
"DestroyJavaVM" #13 prio=5 os_prio=0 tid=0x0000000003525000 nid=0x2f60 waiting on condition
[0x0000000000000000]
 java.lang.Thread.State: RUNNABLE
"Thread-1" #12 prio=5 os_prio=0 tid=0x000000001eb69000 nid=0xd40 waiting for monitor entry
[0x000000001f54f000]
 java.lang.Thread.State: BLOCKED (on object monitor)//没有获取到锁，处于block状态
 at thread.TestDeadLock.lambda$main$1(TestDeadLock.java:28)
 - waiting to lock <0x000000076b5bf1c0> (a java.lang.Object)//等待的锁对象
 - locked <0x000000076b5bf1d0> (a java.lang.Object)//自身获取的锁对象
 at thread.TestDeadLock$$Lambda$2/883049899.run(Unknown Source)
 at java.lang.Thread.run(Thread.java:745)
"Thread-0" #11 prio=5 os_prio=0 tid=0x000000001eb68800 nid=0x1b28 waiting for monitor entry
[0x000000001f44f000]
 java.lang.Thread.State: BLOCKED (on object monitor)
 at thread.TestDeadLock.lambda$main$0(TestDeadLock.java:15)
 - waiting to lock <0x000000076b5bf1d0> (a java.lang.Object)
 - locked <0x000000076b5bf1c0> (a java.lang.Object)
 at thread.TestDeadLock$$Lambda$1/495053715.run(Unknown Source)
 at java.lang.Thread.run(Thread.java:745)

// 略去部分输出，jvm会列出死锁的线程
Found one Java-level deadlock:
=============================
"Thread-1":
 waiting to lock monitor 0x000000000361d378 (object 0x000000076b5bf1c0, a java.lang.Object),
 which is held by "Thread-0"
"Thread-0":
 waiting to lock monitor 0x000000000361e768 (object 0x000000076b5bf1d0, a java.lang.Object),
 which is held by "Thread-1"
Java stack information for the threads listed above:
===================================================
"Thread-1":
 at thread.TestDeadLock.lambda$main$1(TestDeadLock.java:28)
 - waiting to lock <0x000000076b5bf1c0> (a java.lang.Object)
 - locked <0x000000076b5bf1d0> (a java.lang.Object)
 at thread.TestDeadLock$$Lambda$2/883049899.run(Unknown Source)
 at java.lang.Thread.run(Thread.java:745)
"Thread-0":
 at thread.TestDeadLock.lambda$main$0(TestDeadLock.java:15)
 - waiting to lock <0x000000076b5bf1d0> (a java.lang.Object)
 - locked <0x000000076b5bf1c0> (a java.lang.Object)
 at thread.TestDeadLock$$Lambda$1/495053715.run(Unknown Source)
 at java.lang.Thread.run(Thread.java:745)
Found 1 deadlock.
~~~

- 避免死锁要注意加锁顺序
- 另外如果由于某个线程进入了死循环，导致其它线程一直等待，对于这种情况 linux 下可以通过 top 先定位到CPU 占用高的 Java 进程，再利用 top -Hp 进程id 来定位是哪个线程，最后再用 jstack 排查

#### 哲学家就餐问题

**图示**

![1608702696798](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/06/101718-290182.png)

有五位哲学家，围坐在圆桌旁。

- 他们只做两件事，思考和吃饭，思考一会吃口饭，吃完饭后接着思考。
- 吃饭时要用两根筷子吃，桌上共有 5 根筷子，每位哲学家左右手边各有一根筷子。
- 如果筷子被身边的人拿着，自己就得等待

**代码说明**

~~~ java
public class Test27 {
    public static void main(String[] args) {
//        创建5个筷子
        Chopstick c1=new Chopstick("1");
        Chopstick c2=new Chopstick("2");
        Chopstick c3=new Chopstick("3");
        Chopstick c4=new Chopstick("4");
        Chopstick c5=new Chopstick("5");
        new Philosopher("苏格拉底",c1,c2).start();
        new Philosopher("柏拉图",c2,c3).start();
        new Philosopher("亚里士多德",c3,c4).start();
        new Philosopher("牛顿",c4,c5).start();
        new Philosopher("阿基米德",c5,c1).start();
    }
}


class Philosopher extends Thread{

    Chopstick left;
    Chopstick right;
    public Philosopher(String name,Chopstick left,Chopstick right){
        super(name);
        this.left=left;
        this.right=right;
    }
    public void eat() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"  开始吃饭");
        Thread.sleep(2);
    }


    @Override
    public void run() {
        while (true){
//            尝试获取左边的筷子
            synchronized (left){
//                尝试获取右边的筷子
                synchronized (right){
                    try {
                        eat();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }

    }
}

class Chopstick{
    String name;

    public Chopstick(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Chopstick{" +
                "name='" + name + '\'' +
                '}';
    }
}
~~~

- 可以使用 jconsole 检测死锁

#### 活锁

活锁出现在两个线程互相改变对方的结束条件，最后谁也无法结束，例如

~~~ java
public class TestLiveLock {
    static volatile int count = 10;
    static final Object lock = new Object();
    public static void main(String[] args) {
        new Thread(() -> {
            // 期望减到 0 退出循环
            while (count > 0) {
                Thread.sleep(0.2);
                count--;
                log.debug("count: {}", count);
            }
        }, "t1").start();
        new Thread(() -> {
            // 期望超过 20 退出循环
            while (count < 20) {
                Thread.sleep(0.2);
                count++;
                log.debug("count: {}", count);
            }
        }, "t2").start();
    }
}
~~~

- 活锁如何解决，让线程睡眠的时间是一个随机数即可。

#### 饥饿

很多教程中把饥饿定义为，一个线程由于优先级太低，始终得不到 CPU 调度执行，也不能够结束，饥饿的情况不易演示，讲读写锁时会涉及饥饿问题
下面我讲一下我遇到的一个线程饥饿的例子，先来看看使用顺序加锁的方式解决之前的死锁问题

**死锁**

![1608704667177](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/23/142436-176981.png)

**顺序加锁**

![1608704715868](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1608704715868.png)

- 也就是让某一个线程按顺序获取他需要的锁，等他把锁释放后，其他的线程在获取锁。

**顺序加锁解决哲学家问题**

~~~ java
public class Test27 {
    public static void main(String[] args) {
//        创建5个筷子
        Chopstick c1=new Chopstick("1");
        Chopstick c2=new Chopstick("2");
        Chopstick c3=new Chopstick("3");
        Chopstick c4=new Chopstick("4");
        Chopstick c5=new Chopstick("5");
      //哲学家拿筷子是顺序拿取
        new Philosopher("苏格拉底",c1,c2).start();
        new Philosopher("柏拉图",c2,c3).start();
        new Philosopher("亚里士多德",c3,c4).start();
        new Philosopher("牛顿",c4,c5).start();
        new Philosopher("阿基米德",c1,c5).start();
    }
}


class Philosopher extends Thread{

    Chopstick left;
    Chopstick right;
    public Philosopher(String name,Chopstick left,Chopstick right){
        super(name);
        this.left=left;
        this.right=right;
    }
    public void eat() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"  开始吃饭");
        Thread.sleep(2);
    }


    @Override
    public void run() {
        while (true){
//            尝试获取左边的筷子
            synchronized (left){
//                尝试获取右边的筷子
                synchronized (right){
                    try {
                        eat();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }

    }
}
class Chopstick{
    String name;

    public Chopstick(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Chopstick{" +
                "name='" + name + '\'' +
                '}';
    }
}
~~~

###  ReentrantLock

可重入锁。

相对于 synchronized 它具备如下特点

- 可中断，但是synchronized锁就不能被中断。
- 可以设置超时时间（设置一个等待时间，如果时间内没有获取到锁，就放弃争抢锁，执行其他的逻辑）
- 可以设置为公平锁（**防止发生饥饿，比如先到先得**）
- 支持多个条件变量（**也就是说有多个等待队列，因为不同条件发生的等待被放入不同的等待队列**），而synchronized是所条件引起的等待都去wait队列中等待。
- 与synchronized相同之处是都支持可重入，也就是同一个线程对同一个锁对象多次添加锁。
- synchronized是关键字级别保护临界区资源，而ReentrantLock是在对象级别进行保护。

**继承关系**

![1620267926072](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/06/102531-941455.png)

**语法说明**

~~~ java
//创建对象
// 调用lock()方法获取锁
reentrantLock.lock();//加锁在try里面或者外面都可以
try {
 // 临界区
} finally {
 // 释放锁
 reentrantLock.unlock();
}
//加锁和解锁是成对出现
~~~

#### 可重入特性

可重入是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获取这把锁
如果是不可重入锁，那么第二次获得锁时，自己也会被锁挡住，**synchronize和ReentrantLock都是可重入的锁**

**代码说明**

~~~ java
public class Test28 {
//    获取对象
    private static ReentrantLock reentrantLock=new ReentrantLock();
    public static void main(String[] args) {
//        获取锁
        reentrantLock.lock();
        try {
            System.out.println("main get lock");
            m1();
        } finally {
//解锁
            reentrantLock.unlock();
        }
    }

    public static void m1(){
//加锁
//锁重入
        try {
          //再次获取锁，相当于锁重入
            reentrantLock.lock();
            System.out.println("m1 get lock");
            m2();
        }finally {
            reentrantLock.unlock();
        }

    }
    public static void m2(){
//加锁
//锁重入
        try {
            reentrantLock.lock();
            System.out.println("m2 get lock");
        }finally {
            reentrantLock.unlock();
        }
    }
}
//上面的锁重入都是同一个线程在获取锁操作
~~~

#### 可打断性特性

**代码说明**

~~~ java
public class Test29 {
    private static ReentrantLock r=new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println("尝试获取锁");
            try {
//                如果没有竞争，此方法就会获取对象锁，如果有竞争，就会进入阻塞队列
//                可以被其他线程使用interrupt方法进行打断，不要在等下去
                r.lockInterruptibly();
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println("没有获取到锁");
                return;
            }
            try {
                System.out.println("获取到了锁");
            }finally {
                r.unlock();
            }
        },"t1");

//        先让主线程获取锁
        r.lock();
        thread.start();

        Thread.sleep(2);
//        主线程打断t1线程
        thread.interrupt();
    }
}
~~~

> 如果使用的是lock()模式，即使去打断，也不会真正的打断。

#### 超时特性

**代码说明**

~~~ java
public class Test30 {
    public static ReentrantLock lock=new ReentrantLock();
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
//            尝试去获取锁，会返回一个boolean值表示是否获取成功
            try {
                System.out.println(Thread.currentThread().getName()+"  尝试获取锁");
                if(!lock.tryLock(1, TimeUnit.SECONDS)){
    //                没有带参数时间的tryLock(),获取不到锁，会立刻返回
//                    带参数的锁表示等待一段时间，如果换没有获取到锁，就返回
                    System.out.println(Thread.currentThread().getName()+"  没有获取到锁");
                    return;
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println(Thread.currentThread().getName()+"  获取不到锁");
                return;
            }
//            如果获取成功，就执行临界区的代码
            try {
                System.out.println(Thread.currentThread().getName()+"  获取锁成功，执行临界区代码");
            }finally {
                lock.unlock();
            }
        },"t1");
        thread.start();

//        有竞争，先让主线程获取锁
        lock.tryLock();
        System.out.println(Thread.currentThread().getName()+"  获取到了锁");
    }
}
~~~

#### 哲学家就餐问题

~~~ java
public class Test27 {
    public static void main(String[] args) {
//        创建5个筷子
        Chopstick c1=new Chopstick("1");
        Chopstick c2=new Chopstick("2");
        Chopstick c3=new Chopstick("3");
        Chopstick c4=new Chopstick("4");
        Chopstick c5=new Chopstick("5");
        new Philosopher("苏格拉底",c1,c2).start();
        new Philosopher("柏拉图",c2,c3).start();
        new Philosopher("亚里士多德",c3,c4).start();
        new Philosopher("牛顿",c4,c5).start();
        new Philosopher("阿基米德",c1,c5).start();
    }
}

class Philosopher extends Thread{

    Chopstick left;
    Chopstick right;
    public Philosopher(String name,Chopstick left,Chopstick right){
        super(name);
        this.left=left;
        this.right=right;
    }
    public void eat() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"  开始吃饭");
        Thread.sleep(2);
    }


    @Override
    public void run() {
        while (true){
//            尝试获取左边的筷子,使用此方法表示获取不到左边的筷子时候，就放弃等待
            if(left.tryLock()){
                try{
//                    尝试获取右手的筷子
                    if(right.tryLock()){
                        try{
//如果可以走到这里，说明两把筷子都拿到了，所以就可以去吃饭
                            eat();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        } finally {
                            right.unlock();
                        }
                    }

                }finally {
//                    如果某一个哲学家获取锁没有成功，就会释放自己收中的锁，可以避免死锁
                    left.unlock();
                }
            }


        }

    }
}

class Chopstick extends ReentrantLock {
    String name;

    public Chopstick(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Chopstick{" +
                "name='" + name + '\'' +
                '}';
    }
}
~~~

#### 公平锁特性

- synchronized就是不公平的锁，当一个线程持有锁的时候，其他的线程就会进入阻塞队列进行等待，但是当线程把锁释放之后，其他线程是抢占式的获取锁，没有遵循先来先得的原则，可能会发生饥饿现象，所以是不公平的。

- ReentrantLock也是不公平的锁，但是可以通过设置成为公平的锁。公平锁一般没有必要，会降低并发度，公平锁是用来解决饥饿为题的。

~~~ java
 /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }
//默认是非公平的锁
~~~

#### 条件变量特性

**Locj接口**

~~~ java
public interface Lock {

 
    void lock();

    
    void lockInterruptibly() throws InterruptedException;

   
    boolean tryLock();

    
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

   
    void unlock();

    //条件变量
    Condition newCondition();
}

~~~

**条件变量接口**

~~~ java
public interface Condition {

  //调用此方法，线程就会进入条件变量等待
    void await() throws InterruptedException;

 
    void awaitUninterruptibly();

//有时限的等待
    long awaitNanos(long nanosTimeout) throws InterruptedException;

  
    boolean awaitUntil(Date deadline) throws InterruptedException;

    //唤醒条件变量中的某一个线程
    void signal();

 //唤醒条件变量中所有的等待线程
    void signalAll();
}

~~~

synchronized 中也有条件变量，就是我们讲原理时那个 waitSet 休息室，当条件不满足时进入 waitSet 等待

ReentrantLock 的条件变量比 synchronized 强大之处在于，它是支持多个条件变量的，这就好比

- synchronized 是那些不满足条件的线程都在一间休息室等消息
- 而 ReentrantLock 支持多间休息室，有专门等烟的休息室、专门等早餐的休息室、唤醒时也是按休息室来唤醒

**使用要点**

- await 前需要获得锁
- await 执行后，会释放锁，进入 conditionObject 等待
- await 的线程被唤醒（或打断、或超时）后重新竞争 lock 锁
- 竞争 lock 锁成功后，从 await 后继续执行

**代码演示**

~~~ java
public class Test31 {
    public static ReentrantLock lock=new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {
//        返回一个条件变量，可以看做休息室，通一把锁可以有多个条件变量
        Condition condition = lock.newCondition();
        Condition condition1 = lock.newCondition();

//        必须先加锁，在进入休息室,想进入哪一个休息室，就调用哪一个条件变量
        lock.lock();
        condition.wait();


//        其他线程想叫醒你，可以调用下面方法
        condition.signal();
//        也可以把某一个条件变量中的所有线程全部唤醒
        condition.signalAll();
    }
}

//获取条件变量的方法
public Condition newCondition() {
        return sync.newCondition();
    }
~~~

##### 条件变量案例

~~~ java
public class Test32 {

    static ReentrantLock lock = new ReentrantLock();
//    两个条件变量
    static Condition waitCigaretteQueue = lock.newCondition();
    static Condition waitbreakfastQueue = lock.newCondition();
    static volatile boolean hasCigrette = false;
    static volatile boolean hasBreakfast = false;
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            try {
//                添加锁
                lock.lock();
//                判断是否有烟
                System.out.println(Thread.currentThread().getName()+"  有没有烟  "+hasCigrette);
                while (!hasCigrette) {
                    try {
//                        如果没烟，就添加到等烟的休息室
                        System.out.println(Thread.currentThread().getName()+"  进入休息室等待");
                        waitCigaretteQueue.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName()+"  等到了它的烟");
            } finally {
//                解锁操作
                lock.unlock();
            }
        }).start();
        new Thread(() -> {
            try {
                lock.lock();
                System.out.println(Thread.currentThread().getName()+"  有没有早餐  "+hasBreakfast);
                while (!hasBreakfast) {
                    try {
                        System.out.println(Thread.currentThread().getName()+"  进入休息室等待");
                        waitbreakfastQueue.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName()+"  等到了它的早餐");
            } finally {
                lock.unlock();
            }
        }).start();
        Thread.sleep(1);
        sendBreakfast();
        Thread.sleep(1);
        sendCigarette();
    }
    private static void sendCigarette() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"  送烟来了");
            hasCigrette = true;
            waitCigaretteQueue.signal();
        } finally {
            lock.unlock();
        }
    }
    private static void sendBreakfast() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"  送早餐来了");
            hasBreakfast = true;
            waitbreakfastQueue.signal();
        } finally {
            lock.unlock();
        }
    }
}
~~~

#### 模式

同步模式之顺序控制

##### 固定运行顺序

**案例**

先2后1打印

###### **notify/wait版本实现**

~~~ java
public class Test33 {

    static final Object lock=new Object();
//    表示t2线程是否运行过
    static boolean t2Runed=false;
    public static void main(String[] args) {


        Thread t1 = new Thread(() -> {
            synchronized (lock){
//                先获取锁对象
                while (!t2Runed){
                    try {
//                        t1线程进入wait后就会释放锁，此时t2线程可以获取锁
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("1");
            }
        }, "t1");

        Thread t2 = new Thread(() -> {
//            t2线程先获取锁
            synchronized (lock){
                System.out.println("2");
//                表示t2线程已经运行过
                t2Runed=true;
//                唤醒t1线程
                lock.notify();
            }

        }, "t2");

//        如果不加控制，那么系统调用t1 t2线程之间的顺序是不确定的
        t1.start();
        t2.start();
    }
}
~~~

###### park和unpark实现

~~~ java
public class Test34 {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
//            先把t1线程挂起
            LockSupport.park();
            System.out.println("1");
        }, "t1");

        Thread t2 = new Thread(() -> {
            System.out.println("2");
            //            唤醒正在阻塞队列的t1线程
            LockSupport.unpark(t1);

        }, "t2");

        t1.start();
        t2.start();
    }
}
~~~

###### ReentrantLock方法

~~~ java

~~~

##### 交替输出

三个线程交替输出a,b,c各5次。











### 小结

**本章我们需要重点掌握的是**

- 分析多线程访问共享资源时，哪些代码片段属于临界区（多个线程对某一段代码既有读又有写得是临界区），有两种方法实现对临界区的保护，sychronized和ReentrantLock。

  - 使用 synchronized 互斥解决临界区的线程安全问题

    - 掌握 synchronized 锁对象语法
    - 掌握 synchronzied 加载成员方法(锁住this对象)和静态方法（锁住class对象）语法
    - 掌握 wait/notify 同步方法
    - 互斥是保护临界区资源由于线程的上下文切换而产生指令交错，保证临界区代码的原子性，而同步时保证线程由于条件不满足而产生等待，等条件恢复后就继续运行。

  - 使用 lock（指的是ReentrantLock锁） 互斥解决临界区的线程安全问题，比synchronized功能强大。
    - 掌握 lock 的使用细节：可打断、锁超时（保证不会产生死等）、公平锁（sychronized和ReentrantLock默认非公平）、条件变量
  - 学会分析变量的线程安全性、掌握常见线程安全类的使用
  - 了解线程活跃性问题：死锁、活锁、饥饿
  - 应用方面
    - 互斥：使用 synchronized 或 Lock 达到共享资源互斥效果
    - 同步：使用 wait/notify 或 Lock 的条件变量来达到线程间通信效果
  - 原理方面
    - monitor、synchronized 、wait/notify 原理
    - synchronized 进阶原理
    - park & unpark 原理
  - 模式方面
    - 同步模式之保护性暂停（一一对应关系）
    - 异步模式之生产者消费者（非一一对应关系）
    - 同步模式之顺序控制

## 共享模型之内存

monitor主要是关注访问共享变量的时候，保证临界区代码的原子性。

下面学习共享变量在多线程之间的可见性问题和多条指令之间执行的有序性问题。

JMM 即 Java Memory Model，它定义了主存、工作内存抽象概念，底层对应着 CPU 寄存器、缓存、硬件内存、
CPU 指令优化等。

- JMM 体现在以下几个方面
  - 原子性 - 保证指令不会受到线程上下文切换的影响
  - 可见性 - 保证指令不会受 cpu 缓存的影响
  - 有序性 - 保证指令不会受 cpu 指令并行优化的影响

**原子性**

基于基本数据类型的访问，读写都是具备原子性的，更大范围的原子性保证，JAVA内存模型提供了lock和unlock的操作来满足，也可以使用synchronized锁机制来保证原子性。

**可见性**

volatile变量可以保证变量的可见性，但是普通的变量不能保证这一点。

除了volatile关键字保证可见性外，synchronized和final两个关键字也可以保证可见性，synchronized保证可见性是基于在unlock之前，会先把变量的值写会到主存之中保证的，而final关键字的可见性指的是被final修饰的变量在构造器中一旦初始化完成，并且在构造器中没有吧this的引用传递出去，那么其他的线程就可以看见final关键字的值，

**有序性**

java语言中提供volatile和synchronized两个关键字来保证有序性，volatile关键字本身就可以禁止指令重拍，而synchronized则是由一个变量在同一时刻只允许一个线程对其进行lock操作所保证的。

### Java内存模型

`JMM` 即 `Java Memory Model`，它定义了主存、工作内存抽象概念，底层对应着 `CPU` 寄存器、缓存、硬件内存、`CPU` 指令优化等。

主要目的是定义程序中各种变量之间的访问规则。也就是关注虚拟机中国把变量的值存储到内存和从内存中取出变量的值这样的底层细节，这里的变量指的是实例字段，静态字段和数组对象，不包括局部变量和方法的参数，因为这两者是线程私有的，不会被共享。

java内存模型规定的所有变量都保存在主内存中，但是每一条线程都还有各自的工作内存，工作内存中保存的是主内存中变量的副本，线程对所有变量的操作都是在工作内存中进行的，不可以直接操作主内存中的数据。多个线程之间的通信需要通过主内存进行。

`JMM` 体现在以下几个方面

- 原子性 - 保证指令不会受到**线程上下文切换**的影响
- 可见性 - 保证指令不会**受` cpu `缓存**的影响
- 有序性 - 保证指令不会**受` cpu` 指令并行优化**的影响

**volatile关键字可以保证可见性和有序性，不能保证原子性。**

#### 可见性

**退不出循环**

先来看一个现象，`main` 线程对 `run `变量的修改对于` t `线程不可见，导致了` t `线程无法停止：

~~~ java
public class Test38 {
    static boolean flag=false;

    public static void main(String[] args) throws InterruptedException {
        Thread t=new Thread(()->{
            while (flag){}
        });
        t.start();

        System.out.println("主线程开始执行");
        Thread.sleep(1);
        flag=true;
    }
}
~~~

**原因分析**

1. 初始状态，` t` 线程刚开始从主内存读取了` run `的值到工作内存。**主存就是存储共享变量的地方，工作内存是各个线程私有的地方。**

![1608942065426](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/26/082108-998428.png)

2. . 因为` t` 线程要频繁从主内存中读取` run `的值，`JIT `编译器会将 `run `的值缓存至自己工作内存中的高速缓存中，减少对主存中 `run` 的访问，提高效率

![1608942321652](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/27/091734-89675.png)

3.  1 秒之后，`main` 线程修改了` run` 的值，并同步至主存，而 `t `是从自己工作内存中的高速缓存中读取这个变量的值，结果永远是旧值

![1608942362290](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/27/091805-945568.png)

**解决方法**

`volatile`（易变关键字）

- 它可以用来修饰**成员变量和静态成员变量**（局部变量是私有的），他可以避免线程从自己的**工作缓存中查找变量的值，必须到主存中获取它的值**，**线程操作 volatile 变量都是直接操作主存**

**解决方法代码说明**

~~~ java
//使用volatile关键字
public class Test38 {
     volatile static boolean flag=true;

    public static void main(String[] args) throws InterruptedException {
        Thread t=new Thread(()->{
            while (flag){}
        });
        t.start();

        System.out.println("主线程开始执行");
        Thread.sleep(2);
        flag=false;
    }
}
//使用synchronized关键字
public class Test38 {
     volatile static boolean flag=true;
     public static final Object lock=new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread t=new Thread(()->{
            while (flag){
                synchronized (lock){
                    if(!flag){
                        break;
                    }
                }
            }
        });
        t.start();

        System.out.println("主线程开始执行");
        Thread.sleep(2);
       synchronized (lock){
           flag=false;
       }
    }
}
~~~

> `synchronized`是重量级的，所以在解决**可见性**方面推荐使用`volatile`关键字

**可见性和原子性的理解**

前面例子体现的实际就是可见性，它保证的是在多个线程之间，一个线程对 `volatile` 变量的修改对另一个线程可见， 不能保证原子性，**仅用在一个写线程，多个读线程的情况：** 上例从字节码理解是这样的：

~~~ java
getstatic run // 线程 t 获取 run true
getstatic run // 线程 t 获取 run true
getstatic run // 线程 t 获取 run true
getstatic run // 线程 t 获取 run true
putstatic run // 线程 main 修改 run 为 false， 仅此一次
getstatic run // 线程 t 获取 run false
~~~

比较一下之前我们将线程安全时举的例子：两个线程一个 `i++ `一个` i--` ，使用`volatile`关键字只能保证看到**最新值**，不能解决指令交错，也就是不能保证指令的原子性。

~~~ java
// 假设i的初始值为0
getstatic i // 线程2-获取静态变量i的值 线程内i=0
getstatic i // 线程1-获取静态变量i的值 线程内i=0
iconst_1 // 线程1-准备常量1
iadd // 线程1-自增 线程内i=1
putstatic i // 线程1-将修改后的值存入静态变量i 静态变量i=1
iconst_1 // 线程2-准备常量1
isub // 线程2-自减 线程内i=-1
putstatic i // 线程2-将修改后的值存入静态变量i 静态变量i=-1
  
//上面的例子中，对i添加volatile关键字只能保障获取最新的i值，并不能保证指令不会交错执行
~~~

> 注意
>
> - `synchronized` 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。但缺点是
>   `synchronized `是属于重量级操作，性能相对更低
> - volatile关键字可以保证可见性和有序性，不能保证原子性。
> - 如果在前面示例的死循环中加入 `System.out.println()` 会发现即使不加` volatile` 修饰符，线程 t 也能正确看到对 `run `变量的修改了，想一想为什么？

**源码角度看**

~~~ java
public void println(String x) {
        synchronized (this) {
            print(x);
            newLine();
        }
    }
//println()函数里面添加了synchronized关键字，可以保障对共享变量访问的可见性
~~~

#### CPU的缓存结构

##### 指令级并行原理

**名词**

- Clock Cycle Time：主频的概念大家接触的比较多，而 CPU 的 Clock Cycle Time（时钟周期时间），等于主频的倒数，意思是 CPU 能够识别的最小时间单位，比如说 4G 主频的 CPU 的 Clock Cycle Time 就是 0.25 ns，作为对比，我们墙上挂钟的Cycle Time 是 1s，例如，运行一条加法指令一般需要一个时钟周期时间

- CPI:有的指令需要更多的时钟周期时间，所以引出了 CPI （Cycles Per Instruction）指令平均时钟周期数

- IPC:IPC（Instruction Per Clock Cycle） 即 CPI 的倒数，表示每个时钟周期能够运行的指令数

- CPU的执行时间

程序的 CPU 执行时间，即我们前面提到的 user + system 时间，可以用下面的公式来表示

~~~ java
程序 CPU 执行时间 = 指令数 * CPI * Clock Cycle Time
~~~

#####  指令重排序优化

- 事实上，现代处理器会设计为一个时钟周期完成一条执行时间最长的 CPU 指令。为什么这么做呢？可以想到指令还可以再划分成一个个更小的阶段，例如，每条指令都可以分为： 取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回 这 5 个阶段

![1609032637378](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/27/093040-534362.png)

- 在不改变程序结果的前提下，这些指令的各个阶段可以通过重排序和组合来实现指令级并行，这一技术在 80's 中叶到 90's 中叶占据了计算架构的重要地位。

> 提示：
> 分阶段，分工是提升效率的关键！

指令重排的前提是，重排指令不能影响结果，例如

~~~java
// 可以重排的例子
int a = 10; // 指令1
int b = 20; // 指令2
System.out.println( a + b );
// 不能重排的例子
int a = 10; // 指令1
int b = a - 5; // 指令2
~~~

##### 支持流水线的处理器

现代 CPU 支持多级指令流水线，例如支持同时执行取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回 的处理器，就可以称之为五级指令流水线。这时 CPU 可以在一个时钟周期内，同时运行五条指令的不同阶段（相当于一条执行时间最长的复杂指令），IPC = 1，本质上，流水线技术并不能缩短单条指令的执行时间，但它变相地提高了指令地**吞吐率。**

![1609032798655](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/27/093320-796490.png)

多条指令可以同时执行，不会影响结果。

##### SuperScalar 处理器

- 大多数处理器包含多个执行单元，并不是所有计算功能都集中在一起，可以再细分为整数运算单元、浮点数运算单元等，这样可以把多条指令也可以做到并行获取、译码等，CPU 可以在一个时钟周期内，执行多于一条指令，IPC

![1609032914548](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/05/202539-141903.png)

#### CPU 缓存结构原理

##### cpu缓存结构

![1609032965276](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/27/093607-587222.png)

**速度比较**

![1609033018013](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/27/093701-641118.png)

##### 内存屏障

Memory Barrier（Memory Fence）

- 可见性
  - **写屏障（`sfence`）保证在该屏障之前的，对共享变量的改动，都同步到主存当中**
  - **而读屏障（`lfence`）保证在该屏障之后，对共享变量的读取，加载的是主存中最新数据**
- 有序性
  - **写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后**
  - **读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前**

![1609033272474](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/27/094114-539976.png)

### 模式


#### 使用volatile关键字改进两阶段中止模式

如果某一个线程正在睡眠之中而被打断，就会重新设置打断标记，所以我们在catch块中需要重新打断一次，恢复打断标记，这种做法不方便，所以使用一个变量作为标记。

~~~ java
public class Test10 {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination t1=new TwoPhaseTermination();
        t1.start();

//        主线程休眠3.5秒
        Thread.sleep(3500);
//        主线程去打断子线程的执行
        t1.stop();

    }
}

class TwoPhaseTermination{
//    创建一个监控线程
    private Thread monitor;
  //因为两个线程访问这变量，所以要保证可见性
    private static volatile boolean flag=false;
//    启动监控线程
    public void start(){
        monitor=new Thread(()->{
//            在这里时刻监控当前线程是否被打断
            while (true){
                boolean interrupt=Thread.currentThread().isInterrupted();
                if(flag){
                    System.out.println("程序已经被打断........");
                    break;
                }
//                如果没有被打断，就执行监控操作
                try {
//                    在下面这两条语句都有可能被打断
                    Thread.sleep(1000);//这种情况如果被打断，打断标记将会被设置为false，这里打断会抛出异常
//                    下面语句如果被打断，那么是正常被打断，她的的打断标记会设置为true,可以正常退出
                    System.out.println("执行监控记录......");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                  //不需要重新设置打断标记
                }
            }
        });
        monitor.start();
    }
//monitor是一个线程，调用stop方法的是另一个线程，两个线程修改同一个变量，存在可见性问题
//    所以标记需要添加volatile关键字

//    提供停止监控线程
    public void stop(){
//        对线程进行打断
        flag=true;
      //如果当前线程正在睡眠，那么使用interrupt()可以尽快让线程打断
      monitor.interrupt();
    }
}
~~~

#### 同步模式之-baking模式

上面的监控线程，每调用一次，都会创建一个新的监控线程，这样会创建大量的监控线程，这样做是没有意义的。

**定义**

`Balking `（犹豫）模式用在一个线程发现另一个线程或本线程已经做了某一件相同的事，那么本线程就无需再做了，直接结束返回

**代码实现**

~~~ java
public class Test10 {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination t1=new TwoPhaseTermination();
      //两次调用监控线程，会创建两个监控线程
        t1.start();
        t1.start();

//        主线程休眠3.5秒
//        Thread.sleep(3500);
////        主线程去打断子线程的执行
//        t1.stop();

    }
}

class TwoPhaseTermination{
//    创建一个监控线程
    private Thread monitor;
    private static boolean flag=false;

//    添加一个boolean表示start()是否被调用过
    private boolean starting=false;
//    启动监控线程
    public void start(){
//下面的判断在单线程下可以正确执行，但是在多线程下就会出现问题
       synchronized (this){
         //这个锁是保证starting在多线程情况下正确执行，注意和flag区分开，因为starting共享变量既有读也有写，有多行语句，所以使用锁
//           锁中的代码越多，执行时间越长，所以也可以把锁添加在对starting的修改中，并不会影响并发程度
           if(starting){
//            如果starting为真，就说明已经有监控线程了，就不在需要创建监控线程，直接返回
               return;
           }
           starting=true;
         //可以把线程的创建放在锁的外面，可以提高效率，思考为什么？
           monitor=new Thread(()->{
//            在这里时刻监控当前线程是否被打断
               while (true){
                   boolean interrupt=Thread.currentThread().isInterrupted();
                   if(flag){
                       System.out.println("程序已经被打断........");
                       break;
                   }
//                如果没有被打断，就执行监控操作
                   try {
//                    在下面这两条语句都有可能被打断
                       Thread.sleep(1000);//这种情况如果被打断，打断标记将会被设置为false，这里打断会抛出异常
//                    下面语句如果被打断，那么是正常被打断，她的的打断标记会设置为true,可以正常退出
                       System.out.println("执行监控记录......");
                   } catch (InterruptedException e) {
                       e.printStackTrace();
//                  重新设置打断标记
                       Thread.currentThread().interrupt();//在这里把打断标记重新设置为true
                   }
               }
           });
       }
        monitor.start();
    }
//monitor是一个线程，调用stop方法的是另一个线程，两个线程修改同一个变量，存在可见性问题
//    所以标记需要添加volatile关键字

//    提供停止监控线程
    public void stop(){
//        对线程进行打断
        monitor.interrupt();
        flag=true;
    }
}
//多次启动线程，只会创建一个线程
//同步块越短，性能越好，思考为什么？
~~~

**线程安全的单例模式**

~~~ java
public final class Singleton {
    private Singleton() {
    }
    private static Singleton INSTANCE = null;
  //synchronized锁保证多线程的安全，是一种懒惰创建对象，先判断有没有对象，有的话直接返回，没有的话在创建对象
    public static synchronized Singleton getInstance() {
        if (INSTANCE != null) {
            return INSTANCE;
        }

        INSTANCE = new Singleton();
        return INSTANCE;
    }
}
~~~

对比一下保护性暂停模式：保护性暂停模式用在一个线程等待另一个线程的执行结果，当条件不满足时线程等待。

### 有序性

`JVM `会在不影响正确性的前提下，可以调整语句的执行顺序，思考下面一段代码

~~~ java
static int i;
static int j;
// 在某个线程内执行如下赋值操作
i = ...;
j = ...;
//可以看到，至于是先执行 i 还是 先执行 j ，对最终的结果不会产生影响。所以，上面代码真正执行时，既可以是
i = ...;
j = ...;
//也可以是
j = ...;
i = ...;
~~~

这种特性称之为『**指令重排**』，多线程下『指令重排』会影响正确性。为什么要有重排指令这项优化呢？从 CPU执行指令的原理来理解一下吧

**参考前面的`cpu`缓存结构和`cpu`缓存结构原理章节**

**代码示例**

~~~ java
int num=0
   boolean ready = false;
    // 线程1 执行此方法
    public void actor1(I_Result r) {
        if(ready) {
            r.r1 = num + num;
        } else {
            r.r1 = 1;
        }
    }
    // 线程2 执行此方法
    public void actor2(I_Result r) {
        num = 2;
        ready = true;
    }
~~~

- I_Result 是一个对象，有一个属性 r1 用来保存结果，问，可能的结果有几种？
  - 情况1：线程1 先执行，这时 ready = false，所以进入 else 分支结果为 1
  - 情况2：线程2 先执行 num = 2，但没来得及执行 ready = true，线程1 执行，还是进入 else 分支，结果为1
  - 情况3：线程2 执行到 ready = true，线程1 执行，这回进入 if 分支，结果为 4（因为 num 已经执行过了）
  - 结果还有可能是 0，信不信吧！
    - 这种情况下是：线程2 执行 ready = true，切换到线程1，进入 if 分支，相加为 0，再切回线程2 执行 num = 2
    - 这种现象叫做指令重排，是 **JIT 编译器**在运行时的一些优化，这个现象需要通过大量测试才能复现

**解决方法**

volatile 修饰的变量，可以禁用指令重排

~~~ java
volatile boolean ready = false;
//给ready添加volatile关键字，可以防止给ready赋值之前的语句进行重排序操作
~~~

### volatile关键字原理

**两个作用**

1. 保证某一个变量对所有线程的可见性，，也就是当一个线程对某一个变量修改了之后，新的值对于其他的线程来说是立即可知的。
2. 基于volatile的变量在并发的环境下不一定是线程安全的。volatile变量不存在线程一致性问题，但是java里面的运算操作符并非是原子操作的，所以不一定线程安全。就比如race变量用volatile修饰过，虽然改变race变量对于其他线程可见，但是++操作并不是原子操作，所以非线程安全。
3. 使用volatile的第二个语义是可以保证指令重拍操作。

`volatile `的底层实现原理是内存屏障，`Memory Barrier（Memory Fence）`

- 对 `volatile` 变量的**写指令后会加入写屏障**
- 对 `volatile` 变量的**读指令前会加入读屏障**

#### 如何保证可见性

写屏障（`sfence`）保证在该屏障之前的，对共享变量的改动，都同步到主存当中

~~~ java
public void actor2(I_Result r) {
 num = 2;
 ready = true; // ready 是 volatile 赋值带写屏障
 // 写屏障
}
//在赋值操作之前添加写屏障，所有在写屏障之前对共享变量的赋值操作，都会写入主存
~~~

而读屏障（`lfence`）保证在该屏障之后，对共享变量的读取，加载的是主存中最新数据

~~~ java
public void actor1(I_Result r) {
 // 读屏障
 // ready 是 volatile 读取值带读屏障
 if(ready) {
 	r.r1 = num + num;
 } else {
	 r.r1 = 1;
 }
}
//也就是在读取之前添加一个读屏障，在读屏障之后所有的读取操作都会在主存中读取
~~~

**图示**

![1608950754388](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/26/104555-937994.png)

#### 如何保证有序性

**写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后**

~~~ java
 public void actor2(I_Result r) {
        num = 2;
        ready = true; // ready 是 volatile 赋值带写屏障
        // 写屏障
    }
//也就是说num=2指令不会出现在ready=true指令的后面
~~~

**读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前**

~~~ java
public void actor1(I_Result r) {
        // 读屏障
        // ready 是 volatile 读取值带读屏障
        if(ready) {
            r.r1 = num + num;
        } else {
            r.r1 = 1;
        }
    }
~~~

**图示**

![1608951057615](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/26/105058-256618.png)

还是那句话，不能解决指令交错：比如i++或者i–的指令交错问题。

- **写屏障仅仅是保证之后的读能够读到最新的结果，但不能保证读跑到它前面去**
- **而有序性的保证也只是保证了本线程内相关代码不被重排序**，本线程内的读写操作有序，但是两个线程之间的读写操作并不能保证有序，所以这可能就会导致指令交错。
- **volatile只能解决有序性和可见性，不能保证原子性，但是synchronized三者都可以保证。**

![1608951247957](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/26/105410-201648.png)

比如上图中t2线程在t1线程写入之前已经读取i的值，然后在写入，所以volatile不能保证指令交错问题。

#### double-checked locking 问题

以著名的 double-checked locking 单例模式为例

~~~ java
public final class Singleton {
    private Singleton() { }
    private static Singleton INSTANCE = null;
    public static Singleton getInstance() {
        if(INSTANCE == null) { // t2
            // 首次访问会同步，而之后的使用没有 synchronized
            synchronized(Singleton.class) {
                if (INSTANCE == null) { // t1
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
synchronized加的范围太大，对性能的影响较大
上面的代码存在问题，因为INSTANCE可能存在指令重拍问题
~~~

- 以上的实现特点是：
  - 懒惰实例化
  - 首次使用 getInstance() 才使用 synchronized 加锁，后续使用时无需加锁
  - 有隐含的，但很关键的一点：第一个 if 使用了 INSTANCE 变量，是在同步块之外

但在多线程环境下，上面的代码是有问题的，getInstance 方法对应的字节码为：

~~~ java
0: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;//获取静态对象
3: ifnonnull 37 //if语句判断是否是null
6: ldc #3 // class cn/itcast/n5/Singleton //获取类对象进行加锁
8: dup//复制类对象的引用指针
9: astore_0 //存储类对象的引用指针
10: monitorenter //进入同步代码块，创建monitor对象
11: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton; //获取静态变量
14: ifnonnull 27//判断静态变量是否是空，如果不是空，就执行27行指令，返回静态对象
17: new #3 // class cn/itcast/n5/Singleton //如果是空，就创建对象
20: dup //复制实例对象的引用
21: invokespecial #4 // Method "<init>":()V// 调用构造方法
24: putstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
27: aload_0
28: monitorexit
29: goto 37
32: astore_1
33: aload_0
34: monitorexit
35: aload_1
36: athrow
37: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
40: areturn
~~~

其中

- 17 表示创建对象，将对象引用入栈 // new Singleton
- 20 表示复制一份对象引用 // 引用地址
- 21 表示利用一个对象引用，调用构造方法
- 24 表示利用一个对象引用，赋值给 static INSTANCE

也许 jvm 会优化为：先执行 24，再执行 21。如果两个线程 t1，t2 按如下时间序列执行：

![1608956513132](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/26/122155-464624.png)

- 关键在于 0: getstatic 这行代码在 monitor 控制之外，它就像之前举例中不守规则的人，可以越过 monitor 读取INSTANCE 变量的值
- 这时 t1 还未完全将构造方法执行完毕，如果在构造方法中要执行很多初始化操作，那么 t2 拿到的是将是一个未初始化完毕的单例
- 对 INSTANCE 使用 volatile 修饰即可，可以禁用指令重排，但要注意在 JDK 5 以上的版本的 volatile 才会真正有效

> 注意：上面说过，synchronized可以保证操作的可见性，原子性，有序性，但是上面INSTANCE 对象的创建过程还是出现了问题，所以synchronized可以保证操作的可见性，原子性，有序性是有一个前提的，也就是共享变量完全被synchronized锁保护的时候，那么共享变量的使用是没有有序性，原子性和可见性的问题，但是上面的共享变量INSTANCE 并没有被完全的保护，因为在锁之外还有一次判断操作。

#### double-checked locking 解决

~~~ java
public final class Singleton {
    private Singleton() { }
  //添加volatile关键字即可
    private static volatile Singleton INSTANCE = null;
    public static Singleton getInstance() {
        // 实例没创建，才会进入内部的 synchronized代码块
        if (INSTANCE == null) {
            synchronized (Singleton.class) { // t2
                // 也许有其它线程已经创建实例，所以再判断一次
                if (INSTANCE == null) { // t1
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
~~~

字节码上看不出来 volatile 指令的效果

~~~ java
// -------------------------------------> 加入对 INSTANCE 变量的读屏障
0: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
3: ifnonnull 37
6: ldc #3 // class cn/itcast/n5/Singleton
8: dup
9: astore_0
10: monitorenter -----------------------> 保证原子性、可见性
11: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
14: ifnonnull 27
17: new #3 // class cn/itcast/n5/Singleton
20: dup
21: invokespecial #4 // Method "<init>":()V
24: putstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
// -------------------------------------> 加入对 INSTANCE 变量的写屏障
27: aload_0
28: monitorexit ------------------------> 保证原子性、可见性
29: goto 37
32: astore_1
33: aload_0
34: monitorexit
35: aload_1
36: athrow
37: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
40: areturn
~~~

- 如上面的注释内容所示，读写 volatile 变量时会加入内存屏障（Memory Barrier（Memory Fence）），保证下面两点：

  - 可见性

    写屏障（sfence）保证在该屏障之前的 t1 对共享变量的改动，都同步到主存当中
    而读屏障（lfence）保证在该屏障之后 t2 对共享变量的读取，加载的是主存中最新数据

  - 有序性

    写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后
    读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前

  - 更底层是读写变量时使用 lock 指令来多核 CPU 之间的可见性与有序性

**图解**

下面加24put()之前添加写屏障，也就是put()之前的指令不可以被重排序到写屏障之后。如果t2在put之前获取数据，那么获取的是Null值，那么这个时候可以由synchronized来保证安全性。

![1608957604229](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/26/124007-573842.png)

### happens-before

`happens-before` 规定了**对共享变量的写操作对其它线程的读操作可见**，它是**可见性与有序性**的一套规则总结，抛开以下 happens-before 规则，`JMM` 并不能保证一个线程对共享变量的写，对于其它线程对该共享变量的读可见

- 线程**解锁 m** `之前对变量的写，对于接下来对 `m `加锁的其它线程对该变量的读可见,`synchronized`会保证对共享变量的原子性，可见性，有序性。

~~~ java
 //synchronized对共享变量的影响
    static int x;
    static Object m = new Object();
new Thread(()->{
synchronized(m){
        x=10;
        }
        },"t1").start();
        new Thread(()->{
synchronized(m){
        System.out.println(x);
        }
        },"t2").start();
~~~

- 线程对 `volatile` 变量的写，**对接下来其它线程对该变量的读可见**

~~~ java
//volatile修饰的变量，一个线程对变量的修改，对其他线程可见
//volatile本来就是保证可见性问题的
volatile static int x;
new Thread(()->{
        x = 10;
        },"t1").start();
new Thread(()->{
        System.out.println(x);
        },"t2").start();
~~~

- 线程` start `前对变量的写，对该线程开始后对该变量的读可见

~~~ java
static int x;
x = 10;
new Thread(()->{
 System.out.println(x);
},"t2").start();
~~~

- 线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用` t1.isAlive()` 或 `t1.join()`等待它结束）

~~~ java
static int x;
Thread t1 = new Thread(()->{
 x = 10;
},"t1");
t1.start();
t1.join();
System.out.println(x);
~~~

- 线程` t1` 打断 `t2`（`interrupt`）前对变量的写，对于其他线程得知 `t2 `被打断后对变量的读可见（通过
  `t2.interrupted 或 t2.isInterrupted`）

~~~ java
static int x;
    public static void main(String[] args) {
        Thread t2 = new Thread(()->{
            while(true) {
                if(Thread.currentThread().isInterrupted()) {
                    System.out.println(x);
                    break;
                }
            }
        },"t2");
        t2.start();
        new Thread(()->{
            sleep(1);
            x = 10;
          //在这里吧t2线程打断，但是在打断之前对变量已经修改，这个修改对t2线程是可见的
            t2.interrupt();
        },"t1").start();
        while(!t2.isInterrupted()) {
            Thread.yield();
        }
        System.out.println(x);
    }
~~~

- 对变量默认值（0，false，null）的写，对其它线程对该变量的读可见
- 具有传递性，如果` x hb-> y `并且` y hb-> z` 那么有 `x hb-> z `，配合 `volatile` 的防指令重排，有下面的例子

~~~ java
volatile static int x;
static int y;
new Thread(()->{
 y = 10;
 x = 20;
  //在写入x之前会添加一个写屏障，屏障之前的内容都会同步到主存当中
},"t1").start();
new Thread(()->{
 // x=20 对 t2 可见, 同时 y=10 也对 t2 可见
  //因为虚拟机会在读之前添加读屏障，读屏障之前的代码全部会同步到主存内
 System.out.println(x);
},"t2").start();
~~~

> 变量都是指成员变量或静态成员变量

### balking 模式习题

希望 doInit() 方法仅被调用一次，下面的实现是否有问题，为什么？

~~~ java
public class TestVolatile {
 volatile boolean initialized = false;
 void init() {
 if (initialized) {
 return;
 }
 doInit();
 initialized = true;
 }
 private void doInit() {
 }
}
//不会仅仅调用一次，因为initialized被volatile修饰，仅仅可以保证共享变量的可见性，代码中有多次对共享变量进行修改，并不能保证代码执行的原子性，所以有可能两个线程都会执行到initialized = true;，但是都还没有对共享变量进行修改，所以可能执行多次方法，解决方法是用synchronized保证原子性
~~~

> volatile适用于一个线程写，其他线程读的情况，这样可以保证写线程的写对所有的读线程均可见。

### 线程安全单例练习

单例模式有很多实现方法，饿汉、懒汉、静态内部类、枚举类，试分析每种实现下**获取单例对象**（即调用
getInstance）时的线程安全，并思考注释中的问题

> 饿汉式：类加载就会导致该单实例对象被创建
> 懒汉式：类加载不会导致该单实例对象被创建，而是首次使用该对象时才会创建

**实现一**

~~~ java
// 问题1：为什么加 final-->防止有子类继承，然后重写其中的方法，破坏单例模式
// 问题2：如果实现了序列化接口, 还要做什么来防止反序列化破坏单例
public final class Singleton implements Serializable {
 // 问题3：为什么设置为私有? 是否能防止反射创建新的实例?--》防止调用构造器创建对象，不能，利用反射可以重新设置构造器的可见性
 private Singleton() {}
 // 问题4：这样初始化是否能保证单例对象创建时的线程安全?--》可以，静态变量的初始化操作是在类加载阶段完成，类加载阶段是由jvm保证线程安全性的
 private static final Singleton INSTANCE = new Singleton();//饿汉式，类加载时候已经初始化
 // 问题5：为什么提供静态方法而不是直接将 INSTANCE 设置为 public, 说出你知道的理由
  //用方法提供更好的封装性，可以实现懒惰的初始化，还可以支持泛型
 public static Singleton getInstance() {
 return INSTANCE;
 }
  //问题二的答案，这样做返回的时候回直接返回单例的那个对象，而不会直接返回使用反序列化生成的那个对象
 public Object readResolve() {
 return INSTANCE;
 }
}
~~~

**实现二**

~~~ java
// 问题1：枚举单例是如何限制实例个数的,枚举类里面的变量其实就是静态变量，也就是实例的个数
// 问题2：枚举单例在创建时是否有并发问题，没有，应为其成员变量也是静态的，会在类加载阶段初始化，由虚拟机保证线程的安全性
// 问题3：枚举单例能否被反射破坏单例，不能
// 问题4：枚举单例能否被反序列化破坏单例，可以避免反序列化时候出现问题，不需要自己干预
// 问题5：枚举单例属于懒汉式还是饿汉式,饿汉式，类加载时候已经初始化
// 问题6：枚举单例如果希望加入一些单例创建时的初始化逻辑该如何做，添加构造方法
enum Singleton {
 INSTANCE;
}
~~~

**实现三**

~~~ java
public final class Singleton {
 private Singleton() { }
 private static Singleton INSTANCE = null;
 // 分析这里的线程安全, 并说明有什么缺点
  //把锁添加在静态方法上相当于添加在class对象上面，可以控制对静态变量的访问，缺点是锁的范围太大，第一次调用创建对象加锁，以后每一次获取对象，都要加锁，降低性能 
 public static synchronized Singleton getInstance() {
 if( INSTANCE != null ){
 return INSTANCE;
 }
 INSTANCE = new Singleton();//懒汉式
 return INSTANCE;
 }
}
//可以保证线程安全问题，锁住的是类对象
//如果一个对象是null，那么不能加锁，因为无法关联monitor对象
~~~

> 对象是null的话不可以加锁

**实现四**

~~~ java
public final class Singleton {
    private Singleton() { }
    // 问题1：解释为什么要加 volatile ?
 // synchronized代码块中的指令会发生重排序，对于synchronized代码块外面的指令，有可能先做赋值操作，然后在进行初始化，所以外面拿到的引用对象就没有进行初始化，也就是没有调用构造方法
  instance对象没有完全被synchronized锁保护，所以可能会发生指令重排序操作，使用volatile可以避免这种情况发生，也就是说对于获取锁的线程，先对instance做赋值操作，然后在调用构造方法，对于外面的线程，此时instance已经不是空，那么获取的对象实例还没有经过调用沟改造方法初始化，所以使用vaolatie可以防止指令排序操作，也就是防止synchronized内部重排序，即使赋值操作和调用构造方法之间重排序
    private static volatile Singleton INSTANCE = null;

    // 问题2：对比实现3, 说出这样做的意义
  第一次调用需要加锁初始化对象，后续再次获取对象的时候就不需要进行加锁操作，可以支架获取锁。方法三实现每次都粗腰加锁，开销很大
  
    public static Singleton getInstance() {
        if (INSTANCE != null) {
            return INSTANCE;
        }
        synchronized (Singleton.class) {
            // 问题3：为什么还要在这里加为空判断, 之前不是判断过了吗
          //为了防止首次去创建对象，多个线程并发问题造成创建对象超过一个
          //比如t1线程首次来获取对象，发现对象是空的，那么就会获取锁创建对象，但是这个时候换没有创建对象，也就是对象的初始化没有完成，此时t2线程也来获取对象，发现对象是空，也进来创建对象，但是这个时候无法获取到锁，那么就会阻塞，当t1线程创建好对象退出时候，这个时候t2线程获取到了锁，然后创建对象，这种情况就创建了2个对象，所以需要在锁内部再次进行判断，这样就可以避免那种情况的发生
            if (INSTANCE != null) { // t2
                return INSTANCE;
            }
            INSTANCE = new Singleton();
            return INSTANCE;
        }
    }
}
~~~

**实现五**

~~~ java
public final class Singleton {
    private Singleton() { }
    // 问题1：属于懒汉式还是饿汉式
    private static class LazyHolder {
      //懒汉式，因为类的加载机制就是懒汉式，首次使用的时候才会进行加载，所以第一次加载并不会加载内部类，第一次只会加载外部的来，内部类并不会被加载，所以内部的静态变量也不会初始化，只有使用到的时候才会进行加载
        static final Singleton INSTANCE = new Singleton();
    }
    // 问题2：在创建时是否有并发问题，没有，线程安全，由jvm保证线程安全
    public static Singleton getInstance() {
        return LazyHolder.INSTANCE;
    }
}
~~~

### 本章小结

本章重点讲解了 JMM 中的

- 可见性 - 由 JVM 缓存优化引起
- 有序性 - 由 JVM 指令重排序优化引起
- happens-before 规则
- 原理方面
  - CPU 指令并行
  - volatile
- 模式方面
  - 两阶段终止模式的 volatile 改进
  - 同步模式之 balking（用于只希望某一段代码执行一次的情况）

reetrantlock模式问题

线程八锁问题