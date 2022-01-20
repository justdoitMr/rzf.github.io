
<!-- TOC -->

- [HashMap 的数据结构？](#hashmap-的数据结构)
- [为什么在解决 hash 冲突的时候，不直接用红黑树？而选择先用链表，再转红黑树?](#为什么在解决-hash-冲突的时候不直接用红黑树而选择先用链表再转红黑树)
- [不用红黑树，用二叉查找树可以么?](#不用红黑树用二叉查找树可以么)
- [当链表转为红黑树后，什么时候退化为链表?](#当链表转为红黑树后什么时候退化为链表)
- [为什么链表改为红黑树的阈值是 8?](#为什么链表改为红黑树的阈值是-8)
- [HashMap 中  key 的存储索引是怎么计算的？](#hashmap-中--key-的存储索引是怎么计算的)
  - [JDK1.8 为什么要 hashcode 异或其右移十六位的值？](#jdk18-为什么要-hashcode-异或其右移十六位的值)
  - [为什么 hash 值要与length-1相与？](#为什么-hash-值要与length-1相与)
- [补充数组容量计算](#补充数组容量计算)
- [那么为什么要把数组长度设计为2的幂次方呢？](#那么为什么要把数组长度设计为2的幂次方呢)
- [table数组长度永远为2的幂次方](#table数组长度永远为2的幂次方)
- [数组元素 & 链表节点的 实现类](#数组元素--链表节点的-实现类)
- [红黑树节点 实现类](#红黑树节点-实现类)
- [主要使用API（方法、函数）](#主要使用api方法函数)
- [重要参数](#重要参数)
- [默认加载因子是多少？为什么是 0.75，不是 0.6 或者 0.8](#默认加载因子是多少为什么是-075不是-06-或者-08)
- [总结 数据结构 & 参数方面与 `JDK 1.7`的区别](#总结-数据结构--参数方面与-jdk-17的区别)
- [源码分析](#源码分析)
  - [声明1个HashMap的对象](#声明1个hashmap的对象)
  - [步骤2：向HashMap添加数据（成对 放入 键 - 值对）](#步骤2向hashmap添加数据成对-放入-键---值对)
    - [分析1：hash（key）](#分析1hashkey)
    - [问题1：为什么不直接采用经过hashCode（）处理的哈希码 作为 存储数组table的下标位置？](#问题1为什么不直接采用经过hashcode处理的哈希码-作为-存储数组table的下标位置)
    - [问题2：为什么采用 哈希码 与运算(&) （数组长度-1） 计算数组下标？](#问题2为什么采用-哈希码-与运算-数组长度-1-计算数组下标)
    - [问题3：为什么在计算数组下标前，需对哈希码进行二次处理：扰动处理？](#问题3为什么在计算数组下标前需对哈希码进行二次处理扰动处理)
    - [分析2：putVal(hash(key), key, value, false, true);](#分析2putvalhashkey-key-value-false-true)
      - [主要讲解点1：计算完存储位置后，具体该如何存放数据到哈希表中](#主要讲解点1计算完存储位置后具体该如何存放数据到哈希表中)
      - [主要讲解点2：扩容机制（即 resize（）函数方法）](#主要讲解点2扩容机制即-resize函数方法)
      - [此处主要讲解： `JDK 1.8`扩容时，数据存储位置重新计算的方式](#此处主要讲解-jdk-18扩容时数据存储位置重新计算的方式)
  - [步骤3：从HashMap中获取数据](#步骤3从hashmap中获取数据)
  - [步骤4：对HashMap的其他操作](#步骤4对hashmap的其他操作)
  - [源码总结](#源码总结)
  - [与 `JDK 1.7` 的区别](#与-jdk-17-的区别)
    - [数据结构](#数据结构)
    - [获取数据时（获取数据 类似）](#获取数据时获取数据-类似)
    - [扩容机制](#扩容机制)
    - [其他问题](#其他问题)
      - [解决哈希冲突](#解决哈希冲突)
      - [为什么HashMap具备下述特点：键-值（key-value）都允许为空、线程不安全、不保证有序、存储位置随时间变化](#为什么hashmap具备下述特点键-值key-value都允许为空线程不安全不保证有序存储位置随时间变化)
      - [为什么 HashMap 中 String、Integer 这样的包装类适合作为 key 键](#为什么-hashmap-中-stringinteger-这样的包装类适合作为-key-键)
  - [HashMap 中的 `key`若 `Object`类型， 则需实现哪些方法？](#hashmap-中的-key若-object类型-则需实现哪些方法)
- [扩容](#扩容)
  - [先来看下 JDK1.7 的代码：](#先来看下-jdk17-的代码)
  - [jdk1.8优化](#jdk18优化)
  - [链表树化](#链表树化)
- [查找](#查找)
- [插入](#插入)
  - [JDK1.7 和1.8 的put方法区别是什么？](#jdk17-和18-的put方法区别是什么)
- [删除](#删除)
- [遍历](#遍历)
- [equasl和hashcode](#equasl和hashcode)
- [HashMap 的工作原理？](#hashmap-的工作原理)
- [为什么把链表的头插法换为尾插法？](#为什么把链表的头插法换为尾插法)
- [当两个对象的 hashCode 相同会发生什么？](#当两个对象的-hashcode-相同会发生什么)
- [你知道 hash 的实现吗？为什么要这样实现？](#你知道-hash-的实现吗为什么要这样实现)
- [为什么要用异或运算符？](#为什么要用异或运算符)
- [HashMap 的 table 的容量如何确定？loadFactor 是什么？该容量如何变化？这种变化会带来什么问题？](#hashmap-的-table-的容量如何确定loadfactor-是什么该容量如何变化这种变化会带来什么问题)
- [HashMap中put方法的过程？](#hashmap中put方法的过程)
- [数组扩容的过程？](#数组扩容的过程)
- [拉链法导致的链表过深问题为什么不用二叉查找树代替，而选择红黑树？为什么不一直使用红黑树？](#拉链法导致的链表过深问题为什么不用二叉查找树代替而选择红黑树为什么不一直使用红黑树)
- [说说你对红黑树的见解？](#说说你对红黑树的见解)
- [jdk8中对HashMap做了哪些改变？](#jdk8中对hashmap做了哪些改变)
- [HashMap，LinkedHashMap，TreeMap 有什么区别？](#hashmaplinkedhashmaptreemap-有什么区别)
- [HashMap & TreeMap & LinkedHashMap 使用场景？](#hashmap--treemap--linkedhashmap-使用场景)
- [HashMap 和 HashTable 有什么区别？](#hashmap-和-hashtable-有什么区别)
- [Java 中的另一个线程安全的与 HashMap 极其类似的类是什么？同样是线程安全，它与 HashTable 在线程同步上有什么不同？](#java-中的另一个线程安全的与-hashmap-极其类似的类是什么同样是线程安全它与-hashtable-在线程同步上有什么不同)
- [HashMap & ConcurrentHashMap 的区别？](#hashmap--concurrenthashmap-的区别)
- [为什么 ConcurrentHashMap 比 HashTable 效率要高？](#为什么-concurrenthashmap-比-hashtable-效率要高)
- [针对 ConcurrentHashMap 锁机制具体分析（JDK 1.7 VS JDK 1.8）](#针对-concurrenthashmap-锁机制具体分析jdk-17-vs-jdk-18)
- [ConcurrentHashMap 在 JDK 1.8 中，为什么要使用内置锁 synchronized 来代替重入锁 ReentrantLock？](#concurrenthashmap-在-jdk-18-中为什么要使用内置锁-synchronized-来代替重入锁-reentrantlock)
- [ConcurrentHashMap 简单介绍？](#concurrenthashmap-简单介绍)
- [ConcurrentHashMap 的并发度是什么？](#concurrenthashmap-的并发度是什么)
- [Jdk1.7线程不安全体现在哪里](#jdk17线程不安全体现在哪里)
  - [扩容造成死循环分析过程](#扩容造成死循环分析过程)
  - [扩容造成数据丢失分析过程](#扩容造成数据丢失分析过程)
  - [jdk1.8中HashMap](#jdk18中hashmap)

<!-- /TOC -->


![1641433174793](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/093936-353784.png)

## HashMap 的数据结构？

> 在 JDK1.8 中，HashMap 是由 `数组+链表+红黑树`构成(**1.7版本是数组+链表**)
>
> 解释一下，之所以将链表转换为红黑树结构，是因为如果元素发生冲突的很多，那么就会将所有发生冲突的元素连接成一个链表，而链表在查询一个元素的效率很低。
>
> 转换为红黑树结构，可以提高查询效率，因为红黑树天生就是有序的树，查询元素时间复杂度在logn级别。

在 JDK1.7 和 JDK1.8 中有所差别：

在 JDK1.7 中，由“数组+链表”组成，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的。

在 JDK1.8 中，由“数组+链表+红黑树”组成。当链表过长，则会严重影响 HashMap 的性能，红黑树搜索时间复杂度是 O(logn)，而链表是糟糕的 O(n)。因此，JDK1.8 对数据结构做了进一步的优化，引入了红黑树，链表和红黑树在达到一定条件会进行转换：

- 当链表长度超过 8 且数据总量大于等于 64 才会转红黑树。
- 将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树，以减少搜索时间。

~~~java
transient Node<K,V>[] table;
~~~

![1641433269601](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/094111-86779.png)

**关于红黑树**

![1641433312210](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/094154-551311.png)

**如何工作**

当一个值中要存储到HashMap中的时候会根据Key的值来计算出他的hash，通过hash值来确认存放到数组中的位置，如果发生hash冲突就以链表的形式存储，当链表过长的话，HashMap会把这个链表转换成红黑树来存储，如图所示：

![1641383827702](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/195708-85024.png)



## 为什么在解决 hash 冲突的时候，不直接用红黑树？而选择先用链表，再转红黑树?

因为红黑树本身也是一种很复杂的数据结构，需要进行左旋，右旋，变色这些操作来保持平衡，而单链表不需要。

当元素小于 8 个的时候，此时做查询操作，链表结构已经能保证查询性能。

当元素大于 8 个的时候， 红黑树搜索时间复杂度是 O(logn)，而链表是 O(n)，此时需要红黑树来加快查询速度，但是新增节点的效率变慢了。

因此，如果一开始就用红黑树结构，元素太少，新增效率又比较慢，无疑这是浪费性能的。

## 不用红黑树，用二叉查找树可以么?

可以。但是二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。

## 当链表转为红黑树后，什么时候退化为链表?

为6的时候退转为链表。**中间有个差值7可以防止链表和树之间频繁的转换**。假设一下，如果设计成链表个数超过8则链表转换成树结构，链表个数小于8则树结构转换成链表，如果一个HashMap不停的插入、删除元素，链表个数在8左右徘徊，就会频繁的发生树转链表、链表转树，效率会很低。

## 为什么链表改为红黑树的阈值是 8?

是因为泊松分布，我们来看作者在源码中的注释：

![1641384034294](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/200036-127148.png)

## HashMap 中  key 的存储索引是怎么计算的？

首先根据key的值计算出hashcode的值，然后根据hashcode计算出hash值，最后通过hash&（length-1）计算得到存储的位置。看看源码的实现：

**jdk1.7的计算方式**

![1641384234100](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/091947-907623.png)

其中由有4次位运算和5次异或运算。

**jdk1.8的计算方式**

![1641384260739](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/200432-227628.png)

减少了运算次数，一次位运算和一次异或运算。

这里的 Hash 算法本质上就是三步：**取key的 hashCode 值、根据 hashcode 计算出hash值、通过取模计算下标**。其中，JDK1.7和1.8的不同之处，就在于第二步。我们来看下详细过程，以JDK1.8为例，n为table的长度：

![1641384317582](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/200519-906066.png)

### JDK1.8 为什么要 hashcode 异或其右移十六位的值？

因为在JDK 1.7 中扰动了 9 次（其中由有4次位运算和5次异或运算），计算 hash 值的性能会稍差一点点。从速度、功效、质量来考虑，JDK1.8 优化了高位运算的算法，通过hashCode()的高16位异或低16位实现：(h = k.hashCode()) ^ (h >>> 16)让高位参与计算数组的索引，这样做到了将数据均匀的分布在数组当中，还可以减少计算。

**这么做可以在数组 table 的 length 比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销，还可以将元素均匀的分布在数组中**。

> 不管怎么优化，我们都是想要保证计算的下表最好都均匀的分布在数组当中，而不是发生冲突。

### 为什么 hash 值要与length-1相与？

- 把 hash 值对数组长度取模运算，模运算的消耗很大，没有位运算快。

- 当 length 总是 2 的n次方时，h& (length-1) 运算等价于对length取模，也就是 h%length，但是 & 比 % 具有更高的效率。

- 这也就是解释了为什么数组的大小总是要是2的幂次，这样取模运算就和与运算等价了。

  ## 补充数组容量计算  		

HashMap 构造函数允许用户传入的容量不是  2  的  n  次方，因为它可以自动地将传入的容量转换为  2  的  n 次方。会取大于或等于这个数的 且最近的2次幂作为 table 数组的初始容量，使用`tableSizeFor(int)`方法，如 tableSizeFor(10) = 16（2 的 4 次幂），tableSizeFor(20) = 32（2 的 5 次幂），也就是说 table 数组的长度总是 2 的次幂。JDK1.8 源码如下：

首先我们需要知道HashMap是通过一个名为`tableSizeFor`的方法来确保HashMap数组长度永远为2的幂次方的，源码如下：

~~~java
/*找到大于或等于 cap 的最小2的幂，用来做容量阈值*/
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
~~~

**tableSizeFor的功能（不考虑大于最大容量的情况）是返回大于等于输入参数且最近的2的整数次幂的数**。比如10，则返回16。

该算法让最高位的1后面的位全变为1。最后再让结果n+1，即得到了2的整数次幂的值了。

让`cap-1`再赋值给n的目的是另找到的目标值大于或等于原值。例如二进制1000，十进制数值为8。如果不对它减1而直接操作，将得到答案10000，即16。显然不是结果。减1后二进制为111，再进行操作则会得到原来的数值1000，即8。通过一系列位运算大大提高效率。

## 那么为什么要把数组长度设计为2的幂次方呢？

我个人觉得这样设计有以下几个好处：

> 1、当数组长度为2的幂次方时，可以使用位运算来计算元素在数组中的下标

HashMap是通过`index=hash&(table.length-1)`这条公式来计算元素在table数组中存放的下标，就是把元素的hash值和数组长度减1的值做一个与运算，即可求出该元素在数组中的下标，这条公式其实等价于`hash%length`，也就是对数组长度求模取余，只不过**只有当数组长度为2的幂次方时，hash&(length-1)才等价于hash%length**，**使用位运算可以提高效率**。

> 2、 增加hash值的随机性，减少hash冲突

如果 length 为 2 的幂次方，则 length-1 转化为二进制必定是 11111……的形式，这样的话可以使所有位置都能和元素hash值做与运算，如果是如果 length 不是2的次幂，比如length为15，则length-1为14，对应的二进制为1110，在和hash 做与运算时，最后一位永远都为0 ，浪费空间。

> 3、在做新的元素映射的时候，不需要在一一的计算hash值映射到新的数组上面，“因为我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。

## table数组长度永远为2的幂次方

总所周知，HashMap数组长度永远为2的幂次方(**指的是table数组的大小**)，那你有想过为什么吗？

这样做效果上等同于取模，在速度、效率上比直接取模要快得多。

除此之外，2 的 N 次幂有助于减少碰撞的几率。如果 length 为2的幂次方，则 length-1 转化为二进制必定是11111……的形式，在与h的二进制与操作效率会非常的快，而且空间不浪费。我们来举个例子，看下图：

![1641384570302](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/200931-467462.png)

当n=15时，6 和 7 的结果一样，这样表示他们在 table 存储的位置是相同的，也就是产生了碰撞，6、7就会在一个位置形成链表，4和5的结果也是一样，这样就会导致查询速度降低。

如果我们进一步分析，还会发现空间浪费非常大，以 length=15 为例，在 1、3、5、7、9、11、13、15 这八处没有存放数据。因为hash值在与14（即 1110）进行&运算时，得到的结果最后一位永远都是0，即 0001、0011、0101、0111、1001、1011、1101、1111位置处是不可能存储数据的。

## 数组元素 & 链表节点的 实现类

-  `HashMap`中的数组元素 & 链表节点  采用 `Node`类 实现

> 与 `JDK 1.7` 的对比（`Entry`类），仅仅只是换了名字

~~~java
/** 
  * Node  = HashMap的内部类，实现了Map.Entry接口，本质是 = 一个映射(键值对)
  * 实现了getKey()、getValue()、equals(Object o)和hashCode()等方法
  **/  

  static class Node<K,V> implements Map.Entry<K,V> {

        final int hash; // 哈希值，HashMap根据该值确定记录的位置
        final K key; // key
        V value; // value
        Node<K,V> next;// 链表下一个节点

        // 构造方法
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        public final K getKey()        { return key; }   // 返回 与 此项 对应的键
        public final V getValue()      { return value; } // 返回 与 此项 对应的值
        public final String toString() { return key + "=" + value; }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

      /** 
        * hashCode（） 
        */
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

      /** 
        * equals（）
        * 作用：判断2个Entry是否相等，必须key和value都相等，才返回true  
        */
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
~~~

## 红黑树节点 实现类

-  `HashMap`中的红黑树节点 采用 `TreeNode` 类 实现

~~~java
 /**
  * 红黑树节点 实现类：继承自LinkedHashMap.Entry<K,V>类
  */
  static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {  

    // 属性 = 父节点、左子树、右子树、删除辅助节点 + 颜色
    TreeNode<K,V> parent;  
    TreeNode<K,V> left;   
    TreeNode<K,V> right;
    TreeNode<K,V> prev;   
    boolean red;   

    // 构造函数
    TreeNode(int hash, K key, V val, Node<K,V> next) {  
        super(hash, key, val, next);  
    }  
  
    // 返回当前节点的根节点  
    final TreeNode<K,V> root() {  
        for (TreeNode<K,V> r = this, p;;) {  
            if ((p = r.parent) == null)  
                return r;  
            r = p;  
        }  
    } 

~~~

## 主要使用API（方法、函数）

> 与 `JDK 1.7` 基本相同

~~~java
V get(Object key); // 获得指定键的值
V put(K key, V value);  // 添加键值对
void putAll(Map<? extends K, ? extends V> m);  // 将指定Map中的键值对 复制到 此Map中
V remove(Object key);  // 删除该键值对

boolean containsKey(Object key); // 判断是否存在该键的键值对；是 则返回true
boolean containsValue(Object value);  // 判断是否存在该值的键值对；是 则返回true
 
Set<K> keySet();  // 单独抽取key序列，将所有key生成一个Set
Collection<V> values();  // 单独value序列，将所有value生成一个Collection

void clear(); // 清除哈希表中的所有键值对
int size();  // 返回哈希表中所有 键值对的数量 = 数组中的键值对 + 链表中的键值对
boolean isEmpty(); // 判断HashMap是否为空；size == 0时 表示为 空 
~~~

## 重要参数

 `HashMap`中的主要参数 同  `JDK 1.7` ，即：容量、加载因子、扩容阈值

但由于数据结构中引入了 红黑树，故加入了 **与红黑树相关的参数**。具体介绍如下：

~~~java
 /** 
   * 主要参数 同  JDK 1.7 
   * 即：容量、加载因子、扩容阈值（要求、范围均相同）
   */

  // 1. 容量（capacity）： 必须是2的幂 & <最大容量（2的30次方）
  static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 默认容量 = 16 = 1<<4 = 00001中的1向左移4位 = 10000 = 十进制的2^4=16
  static final int MAXIMUM_CAPACITY = 1 << 30; // 最大容量 =  2的30次方（若传入的容量过大，将被最大值替换）

  // 2. 加载因子(Load factor)：HashMap在其容量自动增加前可达到多满的一种尺度 
  final float loadFactor; // 实际加载因子
  static final float DEFAULT_LOAD_FACTOR = 0.75f; // 默认加载因子 = 0.75

  // 3. 扩容阈值（threshold）：当哈希表的大小 ≥ 扩容阈值时，就会扩容哈希表（即扩充HashMap的容量） 
  // a. 扩容 = 对哈希表进行resize操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数
  // b. 扩容阈值 = 容量 x 加载因子
  int threshold;

  // 4. 其他
  transient Node<K,V>[] table;  // 存储数据的Node类型 数组，长度 = 2的幂；数组的每个元素 = 1个单链表
  transient int size;// HashMap的大小，即 HashMap中存储的键值对的数量
 

  /** 
   * 与红黑树相关的参数
   */
   // 1. 桶的树化阈值：即 链表转成红黑树的阈值，在存储数据时，当链表长度 > 该值时，则将链表转换成红黑树
   static final int TREEIFY_THRESHOLD = 8; 
   // 2. 桶的链表还原阈值：即 红黑树转为链表的阈值，当在扩容（resize（））时（此时HashMap的数据存储位置会重新计算），在重新计算存储位置后，当原有的红黑树内数量 < 6时，则将 红黑树转换成链表
   static final int UNTREEIFY_THRESHOLD = 6;
   // 3. 最小树形化容量阈值：即 当哈希表中的容量 > 该值时，才允许树形化链表 （即 将链表 转换成红黑树）
   // 否则，若桶内元素太多时，则直接扩容，而不是树形化
   // 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
   static final int MIN_TREEIFY_CAPACITY = 64;
~~~

table数组里面存放的是Node对象，Node是HashMap的一个内部类，用来表示一个key-value，源码如下：

~~~java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);//^表示相同返回0，不同返回1
        //Objects.hashCode(o)————>return o != null ? o.hashCode() : 0;
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            //Objects.equals(1,b)————> return (a == b) || (a != null && a.equals(b));
            if (Objects.equals(key, e.getKey()) && Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
~~~

总结

- 默认初始容量为`16`，默认负载因子为`0.75`
- `threshold = 数组长度 * loadFactor`，当元素个数大于等于`threshold(容量阈值)`时，HashMap会进行扩容操作
- table数组中存放指向链表的引用
- 在创建一个节点的时候，会直接将节点插入链表中。

这里需要注意的一点是**table数组并不是在构造方法里面初始化的，它是在resize(扩容)方法里进行初始化的**。

> 这里说句题外话：可能有刁钻的面试官会问**为什么默认初始容量要设置为16？为什么负载因子要设置为0.75？**
>
> 我们都知道HashMap数组长度被设计成2的幂次方(下面会讲)，那为什么初始容量不设计成4、8或者32.... 其实这是JDK设计者经过权衡之后得出的一个比较合理的数字，如果默认容量是8的话，当添加到第6个元素的时候就会触发扩容操作，扩容操作是非常消耗CPU的，32的话如果只添加少量元素则会浪费内存，因此设计成16是比较合适的，负载因子也是同理。

## 默认加载因子是多少？为什么是 0.75，不是 0.6 或者 0.8

回答这个问题前，我们来先看下HashMap的默认构造函数：

![1641384100207](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/200147-526347.png)

Node[] table的初始化长度length(默认值是16)，Load factor为负载因子(默认值是0.75)，threshold是HashMap所能容纳键值对的最大值。threshold = length * Load factor。也就是说，在数组定义好长度之后，负载因子越大，所能容纳的键值对个数越多。

默认的loadFactor是0.75，0.75是对**空间和时间效率**的一个平衡选择，一般不要修改，除非在时间和空间比较特殊的情况下 ：

- 如果内存空间很多而又对时间效率要求很高，可以降低负载因子Load factor的值 。
- 相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1。

我们来追溯下作者在源码中的注释（JDK1.7）：

![1641384129417](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/200210-125881.png)

翻译过来大概的意思是：作为一般规则，默认负载因子（0.75）在时间和空间成本上提供了很好的折衷。较高的值会降低空间开销，但提高查找成本（体现在大多数的HashMap类的操作，包括get和put）。设置初始大小时，应该考虑预计的entry数在map及其负载系数，并且尽量减少rehash操作的次数。如果初始容量大于最大条目数除以负载因子，rehash操作将不会发生。


![20220106105335](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20220106105335.png)

## 总结 数据结构 & 参数方面与 `JDK 1.7`的区别

![1641433947623](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/095230-531639.png)

## 源码分析

本次的源码分析主要是根据 **使用步骤** 进行相关函数的详细分析

![1641434023465](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/095354-2684.png)

### 声明1个HashMap的对象

此处主要分析的构造函数

~~~java
/**
  * 函数使用原型
  */
  Map<String,Integer> map = new HashMap<String,Integer>();

 /**
   * 源码分析：主要是HashMap的构造函数 = 4个
   * 仅贴出关于HashMap构造函数的源码
   */

public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable{

    // 省略上节阐述的参数
    
  /**
     * 构造函数1：默认构造函数（无参）
     * 加载因子 & 容量 = 默认 = 0.75、16
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
    }

    /**
     * 构造函数2：指定“容量大小”的构造函数
     * 加载因子 = 默认 = 0.75 、容量 = 指定大小
     */
    public HashMap(int initialCapacity) {
        // 实际上是调用指定“容量大小”和“加载因子”的构造函数
        // 只是在传入的加载因子参数 = 默认加载因子
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
        
    }

    /**
     * 构造函数3：指定“容量大小”和“加载因子”的构造函数
     * 加载因子 & 容量 = 自己指定
     */
    public HashMap(int initialCapacity, float loadFactor) {

        // 指定初始容量必须非负，否则报错  
         if (initialCapacity < 0)  
           throw new IllegalArgumentException("Illegal initial capacity: " +  
                                           initialCapacity); 

        // HashMap的最大容量只能是MAXIMUM_CAPACITY，哪怕传入的 > 最大容量
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;

        // 填充比必须为正  
        if (loadFactor <= 0 || Float.isNaN(loadFactor))  
            throw new IllegalArgumentException("Illegal load factor: " +  
                                           loadFactor);  
        // 设置 加载因子
        this.loadFactor = loadFactor;

        // 设置 扩容阈值
        // 注：此处不是真正的阈值，仅仅只是将传入的容量大小转化为：>传入容量大小的最小的2的幂，该阈值后面会重新计算
        // 下面会详细讲解 ->> 分析1
        this.threshold = tableSizeFor(initialCapacity); 

    }

    /**
     * 构造函数4：包含“子Map”的构造函数
     * 即 构造出来的HashMap包含传入Map的映射关系
     * 加载因子 & 容量 = 默认
     */

    public HashMap(Map<? extends K, ? extends V> m) {

        // 设置容量大小 & 加载因子 = 默认
        this.loadFactor = DEFAULT_LOAD_FACTOR; 

        // 将传入的子Map中的全部元素逐个添加到HashMap中
        putMapEntries(m, false); 
    }
}

   /**
     * 分析1：tableSizeFor(initialCapacity)
     * 作用：将传入的容量大小转化为：>传入容量大小的最小的2的幂
     * 与JDK 1.7对比：类似于JDK 1.7 中 inflateTable()里的 roundUpToPowerOf2(toSize)
     */
    static final int tableSizeFor(int cap) {
     int n = cap - 1;
     n |= n >>> 1;
     n |= n >>> 2;
     n |= n >>> 4;
     n |= n >>> 8;
     n |= n >>> 16;
     return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
~~~

注：（同JDK 1.7类似） 

1. 此处仅用于接收初始容量大小（`capacity`）、加载因子(`Load factor`)，但仍无真正初始化哈希表，即初始化存储数组`table` 
2. 此处先给出结论：**真正初始化哈希表（初始化存储数组table）是在第1次添加键值对时，即第1次调用put（）时。下面会详细说明**

### 步骤2：向HashMap添加数据（成对 放入 键 - 值对）

在该步骤中，与`JDK 1.7`的差别较大：

![1641434377567](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/105427-397102.png)

**添加数据的流程如下**

![1641434412289](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/100030-53701.png)

~~~java
  /**
     * 源码分析：主要分析HashMap的put函数
     */
    public V put(K key, V value) {
        // 1. 对传入数组的键Key计算Hash值 ->>分析1
        // 2. 再调用putVal（）添加数据进去 ->>分析2
        return putVal(hash(key), key, value, false, true);
    }

~~~

#### 分析1：hash（key）

~~~java
   /**
     * 分析1：hash(key)
     * 作用：计算传入数据的哈希码（哈希值、Hash值）
     * 该函数在JDK 1.7 和 1.8 中的实现不同，但原理一样 = 扰动函数 = 使得根据key生成的哈希码（hash值）分布更加均匀、更具备随机性，避免出现hash值冲突（即指不同key但生成同1个hash值）
     * JDK 1.7 做了9次扰动处理 = 4次位运算 + 5次异或运算
     * JDK 1.8 简化了扰动函数 = 只做了2次扰动 = 1次位运算 + 1次异或运算
     */

      // JDK 1.7实现：将 键key 转换成 哈希码（hash值）操作  = 使用hashCode() + 4次位运算 + 5次异或运算（9次扰动）
      static final int hash(int h) {
        h ^= k.hashCode(); 
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
     }

      // JDK 1.8实现：将 键key 转换成 哈希码（hash值）操作 = 使用hashCode() + 1次位运算 + 1次异或运算（2次扰动）
      // 1. 取hashCode值： h = key.hashCode() 
      // 2. 高位参与低位的运算：h ^ (h >>> 16)  
      static final int hash(Object key) {
           int h;
            return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
            // a. 当key = null时，hash值 = 0，所以HashMap的key 可为null      
            // 注：对比HashTable，HashTable对key直接hashCode（），若key为null时，会抛出异常，所以HashTable的key不可为null
            // b. 当key ≠ null时，则通过先计算出 key的 hashCode()（记为h），然后 对哈希码进行 扰动处理： 按位 异或（^） 哈希码自身右移16位后的二进制
     }

   /**
     * 计算存储位置的函数分析：indexFor(hash, table.length)
     * 注：该函数仅存在于JDK 1.7 ，JDK 1.8中实际上无该函数（直接用1条语句判断写出），但原理相同
     * 为了方便讲解，故提前到此讲解
     */
     static int indexFor(int h, int length) {  
          return h & (length-1); 
          // 将对哈希码扰动处理后的结果 与运算(&) （数组长度-1），最终得到存储在数组table的位置（即数组下标、索引）
          }

~~~

- 总结 计算存放在数组 table 中的位置（即数组下标、索引）的过程

> 1. 此处与 `JDK 1.7`的区别在于：`hash`值的求解过程中 哈希码的二次处理方式（扰动处理）
> 2. 步骤1、2 =  `hash`值的求解过程

![1641434580297](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/100303-885831.png)

计算示意图

![1641434614301](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/100409-308658.png)

在了解 如何计算存放数组`table` 中的位置 后，所谓 **知其然 而 需知其所以然**，下面我将讲解为什么要这样计算，即主要解答以下3个问题：

1. 为什么不直接采用经过`hashCode（）`处理的哈希码 作为 存储数组`table`的下标位置？
2. 为什么采用 哈希码 **与运算(&)** （数组长度-1） 计算数组下标？
3. 为什么在计算数组下标前，需对哈希码进行二次处理：扰动处理？

在回答这3个问题前，请大家记住一个核心思想：

> **所有处理的根本目的，都是为了提高 存储key-value的数组下标位置 的随机性 & 分布均匀性，尽量避免出现hash值冲突**。即：对于不同`key`，存储的数组下标位置要尽可能不一样

#### 问题1：为什么不直接采用经过hashCode（）处理的哈希码 作为 存储数组table的下标位置？

- 结论：容易出现 哈希码 与 数组大小范围不匹配的情况，即 计算出来的哈希码可能 不在数组大小范围内，从而导致无法匹配存储位置
- 原因描述

![1641434728714](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/100532-854300.png)

为了解决 “哈希码与数组大小范围不匹配” 的问题，`HashMap`给出了解决方案：**哈希码 与运算（&） （数组长度-1）**，即问题3

#### 问题2：为什么采用 哈希码 与运算(&) （数组长度-1） 计算数组下标？

- 结论：根据HashMap的容量大小（数组长度），按需取 哈希码一定数量的低位 作为存储的数组下标位置，从而 解决 “哈希码与数组大小范围不匹配” 的问题
- 具体解决方案描述

![1641434839209](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/100722-289352.png)

#### 问题3：为什么在计算数组下标前，需对哈希码进行二次处理：扰动处理？

- 结论：加大哈希码低位的随机性，使得分布更均匀，从而提高对应数组存储下标位置的随机性 & 均匀性，最终减少Hash冲突
- 具体描述

![1641434878234](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/100807-612875.png)

至此，关于怎么计算 `key-value` 值存储在`HashMap`数组位置 & 为什么要这么计算，讲解完毕。

#### 分析2：putVal(hash(key), key, value, false, true);

此处有2个主要讲解点：

- 计算完存储位置后，具体该如何 存放数据 到哈希表中
- 具体如何扩容，即 **扩容机制**

##### 主要讲解点1：计算完存储位置后，具体该如何存放数据到哈希表中

由于数据结构中加入了红黑树，所以在存放数据到哈希表中时，需进行多次数据结构的判断：**数组、红黑树、链表**

> 与 `JDK 1.7`的区别： `JDK 1.7`只需判断 数组 & 链表

![1641434991782](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/100954-957554.png)

~~~java
   /**
     * 分析2：putVal(hash(key), key, value, false, true)
     */
     final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {

            Node<K,V>[] tab; Node<K,V> p; int n, i;

        // 1. 若哈希表的数组tab为空，则 通过resize() 创建
        // 所以，初始化哈希表的时机 = 第1次调用put函数时，即调用resize() 初始化创建
        // 关于resize（）的源码分析将在下面讲解扩容时详细分析，此处先跳过
        if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;

        // 2. 计算插入存储的数组索引i：根据键值key计算的hash值 得到
        // 此处的数组下标计算方式 = i = (n - 1) & hash，同JDK 1.7中的indexFor（），上面已详细描述

        // 3. 插入时，需判断是否存在Hash冲突：
        // 若不存在（即当前table[i] == null），则直接在该数组位置新建节点，插入完毕
        // 否则，代表存在Hash冲突，即当前存储位置已存在节点，则依次往下判断：a. 当前位置的key是否与需插入的key相同、b. 判断需插入的数据结构是否为红黑树 or 链表
        if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);  // newNode(hash, key, value, null)的源码 = new Node<>(hash, key, value, next)

    else {
        Node<K,V> e; K k;

        // a. 判断 table[i]的元素的key是否与 需插入的key一样，若相同则 直接用新value 覆盖 旧value
        // 判断原则：equals（）
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;

        // b. 继续判断：需插入的数据结构是否为红黑树 or 链表
        // 若是红黑树，则直接在树中插入 or 更新键值对
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value); ->>分析3

        // 若是链表,则在链表中插入 or 更新键值对
        // i.  遍历table[i]，判断Key是否已存在：采用equals（） 对比当前遍历节点的key 与 需插入数据的key：若已存在，则直接用新value 覆盖 旧value
        // ii. 遍历完毕后仍无发现上述情况，则直接在链表尾部插入数据
        // 注：新增节点后，需判断链表长度是否>8（8 = 桶的树化阈值）：若是，则把链表转换为红黑树
        
        else {
            for (int binCount = 0; ; ++binCount) {
                // 对于ii：若数组的下1个位置，表示已到表尾也没有找到key值相同节点，则新建节点 = 插入节点
                // 注：此处是从链表尾插入，与JDK 1.7不同（从链表头插入，即永远都是添加到数组的位置，原来数组位置的数据则往后移）
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);

                    // 插入节点后，若链表节点>数阈值，则将链表转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) 
                        treeifyBin(tab, hash); // 树化操作
                    break;
                }

                // 对于i
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;

                // 更新p指向下一个节点，继续遍历
                p = e;
            }
        }

        // 对i情况的后续操作：发现key已存在，直接用新value 覆盖 旧value & 返回旧value
        if (e != null) { 
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e); // 替换旧值时会调用的方法（默认实现为空）
            return oldValue;
        }
    }

    ++modCount;

    // 插入成功后，判断实际存在的键值对数量size > 最大容量threshold
    // 若 > ，则进行扩容 ->>分析4（但单独讲解，请直接跳出该代码块）
    if (++size > threshold)
        resize();

    afterNodeInsertion(evict);// 插入成功时会调用的方法（默认实现为空）
    return null;

}

    /**
     * 分析3：putTreeVal(this, tab, hash, key, value)
     * 作用：向红黑树插入 or 更新数据（键值对）
     * 过程：遍历红黑树判断该节点的key是否与需插入的key 相同：
     *      a. 若相同，则新value覆盖旧value
     *      b. 若不相同，则插入
     */

     final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                       int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            TreeNode<K,V> root = (parent != null) ? root() : this;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph; K pk;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.find(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.find(h, k, kc)) != null))
                            return q;
                    }
                    dir = tieBreakOrder(k, pk);
                }

                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    Node<K,V> xpn = xp.next;
                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    xp.next = x;
                    x.parent = x.prev = xp;
                    if (xpn != null)
                        ((TreeNode<K,V>)xpn).prev = x;
                    moveRootToFront(tab, balanceInsertion(root, x));
                    return null;
                }
            }
        }

~~~

**小结**

![1641435074606](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/101133-214644.png)

![1641435097248](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/105511-977879.png)

![1641435124279](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/105513-643667.png)

##### 主要讲解点2：扩容机制（即 resize（）函数方法）

- 扩容流程如下

![1641435217653](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/101341-424259.png)

![1641435238090](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1641435238090.png)

**源码**

~~~java
   /**
     * 分析4：resize（）
     * 该函数有2种使用情况：1.初始化哈希表 2.当前数组容量过小，需扩容
     */
   final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table; // 扩容前的数组（当前数组）
    int oldCap = (oldTab == null) ? 0 : oldTab.length; // 扩容前的数组的容量 = 长度
    int oldThr = threshold;// 扩容前的数组的阈值
    int newCap, newThr = 0;

    // 针对情况2：若扩容前的数组容量超过最大值，则不再扩充
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }

        // 针对情况2：若无超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // 通过右移扩充2倍
    }

    // 针对情况1：初始化哈希表（采用指定 or 默认值）
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;

    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }

    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }

    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;

    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;

                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);

                else { // 链表优化重hash的代码块
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引 + oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}

~~~

扩容流程（含 与 `JDK 1.7` 的对比）

![1641435411483](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/101653-407104.png)

![1641435382962](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/101625-658960.png)

##### 此处主要讲解： `JDK 1.8`扩容时，数据存储位置重新计算的方式

- 计算结论 & 原因解析

![1641435453445](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/101804-108816.png)

**结论示意图**

![1641435497494](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/101843-268788.png)

- `JDK 1.8`根据此结论作出的新元素存储位置计算规则 非常简单，提高了扩容效率，具体如下图

> 这与 `JDK 1.7`在计算新元素的存储位置有很大区别：`JDK 1.7`在扩容后，都需按照原来方法重新计算，即
>  `hashCode（）`->> 扰动处理 ->>`（h & length-1）`）

![1641435605220](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1641435605220.png)

![1641435586636](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/102009-810223.png)

与 `JDK 1.7`的区别

![1641435646194](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/102105-671594.png)

### 步骤3：从HashMap中获取数据

- 假如理解了上述`put（）`函数的原理，那么`get（）`函数非常好理解，因为二者的过程原理几乎相同
-  `get（）`函数的流程如下：

![1641435761386](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/102244-747841.png)

~~~java
/**
   * 函数原型
   * 作用：根据键key，向HashMap获取对应的值
   */ 
   map.get(key)；


 /**
   * 源码分析
   */ 
   public V get(Object key) {
    Node<K,V> e;
    // 1. 计算需获取数据的hash值
    // 2. 通过getNode（）获取所查询的数据 ->>分析1
    // 3. 获取后，判断数据是否为空
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

/**
   * 分析1：getNode(hash(key), key))
   */ 
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;

    // 1. 计算存放在数组table中的位置
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {

        // 4. 通过该函数，依次在数组、红黑树、链表中查找（通过equals（）判断）
        // a. 先在数组中找，若存在，则直接返回
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;

        // b. 若数组中没有，则到红黑树中寻找
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);

            // c. 若红黑树中也没有，则通过遍历，到链表中寻找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

~~~

### 步骤4：对HashMap的其他操作

> 即 对其余使用`API`（函数、方法）的源码分析

-  `HashMap`除了核心的`put（）`、`get（）`函数，还有以下主要使用的函数方法

~~~java

void clear(); // 清除哈希表中的所有键值对
int size();  // 返回哈希表中所有 键值对的数量 = 数组中的键值对 + 链表中的键值对
boolean isEmpty(); // 判断HashMap是否为空；size == 0时 表示为 空 

void putAll(Map<? extends K, ? extends V> m);  // 将指定Map中的键值对 复制到 此Map中
V remove(Object key);  // 删除该键值对

boolean containsKey(Object key); // 判断是否存在该键的键值对；是 则返回true
boolean containsValue(Object value);  // 判断是否存在该值的键值对；是 则返回true
 

~~~

关于上述方法的源码的原理 同 `JDK 1.7`，此处不作过多描述.

### 源码总结

下面，用3个图总结整个源码内容：

> 总结内容 = 数据结构、主要参数、添加 & 查询数据流程、扩容机制

- 数据结构 & 主要参数

![1641435913113](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/102516-246278.png)

### 与 `JDK 1.7` 的区别

`HashMap` 的实现在 `JDK 1.7` 和 `JDK 1.8` 差别较大，具体区别如下

> 1.  `JDK 1.8` 的优化目的主要是：减少 `Hash`冲突 & 提高哈希表的存、取效率

#### 数据结构

![1641436001171](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/102651-522687.png)

#### 获取数据时（获取数据 类似）

![1641436044016](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/102753-341226.png)

#### 扩容机制

![1641436087880](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/102833-930502.png)

#### 其他问题

![1641436121087](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1641436121087.png)

##### 解决哈希冲突

![1641436160791](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/102924-528441.png)

##### 为什么HashMap具备下述特点：键-值（key-value）都允许为空、线程不安全、不保证有序、存储位置随时间变化

![1641436211933](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1641436211933.png)

- 下面主要讲解 `HashMap` 线程不安全的其中一个重要原因：多线程下容易出现`resize（）`死循环
   **本质 = 并发 执行 put（）操作导致触发 扩容行为，从而导致 环形链表，使得在获取数据遍历链表时形成死循环，即Infinite Loop**
- 先看扩容的源码分析`resize（）`

> 关于resize（）的源码分析已在上文详细分析，此处仅作重点分析：transfer（）

~~~java
/**
   * 源码分析：resize(2 * table.length)
   * 作用：当容量不足时（容量 > 阈值），则扩容（扩到2倍）
   */ 
   void resize(int newCapacity) {  
    
    // 1. 保存旧数组（old table） 
    Entry[] oldTable = table;  

    // 2. 保存旧容量（old capacity ），即数组长度
    int oldCapacity = oldTable.length; 

    // 3. 若旧容量已经是系统默认最大容量了，那么将阈值设置成整型的最大值，退出    
    if (oldCapacity == MAXIMUM_CAPACITY) {  
        threshold = Integer.MAX_VALUE;  
        return;  
    }  
  
    // 4. 根据新容量（2倍容量）新建1个数组，即新table  
    Entry[] newTable = new Entry[newCapacity];  

    // 5. （重点分析）将旧数组上的数据（键值对）转移到新table中，从而完成扩容 ->>分析1.1 
    transfer(newTable); 

    // 6. 新数组table引用到HashMap的table属性上
    table = newTable;  

    // 7. 重新设置阈值  
    threshold = (int)(newCapacity * loadFactor); 
} 

 /**
   * 分析1.1：transfer(newTable); 
   * 作用：将旧数组上的数据（键值对）转移到新table中，从而完成扩容
   * 过程：按旧链表的正序遍历链表、在新链表的头部依次插入
   */ 
void transfer(Entry[] newTable) {
      // 1. src引用了旧数组
      Entry[] src = table; 

      // 2. 获取新数组的大小 = 获取新容量大小                 
      int newCapacity = newTable.length;

      // 3. 通过遍历 旧数组，将旧数组上的数据（键值对）转移到新数组中
      for (int j = 0; j < src.length; j++) { 
          // 3.1 取得旧数组的每个元素  
          Entry<K,V> e = src[j];           
          if (e != null) {
              // 3.2 释放旧数组的对象引用（for循环后，旧数组不再引用任何对象）
              src[j] = null; 

              do { 
                  // 3.3 遍历 以该数组元素为首 的链表
                  // 注：转移链表时，因是单链表，故要保存下1个结点，否则转移后链表会断开
                  Entry<K,V> next = e.next; 
                 // 3.3 重新计算每个元素的存储位置
                 int i = indexFor(e.hash, newCapacity); 
                 // 3.4 将元素放在数组上：采用单链表的头插入方式 = 在链表头上存放数据 = 将数组位置的原有数据放在后1个指针、将需放入的数据放到数组位置中
                 // 即 扩容后，可能出现逆序：按旧链表的正序遍历链表、在新链表的头部依次插入
                 e.next = newTable[i]; 
                 newTable[i] = e;  
                 // 访问下1个Entry链上的元素，如此不断循环，直到遍历完该链表上的所有节点
                 e = next;             
             } while (e != null);
             // 如此不断循环，直到遍历完数组上的所有数据元素
         }
     }
 }
~~~

从上面可看出：在扩容`resize（）`过程中，在将旧数组上的数据 转移到 新数组上时，**转移数据操作 = 按旧链表的正序遍历链表、在新链表的头部依次插入**，即在转移数据、扩容后，容易出现**链表逆序的情况**

> 设重新计算存储位置后不变，即扩容前 = 1->2->3，扩容后 = 3->2->1

- 此时若（多线程）并发执行 `put（）`操作，一旦出现扩容情况，则 **容易出现 环形链表**，从而在获取数据、遍历链表时 形成死循环（`Infinite Loop`），即 死锁的状态，具体请看下图：

初始状态、步骤1、步骤2

![1641436388493](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/103311-407775.png)

![1641436367726](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/103337-47697.png)

![1641436422811](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1641436422811.png)

![1641436448532](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1641436448532.png)

![1641436466079](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/103439-575451.png)

注：由于 `JDK 1.8` 转移数据操作 = **按旧链表的正序遍历链表、在新链表的尾部依次插入**，所以不会出现链表 **逆序、倒置**的情况，故不容易出现环形链表的情况。

> 但 `JDK 1.8` 还是线程不安全，因为 无加同步锁保护

##### 为什么 HashMap 中 String、Integer 这样的包装类适合作为 key 键

![1641436532852](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/103603-614352.png)

### HashMap 中的 `key`若 `Object`类型， 则需实现哪些方法？

![1641437222900](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1641437222900.png)

## 扩容

Hashmap 在容量超过负载因子所定义的容量之后，就会扩容。Java 里的数组是无法自动扩容的，方法是将 Hashmap 的大小扩大为原来数组的两倍，并将原来的对象放入新的数组中。

那扩容的具体步骤是什么？让我们看看源码。

### 先来看下 JDK1.7 的代码：

![1641384845762](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/092911-1994.png)

这里就是使用一个容量更大的数组来代替已有的容量小的数组，transfer()方法将原有Entry数组的元素拷贝到新的Entry数组里。

![1641384869586](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/201439-967951.png)

newTable[i] 的引用赋给了 e.next ，也就是使用了单链表的头插入方式，同一位置上新元素总会被放在链表的头部位置；这样先放在一个索引上的元素终会被放到 Entry 链的尾部(如果发生了 hash 冲突的话）。

### jdk1.8优化

JDK1.8做了两处优化：

1. resize 之后，元素的位置在原来的位置，或者原来的位置 +oldCap (原来哈希表的长度）。不需要像 JDK1.7 的实现那样重新计算hash ，只需要看看原来的 hash 值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引 + oldCap ”。这个设计非常的巧妙，省去了重新计算 hash 值的时间。

   如下图所示，n 为 table 的长度，图（a）表示扩容前的 key1 和 key2 两种 key 确定索引位置的示例，图（b）表示扩容后 key1 和key2 两种 key 确定索引位置的示例，其中 hash1 是 key1 对应的哈希与高位运算结果。

![1641384957145](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/201558-181657.png)

**元素在重新计算 hash 之后，因为 n 变为 2倍，那么 n-1 的 mask 范围在高位多 1 bit(红色)**，因此新的index就会发生这样的变化：

![1641385012636](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/201653-736487.png)

2. JDK1.7 中 rehash 的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置（头插法）。JDK1.8 不会倒置，使用尾插法。

下图为 16扩充为 32 的 resize 示意图：

![1641385049202](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1641385049202.png)

HashMap每次扩容都是建立一个新的table数组，**长度和容量阈值都变为原来的两倍**，然后把原数组元素重新映射到新数组上，具体步骤如下：

1. 首先会判断table数组长度，如果大于0说明已被初始化过，那么`按当前table数组长度的2倍进行扩容，阈值也变为原来的2倍`
2. 若table数组未被初始化过，且threshold(阈值)大于0说明调用了`HashMap(initialCapacity, loadFactor)`构造方法，那么就把数组大小设为threshold
3. 若table数组未被初始化，且threshold为0说明调用`HashMap()`构造方法，那么就把数组大小设为`16`，threshold设为`16*0.75`
4. 接着需要判断如果不是第一次初始化，那么扩容之后，要重新计算键值对的位置，并把它们移动到合适的位置上去，如果节点是红黑树类型的话则需要进行红黑树的拆分。

~~~java
/*扩容*/
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;

    //1、若oldCap>0 说明hash数组table已被初始化
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }//按当前table数组长度的2倍进行扩容，阈值也变为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1;
    }//2、若数组未被初始化，而threshold>0说明调用了HashMap(initialCapacity)和HashMap(initialCapacity, loadFactor)构造器
    else if (oldThr > 0)
        newCap = oldThr;//新容量设为数组阈值
    else { //3、若table数组未被初始化，且threshold为0说明调用HashMap()构造方法
        newCap = DEFAULT_INITIAL_CAPACITY;//默认为16
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);//16*0.75
    }

    //若计算过程中，阈值溢出归零，则按阈值公式重新计算
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    //创建新的hash数组，hash数组的初始化也是在这里完成的
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    //如果旧的hash数组不为空，则遍历旧数组并映射到新的hash数组
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;//GC
                if (e.next == null)//如果只链接一个节点，重新计算并放入新数组
                    newTab[e.hash & (newCap - 1)] = e;
                //若是红黑树，则需要进行拆分
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    //rehash————>重新映射到新数组
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        /*注意这里使用的是：e.hash & oldCap，若为0则索引位置不变，不为0则新索引=原索引+旧数组长度*/
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
~~~

在扩容方法里面还涉及到有关红黑树的几个知识点：

### 链表树化

指的就是把链表转换成红黑树，树化需要满足以下两个条件：

- **链表长度大于等于8**
- **table数组长度大于等于64**

为什么table数组容量大于等于64才树化？

因为当table数组容量比较小时，键值对节点 hash 的碰撞率可能会比较高，进而导致链表长度较长。这个时候应该优先扩容，而不是立马树化。

> 还知道哪些hash算法？

Hash函数是指把一个大范围映射到一个小范围，目的往往是为了节省空间，使得数据容易保存。比较出名的有MurmurHash、MD4、MD5等等。

> key 可以为 Null 吗?

可以，key 为 Null 的时候，hash算法最后的值以0来计算，也就是放在数组的第一个位置。

> 一般用什么作为HashMap的key?

一般用Integer、String 这种不可变类当 HashMap 当 key，而且 String 最为常用。

- 因为字符串是不可变的，所以在它创建的时候 hashcode 就被缓存了，不需要重新计算。这就是 HashMap 中的键往往都使用字符串的原因。
- 因为获取对象的时候要用到 equals() 和 hashCode() 方法，那么键对象正确的重写这两个方法是非常重要的,这些类已经很规范的重写了 hashCode() 以及 equals() 方法。

> 用可变类当 HashMap 的 key 有什么问题?

hashcode 可能发生改变，导致 put 进去的值，无法 get 出。如下所示：

![1641385213459](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/202019-137096.png)

## 查找

在看源码之前先来简单梳理一下查找流程：

1. 首先通过自定义的hash方法计算出key的hash值，求出在数组中的位置
2. 判断该位置上是否有节点，若没有则返回null，代表查询不到指定的元素
3. 若有则判断该节点是不是要查找的元素，若是则返回该节点
4. 若不是则判断节点的类型，如果是红黑树的话，则调用红黑树的方法去查找元素
5. 如果是链表类型，则遍历链表调用equals方法去查找元素

HashMap的查找是非常快的，要查找一个元素首先得知道key的hash值，在HashMap中并不是直接通过key的hashcode方法获取哈希值，而是通过内部自定义的`hash`方法计算哈希值，我们来看看其实现：

~~~java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
~~~

`(h = key.hashCode()) ^ (h >>> 16)` 是为了让高位数据与低位数据进行异或，变相的让高位数据参与到计算中，int有 32 位，右移16位就能让低16位和高16位进行异或，也是为了增加hash值的随机性。

知道如何计算hash值后我们来看看`get`方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;//hash(key)不等于key.hashCode
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; //指向hash数组
    Node<K,V> first, e; //first指向hash数组链接的第一个节点，e指向下一个节点
    int n;//hash数组长度
    K k;
    /*(n - 1) & hash ————>根据hash值计算出在数组中的索引index（相当于对数组长度取模，这里用位运算进行了优化）*/
    if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
        //基本类型用==比较，其它用equals比较
        if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            //如果first是TreeNode类型，则调用红黑树查找方法
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {//向后遍历
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

这里要注意的一点就是在HashMap中用 `(n - 1) & hash` 计算key所对应的索引index（相当于对数组长度取模，这里用位运算进行了优化），这点在上面已经说过了，就不再废话了。

## 插入

我们先来看看插入元素的步骤：

1. 首先根据 key 的值计算 hash 值，找到该元素在数组中存储的下标；
2. 如果数组是空的，则调用 resize 进行初始化；
3. 如果没有哈希冲突直接放在对应的数组下标里；
4. 如果冲突了，且 key 已经存在，就覆盖掉 value；
5. 如果冲突后，发现该节点是红黑树，就将这个节点挂在树上；
6. 如果冲突后是链表，判断该链表是否大于 8 ，如果大于 8 并且数组容量小于 64，就进行扩容；
7. 如果链表长度大于 8 并且数组的容量大于等于 64，则将这个结构转换为红黑树；
8. 否则，链表插入键值对，若 key 存在，就覆盖掉 value。

**过程**

![1641384684667](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/201125-370513.png)

![1641433396360](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/06/094319-727986.png)

先看完上面的流程，再来看源码会简单很多，源码如下：

~~~java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab;//指向hash数组
    Node<K,V> p;//初始化为table中第一个节点
    int n, i;//n为数组长度，i为索引

    //tab被延迟到插入新数据时再进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //如果数组中不包含Node引用，则新建Node节点存入数组中即可
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);//new Node<>(hash, key, value, next)
    else {
        Node<K,V> e; //如果要插入的key-value已存在，用e指向该节点
        K k;
        //如果第一个节点就是要插入的key-value，则让e指向第一个节点（p在这里指向第一个节点）
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //如果p是TreeNode类型，则调用红黑树的插入操作（注意：TreeNode是Node的子类）
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //对链表进行遍历，并用binCount统计链表长度
            for (int binCount = 0; ; ++binCount) {
                //如果链表中不包含要插入的key-value，则将其插入到链表尾部
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //如果链表长度大于或等于树化阈值，则进行树化操作
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash);
                    break;
                }
                //如果要插入的key-value已存在则终止遍历，否则向后遍历
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //如果e不为null说明要插入的key-value已存在
        if (e != null) {
            V oldValue = e.value;
            //根据传入的onlyIfAbsent判断是否要更新旧值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //键值对数量大于等于阈值时，则进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);//也是空函数？回调？不知道干嘛的
    return null;
}
~~~

从源码也可以看出**table数组是在第一次调用put方法后才进行初始化的**。这里还有一个知识点就是在**JDK1.8版本HashMap是在链表尾部插入元素的，而在1.7版本里是插入链表头部的**，1.7版本这么设计的原因可能是作者认为新插入的元素使用到的频率会比较高，插入头部的话可以减少遍历次数。

那为什么1.8改成尾插法了呢？主要是因为头插法在多线程环境下可能会导致两个节点互相引用，形成死循环，由于此文主要讲解1.8版本，感兴趣的小伙伴可以去看看1.7版本的源码。

### JDK1.7 和1.8 的put方法区别是什么？

区别在两处：

解决哈希冲突时，JDK1.7 只使用链表，JDK1.8 使用链表+红黑树，当满足一定条件，链表会转换为红黑树。

链表插入元素时，JDK1.7 使用头插法插入元素，在多线程的环境下有可能导致环形链表的出现，扩容的时候会导致死循环。因此，JDK1.8使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，就不会出现链表成环的问题了，但JDK1.8 的 HashMap 仍然是线程不安全的，

## 删除

HashMap的删除操作并不复杂，仅需三个步骤即可完成。

1. 定位桶位置
2. 遍历链表找到相等的节点
3. 第三步删除节点

~~~java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value,boolean matchValue, boolean movable) {
    Node<K,V>[] tab;
    Node<K,V> p;
    int n, index;
    //1、定位元素桶位置
    if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e;
        K k;
        V v;
        // 如果键的值与链表第一个节点相等，则将 node 指向该节点
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            // 如果是 TreeNode 类型，调用红黑树的查找逻辑定位待删除节点
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                // 2、遍历链表，找到待删除节点
                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 3、删除节点，并修复链表或红黑树
        if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
~~~

注意：删除节点后可能破坏了红黑树的平衡性质，removeTreeNode方法会对红黑树进行变色、旋转等操作来保持红黑树的平衡结构，这部分比较复杂.

## 遍历

在工作中HashMap的遍历操作也是非常常用的，也许有很多小伙伴喜欢用for-each来遍历，但是你知道其中有哪些坑吗？

看下面的例子，当我们在遍历HashMap的时候，若使用remove方法删除元素时会抛出ConcurrentModificationException异常

~~~java
 Map<String, Integer> map = new HashMap<>();
        map.put("1", 1);
        map.put("2", 2);
        map.put("3", 3);
        for (String s : map.keySet()) {
            if (s.equals("2"))
                map.remove("2");
        }
~~~

这就是常说的fail-fast(快速失败)机制，这个就需要从一个变量说起

~~~java
transient int modCount;
~~~

在HashMap中有一个名为modCount的变量，它用来表示集合被修改的次数，修改指的是插入元素或删除元素，可以回去看看上面插入删除的源码，在最后都会对modCount进行自增。

当我们在遍历HashMap时，每次遍历下一个元素前都会对modCount进行判断，若和原来的不一致说明集合结果被修改过了，然后就会抛出异常，这是Java集合的一个特性，我们这里以keySet为例，看看部分相关源码：

~~~java
public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}

final class KeySet extends AbstractSet<K> {
    public final Iterator<K> iterator()     { return new KeyIterator(); }
    // 省略部分代码
}

final class KeyIterator extends HashIterator implements Iterator<K> {
    public final K next() { return nextNode().key; }
}

/*HashMap迭代器基类，子类有KeyIterator、ValueIterator等*/
abstract class HashIterator {
    Node<K,V> next;        //下一个节点
    Node<K,V> current;     //当前节点
    int expectedModCount;  //修改次数
    int index;             //当前索引
    //无参构造
    HashIterator() {
        expectedModCount = modCount;
        Node<K,V>[] t = table;
        current = next = null;
        index = 0;
        //找到第一个不为空的桶的索引
        if (t != null && size > 0) {
            do {} while (index < t.length && (next = t[index++]) == null);
        }
    }
    //是否有下一个节点
    public final boolean hasNext() {
        return next != null;
    }
    //返回下一个节点
    final Node<K,V> nextNode() {
        Node<K,V>[] t;
        Node<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();//fail-fast
        if (e == null)
            throw new NoSuchElementException();
        //当前的链表遍历完了就开始遍历下一个链表
        if ((next = (current = e).next) == null && (t = table) != null) {
            do {} while (index < t.length && (next = t[index++]) == null);
        }
        return e;
    }
    //删除元素
    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        current = null;
        K key = p.key;
        removeNode(hash(key), key, null, false, false);//调用外部的removeNode
        expectedModCount = modCount;
    }
}
~~~

相关代码如下，可以看到若modCount被修改了则会抛出ConcurrentModificationException异常。

```java
if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
```

但是对expectedModCount的修改只有在迭代器的删除里面。

那么如何在遍历时删除元素呢？

我们可以看看迭代器自带的remove方法，其中最后两行代码如下：

```java
removeNode(hash(key), key, null, false, false);//调用外部的removeNode
expectedModCount = modCount;
```

意思就是会调用外部remove方法删除元素后，把modCount赋值给expectedModCount，这样的话两者一致就不会抛出异常了，所以我们应该这样写：

```java
Map<String, Integer> map = new HashMap<>();
        map.put("1", 1);
        map.put("2", 2);
        map.put("3", 3);
        Iterator<String> iterator = map.keySet().iterator();
        while (iterator.hasNext()){
            if (iterator.next().equals("2"))
                iterator.remove();
        }
```

这里还有一个知识点就是在遍历HashMap时，我们会发现**遍历的顺序和插入的顺序不一致**，这是为什么？

在HashIterator源码里面可以看出，它是先从桶数组中找到包含链表节点引用的桶。然后对这个桶指向的链表进行遍历。遍历完成后，再继续寻找下一个包含链表节点引用的桶，找到继续遍历。找不到，则结束遍历。这就解释了为什么遍历和插入的顺序不一致，不懂的同学请看下图：

![1641383611751](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/195344-769273.png)

## equasl和hashcode

我在面试中就被问到过HashMap的key有什么限制吗？相信很多人都知道HashMap的key需要重写equals和hashcode方法。

**为什么HashMap的key需要重写equals()和hashcode()方法？**

简单看个例子，这里以Person为例：

~~~java
public class Person {
    Integer id;
    String name;

    public Person(Integer id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == null) return false;
        if (obj == this) return true;
        if (obj instanceof Person) {
            Person person = (Person) obj;
            if (this.id == person.id)
                return true;
        }
        return false;
    }

    public static void main(String[] args) {
        Person p1 = new Person(1, "aaa");
        Person p2 = new Person(1, "bbb");
        HashMap<Person, String> map = new HashMap<>();
        map.put(p1, "这是p1");
        System.out.println(map.get(p2));
    }
}
~~~

- 原生的equals方法是使用==来比较对象的
- 原生的hashCode值是根据内存地址换算出来的一个值

Person类重写equals方法来根据id判断是否相等，当没有重写hashcode方法时，插入p1后便无法用p2取出元素，这是因为p1和p2的哈希值不相等。

HashMap插入元素时是根据元素的哈希值来确定存放在数组中的位置，因此HashMap的key需要重写equals和hashcode方法。

## HashMap 的工作原理？

HashMap 底层是 **hash 数组和单向链表**实现，数组中的每个元素都是链表，由 Node 内部类（实现 Map.Entry接口）实现，HashMap 通过 put & get 方法存储和获取。

存储对象时，将 K/V 键值传给 put() 方法：

1. 调用 hash(K) 方法计算 K 的 hash 值，然后结合数组长度，计算得数组下标；
2. 调整数组大小（当容器中的元素个数大于 capacity * loadfactor 时，容器会进行扩容resize 为 2n）；
3. 如果 K 的 hash 值在 HashMap 中不存在，则执行插入，若存在，则发生碰撞；
4. 如果 K 的 hash 值在 HashMap 中存在，且它们两者 equals 返回 true，则更新键值对；
5.  如果 K 的 hash 值在 HashMap 中存在，且它们两者 equals 返回 false，则插入链表的尾部（**尾插法**）或者红黑树中（树的添加方式）。

（JDK 1.7 之前使用头插法、JDK 1.8 使用尾插法）（注意：当碰撞导致链表大于 TREEIFY_THRESHOLD = 8 时，就把链表转换成红黑树）

获取对象时，将 K 传给 get() 方法：

1. 调用 hash(K) 方法（计算 K 的 hash 值）从而获取该键值所在链表的数组下标；
2. 顺序遍历链表，equals()方法查找相同 Node 链表中 K 值对应的 V 值。

hashCode 是定位的，存储位置；equals是定性的，比较两者是否相等。

## 为什么把链表的头插法换为尾插法？

## 当两个对象的 hashCode 相同会发生什么？

因为  hashCode 相同，不一定就是相等的（equals方法比较），所以两个对象所在数组的下标相同，"碰撞"就此发生。又因为 HashMap  使用链表存储对象，这个 Node 会存储到链表中。为什么要重写 hashcode 和 equals 方法？推荐看下。

## 你知道 hash 的实现吗？为什么要这样实现？

JDK  1.8 中，是通过 hashCode() 的高 16 位异或低 16 位实现的：(h = k.hashCode()) ^ (h  >>> 16)，主要是从速度，功效和质量来考虑的，减少系统的开销，也不会造成因为高位没有参与下标的计算，从而引起的碰撞。

## 为什么要用异或运算符？

保证了对象的 hashCode 的 32 位值只要有一位发生改变，整个 hash() 返回值就会改变。尽可能的减少碰撞。

## HashMap 的 table 的容量如何确定？loadFactor 是什么？该容量如何变化？这种变化会带来什么问题？

1. table 数组大小是由 capacity 这个参数确定的，默认是16，也可以构造时传入，最大限制是1<<30；
2. loadFactor  是装载因子，主要目的是用来确认table 数组是否需要动态扩展，默认值是0.75，比如table 数组大小为 16，装载因子为 0.75  时，threshold 就是12，当 table 的实际大小超过 12 时，table就需要动态扩容；
3. 扩容时，调用 resize() 方法，将 table 长度变为原来的两倍（注意是 table 长度，而不是 threshold）
4. 如果数据很大的情况下，扩展时将会带来性能的损失，在性能要求很高的地方，这种损失很可能很致命。

## HashMap中put方法的过程？

答：“调用哈希函数获取Key对应的hash值，再计算其数组下标；

如果没有出现哈希冲突，则直接放入数组；如果出现哈希冲突，则以链表的方式放在链表后面；

如果链表长度超过阀值( TREEIFY THRESHOLD==8)，就把链表转成红黑树，链表长度低于6，就把红黑树转回链表;

如果结点的key已经存在，则替换其value即可；

如果集合中的键值对大于12，调用resize方法进行数组扩容。”

## 数组扩容的过程？

创建一个新的数组，其容量为旧数组的两倍，并重新计算旧数组中结点的存储位置。结点在新数组中的位置只有两种：

原下标位置或原下标+旧数组的大小。

## 拉链法导致的链表过深问题为什么不用二叉查找树代替，而选择红黑树？为什么不一直使用红黑树？

之所以选择红黑树是为了解决二叉查找树的缺陷，二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。

而红黑树在插入新数据后可能需要通过左旋，右旋、变色这些操作来保持平衡，引入红黑树就是为了查找数据快，解决链表查询深度的问题，我们知道**红黑树属于平衡二叉树**，但是为了保持“平衡”是需要付出代价的，但是该代价所损耗的资源要比遍历线性链表要少，所以当长度大于8的时候，会使用红黑树，如果链表长度很短的话，根本不需要引入红黑树，引入反而会慢。

## 说说你对红黑树的见解？

- 每个节点非红即黑
- 根节点总是黑色的
- 如果节点是红色的，则它的子节点必须是黑色的（反之不一定）
- 每个叶子节点都是黑色的空节点（NIL节点）
- 从根节点到叶节点或空子节点的每条路径，必须包含相同数目的黑色节点（即相同的黑色高度）

## jdk8中对HashMap做了哪些改变？

在java 1.8中，如果链表的长度超过了8，那么链表将转换为红黑树。（桶的数量必须大于64，小于64的时候只会扩容）

发生hash碰撞时，java 1.7 会在链表的头部插入，而java 1.8会在链表的尾部插入

在java 1.8中，Entry被Node替代(换了一个马甲）。

在链表长度大于 8 并且 表的长度大于 64 的时候会转化红黑树。

~~~java
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 并且如果 链表的长度 大于 8 会尝试调用  treeifyBin 方法
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }

~~~

**treeifyBin**

~~~java
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        // 如果表的长度小于 64 会先扩容！！！ 否则 扩容
        // MIN_TREEIFY_CAPACITY = 64;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }

~~~

- 并且如果 链表的长度 大于 8 会尝试调用 treeifyBin 方法
- 在此判断 表的长度是否大于64

## HashMap，LinkedHashMap，TreeMap 有什么区别？

HashMap 参考其他问题；

LinkedHashMap 保存了记录的插入顺序，在用 Iterator 遍历时，先取到的记录肯定是先插入的；遍历比 HashMap 慢；

TreeMap 实现 SortMap 接口，能够把它保存的记录根据键排序（默认按键值升序排序，也可以指定排序的比较器）

## HashMap & TreeMap & LinkedHashMap 使用场景？

一般情况下，使用最多的是 HashMap。

HashMap：在 Map 中插入、删除和定位元素时；

TreeMap：在需要按自然顺序或自定义顺序遍历键的情况下；

LinkedHashMap：在需要输出的顺序和输入的顺序相同的情况下。

## HashMap 和 HashTable 有什么区别？

1. HashMap 是线程不安全的，HashTable 是线程安全的；
2. 由于线程安全，所以 HashTable 的效率比不上 HashMap；
3. HashMap最多只允许一条记录的键为null，允许多条记录的值为null，而 HashTable不允许；
4. HashMap 默认初始化数组的大小为16，HashTable 为 11，前者扩容时，扩大两倍，后者扩大两倍+1；
5. HashMap 需要重新计算 hash 值，而 HashTable 直接使用对象的 hashCode

## Java 中的另一个线程安全的与 HashMap 极其类似的类是什么？同样是线程安全，它与 HashTable 在线程同步上有什么不同？

ConcurrentHashMap 类（是 Java并发包 java.util.concurrent 中提供的一个线程安全且高效的 HashMap 实现）。

HashTable 是使用 synchronize 关键字加锁的原理（就是对对象加锁）；

而针对 ConcurrentHashMap，在 JDK 1.7 中采用 分段锁的方式；JDK 1.8 中直接采用了CAS（无锁算法）+ synchronized。

## HashMap & ConcurrentHashMap 的区别？

除了加锁，原理上无太大区别。另外，HashMap 的键值对允许有null，但是ConCurrentHashMap 都不允许。

## 为什么 ConcurrentHashMap 比 HashTable 效率要高？

HashTable 使用一把锁（锁住整个链表结构）处理并发问题，多个线程竞争一把锁，容易阻塞；

ConcurrentHashMap

- JDK 1.7 中使用分段锁（ReentrantLock + Segment + HashEntry），相当于把一个 HashMap 分成多个段，每段分配一把锁，这样支持多线程访问。锁粒度：基于 Segment，包含多个 HashEntry。

- JDK 1.8 中使用 CAS + synchronized + Node + 红黑树。锁粒度：Node（首结

  点）（实现 Map.Entry）。锁粒度降低了。

## 针对 ConcurrentHashMap 锁机制具体分析（JDK 1.7 VS JDK 1.8）

JDK 1.7 中，采用**分段锁**的机制，实现并发的更新操作，底层采用**数组+链表**的存储结构，包括两个核心静态内部类 Segment 和 HashEntry。

1. Segment 继承 ReentrantLock（重入锁） 用来充当锁的角色，每个 Segment 对象守护每个散列映射表的若干个桶；
2. HashEntry 用来封装映射表的键-值对；
3. 每个桶是由若干个 HashEntry 对象链接起来的链表

![1641381454547](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/191734-937361.png)

> Segment的并发度就是ConcurrentHashMap 的并发度

JDK 1.8 中，采用Node + CAS + Synchronized来保证并发安全。取消类 Segment，直接用 table 
数组存储键值对；

当 HashEntry 对象组成的链表长度超过 TREEIFY_THRESHOLD 时，链表转换为红黑树，提升性能。底层变更为数组 + 链表 + 红黑树。

![1641381515068](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/191835-905504.png)

## ConcurrentHashMap 在 JDK 1.8 中，为什么要使用内置锁 synchronized 来代替重入锁 ReentrantLock？

1. 粒度降低了；
2. JVM 开发团队没有放弃 synchronized，而且基于 JVM 的 synchronized 优化空间更大，更加自然。
3. 在大量的数据操作下，对于 JVM 的内存压力，基于 API 的 ReentrantLock 会开销更多的内存。

## ConcurrentHashMap 简单介绍？

1. **重要的常量：**

~~~java
private transient volatile int sizeCtl;

当为负数时，-1 表示正在初始化，-N 表示 N - 1 个线程正在进行扩容；

当为 0 时，表示 table 还没有初始化；

当为其他正数时，表示初始化或者下一次进行扩容的大小。
~~~

2. **数据结构：**

Node 是存储结构的基本单元，继承 HashMap 中的 Entry，用于存储数据；

TreeNode 继承 Node，但是数据结构换成了二叉树结构，是红黑树的存储结构，用于红黑树中存储数据；

TreeBin 是封装 TreeNode 的容器，提供转换红黑树的一些条件和锁的控制。

3. **存储对象时（put() 方法）：**

如果没有初始化，就调用 initTable() 方法来进行初始化；

如果没有 hash 冲突就直接 CAS 无锁插入；

如果需要扩容，就先进行扩容；

如果存在 hash 冲突，就加锁来保证线程安全，两种情况：一种是链表形式就直接遍历

到尾端插入，一种是红黑树就按照红黑树结构插入；

如果该链表的数量大于阀值 8，就要先转换成红黑树的结构，break 再一次进入循环

如果添加成功就调用 addCount() 方法统计 size，并且检查是否需要扩容。

4. **扩容方法 transfer()：默认容量为 16，扩容时，容量变为原来的两倍。**

helpTransfer()：调用多个工作线程一起帮助进行扩容，这样的效率就会更高。

5. **获取对象时（get()方法）：**

计算 hash 值，定位到该 table 索引位置，如果是首结点符合就返回；

如果遇到扩容时，会调用标记正在扩容结点 ForwardingNode.find()方法，查找该结点，匹配就返回；

以上都不符合的话，就往下遍历结点，匹配就返回，否则最后就返回 null。

## ConcurrentHashMap 的并发度是什么？

程序运行时能够同时更新 ConccurentHashMap 且不产生锁竞争的最大线程数。默认为 16，且可以在构造函数中设置。

当用户设置并发度时，ConcurrentHashMap 会使用大于等于该值的最小2幂指数作为实际并发度（假如用户设置并发度为17，实际并发度则为32）

## Jdk1.7线程不安全体现在哪里

在对table进行扩容到newTable后，需要将原来数据转移到newTable中，在转移元素的过程中，使用的是头插法，也就是链表的顺序会翻转，这里也是形成死循环的关键点。下面进行详细分析。

### 扩容造成死循环分析过程

前提条件：

这里假设

1. hash算法为简单的用key mod链表的大小。
2. 最开始hash表size=2，key=3,7,5，则都在table[1]中。
3. 然后进行resize，使size变成4。

未resize前的数据结构如下：

![1641385549782](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/202559-122054.png)

如果在单线程环境下，最后的结果如下：

![1641385582652](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/202625-200656.png)

这里的转移过程，不再进行详述，只要理解transfer函数在做什么，其转移过程以及如何对链表进行反转应该不难。

然后在多线程环境下，假设有两个线程A和B都在进行put操作。线程A在执行到transfer函数中第11行代码处挂起，因为该函数在这里分析的地位非常重要，因此再次贴出来。

![1641385617297](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/202658-818971.png)

此时线程A中运行结果如下：

![1641385655875](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/202737-590388.png)

线程A挂起后，此时线程B正常执行，并完成resize操作，结果如下：

![1641385681808](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/202802-338476.png)

这里需要特别注意的点：**由于线程B已经执行完毕，根据Java内存模型，现在newTable和table中的Entry都是主存中最新值：7.next=3，3.next=null。**

此时切换到线程A上，在线程A挂起时内存中值如下：e=3，next=7，newTable[3]=null，代码执行过程如下：

~~~java
newTable[3]=e ----> newTable[3]=3
e=next ----> e=7
~~~

此时执行结果如下

![1641385925777](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/203208-215941.png)

**继续循环**

~~~java
e=7
next=e.next ----> next=3【从主存中取值】
e.next=newTable[3] ----> e.next=3【从主存中取值】
newTable[3]=e ----> newTable[3]=7
e=next ----> e=3
~~~

**结果如下**

![1641386042467](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/203404-615171.png)

**再次进入循环**

~~~java
e=3
next=e.next ----> next=null
e.next=newTable[3] ----> e.next=7 即：3.next=7
newTable[3]=e ----> newTable[3]=3
e=next ----> e=null
~~~

注意此次循环：e.next=7，而在上次循环中7.next=3，出现环形链表，并且此时e=null循环结束。

结果如下：

![1641386083112](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/203450-372697.png)

在后续操作中只要涉及轮询hashmap的数据结构，就会在这里发生死循环，造成悲剧。

### 扩容造成数据丢失分析过程

遵照上述分析过程，初始时：

![1641386219488](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/203700-495275.png)

线程A和线程B进行put操作，同样线程A挂起：

![1641386243478](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/203724-496821.png)

此时线程A的运行结果如下：

![1641386278368](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/203805-518997.png)

此时线程B已获得CPU时间片，并完成resize操作：

![1641386299568](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202201/05/203821-924194.png)

同样注意由于线程B执行完成，newTable和table都为最新值：5.next=null。

此时切换到线程A，在线程A挂起时：e=7，next=5，newTable[3]=null。

执行newtable[i]=e，就将7放在了table[3]的位置，此时next=5。接着进行下一次循环：

```
e=5
next=e.next ----> next=null，从主存中取值
e.next=newTable[1] ----> e.next=5，从主存中取值
newTable[1]=e ----> newTable[1]=5
e=next ----> e=null
```

将5放置在table[1]位置，此时e=null循环结束，3元素丢失，并形成环形链表。并在后续操作hashmap时造成死循环。

![1641386338779](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1641386338779.png)

### jdk1.8中HashMap

在jdk1.8中对HashMap进行了优化，在发生hash碰撞，不再采用头插法方式，而是直接插入链表尾部，因此不会出现环形链表的情况，但是在多线程的情况下仍然不安全，这里我们看jdk1.8中HashMap的put操作源码：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null) // 如果没有hash碰撞则直接插入元素
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

这是jdk1.8中HashMap中put操作的主函数， 注意第6行代码，如果没有hash碰撞则会直接插入元素。如果线程A和线程B同时进行put操作，刚好这两条不同的数据hash值一样，并且该位置数据为null，所以这线程A、B都会进入第6行代码中。

假设一种情况，线程A进入后还未进行数据插入时挂起，而线程B正常执行，从而正常插入数据，然后线程A获取CPU时间片，此时线程A不用再进行hash判断了，问题出现：线程A会把线程B插入的数据给覆盖，发生线程不安全。

这里只是简要分析下jdk1.8中HashMap出现的线程不安全问题的体现，后续将会对java的集合框架进行总结，到时再进行具体分析。

首先HashMap是线程不安全的，其主要体现：

1. 在jdk1.7中，在多线程环境下，扩容时会造成环形链或数据丢失。
2. 在jdk1.8中，在多线程环境下，会发生数据覆盖的情况。