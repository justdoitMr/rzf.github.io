
<!-- TOC -->

- [Java线程](#java线程)
  - [创建线程的方法](#创建线程的方法)
    - [方式一：继承于`Thread`类](#方式一继承于thread类)
    - [方式二：使用 Runnable 配合 Thread](#方式二使用-runnable-配合-thread)
    - [原理之 Thread 与 Runnable 的关系](#原理之-thread-与-runnable-的关系)
    - [方法三，FutureTask 配合 Thread](#方法三futuretask-配合-thread)
  - [查看进程的方法](#查看进程的方法)
    - [windows](#windows)
    - [linux](#linux)
    - [Java](#java)
    - [查看进程和线程的工具](#查看进程和线程的工具)
  - [线程运行的原理](#线程运行的原理)
    - [jvm的栈与栈帧](#jvm的栈与栈帧)
    - [线程的运行原理](#线程的运行原理)
    - [线程的运行原理（多线程）](#线程的运行原理多线程)
    - [线程的上下文切换（Thread Context Switch）](#线程的上下文切换thread-context-switch)
  - [常见方法](#常见方法)
    - [Start()与Run()](#start与run)
    - [获取线程的状态](#获取线程的状态)
    - [sleep 与 yield](#sleep-与-yield)
    - [线程优先级](#线程优先级)
  - [案例展示](#案例展示)
    - [使用Sleep来限制对cpu的使用](#使用sleep来限制对cpu的使用)
  - [Join()方法](#join方法)
    - [同步](#同步)
    - [有时效的join](#有时效的join)
  - [interrupt 方法详解](#interrupt-方法详解)
    - [打断park()线程](#打断park线程)
  - [不推荐使用的方法](#不推荐使用的方法)
  - [多线程编程模式](#多线程编程模式)
    - [两阶段终止模式](#两阶段终止模式)
  - [主线程与守护线程](#主线程与守护线程)
  - [线程的状态转换](#线程的状态转换)
  - [本章小结](#本章小结)
- [思维导图总结](#思维导图总结)

<!-- /TOC -->

## Java线程

### 创建线程的方法

#### 方式一：继承于`Thread`类

1. 创建一个继承于`Thread`类的子类
2. 重写`Thread`类的`run()` --> 将此线程执行的操作声明在`run()`中
3. 创建`Thread`类的子类的对象
4. 通过此对象调用`start()`

> 主方法也对应一个`main()`线程，这是一个`java`程序启动默认的线程

- 代码测试

```java
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
```

Thread继承与Runnable接口。

![1612182587865](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/202949-652462.png)

- 测试二,使用匿名内部类的方法

```java
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
```

#### 方式二：使用 Runnable 配合 Thread

创建多线程的方式二：实现`Runnable`接口

1. 创建一个实现了`Runnable`接口的类
2. 实现类去实现`Runnable`中的抽象方法：run()
3. 创建实现类的对象
4. 将此对象作为参数传递到`Thread`类的构造器中，创建`Thread`类的对象
5. 通过`Thread`类的对象调用`start()`

**查看Runnable接口的源码**

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();//只有一个run()抽象方法
}
```

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

```java
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
```

- 比较创建线程的两种方式。
  - 开发中：优先选择：实现`Runnable`接口的方式，原因：实现的方式没有类的单继承性的局限性，实现的方式更适合来处理多个线程有共享数据的情况。 

#### 原理之 Thread 与 Runnable 的关系

**从源码的角度看看`Runnalle`的工作原理**

```java
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
```

**源码角度理解`Thread`方法**

```java
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
```

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

```java
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
```

**代码演示**

```java
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

```

- `Future`提供了三种功能：   　　

1. 判断任务是否完成；   　　
2. 能够中断任务；   　　
3. 能够获取任务执行结果。

> 多个线程的执行顺序是有底层的调度器进行调度的，无法干预。

### 查看进程的方法

#### windows

```java
//任务管理器可以查看进程和线程数，也可以用来杀死进程
tasklist 查看进程
taskkill 杀死进程
tasklist | findstr java //通过筛选查看java进程
taskkill /F /PID 号码 //强制杀死某一个进程
jps //是jdk自带的查看进程的命令
```

#### linux

```java
ps -fe //查看所有进程
ps -fT -p 进程号 //查看某个进程（PID）的所有线程
kill  进程号 //杀死进程的pid
top //按大写 H 切换是否显示线程,top命令可以动态展示进程的信息
top -H（表示查看某一个线程） -p（表示进程的id）// 查看某个进程（PID）的所有线程
//使用grep和管道运算符
ps -fe  | grep java(表示关键字)
```

#### Java

```java
javac // 编译
java //运行java程序
jps 命令查看所有 Java 进程
jstack 进程pid // 查看某个 Java 进程（PID）的所有线程状态
jconsole //来查看某个 Java 进程中线程的运行情况（图形界面）
```

#### 查看进程和线程的工具

在cmd窗口中输入jconsole。

### 线程运行的原理

#### jvm的栈与栈帧

拟虚拟机栈描述的是`Java`方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧(`stack  frame`)用于存储**局部变量表、操作数栈、动态链接、方法出口等信息**，是属于线程的私有的。当`java`中使用多线程时，每个线程都会维护它自己的栈帧！每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法。

#### 线程的运行原理

**栈与栈帧的演示**

```java
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
```

1. 当我们启动一个`java`程序的时候，就相当于启动一个`jvm`进程，`jvm`进程会分配到运行时数据区的内存资源，`cpu`等资源，首先会在栈中存放主线程的栈帧，压入栈底部，同时我们也可以看到`main`线程对应的参数,每一个方法的参数和局部变量信息存放在其对应栈帧中的局部变量表中。

![1608084042999](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/113634-678149.png)

2. 接下来我们进入`test01()`方法的内部，那么在`java`栈中会压入一个栈帧存放`test01()`方法的运行信息，同时把`test01()`方法中的参数和局部变量全部存储到`test01()`方法对应的局部变量表中。

![1608084325812](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/100527-364772.png)

3. 现在`test02()`方法也被压入栈帧当中。

![1643157000600](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/26/083001-969245.png)

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

```java
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
```

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

```java
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
```

通过上面的输出我们发现，虽然新创建了一个线程去执行别的操作，但是最后打印出执行操作的是`main`线程，并不是我们新创建的线程`t1`。下面我们调用`start`启动线程。

- 调用`start`启动线程

```java
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
```

输出结果展示`jvm`重新给我们新启动了一个线程`t1`去执行我们的代码，而不再是主线程去执行。

**解释**：如果直接调用`run`方法去启动线程，其实`jvm`是没有给我们新创建线程，而是使用的是主线程去执行我们的所有操作，但是如果使用的是`start`方法启动，那么`jvm`会为我们新创建一个线程去执行其他的代码。

#### 获取线程的状态

```java
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
```

#### sleep 与 yield

**Sleep**

调用 sleep 会让当前线程从 Running 进入 Timed Waiting 状态（阻塞）

2. 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException
3. 睡眠结束后的线程未必会立刻得到执行，也就是线程从**阻塞—>就绪状态。**
4. 建议用 `TimeUnit` 的 sleep 代替 Thread 的 sleep 来获得更好的可读性

```java
 TimeUnit.SECONDS.sleep(1000);
```

5. Sleep方法写在哪一个线程里面，就阻塞哪一个线程。

**代码测试**

```java
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
```

**yield**

1. 调用 yield 会让当前线程从 Running 进入 Runnable 就绪状态，然后调度执行其它线程，也就是使线程从**运行—>就绪。**
2. 具体的实现依赖于操作系统的任务调度器，可能当前没有就绪的线程，那么此时调度器还会把资源分配给当前线程使用。

#### 线程优先级

- 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它
- 如果` cpu `比较忙，那么优先级高的线程会获得更多的时间片，但` cpu `闲时，优先级几乎没作用

**代码演示**

```java
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
```

虽然可以设置线程的优先级，但是最终线程的运行状况还是由我们的任务调度器决定的，这两个方法最终仅仅是给任务调度器一个提示而已，我们是无法干预的。

### 案例展示

#### 使用Sleep来限制对cpu的使用

- 在没有利用 `cpu` 来计算时，不要让 while(true) 空转浪费` cpu`，这时可以使用 yield 或 sleep 来让出 `cpu `的使用权给其他程序

```java
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
```

**`cpu`占用率**：此时发现占用率很高，第一个`java`程序就是我们运行这的程序。但是如果添加上sleep语句，`cpu`的利用率很快就可以降下来。

![1608108862899](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/16/165423-409788.png)

- 可以用 wait 或 条件变量达到类似的效果
- 不同的是，后两种都需要加锁，并且需要相应的唤醒操作，一般适用于要进行同步的场景
- sleep 适用于无需锁同步的场景

### Join()方法

**为什么会需要join方法**，先来看看下面的程序输出什么

```java
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
```

- 分析
  - 因为主线程和线程`t1 `是并行执行的，`t1 `线程需要 1 秒之后才能算出 r=10
  - 而主线程一开始就要打印 r 的结果，所以只能打印出 r=0
- 解决方法
  - 用 sleep 行不行？为什么？不可以，因为我们的子线程的运行是不可预测的，也就是我们不知道从运行开始到结束花费多少时间，也就是我们的主线程等待的时间不好计算。
  - 用 join，加在` t1.start()` 之后即可，join()方法是等待一个线程结束，哪一个线程对象调用此方法，join()就等待哪一个线程。

**代码测试**

```java
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
```

**图解**

![1608184506504](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/17/135508-560676.png)

#### 同步

- 需要等待结果返回，才能继续运行就是同步
- 不需要等待结果返回，就能继续运行就是异步

**代码说明**

```java
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
```

- 分析如下
  - 第一个 join：等待 t1 时, t2 并没有停止, 而在运行
  - 第二个 join：1s 后, 执行到此, t2 也运行了 1s, 因此也只需再等待 1s
  - 如果颠倒两个 join 呢？输出结果一致

**图解等待过程**

![1608185219386](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/17/140702-314322.png)

#### 有时效的join

**代码说明**

```java
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

```



### interrupt 方法详解

**打断 sleep，wait，join 的线程**

- 这几个方法都会让线程进入阻塞状态
- 打断 `sleep `的线程, 会清空打断状态，也就是会设置打断状态为`false`，一般情况下，对于正常运行的程序，被其他线程打断后，其打断状态是真，以 sleep 为例。

**代码说明**

```java
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
```

**打断正常运行的程序**

**打断正常运行的线程, 不会清空打断状态**，打断正常运行的程序，程序不会立马进入阻塞状态，相反还会继续执行，我们可以根据打断状态标记做一些其他的工作，然后在结束线程的执行。

**代码说明**

```java
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
```

#### 打断park()线程

打断 `park` 线程, 不会清空打断状态，也就是打断之后，状态是`true`。

**代码说明**

```java
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
```

**修改状态标记**

> 可以使用 `Thread.interrupted()` 清除打断状态

```java
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

```

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

```java
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
```

### 主线程与守护线程

默认情况下，`Java` 进程需要等待所有线程都运行结束，才会结束。有一种特殊的线程叫做守护线程，只要其它非守护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束。

**代码说明**

```java
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
```

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

```java
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
```

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