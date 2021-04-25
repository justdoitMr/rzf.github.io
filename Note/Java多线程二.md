共享模型之无锁(乐观锁)

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

### 解决思路-加锁

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

### 使用无锁的方式解决

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

- 获取共享变量时，为了保证该变量的可见性，需要使用` volatile `修饰。
- 它可以用来修饰**成员变量和静态成员变量**，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作` volatile` 变量都是直接操作主存。即一个线程对 `volatile` 变量的修改，对另一个线程可见。

> 注意
> volatile 仅仅保证了共享变量的可见性，让其它线程能够看到最新值，但不能解决指令交错问题（不能保证原子性）

`CAS` 必须借助 `volatile` 才能读取到共享变量的最新值来实现【比较并交换】的效果，也就是`cas`操作要对其他线程对共享变量的修改可见，如果共享变量没有使用`volatile`进行修饰，那么获取的可能就不是最新的值。

~~~ java
private volatile int value;
~~~

#### 为什么无锁效率高

- 无锁情况下，即使重试失败，线程始终在高速运行，没有停歇，而 synchronized 会让线程在没有获得锁的时候，发生上下文切换，进入阻塞。
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

### 原子整数

`J.U.C` 并发包提供了：（是工具包，所有的包装都是保证操作是原子的。底层都是送`cas`算法实现）

- AtomicBoolean
- AtomicInteger
- AtomicLong

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
// 其中函数中的操作能保证原子，但函数需要无副作用
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
     * IntUnaryOperator:操作参数
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

### 原子引用

为什么需要原子引用类型？

- AtomicReference
- AtomicMarkableReference
- AtomicStampedReference

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

#### ABA问题

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

- 主线程仅能判断出共享变量的值与最初值 A 是否相同，不能感知到这种从 A 改为 B 又 改回 A 的情况，如果主线程希望：只要有其它线程【动过了】共享变量，那么自己的 cas 就算失败，这时，仅比较值是不够的，需要再加一个版本号

#### **AtomicStampedReference解决**

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

#### **AtomicMarkableReference**

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

### 原子数组

用来保护数组里面的元素

- AtomicIntegerArray
- AtomicLongArray
- AtomicReferenceArray



### 字段更新器

用来保护某一个对象中的属性或者说是成员变量。

- AtomicReferenceFieldUpdater // 域 字段
- AtomicIntegerFieldUpdater
- AtomicLongFieldUpdater

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

**代码说明**

~~~ java
 private static <T> void demo(Supplier<T> adderSupplier, Consumer<T> action) {
        T adder = adderSupplier.get();
        long start = System.nanoTime();
        List<Thread> ts = new ArrayList<>();
        // 4 个线程，每人累加 50 万
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
//比较操作
for (int i = 0; i < 5; i++) {
 	demo(() -> new LongAdder(), adder -> adder.increment());
}
for (int i = 0; i < 5; i++) {
 	demo(() -> new AtomicLong(), adder -> adder.getAndIncrement());
}

~~~

- 性能提升的原因很简单，就是在有竞争时，设置多个累加单元，`Therad-0` 累加` Cell[0]`，而 `Thread-1 `累加`Cell[1]`... 最后将结果汇总。这样它们在累加时操作的不同的 `Cell `(共享)变量，因此减少了` CAS `重试失败，从而提高性能。

### LongAdder源码解读

LongAdder 类有几个关键域

~~~ java
// 累加单元数组, 懒惰初始化
transient volatile Cell[] cells;
// 基础值, 如果没有竞争, 则用 cas 累加这个域
transient volatile long base;
// 在 cells 创建或扩容时, 置为 1, 表示加锁
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





### UnSafe

#### 概述

Unsafe 对象提供了非常底层的，操作内存、线程的方法，Unsafe 对象不能直接调用，只能通过反射获得

**代码说明**

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

- 属性用 final 修饰保证了该属性是只读的，不能修改
- 类用 final 修饰保证了该类中的方法不能被覆盖，防止子类无意间破坏不可变性

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

定义 英文名称：Flyweight pattern. 当需要重用数量有限的同一类对象时

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

### 自定义数据库连接池

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

- 如果一个变量被声明为final变量，那么会在赋值之后添加一个写屏障，写屏障可以保证两个问题，写屏障之前的语句不会被重排序到写屏障之后，写屏障之前的所有赋值或者修改操作，在写屏障之后都会被同步到主存中，也就是对其他线程可见。
- 给变量添加final关键字就保证了其他线程只能看到final当前的值，保证对其他的线程可见。

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
        System.out.println(TestFinal.A);
        System.out.println(TestFinal.B);

        System.out.println(new TestFinal().a);
        System.out.println(new TestFinal().b);
    }
}
~~~

对于final修饰的变量，如果数值较小，会赋值一份到其他方法的栈中，如果数值较大，会赋值一份到其他类的常量池中，相当于做的一种优化操作，如果不添加final，那么获取静态变量使用getstatic指令，相当于去共享内存中获取，比栈内存中的效率低。

## 线程池

### 自定义线程池

~~~ java

~~~





### ThreadPoolExecutor线程池

**类图**

![1609476719311](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1609476719311.png)

- ExecutorService线程池最基本的接口，包括提交任务，关闭线程池等
- ScheduledExecutorService:扩展接口，在基础接口放入功能上增加任务调度的功能。
- ThreadPoolExecutor:线程池的最基本实现
- SchedualThreadPoolExecutor:带有任务调度的最基础实现

#### 线程池的状态

ThreadPoolExecutor 使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量

![1609477104864](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1609477104864.png)

从数字上比较，TERMINATED > TIDYING > STOP > SHUTDOWN > RUNNING（最高位代表符号位）

这些信息存储在一个原子变量 ctl 中，目的是将线程池状态与线程个数合二为一，这样就可以用一次 cas 原子操作进行赋值，如果分开的话就要使用两次cas操作。

~~~ java
// c 为旧值， ctlOf 返回结果为新值
ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))));
ctlOf(targetState, workerCountOf(c))//表示合并线程状态和线程数量为一个值
// rs 为高 3 位代表线程池状态， wc 为低 29 位代表线程个数，ctl 是合并它们
private static int ctlOf(int rs, int wc) { return rs | wc; }
~~~

**构造方法**

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

救济线程是针对突发量大，线程池中线程数量不足的情况，如果线程池中线程不够，阻塞队列也蛮了，那么就会创建救济线程，救济线程最大的特点是有生存时间，当救济线程也不够的话，就会执行拒绝策略。

**工作方式**

![1609477930176](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202101/01/131212-741170.png)

- 线程池中刚开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务。

- 当线程数达到 corePoolSize 并没有线程空闲，这时再加入任务，新加的任务会被加入workQueue 队列排
  队，直到有空闲的线程。

- 如果队列选择了有界队列，那么任务超过了队列大小时，会创建 maximumPoolSize - corePoolSize 数目的线程来救急。（也就是救济线程），如果选择的队列是无界的，那么此时就没有什么救济线程。

- 如果线程到达 maximumPoolSize 仍然有新任务这时会执行拒绝策略。拒绝策略 jdk 提供了 4 种实现，其它
  著名框架也提供了实现

  - AbortPolicy 让调用者抛出 RejectedExecutionException 异常，这是默认策略
  - CallerRunsPolicy 让调用者运行任务
  - DiscardPolicy 放弃本次任务
  - DiscardOldestPolicy 放弃队列中最早的任务，本任务取而代之

  上面是jdk提供的实现，下面是框架的实现。

  - Dubbo 的实现，在抛出 RejectedExecutionException 异常之前会记录日志，并 dump 线程栈信息，方
    便定位问题
  - Netty 的实现，是创建一个新线程来执行任务
  - ActiveMQ 的实现，带超时等待（60s）尝试放入队列，类似我们之前自定义的拒绝策略
  - PinPoint 的实现，它使用了一个拒绝策略链，会逐一尝试策略链中每种拒绝策略

- 当高峰过去后，超过corePoolSize 的救急线程如果一段时间没有任务做，需要结束节省资源，这个时间由
  keepAliveTime 和 unit 来控制。

![1609478329567](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/thread/202101/01/131851-399649.png)

根据这个构造方法，JDK Executors 类中提供了众多工厂方法来创建各种用途的线程池，因为普通的构造方法提供的参数太多，所以jdk提供一个工具类来使用。

#### newFixedThreadPool（固定线程数）

~~~ java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,//核心线程数和最大线程数被设置为nThreads,表示救济线程数设置为0，救济线程=最大线程数-核心线程数，可以看到对返回的对象没有进行包装操作
                0L, TimeUnit.MILLISECONDS,
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

#### newCachedThreadPool（带缓冲功能）

~~~ java
 public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                60L, TimeUnit.SECONDS,
                new SynchronousQueue<Runnable>());
    }
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

#### newSingleThreadExecutor（单线程执行器）

~~~ java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService//对返回的对象进行包装操作，以后调用只能够调用接口中的方法
                (new ThreadPoolExecutor(1, 1,//没有救济线程
                        0L, TimeUnit.MILLISECONDS,
                        new LinkedBlockingQueue<Runnable>()));
    }
~~~

**使用场景：**

- 希望多个任务排队执行。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。任务执行完毕，这唯一的线程也不会被释放。

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
// 提交 tasks 中所有任务，带超时时间
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

### 异步模式之工作线程

#### 定义

让有限的工作线程（Worker Thread）来轮流异步处理无限多的任务。也可以将其归类为分工模式，它的典型实现就是线程池，也体现了经典设计模式中的享元模式。

**案例**

例如，海底捞的服务员（线程），轮流处理每位客人的点餐（任务），如果为每位客人都配一名专属的服务员，那么成本就太高了（对比另一种多线程设计模式：Thread-Per-Message）

**注意，不同任务类型应该使用不同的线程池，这样能够避免饥饿，并能提升效率**

例如，如果一个餐馆的工人既要招呼客人（任务类型A），又要到后厨做菜（任务类型B）显然效率不咋地，分成服务员（线程池A）与厨师（线程池B）更为合理，当然你能想到更细致的分工

#### 饥饿

固定大小线程池会有饥饿现象

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

#### 自定义线程池





### 任务调度线程池（延时执行任务）

在『任务调度线程池』功能加入之前，可以使用 java.util.Timer 来实现定时功能，Timer 的优点在于简单易用，但由于所有任务都是由同一个线程来调度，因此所有任务都是串行执行的，同一时间只能有一个任务在执行，前一个任务的延迟或异常都将会影响到之后的任务。

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
        // 但由于 timer 内只有一个线程来顺序执行队列中的任务，因此『任务1』的延时，影响了『任务2』的执行
        timer.schedule(task1, 1000);
        timer.schedule(task2, 1000);
    }
~~~

使用 ScheduledExecutorService 改写：

~~~ java
public class TestTimer {

    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newScheduledThreadPool(2);
//        第一个参数代表执行的任务，第二个是延迟的时间
        service.schedule(()->{
            try {
                Thread.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"  task 1");
        },1, TimeUnit.SECONDS);

        service.schedule(()->{
            System.out.println(Thread.currentThread().getName()+"  task 2");
        },1, TimeUnit.SECONDS);
    }
}
//两个线程几乎是同时执行的，也就是说不会相互影响，虽然第一个线程睡了2秒，但是并不会影响第二个线程的执行
//如果线程池的大小小于创建的线程个数，那么线程之间还是会串行执行
//即使第一个线程执行过程中出现异常，第二个任务还是可以正常执行，但是对于Timer就不行，第二个任务不能执行
~~~

**定时执行任务**

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

scheduleWithFixedDelay 例子：

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

**正确处理异常信息**

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

#### 使用

提交给 Fork/Join 线程池的任务需要继承 RecursiveTask（有返回值）或 RecursiveAction（没有返回值），例如下面定义了一个对 1~n 之间的整数求和的任务

~~~ java
public class TestForkJoin {
    public static void main(String[] args) {
//        无参构造，那么线程数等于cpu的核心数
        ForkJoinPool pool = new ForkJoinPool(4);

//        调用方法执行任务
        System.out.println(pool.invoke(new MyTask(5)));
    }
}


class MyTask extends RecursiveTask{

    private int n;

    public MyTask(int n) {
        this.n = n;
    }

    @Override
    protected Integer compute() {
//        把计算任务拆分开计算，类似于递归计算
       
        if(n == 1)
            return 1;
        //拆分任务
        MyTask myTask = new MyTask(n - 1);
        myTask.fork();//让一个线程去执行任务

        int result=n+(Integer) myTask.join();//获取结果

        return result;
    }
}
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

**改进**

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

## JUC

###  AQS 原理

#### 概述

全称是 AbstractQueuedSynchronizer，是**阻塞式锁**和相关的同步器工具的框架，也就是其他的同步器框架都是继承与此框架的，调用父类中的方法。（CAS是一种乐观锁)，基于模板的方法实现的。

**特点**

- 用 state （整数值）属性来表示资源的状态（分独占模式和共享模式），子类需要定义如何维护这个状态，控制如何获取锁和释放锁
  - getState - 获取 state 状态
  - setState - 设置 state 状态
  - compareAndSetState - cas 机制设置 state 状态（防止多个线程同时修改状态）
  - 独占模式是只有一个线程能够访问资源，而共享模式可以允许多个线程访问资源
- **提供了基于 FIFO 的等待队列，类似于 Monitor 的 EntryList**
- 条件变量来实现等待、唤醒机制，支持多个条件变量，类似于 Monitor 的 WaitSet

**如何使用**

~~~ java
//子类主要实现这样一些方法（默认抛出 UnsupportedOperationException）
tryAcquire
tryRelease
tryAcquireShared
tryReleaseShared
isHeldExclusively
~~~

**获取锁**

~~~java
// 如果获取锁失败
if (!tryAcquire(arg)) {
 // 入队, 可以选择阻塞当前线程 park unpark
}
//只会尝试一次获取锁，获取锁成功的话就可以修改状态标记
~~~

**释放锁**

~~~ java
// 如果释放锁成功
if (tryRelease(arg)) {//返回true表示释放锁成功
 // 让阻塞线程恢复运行
}
~~~

#### 实现不可重入锁

**自定义同步器**

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

**自定义锁**

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

## LOCK接口

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

> LOck接口的实现都是通过聚合一个同步器的子类来完成线程的访问控制

**Lock接口的实现子类**

![1614075272189](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/23/181434-121988.png)

### 队列同步器

1. 什么是队列同步器？

队列同步器（AbstractQueuedSynchronizer），是用来构建锁或者其他同步组件的基础框架，他使用一个int类型的成员变量表示同步状态，通过内置的fifo队列完成资源获取线程的排队工作，同步器的主要使用方式是继承，子类通过继承的方法实现抽象方法来管理同步状态。同步器既可以支持独占式的获取同步状态，也可以支持共享式的获取同步状态，同步器面向的是锁的实现者。

同步器的实现是基于模板设计模式

2. 队列同步器类图

![1614075954630](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/23/182557-47954.png)

**源码解读**

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

## ReentrantLock 原理(重入锁)

- 表示该锁支持一个线程对资源进行重复加锁，该锁还支持获取锁时的公平与非公平性



![1610151297656](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/081459-832258.png)

### 非公平锁的实现原理

**加锁的流程实现**

先从构造器开始看，默认为非公平锁实现

~~~ java
public ReentrantLock() {
 sync = new NonfairSync();
}
//NonfairSync 继承自 AQS
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
//也即是进入else语句块
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&//进行尝试加锁
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

![1610152307914](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/083150-51836.png)

当前线程进入 acquireQueued 逻辑

-  acquireQueued 会在一个死循环中不断尝试获得锁，失败后进入 park 阻塞
-  如果自己是紧邻着 head（排第二位），那么再次 tryAcquire 尝试获取锁，当然这时 state 仍为 1，失败
- . 进入 shouldParkAfterFailedAcquire 逻辑，将前驱 node，即 head 的 waitStatus 改为 -1，这次返回 false

**孕源码解读**

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

![1610152514504](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1610152514504.png)

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
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);//唤醒其后继节点
            return true;
        }
        return false;
    }
~~~

![1610153287637](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/084814-366420.png)

- 当前队列不为 null，并且 head 的 waitStatus = -1，进入 unparkSuccessor 流程
- 找到队列中离 head 最近的一个 Node（没取消的），unpark 恢复其运行，本例中即为 Thread-1
- 回到 Thread-1 的 acquireQueued 流程

![1610153399056](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1610153399056.png)

如果加锁成功（没有竞争），会设置

- exclusiveOwnerThread 为 Thread-1，state = 1
- head 指向刚刚 Thread-1 所在的 Node，该 Node 清空 Thread
- 原本的 head 因为从链表断开，而可被垃圾回收

如果这时候有其它线程来竞争（非公平的体现），例如这时有 Thread-4 来了

![1610153543523](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1610153543523.png)

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
                int c = getState();
                if (c == 0) {
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
                return false;
            }

            // Sync 继承过来的方法, 方便阅读, 放在此处
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
                // interrupted 会清除打断标记
                return Thread.interrupted();
            }

            final boolean acquireQueued(final Node node, int arg) {
                boolean failed = true;
                try {
                    boolean interrupted = false;
                    for (;;) {
                        final Node p = node.predecessor();
                        if (p == head && tryAcquire(arg)) {
                            setHead(node);
                            p.next = null;
                            failed = false;
                            // 还是需要获得锁后, 才能返回打断状态
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

![1610156397464](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1610156397464.png)

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

ReentrantLock是一个排它锁，同一时刻只允许一个线程访问，而读写锁同一时刻允许多个线程访问，但是在写线程访问的时候，所有的其他读线程和写线程全部被阻塞，读写锁维护了一对锁，读锁和写锁。

###  ReentrantReadWriteLock

~~~java
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable
~~~

**public class ReentrantReadWriteLock特性**

- 公平性选择
- 重进入，读线程获取读锁后，可以再次获取读锁，写线程获取写锁后，可以再次获取写锁，同时也可以获取读锁。
- 锁降级

当读操作远远高于写操作时，这时候使用 读写锁 让 读-读 可以并发，提高性能。 类似于数据库中的 select ...
from ... lock in share mode

提供一个 数据容器类 内部分别使用读锁保护数据的 read() 方法，写锁保护数据的 write() 方法

**多个线程读操作**

~~~ java
public class Test {
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

**注意事项**

- 读锁不支持条件变量
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
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    void processCachedData() {
        rwl.readLock().lock();
        if (!cacheValid) {
            // 获取写锁前必须释放读锁
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
                // 判断是否有其它线程已经获取了写锁、更新了缓存, 避免重复更新
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

~~~ java
t1 w.lock，t2 r.lock
~~~

t1 成功上锁，流程与 ReentrantLock 加锁相比没有特殊之处，不同是写锁状态占了 state 的低 16 位，而读锁
使用的是 state 的高 16 位

![1610163372436](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1610163372436.png)

















###  StampedLock(读锁)

该类自 JDK 8 加入，是为了进一步优化读性能，它的特点是在使用读锁、写锁时都必须配合【戳】使用
**加解读锁**

~~~ java
long stamp = lock.readLock();
lock.unlockRead(stamp);
~~~

**加解写锁**

~~~ java
long stamp = lock.writeLock();
lock.unlockWrite(stamp);
~~~

乐观读，StampedLock 支持 tryOptimisticRead() 方法（乐观读），读取完毕后需要做一次 戳校验 如果校验通
过，表示这期间确实没有写操作，数据可以安全使用，如果校验没通过，需要重新获取读锁，保证数据安全。

~~~ java
long stamp = lock.tryOptimisticRead();//底层实现不会真的加锁
// 验戳
if(!lock.validate(stamp)){
 // 锁升级
}
~~~

提供一个 数据容器类 内部分别使用读锁保护数据的 read() 方法，写锁保护数据的 write() 方法

~~~ java
class DataContainerStamped {
    private int data;
    private final StampedLock lock = new StampedLock();
    public DataContainerStamped(int data) {
        this.data = data;
    }
    public int read(int readTime) {
        long stamp = lock.tryOptimisticRead();
        log.debug("optimistic read locking...{}", stamp);
        sleep(readTime);
        if (lock.validate(stamp)) {
            log.debug("read finish...{}, data:{}", stamp, data);
            return data;
        }
        // 锁升级 - 读锁
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
//从输出结果看，实际上是没有加锁的
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

locksupport定义了一组公共的静态方法，这些方法用来对线程进行操作。、

## Condition接口

![1614080480842](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614080480842.png)

condition接口提供了一组监视器方法，可以和Lock配合实现等待通知模式，condition对象是由Lock对象创建出来的，也就是说condition对象依赖于kock对象。

object也是一个对象监视器，一个对象拥有一个同步队列和等待队列

并发包中的lock拥有一个同步队列和多个等待队列

condition的实现是同步器的内部类

下面我们可以看到，实现在同步器类中，因此每一个conditoin对象都可以访问同步器提供的方法，相当于condition对象都拥有所属同步器的引用。

![1614081190784](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/23/195314-244438.png)

## Semaphore

 信号量，用来限制能同时访问共享资源的线程上限

~~~ java
public class SemaphoreTest {
    public static void main(String[] args) {
//        创建一个semaphore对象，可以对访问资源的线程个数进行限制
        Semaphore semaphore = new Semaphore(3);//限制访问资源的线程有3个
//        启动10个线程
        for (int i = 0; i <10 ; i++) {
            new Thread(()->{
//                首先获取许可
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
//                    释放锁
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

![1610183072614](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/170437-561568.png)

假设其中 Thread-1，Thread-2，Thread-4 cas 竞争成功，而 Thread-0 和 Thread-3 竞争失败，进入 AQS 队列
park 阻塞

![1610183118418](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1610183118418.png)

这时 Thread-4 释放了 permits，状态如下

![1610183149393](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/170552-854594.png)

接下来 Thread-0 竞争成功，permits 再次设置为 0，设置自己为 head 节点，断开原来的 head 节点，unpark 接
下来的 Thread-3 节点，但由于 permits 是 0，因此 Thread-3 在尝试不成功后再次进入 park 状态

![1610183190149](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/09/170736-801231.png)

### 源码解读

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

### ConcurrentHashMap原理

HashMap在jdk7中插入节点使用的是头插法，而在jdk8中使用的是尾插法插入元素。

#### hashMap7中多线程访问并发死链问题

在jdk8中没有并发死链问题。

