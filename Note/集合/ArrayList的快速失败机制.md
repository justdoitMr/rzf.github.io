## ArrayList的快速失败机制

### fail-fast简介

概念：

在使用迭代器对集合对象进行遍历的时候，如果 A 线程正在对集合进行遍历，此时 B 线程对集合进行修改（**增加、删除、修改**），或者 **A 线程在遍历过程中对集合进行修改**，都会导致 A 线程抛出 ConcurrentModificationException 异常。

快速失败是指某个线程在迭代集合类的时候，不允许其他线程修改该集合类的内容，这样迭代器迭代出来的结果就会不准确。

比如用iterator迭代collection的时候，iterator就是另外起的一个线程，它去迭代collection，如果此时用collection.remove(obj)这个方法修改了collection里面的内容的时候，就会出现ConcurrentModificationException异常,这时候该迭代器就快速失败。

### 快速失败相对于安全失败：

安全失败概念：

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

fail-fast 机制是java集合(Collection)中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。

例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

在详细介绍fail-fast机制的原理之前，先通过一个示例来认识fail-fast。

> 对于ArrayList出现快速失败的前提是使用了iterator进行遍历。

### 使用快速失败机制的好处

这样做的好处是：对于非并发集合，在用迭代器对其迭代时，若有其他线程修改了增减了集合中的内容，这个迭代会马上感知到，并且立即抛出 ConcurrentModificationException 异常，而不是在迭代完成后或者某些并发的场景才报错。

###  fail-fast示例

单线程，在iterator遍历的同时，插入新的参数。会导致快速失败

~~~java
 * Created by zongx on 2019/11/26.
 */
public class FastFailDemo {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(i);
        }
 
        for (Integer integer : list) {
            list.add(integer);
            System.out.println(integer);
        }
 
    }
}
~~~

单线程，使用randomaccess遍历List，即使在边读边写也不会出现快速失败

```java
package collectionFoudation.arrayList;
 
import java.util.ArrayList;
import java.util.List;
 
/**
 * Created by zongx on 2019/11/26.
 */
public class FastFailDemo {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(i);
        }
        for(int i =0 ; i< list.size(); i++) {
            list.add(i);
            System.out.println(i);
        }
 
    }
}
```
多线程，在一个线程读时，另一个线程写入list，读线程会快速失败

```java
package collectionFoudation.arrayList;
 
import java.util.ArrayList;
import java.util.List;

public class FastFailDemo {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(i);
        }
        new MyThread1(list).start();
        new MyThread2(list).start();
    }
}
 
class MyThread1 extends Thread {
    private List<Integer> list;
 
    public MyThread1( List<Integer> list) {
        this.list = list;
    }
 
    @Override
    public void run() {
        for (Integer integer : list) {
            System.out.println("MyThread1" + list.size());
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
 
class MyThread2 extends Thread {
    private List<Integer> list;
 
    public MyThread2( List<Integer> list) {
        this.list = list;
    }
 
    @Override
    public void run() {
 
        for (int i = 0; i < 10; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list.add(i);
            System.out.println("MyThread2 " + list.size());
        }
    }
}
```
当某一个线程遍历list的过程中，list的内容被另外一个线程所改变了；就会抛出ConcurrentModificationException异常，产生fail-fast事件

### fail-fast解决办法

fail-fast机制，是一种错误检测机制。它只能被用来检测错误，因为JDK并不保证fail-fast机制一定会发生。若在多线程环境下使用fail-fast机制的集合，建议使用“java.util.concurrent包下的类”去取代“java.util包下的类”。所以，本例中只需要将ArrayList替换成java.util.concurrent包下对应的类即可。
即，将代码

~~~java
List<Integer> list = new ArrayList<>();

替换为

List<Integer> list = new CopyOnWriteArrayList<>();
~~~

则可以解决该办法。

CopyOnWriteArrayList与ArrayList不同：

1.  和ArrayList继承于AbstractList不同，CopyOnWriteArrayList没有继承于AbstractList，它仅仅只是实现了List接口。
2. ArrayList的iterator()函数返回的Iterator是在AbstractList中实现的；而CopyOnWriteArrayList是自己实现Iterator。
3. ArrayList的Iterator实现类中调用next()时，会“调用checkForComodification()比较‘expectedModCount’和‘modCount’的大小”；但是，CopyOnWriteArrayList的Iterator实现类中，没有所谓的checkForComodification()，更不会抛出ConcurrentModificationException异常！

### fail-fast原理

产生fail-fast事件，是通过抛出ConcurrentModificationException异常来触发的。
那么，ArrayList是如何抛出ConcurrentModificationException异常的呢?

我们知道，ConcurrentModificationException是在操作Iterator时抛出的异常。我们先看看Iterator的源码。ArrayList的Iterator是在父类AbstractList.java中实现的。代码如下

~~~java
package java.util;
 
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
 
    ...
 
    // AbstractList中唯一的属性
    // 用来记录List修改的次数：每修改一次(添加/删除等操作)，将modCount+1
    protected transient int modCount = 0;
 
    // 返回List对应迭代器。实际上，是返回Itr对象。
    public Iterator<E> iterator() {
        return new Itr();
    }
 
    // Itr是Iterator(迭代器)的实现类
    private class Itr implements Iterator<E> {
        int cursor = 0;
 
        int lastRet = -1;
 
        // 修改数的记录值。
        // 每次新建Itr()对象时，都会保存新建该对象时对应的modCount；
        // 以后每次遍历List中的元素的时候，都会比较expectedModCount和modCount是否相等；
        // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
        int expectedModCount = modCount;
 
        public boolean hasNext() {
            return cursor != size();
        }
 
        public E next() {
            // 获取下一个元素之前，都会判断“新建Itr对象时保存的modCount”和“当前的modCount”是否相等；
            // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
            checkForComodification();
            try {
                E next = get(cursor);
                lastRet = cursor++;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }
 
        public void remove() {
            if (lastRet == -1)
                throw new IllegalStateException();
            checkForComodification();
 
            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }
 
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
 
    ...
}
~~~

从中，我们可以发现在调用 next() 和 remove()时，都会执行 checkForComodification()。若 “modCount 不等于 expectedModCount”，则抛出ConcurrentModificationException异常，产生fail-fast事件。

要搞明白 fail-fast机制，我们就要需要理解什么时候“modCount 不等于 expectedModCount”！
从Itr类中，我们知道 expectedModCount 在创建Itr对象时，被赋值为 modCount。

通过Itr，我们知道：expectedModCount不可能被修改为不等于 modCount。所以，需要考证的就是modCount何时会被修改。

接下来，我们查看ArrayList的源码，来看看modCount是如何被修改的。

~~~java
package java.util;
 
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
 
    ...
 
    // list中容量变化时，对应的同步函数
    public void ensureCapacity(int minCapacity) {
        modCount++;
        int oldCapacity = elementData.length;
        if (minCapacity > oldCapacity) {
            Object oldData[] = elementData;
            int newCapacity = (oldCapacity * 3)/2 + 1;
            if (newCapacity < minCapacity)
                newCapacity = minCapacity;
            // minCapacity is usually close to size, so this is a win:
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
    }
 
 
    // 添加元素到队列最后
    public boolean add(E e) {
        // 修改modCount
        ensureCapacity(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
 
 
    // 添加元素到指定的位置
    public void add(int index, E element) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(
            "Index: "+index+", Size: "+size);
 
        // 修改modCount
        ensureCapacity(size+1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
             size - index);
        elementData[index] = element;
        size++;
    }
 
    // 添加集合
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        // 修改modCount
        ensureCapacity(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
   
 
    // 删除指定位置的元素 
    public E remove(int index) {
        RangeCheck(index);
 
        // 修改modCount
        modCount++;
        E oldValue = (E) elementData[index];
 
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--size] = null; // Let gc do its work
 
        return oldValue;
    }
 
 
    // 快速删除指定位置的元素 
    private void fastRemove(int index) {
 
        // 修改modCount
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // Let gc do its work
    }
 
    // 清空集合
    public void clear() {
        // 修改modCount
        modCount++;
 
        // Let gc do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;
 
        size = 0;
    }
 
    ...
}
~~~

从中，我们发现：无论是add()、remove()，还是clear()，只要涉及到修改集合中的元素个数时，都会改变modCount的值。

接下来，我们再系统的梳理一下fail-fast是怎么产生的。步骤如下：

1. 新建了一个ArrayList，名称为arrayList。
2. 向arrayList中添加内容。
3. 新建一个“线程a”，并在“线程a”中通过Iterator反复的读取arrayList的值。
4. 新建一个“线程b”，在“线程b”中删除arrayList中的一个“节点A”。
5. 这时，就会产生有趣的事件了。

在某一时刻，“线程a”创建了arrayList的Iterator。此时“节点A”仍然存在于arrayList中，创建arrayList时，expectedModCount = modCount(假设它们此时的值为N)。

在“线程a”在遍历arrayList过程中的某一时刻，“线程b”执行了，并且“线程b”删除了arrayList中的“节点A”。“线程b”执行remove()进行删除操作时，在remove()中执行了“modCount++”，此时modCount变成了N+1！

“线程a”接着遍历，当它执行到next()函数时，调用checkForComodification()比较“expectedModCount”和“modCount”的大小；而“expectedModCount=N”，“modCount=N+1”,这样，便抛出ConcurrentModificationException异常，产生fail-fast事件。

至此，我们就完全了解了fail-fast是如何产生的！

即，当多个线程对同一个集合进行操作的时候，某线程访问集合的过程中，该集合的内容被其他线程所改变(即其它线程通过add、remove、clear等方法，改变了modCount的值)；这时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

### 快速失败机制不能得到保证

- 在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的 remove 方法，其他任何时间任何方式的修改，迭代器都将抛出ConcurrentModificationException。
- 迭代器的快速失败行为不能得到保证，一般来说，存在非同步的并发修改时，不可能作出任何的保证。快速失败迭代器只是尽最大努力抛出ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是错误的，正确做法是：迭代器的快速失败行为应该仅用于检测程序错误。

### 为什么快速失败机制不能得到保证

#### 案例代码

~~~java
public static void main(String[] args) throws InterruptedException {
        List<String> list = new ArrayList<String>();　　　　 list.add（"第零个"）;
        list.add("第一个");
        list.add("第二个");
        list.add("第三个");
        list.add("第四个");
        
        for (String str : list) {
            if (str.equals("第三个")) {
                System.out.println("删除：" + str);
                list.remove(str);
            }
        }
        System.out.println(list);
    }
~~~

知道快速失败机制的可能都会说，不能在foreach循环里用集合直接删除，应该使用iterator的remove()方法，否则会报错：java.util.ConcurrentModificationException

但是这个代码的真实输出结果却是：

![1641367329281](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/152248-922315.png)

java的foreach 和 快速失败机制还是先简单介绍一下：

**foreach过程：**

Java在通过foreach遍历集合列表时，会先为列表创建对应的迭代器，并通过调用迭代器的hasNext()函数判断是否含下一个元素，若有则调用iterator.next()获取继续遍历，没有则结束遍历。

**快速失败机制：**

因为非线程安全，迭代器的next()方法调用时会判断modCount==expectedModCount，否则抛出ConcurrentModIficationException。modCount只要元素数量变化（add，remove）就+1，而集合的add和remove方法却不会更新expectedModCount，只有迭代器remove会重置expectedModCount=modCount，并将cursor往前一位。所以在使用迭代器循环的时候，只能使用迭代器的修改。

#### 分析

所以由上面的foreach介绍我们可以知道上面的foreach循环代码可以写成如下形式：

~~~java
for (Iterator iterator = list.iterator(); iterator.hasNext();) {
            String str = (String) iterator.next();
            if (str.equals("第三个")) {
                System.out.println("删除：" + str);
                list.remove(str);
            }
        }
~~~

重点就在于 iterator.next() 这里，我们看看next()的源码：【此处的迭代器是ArrayList的私有类，实现了迭代器接口Iterator，重写了各种方法】

~~~java
public E next() {
            checkForComodification();
            try {
                int i = cursor;
                E next = get(i);
                lastRet = i;
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }
~~~

注意到第7行！也就是说，**cursor最开始是 i，调用next()后就将第 i 个元素返回，然后变成下一个元素的下标了**，所以在遍历到倒数第二个元素的时候cursor已经为最后一个元素的下标（size-1）了，

注意了！在调用集合或者.remove(int)的方法时，并不会对cursor进行改变，【具体操作：将元素删除后，调用System.arraycopy让后面的的元素往前移动一位，并将最后一个元素位释放】，而本程序中此时的size变成了原来的size-1=4，而cursor还是原来的size-1=4，二者相等！，再看看判定hasNext()：

~~~java
 public boolean hasNext() {
            return cursor != size();
        }
~~~

此时cursor==size()==4，程序以为此时已经遍历完毕，所以根本不会进入循环中，也就是说根本不会进入到next()方法里，也就不会有checkForComodification() 的判断。

#### 验证

我们在程序中foreach循环的第一句获取str后面加入一个打印，  System.out.println(str); ，

这样每次进入foreach循环就会打印1，其他不变，我们再来运行一次，结果如下：

![1641367722952](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/152844-96658.png)

显然，第四个元素没有被遍历到，分析正确，那假如使用iterator.remove()呢？

那我们再来看看iterator.remove()的源码，【此处的iterator是ArrayList的私有类，实现了迭代器接口Iterator，重写了各种方法】

~~~java
public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }
~~~

看第7行，内部其实也是调用的所属list的remove(int)方法，但是不同的地方要注意了：

第9行：将cursor--，也就是说在删除当前元素后，他又把移动后的指针放回了当前元素下标，所以继续循环不会跳过当前元素位的新值；

第11行：expectedModCount = modCount; 更新expectedModCount，使二者相同，在继续循环中不会被checkForComodification()检测出报错。

#### 结论

1. 每次foreach循环开始时next()方法会使cursor变为**下一个元素下标**；

2. ArrayList的remove()方法执行完后，下一个元素移动到被删除元素位置上，由1可知，cursor此时指向原来被删除元素的下一个的下一个元素所在位置，此时继续foreach循环，被删除元素的下一个元素不会被遍历到；因为被删除元素的下一个元素现在放在被删除元素的位置上。

3. checkForComodification()方法用来实现快速失败机制的判断，此方法在iterator.next()方法中，必须在进入foreach循环后才会被调用；

4. 由2，当ArrayList的remove()方法在foreach删除倒数第二个元素时，继续foreach循环，**倒数第一个元素会被跳过，从而退出循环**，联合3可知，在删除倒数第二个元素后，并不会进入快速失败机制的判断。

5. iterator.remove()方法会在删除和移动元素后将cursor放回正确的位置，不会导致元素跳过，并且更新expectedModCount，是一个安全的选择。