<!-- TOC -->

- [共享模型之无锁(乐观锁)](#共享模型之无锁乐观锁)
  - [问题提出](#问题提出)
    - [解决思路-加锁](#解决思路-加锁)
    - [使用无锁的方式解决](#使用无锁的方式解决)
  - [CAS与Volatile](#cas与volatile)
    - [volatile](#volatile)
    - [为什么无锁效率高](#为什么无锁效率高)
    - [CAS的特点](#cas的特点)
  - [原子类型](#原子类型)
    - [原子整数](#原子整数)
    - [原子引用](#原子引用)
      - [AtomicReference](#atomicreference)
      - [ABA问题](#aba问题)
      - [**AtomicStampedReference解决**](#atomicstampedreference解决)
      - [**AtomicMarkableReference**](#atomicmarkablereference)
    - [原子数组](#原子数组)
    - [字段更新器](#字段更新器)
  - [原子累加器](#原子累加器)
    - [LongAdder源码解读](#longadder源码解读)
    - [原理之伪共享](#原理之伪共享)
  - [UnSafe](#unsafe)
    - [概述](#概述)
    - [Unsafe CAS 操作](#unsafe-cas-操作)
    - [CAS自定义原子整数类](#cas自定义原子整数类)
  - [小结](#小结)
- [共享模型之不可变](#共享模型之不可变)
  - [日期转换的问题](#日期转换的问题)
    - [不可变类的设计](#不可变类的设计)
    - [保护性拷贝](#保护性拷贝)
  - [享元模式](#享元模式)
      - [介绍](#介绍)
      - [包装类](#包装类)
    - [自定义数据库连接池](#自定义数据库连接池)
  - [Final原理（静态成员变量或者成员变量）](#final原理静态成员变量或者成员变量)
    - [设置Final变量的原理](#设置final变量的原理)
    - [获取final变量的原理](#获取final变量的原理)
  - [小结](#小结-1)
- [线程池](#线程池)
  - [自定义线程池](#自定义线程池)
  - [ThreadPoolExecutor线程池](#threadpoolexecutor线程池)
    - [线程池的状态](#线程池的状态)
    - [构造方法](#构造方法)
    - [Executors工具](#executors工具)
      - [newFixedThreadPool（固定线程数）](#newfixedthreadpool固定线程数)
      - [newCachedThreadPool（带缓冲功能）](#newcachedthreadpool带缓冲功能)
      - [newSingleThreadExecutor（单线程执行器）](#newsinglethreadexecutor单线程执行器)
    - [提交任务](#提交任务)
    - [关闭线程池](#关闭线程池)
  - [异步模式之工作线程](#异步模式之工作线程)
    - [定义](#定义)
    - [饥饿](#饥饿)
    - [创建多少线程合适](#创建多少线程合适)
  - [任务调度线程池（延时执行任务）](#任务调度线程池延时执行任务)
    - [使用Timer类](#使用timer类)
    - [ScheduledExecutorService](#scheduledexecutorservice)
    - [定时执行任务](#定时执行任务)
    - [scheduleWithFixedDelay](#schedulewithfixeddelay)
    - [正确处理异常信息](#正确处理异常信息)
    - [应用-定时执行任务](#应用-定时执行任务)
  - [Fork/Join线程池](#forkjoin线程池)
    - [概念](#概念)
    - [使用](#使用)
    - [改进](#改进)
- [并发工具JUC](#并发工具juc)
  - [AQS 原理](#aqs-原理)
    - [概述](#概述-1)
    - [如何使用](#如何使用)
      - [**获取锁**](#获取锁)
      - [**释放锁**](#释放锁)
    - [实现不可重入锁](#实现不可重入锁)
      - [自定义同步器](#自定义同步器)
      - [实现自定义锁](#实现自定义锁)
  - [LOCK接口](#lock接口)
- [ReentrantLock 原理(重入锁)](#reentrantlock-原理重入锁)
  - [非公平锁的实现原理](#非公平锁的实现原理)
    - [加锁源码解读](#加锁源码解读)
    - [解锁源码](#解锁源码)
  - [可重入原理](#可重入原理)
  - [可打断模式](#可打断模式)
    - [不可打断模式](#不可打断模式)
    - [可打断模式](#可打断模式-1)
  - [公平锁实现原理](#公平锁实现原理)
  - [条件变量的实现原理](#条件变量的实现原理)
    - [await 流程](#await-流程)
    - [Signal流程](#signal流程)
    - [源码](#源码)
- [读写锁](#读写锁)
  - [ReentrantReadWriteLock](#reentrantreadwritelock)
  - [**注意事项**](#注意事项)
  - [读写锁原理](#读写锁原理)
    - [图解原理](#图解原理)
  - [StampedLock(读锁)](#stampedlock读锁)
- [LockSupport](#locksupport)
- [Semaphore](#semaphore)
  - [semaphore的原理](#semaphore的原理)
    - [加锁原理](#加锁原理)
    - [释放锁](#释放锁-1)
  - [Acquire原理](#acquire原理)
- [CountdownLatch](#countdownlatch)
  - [配合线程池](#配合线程池)
  - [模拟游戏加载](#模拟游戏加载)
- [CyclicBarrier](#cyclicbarrier)
- [线程安全集合类](#线程安全集合类)
  - [分类](#分类)

<!-- /TOC -->
## 共享模型之无锁(乐观锁)

### 问题提出

有如下需求，保证 account.withdraw 取款方法的线程安全

**代码说明**

~~~ java
public class Test39 {
    public static void main(String[] args) {
        Account.demo(new AccountUnsafe(10000));
    }
}

interface Account {
    // 获取余额
    Integer getBalance();
    // 取款
    void withdraw(Integer amount);
    /**
     * 方法内会启动 1000 个线程，每个线程做 -10 元 的操作
     * 如果初始余额为 10000 那么正确的结果应当是 0
     */
    static void demo(Account account) {
        List<Thread> ts = new ArrayList<>();
        long start = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            ts.add(new Thread(() -> {
                account.withdraw(10);
            }));
        }
        ts.forEach(Thread::start);
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        long end = System.nanoTime();
        System.out.println(account.getBalance()
                + " cost: " + (end-start)/1000_000 + " ms");
    }
}
class AccountUnsafe implements Account {
    private Integer balance;
    public AccountUnsafe(Integer balance) {
        this.balance = balance;
    }
    @Override
    public Integer getBalance() {
        return balance;
    }
    @Override
    public void withdraw(Integer amount) {
        balance -= amount;
    }
}
~~~

#### 解决思路-加锁

解决方法是使用`synchronized`锁去保护我们共享资源的安全性。

~~~ java
 @Override
    public Integer getBalance() {
        synchronized (this){
            return balance;
        }
    }
    @Override
    public void withdraw(Integer amount) {
       synchronized (this){
           balance -= amount;
       }
    }
~~~

#### 使用无锁的方式解决

**代码说明**

~~~ java
public class Test39 {
    public static void main(String[] args) {
        Account.demo(new AccountsCas(10000));
    }
}

interface Account {
    // 获取余额
    Integer getBalance();
    // 取款
    void withdraw(Integer amount);
    /**
     * 方法内会启动 1000 个线程，每个线程做 -10 元 的操作
     * 如果初始余额为 10000 那么正确的结果应当是 0
     */
    static void demo(Account account) {
        List<Thread> ts = new ArrayList<>();
        long start = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            ts.add(new Thread(() -> {
                account.withdraw(10);
            }));
        }
        ts.forEach(Thread::start);
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        long end = System.nanoTime();
        System.out.println(account.getBalance()
                + " cost: " + (end-start)/1000_000 + " ms");
    }
}

//基于无锁的实现
class AccountsCas implements Account{
    private AtomicInteger balance;//原子整数

    public AccountsCas(int balance) {
        this.balance = new AtomicInteger(balance);
    }

    @Override
    public Integer getBalance() {
        return balance.get();
    }

    @Override
    public void withdraw(Integer amount) {
        while (true){
//            获取余额的最新值
            int pre=balance.get();
//            next就是修改后的余额
            int next=pre-amount;
//            上面的步骤都是在线程的工作内存中完成，并没有同步到主存当中
//            第一个参数表示从主存中获取的最新值，第二个表示修改后的值
            if(balance.compareAndSet(pre,next)){
                break;
//                如果修改成功就返回true，然后退出
            }
        }

    }
}
~~~

### CAS与Volatile

前面看到的 AtomicInteger 的解决方法，内部并没有用锁来保护共享变量的线程安全。那么它是如何实现的呢？

~~~ java
public void withdraw(Integer amount) {
        while(true) {
            // 需要不断尝试，直到成功为止
            while (true) {
                // 比如拿到了旧值 1000
                int prev = balance.get();
                // 在这个基础上 1000-10 = 990
                int next = prev - amount;
                 /*
                 compareAndSet 正是做这个检查，在 set 前，先比较 prev 与当前值
                 - 不一致了，next 作废，返回 false 表示失败
                 比如，别的线程已经做了减法，当前值已经被减成了 990
                 那么本线程的这次 990 就作废了，进入 while 下次循环重试
                 - 一致，以 next 设置为新值，返回 true 表示成功
                 */
              //比较并设置，compareAndSet（）方法是原子操作
                if (balance.compareAndSet(prev, next)) {
                    break;
                }
            }
        }
    }
~~~

其中的关键是 `compareAndSet`，它的简称就是 CAS （也有 Compare And Swap 的说法），它必须是原子操作,此方法是在底层`cpu`层面实现的原子操作。

**图解**

![1609075267184](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/27/212108-861949.png)

线程一每一次都会把`pre`的值和`Account`对象的值进行比较，如果不相等，那么就是这次修改失败，等待下一次重新获取修改，否则相等，就可以修改`Account`的值。

> 其实` CAS` 的底层是 `lock cmpxchg `指令（X86 架构），在单核` CPU` 和多核 `CPU `下都能够保证【比较-交
> 换】的原子性。
>
> 在多核状态下，某个核执行到带 `lock `的指令时，`CPU` 会让总线锁住，当这个核把此指令执行完毕，再
> 开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子
> 的。

#### volatile

**AtomicInteger源码**

~~~ java
//具体存储的值保存在下面的属性中
private volatile int value;
cas必须配合volatile才可以获取最新值
~~~

- 获取共享变量时，为了保证该变量的可见性，需要使用` volatile `修饰。
- 它可以用来修饰**成员变量和静态成员变量**，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作` volatile` 变量都是直接操作主存。即一个线程对 `volatile` 变量的修改，对另一个线程可见。

> 注意
> volatile 仅仅保证了共享变量的可见性，让其它线程能够看到最新值，但不能解决指令交错问题（不能保证原子性）

`CAS` 必须借助 `volatile` 才能读取到共享变量的最新值来实现【比较并交换】的效果，也就是`cas`操作要对其他线程对共享变量的修改可见，如果共享变量没有使用`volatile`进行修饰，那么获取的可能就不是最新的值。

~~~ java
private volatile int value;
~~~

#### 为什么无锁效率高

- **无锁情况下，即使重试失败，线程始终在高速运行，没有停歇，而 synchronized 会让线程在没有获得锁的时候，发生上下文切换，进入阻塞。**
- 打个比喻线程就好像高速跑道上的赛车，高速运行时，速度超快，一旦发生上下文切换，就好比赛车要减速、熄火，等被唤醒又得重新打火、启动、加速... 恢复到高速运行，代价比较大
- 但无锁情况下，因为线程要保持运行，需要额外 CPU 的支持，CPU 在这里就好比高速跑道，没有额外的跑道，线程想高速运行也无从谈起，虽然不会进入阻塞，但由于没有分到时间片，仍然会进入可运行状态，还是会导致上下文切换。所以线程数要少于cpu的核心数，否则还会发生上下文切换。
- 线程数少于cpu核心数使用cas是非常合适的。

#### CAS的特点

结合 `CAS `和 `volatile `可以实现无锁并发，适用于线程数少、多核 `CPU `的场景下。

- `CAS` 是基于**乐观锁的**思想：最乐观的估计，不怕别的线程来修改共享变量，就算改了也没关系，我吃亏点再重试呗。
- `synchronized` 是基于**悲观锁**的思想：最悲观的估计，得防着其它线程来修改共享变量，我上了锁你们都别想改，我改完了解开锁，你们才有机会。
- `CAS `体现的是**无锁并发(不断循环重试)、无阻塞并发（无上下文切换）**，请仔细体会这两句话的意思
  - 因为没有使用 `synchronized`，所以线程不会陷入阻塞，这是效率提升的因素之一
  - 但如果竞争激烈，可以想到重试必然频繁发生，反而效率会受影响

### 原子类型

#### 原子整数

`J.U.C` 并发包提供了：（是工具包，所有的包装都是保证操作是原子的。底层都是基于`cas`算法实现）

- AtomicBoolean
- AtomicInteger
- AtomicLong

**继承关系**

![1620365258345](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/191152-430466.png)

~~~ java
public class AtomicInteger extends Number implements java.io.Serializable{}

//属性
private static final long serialVersionUID = 6214790243416807050L;
   // setup to use Unsafe.compareAndSwapInt for updates
private static final Unsafe unsafe = Unsafe.getUnsafe();//保证是原子操作
private static final long valueOffset;//基于偏移量去获取值
private volatile int value;//存储具体的值
~~~

**以 AtomicInteger 为例**

~~~ java
public class AtomicIntegerDemo {
    public static void main(String[] args) {
        AtomicInteger a=new AtomicInteger(0);
//        AtomicInteger的操作都是原子操作
//        System.out.println(a.incrementAndGet());//等价于++a
//        System.out.println(a.getAndIncrement());//a++
//        获取value的最新值
//        System.out.println(a.get());
//          获取并且增加
        System.out.println(a.getAndAdd(5));
//        先增加然后在获取
        System.out.println(a.addAndGet(5));
    }
}

AtomicInteger i = new AtomicInteger(0);
// 获取并自增（i = 0, 结果 i = 1, 返回 0），类似于 i++
System.out.println(i.getAndIncrement());
// 自增并获取（i = 1, 结果 i = 2, 返回 2），类似于 ++i
System.out.println(i.incrementAndGet());
// 自减并获取（i = 2, 结果 i = 1, 返回 1），类似于 --i
System.out.println(i.decrementAndGet());
// 获取并自减（i = 1, 结果 i = 0, 返回 1），类似于 i--
System.out.println(i.getAndDecrement());
// 获取并加值（i = 0, 结果 i = 5, 返回 0）
System.out.println(i.getAndAdd(5));
// 加值并获取（i = 5, 结果 i = 0, 返回 0）
System.out.println(i.addAndGet(-5));
// 获取并更新（i = 0, p 为 i 的当前值, 结果 i = -2, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.getAndUpdate(p -> p - 2));
// 更新并获取（i = -2, p 为 i 的当前值, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用，参数是蓝魔大表达式
System.out.println(i.updateAndGet(p -> p + 2));
// 获取并计算（i = 0, p 为 i 的当前值, x 为参数1, 结果 i = 10, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
// getAndUpdate 如果在 lambda 中引用了外部的局部变量，要保证该局部变量是 final 的
// getAndAccumulate 可以通过 参数1 来引用外部的局部变量，但因为其不在 lambda 中因此不必是 final
System.out.println(i.getAndAccumulate(10, (p, x) -> p + x));
// 计算并获取（i = 10, p 为 i 的当前值, x 为参数1, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.accumulateAndGet(-10, (p, x) -> p + x));

//模拟updataAndSet()操作
 /**
     * 对代码进行封装
     * @param a
     * IntUnaryOperator:操作参数，也即是对传进来的值做的操作
     */
    public static void updateAndGet(AtomicInteger a, IntUnaryOperator i){
        while (true){
//            首先获取最新的值
            int pre=a.get();
//            更新后的值
//            对计算的操作，作为参数传递，可以做各种运算
            int next=i.applyAsInt(pre);
            if(a.compareAndSet(pre,next)){
                break;
            }
        }
      //可以添加返回值
    }
~~~

**IntUnaryOperator接口**

~~~ java
@FunctionalInterface
public interface IntUnaryOperator {

    /**
     * Applies this operator to the given operand.
     *
     * @param operand the operand
     * @return the operator result
     */
    int applyAsInt(int operand);

    /**
     * Returns a composed operator that first applies the {@code before}
     * operator to its input, and then applies this operator to the result.
     * If evaluation of either operator throws an exception, it is relayed to
     * the caller of the composed operator.
     *
     * @param before the operator to apply before this operator is applied
     * @return a composed operator that first applies the {@code before}
     * operator and then applies this operator
     * @throws NullPointerException if before is null
     *
     * @see #andThen(IntUnaryOperator)
     */
    default IntUnaryOperator compose(IntUnaryOperator before) {
        Objects.requireNonNull(before);
        return (int v) -> applyAsInt(before.applyAsInt(v));
    }

    /**
     * Returns a composed operator that first applies this operator to
     * its input, and then applies the {@code after} operator to the result.
     * If evaluation of either operator throws an exception, it is relayed to
     * the caller of the composed operator.
     *
     * @param after the operator to apply after this operator is applied
     * @return a composed operator that first applies this operator and then
     * applies the {@code after} operator
     * @throws NullPointerException if after is null
     *
     * @see #compose(IntUnaryOperator)
     */
    default IntUnaryOperator andThen(IntUnaryOperator after) {
        Objects.requireNonNull(after);
        return (int t) -> after.applyAsInt(applyAsInt(t));
    }

    /**
     * Returns a unary operator that always returns its input argument.
     *
     * @return a unary operator that always returns its input argument
     */
    static IntUnaryOperator identity() {
        return t -> t;
    }
}
~~~

#### 原子引用

##### AtomicReference

为什么需要原子引用类型？

- AtomicReference
- AtomicMarkableReference
- AtomicStampedReference

**AtomicReference**

~~~ java
public class AtomicReference<V> implements java.io.Serializable {}
//属性信息
 		private static final long serialVersionUID = -1848883965231344442L;
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset
    private volatile V value;
~~~

**代码说明**

~~~ java
public interface DecimalAccount {
    // 获取余额
    BigDecimal getBalance();
    // 取款
    void withdraw(BigDecimal amount);
    /**
     * 方法内会启动 1000 个线程，每个线程做 -10 元 的操作
     * 如果初始余额为 10000 那么正确的结果应当是 0
     */
    static void demo(DecimalAccount account) {
        List<Thread> ts = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            ts.add(new Thread(() -> {
                account.withdraw(BigDecimal.TEN);
            }));
        }
        ts.forEach(Thread::start);
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println(account.getBalance());
    }
}
~~~

试着提供不同的 DecimalAccount 实现，实现安全的取款操作

**不安全的实现**

~~~ java
class DecimalAccountUnsafe implements DecimalAccount {
  //共享变量没有被保护
    BigDecimal balance;
    public DecimalAccountUnsafe(BigDecimal balance) {
        this.balance = balance;
    }
    @Override
    public BigDecimal getBalance() {
        return balance;
    }
    @Override
    public void withdraw(BigDecimal amount) {
        BigDecimal balance = this.getBalance();
        this.balance = balance.subtract(amount);
    }
}
~~~

**安全实现-使用锁**

~~~ java
class DecimalAccountSafeLock implements DecimalAccount {
  private final Object lock = new Object();
    BigDecimal balance;
    public DecimalAccountSafeLock(BigDecimal balance) {
        this.balance = balance;
    }
    @Override
    public BigDecimal getBalance() {
        return balance;
    }
    @Override
    public void withdraw(BigDecimal amount) {
        synchronized (lock) {
            BigDecimal balance = this.getBalance();
            this.balance = balance.subtract(amount);
        }
    }
}
~~~

**安全实现-使用 CAS**

~~~ java
class BigdecimalAccountCas implements DecimalAccount{
//使用原子引用类对balance进行包装
    private AtomicReference <BigDecimal> balance;

    public BigdecimalAccountCas(BigDecimal balance) {
        this.balance = new AtomicReference(balance);
    }

    @Override
    public BigDecimal getBalance() {
        return balance.get();
    }

    @Override
    public void withdraw(BigDecimal amount) {
        while (true){
//            获取最新的值
            BigDecimal pre=balance.get();
//            获取取款后的值
            BigDecimal next=pre.subtract(amount);
//            做更新操作
            if (balance.compareAndSet(pre,next)) {
                break;
            }
        }
    }
}
~~~

##### ABA问题

~~~ java
 static AtomicReference<String> ref = new AtomicReference<>("A");
    public static void main(String[] args) throws InterruptedException {
        System.out.println("main start...");;
        // 获取值 A
        // 这个共享变量被它线程修改过？
        String prev = ref.get();
        other();
        Thread.sleep(1);
        // 尝试改为 C
        System.out.printf("change A->C {}", ref.compareAndSet(prev, "C"));
    }
    private static void other() {
        new Thread(() -> {
            System.out.printf("change A->B {}", ref.compareAndSet(ref.get(), "B"));
        }, "t1").start();
        Thread.sleep(0.5);
       
        new Thread(() -> {
            System.out.printf("change B->A {}", ref.compareAndSet(ref.get(), "A"));
        }, "t2").start();
    }
//输出
11:29:52.325 c.Test36 [main] - main start...
11:29:52.379 c.Test36 [t1] - change A->B true
11:29:52.879 c.Test36 [t2] - change B->A true
11:29:53.880 c.Test36 [main] - change A->C true
~~~

- 主线程仅能判断出共享变量的值与最初值 A 是否相同，不能感知到这种从 A 改为 B 又 改回 A 的情况，如果主线程希望：只要有其它线程【动过了】共享变量，那么自己的 cas 就算失败，这时，仅比较值是不够的，需要再加一个版本号。使用版本号来区分是否有其他线程对共享变量进行了修改。

##### **AtomicStampedReference解决**

~~~ java
public class ABAdemo {
//    AtomicStampedReference多一个版本号,第二个参数设置版本号为0
    static AtomicStampedReference<String> ref = new AtomicStampedReference<>("A",0);
    public static void main(String[] args) throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"  main start...");;
        // 获取值 A
        // 这个共享变量被它线程修改过？
        String prev = ref.getReference();
//        获取版本号
        int stmp=ref.getStamp();
        System.out.println(Thread.currentThread().getName()+"  temp "+stmp);
        other();
        Thread.sleep(1);
        // 尝试改为 C,还需要输入版本号，并且版本号需要+1
        System.out.printf(Thread.currentThread().getName()+"  change A->C {}", ref.compareAndSet(prev, "C",stmp,stmp+1));
        System.out.println();
    }
    private static void other() throws InterruptedException {
        new Thread(() -> {
            int temp=ref.getStamp();
            System.out.println(Thread.currentThread().getName()+"  temp "+temp);
            System.out.printf(Thread.currentThread().getName()+"  change A->B {}", ref.compareAndSet(ref.getReference(), "B",temp,temp+1));
            System.out.println();
        }, "t1").start();
        Thread.sleep(1);

        new Thread(() -> {
            int temp=ref.getStamp();
            System.out.println(Thread.currentThread().getName()+"  temp "+temp);
            System.out.printf(Thread.currentThread().getName()+"  change B->A {}", ref.compareAndSet(ref.getReference(), "A",temp,temp+1));
            System.out.println();
        }, "t2").start();
    }
}
~~~

- `AtomicStampedReference` 可以给原子引用加上版本号，追踪原子引用整个的变化过程，如： `A -> B -> A ->C `，通过`AtomicStampedReference`，我们可以知道，引用变量中途被更改了几次。
- 但是有时候，并不关心引用变量更改了几次，只是单纯的关心是否更改过，所以就有了`AtomicMarkableReference`

**图示**

![1609141695831](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202012/28/154817-461459.png)

##### **AtomicMarkableReference**

~~~ java
public class AtomicMarkableReferenceDemo {
    public static void main(String[] args) throws InterruptedException {
//垃圾袋就相当于一个保护的资源
        Garbage garbage=new Garbage("装满垃圾");
//        bool参数变量表示维护共享变量的状态是否发生改变
        AtomicMarkableReference<Garbage>ref=new AtomicMarkableReference<>(garbage,true);
//      启动主线程
        System.out.println(Thread.currentThread().getName()+"  启动");
//        获取垃圾袋
        Garbage pre=ref.getReference();
        System.out.println(pre.toString());

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"  starting");
         garbage.setDesc("空垃圾袋");
         ref.compareAndSet(pre,garbage,true,false);
            System.out.println(garbage.toString());

        },"保洁阿姨").start();

        Thread.sleep(2);
        System.out.println("是否需要换新的垃圾袋");
//        两个布尔变量表示如果元来的值是true，才做替换操作，替换好后修改值为false,如果标记中途
//        被别的线程修改为false,那么久不做更新操作
        boolean success=ref.compareAndSet(pre,new Garbage("空的垃圾袋"),true,false);
        System.out.println("换了么"+success);
        System.out.println(ref.getReference().toString());


    }
}

class Garbage{
    String desc;

    public Garbage(String desc) {
        this.desc = desc;
    }

    public String getDesc() {
        return desc;
    }

    @Override
    public String toString() {
        return "Garbage{" +
                "desc='" + desc + '\'' +
                '}';
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }
}
~~~

#### 原子数组

用来保护数组里面的元素

- AtomicIntegerArray
- AtomicLongArray
- AtomicReferenceArray

**AtomicIntegerArray**

~~~ java
public class AtomicIntegerArray implements java.io.Serializable{}

public class AtomicReferenceArray<E> implements java.io.Serializable {}
~~~

**案例**

~~~ java
public class Test40 {

    public static void main(String[] args) {
      demo(
//              第一个参数
              ()-> new int[10],
//              第二个参数
              (array)->array.length,
//              第三个参数，表示对数据的操作
              (array,index)->array[index]++,
              array-> System.out.println(Arrays.toString(array))
      );

    }

    /**
     参数1，提供数组、可以是线程不安全数组或线程安全数组
     参数2，获取数组长度的方法
     参数3，自增方法，回传 array, index
     参数4，打印数组的方法
     */
//    函数式接口
// supplier 提供者 无中生有 ()->结果，没有参数，需要提供结果
// function 函数 一个参数一个结果 (参数)->结果 , BiFunction (参数1,参数2)->结果
// consumer 消费者 一个参数没结果 (参数)->void, BiConsumer (参数1,参数2)->
    private static <T> void demo(
            Supplier<T> arraySupplier,
            Function<T, Integer> lengthFun,
            BiConsumer<T, Integer> putConsumer,
            Consumer<T> printConsumer ) {
        List<Thread> ts = new ArrayList<>();
//        获取Supplier接口提供的数组
        T array = arraySupplier.get();
        int length = lengthFun.apply(array);
        for (int i = 0; i < length; i++) {
            // 每个线程对数组作 10000 次操作
            ts.add(new Thread(() -> {
                for (int j = 0; j < 10000; j++) {
                    putConsumer.accept(array, j%length);
                }
            }));
        }
        ts.forEach(t -> t.start()); // 启动所有线程
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }); // 等所有线程结束
        printConsumer.accept(array);
    }
}
//结果，每一个值应该是10000，但是现在都没有10000
[9803, 9830, 9803, 9799, 9803, 9812, 9805, 9816, 9813, 9797]
~~~

**使用原子数组**

~~~ java
public class Test40 {

    public static void main(String[] args) {
      demo(
//              第一个参数
              ()-> new int[10],
//              第二个参数
              (array)->array.length,
//              第三个参数，表示对数据的操作
              (array,index)->array[index]++,
              array-> System.out.println(Arrays.toString(array))

      );
        demo(
//              第一个参数
                ()-> new AtomicIntegerArray(10),
//              第二个参数
                (array)->array.length(),
//              第三个参数，表示对数据的操作
                (array,index)->array.getAndIncrement(index),
                array-> System.out.println(array)

        );

    }

    /**
     参数1，提供数组、可以是线程不安全数组或线程安全数组
     参数2，获取数组长度的方法
     参数3，自增方法，回传 array, index
     参数4，打印数组的方法
     */
//    函数式接口
// supplier 提供者 无中生有 ()->结果，没有参数，需要提供结果
// function 函数 一个参数一个结果 (参数)->结果 , BiFunction (参数1,参数2)->结果
// consumer 消费者 一个参数没结果 (参数)->void, BiConsumer (参数1,参数2)->
    private static <T> void demo(
            Supplier<T> arraySupplier,
            Function<T, Integer> lengthFun,
            BiConsumer<T, Integer> putConsumer,
            Consumer<T> printConsumer ) {
        List<Thread> ts = new ArrayList<>();
//        获取Supplier接口提供的数组
        T array = arraySupplier.get();
        int length = lengthFun.apply(array);
        for (int i = 0; i < length; i++) {
            // 每个线程对数组作 10000 次操作
            ts.add(new Thread(() -> {
                for (int j = 0; j < 10000; j++) {
                    putConsumer.accept(array, j%length);
                }
            }));
        }
        ts.forEach(t -> t.start()); // 启动所有线程
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }); // 等所有线程结束
        printConsumer.accept(array);
    }
}

//结果
[9090, 8966, 9058, 9066, 9823, 9824, 9837, 9815, 9817, 9838]
[10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000]
~~~

**函数式接口**

~~~ java
//提供无参数的方法，返回需要的结果
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
//提供一个参数，返回一个结果
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
}
//两个参数，返回一个结果
@FunctionalInterface
public interface BiConsumer<T, U> {

    /**
     * Performs this operation on the given arguments.
     *
     * @param t the first input argument
     * @param u the second input argument
     */
    void accept(T t, U u);

}
//一个参数没有结果
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

}
~~~

#### 字段更新器

用来保护某一个对象中的属性或者说是成员变量。保证多个线程访问成员变量时候，成员变量的线程安全性。

- AtomicReferenceFieldUpdater // 域 字段，引用类型
- AtomicIntegerFieldUpdater //整数类型
- AtomicLongFieldUpdater //长整型类型

利用字段更新器，可以针对对象的某个域（`Field`）进行原子操作，只能配合` volatile` 修饰的字段使用，否则会出现异常

**代码说明**

~~~ java
public class FieldUpdateDemo {

    public static void main(String[] args) {
        Student student=new Student();
//    创建字段引用更新器，保证对象属性赋值的原子性
//    类名，字段类型，字段名称
        AtomicReferenceFieldUpdater updater=
                AtomicReferenceFieldUpdater.newUpdater(Student.class,String.class,"name");
//保证对对象的属性的更新是原子操作
        updater.compareAndSet(student,null,"张三");
        System.out.println(student);
    }

}

class Student{
//    cas操作必须结合volatile来保证共享变量的可见性
    volatile String name;//字符串引用类型

    @Override
    public String toString() {
        return "Studend{" +
                "name='" + name + '\'' +
                '}';
    }
}
~~~

### 原子累加器

`jdk`中已经有了原子整数类，为什么需要原子累加器，还是效率问题，原子累加器的效率比较高。

原子累加器就是对一个数做累加操作。

![1620371101105](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/07/150502-883473.png)

**代码说明**

~~~ java
public class Test41 {

    public static void main(String[] args) {
//        使用原子长整型类别
        demo(
                ()->new AtomicLong(0),
                (adder)->adder.getAndIncrement()
        );
//        使用原子整数类
        demo(
                ()->new LongAdder(),
                (adder)->adder.decrement()
        );

    }

    private static <T> void demo(Supplier<T> adderSupplier, Consumer<T> action) {
//        获取返回的结果
        T adder = adderSupplier.get();
        long start = System.nanoTime();
        List<Thread> ts = new ArrayList<>();
        // 40 个线程，每人累加 50 万
        for (int i = 0; i < 40; i++) {
            ts.add(new Thread(() -> {
                for (int j = 0; j < 500000; j++) {
                    action.accept(adder);
                }
            }));
        }
        ts.forEach(t -> t.start());
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();

            }
        });
        long end = System.nanoTime();
        System.out.println(adder + " cost:" + (end - start)/1000_000);
    }

}

~~~

- 性能提升的原因很简单，就是在有竞争时，设置多个累加单元，`Therad-0` 累加` Cell[0]`，而 `Thread-1 `累加`Cell[1]`... 最后将结果汇总。这样它们在累加时操作的不同的 `Cell `(共享)变量，因此减少了` CAS `重试失败，从而提高性能。

#### LongAdder源码解读

- LongAdder 类有几个关键域

- transient表示防止序列化

~~~ java
// 累加单元数组, 懒惰初始化
transient volatile Cell[] cells;
// 基础值, 如果没有竞争, 则用 cas 累加这个域
transient volatile long base;
// 在 cells 创建或扩容时, 置为 1, 表示加锁，设置为0表示没有加锁
transient volatile int cellsBusy;
//添加volatile保证共享变量内存的可见性，添加transient表示不可以进行序列化
~~~

**cas模拟锁实现**

~~~ java
public class LockCas {
//    0 表示没有添加锁
//    1 表示添加锁
    private AtomicInteger lock=new AtomicInteger(0);

    public void lock(){
        while (true){
            if(lock.compareAndSet(0,1)){
                System.out.println("加锁操作");
                break;
            }
        }
    }

    public void unlock(){
        System.out.println("解锁操作");
        lock.set(0);
    }

    public static void main(String[] args) {
        LockCas lock = new LockCas();
        new Thread(() -> {
            System.out.println("begin...");
            lock.lock();
            try {
                System.out.println("lock...");
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }).start();
        new Thread(() -> {
            System.out.println("begin...");
            lock.lock();
            try {
                System.out.println("lock...");
            } finally {
                lock.unlock();
            }
        }).start();
    }
}
~~~

#### 原理之伪共享

其中 Cell 即为累加单元

~~~ java
//下面的注解表示防止缓存行的伪共享
@sun.misc.Contended
static final class Cell {
  //value保存累加的结果
        volatile long value;
        Cell(long x) { value = x; }
  //cas累加操作
        final boolean cas(long cmp, long val) {
            return UNSAFE.compareAndSwapLong(this, valueOffset, cmp, val);
        }

        // Unsafe mechanics
        private static final sun.misc.Unsafe UNSAFE;
        private static final long valueOffset;
    }
~~~

**缓存与内存的速度比较**

![1620375800745](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/26/131922-845021.png)

![1620375816073](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/26/131928-485422.png)

因为 CPU 与 内存的速度差异很大，需要靠预读数据至缓存来提升效率。
而缓存以缓存行为单位，每个缓存行对应着一块内存，一般是 64 byte（8 个 long）
缓存的加入会造成数据副本的产生，即同一份数据会缓存在不同核心的缓存行中
CPU 要保证数据的一致性，如果某个 CPU 核心更改了数据，其它 CPU 核心对应的整个缓存行必须失效

![1620375846889](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/26/131933-450234.png)

![1620375860808](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/07/162424-513670.png)



源码不好理解

### UnSafe

#### 概述

Unsafe 对象提供了非常底层的，操作内存、线程的方法，Unsafe 对象不能直接调用，只能通过**反射**获得

~~~ java
package sun.misc;
public final class Unsafe {}
~~~

**获取unsafe对象**

~~~ java
public class TestUnSafe {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
//        getDeclaredFields方法可以获取私有属性,通过反射获取属性
        Field declaredFields = Unsafe.class.getDeclaredField("theUnsafe");
//        设置允许访问私有变量的属性
        declaredFields.setAccessible(true);
//        获取成员变量的属性值
        Unsafe o = (Unsafe) declaredFields.get(null);
        System.out.println(o);//成功获取unsafe对象
        
    }
}
~~~

#### Unsafe CAS 操作

~~~ java
public class Test42 {

    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {

//        使用更底层的unsafe对象对MyStudent对象进行修改
        Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
        theUnsafe.setAccessible(true);
        Unsafe unsafe = (Unsafe)theUnsafe.get(null);
//        unsafe就是我们获取到的unsafe对象
        MyStudent myStudent = new MyStudent();

//        1 获取属性的偏移量，获取MyStudent类中属性id相对于MyStudent类的偏移地址
        long id = unsafe.objectFieldOffset(MyStudent.class.getDeclaredField("id"));
        long name = unsafe.objectFieldOffset(MyStudent.class.getDeclaredField("name"));
//       2 对属性的取值进行操作
        unsafe.compareAndSwapInt(myStudent,id,1,10);
        unsafe.compareAndSwapObject(myStudent,name,null,"zhangsan");
        System.out.println(myStudent.toString());
    }
}

class MyStudent {
    volatile int id;
    volatile String name;

    @Override
    public String toString() {
        return "MyStudent{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
~~~

#### CAS自定义原子整数类

使用自定义的 AtomicData 实现之前线程安全的原子整数 Account 实现

~~~ java
public class Test43 {

    public static void main(String[] args) {

        AtomicData atomicData = new AtomicData(10000);
        MyAccount.demo(atomicData);
    }
}

class  UnsafeAccessor
{
    private static Unsafe unsafe ;
    static {
        try {
            Field theUnsafe = null;
            theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
            theUnsafe.setAccessible(true);
            unsafe = (Unsafe)theUnsafe.get(null);
        } catch (NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
        }

    }

    public static Unsafe getUnsafe(){
        return unsafe;
    }
}
class AtomicData implements MyAccount{
    private volatile int data;
    static final Unsafe unsafe;
//    data的偏移量
    static final long DATA_OFFSET;

    static {
        unsafe = UnsafeAccessor.getUnsafe();
        try {
            // data 属性在 DataContainer 对象中的偏移量，用于 Unsafe 直接访问该属性
            DATA_OFFSET = unsafe.objectFieldOffset(AtomicData.class.getDeclaredField("data"));
        } catch (NoSuchFieldException e) {
            throw new Error(e);
        }
    }
    public AtomicData(int data) {
        this.data = data;
    }
    public void decrease(int amount) {
        int oldValue;
        while(true) {
            // 获取共享变量旧值，可以在这一行加入断点，修改 data 调试来加深理解
            oldValue = data;
            // cas 尝试修改 data 为 旧值 + amount，如果期间旧值被别的线程改了，返回 false
            if (unsafe.compareAndSwapInt(this, DATA_OFFSET, oldValue, oldValue - amount)) {
                return;
            }
        }
    }
    public int getData() {
        return data;
    }

    @Override
    public Integer getBalance() {
        return getData();
    }

    @Override
    public void withdraw(Integer amount) {
        decrease(amount);

    }
}
interface MyAccount {
    // 获取余额
    Integer getBalance();
    // 取款
    void withdraw(Integer amount);
    /**
     * 方法内会启动 1000 个线程，每个线程做 -10 元 的操作
     * 如果初始余额为 10000 那么正确的结果应当是 0
     */
    static void demo(MyAccount account) {
        List<Thread> ts = new ArrayList<>();
        long start = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            ts.add(new Thread(() -> {
                account.withdraw(10);
            }));
        }
        ts.forEach(Thread::start);
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        long end = System.nanoTime();
        System.out.println(account.getBalance()
                + " cost: " + (end-start)/1000_000 + " ms");
    }
}
~~~

### 小结

![1620377818218](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/07/165704-238281.png)



##  共享模型之不可变

###  日期转换的问题

**问题提出**

下面的代码在运行时，由于 `SimpleDateFormat `不是线程安全的

~~~ java
public class Test1 {
    public static void main(String[] args) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    System.out.println(sdf.parse("1951-04-21"));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
//有很大几率出现 java.lang.NumberFormatException 或者出现不正确的日期解析结果
~~~

**添加重量级锁**

~~~ java
public class Test1 {
    public static void main(String[] args) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                  //加锁一般添加在共享变量的上面
                   synchronized (sdf){
                       System.out.println(sdf.parse("1951-04-21"));
                   }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }).start();
        }

    }
}
~~~

**使用线程安全的日期类**

如果一个对象在不能够修改其内部状态（属性），那么它就是线程安全的，因为不存在并发修改啊！这样的对象在Java 中有很多，例如在 Java 8 后，提供了一个新的日期格式化类：不可变对象，实际是另一种避免竞争的方式。

~~~ java
public class Test1 {
    public static void main(String[] args) {
//        使用新的日期类，线程安全的方法
        DateTimeFormatter dateTimeFormatter=DateTimeFormatter.ofPattern("yyyy-MM-dd");
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                TemporalAccessor parse = dateTimeFormatter.parse("1985-05-12");
                System.out.println(parse);
            }).start();
        }

    }
~~~

#### 不可变类的设计

另一个大家更为熟悉的 String 类也是不可变的，以它为例，说明一下不可变设计的要素

~~~ java
public final class String
 implements java.io.Serializable, Comparable<String>, CharSequence {
 /** The value is used for character storage. */
 private final char value[];//设计为final表示其引用不可以在改变
 /** Cache the hash code for the string */
 private int hash; // Default to 0 //缓存对象的哈希码

 // String的成员变量都是不可变的
 //设置为final只可以保证引用不会改变，但是不能保证其内容也不能改变

}

//有参数的构造方法，我们可以看到，输入的字符串被赋值给String类中的value数组
 public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }
    
//如果传入的参数 是一个数组,那么会把数组中的内容拷贝到String中value数组中，这样两个引用指向的就不是同一个数组的引用，外面对数组的改变，并不会String数组中的内容，这种思想叫做保护性拷贝
public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }
~~~

**final 的使用**

发现该类、类中所有属性都是 final 的

- 属性用 final 修饰保证了该属性是**只读的，不能修改**
- 类用 final 修饰保证了该类中的方法**不能被覆盖，防止子类无意间破坏不可变性**

#### 保护性拷贝

但有同学会说，使用字符串时，也有一些跟修改相关的方法啊，比如 substring 等，那么下面就看一看这些方法是如何实现的，就以 substring 为例：

~~~ java
 public String substring(int beginIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        int subLen = value.length - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
    }
~~~

- 发现其内部是调用 String 的构造方法创建了一个新字符串，也就是没有对原始的字符串进行修改的操作，创建新的字符串作为返回结果，再进入这个构造看看，是否对 final char[] value 做出了修改：

~~~ java
public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= value.length) {
                this.value = "".value;
                return;
            }
        }
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
~~~

结果发现也没有，构造新字符串对象时，会生成新的 char[] value，对内容进行复制 。这种通过创建副本对象来避免共享的手段称之为【保护性拷贝（defensive copy）】

### 享元模式

##### 介绍

定义 英文名称：Flyweight pattern.

 **当需要重用数量有限的同一类对象时，可以使用享元模式**

> wikipedia： A flyweight is an object that minimizes memory usage by sharing as much data as
> possible with other similar objects

出自 "Gang of Four" design patterns

归类 Structual patterns

##### 包装类

在JDK中 Boolean，Byte，Short，Integer，Long，Character 等包装类提供了 valueOf 方法，例如 Long 的
valueOf 会缓存 -128~127 之间的 Long 对象，在这个范围之间会重用对象，大于这个范围，才会新建 Long 对
象：

~~~ java
public static Long valueOf(long l) {
        final int offset = 128;
  //如果数字是在-128-127之间的话，并不会真的创建一个真正的long长整形数字，而是去缓存中查找
        if (l >= -128 && l <= 127) { // will cache
            return LongCache.cache[(int)l + offset];
        }
        return new Long(l);
    }

//缓存空间，其实就是一个数组，这样做可以避免对象的重复创建
private static class LongCache {
        private LongCache(){}

        static final Long cache[] = new Long[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Long(i - 128);
        }
    }
~~~

**构造方法**

~~~ java
private static class LongCache {
        private LongCache(){}

        static final Long cache[] = new Long[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Long(i - 128);
        }
    }
//查看long类型的初始化方法，发现LongCache是一个静态的内部类，初始化时候会把-128-127初始化到一个数组中
~~~

> 注意：
>
> - Byte, Short, Long 缓存的范围都是 -128~127
> - Character 缓存的范围是 0~127
> - Integer的默认范围是 -128~127
>   - 最小值不能变
>   - 但最大值可以通过调整虚拟机参数 `
>     -Djava.lang.Integer.IntegerCache.high` 来改变
> - Boolean 缓存了 TRUE 和 FALSE



String 常量池，BigDecimal BigInteger等都是线程安全的类，单个方法运行线程安全，方法都是原子的操作，但是注意这些原子操作的组合可能并不是线程安全的。

后面两个类都是继承自Number类

![1620531814188](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/09/114338-792369.png)

#### 自定义数据库连接池

例如：一个线上商城应用，QPS 达到数千，如果每次都重新创建和关闭数据库连接，性能会受到极大影响。 这时预先创建好一批连接，放入连接池。一次请求到达后，从连接池获取连接，使用完毕后再还回连接池，这样既节约了连接的创建和关闭时间，也实现了连接的重用，不至于让庞大的连接数压垮数据库。

**代码实现**

~~~ java
public class PoolConnectionDemo {
    public static void main(String[] args) {
        Pool pool=new Pool(2);
        for (int i = 0; i <5; i++) {
            new Thread(()->{
                try {
                    Connection connection=pool.borrow();
//                    释放链接
                    Thread.sleep(3);
                    pool.free(connection);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();

        }
    }

}

class Pool{
//    定义连接池的大小
    private final int poolSize;
//    定义链接对象的数组
    private Connection []connections;
//    链接数组的状态，0表示空闲，1表示是繁忙,普通数组不是线程安全的，在这里使用的是原子数组
    private AtomicIntegerArray status;

    public Pool(int poolSize) {
        this.poolSize = poolSize;
        connections=new Connection[poolSize];
        status=new AtomicIntegerArray(new int[poolSize]);
        for(int i=0;i<poolSize;i++){
            connections[i]=new MockConnection(i+1+" ");
        }
    }
//    借链接
    public Connection borrow() throws InterruptedException {
        while (true){
            for (int po = 0; po <poolSize ; po++) {
                if (status.get(po)==0) {//有空闲链接
//                    设置链接状态，这里需要考虑线程安全问题
                    if (status.compareAndSet(po,0,1)) {
                        System.out.println(Thread.currentThread().getName()+"  获取链接  "+connections[po]);
//                        返回链接对象,只有原子操作成功，才返回链接对象
                        return connections[po];
                    }

                }
//                如果没有空闲的链接,就调用wait进入等待队列等待
                synchronized (this){
                    System.out.println(Thread.currentThread().getName()+"  等待链接  "+connections[po]);
                    this.wait();
                }

            }
        }

    }

//    还链接
    public void free(Connection connection){
        for (int i = 0; i < poolSize; i++) {
//            首先判断归还的链接是否是连接池中的对象
            if(connection == connections[i]){
//                在这里不会发生竞争
                status.set(i,0);
                //        通知等待队列中的线程,只要找到一个对象，就退出，然后唤醒等待队列中的线程
                synchronized (this){
                    System.out.println(Thread.currentThread().getName()+"  归还链接  "+connection);
                    this.notifyAll();
                }
                break;
            }

        }


    }
}
//链接对象
class MockConnection implements Connection{
    private String name;

    public MockConnection(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "MockConnection{" +
                "name='" + name + '\'' +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public Statement createStatement() throws SQLException {
        return null;
    }

    @Override
    public PreparedStatement prepareStatement(String sql) throws SQLException {
        return null;
    }

    @Override
    public CallableStatement prepareCall(String sql) throws SQLException {
        return null;
    }

    @Override
    public String nativeSQL(String sql) throws SQLException {
        return null;
    }

    @Override
    public void setAutoCommit(boolean autoCommit) throws SQLException {

    }

    @Override
    public boolean getAutoCommit() throws SQLException {
        return false;
    }

    @Override
    public void commit() throws SQLException {

    }

    @Override
    public void rollback() throws SQLException {

    }

    @Override
    public void close() throws SQLException {

    }

    @Override
    public boolean isClosed() throws SQLException {
        return false;
    }

    @Override
    public DatabaseMetaData getMetaData() throws SQLException {
        return null;
    }

    @Override
    public void setReadOnly(boolean readOnly) throws SQLException {

    }

    @Override
    public boolean isReadOnly() throws SQLException {
        return false;
    }

    @Override
    public void setCatalog(String catalog) throws SQLException {

    }

    @Override
    public String getCatalog() throws SQLException {
        return null;
    }

    @Override
    public void setTransactionIsolation(int level) throws SQLException {

    }

    @Override
    public int getTransactionIsolation() throws SQLException {
        return 0;
    }

    @Override
    public SQLWarning getWarnings() throws SQLException {
        return null;
    }

    @Override
    public void clearWarnings() throws SQLException {

    }

    @Override
    public Statement createStatement(int resultSetType, int resultSetConcurrency) throws SQLException {
        return null;
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency) throws SQLException {
        return null;
    }

    @Override
    public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency) throws SQLException {
        return null;
    }

    @Override
    public Map<String, Class<?>> getTypeMap() throws SQLException {
        return null;
    }

    @Override
    public void setTypeMap(Map<String, Class<?>> map) throws SQLException {

    }

    @Override
    public void setHoldability(int holdability) throws SQLException {

    }

    @Override
    public int getHoldability() throws SQLException {
        return 0;
    }

    @Override
    public Savepoint setSavepoint() throws SQLException {
        return null;
    }

    @Override
    public Savepoint setSavepoint(String name) throws SQLException {
        return null;
    }

    @Override
    public void rollback(Savepoint savepoint) throws SQLException {

    }

    @Override
    public void releaseSavepoint(Savepoint savepoint) throws SQLException {

    }

    @Override
    public Statement createStatement(int resultSetType, int resultSetConcurrency, int resultSetHoldability) throws SQLException {
        return null;
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability) throws SQLException {
        return null;
    }

    @Override
    public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability) throws SQLException {
        return null;
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int autoGeneratedKeys) throws SQLException {
        return null;
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int[] columnIndexes) throws SQLException {
        return null;
    }

    @Override
    public PreparedStatement prepareStatement(String sql, String[] columnNames) throws SQLException {
        return null;
    }

    @Override
    public Clob createClob() throws SQLException {
        return null;
    }

    @Override
    public Blob createBlob() throws SQLException {
        return null;
    }

    @Override
    public NClob createNClob() throws SQLException {
        return null;
    }

    @Override
    public SQLXML createSQLXML() throws SQLException {
        return null;
    }

    @Override
    public boolean isValid(int timeout) throws SQLException {
        return false;
    }

    @Override
    public void setClientInfo(String name, String value) throws SQLClientInfoException {

    }

    @Override
    public void setClientInfo(Properties properties) throws SQLClientInfoException {

    }

    @Override
    public String getClientInfo(String name) throws SQLException {
        return null;
    }

    @Override
    public Properties getClientInfo() throws SQLException {
        return null;
    }

    @Override
    public Array createArrayOf(String typeName, Object[] elements) throws SQLException {
        return null;
    }

    @Override
    public Struct createStruct(String typeName, Object[] attributes) throws SQLException {
        return null;
    }

    @Override
    public void setSchema(String schema) throws SQLException {

    }

    @Override
    public String getSchema() throws SQLException {
        return null;
    }

    @Override
    public void abort(Executor executor) throws SQLException {

    }

    @Override
    public void setNetworkTimeout(Executor executor, int milliseconds) throws SQLException {

    }

    @Override
    public int getNetworkTimeout() throws SQLException {
        return 0;
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException {
        return null;
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return false;
    }
}
~~~

- 以上实现没有考虑：
  - 连接的动态增长与收缩
  - 连接保活（可用性检测）
  - 等待超时处理
  - 分布式 hash
- 对于关系型数据库，有比较成熟的连接池实现，例如c3p0, druid等 对于更通用的对象池，可以考虑使用apache commons pool，例如redis连接池可以参考jedis中关于连接池的实现

### Final原理（静态成员变量或者成员变量）

#### 设置Final变量的原理

理解了 volatile 原理，再对比 final 的实现就比较简单了

~~~ java
public class TestFinal {
 final int a = 20;
}
~~~

**字节码角度**

~~~ java
0: aload_0
1: invokespecial #1 // Method java/lang/Object."<init>":()V
4: aload_0
5: bipush 20
7: putfield #2 // Field a:I   给成员变量赋值操作
 <-- 写屏障   //添加写屏障
10: return
~~~

- 如果一个变量被声明为final变量，那么会在赋值之后添加一个写屏障，**写屏障可以保证两个问题，写屏障之前的语句不会被重排序到写屏障之后，写屏障之前的所有赋值或者修改操作，在写屏障之后都会被同步到主存中，也就是对其他线程可见。**
- 给变量添加final关键字就保证了其他线程只能看到final当前的值，保证对其他的线程可见。
- 如果一个变量没有添加final关键字，那么会有两步操作，**第一步是分配空间，第二部是对变量进行赋值**

#### 获取final变量的原理

**代码说明**

~~~ java
public class TestFinal {
    static  int A=10;
    final static int B=Short.MAX_VALUE+1;

    final int a=20;
    final int b=Integer.MAX_VALUE;
}

class UseFinal{
    public void test(){
     // 获取使用final修饰的变量，相当于其他类中的final变量赋值到了一份到其他线程的栈中，如果其他类中的变量没有使用final修饰，那么会去其他类的变量表中获取变量的值，使用的是getstatic指令，从堆内存中读取
      使用final优化的原理就是把其他类中final修饰的变量赋值了一份到其他方法的方法栈中，如果没有添加final关键字，相当于去其他类的堆内存中获取
        System.out.println(TestFinal.A);
        System.out.println(TestFinal.B);

        System.out.println(new TestFinal().a);
        System.out.println(new TestFinal().b);
    }
}
~~~

对于final修饰的变量，如果数值较小，会赋值一份到其他方法的栈中，如果数值较大，会赋值一份到其他类的常量池中，相当于做的一种优化操作，如果不添加final，那么获取静态变量使用getstatic指令，相当于去共享内存中获取，比栈内存中的效率低。

### 小结

![1620534003752](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/09/122006-305362.png)

## 线程池

### 自定义线程池

~~~ java
public class ThreadPoolDemo {
    public static void main(String[] args) {
//        创建线程池对象
        ThreadPool threadPool=new ThreadPool(2,1000,TimeUnit.MICROSECONDS,10,(queue,task)->{

//            1 死等
//            queue.put(task);
//                2 带超时的等待
            queue.Offer(task,500,TimeUnit.MICROSECONDS);
//                3 放弃执行任务
            System.out.println("什么都不做，抛弃任务");
////                4 让调用者抛出异常,某个任务抛出异常，其后面的任务都不会执行
            throw new RuntimeException("任务执行失败"+task);
//                5 让调用者自己执行任务
//            task.run();//实际是主线程执行的任务
        });
        for (int i = 0; i < 15; i++) {
            int j=i;
            threadPool.execute(()->{
                try {
                    Thread.sleep(1000l);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("j= "+j);
            });
        }
    }
}

interface RejectPolicy<T>{
    public void reject(BlockingPool<T> blockingPool,T task);
}

//定义线程池
class ThreadPool{
//    定义任务队列
    private BlockingPool<Runnable> taskqueue;

//    定义线程的集合,线程共享，所以要保证安全性
    private HashSet<Workers> workers=new HashSet<>();

//    定义核心线程数
    private int coreSize;

//    定义超时时间，也就是线程等够一个时间，没有任务的话就退出
    private long timeout;

//    定义超时时间单位
    private TimeUnit unit;

    private RejectPolicy<Runnable> rejectPolicy;

    public ThreadPool(int coreSize, long timeout, TimeUnit unit,int queueCapcity,RejectPolicy rejectPolicy) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.unit = unit;
        this.taskqueue=new BlockingPool<>(queueCapcity);
        this.rejectPolicy=rejectPolicy;
    }

    public void execute(Runnable task){
//        如果任务数量超过coreSize大小，就进入阻塞队列，否则就开始执行
        synchronized (workers){
            if(workers.size() < coreSize){
                Workers workers1=new Workers(task);
                System.out.println("新增workers  "+workers1+"  "+task);
//            把创建的workers添加到hashset的集合中去
                workers.add(workers1);
                workers1.start();
            }else {
//            添加到任务队列
                System.out.println("加入任务队列  "+task);
//                taskqueue.put(task);
//                队列满了如何做：
//                1 死等
//                2 带超时的等待
//                3 放弃执行任务
//                4 让调用者抛出异常
//                5 让调用者自己执行任务
                taskqueue.tryPut(rejectPolicy,task);
            }
        }
    }

    //    对线程进行包装
    class Workers extends Thread{
        private Runnable task;
        public Workers(Runnable task){
            this.task=task;
        }

        @Override
        public void run() {
//            1 当task不是空，也就是有任务，那么就执行任务
//            2，查看任务队列是否有任务，如果有任务，取出任务开始执行
//            第二个条件表示判断任务队列是否是空，如果不空就取出任务开始执行
//            while (task != null || (task=taskqueue.take())!= null){
            while (task != null || (task=taskqueue.pool(timeout,unit))!= null){
//                如果任务不是空，那么就一直执行
                System.out.println("正在执行任务对  :"+task);
               try {
                   task.run();
               }catch (Exception e){
                   e.printStackTrace();
               }finally {
                   task=null;
               }
            }
//            移除当前对象，表示执行完毕
            synchronized (workers){
                System.out.println("移除任务对象  ："+task);
                workers.remove(this);
            }
        }
    }

}

class BlockingPool<T>{
//    定义一个双向链表的阻塞队列
    private Deque <T>queue=new ArrayDeque();

//    需要添加一把锁，因为多个线程访问
    private ReentrantLock lock=new ReentrantLock();

//    生产者的条件变量
    private Condition fullWait=lock.newCondition();

//    消费者条件变量
    private Condition emptyWait=lock.newCondition();
//    线程池的容量
    private int capicity;

    public BlockingPool(int capicity) {
        this.capicity = capicity;
    }

    /**
     * 带超时的阻塞获取
     * @param timeOut 超时时间
     * @param timeUnit 时间单位 方便做时间转换的类
     * @return
     */
    public T pool(long timeOut, TimeUnit timeUnit){

        T t=null;
        lock.lock();//加锁
        try {
//            将时间转换为以纳秒为单位，这里存在虚假唤醒问题，也就是这一次没有等够timeOut时间，但是下一次需
//            要从头开始等待，但是awaitNanos返回的是上一次等待的时间和总的需要等待时间的差值，也就是需要等待的时间
            long l = timeUnit.toNanos(timeOut);
            while (queue.isEmpty()){
                if(l < 0)
//                    剩余时间小于0，就是没有等到
                    return null;
               l= emptyWait.awaitNanos(l);
            }
            t = queue.removeFirst();
//            唤醒添加元素的队列中某一个元素
            fullWait.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return t;
    }

//    阻塞获取
    public T take(){
        T t=null;
        lock.lock();//加锁
        try {
            while (queue.isEmpty()){
                emptyWait.await();
            }
            t = queue.removeFirst();
//            唤醒添加元素的队列中某一个元素
            fullWait.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return t;
    }

//    阻塞添加
    public void put(T t){
        lock.lock();
        try{
            while (queue.size() == capicity){
//                添加到另外一个条件变量
                System.out.println("等待加入任务队列  "+t);
                fullWait.wait();
            }
            System.out.println("加入任务队列  "+t);
            queue.addLast(t);
//            唤醒一个线程
            emptyWait.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
//带超时的阻塞添加
    public boolean Offer(T task,long timeout,TimeUnit unit){
        lock.lock();
        try{
            long nanos=unit.toNanos(timeout);
            while (queue.size() == capicity){
//                添加到另外一个条件变量
                System.out.println("等待加入任务队列  "+task);
                if(nanos < 0){
                    return false;
                }
                nanos=fullWait.awaitNanos(nanos);
            }
            System.out.println("加入任务队列  "+task);
            queue.addLast(task);
//            唤醒一个线程
            emptyWait.signal();
            return true;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return false;
    }

    //    获取大小
    public int size()
    {
        lock.lock();
        try
        {
            return queue.size();
        }finally {
            lock.unlock();
        }
    }

    public void tryPut(RejectPolicy rejectPolicy,T task){
        lock.lock();
        try {
            if(queue.size() == capicity){
//                这里使用的是策略模式
                rejectPolicy.reject(this,task);
            }else {
                System.out.println("加入任务队列  "+task);
                queue.addLast(task);
//            唤醒一个线程
                emptyWait.signal();
            }
        }finally {
            lock.unlock();
        }
    }
}
~~~

### ThreadPoolExecutor线程池

ThreadPoolExecutor线程池是jdk自带的线程池。

**类图**

![1620538602660](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/09/133646-841285.png)

- ExecutorService线程池最基本的接口，包括提交任务，关闭线程池等
- ScheduledExecutorService:扩展接口，在基础接口放入功能上增加任务调度的功能。
- ThreadPoolExecutor:线程池的最基本实现
- SchedualThreadPoolExecutor:带有任务调度的最基础实现

**Executor**

~~~ java
public interface Executor {
//执行线程
    void execute(Runnable command);
}
~~~

#### 线程池的状态

ThreadPoolExecutor 使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量

![1609477104864](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/07/152457-620669.png)

从数字上比较，TERMINATED > TIDYING > STOP > SHUTDOWN > RUNNING（最高位代表符号位）

这些信息存储在一个原子变量 ctl 中，目的是将线程池状态与线程个数合二为一，这样就可以用一次 cas 原子操作进行赋值，如果分开的话就要使用两次cas操作。

~~~ java
// c 为旧值， ctlOf 返回结果为新值
ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))));
ctlOf(targetState, workerCountOf(c))//表示合并线程状态和线程数量为一个值
// rs 为高 3 位代表线程池状态， wc 为低 29 位代表线程个数，ctl 是合并它们
private static int ctlOf(int rs, int wc) { return rs | wc; }
~~~

#### 构造方法

~~~ java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
~~~

- corePoolSize 核心线程数目 (最多保留的线程数)
- maximumPoolSize 最大线程数目
- keepAliveTime 生存时间 - 针对救急线程
- unit 时间单位 - 针对救急线程
- workQueue 阻塞队列
- threadFactory 线程工厂 - 可以为线程创建时起个好名字
- handler 拒绝策略

jdk中的线程分为核心线程和救济线程，救济线程是针对突发量大，线程池中线程数量不足的情况，如果线程池中线程不够，阻塞队列也蛮了，那么就会创建救济线程，救济线程最大的特点是有生存时间，当救济线程也不够的话，就会执行拒绝策略。

- 最大线程个数=救济线程数+核心线程数
- 生存时间是针对救济线程来说的。
- 时间单位也是针对救济线程来说的。
- 当核心线程用完之后，阻塞队列也满的情况下，如果有新的线程到来，那么会看是否还有救济线程，救济线程=最大线程数-核心线程数，如果有救济线程，就创建救济线程执行任务，否则就执行拒绝策略。

**工作方式**

![1609477930176](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202101/01/131212-741170.png)

- 线程池中刚开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务。

- 当线程数达到 corePoolSize 并没有线程空闲，这时再加入任务，新加的任务会被加入workQueue 队列排
  队，直到有空闲的线程。

- **如果队列选择了有界队列**，那么任务超过了队列大小时，会创建 maximumPoolSize - corePoolSize 数目的线程来救急。（也就是救济线程），如果选择的队列是无界的，那么此时就没有什么救济线程。

- 如果线程到达 maximumPoolSize 仍然有新任务这时会执行拒绝策略。拒绝策略 jdk 提供了 4 种实现，其它
  著名框架也提供了实现

  - AbortPolicy 让调用者抛出 RejectedExecutionException 异常，这是默认策略
  - CallerRunsPolicy 让调用者运行任务
  - DiscardPolicy 放弃本次任务
  - DiscardOldestPolicy 放弃队列中最早的任务，本任务取而代之

  ![1609478329567](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202101/01/131851-399649.png)

  上面是jdk提供的实现，下面是框架的实现。

  - Dubbo 的实现，在抛出 RejectedExecutionException 异常之前会记录日志，并 dump 线程栈信息，方
    便定位问题
  - Netty 的实现，是创建一个新线程来执行任务
  - ActiveMQ 的实现，带超时等待（60s）尝试放入队列，类似我们之前自定义的拒绝策略
  - PinPoint 的实现，它使用了一个拒绝策略链，会逐一尝试策略链中每种拒绝策略

- 当高峰过去后，超过corePoolSize 的救急线程如果一段时间没有任务做，需要结束节省资源，这个时间由
  keepAliveTime 和 unit 来控制。

根据这个构造方法，JDK Executors 类中提供了众多工厂方法来创建各种用途的线程池，因为普通的构造方法提供的参数太多，所以jdk提供一个工具类来使用。

~~~ java
//工具类方法
public class Executors {}
~~~

#### Executors工具

##### newFixedThreadPool（固定线程数）

~~~ java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,//核心线程数和最大线程数被设置为nThreads,表示救济线程数设置为0，救济线程=最大线程数-核心线程数，可以看到对返回的对象没有进行包装操作
                0L, TimeUnit.MILLISECONDS,
                //阻塞队列的实现 LinkedBlockingQueue
                new LinkedBlockingQueue<Runnable>());
    }
~~~

**特点**

- 核心线程数 == 最大线程数（没有救急线程被创建），因此也无需超时时间（救济线程有超时时间，也就是超过一段时间后就会被回收）
- 阻塞队列是无界的，可以放任意数量的任务

> 评价
>
> - 适用于任务量已知，相对耗时的任务

**代码说明**

~~~ java
public class ExecutorTest {
    public static void main(String[] args) {
//        创建固定大小的线程池
        ExecutorService executorService = Executors.newFixedThreadPool(2);
//        让线程池中的线程执行任务
        executorService.execute(()->{
            System.out.println(Thread.currentThread().getName()+"  1");
        });
        executorService.execute(()->{
            System.out.println(Thread.currentThread().getName()+"  2");
        });
        executorService.execute(()->{
            System.out.println(Thread.currentThread().getName()+"  3");
        });
    }
}
//输出结果
pool-1-thread-1  1
pool-1-thread-2  2
pool-1-thread-1  3
  
//创建线程使用的是线程工厂
 DefaultThreadFactory() {
            SecurityManager s = System.getSecurityManager();
            group = (s != null) ? s.getThreadGroup() :
                                  Thread.currentThread().getThreadGroup();
            namePrefix = "pool-" +
                          poolNumber.getAndIncrement() +
                         "-thread-";
        }
~~~

- 固定大小的线程池中核心线程是固定的，所以线程执行完毕后不会主动结束自己
- 线程池中的线程都是非守护线程，也就是不会随着主线程的结束而结束

##### newCachedThreadPool（带缓冲功能）

~~~ java
 public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                60L, TimeUnit.SECONDS,
                new SynchronousQueue<Runnable>());
    }
//创建出来的全部是救济线程
~~~

**特点**

- 核心线程数是 0， 最大线程数是 Integer.MAX_VALUE，救急线程的空闲生存时间是 60s，意味着救济线程超过60s后就会被回收，创建的线程全部是救济线程。
  - 全部都是救急线程（60s 后可以回收）
  - 救急线程可以无限创建
- 队列采用了 SynchronousQueue 实现特点是，它没有容量，没有线程来取是放不进去的（一手交钱、一手交货）（也就是你想放到队列中一个任务，但是此时如果没有线程来队列中取走一个任务，你是放不进去任务的）。

**代码说明**

~~~ java
public class TestSynchronizedQueue {
    public static void main(String[] args) throws InterruptedException {
        SynchronousQueue<Integer> integers = new SynchronousQueue<>();
        new Thread(() -> {
            try {
                System.out.println(Thread.currentThread().getName()+"  putting..."+1);
                integers.put(1);
                System.out.println(Thread.currentThread().getName()+"  putted..."+1);
                System.out.println(Thread.currentThread().getName()+"  putting..."+2);
                integers.put(2);
                System.out.println(Thread.currentThread().getName()+"  putted..."+2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1").start();
       Thread.sleep(1);
        new Thread(() -> {
            try {
                System.out.println(Thread.currentThread().getName()+"  taking...."+1);
                integers.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t2").start();
        Thread.sleep(1);
        new Thread(() -> {
            try {
                System.out.println(Thread.currentThread().getName()+"  taking....."+2);
                integers.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t3").start();
    }
}
//输出，可以看出，只有t2线程取走的时候,t1线程才会真正的放入数据
t1  putting...1
t2  taking....1
t1  putted...1
t1  putting...2
t3  taking.....2
t1  putted...2
~~~

> 评价 
>
> - 整个线程池表现为线程数会根据任务量不断增长，没有上限，当任务执行完毕，空闲 1分钟后释放线
>   程。 适合任务数比较密集，但每个任务执行时间较短的情况

##### newSingleThreadExecutor（单线程执行器）

~~~ java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService//对返回的对象进行包装操作，以后调用只能够调用接口中的方法
                (new ThreadPoolExecutor(1, 1,//没有救济线程
                        0L, TimeUnit.MILLISECONDS,
                        new LinkedBlockingQueue<Runnable>()));
    }
~~~

**使用场景：**

- 希望多个任务排队执行，也就是串行执行时候。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。任务执行完毕，这唯一的线程也不会被释放。

**区别**

- 自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而线程池还会新建一个线程，保证池的正常工作

~~~ java
//代码测试
public class TestSinglePool {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        executorService.execute(()->{
//            线程执行会失败
            System.out.println(Thread.currentThread().getName()+"  "+1/0);
        });
        executorService.execute(()->{
            System.out.println(Thread.currentThread().getName()+"  执行成功");
        });
        executorService.execute(()->{
            System.out.println(Thread.currentThread().getName()+"  执行成功");
        });
    }
}
~~~

- Executors.newSingleThreadExecutor() 线程个数始终为1，不能修改
  - FinalizableDelegatedExecutorService 应用的是装饰器模式，只对外暴露了 ExecutorService 接口，因
    此不能调用 ThreadPoolExecutor 中特有的方法，也就是只可以调用接口中的方法
- Executors.newFixedThreadPool(1) 初始时为1，以后还可以修改
  - 对外暴露的是 ThreadPoolExecutor 对象（直接是实现的对象），可以强转后调用 setCorePoolSize 等方法进行修改

#### 提交任务

~~~ java
// 执行任务
 void execute(Runnable command);
// 提交任务 task，用返回值 Future 获得任务执行结果,Callable多一个返回任务的结果
    <T> Future<T> submit(Callable<T> task);
//案例
public class TestSubmit {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
//        使用future对象接受我们返回的结果
//        Callable也可以使用lambda表达式简化
       Future<String >future= executorService.submit(new Callable<String>() {
            @Override
            public String call() throws Exception {
//                泛型表示的是返回结果的类型
                System.out.println("running");
                Thread.sleep(2);
                return "Hello";
            }
        });

        System.out.println(future.get());
    }
}

// 提交 tasks 中所有任务，提交的任务是一个集合
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
            throws InterruptedException;
// 提交 tasks 中所有任务，，一定时间内任务没有完成的话就取消任务
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
            throws InterruptedException;
// 提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
            throws InterruptedException, ExecutionException;
// 提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消，带超时时间
<T> T invokeAny(Collection<? extends Callable<T>> tasks,
        long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
~~~

**案例演示**

~~~ java
public class TestSubmit {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
//返回的结果是list集合。里面是future类型，存储多个结果
        List<Future<Integer>> futures = executorService.invokeAll(Arrays.asList(
//                集合里面有多个Callable任务，
//                Callable接口有一个抽象方法
          //集合中存储的是任务队列
                () -> {
                    System.out.println("begin in.... ");
                    Thread.sleep(2);
                    return 1;
                },
                () -> {
                    System.out.println("begin in.... ");
                    Thread.sleep(3);
                    return 2;
                },
                () -> {
                    System.out.println("begin in.... ");
                    Thread.sleep(4);
                    return 3;
                }
        ));
        futures.forEach(f->{
            try {
                System.out.println(f.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        });
    }
//带超时时间
public class TestSubmit {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
//返回一个结果 1，因为执行时间最短
        Integer futures = executorService.invokeAny(Arrays.asList(
//                集合里面有多个Callable任务，
//                Callable接口有一个抽象方法
                () -> {
                    System.out.println("begin in.... ");
                    Thread.sleep(2);
                    return 1;
                },
                () -> {
                    System.out.println("begin in.... ");
                    Thread.sleep(3);
                    return 2;
                },
                () -> {
                    System.out.println("begin in.... ");
                    Thread.sleep(4);
                    return 3;
                }
        ));
        System.out.println(futures);
    }
}
~~~

**Feature接口**

![1620542832858](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/09/144715-257198.png)

#### 关闭线程池

- 关闭线程池
  - shutdown:只会打断空闲的线程,不会影响正在执行的和已经提交到任务队列中的任务。
  -  shutdownNow：打断所有的线程，会影响正在执行的任务和已经提交到任务队列中的任务

~~~ java
/*
线程池状态变为 STOP
- 不会接收新任务
- 会将队列中的任务返回
- 并用 interrupt 的方式中断正在执行的任务
*/
List<Runnable> shutdownNow();
~~~

**源码实现**

~~~ java
 public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();//添加锁
        try {
            checkShutdownAccess();
            // 修改线程池状态
            advanceRunState(SHUTDOWN);
            // 仅会打断空闲线程
            interruptIdleWorkers();
            onShutdown(); // 扩展点 ScheduledThreadPoolExecutor
        } finally {
            mainLock.unlock();
        }
        // 尝试终结(没有运行的线程可以立刻终结，如果还有运行的线程也不会等)其他的线程运行完毕自己结束
        tryTerminate();
    }
~~~

**shutdownNow**

~~~ java
/*
线程池状态变为 STOP
- 不会接收新任务
- 会将队列中的任务返回
- 并用 interrupt 的方式中断正在执行的任务
*/
List<Runnable> shutdownNow();
~~~

**源码解读**

~~~ java
public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            // 修改线程池状态
            advanceRunState(STOP);
            // 打断所有线程
            interruptWorkers();
            // 获取队列中剩余任务
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        // 尝试终结
        tryTerminate();
        return tasks;
~~~

**其他方法**

~~~ java
// 不在 RUNNING 状态的线程池，此方法就返回 true
boolean isShutdown();
// 线程池状态是否是 TERMINATED
boolean isTerminated();
// 调用 shutdown 后，由于调用线程并不会等待所有任务运行结束，因此如果它想在线程池 TERMINATED 后做些事情，可以利用此方法等待
boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
~~~

**案例**

~~~ java
public class Test45 {

    public static void main(String[] args) {

        ExecutorService pool = Executors.newFixedThreadPool(2);
        Future<Integer> res=pool.submit(()-> {

                System.out.println("task 1 running ");
                Thread.sleep(1);
                System.out.println("task 1 finnshed");
                return 1;
        });

        Future<Integer> res1 = pool.submit(() -> {

                System.out.println("task 2 running ");
                Thread.sleep(1);
                System.out.println("task 2 finnshed");
                return 2;
        });

        Future<Integer> res2 = pool.submit(() -> {

            System.out.println("task 3 running ");
            Thread.sleep(1);
            System.out.println("task 3 finnshed");
            return 3;
        });

        System.out.println("shutdown");
        pool.shutdown();
    }
}

//可以发现，shotdown并不会取消正在执行的任务，也不会影响提交到队列中的任务，但是不能在shotdown后面在提交任务，shotdown也不会阻塞主线程其他的代码执行
task 1 running 
task 2 running 
shutdown
task 1 finnshed
task 2 finnshed
task 3 running 
task 3 finnshed
~~~

### 异步模式之工作线程

#### 定义

让有限的工作线程（Worker Thread）来轮流异步处理无限多的任务。也可以将其归类为分工模式，它的典型实现就是线程池，也体现了经典设计模式中的享元模式。

**案例**

例如，海底捞的服务员（线程），轮流处理每位客人的点餐（任务），如果为每位客人都配一名专属的服务员，那么成本就太高了（对比另一种多线程设计模式：Thread-Per-Message）

**注意，不同任务类型应该使用不同的线程池，这样能够避免饥饿，并能提升效率**，点餐和做菜采用不同的线程池

例如，如果一个餐馆的工人既要招呼客人（任务类型A），又要到后厨做菜（任务类型B）显然效率不咋地，分成服务员（线程池A）与厨师（线程池B）更为合理，当然你能想到更细致的分工

#### 饥饿

固定大小线程池会有饥饿现象，是因为线程数量的不足导致饥饿。

- 两个工人是同一个线程池中的两个线程
- 他们要做的事情是：为客人点餐和到后厨做菜，这是两个阶段的工作
  - 客人点餐：必须先点完餐，等菜做好，上菜，在此期间处理点餐的工人必须等待
  - 后厨做菜：没啥说的，做就是了
- 比如工人A 处理了点餐任务，接下来它要等着 工人B 把菜做好，然后上菜，他俩也配合的蛮好
- 但现在同时来了两个客人，这个时候工人A 和工人B 都去处理点餐了，这时没人做饭了，饥饿
- 检测死锁可以使用jconcle工具。

**饥饿问题**

~~~ java
public class TestDeadLock {
    static final List<String> MENU = Arrays.asList("地三鲜", "宫保鸡丁", "辣子鸡丁", "烤鸡翅");
    static Random RANDOM = new Random();

    static String cooking() {
        return MENU.get(RANDOM.nextInt(MENU.size()));
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
      //外面线程是负责点餐的线程
        executorService.execute(() -> {
            log.debug("处理点餐...");
          //里面线程是负责做菜的线程
            Future<String> f = executorService.submit(() -> {
                log.debug("做菜");
                return cooking();
            });
            try {
                log.debug("上菜: {}", f.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        });
      //单独执行上面一段程序可以成功，但是上下的程序员一起执行就无法执行，因为总共需要4个线程，而线程池中只有2个线程
        executorService.execute(() -> {
            log.debug("处理点餐...");
            Future<String> f = executorService.submit(() -> {
                log.debug("做菜");
                return cooking();
            });
            try {
                log.debug("上菜: {}", f.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        });
    }
}
~~~

解决方法可以增加线程池的大小，不过不是根本解决方案，还是前面提到的，不同的任务类型，采用不同的线程池，例如：

~~~ java
public class TestDeadLock {
    static final List<String> MENU = Arrays.asList("地三鲜", "宫保鸡丁", "辣子鸡丁", "烤鸡翅");
    static Random RANDOM = new Random();

    static String cooking() {
        return MENU.get(RANDOM.nextInt(MENU.size()));
    }

    public static void main(String[] args) {
//        不同的任务类型使用不同的线程池
        ExecutorService WaiterPool = Executors.newFixedThreadPool(2);
        ExecutorService cookPool = Executors.newFixedThreadPool(2);
//        WaiterPool线程池专门用来点餐
        WaiterPool.execute(() -> {
            System.out.println(Thread.currentThread().getName() + "  处理点餐...");
//            cookPool线程池专门用来做饭
            Future<String> f = cookPool.submit(() -> {
                System.out.println(Thread.currentThread().getName() + "  做菜");
                return cooking();
            });
            try {
                System.out.println(f.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        });
        WaiterPool.execute(() -> {
            System.out.println(Thread.currentThread().getName() + "  处理点餐...");
            Future<String> f = cookPool.submit(() -> {
                System.out.println(Thread.currentThread().getName() + "  做菜");
                return cooking();
            });
            try {
                System.out.println(f.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        });
    }
}
~~~

#### 创建多少线程合适

- 过小会导致程序不能充分地利用系统资源、容易导致饥饿
- 过大会导致更多的线程上下文切换，占用更多内存

**CPU密集型运算**

通常采用 cpu 核数 + 1 能够实现最优的 CPU 利用率，+1 是保证当线程由于页缺失故障（操作系统）或其它原因导致暂停时，额外的这个线程就能顶上去，保证 CPU 时钟周期不被浪费

**IO密集型运算**

CPU 不总是处于繁忙状态，例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率。

- 经验公式如下
  - 线程数 = 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间
- 例如 4 核 CPU 计算时间是 50% ，其它等待时间是 50%，期望 cpu 被 100% 利用，套用公式
  4 * 100% * 100% / 50% = 8
- 例如 4 核 CPU 计算时间是 10% ，其它等待时间是 90%，期望 cpu 被 100% 利用，套用公式
  4 * 100% * 100% / 10% = 40

### 任务调度线程池（延时执行任务）

在『任务调度线程池』功能加入之前，可以使用 java.util.Timer 来实现定时功能，Timer 的优点在于简单易用，但由于所有任务都是由同一个线程来调度，因此所有任务都是串行执行的，同一时间只能有一个任务在执行，前一个任务的延迟或异常都将会影响到之后的任务。

####  使用Timer类

**代码说明**

~~~ java
public static void main(String[] args) {
        Timer timer = new Timer();
        TimerTask task1 = new TimerTask() {
            @Override
            public void run() {
                log.debug("task 1");
              1/0;//这里的异常可以使用try-catch进行自己处理
                sleep(2);
            }
        };
        TimerTask task2 = new TimerTask() {
            @Override
            public void run() {
                log.debug("task 2");
            }
        };
        // 使用 timer 添加两个任务，希望它们都在 1s 后执行
        // 但由于 timer 内只有一个线程来顺序执行队列中的任务，因此『任务1』的延时，影响了『任务2』的执行，也就是如果任务一的执行时间较长，那么任务二的执行时间也会推迟，如果任务一发生异常，也会导致任务二不执行
        timer.schedule(task1, 1000);
        timer.schedule(task2, 1000);
    }
~~~

#### ScheduledExecutorService 

使用 ScheduledExecutorService 改写：

~~~ java
public class TestTimer {

    public static void main(String[] args) {
        //创建一个带有调度功能的线程池
        ScheduledExecutorService service = Executors.newScheduledThreadPool(2);
//        第一个参数代表执行的任务，第二个是延迟的时间
        service.schedule(()->{
            try {
//                延时2秒在执行
                Thread.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"  task 1");
        },1, TimeUnit.SECONDS);

        //第一个参数是延时的任务，第二个是延时的时间
        service.schedule(()->{
            System.out.println(Thread.currentThread().getName()+"  task 2");
        },1, TimeUnit.SECONDS);
//        上面两个任务几乎是同时执行的，第一个任务虽然延时2秒，但是并不会影响第二个任务的执行
    }
}
//两个线程几乎是同时执行的，也就是说不会相互影响，虽然第一个线程睡了2秒，但是并不会影响第二个线程的执行
//如果线程池的大小小于创建的线程个数，那么线程之间还是会串行执行
//即使第一个线程执行过程中出现异常，第二个任务还是可以正常执行，但是对于Timer就不行，第二个任务不能执行
~~~

#### 定时执行任务

每隔一段时间，执行一次任务。

scheduleAtFixedRate 例子：

~~~ java
public class TestTimer {

    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newScheduledThreadPool(2);
//      定时执行任务
//        第一个参数：延时的对象，第二个是延时的时间，第三个是执行的间隔
        System.out.println(Thread.currentThread().getName()+"  starting");
        service.scheduleAtFixedRate(()->{
            System.out.println(Thread.currentThread().getName()+"  starting");
        },1,1,TimeUnit.SECONDS);
    }
  //结果，也就是每隔一秒执行一次任务
main  starting
pool-1-thread-1  starting
pool-1-thread-1  starting
pool-1-thread-1  starting
pool-1-thread-2  starting
~~~

scheduleAtFixedRate 例子（任务执行时间超过了间隔时间）：

~~~ java
ScheduledExecutorService pool = Executors.newScheduledThreadPool(1);
log.debug("start...");
pool.scheduleAtFixedRate(() -> {
 log.debug("running...");
 sleep(2);
}, 1, 1, TimeUnit.SECONDS);
//输出分析：一开始，延时 1s，接下来，由于任务执行时间 > 间隔时间，间隔被『撑』到了 2s
21:44:30.311 c.TestTimer [main] - start...
21:44:31.360 c.TestTimer [pool-1-thread-1] - running...
21:44:33.361 c.TestTimer [pool-1-thread-1] - running...
21:44:35.362 c.TestTimer [pool-1-thread-1] - running...
21:44:37.362 c.TestTimer [pool-1-thread-1] - running...
~~~

#### scheduleWithFixedDelay

~~~ java
public class TestTimer {

    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newScheduledThreadPool(2);
//      定时执行任务
//        第一个参数：延时的对象，第二个是延时的时间，第三个是执行的间隔
//        System.out.println(Thread.currentThread().getName()+"  starting");
//        service.scheduleAtFixedRate(()->{
//            System.out.println(Thread.currentThread().getName()+"  starting");
//        },1,1,TimeUnit.SECONDS);
//      第一个参数是任务对象，第二个是初始的延时，第三个是任务和任务之间的间隔时间，
        service.scheduleWithFixedDelay(()->{
            System.out.println(Thread.currentThread().getName()+"  starting");
            try {
//                这里任务执行2秒，所以控制台打印的是每隔3秒打印一次，应为delay参数是从上一次任务结束的时间计算起
                Thread.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },1,1,TimeUnit.SECONDS);
    }
~~~

> 评价 整个线程池表现为：线程数固定，任务数多于线程数时，会放入无界队列排队。任务执行完毕，这些线程也不会被释放。用来执行延迟或反复执行的任务

#### 正确处理异常信息

~~~ java
//主动铺货异常
ExecutorService pool = Executors.newFixedThreadPool(1);
pool.submit(()->{
        try{
        log.debug("task1");
        int i=1/0;
        }catch(Exception e){
        log.error("error:",e);
        }
        });

//使用future对象打印异常信息
ExecutorService pool = Executors.newFixedThreadPool(1);
    Future<Boolean> f = pool.submit(() -> {
        log.debug("task1");
        int i = 1 / 0;
        return true;
    });
log.debug("result:{}", f.get());
~~~

#### 应用-定时执行任务

**代码展示**

~~~ java
public class TestTimer {

    public static void main(String[] args) {
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(1);

        //一周的时间间隔
        long period=1000*60*60*24*7;

        //获取当前时间
        LocalDateTime time=LocalDateTime.now();
//        在当前的时间基础上面获取周四时间,修改 周使用的是一个枚举值
        LocalDateTime time1=time.withHour(18).withSecond(0).withSecond(0).withNano(0).with(DayOfWeek.THURSDAY);
        System.out.println(time1);
//        如果当前时间大于本周的周四，那么就找下周的周四
        if(time.compareTo(time1)>0){
//            在本周的基础上面添家一周
            time.plusWeeks(1);

        }
//        计算当前时间和周四时间的差值
//        System.out.println(Duration.between(time,time1).toMillis());
        long initialDelay=Duration.between(time,time1).toMillis();
//        第一个参数是任务对象，第二个是延时时间，也就是对于当前时间，需要延时多少，第三个是间隔时间，间隔多长时间执行一次
        pool.scheduleAtFixedRate(()->{
            System.out.println("running");
        },initialDelay,period,TimeUnit.MILLISECONDS);
    }
~~~

### Fork/Join线程池

#### 概念

- Fork/Join 是 JDK 1.7 加入的新的线程池实现，它体现的是一种分治思想，适用于能够进行任务拆分的 cpu 密集型运算
- 所谓的任务拆分，是将一个大任务拆分为算法上相同的小任务，直至不能拆分可以直接求解。跟递归相关的一些计算，如归并排序、斐波那契数列、都可以用分治思想进行求解
- Fork/Join 在分治的基础上加入了多线程，可以把每个任务的分解和合并交给不同的线程来完成，进一步提升了运算效率
- **Fork/Join 默认会创建与 cpu 核心数大小相同的线程池**
- 适合任务可以拆分的计算，适合cpu密集型的任务。

**继承关系**

![1620711317062](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/11/133520-568558.png)
**RecursiveTask继承关系**

![1620711444803](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/11/133727-247727.png)

#### 使用

提交给 Fork/Join 线程池的任务需要继承 RecursiveTask（有返回值）或 RecursiveAction（没有返回值），例如下面定义了一个对 1~n 之间的整数求和的任务

~~~ java
public class FolkJoinTest {

    public static void main(String[] args) {

        ForkJoinPool forkJoinPool = new ForkJoinPool(4);
        Integer invoke = forkJoinPool.invoke(new MyTask(6));
        System.out.println(invoke);
    }
}

class MyTask extends RecursiveTask<Integer>{

    private int n;

    public MyTask(int n) {
        this.n = n;
    }

    @Override
    protected Integer compute() {

//        此方法是计算的逻辑，做任务的拆分计算

//        终止条件
        if(n == 1){
            return 1;
        }
//        如果n不等于1，那么就创建一个新的任务
        MyTask myTask = new MyTask(n - 1);
//        启动一个新的线程执行我们的任务
        ForkJoinTask<Integer> fork = myTask.fork();
//        获取任务的结果
        Integer res = n+fork.join();
//        最终返回任务的执行结果
        return res;
    }
}
//因为我们设置线程池的大小是4，所以有四个线程参与计算
[ForkJoinPool-1-worker-0] - fork() 2 + {1}
[ForkJoinPool-1-worker-1] - fork() 5 + {4}
[ForkJoinPool-1-worker-0] - join() 1
[ForkJoinPool-1-worker-0] - join() 2 + {1} = 3
[ForkJoinPool-1-worker-2] - fork() 4 + {3}
[ForkJoinPool-1-worker-3] - fork() 3 + {2}
[ForkJoinPool-1-worker-3] - join() 3 + {2} = 6
[ForkJoinPool-1-worker-2] - join() 4 + {3} = 10
[ForkJoinPool-1-worker-1] - join() 5 + {4} = 15
15 
~~~

**图解**

![1609730937838](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202101/04/112859-423215.png)

#### 改进

上面的拆分任务多个任务之间具有依赖性，浪费资源。

~~~ java
class AddTask3 extends RecursiveTask<Integer> {

    int begin;
    int end;
    public AddTask3(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }
    @Override
    public String toString() {
        return "{" + begin + "," + end + '}';
    }
    @Override
    protected Integer compute() {
        // 5, 5
        if (begin == end) {
            log.debug("join() {}", begin);
            return begin;
        }
        // 4, 5
      
        if (end - begin == 1) {
            log.debug("join() {} + {} = {}", begin, end, end + begin);
            return end + begin;
        }

        // 1 5
        int mid = (end + begin) / 2; // 3
        AddTask3 t1 = new AddTask3(begin, mid); // 1,3
        t1.fork();
        AddTask3 t2 = new AddTask3(mid + 1, end); // 4,5
        t2.fork();
        log.debug("fork() {} + {} = ?", t1, t2);
        int result = t1.join() + t2.join();
        log.debug("join() {} + {} = {}", t1, t2, result);
        return result;
    }
}
//然后提交给 ForkJoinPool 来执行
public static void main(String[] args) {
 ForkJoinPool pool = new ForkJoinPool(4);
 System.out.println(pool.invoke(new AddTask3(1, 10)));
}

[ForkJoinPool-1-worker-0] - join() 1 + 2 = 3
[ForkJoinPool-1-worker-3] - join() 4 + 5 = 9
[ForkJoinPool-1-worker-0] - join() 3
[ForkJoinPool-1-worker-1] - fork() {1,3} + {4,5} = ?
[ForkJoinPool-1-worker-2] - fork() {1,2} + {3,3} = ?
[ForkJoinPool-1-worker-2] - join() {1,2} + {3,3} = 6
[ForkJoinPool-1-worker-1] - join() {1,3} + {4,5} = 15
15 

~~~

**执行过程**

![1609731352077](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202101/04/113553-107330.png)

## 并发工具JUC

`JUC—java util concurrent`

###  AQS 原理

#### 概述

- 全称是 AbstractQueuedSynchronizer(抽象队列同步器)，是**阻塞式锁**和相关的同步器工具的框架，也就是其他的同步器框架都是继承与此框架的，调用父类中的方法。（CAS是一种乐观锁)，基于模板的方法实现的。AQS是一种阻塞式的锁，类似于我们的synchronized锁。

- 什么是队列同步器？

  队列同步器（AbstractQueuedSynchronizer），是用来构建锁或者其他同步组件的基础框架，他使用一个int类型的成员变量表示同步状态，通过内置的fifo队列完成资源获取线程的排队工作，同步器的主要使用方式是继承，子类通过继承的方法实现抽象方法来管理同步状态。同步器既可以支持独占式的获取同步状态，也可以支持共享式的获取同步状态，同步器面向的是锁的实现者。

**类图继承关系**

![1614075954630](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/23/182557-47954.png)

同步器的实现是基于模板设计模式

**源码**

~~~ java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    /** Use serial ID even though all fields transient. */
    private static final long serialVersionUID = 3737899427754241961L;

    /**
     * Empty constructor for use by subclasses.
     */
    protected AbstractOwnableSynchronizer() { }

    /**
     * The current owner of exclusive mode synchronization.
     */
    private transient Thread exclusiveOwnerThread;

    /**
     * Sets the thread that currently owns exclusive access.
     * A {@code null} argument indicates that no thread owns access.
     * This method does not otherwise impose any synchronization or
     * {@code volatile} field accesses.
     * @param thread the owner thread
     */
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }

    /**
     * Returns the thread last set by {@code setExclusiveOwnerThread},
     * or {@code null} if never set.  This method does not otherwise
     * impose any synchronization or {@code volatile} field accesses.
     * @return the owner thread
     */
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
~~~

**特点**

- 用 state （整数值）属性来表示资源的状态（**分独占模式和共享模式**），子类需要定义如何维护这个状态，控制如何获取锁和释放锁

  ~~~ java
  private volatile int state;
  ~~~

  - getState - 获取 state 状态
  - setState - 设置 state 状态
  - compareAndSetState - cas 机制设置 state 状态（防止多个线程同时修改状态）
  - 独占模式是只有一个线程能够访问资源，而共享模式可以允许多个线程访问资源

- **提供了基于 FIFO 的等待队列，类似于 Monitor 的 EntryList**

- **条件变量来实现等待、唤醒机制，支持多个条件变量，类似于 Monitor 的 WaitSet**

#### 如何使用

~~~ java
//子类主要实现这样一些方法（默认抛出 UnsupportedOperationException）
protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }
protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }
protected boolean tryReleaseShared(int arg) {
        throw new UnsupportedOperationException();
    }
protected boolean isHeldExclusively() {
        throw new UnsupportedOperationException();
    }
//上面方法默认都是抛出异常，可以对其进行覆盖操作
~~~

##### **获取锁**

~~~java
// 如果获取锁失败
if (!tryAcquire(arg)) {
 // 入队, 可以选择阻塞当前线程 park unpark
}
//只会尝试一次获取锁，获取锁成功的话就可以修改状态标记
~~~

##### **释放锁**

~~~ java
// 如果释放锁成功
if (tryRelease(arg)) {//返回true表示释放锁成功
 // 让阻塞线程恢复运行
}
~~~

> 主要是通过继承阻塞队列来重用父类的功能。

#### 实现不可重入锁

##### 自定义同步器

~~~ java
 //    实现同步器类 独占锁，需要实现以下方法
    class MySync extends AbstractQueuedSynchronizer {

        //        尝试获取锁
        @Override
        protected boolean tryAcquire(int arg) {
//        初始值0表示未加锁，1表示加锁，使用cas算法是防止有其他线程此时也加锁
            if (compareAndSetState(0, 1)) {
//            加锁成功
//            设置当前持有锁的线程
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
//        没有加锁成功
            return false;
        }

        //      尝试释放锁
        @Override
        protected boolean tryRelease(int arg) {
            /**
             * state是volatile修饰的，在这里吧statze的设置放在setExclusiveOwnerThread(null);后面是防止对owner
             * 的修改对其他线程不可见，volatile变量可以保证在其之前对变量的修改对其他的线程可见
             * volatile在写之前加了屏障
             */
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        //    是否持有独占锁
        @Override
        protected boolean isHeldExclusively() {
//        等于1就说明当前线程持有锁
            return getState() == 1;
        }

        //    条件变量
        public Condition newCondition() {

//        ConditionObject 是AQS中提供的内部类
            return new ConditionObject();
        }
    }

~~~

##### 实现自定义锁

- 有了自定义同步器，很容易复用 AQS ，实现一个功能完备的自定义锁

~~~ java
public class ABStEST {
    public static void main(String[] args) {
//        创建一把锁
        MyLock myLock = new MyLock();
        new Thread(()->{
            myLock.lock();//加锁
//            myLock.lock();错误，不可重入锁
            try {
                Thread.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            try {
                System.out.println(Thread.currentThread().getName()+"  locking....");
            }finally {
                System.out.println(Thread.currentThread().getName()+"  unlocking ......");
                myLock.unlock();
            }
        },"t1").start();

        new Thread(()->{
            myLock.lock();//加锁

            try {
                System.out.println(Thread.currentThread().getName()+"  locking....");
            }finally {
                System.out.println(Thread.currentThread().getName()+"  unlocking ......");
                myLock.unlock();
            }
        },"t2").start();

    }
}


/**
 * 自定义锁有两步
 * 1 实现Lock接口
 * 2 继承AbstractQueuedSynchronizer同步器类
 */
//自定义锁，不可重入锁
class MyLock implements Lock {
    private MySync mySync=new MySync();

    //加锁，第一次不成功的话会放入阻塞队列中等待
    @Override
    public void lock() {
//        实现加锁功能，如果加锁不成功，就会放入等待队列
        mySync.acquire(1);

    }

    //加锁过程中可以被打断，可以在加锁的过程中被其他线程打断
    @Override
    public void lockInterruptibly() throws InterruptedException {
        mySync.acquireInterruptibly(1);

    }

    //尝试加锁，一次不成功就会返回，不会死等，要和lock区分开
    @Override
    public boolean tryLock() {

        return mySync.tryAcquire(1);
    }

    //带超时的加锁
    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return mySync.tryAcquireNanos(1,unit.toNanos(time));
    }

    //解锁
    @Override
    public void unlock() {
        /**
         * tryrelase仅仅将状态修改为0，owner修改为null,不会唤醒等待队列中的线程
         * relase还会唤醒正在阻塞的线程
         */
        mySync.release(1);

    }

    // 创建条件变量
    @Override
    public Condition newCondition() {
        return mySync.newCondition();
    }
}
~~~

### LOCK接口

Lock接口提供了synchronized关键字所不具备的特征如下：

1. 尝试非阻塞的获取锁，当前线程尝试获取锁的时候，如果这一时刻锁没有被其他线程获取到，那么当前线程成功获取并持有锁。
2. 可以被中断的获取锁，与synchronized不同，获取锁的线程可以相应中断，当获取到锁的线程被中断的时候，中断异常将会抛出，同时会释放锁
3. 超时获取锁，在指定时间之前获取锁，如果没有获取到锁，那么就会返回。

**Lock是一个接口**

~~~ sql
public interface Lock
~~~

**Lock接口提供的方法**

![1614075116624](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/23/181159-489544.png)

**继承关系**

![1620714935346](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/11/143537-848634.png)

> LOck接口的实现都是通过聚合一个同步器的子类来完成线程的访问控制

**Lock接口的实现子类**

![1614075272189](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/23/181434-121988.png)

**locks包类图结构**

![1620800927205](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/12/142851-698770.png)

## ReentrantLock 原理(重入锁)

- 表示该锁支持一个线程对资源进行重复加锁，该锁还支持获取锁时的公平与非公平性



![1610151297656](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/081459-832258.png)

### 非公平锁的实现原理

**加锁的流程实现**

先从构造器开始看，默认为非公平锁实现

~~~ java
public ReentrantLock() {
  //默认是非公平的实现
 sync = new NonfairSync();
}
//NonfairSync 继承自 AQS
public void lock() {
  //在这里调用的是队列同步器的加锁实现，并且调用的是非公平的实现
        sync.lock();
    }
//非公平中加锁的实现
final void lock() {
  //使用的是cas方法加锁
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
~~~

**加锁的过程**

没有竞争的时候

![1610151812866](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/082335-485435.png)

~~~ java
final void lock() {
            if (compareAndSetState(0, 1))//设置为1表示加锁
                setExclusiveOwnerThread(Thread.currentThread());//设置owner为当前的线程
            else
                acquire(1);
        }
~~~

**当出现第一个竞争时候**

~~~ java
final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
//也就是进入else语句块
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&//进行尝试加锁
            //如果加锁没有成功，尝试创建节点对象加入阻塞队列
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
~~~

![1610152163512](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/082926-366104.png)

Thread-1 执行了

- CAS 尝试将 state 由 0 改为 1，结果失败
- 进入 tryAcquire 逻辑，这时 state 已经是1，结果仍然失败
- 接下来进入 addWaiter 逻辑，构造 Node 队列
  - 图中黄色三角表示该 Node 的 waitStatus 状态，其中 0 为默认正常状态
  - Node 的创建是懒惰的
  - 其中第一个 Node 称为 Dummy（哑元）或哨兵，用来占位，并不关联线程
  - 把当前没有获取锁的线程关联到node对象上，然后添加到队列中等待锁。

![1610152307914](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/083150-51836.png)

当前线程进入 acquireQueued 逻辑

-  acquireQueued 会在一个死循环中不断尝试获得锁，失败后进入 park 阻塞
-  如果自己是紧邻着 head（排第二位），那么再次 tryAcquire 尝试获取锁，当然这时 state 仍为 1，失败
- 进入 shouldParkAfterFailedAcquire 逻辑，将前驱 node，即 head 的 waitStatus 改为 -1，这次返回 false

**源码解读**

~~~ java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();//获取p节点的前驱节点
                if (p == head && tryAcquire(arg)) {//再次尝试加锁
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
~~~

![1610152514504](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/113411-491780.png)

-  shouldParkAfterFailedAcquire 执行完毕回到 acquireQueued ，再次 tryAcquire 尝试获取锁，当然这时
  state 仍为 1，失败
- 当再次进入 shouldParkAfterFailedAcquire 时，这时因为其前驱 node 的 waitStatus 已经是 -1，这次返回
  true
- 进入 parkAndCheckInterrupt， Thread-1 park（灰色表示）

![1610152636727](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1610152636727.png)

再次有多个线程经历上述过程竞争失败，变成这个样子

![1610152660201](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/083742-157536.png)

Thread-0 释放锁，进入 tryRelease 流程，如果成功

- 设置 exclusiveOwnerThread 为 null
- state = 0

~~~ java
 public void unlock() {
        sync.release(1);
    }
//队列同步器的释放锁方法
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);//唤醒其后继节点
            return true;
        }
        return false;
    }
//释放锁的逻辑
protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

~~~

![1610153287637](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/084814-366420.png)

- 当前队列不为 null，并且 head 的 waitStatus = -1，进入 unparkSuccessor 流程
- 找到队列中离 head 最近的一个 Node（没取消的），unpark 恢复其运行，本例中即为 Thread-1
- 回到 Thread-1 的 acquireQueued 流程

![1610153399056](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/113736-100562.png)

如果加锁成功（没有竞争），会设置

- exclusiveOwnerThread 为 Thread-1，state = 1
- head 指向刚刚 Thread-1 所在的 Node，该 Node 清空 Thread
- 原本的 head 因为从链表断开，而可被垃圾回收

如果这时候有其它线程来竞争（非公平的体现），例如这时有 Thread-4 来了

![1610153543523](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/113755-809093.png)

如果不巧又被 Thread-4 占了先

- Thread-4 被设置为 exclusiveOwnerThread，state = 1
- Thread-1 再次进入 acquireQueued 流程，获取锁失败，重新进入 park 阻塞

#### 加锁源码解读

~~~ java
  // Sync 继承自 AQS
        static final class NonfairSync extends Sync {
            private static final long serialVersionUID = 7316153563782823691L;

            // 加锁实现
            final void lock() {
                // 首先用 cas 尝试（仅尝试一次）将 state 从 0 改为 1, 如果成功表示获得了独占锁
                if (compareAndSetState(0, 1))
                    setExclusiveOwnerThread(Thread.currentThread());
                else
                    // 如果尝试失败，进入 ㈠
                    acquire(1);
            }

            // ㈠ AQS 继承过来的方法, 方便阅读, 放在此处
            public final void acquire(int arg) {
                // ㈡ tryAcquire
                if (
                        !tryAcquire(arg) &&
                                // 当 tryAcquire 返回为 false 时, 先调用 addWaiter ㈣, 接着 acquireQueued ㈤
                                acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
                ) {
                    selfInterrupt();
                }
            }

            // ㈡ 进入 ㈢
            protected final boolean tryAcquire(int acquires) {
                return nonfairTryAcquire(acquires);
            }

            // ㈢ Sync 继承过来的方法, 方便阅读, 放在此处
            final boolean nonfairTryAcquire(int acquires) {
                final Thread current = Thread.currentThread();
                int c = getState();
                // 如果还没有获得锁
                if (c == 0) {
                    // 尝试用 cas 获得, 这里体现了非公平性: 不去检查 AQS 队列
                    if (compareAndSetState(0, acquires)) {
                        setExclusiveOwnerThread(current);
                        return true;
                    }
                }
                // 如果已经获得了锁, 线程还是当前线程, 表示发生了锁重入
                else if (current == getExclusiveOwnerThread()) {
                    // state++
                    int nextc = c + acquires;
                    if (nextc < 0) // overflow
                        throw new Error("Maximum lock count exceeded");
                    setState(nextc);
                    return true;
                }
                // 获取失败, 回到调用处
                return false;
            }

            // ㈣ AQS 继承过来的方法, 方便阅读, 放在此处
            private Node addWaiter(Node mode) {
                // 将当前线程关联到一个 Node 对象上, 模式为独占模式
                Node node = new Node(Thread.currentThread(), mode);
                // 如果 tail 不为 null, cas 尝试将 Node 对象加入 AQS 队列尾部
                Node pred = tail;
                if (pred != null) {
                    node.prev = pred;
                    if (compareAndSetTail(pred, node)) {
                        // 双向链表
                        pred.next = node;
                        return node;
                    }
                }
                // 尝试将 Node 加入 AQS, 进入 ㈥
                enq(node);
                return node;
            }

            // ㈥ AQS 继承过来的方法, 方便阅读, 放在此处
            private Node enq(final Node node) {
                for (;;) {
                    Node t = tail;
                    if (t == null) {
                        // 还没有, 设置 head 为哨兵节点（不对应线程，状态为 0）
                        if (compareAndSetHead(new Node())) {
                            tail = head;
                        }
                    } else {
                        // cas 尝试将 Node 对象加入 AQS 队列尾部
                        node.prev = t;
                        if (compareAndSetTail(t, node)) {
                            t.next = node;
                            return t;
                        }
                    }
                }
            }

            // ㈤ AQS 继承过来的方法, 方便阅读, 放在此处
            final boolean acquireQueued(final Node node, int arg) {
                boolean failed = true;
                try {
                    boolean interrupted = false;
                    for (;;) {
                        final Node p = node.predecessor();
                        // 上一个节点是 head, 表示轮到自己（当前线程对应的 node）了, 尝试获取
                        if (p == head && tryAcquire(arg)) {
                            // 获取成功, 设置自己（当前线程对应的 node）为 head
                            setHead(node);
                            // 上一个节点 help GC
                            p.next = null;
                            failed = false;
                            // 返回中断标记 false
                            return interrupted;
                            注意
                            是否需要 unpark 是由当前节点的前驱节点的 waitStatus == Node.SIGNAL 来决定，而不是本节点的
                            waitStatus 决定
                            解锁源码
                        }
                        if (
                            // 判断是否应当 park, 进入 ㈦
                                shouldParkAfterFailedAcquire(p, node) &&
                                        // park 等待, 此时 Node 的状态被置为 Node.SIGNAL ㈧
                                        parkAndCheckInterrupt()
                        ) {
                            interrupted = true;
                        }
                    }
                } finally {
                    if (failed)
                        cancelAcquire(node);
                }
            }

            // ㈦ AQS 继承过来的方法, 方便阅读, 放在此处
            private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
                // 获取上一个节点的状态
                int ws = pred.waitStatus;
                if (ws == Node.SIGNAL) {
                    // 上一个节点都在阻塞, 那么自己也阻塞好了
                    return true;
                }
                // > 0 表示取消状态
                if (ws > 0) {
                    // 上一个节点取消, 那么重构删除前面所有取消的节点, 返回到外层循环重试
                    do {
                        node.prev = pred = pred.prev;
                    } while (pred.waitStatus > 0);
                    pred.next = node;
                } else {
                    // 这次还没有阻塞
                    // 但下次如果重试不成功, 则需要阻塞，这时需要设置上一个节点状态为 Node.SIGNAL
                    compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
                }
                return false;
            }

            // ㈧ 阻塞当前线程
            private final boolean parkAndCheckInterrupt() {
                LockSupport.park(this);
                return Thread.interrupted();
            }
        }
~~~

> 是否需要 unpark 是由当前节点的前驱节点的 waitStatus == Node.SIGNAL 来决定，而不是本节点的
> waitStatus 决定

#### 解锁源码

~~~ java
  // Sync 继承自 AQS
        static final class NonfairSync extends Sync {
            // 解锁实现
            public void unlock() {
                sync.release(1);
            }

            // AQS 继承过来的方法, 方便阅读, 放在此处
            public final boolean release(int arg) {
                // 尝试释放锁, 进入 ㈠
                if (tryRelease(arg)) {
                    // 队列头节点 unpark
                    Node h = head;
                    if (
                        // 队列不为 null
                            h != null &&
                                    // waitStatus == Node.SIGNAL 才需要 unpark
                                    h.waitStatus != 0
                    ) {
                        // unpark AQS 中等待的线程, 进入 ㈡
                        unparkSuccessor(h);
                    }
                    return true;
                }
                return false;
            }

            // ㈠ Sync 继承过来的方法, 方便阅读, 放在此处
            protected final boolean tryRelease(int releases) {
                // state--
                int c = getState() - releases;
                if (Thread.currentThread() != getExclusiveOwnerThread())
                    throw new IllegalMonitorStateException();
                boolean free = false;
                // 支持锁重入, 只有 state 减为 0, 才释放成功
                if (c == 0) {
                    free = true;
                    setExclusiveOwnerThread(null);
                }
                setState(c);
                return free;
            }

            // ㈡ AQS 继承过来的方法, 方便阅读, 放在此处
            private void unparkSuccessor(Node node) {
                // 如果状态为 Node.SIGNAL 尝试重置状态为 0
                // 不成功也可以
                int ws = node.waitStatus;
                if (ws < 0) {
                    compareAndSetWaitStatus(node, ws, 0);
                }
                // 找到需要 unpark 的节点, 但本节点从 AQS 队列中脱离, 是由唤醒节点完成的
                2. 可重入原理
                Node s = node.next;
                // 不考虑已取消的节点, 从 AQS 队列从后至前找到队列最前面需要 unpark 的节点
                if (s == null || s.waitStatus > 0) {
                    s = null;
                    for (Node t = tail; t != null && t != node; t = t.prev)
                        if (t.waitStatus <= 0)
                            s = t;
                }
                if (s != null)
                    LockSupport.unpark(s.thread);
            }
        }
~~~

### 可重入原理

~~~ java
 static final class NonfairSync extends Sync {
            // ...

            // Sync 继承过来的方法, 方便阅读, 放在此处
            final boolean nonfairTryAcquire(int acquires) {
                final Thread current = Thread.currentThread();
              //获取当前锁的状态
                int c = getState();
              //第一次获取锁的状态
                if (c == 0) {
                  //如果锁空闲，那么使用cas方法进行加锁操作
                    if (compareAndSetState(0, acquires)) {
                        setExclusiveOwnerThread(current);
                        return true;
                    }
                }
                // 如果已经获得了锁, 线程还是当前线程, 表示发生了锁重入
                else if (current == getExclusiveOwnerThread()) {
                    // state++，acquires为1
                    int nextc = c + acquires;
                    if (nextc < 0) // overflow
                        throw new Error("Maximum lock count exceeded");
                    setState(nextc);
                    return true;
                }
                return false;
            }

            // Sync 继承过来的方法, 方便阅读, 放在此处
            protected final boolean tryRelease(int releases) {
                // state--，这里release的值一般是1，因为是重入锁，所以每次释放，都会-1
                int c = getState() - releases;
                if (Thread.currentThread() != getExclusiveOwnerThread())
                    throw new IllegalMonitorStateException();
                boolean free = false;
                // 支持锁重入, 只有 state 减为 0, 才释放成功
                if (c == 0) {
                    free = true;
                    setExclusiveOwnerThread(null);
                }
                //否则怒释放锁，仅仅是让锁的计数器-1
                setState(c);
                return free;
            }
        }
~~~

### 可打断模式

#### 不可打断模式

在此模式下，即使它被打断，仍会驻留在 AQS 队列中，一直要等到获得锁后方能得知自己被打断了

~~~ java
  // Sync 继承自 AQS
        static final class NonfairSync extends Sync {
            // ...

            private final boolean parkAndCheckInterrupt() {
                // 如果打断标记已经是 true, 则 park 会失效
                LockSupport.park(this);
                // interrupted 会清除打断标记，返回是否被打断过
                return Thread.interrupted();
            }

            final boolean acquireQueued(final Node node, int arg) {
                boolean failed = true;
                try {
                  //默认打断表示为false
                    boolean interrupted = false;
                    for (;;) {
                        final Node p = node.predecessor();
                        if (p == head && tryAcquire(arg)) {
                            setHead(node);
                            p.next = null;
                            failed = false;
                            // 还是需要获得锁后, 才能返回打断状态
                          /
                            return interrupted;
                        }
                        if (
                                shouldParkAfterFailedAcquire(p, node) &&
                                        parkAndCheckInterrupt()
                        ) {
                            // 如果是因为 interrupt 被唤醒, 返回打断状态为 true
                            interrupted = true;
                        }
                    }
                } finally {
                    if (failed)
                        cancelAcquire(node);
                }
            }

            public final void acquire(int arg) {
                if (
                        !tryAcquire(arg) &&
                                acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
                ) {
                    // 如果打断状态为 true
                    selfInterrupt();
                }
            }

            static void selfInterrupt() {
                // 重新产生一次中断
                Thread.currentThread().interrupt();
            }
        }
~~~

#### 可打断模式

~~~ java
static final class NonfairSync extends Sync {
            public final void acquireInterruptibly(int arg) throws InterruptedException {
                if (Thread.interrupted())
                    throw new InterruptedException();
                // 如果没有获得到锁, 进入 ㈠
                if (!tryAcquire(arg))
                    doAcquireInterruptibly(arg);
            }

            // ㈠ 可打断的获取锁流程
            private void doAcquireInterruptibly(int arg) throws InterruptedException {
                final Node node = addWaiter(Node.EXCLUSIVE);
                boolean failed = true;
                try {
                    for (;;) {
                        final Node p = node.predecessor();
                        if (p == head && tryAcquire(arg)) {
                            setHead(node);
                            p.next = null; // help GC
                            failed = false;
                            return;
                        }
                        if (shouldParkAfterFailedAcquire(p, node) &&
                                parkAndCheckInterrupt()) {
                            // 在 park 过程中如果被 interrupt 会进入此
                            // 这时候抛出异常, 而不会再次进入 for (;;)
                            throw new InterruptedException();
                        }
                    }
                } finally {
                    if (failed)
                        cancelAcquire(node);
                }
            }
        }=
~~~

> 不可打断模式只是重新设置打断标记而已，可打断模式会抛出异常。

### 公平锁实现原理

~~~ java
 static final class FairSync extends Sync {
            private static final long serialVersionUID = -3000897897090466540L;
            final void lock() {
                acquire(1);
            }

            // AQS 继承过来的方法, 方便阅读, 放在此处
            public final void acquire(int arg) {
                if (
                        !tryAcquire(arg) &&
                                acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
                ) {
                    selfInterrupt();
                }
            }
            // 与非公平锁主要区别在于 tryAcquire 方法的实现
            protected final boolean tryAcquire(int acquires) {
                final Thread current = Thread.currentThread();
                int c = getState();
                if (c == 0) {
                    // 先检查 AQS 队列中是否有前驱节点, 没有才去竞争
                    if (!hasQueuedPredecessors() &&
                            compareAndSetState(0, acquires)) {
                        setExclusiveOwnerThread(current);
                        return true;
                    }
                }
                else if (current == getExclusiveOwnerThread()) {
                    int nextc = c + acquires;
                    if (nextc < 0)
                        throw new Error("Maximum lock count exceeded");
                    setState(nextc);
                    return true;
                }
                return false;
            }

            // ㈠ AQS 继承过来的方法, 方便阅读, 放在此处
            public final boolean hasQueuedPredecessors() {
                Node t = tail;
                Node h = head;
                Node s;
                // h != t 时表示队列中有 Node
                return h != t &&
                        (
                                // (s = h.next) == null 表示队列中还有没有老二
                                (s = h.next) == null ||
                // 或者队列中老二线程不是此线程
                s.thread != Thread.currentThread()
 );
            }
        }
~~~

### 条件变量的实现原理

每个条件变量其实就对应着一个等待队列，其实现类是 ConditionObject

condition接口提供了一组监视器方法，可以和Lock配合实现等待通知模式，condition对象是由Lock对象创建出来的，也就是说condition对象依赖于kock对象。

object也是一个对象监视器，一个对象拥有一个同步队列和等待队列

并发包中的lock拥有一个同步队列和多个等待队列

condition的实现是同步器的内部类

下面我们可以看到，实现在同步器类中，因此每一个conditoin对象都可以访问同步器提供的方法，相当于condition对象都拥有所属同步器的引用。

**Condition拥有的方法**

![1621223132698](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/114534-658958.png)

**继承关系**

![1614081190784](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/23/195314-244438.png)

#### await 流程

- 开始 Thread-0 持有锁，调用 await，进入 ConditionObject 的 addConditionWaiter 流程
- 创建新的 Node 状态为 -2（Node.CONDITION），关联 Thread-0，加入等待队列尾部

![1610155364444](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/092251-461763.png)

接下来进入 AQS 的 fullyRelease 流程，释放同步器上的锁

![1610155421827](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/092432-409623.png)

unpark AQS 队列中的下一个节点，竞争锁，假设没有其他竞争线程，那么 Thread-1 竞争成功

![1610155501534](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/092503-543094.png)

park 阻塞 Thread-0

![1610155527528](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1610155527528.png)

#### Signal流程

~~~ java
public final void signal() {
            if (!isHeldExclusively())//判断当前线程是否是锁的持有者
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;//找队首元素
            if (first != null)
                doSignal(first);
        }
//把等待队列中的线程转移到等待队列中去获取锁
 private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)//获取下一个节点
                    lastWaiter = null;
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
final boolean transferForSignal(Node node) {
        /*
         * If cannot change waitStatus, the node has been cancelled.
         */
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;

        /*
         * Splice onto queue and try to set waitStatus of predecessor to
         * indicate that thread is (probably) waiting. If cancelled or
         * attempt to set waitStatus fails, wake up to resync (in which
         * case the waitStatus can be transiently and harmlessly wrong).
         */
        Node p = enq(node);
        int ws = p.waitStatus;
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
~~~

假设 Thread-1 要来唤醒 Thread-0

![1610156363063](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/093924-510160.png)

进入 ConditionObject 的 doSignal 流程，取得等待队列中第一个 Node，即 Thread-0 所在 Node

![1610156397464](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/11/152239-523521.png)

执行 transferForSignal 流程，将该 Node 加入 AQS 队列尾部，将 Thread-0 的 waitStatus 改为 0，Thread-3 的
waitStatus 改为 -1

![1610156424038](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/094031-587595.png)

Thread-1 释放锁，进入 unlock 流程，

#### 源码

~~~ java
public class ConditionObject implements Condition, java.io.Serializable {
            private static final long serialVersionUID = 1173984872572414699L;

            // 第一个等待节点
            private transient Node firstWaiter;

            // 最后一个等待节点
            private transient Node lastWaiter;
            public ConditionObject() { }
            // ㈠ 添加一个 Node 至等待队列
            private Node addConditionWaiter() {
                Node t = lastWaiter;
                // 所有已取消的 Node 从队列链表删除, 见 ㈡
                if (t != null && t.waitStatus != Node.CONDITION) {
                    unlinkCancelledWaiters();
                    t = lastWaiter;
                }
                // 创建一个关联当前线程的新 Node, 添加至队列尾部
                Node node = new Node(Thread.currentThread(), Node.CONDITION);
                if (t == null)
                    firstWaiter = node;
                else
                    t.nextWaiter = node;
                lastWaiter = node;
                return node;
            }
            // 唤醒 - 将没取消的第一个节点转移至 AQS 队列
            private void doSignal(Node first) {
                do {
                    // 已经是尾节点了
                    if ( (firstWaiter = first.nextWaiter) == null) {
                        lastWaiter = null;
                    }
                    first.nextWaiter = null;
                } while (
                    // 将等待队列中的 Node 转移至 AQS 队列, 不成功且还有节点则继续循环 ㈢
                        !transferForSignal(first) &&
                                // 队列还有节点
                                (first = firstWaiter) != null
                );
            }

            // 外部类方法, 方便阅读, 放在此处
            // ㈢ 如果节点状态是取消, 返回 false 表示转移失败, 否则转移成功
            final boolean transferForSignal(Node node) {
                // 如果状态已经不是 Node.CONDITION, 说明被取消了
                if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
                    return false;
                // 加入 AQS 队列尾部
                Node p = enq(node);
                int ws = p.waitStatus;
                if (
                    // 上一个节点被取消
                        ws > 0 ||
                                // 上一个节点不能设置状态为 Node.SIGNAL
                                !compareAndSetWaitStatus(p, ws, Node.SIGNAL)
                ) {
                    // unpark 取消阻塞, 让线程重新同步状态
                    LockSupport.unpark(node.thread);
                }
                return true;
            }
            // 全部唤醒 - 等待队列的所有节点转移至 AQS 队列
            private void doSignalAll(Node first) {
                lastWaiter = firstWaiter = null;
                do {
                    Node next = first.nextWaiter;
                    first.nextWaiter = null;
                    transferForSignal(first);
                    first = next;
                } while (first != null);
            }

            // ㈡
            private void unlinkCancelledWaiters() {
                // ...
            }
            // 唤醒 - 必须持有锁才能唤醒, 因此 doSignal 内无需考虑加锁
            public final void signal() {
                if (!isHeldExclusively())
                    throw new IllegalMonitorStateException();
                Node first = firstWaiter;
                if (first != null)
                    doSignal(first);
            }
            // 全部唤醒 - 必须持有锁才能唤醒, 因此 doSignalAll 内无需考虑加锁
            public final void signalAll() {
                if (!isHeldExclusively())
                    throw new IllegalMonitorStateException();
                Node first = firstWaiter;
                if (first != null)
                    doSignalAll(first);
            }
            // 不可打断等待 - 直到被唤醒
            public final void awaitUninterruptibly() {
                // 添加一个 Node 至等待队列, 见 ㈠
                Node node = addConditionWaiter();
                // 释放节点持有的锁, 见 ㈣
                int savedState = fullyRelease(node);
                boolean interrupted = false;
                // 如果该节点还没有转移至 AQS 队列, 阻塞
                while (!isOnSyncQueue(node)) {
                    // park 阻塞
                    LockSupport.park(this);
                    // 如果被打断, 仅设置打断状态
                    if (Thread.interrupted())
                        interrupted = true;
                }
                // 唤醒后, 尝试竞争锁, 如果失败进入 AQS 队列
                if (acquireQueued(node, savedState) || interrupted)
                    selfInterrupt();
            }
            
            // 外部类方法, 方便阅读, 放在此处
            // ㈣ 因为某线程可能重入，需要将 state 全部释放
            final int fullyRelease(Node node) {
                boolean failed = true;
                try {
                    int savedState = getState();
                    if (release(savedState)) {
                        failed = false;
                        return savedState;
                    } else {
                        throw new IllegalMonitorStateException();
                    }
                } finally {
                    if (failed)
                        node.waitStatus = Node.CANCELLED;
                }
            }
            // 打断模式 - 在退出等待时重新设置打断状态
            private static final int REINTERRUPT = 1;
            // 打断模式 - 在退出等待时抛出异常
            private static final int THROW_IE = -1;
            // 判断打断模式
            private int checkInterruptWhileWaiting(Node node) {
                return Thread.interrupted() ?
                        (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
                        0;
            }
            // ㈤ 应用打断模式
            private void reportInterruptAfterWait(int interruptMode)
                    throws InterruptedException {
                if (interruptMode == THROW_IE)
                    throw new InterruptedException();
                else if (interruptMode == REINTERRUPT)
                    selfInterrupt();
            }
            // 等待 - 直到被唤醒或打断
            public final void await() throws InterruptedException {
                if (Thread.interrupted()) {
                    throw new InterruptedException();
                }
                // 添加一个 Node 至等待队列, 见 ㈠
                Node node = addConditionWaiter();
                // 释放节点持有的锁
                int savedState = fullyRelease(node);
                int interruptMode = 0;
                // 如果该节点还没有转移至 AQS 队列, 阻塞
                while (!isOnSyncQueue(node)) {
                    // park 阻塞
                    LockSupport.park(this);
                    // 如果被打断, 退出等待队列
                    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                        break;
                }
                // 退出等待队列后, 还需要获得 AQS 队列的锁
                if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                    interruptMode = REINTERRUPT;
                // 所有已取消的 Node 从队列链表删除, 见 ㈡
                if (node.nextWaiter != null)
                    unlinkCancelledWaiters();
                // 应用打断模式, 见 ㈤
                if (interruptMode != 0)
                    reportInterruptAfterWait(interruptMode);
            }
            // 等待 - 直到被唤醒或打断或超时
            public final long awaitNanos(long nanosTimeout) throws InterruptedException {
                if (Thread.interrupted()) {
                    throw new InterruptedException();
                }
                // 添加一个 Node 至等待队列, 见 ㈠
                Node node = addConditionWaiter();
                // 释放节点持有的锁
                int savedState = fullyRelease(node);
                // 获得最后期限
                final long deadline = System.nanoTime() + nanosTimeout;
                int interruptMode = 0;
                // 如果该节点还没有转移至 AQS 队列, 阻塞
                while (!isOnSyncQueue(node)) {
                    // 已超时, 退出等待队列
                    if (nanosTimeout <= 0L) {
                        transferAfterCancelledWait(node);
                        break;
                    }
                    // park 阻塞一定时间, spinForTimeoutThreshold 为 1000 ns
                    if (nanosTimeout >= spinForTimeoutThreshold)
                        LockSupport.parkNanos(this, nanosTimeout);
                    // 如果被打断, 退出等待队列
                    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                        break;
                    nanosTimeout = deadline - System.nanoTime();
                }
                // 退出等待队列后, 还需要获得 AQS 队列的锁
                if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                    interruptMode = REINTERRUPT;
                // 所有已取消的 Node 从队列链表删除, 见 ㈡
                if (node.nextWaiter != null)
                    unlinkCancelledWaiters();
                // 应用打断模式, 见 ㈤
                if (interruptMode != 0)
                    reportInterruptAfterWait(interruptMode);
                return deadline - System.nanoTime();
            }
         
            // 等待 - 直到被唤醒或打断或超时, 逻辑类似于 awaitNanos
            public final boolean awaitUntil(Date deadline) throws InterruptedException {
                // ...
            }
            // 等待 - 直到被唤醒或打断或超时, 逻辑类似于 awaitNanos
            public final boolean await(long time, TimeUnit unit) throws InterruptedException {
                // ...
            }
            // 工具方法 省略 ...
        }
~~~

## 读写锁

ReentrantLock是一个排它锁，同一时刻只允许一个线程访问，而读写锁同一时刻允许多个线程访问，但是在写线程访问的时候，所有的其他读线程和写线程全部被阻塞，读写锁维护了一对锁**，读锁和写锁。也叫做读写分离**。

###  ReentrantReadWriteLock

~~~java
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable
~~~

**public class ReentrantReadWriteLock特性**

- 公平性选择
- 重进入，读线程获取读锁后，可以再次获取读锁，写线程获取写锁后，可以再次获取写锁，同时也可以获取读锁。
- 锁降级
- 读写互斥，写写互斥，读读并发。

当读操作远远高于写操作时，这时候使用 读写锁 让 读-读 可以并发，提高性能。 类似于数据库中的 select ...
from ... lock in share mode

提供一个 数据容器类，内部分别使用读锁保护数据的 read() 方法，写锁保护数据的 write() 方法

**继承结构**

![1620793807580](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/12/123011-955753.png)

**多个线程读操作**

~~~ java
public class Test50 {

    public static void main(String[] args) {
        DataContainer data=new DataContainer();

        new Thread(()->{
            data.read();
        },"t1").start();

        new Thread(()->{
            data.read();
        },"t2").start();
    }

    }

class DataContainer{
    private Object data;

    //    创建一个读写锁
    private ReentrantReadWriteLock reentrantReadWriteLock=new ReentrantReadWriteLock();
//    获取读锁
    private ReentrantReadWriteLock.ReadLock r=reentrantReadWriteLock.readLock();
//    获取写锁
    private ReentrantReadWriteLock.WriteLock w=reentrantReadWriteLock.writeLock();

    public Object read(){
        System.out.println(Thread.currentThread().getName()+"  获取读锁");
        r.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"  读取数据.....");
            Thread.sleep(5);
            return data;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName()+"  释放读锁");
            r.unlock();
        }
        return null;
    }
    public void write(){
        System.out.println(Thread.currentThread().getName()+"  获取写锁");
        w.lock();
        try{
            System.out.println(Thread.currentThread().getName()+"  写入操作....");
        }finally {
            System.out.println(Thread.currentThread().getName()+"  释放写锁");
            w.unlock();
        }
    }
}
~~~

**一个线程读一个线程写**

~~~ java
public class Test {
    public static void main(String[] args) {
        DataContainer data=new DataContainer();

        new Thread(()->{
            data.read();
        },"t1").start();

        new Thread(()->{
            data.write();
        },"t2").start();


    }
}

class DataContainer{
    private Object data;

//    创建一个读写锁
    private ReentrantReadWriteLock reentrantReadWriteLock=new ReentrantReadWriteLock();
    private ReentrantReadWriteLock.ReadLock r=reentrantReadWriteLock.readLock();
    private ReentrantReadWriteLock.WriteLock w=reentrantReadWriteLock.writeLock();


    public Object read(){
        System.out.println(Thread.currentThread().getName()+"  获取读锁");
        r.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"  读取数据.....");
            Thread.sleep(5);
          return data;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName()+"  释放读锁");
            r.unlock();
        }
        return null;
    }
    public void write(){
        System.out.println(Thread.currentThread().getName()+"  获取写锁");
       w.lock();
       try{
           System.out.println(Thread.currentThread().getName()+"  写入操作....");
       }finally {
           System.out.println(Thread.currentThread().getName()+"  释放写锁");
           w.unlock();
       }


    }
}
~~~

> 多个线程读可以并发操作，有读有写是互斥操作。

### **注意事项**

- **读锁不支持条件变量**
- 重入时升级不支持：即持有读锁的情况下去获取写锁，会导致获取写锁永久等待

~~~ java
r.lock();
        try{
        // ...
        w.lock();//不可以获取写锁，想要获取写锁，必须先把读锁释放开
        try{
        // ...
        }finally{
        w.unlock();
        }
        }finally{
        r.unlock();
        }
~~~

重入时降级支持：即持有写锁的情况下去获取读锁

~~~ java
class CachedData {
    Object data;
    // 是否有效，如果失效，需要重新计算 data
    volatile boolean cacheValid;
  //获取读写锁
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    void processCachedData() {
      //首先添加读锁
        rwl.readLock().lock();
      //判断缓存数据是否时效，如果时效，那么重新获取写锁，修改数据
        if (!cacheValid) {
            // 获取写锁前必须释放读锁
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
                // 判断是否有其它线程已经获取了写锁、更新了缓存, 避免重复更新
              //这里使用了双重检查
                if (!cacheValid) {
                    data = ...
                    cacheValid = true;
                }
                // 降级为读锁, 释放写锁, 这样能够让其它线程读取缓存
              //在写锁释放前可以先获取读锁，然后在释放写锁
                rwl.readLock().lock();
            } finally {
                rwl.writeLock().unlock();
            }
        }
        // 自己用完数据, 释放读锁
        try {
            use(data);
        } finally {
            rwl.readLock().unlock();//最后释放读锁
        }
    }
}
~~~

### 读写锁原理

#### 图解原理

读写锁用的是同一个 Sycn 同步器，因此等待队列、state 等也是同一个

**t1 w.lockt2 r.lock**

1. t1 成功上锁，流程与 ReentrantLock 加锁相比没有特殊之处，**不同是写锁状态占了 state 的低 16 位，而读锁使用的是 state 的高 16 位**

![1610163372436](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/12/122846-445249.png)

~~~ java
//写锁的加锁方法，调用的是同步器的加锁方法
public void lock() {
            sync.acquire(1);
}

//使用的是队列同步器的加锁
//aqs中的acquire()方法
public final void acquire(int arg) {
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

//子类中尝试重写获取锁的方法
protected final boolean tryAcquire(int acquires) {
            Thread current = Thread.currentThread();
            int c = getState();
  //w表示的是写锁的部分
            int w = exclusiveCount(c);
  //c不是0，可能加了读锁，也可能是写锁
            if (c != 0) {
                // (Note: if c != 0 and w == 0 then shared count != 0)
              //如果w==0，表示其他线程添加的是读锁，否则就是添加了写锁，那么就判断当前加写锁的线程是否和上次的线程是一个线程
              //下面表示添加的是读锁，后半部分表示添加的写锁是否是自己添加的是，是否发生了重入
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
              //判断写锁的重入次数是否超过最大值
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // Reentrant acquire
              //锁重入。写锁+1 
                setState(c + acquires);
                return true;
            }
  //也就是c==0
  //首先判断写锁是否应该阻塞，第二个条件是使用cas设置加锁
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
  //设置owner为当前的额线程
            setExclusiveOwnerThread(current);
            return true;
        }
~~~

2. t2 执行 r.lock，这时进入读锁的 sync.acquireShared(1) 流程，首先会进入 tryAcquireShared 流程。如果有写锁占据，那么 tryAcquireShared 返回 -1 表示失败

> tryAcquireShared 返回值表示
> -1 表示失败
>
> 0 表示成功，但后继节点不会继续唤醒
> 正数表示成功，而且数值是还有几个后继节点需要唤醒，读写锁返回 1

![1620795446425](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1620795446425.png)

~~~ java
//读锁加锁的过程
public void lock() {
            sync.acquireShared(1);
        }
//尝试去获取读锁
public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
          //如果返回-1，就会进入下面的方法
            doAcquireShared(arg);
    }

//子类的实现
protected final int tryAcquireShared(int unused) {
          
            Thread current = Thread.currentThread();
            int c = getState();
  //判断写锁部分是否为0，并且判断添加写锁的是否是当前线程，如果不是就返回-1
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;//这里返回表示失败
            int r = sharedCount(c);
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    firstReaderHoldCount++;
                } else {
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            return fullTryAcquireShared(current);
        }
~~~

3. 这时会进入 sync.doAcquireShared(1) 流程，首先也是调用 addWaiter 添加节点，不同之处在于节点被设置为Node.SHARED 模式而非 Node.EXCLUSIVE 模式，注意此时 t2 仍处于活跃状态

![1620795943701](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/12/130546-265706.png)

~~~ java
private void doAcquireShared(int arg) {
  //加入一个共享的线程节点
  //向阻塞队列中添加一个节点之后，线程还处于活跃状态，因为下面自旋操作还没有被执行
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
              //找插入节点的前驱节点
                final Node p = node.predecessor();
                if (p == head) {
                  //如果前驱节点是头结点，那么他就有资格再次去争抢锁，所以下面方法再次去尝试获取锁
                  //返回-1表示失败，返回+1表示成功
                    int r = tryAcquireShared(arg);
                  //如果换没有获取到锁，也就是返回-1，那么不会执行下面的if语句
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
              //是否应该阻塞线程，如果返回false的话，那么就会再一次执行循环操作，也就是自旋操作
              //尝试获取锁，在这里会自旋3次然后会park()当前线程，阻塞
                if (shouldParkAfterFailedAcquire(p, node) &&
                    //阻塞当前线程
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
~~~

4. t2 会看看自己的节点是不是老二，如果是，还会再次调用 tryAcquireShared(1) 来尝试获取锁
5. 如果没有成功，在 doAcquireShared 内 for (;;) 循环一次，把前驱节点的 waitStatus 改为 -1，再 for (;;) 循环一次尝试 tryAcquireShared(1) 如果还不成功，那么在 parkAndCheckInterrupt() 处 park

**t3 r.lock，t4 w.lock**

这种状态下，假设又有 t3 加读锁和 t4 加写锁，这期间 t1 仍然持有锁，就变成了下面的样子

因为t2,t3添加的是读锁，所以状态是shared状态，而t4是写锁，所以是独占状态。

并且t2,t3的waitstate是-1，表示有职责唤醒其后继的节点。

![1620796362473](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/12/131308-89689.png)

**t1 w.unlock**

这时会走到写锁的 sync.release(1) 流程，调用 sync.tryRelease(1) 成功，变成下面的样子

![1620867220119](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/13/085342-301400.png)

~~~ java
public void unlock() {
            sync.release(1);
        }

 public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
          //判断头结点是否是空和waitstate是否为0 此时为-1
            if (h != null && h.waitStatus != 0)
              //唤醒其后继节点
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

   protected final boolean tryRelease(int releases) {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
     //因为是重入锁，所以需要-1
            int nextc = getState() - releases;
     //判断锁的重入次数是否是0
            boolean free = exclusiveCount(nextc) == 0;
            if (free)
                setExclusiveOwnerThread(null);
            setState(nextc);
            return free;
        }
~~~

接下来执行唤醒流程 sync.unparkSuccessor，即让老二恢复运行，这时 t2 在 doAcquireShared 内
parkAndCheckInterrupt() 处恢复运行

这回再来一次 for (;;) 执行 tryAcquireShared 成功则让读锁计数加一

![1620867701072](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/13/090143-818838.png)

这时 t2 已经恢复运行，接下来 t2 调用 setHeadAndPropagate(node, 1)，它原本所在节点被置为头节点

![1620868029551](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1620868029551.png)

事情还没完，在 setHeadAndPropagate 方法内还会检查下一个节点是否是 shared，如果是则调用
doReleaseShared() 将 head 的状态从 -1 改为 0 并唤醒老二，这时 t3 在 doAcquireShared 内
parkAndCheckInterrupt() 处恢复运行

![1620868071976](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/13/090802-621575.png)

这回再来一次 for (;;) 执行 tryAcquireShared 成功则让读锁计数加一

![1620868103949](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/125226-269240.png)

这时 t3 已经恢复运行，接下来 t3 调用 setHeadAndPropagate(node, 1)，它原本所在节点被置为头节点

![1620868132814](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1620868132814.png)

下一个节点不是 shared 了，因此不会继续唤醒 t4 所在节点

**t2 r.unlock，t3 r.unlock**

t2 进入 sync.releaseShared(1) 中，调用 tryReleaseShared(1) 让计数减一，但由于计数还不为零

![1621227965223](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621227965223.png)

t3 进入 sync.releaseShared(1) 中，调用 tryReleaseShared(1) 让计数减一，这回计数为零了，进入
doReleaseShared() 将头节点从 -1 改为 0 并唤醒老二，即

![1621228011863](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/130709-806611.png)

之后 t4 在 acquireQueued 中 parkAndCheckInterrupt 处恢复运行，再次 for (;;) 这次自己是老二，并且没有其他
竞争，tryAcquire(1) 成功，修改头结点，流程结束

![1621228041588](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/130744-97131.png)

**源码**

~~~ java
static final class NonfairSync extends Sync {
 
 // ReadLock 方法, 方便阅读, 放在此处
 public void unlock() {
 sync.releaseShared(1);
 }
 
 // AQS 继承过来的方法, 方便阅读, 放在此处
 public final boolean releaseShared(int arg) {
 if (tryReleaseShared(arg)) {
 doReleaseShared();
 return true;
 }
 return false;
 }
 
 // Sync 继承过来的方法, 方便阅读, 放在此处
 protected final boolean tryReleaseShared(int unused) {
 // ... 省略不重要的代码
 for (;;) {
 int c = getState();
 int nextc = c - SHARED_UNIT;
 if (compareAndSetState(c, nextc)) {
 // 读锁的计数不会影响其它获取读锁线程, 但会影响其它获取写锁线程
 // 计数为 0 才是真正释放
 return nextc == 0;
 }
}
 }
 
 // AQS 继承过来的方法, 方便阅读, 放在此处
 private void doReleaseShared() {
 // 如果 head.waitStatus == Node.SIGNAL ==> 0 成功, 下一个节点 unpark
 // 如果 head.waitStatus == 0 ==> Node.PROPAGATE
 for (;;) {
 Node h = head;
 if (h != null && h != tail) {
 int ws = h.waitStatus;
 // 如果有其它线程也在释放读锁，那么需要将 waitStatus 先改为 0
 // 防止 unparkSuccessor 被多次执行
 if (ws == Node.SIGNAL) {
 if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
 continue; // loop to recheck cases
 unparkSuccessor(h);
 }
 // 如果已经是 0 了，改为 -3，用来解决传播性，见后文信号量 bug 分析
 else if (ws == 0 &&
 !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
 continue; // loop on failed CAS
 }
 if (h == head) // loop if head changed
 break;
 }
 } 
}
~~~

###  StampedLock(读锁)

该类自 JDK 8 加入，是为了进一步优化读性能，它的特点是在使用读锁、写锁时都必须配合【戳】使用
**加解读锁**

~~~ java
//加锁返回的是一个时间戳
long stamp = lock.readLock();
//解锁时候必须割据获取锁时候的时间戳进行解锁
lock.unlockRead(stamp);
~~~

**加解写锁**

~~~ java
long stamp = lock.writeLock();
//根据时间戳进行解锁
lock.unlockWrite(stamp);
~~~

乐观读，StampedLock 支持 tryOptimisticRead() 方法（乐观读），读取完毕后需要做一次 戳校验 如果校验通
过，表示这期间确实没有写操作，数据可以安全使用，如果校验没通过，需要重新获取读锁，保证数据安全。

~~~ java
long stamp = lock.tryOptimisticRead();//底层实现不会真的加锁，返回一个时间戳
// 验戳
if(!lock.validate(stamp)){//验证读锁
 // 锁升级
}
~~~

提供一个 数据容器类 内部分别使用读锁保护数据的 read() 方法，写锁保护数据的 write() 方法

~~~ java
class DataContainerStamped {
    private int data;
  //读写锁
    private final StampedLock lock = new StampedLock();
    public DataContainerStamped(int data) {
        this.data = data;
    }
    public int read(int readTime) {
      //加写锁
        long stamp = lock.tryOptimisticRead();
        log.debug("optimistic read locking...{}", stamp);
        sleep(readTime);
      //验证时间戳，如果验证通过，那么可以直接返回数据
        if (lock.validate(stamp)) {
            log.debug("read finish...{}, data:{}", stamp, data);
            return data;
        }
        // 锁升级 - 读锁，验证失败，锁升级，升级为读锁，从乐观读锁到读锁的升级
        log.debug("updating to read lock... {}", stamp);
        try {
            stamp = lock.readLock();
            log.debug("read lock {}", stamp);
            sleep(readTime);
            log.debug("read finish...{}, data:{}", stamp, data);
            return data;
        } finally {
            log.debug("read unlock {}", stamp);
            lock.unlockRead(stamp);
        }
    }
    public void write(int newData) {
      //加写锁
        long stamp = lock.writeLock();
        log.debug("write lock {}", stamp);
        try {
            sleep(2);
            this.data = newData;
        } finally {
            log.debug("write unlock {}", stamp);
            lock.unlockWrite(stamp);
        }
    }
}
~~~

**代码测试**

~~~ java
 public static void main(String[] args) {
        DataContainerStamped dataContainer = new DataContainerStamped(1);
        new Thread(() -> {
            dataContainer.read(1);
        }, "t1").start();
        sleep(0.5);
        new Thread(() -> {
            dataContainer.read(0);
        }, "t2").start();
    }
//从输出结果看，实际上是没有加锁的，乐观读锁，实际上就是没有真正的加锁
15:58:50.217 c.DataContainerStamped [t1] - optimistic read locking...256
15:58:50.717 c.DataContainerStamped [t2] - optimistic read locking...256
15:58:50.717 c.DataContainerStamped [t2] - read finish...256, data:1
15:58:51.220 c.DataContainerStamped [t1] - read finish...256, data:1 
~~~

**测试 读-写 时优化读补加读锁**

~~~ java
 public static void main(String[] args) {
        DataContainerStamped dataContainer = new DataContainerStamped(1);

        new Thread(() -> {
            dataContainer.read(1);
        }, "t1").start();
        sleep(0.5);
        new Thread(() -> {
            dataContainer.write(100);
        }, "t2").start();
    }
15:57:00.219 c.DataContainerStamped [t1] - optimistic read locking...256
15:57:00.717 c.DataContainerStamped [t2] - write lock 384
15:57:01.225 c.DataContainerStamped [t1] - updating to read lock... 256
15:57:02.719 c.DataContainerStamped [t2] - write unlock 384
15:57:02.719 c.DataContainerStamped [t1] - read lock 513
15:57:03.719 c.DataContainerStamped [t1] - read finish...513, data:1000
15:57:03.719 c.DataContainerStamped [t1] - read unlock 513 
~~~

> 注意
> StampedLock 不支持条件变量
> StampedLock 不支持可重入

## LockSupport

locksupport定义了一组公共的静态方法，这些方法用来对线程进行操作。

![1621223544068](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/115226-481486.png)

## Semaphore

 **信号量，用来限制能同时访问共享资源的线程上限**，多个共享资源。

**源码**

~~~java
public class Semaphore implements java.io.Serializable{}
~~~

**案例**

~~~ java
public class SemaphoreTest {
    public static void main(String[] args) {
//        创建一个semaphore对象，可以对访问资源的线程个数进行限制
        Semaphore semaphore = new Semaphore(3);//限制访问资源的线程有3个
//        启动10个线程
        for (int i = 0; i <10 ; i++) {
            new Thread(()->{
//                首先获取许可，许可数会-1
                try {
                    semaphore.acquire();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                try {
                    System.out.println("starting...");
                    Thread.sleep(1);
                    System.out.println("end.....");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
//                    释放锁，许可会+1
                    semaphore.release();
                }
            }).start();
        }
    }
}
~~~

### semaphore的原理

 #### 加锁原理

Semaphore 有点像一个停车场，permits 就好像停车位数量，当线程获得了 permits 就像是获得了停车位，然后停车场显示空余车位减一

刚开始，permits（state）为 3，这时 5 个线程来获取资源

~~~ java
//构造方法
public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }
//同步器类，调用的是父类的构造方法
NonfairSync(int permits) {
            super(permits);
        }
//父类的构造方法
Sync(int permits) {
            setState(permits);
        }
//可以看到，其实就是把3赋值给aqs的state变量
protected final void setState(int newState) {
        state = newState;
    }

~~~

![1610183072614](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/170437-561568.png)

假设其中 Thread-1，Thread-2，Thread-4 cas 竞争成功，而 Thread-0 和 Thread-3 竞争失败，进入 AQS 队列
park 阻塞

~~~ java
//获取锁的方法
public void acquire(int permits) throws InterruptedException {
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireSharedInterruptibly(permits);
    }
//调用的是同步器的方法
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
  //这里是尝试获取锁，返回数小于0，说明获取锁失败
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }
final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
              //剩余的资源数量
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                  //如果剩余资源个数小于0，那么就不会执行或运算后面的运算，会直接返回剩余的资源数量
                  //也就是进入doAcquireSharedInterruptibly(arg);方法
                    return remaining;
            }
        }
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
  //创建一个节点
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
              //找到头结点
                final Node p = node.predecessor();
                if (p == head) {
                  //如果前驱是头结点，那么此时前驱节点的后继才有可能去竞争锁
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
              //如果上面资源少于0,那么线程会在这里阻塞
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
~~~

![1610183118418](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1610183118418.png)

#### 释放锁

~~~ java
 //调用释放锁的方法
public void release() {
        sync.releaseShared(1);
    }
public final boolean releaseShared(int arg) {
  //尝试释放锁，如果成功，就返回true
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }

 private void doReleaseShared() {
     
        for (;;) {
          //获取头结点
            Node h = head;
            if (h != null && h != tail) {
              //获取头结点的状态
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {//SIGNAL=-1
                  //使用cas设置状态为0
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                  //下面方法会唤醒后继节点
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }

~~~

这时 Thread-4 释放了 permits，状态如下

![1610183149393](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/170552-854594.png)

接下来 Thread-0 竞争成功，permits 再次设置为 0，设置自己为 head 节点，断开原来的 head 节点，unpark 接
下来的 Thread-3 节点，但由于 permits 是 0，因此 Thread-3 在尝试不成功后再次进入 park 状态

![1610183190149](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/170736-801231.png)

### Acquire原理

~~~ java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = -2694183684443567898L;
    NonfairSync(int permits) {
        // permits 即 state
        super(permits);
    }

    // Semaphore 方法, 方便阅读, 放在此处
    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }

    // 尝试获得共享锁
    protected int tryAcquireShared(int acquires) {
        return nonfairTryAcquireShared(acquires);
    }

    // Sync 继承过来的方法, 方便阅读, 放在此处
    final int nonfairTryAcquireShared(int acquires) {
        for (;;) {
            int available = getState();
            int remaining = available - acquires;
            if (
                // 如果许可已经用完, 返回负数, 表示获取失败, 进入 doAcquireSharedInterruptibly
                    remaining < 0 ||
                            // 如果 cas 重试成功, 返回正数, 表示获取成功
                            compareAndSetState(available, remaining)
 ) {
                return remaining;
            }
        }
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    // 再次尝试获取许可
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        // 成功后本线程出队（AQS）, 所在 Node设置为 head
                        // 如果 head.waitStatus == Node.SIGNAL ==> 0 成功, 下一个节点 unpark
                        // 如果 head.waitStatus == 0 ==> Node.PROPAGATE
                        // r 表示可用资源数, 为 0 则不会继续传播
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                // 不成功, 设置上一个节点 waitStatus = Node.SIGNAL, 下轮进入 park 阻塞
                if (shouldParkAfterFailedAcquire(p, node) &&
                        parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

    // Semaphore 方法, 方便阅读, 放在此处
    public void release() {
        sync.releaseShared(1);
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }

    // Sync 继承过来的方法, 方便阅读, 放在此处
    protected final boolean tryReleaseShared(int releases) {
        for (;;) {
            int current = getState();
            int next = current + releases;
            if (next < current) // overflow
                throw new Error("Maximum permit count exceeded");
            if (compareAndSetState(current, next))
                return true;
        }
    }
}
~~~

## CountdownLatch

用来进行线程同步协作，等待所有线程完成倒计时。
其中构造参数用来初始化等待计数值，await() 用来等待计数归零，countDown() 用来让计数减一

**源码**

~~~ java
public class CountDownLatch {}
~~~

**案例**

~~~ java
public class Test51 {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(3);

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"  begin...");
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            latch.countDown();
            System.out.println(Thread.currentThread().getName()+latch.getCount());
        }).start();

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"  begin...");
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            latch.countDown();
            System.out.println(Thread.currentThread().getName()+latch.getCount());
        }).start();

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"  begin...");
            try {
                Thread.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            latch.countDown();
            System.out.println(Thread.currentThread().getName()+latch.getCount());
        }).start();

        System.out.println(Thread.currentThread().getName()+"  waiting...");
        latch.await();
        System.out.println(Thread.currentThread().getName()+"  wait end...");

    }
}

~~~

### 配合线程池

~~~ java
public class Test52 {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(4);
        CountDownLatch latch = new CountDownLatch(3);

        executorService.submit(() -> {
            System.out.println(Thread.currentThread().getName()+"  begin...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            latch.countDown();
            System.out.println(Thread.currentThread().getName()+latch.getCount());
        });
        executorService.submit(() -> {
            System.out.println(Thread.currentThread().getName()+"  begin...");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            latch.countDown();
            System.out.println(Thread.currentThread().getName()+latch.getCount());
        });
        executorService.submit(() -> {
            System.out.println(Thread.currentThread().getName()+"  begin...");
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            latch.countDown();
            System.out.println(Thread.currentThread().getName()+latch.getCount());
        });
        executorService.submit(() -> {
//            最后一个任务等待前面的任务完成
            try {
                System.out.println(Thread.currentThread().getName()+"  waiting...");
                latch.await();
                System.out.println(Thread.currentThread().getName()+"  wait end...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
} 
~~~

### 模拟游戏加载

~~~ java
public class Test53 {

    public static void main(String[] args) throws InterruptedException {

        ExecutorService executorService = Executors.newFixedThreadPool(10);
        CountDownLatch countDownLatch = new CountDownLatch(10);
        String []all = new String[10];
//        添加随机睡眠时间
        Random random = new Random();

        for (int j = 0; j <10; j++) {
            final int num=j;
            executorService.submit(()->{
                for (int i = 0; i <=100; i++) {
                    try {
                        Thread.sleep(random.nextInt(100));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
//                    lambda只能引用局部的常量，不可以引用局部的变量
                    all[num]=i+"%";
//            "\r"表示覆盖掉上一次的结果
                    System.out.print("\r"+Arrays.toString(all));

                }
                countDownLatch.countDown();
            });
        }

//        等待所有的玩家都结束，就开始启动游戏
        countDownLatch.await();
        System.out.println("\n游戏加载结束");

        executorService.shutdown();
    }
}
~~~

## CyclicBarrier

循环栅栏，用来进行线程协作，等待线程满足某个计数。构造时设置『计数个数』，每个线程执
行到某个需要“同步”的时刻调用 await() 方法进行等待，当等待的线程数满足『计数个数』时，继续执行

**案例**

~~~ java
public class Test54 {

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        CountDownLatch countDownLatch = new CountDownLatch(2);

        for (int i = 0; i <3; i++) {
            executorService.submit(()->{
                System.out.println(Thread.currentThread().getName()+"  start");
                try {
                    Thread.sleep(1000);
                    countDownLatch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            },"t1");
            executorService.submit(()->{
                System.out.println(Thread.currentThread().getName()+"  start");
                try {
                    Thread.sleep(1000);
                    countDownLatch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            },"t2");

//            主线程在这里等待
            countDownLatch.await();
        }

        executorService.shutdown();
    }
}

~~~

- 可以看到使用countDownLatch可以完成上面的 功能，多次循环定时任务。但是也存在问题，就是每循环一次，countDownLatch对象也会被创建一次。并且countDownLatch对象也不可以被重置。

**下面使用CyclicBarrier进行改进**

最大的好处是可以重新设置值。

~~~ java
public class Test55 {

    public static void main(String[] args) {


        ExecutorService executorService = Executors.newFixedThreadPool(5);
//        第二个参数是定时任务
        CyclicBarrier cyclicBarrier = new CyclicBarrier(2,()->{
            System.out.println("task is over");
        });

        for (int i = 0; i <3; i++) {
            executorService.submit(()->{
                System.out.println(Thread.currentThread().getName()+" starting...");
                try {
                    Thread.sleep(1000);
//                下面方法每一次都会-1
                    cyclicBarrier.await();
                    System.out.println(Thread.currentThread().getName()+" ending...");
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
            executorService.submit(()->{
                System.out.println(Thread.currentThread().getName()+" starting...");
                try {
                    Thread.sleep(3000);
//                下面方法每一次都会-1
                    cyclicBarrier.await();
                    System.out.println(Thread.currentThread().getName()+" ending...");
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
        }
    }
}
//上面设置的线程个数和任务的个数不一致，运行下面结果发现第一次运行启动很多个任务，如果这样的话，每一个任务运行时间是不一样的，可能多个运行时间短的线程先把计数器减为0，
pool-1-thread-1 starting...
pool-1-thread-2 starting...
pool-1-thread-3 starting...
pool-1-thread-4 starting...
pool-1-thread-5 starting...
task is over
pool-1-thread-3 ending...
pool-1-thread-5 ending...
pool-1-thread-3 starting...
task is over
pool-1-thread-4 ending...
pool-1-thread-1 ending...
task is over
pool-1-thread-3 ending...
pool-1-thread-2 ending...
//如果线程个数和任务个数保持一致
pool-1-thread-1 starting...
pool-1-thread-2 starting...
task is over
pool-1-thread-2 ending...
pool-1-thread-1 ending...
pool-1-thread-2 starting...
pool-1-thread-1 starting...
task is over
pool-1-thread-1 ending...
pool-1-thread-2 ending...
pool-1-thread-2 starting...
pool-1-thread-1 starting...
task is over
pool-1-thread-2 ending...
pool-1-thread-1 ending...

~~~

> 线程的数量和任务的数量最好保持一致。

## 线程安全集合类

### 分类

![1610236843410](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/10/080045-994042.png)

HashTable是map的实现，Vector是List的实现。

线程安全集合类可以分为三大类：

- 遗留的线程安全集合如 Hashtable ， Vector

- 使用 Collections 装饰的线程安全集合，如：

  - Collections.synchronizedCollection
  - Collections.synchronizedList
  - Collections.synchronizedMap
  - Collections.synchronizedSet
  - Collections.synchronizedNavigableMap
  - Collections.synchronizedNavigableSet
  - Collections.synchronizedSortedMap
  - Collections.synchronizedSortedSet

- java.util.concurrent.*

  重点介绍 java.util.concurrent.* 下的线程安全集合类，可以发现它们有规律，里面包含三类关键词：
  Blocking、CopyOnWrite、Concurrent

  - Blocking 大部分实现基于锁，并提供用来阻塞的方法
  - CopyOnWrite 之类容器修改开销相对较重
  - Concurrent 类型的容器
    - 内部很多操作使用 cas 优化，一般可以提供较高吞吐量
    - 弱一致性
      - 遍历时弱一致性，例如，当利用迭代器遍历时，如果容器发生修改，迭代器仍然可以继续进行遍
        历，这时内容是旧的
      - 求大小弱一致性，size 操作未必是 100% 准确
      - 读取弱一致性

> 遍历时如果发生了修改，对于非安全容器来讲，使用 fail-fast 机制也就是让遍历立刻失败，抛出
> ConcurrentModificationException，不再继续遍历

