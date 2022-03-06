## Java集合面试高频考点

## 什么是Java的集合？使用集合有什么好处？

Java 的集合也称为容器，是用来**存放数据的容器**；不过注意，集合存放的只能是引用数据类型的数据，也就是一个个的对象（如果存入基本数据类型的数据，会自动装箱成包装类）。 

 **集合的好处：** 

1.  集合的长度是可变的。 
2.  集合可以存放不同类型的对象。 
3.  使用集合之后，可以像操作基本数据类型那样来操作对象。 
4.  集合为我们提供了多种数据结构和操作的API，选用合适的集合，能够提程序性能和开发效率。 

### Java集合综述 

Java 集合，主要是由两大接口派生而来：一个是 Collection接口，主要用于存放单一元素；另一个是 Map 接口，主要用于存放键值对 。

**collection接口**

![1644378594210](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/114955-278078.png)

**Map接口**

![1644378606491](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/115007-802639.png)

List是一个继承于Collection的接口，即List是集合中的一种。List是有序的队列，List中的每一个元素都有一个索引。和Set不同，List中允许有重复的元素，可以插入多个null元素。实现List接口的集合主要有：ArrayList、LinkedList、Vector、Stack。 

 set是一个继承于Collection的接口，Set是一种不包括重复元素的Collection。 它是一个无序容器，你无法保证每个元素的存储顺序。与List一样，它同样允许null的存在但是仅有一个。由于Set接口的特殊性，所有传入Set集合中的元素都必须不同，关于API方面。Set的API和Collection完全一样。实现了Set接口的集合有：HashSet、TreeSet、LinkedHashSet、EnumSet。 

 map与List、Set接口不同，它是由一系列键值对组成的集合，提供了key到Value的映射。在Map中它保证了key与value之间的一一对应关系。也就是说一个key对应一个value，所以它不能存在相同的key值，当然value值可以相同。实现map的集合有：HashMap、HashTable、TreeMap、WeakHashMap。 

 总结：    

-  List、Set都是继承自Collection接口，Map则不是 
-  List特点：元素有放入顺序，元素可重复 ，Set特点：元素无放入顺序，元素不可重复，重复元素会覆盖掉，（注意：元素虽然无放入顺序，但是元素在set中的位置是有该元素的HashCode决定的，其位置其实是固定的，加入Set 的Object必须定义equals()方法 ，另外list支持for循环，也就是通过下标来遍历，也可以用迭代器，但是set只能用迭代，因为他无序，无法用下标来取得想要的值。） 
-  Set和List对比：      
  -  Set：检索元素效率低下，删除和插入效率高，插入和删除不会引起元素位置改变。 
  -  List：和数组类似，List可以动态增长，查找元素效率高，插入删除元素效率低，因为会引起其他元素位置改变。 
-  Map适合储存键值对的数据 
-  线程安全集合类与非线程安全集合类 :      
  -  LinkedList、ArrayList、HashSet是非线程安全的，Vector是线程安全的; 
  -  HashMap是非线程安全的，HashTable是线程安全的; 
  -  StringBuilder是非线程安全的，StringBuffer是线程安全的。 

## 常用的集合类以及它们的特点？ 

Java的集合类有两个父接口：Collection 接口和 Map 接口。 

 Collection接口主要的子接口：List接口、Set接口。 

 Map接口的主要实现类：HashMap、Hashtable、TreeMap 等。 

- List接口的主要实现类：ArrayList、Vector、LinkedList 等。 
  - List 以索引来存取元素，有序的，元素是允许重复的，可以插入多个null； 

-  Set接口的主要实现类：HashSet、TreeSet、LinkedHashSet 等。 
  - Set 不能存放重复元素，无序的，只允许一个null； 
- Map 保存键值对映射；
- List 底层实现有数组、[链表](https://www.nowcoder.com/jump/super-jump/word?word=%E9%93%BE%E8%A1%A8)两种方式；Set、Map 容器有基于哈希存储和[红黑树](https://www.nowcoder.com/jump/super-jump/word?word=%E7%BA%A2%E9%BB%91%E6%A0%91)两种方式实现
- Set 基于 Map 实现，Set 里的元素值就是 Map的键值。 

**脑图**

![1644383711619](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/131512-158738.png)

### Java集合框架中的快速失败（fail—fast）机制 

fail-fast机制，即快速失败机制，是java集合框架中的一种**错误检测机制**。

多线程下用**迭代器**遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除），则会抛出ConcurrentModification Exception。fail-fast机制并不保证在不同步的修改下一定会抛出异常，这种机制一般仅用于检测bug。

采用快速失败机制的集合容器，使用迭代器进行遍历集合时，除了通过迭代器自身的 `remove()` 方法之外，对集合进行任何其他方式的结构性修改，则会抛出 ConcurrentModificationException 异常。

在 `java.util` 包下的集合类都采用的是快速失败机制，不能在多线程下发生并发修改（迭代过程中被修改）。

#### 那么在实际测试代码当中是如何表现的呢？

------

> 先说结论：
>
> 在用for遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除），则会抛出ConcurrentModificationException。在单线程下用迭代器遍历修改，则不会报错。在多线程环境下则会报错。
>
> **例如：**假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候，线程2 **修改了集合A的结构**（是结构上面的修改，而不是简单的修改集合元素的内容），那么这个时候程序就**可能**会抛出 **ConcurrentModificationException**异常，从而产生fast-fail快速失败。 
>
>  而迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedModCount值，是的话就返回遍历；否则抛出异常，终止遍历。

#### 实现原理

原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会**改变modCount的值**。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。

这里异常的抛出条件是检测到 **modCount!=expectedmodCount**这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。

>1. 迭代器是作为当前集合的内部类实现的，当迭代器创建的时候保持了当前集合的引用；
>2. 集合内部维护一个int变量modCount，用来记录集合被修改的次数，比如add，remove等都会使该字段递增；
>3. modCount这个参数记录了某个List改变大小的次数，如果modCount改变的不符合预期，那么就会抛出异常。
>4. 迭代器内部也维护着当前集合的修改次数的字段(expectedmodCount)，迭代器创建时该字段初始化为集合的modCount值
>5. 当每一次迭代时，迭代器会比较迭代器维护的字段和modCount的值是否相等，如果不相等就抛ConcurrentModifiedException异常；
>6. 当然，如果用迭代器调用remove方法，那么集合和迭代器维护的修改次数都会递增，以保持两个状态的一致。

**优缺点**

- 单线程下效率相对较高。
- 多线程环境下，线程不安全。

那么如何解决这种问题？ 

1.  在遍历过程中，所有涉及到改变modCount值得地方全部加上synchronized。 
2.  使用 JUC 中的线程安全类来替代，比如使用 CopyOnWriteArrayList 来替代 ArrayList ，使用ConcurrentHashMap 来替代 HashMap 。 

### 安全失败

采用安全失败机制的集合容器，使用迭代器进行遍历时**不是直接在集合内容上访问**的，而是将原有集合内容进行**拷贝**，在拷贝的集合上进行遍历。

#### **原理**

迭代器在遍历时访问的是**拷贝的集合**，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发 ConcurrentModificationException 异常。

#### **优缺点**

- 由于对集合进行了拷贝，避免了 ConcurrentModificationException 异常，但拷贝时产生大量的无效对象，开销大。
- 无法保证读取到的数据是原集合中最新的数据，即迭代器进行遍历的是拷贝的集合，在遍历期间原集合发生的修改，迭代器是检测不到的。

>  **Ps：java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。**

###  fail-fast和fail-safe区别？ 

-  并发修改：    
  -  当一个或多个线程正在遍历一个集合Collection，此时另一个线程修改了这个集合的内容（添加，删除或者修改）。 
- fail-fast（快速失败）
  -  java集合中的一种机制， 在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改）（list的set就不会有），则会抛出Concurrent Modification Exception。 
  -  原理：      
    -  迭代器在遍历过程中是直接访问内部数据的，因此内部的数据在遍历的过程中不该被修改。 
    -  为了保证不被修改，迭代器内部维护了一个标记 “mode” ，当集合结构改变（添加删除或者修改），标记"mode"会被修改，而迭代器每次的hasNext()和next()方法都会检查该"mode"是否被改变，当检测到被修改时，抛出Concurrent Modification Exception。 
    -  这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出。 
    -  因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。 
  -  场景：      
    -  java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）算是一种安全机制吧。 
- 安全失败（fail—safe）
  -  java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。 
  -  fail-safe任何对集合结构的修改都会在一个复制的集合上进行修改，因此不会抛出ConcurrentModificationException 
  -  问题：      
    -  需要复制集合，产生大量的无效对象，开销大 
    -  无法保证读取的数据是目前原始数据结构中的数据。 

## List 

###  ArrayList、LinkedList、Vector 各自的特点以及优缺点？ 

![1644383788472](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/131633-987378.png)

### ArrayList 了解吗？

`ArrayList` 的底层是动态数组，它的容量能动态增长。在添加大量元素前，应用可以使用`ensureCapacity`操作增加 `ArrayList` 实例的容量。ArrayList 继承了 AbstractList ，并实现了 List 接口。  

扩容：如何不指定初始长度，会先生成一个空数组，然后扩容成10;

指定长度，生成长度数组，每次扩容会变成原来的1.5倍，然后新建一个数组进行元素复制。 

### ArrayList 的扩容机制？

-  先计算新数组的大小    
  -  一般为原数组大小的1.5倍 
  -  如果不足的话，则扩容到期望大小      
    -  addAll方法 
-  然后按照新数组的大小来创建新数组 
-  将旧数组的元素移动到新数组中 
-  将引用指向新数组 

ArrayList扩容的本质就是计算出新的扩容数组的size后实例化，并将原有数组内容复制到新数组中去。**默认情况下，新的容量会是原容量的1.5倍**。以JDK1.8为例说明:

~~~java
public boolean add(E e) {
    //判断是否可以容纳e，若能，则直接添加在末尾；若不能，则进行扩容，然后再把e添加在末尾
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //将e添加到数组末尾
    elementData[size++] = e;
    return true;
    }

// 每次在add()一个元素时，arraylist都需要对这个list的容量进行一个判断。通过ensureCapacityInternal()方法确保当前ArrayList维护的数组具有存储新元素的能力，经过处理之后将元素存储在数组elementData的尾部

private void ensureCapacityInternal(int minCapacity) {
      ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
        //如果传入的是个空数组则最小容量取默认容量与minCapacity之间的最大值
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

  private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // 若ArrayList已有的存储能力满足最低存储要求，则返回add直接添加元素；如果最低要求的存储能力>ArrayList已有的存储能力，这就表示ArrayList的存储能力不足，因此需要调用 grow();方法进行扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }


private void grow(int minCapacity) {
        // 获取elementData数组的内存空间长度
        int oldCapacity = elementData.length;
        // 扩容至原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //校验容量是否够
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //若预设值大于默认的最大值，检查是否溢出
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 调用Arrays.copyOf方法将elementData数组指向新的内存空间
         //并将elementData的数据复制到新的内存空间
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
~~~

### ArrayList为什么线程不安全还使用他呢？ 

-  因为ArrayList在正常使用的场景中，都是用来做查询的，不会涉及太频繁的增删，如果涉及频繁的增删的话，可以采用LinkedList，如果还需要线程安全的话，可以使用vector。 

### ArrayList的底层实现是数组，添加数据的话，会有问题吗？ 

-  ArrayList可以通过构造函数指定底层数组的大小 
-  无参构造器赋值底层数组为一个默认的空数组，只有真正添加数据时，才分配默认10的初始容量。 
-  ArrayList通过扩容的方式来实现长度动态增长。 
-  比如如果一个10的数组装满了，再新增的时候，会重新定义一个长度为10+10/2的数组，然后把原数组的数据原封不动的复制到新数组中，这个时候再把指向原数组的地址换到新数组。

### ArrayList在增删的时候是怎么做的么？主要说一下他为啥慢。 

-  首先，在新增方面，ArrayList有指定index新增，也有直接新增的 
-  在新增之前，会先校验数组的长度是否足够，不足的话会进行扩容。 
-  指定位置的新增：是在校验之后，对数组进行copy。比如说，要在5这个位置插入元素，则复制原数组从5的位置到末尾的数组，然后将其放在原数组5+1的位置处，最后将新增元素放到5的位置，则完成了新增。由此可见，ArrayList在指定位置的新增效率很低。 
-  直接新增的话，是在校验之后，直接在数组的尾部进行添加，由于不用复制数组，因此速度较快 
-  其次，在删除方面，ArrayList所采用的方法与新增相似，都是采用数组copy的方法 
-  比如说，要删除5位置的元素，那ArrayList就将5+1到数组末尾的元素复制到5位置处，5位置处的元素被覆盖了，看起来就像是被删除了。由此可见，删除方法效率同样很低。 

### ArrayList插入删除一定慢么？ 

-  这取决于插入和删除的位置距离数组的末端有多远。 
-  ArrayList作为堆栈来用还是比较合适的，push和pop操作完全不涉及数据移动操作。

### 怎么在遍历 ArrayList 时移除一个元素？

foreach删除会导致快速失败问题，可以使用迭代器的 remove() 方法。

~~~java
Iterator itr = list.iterator();
while(itr.hasNext()) {
      if(itr.next().equals("jay") {
        itr.remove();
      }
}
~~~

### ArrayList 和 Vector 的区别/异同？ 

Vector类 是List接口的古老实现类(JDK1.0就有了)，ArrayList类 是List接口的主要的常用的实现类(JDK1.2新增的)。 

 Vector类 的方法全都是同步的，两个线程可以安全的访问一个Vector对象； 但是如果一个线程访问Vector对象的话，要在同步操作上花费大量时间；

 而 ArrayList 不是同步的，如果不需要保证线程的安全，建议使用ArrayList，效率较高； （或者直接简单点说：Vector 线程安全但效率低，ArrayList 线程不安全但效率高。） 

 Vector扩容方式默认是 当前容量的1倍；ArrayList扩容是 当前容量×1.5+1 。 

### ArrayList 和 LinkedList 的区别/异同？ 

 （其实大部分的区别就是数据结构的区别，一个是数组，一个是双向[链表]()）。 

1.  ArrayList 底层使用的是数组实现，LinkedList 底层使用的是双向[链表]()实现。 
2.  ArrayList 随机查找和遍历速度快，插入删除速度慢；LinkedList 随机查找和遍历速度快，插入和删除速度快。 
3.  ArrayList 插入和删除元素的速度会受插入位置的影响；LinkedList 插入和删除元素的速度不会受插入位置的影响。 
4.  ArrayList 内存空间会耗费在列表后面的预留空间；LinkedList 内存空间会耗费在每个数据要多存储一个前驱和后继。 
5.  ArrayList 需要扩容，扩容是 当前容量×1.5+1 ； LinkedList 无需扩容。 
6.  ArrayList 和 LinkedList 都不是同步的，都是不保证线程安全。 

### ArryList 是线程不安全的？为什么？ 

ArrayList 是线程不安全的，因为ArrayList里的方法没有加锁，也没有使用其他保证线程安全的措施；当多个线程来对 ArrayList 进行操作时，就会出现并发修改异常。 

 **可以来演示一下集合类线程不安全的情况：** 

 多个线程向一个 ArrayList 中插入元素，代码如下： 

![1644384115931](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/132157-652125.png)

然后运行这段代码，会报 **java.util.ConcurrentModificationException（并发修改异常）** 

ArrayList 的 add() 只是先检查了容量大小，然后就直接插入数据了，并没有做任何保证线程安全的操作，如此一来，多个线程同时来调用这个方法，就会出现线程安全的问题。

### 如何解决 ArrayList 线程不安全的问题？ 

解决 ArrayList 线程安全的办法有3种： 

1.  **Vector 类替代。** 
2.  **Collections 工具类转换。** 
3.  **JUC 中的 CopyOnWriteArrayList。**

#### **解决集合类不安全的方法 1 —— Vector**

Vector 是 List 接口的古老实现类，ArrayList 是 List 接口后面新增的实现类。除了线程安全问题与扩容方式不同，Vector 几乎与 ArrayList 一样。 

 所以，可以把 Vector 作为解决 ArrayList 线程安全的一种方式（不过 Vector 效率太低）。是因为 Vector 的 add 方法加了synchronized锁。

> （顺便说一下，其实 Vector 很多其他方法也加了锁，比如读方法，相当于读的时候，同一时刻也只能有一个线程能读，效率很低。） 

#### **解决集合类不安全的方法 2 —— Collections**

 Collections 是 Collection 的工具类，其中就提供了一个 synchronizedList() 方法，可以将线程不安全的 ArrayList 转换成线程安全的。  

![1644384365672](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/132606-295559.png)

原理是在 arrayList 的 add() 的外面套了一层 synchronized 锁！并且 Collections 工具类也支持将 HashMap, HashSet 之类的转换成线程安全的。 

#### **解决集合类不安全的方法 3 —— CopyOnWriteArrayList（写时复制）**

 CopyOnWriteArrayList 是 java.util.concurrent 包里的类，是个线程安全的类。   

![1644384442660](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/132723-590096.png)

这个相比于前面那些，效率又好，读的又快，又能保证一致性。 

 **CopyOnWriteArrayList** 的思想是 写时复制。

 **写时复制：**我们要向一个文件中添加新数据时，先将原来文件拷贝一份，然后在这个拷贝文件上进行添加；而此时如果有别人读取数据，还是从原文件读取；添加数据完成后，再用这个拷贝文件替换掉原来的文件。这样做的好处是，读写分离，写的是拷贝文件，读的是原文件，可以支持多线程并发读取，而不需要加锁。 

同样，JUC 也有 HashMap, HashSet 对应线程安全的实现：

- HashSet => CopyOnWriteArraySet

- HashMap => ConcurrentHashMap   

> 这个相比于前面那些，效率又好，读的又快，又能保证一致性。 
>
>  **CopyOnWriteArrayList** 的思想是 **写时复制**。 
>
>  **写时复制：**我们要向一个文件中添加新数据时，先将原来文件拷贝一份，然后在这个拷贝文件上进行添加；而此时如果有别人读取数据，还是从原文件读取；添加数据完成后，再用这个拷贝文件替换掉原来的文件。这样做的好处是，读写分离，写的是拷贝文件，读的是原文件，可以支持多线程并发读取，而不需要加锁。 
>
>  其中的 setArray 方法中的 array 是用 volatile 修饰的，可以保证可见性： 

## CopyOnWriteArrayList 

### 读写分离 

写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。

写操作需要加锁，防止并发写入时导致写入数据丢失。写操作结束之后需要把原始数组指向新的复制数组。  

### 优缺点 

**优点**：CopyOnWriteArrayList在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。  

**缺点**：内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；  

数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。  

## Map

### HashMap的底层实现原理？ 

**Java7**： 

-  在数组中的每一个节点称为entry 
-  采用头插法进行节点插入 
-  （使用头插导致HashMap在多线程环境下进行扩容时，出现死循环的问题） 

 **Java8**：    

-  在数组中的每一个节点称为node 
-  采用尾插法进行节点插入 

HashMap 使用数组+[链表]()+[红黑树]()（JDK1.8增加了[红黑树]()部分）实现的， [链表]()长度大8（TREEIFY_THRESHOLD）时并且数组的长度已经达到64时候，会把[链表]()转换为红黑树，[红黑树]()节点个数小于6（UNTREEIFY_THRESHOLD）时才转化为[链表]()，防止频繁的转化。  

> **jdk7及jdk7之前，底层是用 数组+链表 来实现的；**

1. new HashMap() 之后，低层并不会直接创建数组。而是等 put 数据时才会创建数组，也就是是懒加载过程。

2.  put 数据时（ .put(key,value) ），如果是第一次向这个集合中put数据，会先创建一个长度为 16 的一维数组（ Node[] table ），然后存储数据。存储数据时会先调用 key 所在类的 hashCode 方法，计算出此key的哈希值，再将此哈希值经过处理计算后，得到该数据在数组table上的位置。   

3. 然后根据此位置来分情况判断是否存储：

   1. 情况一：

      此位置为空， 直接在此位置上存储put的数据。 

   2. 情况二：

       此位置不为空，则说明此位置上已有一个或多个数据了（多个数据以[链表]()形式存储）；

       那么将 put数据的key的哈希值 与 此位置上已有数据的key的哈希值进行依次比较；

      如果和它们都不同，则存储put的数据；将put的数据放在此位置上，原有数据以[链表]()形式存储：   

   3. 情况三：

      此位置不为空，且 put数据的key (假设为 key1) 的哈希值 与 此位置上已有的某个数据的key (假设为 key2 ) 的哈希值相同；

      则调用 key1 所在类的 equals() 方法与 key2 比较；（此 equals() 方法是重写过的，比较的是值；）

      若不同 (既返回false)，则存储put的数据；

      同样，将put的数据放在此位置上，原有数据以[链表](https://www.nowcoder.com/jump/super-jump/word?word=%E9%93%BE%E8%A1%A8)形式存储。 

   4. 情况四：

      若情况三中，key1，key2 equals()方法比较后的结果是相同 (既返回true)，

      则用 key1 的value1 替换 key2 的value2。 

4. 扩容：当存储的数据超出临界值，且要存放数据的位置非空时，则扩容，扩容为原来容量的2倍。
    临界值 = 当前容量 x 填充因子 (填充因子是 0.75) 

> **jdk8及jdk8之后：底层是用 数组+链表+红黑树 来实现的；**

1. new HashMap() 之后，底层并不会直接创建数组。而是等 put 数据时才会创建数组。 
2. put 数据时（ .put(key,value) ），如果是第一次向这个集合中put数据，会先创建一个长度为 16 的一维数组（ Node[] table ），然后存储数据。 
3. 存储数据的过程 和 jdk7及之前基本一样，既 先计算哈希值，然后分4种情况判断。 
4. 不同点在于 用[链表](https://www.nowcoder.com/jump/super-jump/word?word=%E9%93%BE%E8%A1%A8)存储数据时：
   1. **jdk7 是将新数据放在数组位置上，原有数据以链表形式存储在后面，也就是采用的是头插法；**
   2. **jdk8 是原有数据位置不变，而新数据以链表形式存储在最后，尾插法插入。**

![1644385134093](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/133855-880747.png)

![1644385165379](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/133926-181046.png)

5. 容量扩充
   1.  如果使用的是默认初始容量，每次扩充，容量变为原来的 2 倍；
   2. 如果使用的是自己指定的初始容量，会先将这个容量扩充为 2 的幂次方大小。 

6. jdk8还有一点不同的是，当数组的某一索引位置上的 以[链表](https://www.nowcoder.com/jump/super-jump/word?word=%E9%93%BE%E8%A1%A8)形式存储的数据 大于 8 个，且当前数组长度大于64时，此索引位置上所有数据改为[红黑树](https://www.nowcoder.com/jump/super-jump/word?word=%E7%BA%A2%E9%BB%91%E6%A0%91)存储，这样可以减少搜索查找的时间。

### 使用的hash算法？

Hash算法：取key的hashCode值、高位运算、取模运算。

~~~java
h=key.hashCode() //第一步 取hashCode值
h^(h>>>16)  //第二步 高位参与运算，减少冲突
return h&(length-1);  //第三步 取模运算
~~~

在JDK1.8的实现中，优化了高位运算的[算法]()，通过hashCode()的高16位异或低16位实现的：这么做可以在数组比较小的时候，也能保证考虑到高低位都参与到Hash的计算中，可以减少冲突，同时不会有太大的开销。  

### HashMap的扩容方式？负载因子是多少？为什是这么多？ 

-  扩容后的容量为扩容前的两倍 
-  负载因子0.75 
-  如果负载因子太大，假设为1，那么HashMap的空间利用率较高，但是hash冲突严重，[链表]()较长，查询效率低 
-  如果负载因子较小，假设为0.5，那么HashMap的冲突较少，查询效率高，但是空间利用率低，1m的数据需要用2m的空间来存储。 
-  0.75就是综合考虑了空间利用率和查询时间的结果。 

### HashMap 容量的长度为什么总是2的幂次方？ 

 为了让HashMap存取高效，要尽量减少碰撞，就是要尽量把数据分配均匀。

 Hash值大概有40亿的映射空间，只要哈希函数映射得比较均匀松散，一般来说是很难有碰撞得。 但是解决碰撞之后，新的问题也出现了，内存放不下这40亿长度得数组。

 所以用之前需要先对 数组的长度取模运算，得到的余数才能作为要存放的位置(既对应的数组下标)。 

 运算方法：

-  我们可以用 hash值和数组长度 取模，也就是 hash%n；
- 但这样的运算速度不够快，而如果我们保证数组长度为 2的幂次方时，我们就可以将式子改成 (n-1)&hash ，运算速度会大幅提升； 

 所以HashMap 容量的长度总是2的幂次方大小。 

当使用2的幂的数字作为长度时，Length-1的值是所有二进制位全为1，这种情况下，index的结果等同于HashCode后几位的值。 

### 扩容过程？

1.8扩容机制：当元素个数大于threshold时，会进行扩容，使用2倍容量的数组代替原有数组。采用尾插入的方式将原数组元素拷贝到新数组。1.8扩容之后[链表]()元素相对位置没有变化，而1.7扩容之后[链表]()元素会倒置。

1.7[链表]()新节点采用的是头插法，这样在线程一扩容迁移元素时，会将元素顺序改变，导致两个线程中出现元素的相互指向而形成循环[链表]()，1.8采用了尾插法，避免了这种情况的发生。

原数组的元素在重新计算hash之后，因为数组容量n变为2倍，那么n-1的mask范围在高位多1bit。在元素拷贝过程不需要重新计算元素在数组中的位置，只需要看看原来的hash值新增的那个bit是1还是0，是0的话索引没变，是1的话索引变成“原索引+oldCap”（根据`e.hash & (oldCap - 1) == 0`判断） 。

这样可以省去重新计算hash值的时间，而且由于新增的1bit是0还是1可以认为是随机的，因此resize的过程会均匀的把之前的冲突的节点分散到新的bucket。

### 知道HashMap 扩容时候的死循环问题吗？ 

 HashMap 1.7 插入数据时，使用的是头插法，并发下扩容时的Rehash，会出现死循环问题； 

 而 HashMap 1.8 插入数据时，改成了尾插法，解决了扩容时的死循环问题。 

 （如果还要具体一点的话，可以说一说rehash的流程，建议找篇文章看看，或者后面有时间我再写一篇。） 

###  jdk1.7版hashmap在多线程环境下的死循环问题介绍一下？ 

Java7在多线程操作HashMap时可能引起死循环，原因是扩容转移后前后[链表]()顺序倒置，在转移过程中修改了原来[链表]()中节点的引用关系。 

 Java8在同样的前提下并不会引起死循环，原因是扩容转移后前后[链表]()顺序不变，保持之前节点的引用关系，因为java8采用的是尾插法解决的，java7是采用头插入法，导致重新rehash后数据顺序相反。

[jdk1.7版hashmap在多线程环境下的死循环问题](https://blog.csdn.net/qq_33788242/article/details/89706962)

### 如何解决 HashMap 线程不安全的问题？ 

解决 HashMap 线程安全的办法同样也有3种： 

1.  **Hashtable类替代。** 
2.  **Collections 工具类转换。** 
3.  **JUC 中的 ConcurrentHashMap 替代。** 

 **Hashtable类替代** 和 **Collections 工具类转换** 这两种方法和在 ArrayList 里的用法一样，照着说就可以。 

 这里主要说一下 **ConcurrentHashMap：** 

 用**ConcurrentHashMap** 替代HashMap，可以解决线程安全的问题，但是其实 **ConcurrentHashMap** 也是有 jdk1.7 和 jdk1.8 的区别。 

- **jdk1.7**
   **采用Segment分段锁方式保证线程安全，将数据分成一段一段的存储，然后每一段数据单独一个锁；**
   所以当一个线程占用一个锁访问其中的一段数据时，其他段的数据可以被其他线程访问；
   （jdk1.7 ConcurrentHashMap底层结构由 Segment数组和 HashEntry数组 组成。一个ConcurrentHashMap里包含一个 Segment数组，数组中的每个Segment都包含一个HashEntry数组，每个HashEntry是一个[链表]()结构的元素。） 

![1644385510637](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/134511-223284.png)

-  **jdk1.8**
   **取消了Segment分段锁方式，改成了用 CAS 和 synchronized 来保证线程安全。**
   jdk1.8 的ConcurrentHashMap中锁的锁更细粒度了， synchronized 只锁定当前[链表]()或[红黑树]()的首节点，这样只要hash不冲突，就不会有线程安全问题，效率大幅提升。
   （jdk1.8 ConcurrentHashMap 底层结构由 数组+[链表]()+[红黑树]() 实现。） 

![1644385531383](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/134532-525715.png)

### put方法流程？

1. 如果table没有初始化就先进行初始化过程 
2. 使用hash[算法]()计算key的索引 
3. 判断索引处有没有存在元素，没有就直接插入 
4. 如果索引处存在元素，则遍历插入，有两种情况，一种是[链表]()形式就直接遍历到尾端插入，一种是[红黑树]()就按照[红黑树]()结构插入 
5. [链表]()的数量大于阈值8，就要转换成[红黑树]()的结构 
6. 添加成功后会检查是否需要扩容 

![1644387440516](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/141721-443235.png)

### 红黑树的特点？

- 每个节点或者是黑色，或者是红色。 
- 根节点是黑色。 
- 每个叶子节点（NIL）是黑色。 
- 如果一个节点是红色的，则它的子节点必须是黑色的。 
- 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。 

### 为什么使用红黑树而不使用AVL树？

ConcurrentHashMap 在put的时候会加锁，使用[红黑树]()插入速度更快，可以减少等待锁释放的时间。[红黑树]()是对AVL树的优化，只要求部分平衡，用非严格的平衡来换取增删节点时候旋转次数的降低，提高了插入和删除的性能。

> avl适合查询的结构，红黑树适合插入删除的场景。

### hashmap线程不安全的表现有哪些？ 

-  put操作    
  -  两个线程，在插入K-V键值对的时候都计算到了同一个桶位置，一个线程先插入，后面一个线程后插入，后面线程的键值对会覆盖掉前面的键值对 
-  扩容操作    
  -  扩容的时候是通过++size>阈值来判断的，多线程执行++size的结果会不准确导致无法扩容 
  -  多个线程进入扩容逻辑，第一个线程把扩容后的table和阈值赋值以后，第二个线程进来，有可能会导致直接扩容四倍或者更多倍 
-  删除操作    
  -  有可能把别人刚更新的节点给删除了 

### 在解决 hash 冲突的时候，为什么选择先用[链表]()，再转[红黑树]()?

因为[红黑树]()需要进行左旋，右旋，变色这些操作来保持平衡，而单[链表]()不需要。当元素小于 8 个的时候，[链表]()结构可以保证查询性能。当元素大于 8 个的时候， [红黑树]()搜索时间复杂度是 O(logn)，而[链表]()是 O(n)，此时需要[红黑树]()来加快查询速度，但是插入和删除节点的效率变慢了。如果一开始就用[红黑树]()结构，元素太少，插入和删除节点的效率又比较慢，浪费性能。

### HashMap默认加载因子是多少？为什么是 0.75？

先看下HashMap的默认构造函数：

~~~java
int threshold;             // 容纳键值对的最大值
final float loadFactor;    // 负载因子
int modCount;  
int size;  
~~~

Node[] table的初始化长度length为16，默认的loadFactor是0.75，0.75是对空间和时间效率的一个平衡选择，根据泊松分布，loadFactor 取0.75碰撞最小。一般不会修改，除非在时间和空间比较特殊的情况下 ：

- 如果内存空间很多而又对时间效率要求很高，可以降低负载因子Load factor的值 。
- 如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1。

### 一般用什么作为HashMap的key?

一般用Integer、String 这种不可变类当 HashMap 当 key。String类比较常用。

- 因为 String 是不可变的，所以在它创建的时候 hashcode 就被缓存了，不需要重新计算。这就是 HashMap 中的key经常使用字符串的原因。 
- 获取对象的时候要用到 equals() 和 hashCode() 方法，而Integer、String这些类都已经重写了 hashCode() 以及 equals() 方法，不需要自己去重写这两个方法。 
- 如果使用自定义对象去作为key，那么一定需要重写equals()方法和hashCode()方法。

### HashMap为什么线程不安全？

- 多线程下扩容死循环。JDK1.7中的 HashMap 使用头插法插入元素，在多线程的环境下，扩容的时候有可能导致**环形链表**的出现，形成死循环。 
- 在JDK1.8中，在多线程环境下，会发生**数据覆盖**的情况。 

### HashMap和HashTable的区别？

HashMap和Hashtable都实现了Map接口。

1. HashMap可以接受为null的key和value，key为null的键值对放在下标为0的头结点的[链表]()中，而Hashtable则不行。 
2. HashMap是非线程安全的，HashTable是线程安全的。Jdk1.5提供了ConcurrentHashMap，它是HashTable的替代。 
3. Hashtable很多方法是同步方法，在单线程环境下它比HashMap要慢。 
4. 哈希值的使用不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值。 

### LinkedHashMap底层原理？

HashMap是无序的，迭代HashMap所得到元素的顺序并不是它们最初放到HashMap的顺序，即不能保持它们的插入顺序。

LinkedHashMap继承于HashMap，是HashMap和LinkedList的融合体，具备两者的特性。每次put操作都会将entry插入到双向[链表]()的尾部。

![1644387997654](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/142638-189844.png)

### ConcurrentHashMap能完全替代Hashtable吗？ 

 不能。 

 首先说一下 ConcurrentHashMap 和 Hashtable的异同： 

1.  线程安全：
    ConcurrentHashMap 和 Hashtable 都是线程安全的； 
2.  底层数据结构：
    ConcurrentHashMap 的底层数据结构： jdk1.7 是用 分段数组+[链表]() 实现，jdk1.8 是用 数组+[链表]()+[红黑树]() 实现，[红黑树]()可以保证查找效率；
    Hashtable 底层数据结构是用 数组+[链表]() 实现。 
3.  保证线程安全的方式：
    ConcurrentHashMap jdk1.7 是用分段锁的方式保证线程安全，jdk1.8 是用 synchronized 和 CAS 保证线程安全；
    Hashtable 是用全表锁来保证线程安全（既一个Hashtable 只用一把锁），这种的方式的效率非常低。 

 这样一看，好像ConcurrentHashMap什么都比Hashtable好啊！为什么还是不能完全替代Hashtable ？ 

 原因在于**一致性**： 

虽然ConcurrentHashMap 的效率远高于 Hashtable，但因为 **ConcurrentHashMap的迭起器是弱一致性的，而Hashtable的迭代器是强一致性的。**所以ConcurrentHashMap是不能能完全替代Hashtable的。 

 弱一致性 简单来说就比如 我put了一个数据进去，本来应该立刻就可以get到，但是却可能在一段时间内 get不到，一致性比较弱。

 而如果是 强一致性 的话，加入数据后，马上就能get到。 

 其实要变成强一致性，就要处处用锁，甚至是用全局锁，Hashtable就是全局锁，但是这样的效率会很低。

 而ConcurrentHashMap 为了提升效率，一致性自然会变弱。 

## Hashtable的特点介绍一下？ 

-  Hashtable相比于hashmap，它是线程安全的，但是他的效率低下。 
-  原因是他对数据操作时会对整个方法进行上锁 
-  与HashMap的区别：    
  - Hashtable 是不允许键或值为 null 的，HashMap 的键值则都可以为 null
    -  hashtable,concurrenthashmap它们是用于多线程的，并发的 ，如果map.get(key)得到了null，不能判断到底是映射的value是null,还是因为没有找到对应的key而为空，而用于单线程状态的hashmap却可以用containKey（key） 去判断到底是否包含了这个null。 
    -  hashtable不能用containKey(key)来判断到底是否含有key的原因是因为一个线程先get(key)再containKey(key)，这两个方法的中间时刻，其他线程怎么操作这个key都会可能发生，例如删掉这个key 
  - 实现方法不同
    -  Hashtable 继承了 Dictionary类，而 HashMap 继承的是 AbstractMap 类。 
  - 初始化容量不同
    -  HashMap 的初始容量为：16，Hashtable 初始容量为：11，两者的负载因子默认都是：0.75。 
  - 扩容机制不同
    -  当现有容量大于总容量 * 负载因子时，HashMap 扩容规则为当前容量翻倍，Hashtable 扩容规则为当前容量翻倍 + 1 
  - 迭代器不同 
    -  HashMap 中的 Iterator 迭代器是 fail-fast 的，而 Hashtable 的 Enumerator 不是 fail-fast 的。        
      -  Hashtable 除了可以使用 entrySet 进行遍历以外，还可以基于枚举的方式来进行遍历 elements（Enumeration） 
      -  hashtable的failfast是用synchronized实现的 
    -  所以，当一个线程在遍历HashMap 时，如果有其它线程改变了HashMap 的结构，将抛出ConcurrentModificationException 异常，而Hashtable 不会

## 讲一下TreeMap？

TreeMap是一个能比较元素大小的Map集合，会对传入的key进行了大小[排序](https://www.nowcoder.com/jump/super-jump/word?word=%E6%8E%92%E5%BA%8F)。可以使用元素的自然顺序，也可以使用集合中自定义的比较器来进行[排序](https://www.nowcoder.com/jump/super-jump/word?word=%E6%8E%92%E5%BA%8F)。

~~~java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable {
}
~~~

TreeMap 的继承结构：

![1644388069753](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/142750-583980.png)

**TreeMap的特点：**

1. TreeMap是有序的key-value集合，通过[红黑树]()实现。根据键的自然顺序进行[排序]()或根据提供的Comparator进行[排序]()。 
2. TreeMap继承了AbstractMap，实现了NavigableMap接口，支持一系列的导航方法，给定具体搜索目标，可以返回最接近的匹配项。如floorEntry()、ceilingEntry()分别返回小于等于、大于等于给定键关联的Map.Entry()对象，不存在则返回null。lowerKey()、floorKey、ceilingKey、higherKey()只返回关联的key。 

### HashMap 是 TreeMap 如何选用？ 

 在需要大量插入、删除和查找元素这种操作的，选择HashMap，因为HashMap 底层使用数据+[链表]()+[红黑树]()实现，对于插入、删除、查找的性能都不错，但是HashMap的结果是没有[排序]()的。 

 在需要对集合[排序]()的时候，选择 TreeMap ，TreeMap 基于[红黑树]()实现，TreeMap 的映射根据键的自然顺序进行[排序]()，或者根据创建映射时提供的 Comparator 进行[排序]()，具体取决于使用的构造方法。

### 解决hash冲突的办法有哪些？HashMap用的哪种？

解决Hash冲突方法有:开放定址法、再哈希法、链地址法。HashMap中采用的是 链地址法 。

开放定址法    

-  发生冲突时在散列表（也就是数组里）里去寻找合适的位置存取对应的元素      
  -  线性探索        
    -  直接找相邻的位置 
  -  平方探索        
    -  不找相邻的，跳跃探索。比如按照 i^2 的规律来跳跃探测 
  -  双散列（再哈希） 

 再哈希法    

-  有多个不同的Hash函数，当发生冲突时，使用第二个，第三个，….，等哈希函数计算地址，直到无冲突 

 链地址法（HashMap采用的方法）    

-  每个[哈希表]()节点都有一个next指针，多个[哈希表]()节点可以用next指针构成一个单向[链表]()，被分配到同一个索引上的多个节点可以用这个单向 [链表]()连接起来 

 建立公共溢出区 

- 开放定址法基本思想就是，如果`p=H(key)`出现冲突时，则以`p`为基础，再次hash，`p1=H(p)`,如果p1再次出现冲突，则以p1为基础，以此类推，直到找到一个不冲突的哈希地址`pi`。 因此开放定址法所需要的hash表的长度要大于等于所需要存放的元素，而且因为存在再次hash，所以`只能在删除的节点上做标记，而不能真正删除节点。` 
- 再哈希法提供多个不同的hash函数，当`R1=H1(key1)`发生冲突时，再计算`R2=H2(key1)`，直到没有冲突为止。 这样做虽然不易产生堆集，但增加了计算的时间。 
- 链地址法将哈希值相同的元素构成一个同义词的单[链表](),并将单[链表]()的头指针存放在[哈希表]()的第i个单元中，查找、插入和删除主要在同义词[链表]()中进行。[链表]()法适用于经常进行插入和删除的情况。 

## Set

## HashSet底层原理？

HashSet 基于 HashMap 实现。放入HashSet中的元素实际上由HashMap的key来保存，而HashMap的value则存储了一个静态的Object对象。

~~~java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map; //基于HashMap实现
    //...
}
~~~

### HashSet、LinkedHashSet 和 TreeSet 的区别？

`HashSet` 是 `Set` 接口的主要实现类 ，`HashSet` 的底层是 `HashMap`，线程不安全的，可以存储 null 值；

`LinkedHashSet` 是 `HashSet` 的子类，能够按照添加的顺序遍历；

`TreeSet` 底层使用[红黑树]()，能够按照添加元素的顺序进行遍历，[排序]()的方式可以自定义。

### HashSet 和 HashMap 的区别？ 

HashMap是实现了Map接口，存储的是键值对；HashSet 是实现了Set接口，只存储对象。 

 HashMap 使用键来计算哈希值；HashSet 是使用成员对象来计算哈希值； 

 HashMap 比 HashSet 快。 

 HashSet 的底层其实是基于 HashMap 实现的，大部分方法都是直接调用 HashMap中的方法。
 看下源码： 

![1644385812905](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/135013-785382.png)

> HashSet中大部分调用的是HashMap中的方法，一小部分是自己实现的方法。

## HashMap(jdk1.8)

![1644380207165](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/121648-309480.png)

### get流程 

   1.hashcode值经过一个扰动函数（高八位和低八位进行异或，加大低位的随机性，减小碰撞概率），然后进行取模，取得下标  

2. 判断数组元素是否为空，遍历链表调用equals方法找value值

Put流程 

   1.先判断当前数组是否为空，如果为空调用resize方法扩容为16  

2. hashcode值经过一个扰动函数（高八位和低八位进行异或，加大低位的随机性，减小碰撞概率），然后进行取模，取得下标 

3. 判断数组元素是否为空，为空直接插入，不为空调用equal方法查找key，key存在就进行覆盖；不存在的话1.7采用头插法插入[链表]()中，1.8采用尾插法插入[链表]()中，如果[链表]()长度超过8而且数组长度大于64，会转化成[红黑树]()（O(logn)）； 

4. 然后判断当前数组元素个数是否大于数组长度*0.75的负载因子，然后新建一个数组长度2倍的数组，重新计算hash值；1.8的时候会先判断扩容后的hash值新增的位是0还是1，是0的话下标不变，是1的话下标会增加扩容的长度

扩展： 

>​    **1. 线程不安全**   
>
>​     1.7扩容时会因为头插造成循环[链表]()和数据覆盖；1.8采用尾插法并进行hash碰撞检测，多线程下容易造成数据覆盖    
>
>​     **2. 为什么长度为8变成红黑树，6变成链表**    
>
> 	2.1 [红黑树]()时间复杂度o(logn)，但是空间复杂度高，时间和空间的均衡 
>	
> 	2.2 泊松分布得出[链表]()长度超过8的概率很低     
>
>​      **3.**     **负载因子为什么是0.75**     
>
> 	3.1 提高空间利用率和减少查询成本的折中 
>
>​	3.2 根据泊松分布，0.75时[链表]()长度超过8的概率很低      
>
>​       **4.红黑树**      
>
> [红黑树]()是每个节点都带有颜色属性的二叉查找树，颜色为红色或黑色。在二叉查找树强制一般要求以外，对于任何有效的[红黑树]()增加了如下的额外要求： 
>
> 1、节点是红色或黑色。 
>
> 2、根是黑色。 
>
> 3、所有叶子都是黑色（叶子是NIL节点）。 
>
> 4、每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。） 
>
> 5、从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。 

## Hashtable 

Hashtable使用synchronized来进行同步  

> **1.取map所有值** 
>
>  1.1 keyset：将key值存入set集合，然后通过get()方法取value值 
>
>  1.2 entrySet：存放key和value的键值对 
>
>   **2.**Hashtable和Hashmap区别
>
>  2.1 Hashmap线程不安全，Hashtable线程安全 
>
>  2.2 Hashmap允许有一个null key，Hashtable不允许 
>
>  2.3 Hashmap初始容量为16，每次扩容2倍；Hashtable初始容量为11，每次扩容变成2n+1。Hashmap会将给定容量扩容成2的幂次方 

## ConcurrentHashMap 

### jdk1.7实现 

 一个ConcurrentHashMap里包含一个Segment类数组，Segment类实现ReentrantLock，包括一个HashEntry数组和一个count变量表示HashEntry对象的个数； 

  put操作时会对当前Segment对象进行加锁；get时因为将count和HashEntry的value设置成volatile，保证了可见性，除非读到了null值才会加锁重读；size操作是通过尝试两次统计count的和，如果有变化给每个Segment加锁进行统计。 

Segment数组的意义就是将一个大的table分割成多个小的table来进行加锁，也就是上面的提到的锁分离技术，而每一个Segment元素存储的是HashEntry数组+[链表]()，这个和HashMap的数据存储结构一样。 

 put操作：      

-  从上Segment的继承体系可以看出，Segment实现了ReentrantLock,也就带有锁的功能，当执行put操作时，会进行第一次key的hash来定位Segment的位置，如果该Segment还没有初始化，即通过CAS操作进行赋值，然后进行第二次hash操作，找到相应的HashEntry的位置，这里会利用继承过来的锁的特性，在将数据插入指定的HashEntry位置时（[链表]()的尾端），会通过继承ReentrantLock的tryLock（）方法尝试去获取锁，如果获取成功就直接插入相应的位置，如果已经有线程获取该Segment的锁，那当前线程会以自旋的方式去继续的调用tryLock（）方法去获取锁，超过指定次数就挂起，等待唤醒 

 get操作：      

-  ConcurrentHashMap的get操作跟HashMap类似，只是ConcurrentHashMap第一次需要经过一次hash定位到Segment的位置，然后再hash定位到指定的HashEntry，遍历该HashEntry下的[链表]()进行对比，成功就返回，不成功就返回null 

 size操作:      

-  第一种方案他会使用不加锁的模式去尝试多次计算ConcurrentHashMap的size，最多三次，比较前后两次计算的结果，结果一致就认为当前没有元素加入，计算的结果是准确的 
-  第二种方案是如果第一种方案不符合，他就会给每个Segment加上锁，然后计算ConcurrentHashMap的size返回 

### Jdk1.8实现 

ConcurrentHashMap取消了Segment分段锁，采用CAS和synchronized  来保证并发安全。数据结构跟HashMap1.8的结构类似，数组+[链表]()/红黑[二叉树]()；  

put操作时如果没有hash冲突则用CAS插入元素，否则用synchronized锁定当前[链表]()或红黑[二叉树]()的首节点，添加成功后会计算size；

- put操作： 

  1.  如果没有初始化就先调用initTable（）方法来进行初始化过程 
  2.  如果没有hash冲突（该桶为空）就直接CAS插入 
  3.  如果还在进行扩容操作就先进行扩容 
  4.  如果存在hash冲突，就对该桶的头节点进行加锁来保证线程安全，这里有两种情况，一种是[链表]()形式就直接遍历到尾端插入，一种是[红黑树]()就按照[红黑树]()结构插入， 
  5.  最后一个如果该[链表]()的数量大于阈值8，就要先转换成黑红树的结构，break再一次进入循环 
  6.  如果添加成功就调用addCount（）方法统计size，并且检查是否需要扩容 

  -  在并发处理中使用的是乐观锁，当有冲突的时候才进行并发处理 

-  get操作： 

  -  计算hash值，定位到该table索引位置，如果是首节点符合就返回 
  -  如果遇到扩容的时候，会调用标志正在扩容节点ForwardingNode的find方法，查找该节点，匹配就返回 
  -  以上都不符合的话，就往下遍历节点，匹配就返回，否则最后就返回null 

-  size操作： 

  -  在JDK1.8版本中，对于size的计算，在扩容和addCount()方法就已经有处理了，JDK1.7是在调用size()方法才去计算，其实在并发集合中去计算size是没有多大的意义的，因为size是实时在变的，只能计算某一刻的大小，但是某一刻太快了，人的感知是一个时间段，所以并不是很精确 

 总结： 

-  JDK1.8版本的ConcurrentHashMap的数据结构已经接近HashMap，相对而言，ConcurrentHashMap只是增加了同步的操作来控制并发，从JDK1.7版本的ReentrantLock+Segment+HashEntry，到JDK1.8版本中synchronized+CAS+HashEntry+[红黑树]()

### put()执行流程

在put的时候需要锁住Segment，保证并发安全。调用get的时候不加锁，因为node数组成员val和指针next是用volatile修饰的，更改后的值会立刻刷新到主存中，保证了可见性，node数组table也用volatile修饰，保证在运行过程对其他线程具有可见性。  

~~~java
transient volatile Node<K,V>[] table;

static class Node<K,V> implements Map.Entry<K,V> {
    volatile V val;
    volatile Node<K,V> next;
}
~~~

put 操作流程：

1. 如果table没有初始化就先进行初始化过程 
2. 使用hash[算法]()计算key的位置 
3. 如果这个位置为空则直接CAS插入，如果不为空的话，则取出这个节点来 
4. 如果取出来的节点的hash值是MOVED(-1)的话，则表示当前正在对这个数组进行扩容，复制到新的数组，则当前线程也去帮助复制 
5. 如果这个节点，不为空，也不在扩容，则通过synchronized来加锁，进行添加操作，这里有两种情况，一种是[链表]()形式就直接遍历到尾端插入或者覆盖掉相同的key，一种是[红黑树]()就按照[红黑树]()结构插入 
6. [链表]()的数量大于阈值8，就会转换成[红黑树]()的结构或者进行扩容（table长度小于64） 
7. 添加成功后会检查是否需要扩容 

### 扩容机制

数组扩容transfer方法中会设置一个步长，表示一个线程处理的数组长度，最小值是16。在一个步长范围内只有一个线程会对其进行复制移动操作。

#### ConcurrentHashMap 和 Hashtable 的区别？

1. Hashtable通过使用synchronized修饰方法的方式来实现多线程同步，因此，Hashtable的同步会锁住整个数组。在高并发的情况下，性能会非常差。ConcurrentHashMap采用了更细粒度的锁来提高在并发情况下的效率。注：Synchronized容器（同步容器）也是通过synchronized关键字来实现线程安全，在使用的时候会对所有的数据加锁。 
2. Hashtable默认的大小为11，当达到阈值后，每次按照下面的公式对容量进行扩充：newCapacity = oldCapacity * 2 + 1。ConcurrentHashMap默认大小是16，扩容时容量扩大为原来的2倍。 

## 迭代器 （Iterator ）

### 什么是迭代器（Iterator ）？ 

首先要知道：迭代器是一种模式，它可以使得对于序列类型的数据结构的遍历行为与被遍历的对象分离，也就是可以使我们无需关心该序列的底层结构是什么样子的。只要拿到这个对象,使用迭代器就可以遍历这个对象的内部。 

 Java 为我们提供了一个迭代器的接口就是 Iterator 。 

![1644385931710](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/135212-954358.png)

1. next()：返回序列中的下一个元素。 
2.  hasNext()：检查序列中是否还有元素。 
3.  使用remove()：将迭代器新返回的元素删除。 
4. Java 采用了迭代器来为各种容器提供了公共的操作接口。这样使得对容器的遍历操作与其具体的底层实现相隔离，达到解耦的效果。 

###  Iterator 和foreach 遍历集合的区别？ 

Iterator 和 foreach 都可以遍历集合； 

foreach 不可以在遍历的过程中删除元素，不然会出现 **并发修改（ConcurrentModificationException）** （基于快速失败机制，等下会说）； 

 使用 Iterator 遍历集合时，可以删除集合中的元素：

![1644386018921](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/135339-88643.png)

### Iterator 和 ListIterator 有什么区别？

ListIterator 是 Iterator的增强版。

- ListIterator遍历可以是逆向的，因为有previous()和hasPrevious()方法，而Iterator不可以。 
- ListIterator有add()方法，可以向List添加对象，而Iterator却不能。 
- ListIterator可以定位当前的索引位置，因为有nextIndex()和previousIndex()方法，而Iterator不可以。 
- ListIterator可以实现对象的修改，set()方法可以实现。Iierator仅能遍历，不能修改。 
- ListIterator只能用于遍历List及其子类，Iterator可用来遍历所有集合。 

## 讲一下ArrayDeque？

ArrayDeque实现了双端队列，内部使用循环数组实现，默认大小为16。它的特点有：

1. 在两端添加、删除元素的效率较高
2. 根据元素内容查找和删除的效率比较低。
3. 没有索引位置的概念，不能根据索引位置进行操作。

ArrayDeque和LinkedList都实现了Deque接口，如果只需要从两端进行操作，ArrayDeque效率更高一些。如果同时需要根据索引位置进行操作，或者经常需要在中间进行插入和删除（LinkedList有相应的 api，如add(int index, E e)），则应该选LinkedList。

ArrayDeque和LinkedList都是线程不安全的，可以使用Collections工具类中synchronizedXxx()转换成线程同步。

## 哪些集合类是线程安全的？哪些不安全？

线性安全的集合类：

- Vector：比ArrayList多了同步机制。 
- Hashtable。 
- ConcurrentHashMap：是一种高效并且线程安全的集合。 
- Stack：栈，也是线程安全的，继承于Vector。 

线性不安全的集合类：

- Hashmap 
- Arraylist 
- LinkedList 
- HashSet 
- TreeSet 
- TreeMap 

## 并发容器

JDK 提供的这些容器大部分在 `java.util.concurrent` 包中。

- **ConcurrentHashMap:** 线程安全的 HashMap 
- **CopyOnWriteArrayList:** 线程安全的 List，在读多写少的场合性能非常好，远远好于 Vector. 
- **ConcurrentLinkedQueue:** 高效的并发队列，使用[链表]()实现。可以看做一个线程安全的 LinkedList，这是一个非阻塞队列。 
- **BlockingQueue:** 阻塞队列接口，JDK 内部通过[链表]()、数组等方式实现了这个接口。非常适合用于作为数据共享的通道。 
- **ConcurrentSkipListMap:** 跳表的实现。使用跳表的数据结构进行快速查找。

### ConcurrentLinkedQueue

非阻塞队列。高效的并发队列，使用[链表]()实现。可以看做一个线程安全的 LinkedList，通过 CAS 操作实现。

如果对队列加锁的成本较高则适合使用无锁的 ConcurrentLinkedQueue 来替代。适合在对性能要求相对较高，同时有多个线程对队列进行读写的场景。

**非阻塞队列中的几种主要方法：**

- add(E e) : 将元素e插入到队列末尾，如果插入成功，则返回true；如果插入失败（即队列已满），则会抛出异常；
- remove() ：移除队首元素，若移除成功，则返回true；如果移除失败（队列为空），则会抛出异常；
- offer(E e) ：将元素e插入到队列末尾，如果插入成功，则返回true；如果插入失败（即队列已满），则返回false；
- poll() ：移除并获取队首元素，若成功，则返回队首元素；否则返回null；
- peek() ：获取队首元素，若成功，则返回队首元素；否则返回null

对于非阻塞队列，一般情况下建议使用offer、poll和peek三个方法，不建议使用add和remove方法。因为使用offer、poll和peek三个方法可以通过返回值判断操作成功与否，而使用add和remove方法却不能达到这样的效果。

### 阻塞队列

阻塞队列是java.util.concurrent包下重要的数据结构，BlockingQueue提供了线程安全的队列访问方式：当阻塞队列进行插入数据时，如果队列已满，线程将会阻塞等待直到队列非满；从阻塞队列取数据时，如果队列已空，线程将会阻塞等待直到队列非空。并发包下很多高级同步类的实现都是基于BlockingQueue实现的。BlockingQueue 适合用于作为数据共享的通道。

使用阻塞[算法]()的队列可以用一个锁（入队和出队用同一把锁）或两个锁（入队和出队用不同的锁）等方式来实现。

阻塞队列和一般的队列的区别就在于：

1. 多线程支持，多个线程可以安全的访问队列 
2. 阻塞操作，当队列为空的时候，消费线程会阻塞等待队列不为空；当队列满了的时候，生产线程就会阻塞直到队列不满。 

![1644389429770](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/09/145030-536538.png)

#### JDK提供的阻塞队列

JDK 7 提供了7个阻塞队列，如下

**ArrayBlockingQueue** 

有界阻塞队列，底层采用数组实现。ArrayBlockingQueue 一旦创建，容量不能改变。其并发控制采用可重入锁来控制，不管是插入操作还是读取操作，都需要获取到锁才能进行操作。此队列按照先进先出（FIFO）的原则对元素进行[排序]()。默认情况下不能保证线程访问队列的公平性，参数`fair`可用于设置线程是否公平访问队列。为了保证公平性，通常会降低吞吐量。

~~~java
private static ArrayBlockingQueue<Integer> blockingQueue = new ArrayBlockingQueue<Integer>(10,true);//fair
~~~

**LinkedBlockingQueue**

LinkedBlockingQueue是一个用单向[链表]()实现的有界阻塞队列，可以当做无界队列也可以当做有界队列来使用。通常在创建 LinkedBlockingQueue 对象时，会指定队列最大的容量。此队列的默认和最大长度为`Integer.MAX_VALUE`。此队列按照先进先出的原则对元素进行[排序]()。与 ArrayBlockingQueue 相比起来具有更高的吞吐量。

**PriorityBlockingQueue** 

支持优先级的**无界**阻塞队列。默认情况下元素采取自然顺序升序排列。也可以自定义类实现`compareTo()`方法来指定元素[排序]()规则，或者初始化PriorityBlockingQueue时，指定构造参数`Comparator`来进行[排序]()。

PriorityBlockingQueue 只能指定初始的队列大小，后面插入元素的时候，如果空间不够的话会**自动扩容**。

PriorityQueue 的线程安全版本。不可以插入 null 值，同时，插入队列的对象必须是可比较大小的（comparable），否则报 ClassCastException 异常。它的插入操作 put 方法不会 block，因为它是无界队列（take 方法在队列为空的时候会阻塞）。

**DelayQueue** 

支持延时获取元素的无界阻塞队列。队列使用PriorityBlockingQueue来实现。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。

**SynchronousQueue**

不存储元素的阻塞队列，每一个put必须等待一个take操作，否则不能继续添加元素。支持公平访问队列。

SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身不存储任何元素，非常适合传递性场景。SynchronousQueue的吞吐量高于LinkedBlockingQueue和ArrayBlockingQueue。

**LinkedTransferQueue**

由[链表]()结构组成的无界阻塞TransferQueue队列。相对于其他阻塞队列，多了tryTransfer和transfer方法。

transfer方法：如果当前有消费者正在等待接收元素（take或者待时间限制的poll方法），transfer可以把生产者传入的元素立刻传给消费者。如果没有消费者等待接收元素，则将元素放在队列的tail节点，并等到该元素被消费者消费了才返回。

tryTransfer方法：用来试探生产者传入的元素能否直接传给消费者。如果没有消费者在等待，则返回false。和上述方法的区别是该方法无论消费者是否接收，方法立即返回。而transfer方法是必须等到消费者消费了才返回。

## 不同集合容量与扩容系数？ 

-  ArrayList 
  -  1.7：容量10，饿汉式，扩容1.5倍+1 
  -  1.8：容量10，懒汉式，扩容1.5倍。1.5倍大小不足时，按照需要的容量进行扩容 
-  vector 
  -  容量10，饿汉式，默认扩容2倍，构造函数指定了扩容大小则按指定的进行扩容 
-  HashMap 
  -  初始容量16，负载因子0.75，扩容2倍 
  -  树形化阈值：8，总64元素 
  -  链式化阈值：6，下一次元素迁移中变回来 
-  HashTable 
  -  初始容量11，负载因子0.75，扩容2倍+1 
-  ConcurrentHashMap 
  -  初始容量16，负载因子0.75，扩容2倍 

## java8的ConcurrentHashMap为何放弃分段锁？ 

-  在JDK1.7中，通过控制Segment的个数来控制并发级别（Segment的个数在ConcurrentHashMap创建后是不可以改变的） 
-  随着元素的增多，每个Segment包含的元素越多，锁的粒度会越来越大，竞争会逐渐激烈 
-  而且分段会造成内存空间的碎片化，比较浪费内存空间 
-  优点： 
  -  相比于将整个map加锁的方法，分段机制能够提高并发程度 
-  在JDK1.8时，如果发生hash冲突，会对[链表]()的头结点使用synchronized加锁，然后进行插入操作 
-  如果没有冲突，就使用CAS来进行插入 
-  随着元素的增多，会进行扩容操作，锁的粒度维持在一个较低的水平 
-  使用synchronized加锁，可以避免所有节点都继承ReentranLock，因为只需要[链表]()的头节点进行加锁操作就可以了，没必要所有节点都具备加锁功能，能够有效减少内存的开销 
-  synchronized通过锁升级等相应的处理后，性能与ReentranLock持平 

