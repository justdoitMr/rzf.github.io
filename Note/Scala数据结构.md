#  `Scala`数据结构
<!-- TOC -->

- [`Scala`数据结构](#scala数据结构)
    - [第六章，集合（上）](#第六章集合上)
        - [6.1，数据结构特点](#61数据结构特点)
        - [6.2，Scala不可变集合继承关系一览图](#62scala不可变集合继承关系一览图)
        - [6.3，Scala可变集合继承关系一览图](#63scala可变集合继承关系一览图)
        - [6.4，数组-定长数组(声明泛型)](#64数组-定长数组声明泛型)
        - [6.5，数组-变长数组(声明泛型)](#65数组-变长数组声明泛型)
        - [6.6，定长数组与变长数组的转换](#66定长数组与变长数组的转换)
        - [6.7，多维数组的定义和使用](#67多维数组的定义和使用)
        - [6.8，Scala数组转Java的List](#68scala数组转java的list)
        - [6.9，Java的List转Scala数组(mutable.Buffer)](#69java的list转scala数组mutablebuffer)
        - [6.10，元组Tuple-元组的基本使用](#610元组tuple-元组的基本使用)
        - [6.11，列表 List-创建List](#611列表-list-创建list)
        - [6.1.2，列表 ListBuffer](#612列表-listbuffer)
        - [6.13，队列 Queue-基本介绍](#613队列-queue-基本介绍)
        - [6.14，映射 Map-基本介绍](#614映射-map-基本介绍)
        - [6.15，集 Set-基本介绍](#615集-set-基本介绍)
    - [第七章，集合（下）](#第七章集合下)
        - [7.1，集合元素的映射-`map`映射操作](#71集合元素的映射-map映射操作)
        - [7.2，集合元素的映射-map映射操作](#72集合元素的映射-map映射操作)
            - [7.2.1，map映射操作](#721map映射操作)
            - [7.2.2，flatmap映射：flat即压扁，压平，扁平化映射](#722flatmap映射flat即压扁压平扁平化映射)
            - [7.2.3，集合元素的过滤-filter](#723集合元素的过滤-filter)
            - [7.2.4，化简, 也称归约](#724化简-也称归约)
            - [7.2.5，折叠](#725折叠)
            - [7.2.6，扫描](#726扫描)
            - [7.2.7，扩展-拉链(合并)](#727扩展-拉链合并)
            - [7.2.8，扩展-迭代器](#728扩展-迭代器)
            - [7.2.9，扩展-流 Stream](#729扩展-流-stream)
            - [7.2.10，扩展-视图 View(优化手段：推迟执行)](#7210扩展-视图-view优化手段推迟执行)
            - [7.2.11，扩展-并行集合](#7211扩展-并行集合)
            - [7.2.12，扩展-操作符](#7212扩展-操作符)
    - [第八章，模式匹配（`match`)](#第八章模式匹配match)
        - [8.1，match](#81match)
        - [8.2，模式中的变量及返回值](#82模式中的变量及返回值)
        - [8.3，类型匹配](#83类型匹配)
        - [8.4，数组匹配](#84数组匹配)
        - [8.5，匹配列表](#85匹配列表)
        - [8.6，匹配元组](#86匹配元组)
        - [8.7，对象匹配](#87对象匹配)
        - [8.8，变量声明中的模式](#88变量声明中的模式)
        - [8.9，for表达式中的模式](#89for表达式中的模式)
        - [8.10，样例类](#810样例类)
    - [第九章，函数式编程高级](#第九章函数式编程高级)
        - [9.1，先看一个需求](#91先看一个需求)
        - [9.2，偏函数](#92偏函数)
        - [9.3，作为参数的函数](#93作为参数的函数)
        - [9.4，匿名函数](#94匿名函数)
        - [9.5，高阶函数](#95高阶函数)
        - [9.6，参数(类型)推断](#96参数类型推断)
        - [9.7，闭包(closure)](#97闭包closure)
        - [9.8，函数柯里化](#98函数柯里化)
        - [9.9，控制抽象](#99控制抽象)
        - [第十章，泛型](#第十章泛型)

<!-- /TOC -->
## 第六章，集合（上）

### 6.1，数据结构特点

1. `Scala`集合概念

- `Scala`同时支持不可变集合和可变集合 

- 两个主要的包：

  - 不可变集合：`scala.collection.immutable`
  - 可变集合： ` scala.collection.mutable`

  - `Scala`默认采用不可变集合，对于几乎所有的集合类，`Scala`都同时提供了可变(`mutable`)和不可变(`immutable`)的版本
  - `Scala`的集合有三大类：序列`Seq`（线性有序）、集`Set`（无序不重复）、映射`Map`（键值对），所有的集合都扩展自`Iterable`特质，在`Scala`中集合有可变`（mutable）`和不可变`（immutable）`两种类型。 

2. 可变集合和不可变集合
   1. 不可变集合：scala不可变集合，就是这个集合本身不能动态变化。(类似java的数组，是不可以动态增长的)，可变不可变是针对集合的容量大小的，而非是否可以修改内容。
   2. 可变集合:可变集合，就是这个集合本身可以动态变化的。(比如:ArrayList , 是可以动态增长的) ，容量大小可以改变。

### 6.2，Scala不可变集合继承关系一览图

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/202405-138636.png)

- 总结
  - set和map是java中也有的集合

### 6.3，Scala可变集合继承关系一览图

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/202407-694394.png)

- 说明：
  - 可变集合比不可变集合更加丰富。
  - 在seq集合中增加了buffer集合。
  - 在开发中，经常用到ArrayBuffer和ListBuffer集合。
  - 如果涉及到线程安全问题，可以使用syn开头的集合。

### 6.4，数组-定长数组(声明泛型)

1. 第一种方式定义数组
   这里的数组等同于Java中的数组,中括号的类型就是数组的类型

   ~~~ java
   package qq.com.rzf.collection
   
   object MyArray {
     def main(args: Array[String]): Unit = {
       /**
        * 声明一个数组：[int]标示泛型（4）标示数组的大小
        * 如果想让数组中什么都存放，可以声明为[Any]
        * 没有赋值，各个元素值为0
        * 访问用（），而不是[]
        */
       val arr01 = new Array[Int](4)
       println(arr01.length)
       println("arr01(0)=" + arr01(0))
   
       for (i <- arr01) { //遍历
         println(i)
       }
   
       println("--------------------")
       arr01(3) = 10
   
       for (i <- arr01) {
         println(i)
       }
     }
   
   }
   //底层源码
    int[] arr01 = new int[4];
   ~~~

2. 在定义数组时，直接赋值，使用apply方法创建数组对象

   ~~~java
    /**
      * 使用的是object array 的apply方法
      * 创建数组的时候直接初始化，根据数组值类型自动推断，因为数组中有整数和字符串，那么数组泛型就是any
      */
     var arr02 = Array(1, 3, "xxx")
     for (i <- arr02) {
       println(i)
     }
     println("********************")
     for(index <- 0 to arr02.length-1){
      printf("arr02[%d]=%s",index,arr02(index))
     }
   ~~~

### 6.5，数组-变长数组(声明泛型)

~~~ java
package qq.com.rzf.collection
import scala.collection.mutable.ArrayBuffer
object arraybuffer {
  def main(args: Array[String]): Unit = {
    val arr01 = ArrayBuffer[Any](3,4,5,6,7)
    println("arr01(1)=" + arr01(1))
    for (i <- arr01) {
      println(i)
    }
    println(arr01.length)
    println("arr01.hash=" + arr01.hashCode())
    //append支持可变参数，append在底层创建新数组返回，即java中数组的扩容
    arr01.append(90.0,13)
    println("arr01.hash=" + arr01.hashCode())
    arr01(1) = 89 //修改
    println("--------------------------")
    for (i <- arr01) {
      println(i)
    }
    //根据下表删除
    arr01.remove(0)
    println("--------------------------")
    for (i <- arr01) {
      println(i)
    }
    println("最新的长度=" + arr01.length)

  }
}

~~~

1. 变长数组分析小结
   1. ArrayBuffer是变长数组，类似java的ArrayList

   2.  val arr2 = ArrayBuffer[Int]() 也是使用的apply方法构建对象

   3.  def append(elems: A*) { appendAll(elems) } 接收的是可变参数.

   4. 每append一次，arr在底层会重新分配空间，进行扩容，arr2的内存地址会发生变化，也就成为新的ArrayBuffer

### 6.6，定长数组与变长数组的转换

~~~java
arr1.toBuffer  //定长数组转可变数组
arr2.toArray  //可变数组转定长数组
~~~

- 说明：

  - arr2.toArray 返回结果才是一个定长数组， arr2本身没有变化

  - arr1.toBuffer返回结果才是一个可变数组， arr1本身没有变化

  ~~~ java
  package qq.com.rzf.collection
  import scala.collection.mutable.ArrayBuffer
  object ArrayToBuffer {
    def main(args: Array[String]): Unit = {
      var ab=ArrayBuffer[Int]()
      //添加数据
      ab.append(1)
      ab.append(2)
      println(ab)
      //把变长数组转换为定长数组
      //但是此处ab没有收到影响，只是重新返回一个定长数组类型
      var arr=ab.toArray
      println(arr.length)
      for(i<- arr){
        println(i)
      }
    }
  }
  
  ~~~

### 6.7，多维数组的定义和使用

1. 定义

   ~~~ java
   val arr = Array.ofDim[Double](3,4)
       //说明：
   arr 是一个二维数组
   有三个元素[一维数组]
   每个一维数组存放4个值
   ~~~

2. 案例

   ~~~ java
   package qq.com.rzf.collection
   
   object MultiplyArray {
     def main(args: Array[String]): Unit = {
       var multArr=Array.ofDim[Int](3,4)//创建3行4列的二维数组
       //遍历
       for(i <- multArr){//取出每一行元素
         for(j <- i){//取出每一行中的每一个元素
           print(j+"\t")
         }
         println()
       }
       //取出单个元素
       println(multArr(1)(1))
       //修改值
       multArr(1)(1)=100
       println(multArr(1)(1))
       //第二种遍历方式
       for(i <- 0 to multArr.length-1){//取出每一行元素
         for(j <- 0 to multArr(i).length-1){//对取出来的每一行元素遍历
           printf("multArr(%d)(%d)=%d\t",i,j,multArr(i)(j))//格式化输出
         }
         println()
       }
     }
   }
   
   ~~~

### 6.8，Scala数组转Java的List 

~~~ java
package qq.com.rzf.collection

import scala.collection.mutable.ArrayBuffer

object ScalaToJava {
  def main(args: Array[String]): Unit = {
    // Scala集合和Java集合互相转换
    val arr = ArrayBuffer("1", "2", "3")
    //implicit def bufferAsJavaList[A]
    // (b : scala.collection.mutable.Buffer[A])
    // : java.util.List[A] = { /* compiled code */ }
    import scala.collection.JavaConversions.bufferAsJavaList
    // ProcessBuilder方法接收参数List<String> command，然后返回commanad
    //这里使用上面的隐式转换函数，
    val javaArr = new ProcessBuilder(arr)
    //public List<String> command()方法返回java中的list
    val arrList = javaArr.command()

    println(arrList)

  }
}

~~~

### 6.9，Java的List转Scala数组(mutable.Buffer)

~~~ java
package qq.com.rzf.collection

import scala.collection.mutable.ArrayBuffer

object JavaToScala {
  def main(args: Array[String]): Unit = {
    // Scala集合和Java集合互相转换
    val arr = ArrayBuffer("1", "2", "3")
    //implicit def bufferAsJavaList[A]
    // (b : scala.collection.mutable.Buffer[A])
    // : java.util.List[A] = { /* compiled code */ }
    import scala.collection.JavaConversions.bufferAsJavaList
    // ProcessBuilder方法接收参数List<String> command，然后返回commanad
    //这里使用上面的隐式转换函数，
    val javaArr = new ProcessBuilder(arr)
    //public List<String> command()方法返回java中的list
    val arrList = javaArr.command()

    println(arrList)
    import scala.collection.JavaConversions.asScalaBuffer
    import scala.collection.mutable

    // java.util.List ==> Buffer
    val scalaArr: mutable.Buffer[String] = arrList

    scalaArr.append("jack")
    println(scalaArr)


  }
}
~~~

### 6.10，元组Tuple-元组的基本使用

- 基本介绍
  元组也是可以理解为一个容器，可以存放各种相同或不同类型的数据。
  说的简单点，就是将多个无关的数据封装为一个整体，称为元组, 最大的特点灵活,对数据没有过多的约束。 

  注意：元组中最大只能有22个元素

1. 元组的创建

   ~~~ java
   package qq.com.rzf.collection
   
   object TuplDemo01 {
     def main(args: Array[String]): Unit = {
       /**
        * tuple1是一个tuple类型是tuple5
        * 为了搞笑操作元组，编译器根据元组中元素个数不同，对应为不同元组类型
        * 分别是：tuple1--tuple22
        */
       val tuple1 = (1, 2, 3, "hello", 4)
       println(tuple1)
     }
   }
   
   ~~~

2. 元组Tuple-元组数据的访问

   - 基本介绍
     访问元组中的数据,可以采用顺序号（_顺序号），也可以通过索引（productElement）访问。

   ~~~ java
   package qq.com.rzf.collection
   
   object TuplDemo01 {
     def main(args: Array[String]): Unit = {
       /**
        * tuple1是一个tuple类型是tuple5
        * 为了搞笑操作元组，编译器根据元组中元素个数不同，对应为不同元组类型
        * 分别是：tuple1--tuple22
        */
       val tuple1 = (1, 2, 3, "hello", 4)
       println(tuple1)
       //var tup1 = 2 -> “two”
       //元组访问的两种方式
       //第一种
       println(tuple1._1)
       //第二种,第二种只是包装而已
       /*
       底层代码：模式匹配
       override def productElement(n: Int) = n match {
       case 0 => _1
       case 1 => _2
       case 2 => _3
       case 3 => _4
       case 4 => _5
       case _ => throw new IndexOutOfBoundsException(n.toString())
    }
        */
       println(tuple1.productElement(0))
     }
   }
   
   ~~~

3. 元组Tuple-元组数据的遍历

   - Tuple是一个整体，遍历需要调其迭代器。

     ~~~ java
      //元组的遍历要用迭代器
         for(tmp <- tuple1.productIterator){
           println(tmp)
         }
     ~~~

### 6.11，列表 List-创建List

1. 基本介绍
   Scala中的List 和Java List 不一样，在Java中List是一个接口，真正存放数据是ArrayList，而Scala的List可以直接存放数据，就是一个object，默认情况下Scala的List是不可变的，List属于序列Seq。\

2. 创建List的应用案例

   ~~~ java
   package qq.com.rzf.collection
   import scala.collection.immutable.List//不可变
   object ListDemo {
     def main(args: Array[String]): Unit = {
       /*
       默认情况下引入scala.collection.immutable.List包下的list，即不可变
       如果需要可变的，就用listBuffer
       list也在package object scala包中做了声明，所以使用时不需要导包
        */
       //list泛型是any
       var list=List(1,2,3,4,"hello")
       println(list)
       var list1=Nil//空集合
       //Nil也是一个集合，即List()，里面内容为空
       println(list1)
         //列表的访问,列表的下表也是从0开始
       var t=list(1)
       println(t)
   
     }
   }
   
   ~~~

   - 创建List的应用案例小结

   List 在 scala包对象声明的,因此不需要引入其它包也可以使用

   val List = scala.collection.immutable.List

   List 中可以放任何数据类型，比如 arr1的类型为 List[Any]

   如果希望得到一个空列表，可以使用Nil对象, 在 scala包对象声明的,因此不需要引入其它包也可以使用val Nil = scala.collection.immutable.Nil

3. 列表 List-元素的追加

   - 基本介绍
     向列表中增加元素, **会返回新的列表/集合对象**。注意：Scala中List元素的追加形式非常独特，和Java不一样。

   - 方式1-在列表的最后增加数据

     ~~~java
     var list1 = List(1, 2, 3, "abc")
     // :+运算符表示在列表的最后增加数据
     val list2 = list1 :+ 4
     println(list1) //list1没有变化
     println(list2) //新的列表结果是 [1, 2, 3, "abc", 4]
     ~~~

   - 方式2-在列表的最前面增加数据

     ~~~ java
     var list1 = List(1, 2, 3, "abc")
     // :+运算符表示在列表的最后增加数据
     val list2 = 4 +: list1
     println(list1) //list1没有变化
     println(list2) //新的列表结果是?
     ~~~

   - 方式3-在列表的最后增加数据

     - 说明:

       - 符号::表示向集合中  新建集合添加元素。
       - 运算时，集合对象一定要放置在最右边，
         运算规则，从右向左。
       - ::: 运算符是将集合中的每一个元素加入到空集合中去
       - `list:::list1`标示把两边的集合全部打散，归并到一个集合中去，所以`:::`两边必须全部是集合，`:::`两边不能有数据。

       ~~~ java
       package qq.com.rzf.collection
       
       object List01 {
         def main(args: Array[String]): Unit = {
           var list04=List(11,12,13,"hello")
           /**
            * ::符号的使用：
            * var list02 = 1::2::3::4::list03::Nil步骤
            * 操作步骤：
            * 1,先创建空的集合list02
            * 2,把集合list04添加进去
            * 3，添加元素4,
            * 4，添加元素3，依次从后往前添加
            */
           var list02 = 1::2::3::4::list04::Nil
           println(list02)//结果List(1, 2, 3, 4, List(11, 12, 13, hello))
           /**
            * :::标示把集合中的元素打散在放入另一个集合
            *
            */
           var list05 = 1::2::3::4::list04:::Nil
           println(list05)//List(1, 2, 3, 4, 11, 12, 13, hello)
       
         }
       }
       
       ~~~

     - 练习

       ~~~ java
       val list1 = List(1, 2, 3, "abc")
       val list5 = 4 :: 5 :: 6 :: list1 
       println(list5) // (4, 5, 6, 1, 2, 3, “abc”)
       val list1 = List(1, 2, 3, "abc")
       val list5 = 4 :: 5 :: 6 :: list1 :: 9//语法错误
       println(list5) // 报错
       val list1 = List(1, 2, 3, "abc")
       val list5 = 4 :: 5 :: list1 ::: list1 ::: Nil
       println(list5) // (4, 5, 1, 2, 3, “abc”,1, 2, 3, “abc”)
       ~~~

### 6.1.2，列表 ListBuffer

- 前面提到的`list`每一次进行添加或者删除后，都会重新产生一个新的集合，操作很不方便。但是`ListBuffer`可以在集合本身上面修改，集合本身发生变化。

  ~~~ java
  package qq.com.rzf.collection
  import scala.collection.mutable.ListBuffer
  object MyListBuffer {
    def main(args: Array[String]): Unit = {
      //创建listBuffer
      //如果泛型改为any,一个集合中可以存放多用数据类型
      var lb=ListBuffer[Int](1,2,3,4,5)
      //访问和遍历集合
      println(lb(2))//访问第三个元素
      println("***********************")
      for(tmp <- lb){
        println(tmp)
      }
      //创建一个空的集合，逐次添加元素，此处是动态添加元素，对集合本身有改变
      var lb1=new ListBuffer[Any]
      //两种方式添加元素
      lb1 += 3 //添加第一个元素
      lb1.append("hello",4,5,6)//变参数函数
      /**
       * ++=:标示把一个集合添加到另一个集合中去
       */
      lb1 ++= lb//标示把lb集合中元素添加到lb1集合中,打散添加
      //结果ListBuffer(3, hello, 4, 5, 6, 1, 2, 3, 4, 5)
      println(lb1)
  
      /**
       * ++ :标示两个集合直接相加
       */
      var lb2=lb ++ lb1
      println(lb2)
      //结果ListBuffer(1, 2, 3, 4, 5, 3, hello, 4, 5, 6, 1, 2, 3, 4, 5)
  
      /**
       * :+ 标示把集合和元素相加，赋给另一个集合
       * 但是此集合本身不会发生变化
       */
      var lb3=lb :+ 999
      println(lb3)//结果：ListBuffer(1, 2, 3, 4, 5, 999)
      println(lb)//本身没有发生变化
      //删除元素
      lb3.remove(1)
      println(lb3)//删除第二个元素ListBuffer(1, 3, 4, 5, 999)
    }
  }
  
  ~~~

### 6.13，队列 Queue-基本介绍

1. 队列的说明

   1. 队列是一个有序列表，在底层可以用数组或是链表来实现。
   2. 其输入和输出要遵循先入先出的原则。即：先存入队列的数据，要先取出。后存入的要后取出
   3. 在Scala中，由设计者直接给我们提供队列类型使用。
   4. 在scala中, 有 scala.collection.mutable.Queue 和 scala.collection.immutable.Queue , 一般来说，我们在开发中通常使用可变集合中的队列。
   5. 队列底层可以用数组，也可以用链表实现。

2. 队列举例

   ~~~ java
   package qq.com.rzf.collection
   
   import scala.collection.mutable
   
   object MyQueue {
     def main(args: Array[String]): Unit = {
       //创建一个队列
       val q1 = new mutable.Queue[Any]
       //向队列中添加元素
       //添加一个元素
       q1 += 20
       println(q1)// Queue(20)
       //把list中的元素添加到队列Queue(20, 2, 4, 6)
       q1 ++= List(2,4,6)
       println(q1)
       //把list集合添加到队列中
       q1 += List(1,2,3)
       println(q1)//Queue(20, 2, 4, 6, List(1, 2, 3))
   
   
     }
   }
   
   ~~~

3. 队列 Queue-删除和加入队列元素(删除添加是对队列本身的改变)

   ~~~ java
   package qq.com.rzf.collection
   
   import scala.collection.mutable
   
   object MyQueue {
     def main(args: Array[String]): Unit = {
       //创建一个队列
       val q1 = new mutable.Queue[Any]
       //向队列中添加元素
       //添加一个元素
       q1 += 20
       println(q1)// Queue(20)
       //把list中的元素添加到队列Queue(20, 2, 4, 6)
       q1 ++= List(2,4,6)
       println(q1)
       //把list集合添加到队列中
       q1 += List(1,2,3)
       println(q1)//Queue(20, 2, 4, 6, List(1, 2, 3))
       //向队列中添加元素,从尾部添加
       q1.enqueue("xiaoming")
       q1.enqueue(112)
       println(q1)
       //删除队头元素
       q1.dequeue()
       println(q1)
   
   
     }
   }
   ~~~

4. 队列 Queue-返回队列的元素

   ~~~ java
   //返回队列的元素，相当于对队列的读取操作，本身不改变队列
       //获取队列的第一个元素
       println(q1.head)
       //获取队列的对尾元素
       println(q1.last)
       //返回队列中除了第一个元素的所有其他元素，以队列形式返回
       //返回队列的尾部
   	//即：返回除了第一个以外剩余的元素， 可以级联使用，这个在递归时使用较多。
       println(q1.tail.tail)
   
   ~~~

### 6.14，映射 Map-基本介绍

1. Java中的Map回顾

   - HashMap 是一个散列表(数组+链表)，它存储的内容是键值对(key-value)映射，Java中的HashMap是无序的，**key不能重复，但是`value`可以重复**

2. 案例：

   ~~~ java
   package qq.com.rzf.collection;
   
   import java.util.HashMap;
   
   public class JavaMap {
       public static void main(String[] args) {
           HashMap<String,Integer> hm = new HashMap();
           hm.put("no1", 100);
           hm.put("no2", 200);
           hm.put("no3", 300);
           hm.put("no4", 400);
           hm.put("no1",1000);//key相同，在底层做的是更新操作
           System.out.println(hm);//无序，输出结果和存放结果不一定一样
           System.out.println(hm.get("no2"));
   
       }
   }
   ~~~

3. Scala中的Map介绍

   1. 基本介绍

      1. Scala中的Map 和Java类似，也是一个散列表，它存储的内容也是键值对(key-value)映射，Scala中不可变的Map是有序的，可变的Map是无序的。

      2. Scala中，有可变Map (scala.collection.mutable.Map) 和 不可变Map(scala.collection.immutable.Map) 

   2. 方式1-构造不可变映射(不可以对集合进行动态删除和增加操作)

      - Scala中的不可变Map是有序，构建Map中的元素底层是Tuple2类型。

        ~~~ java
        package qq.com.rzf.collection
        
        object ScalaMap {
          def main(args: Array[String]): Unit = {
            //在包package scala中，不可改变val Map         = immutable.Map
            //创建map集合并且初始化元素
            //key-value支持的类型是any,在map的底层，每一对键值对是tuple2
            var myMap=Map(1->"小明",2->"小红",3->"小芮")
            println(myMap)
          }
        }
        /*
        val map1 = Map("Alice" -> 10, "Bob" -> 20, "Kotlin" -> "北京")
        小结
        1.从输出的结果看到，输出顺序和声明顺序一致
        2.构建Map集合中，集合中的元素其实是Tuple2类型 
        3.默认情况下（即没有引入其它包的情况下）,Map是不可变map
        4.为什么说Map中的元素是Tuple2 类型 [反编译或看对应的apply]*/
        
        ~~~

   3. 方式2-构造可变映射(需要指定可变Map的包)

      ~~~ java
      package qq.com.rzf.collection
      
      import scala.collection.mutable
      
      object ScalaMap {
        def main(args: Array[String]): Unit = {
          //在包package scala中，不可改变val Map         = immutable.Map
          //创建map集合并且初始化元素
          //key-value支持的类型是any,在map的底层，每一对键值对是tuple2
          var myMap=Map(1->"小明",2->"小红",3->"小芮")
          println(myMap)//Map(1 -> 小明, 2 -> 小红, 3 -> 小芮)
          //可以改变map集合
          var myMap1=mutable.Map(1->"小花",2->"小冬",3->"小名")
          println(myMap1)//Map(2 -> 小冬, 1 -> 小花, 3 -> 小名)和输入顺序不一致
        }
      }
      
      ~~~

   4. 方式3-创建空的映射

      ~~~ java
      val map3 = new scala.collection.mutable.HashMap[String, Int]
      println(map3)
      ~~~

   5. 方式4-对偶元组

      - 即创建包含键值对的二元组， 和第一种方式等价，只是形式上不同而已。
        对偶元组 就是只含有两个数据的元组。

      ~~~ java
      val map4 = mutable.Map( ("A", 1), ("B", 2), ("C", 3),("D", 30) )
      println("map4=" + map4)
      println(map4("A"))
      ~~~

   6. 映射 Map-取值

      1. 方式1-使用map(key) 

      ~~~ java
      //方式1-使用map(key) 
       var value=myMap(1)
       println(value)
      ~~~

      - 说明:
        1) 如果键存在, 返回对应的值对象
        2) 如果键不存在, 抛出异常
        3) 在java中 如果不存在返回null

      2. 方式2-使用contains方法检查是否存在key

         ~~~ java
         if(myMap.contains(5)){
               println(myMap(5))
             }else{
               println("值不存在")
             }
         ~~~

         - 说明：
           - 返回Boolean
             1.如果key存在，则返回true
             2.如果key不存在，则返回false
           - 使用containts先判断在取值，可以防止异常，并加入相应的处理逻辑

      3. 方式3-使用map.get(key).get取值 

         - 通过 映射.get(键) 这样的调用返回一个Option对象，要么是Some，要么是None

           ~~~ java
           println(myMap.get(3))//得到的是some对象
           println(myMap.get(3).get)//得到some里面的值,如果返回none,就取不到值，取值会抛出异常
           ~~~

      4. 方式4-使用map4.getOrElse()取值 

         - getOrElse 方法 : def getOrElse`[V1 >: V] (key: K, default: => V1)`

         - 说明：
           如果key存在，返回key对应的值。
           如果key不存在，返回默认值。在java中底层有很多类似的操作。

           ~~~ java
           val map4 = mutable.Map( ("A", 1), ("B", "北京"), ("C", 3) )
           println(map4.getOrElse("A","默认"))
           ~~~

      5. 如何选择取值方式建议

         1. 如果我们确定key是存在的，应该使用map("key") ,速度快.
         2. 如果我们不确定key是否存在， 而且在不存在时，有业务逻辑处理就是用map.contains() 配合 map("key")
         3. 如果只是简单的希望返回一个值，就使用getOrElse() 

      6. 映射 Map-对map修改、添加和删除

         1. 修改操作

         ~~~ java
         //更新元素
          //更新元素
             myMap1(1)="aaa"
             println(myMap1)
         ~~~

         -  说明:

          map 是可变的，才能修改，否则报错

         如果key存在：则修改对应的值,key不存在,等价于添加一个key-val

         2. 添加操作

            ~~~java
            //集合前提是可变的
            //map增加元素
                //增加单个元素
                myMap1 +=(5 ->"xxx")
                //增加多个元素
                myMap1 +=(6->"aaa",7->"bbb")
            ~~~

            - 如果增加元素已经存在，那么会执行更新操作。

         3. 删除操作

            ~~~ java
            //map删除元素
                myMap1 -=(1,5)//删除不存在元素也不会报错
                println(myMap)
            ~~~

            - 说明:1,5是要删除的元素的key值，可以删除多个元素，如果key存在，就删除，如果不存在，不会报错。

      7. 映射 Map-对map遍历

         - map的遍历支持多种形式:

         ~~~ java
         val map1 = mutable.Map( ("A", 1), ("B", "北京"), ("C", 3) )
         for ((k, v) <- map1) println(k + " is mapped to " + v)
         for (v <- map1.keys) println(v)
         for (v <- map1.values) println(v) //早上 
         for(v <- map1) println(v._1,v._2) //v是Tuple2
         
         //说明
         //1.每遍历一次，返回的元素是Tuple2
         //2.取出的时候，可以按照元组的方式来取
         
         ~~~

### 6.15，集 Set-基本介绍

-  集是不重复元素的结合。集不保留顺序，默认是以哈希集实现

1. Java中Set的回顾

   - java中，HashSet是实现Set<E>接口的一个实体类，数据是以哈希表的形式存放的，里面的不能包含重复数据。**Set接口是一种不包含重复元素的** collection，HashSet中的数据也是**没有顺序的**。 

     ~~~java
     public class JavaSet {
         public static void main(String[] args) {
             HashSet hs = new HashSet<String>();
             hs.add("jack");
             hs.add("tom");
             hs.add("jack");
             hs.add("jack2");
             System.out.println(hs);
     
         }
     }
     ~~~

2.  Scala中Set的说明

   - 默认情况下，Scala 使用的是不可变集合，如果你想使用可变集合，需要引用 scala.collection.mutable.Set 包

   1. Scala的可变Set和不可变Set的举例

      ~~~ java
      package qq.com.rzf.collection
      
      object MySet {
        def main(args: Array[String]): Unit = {
          //可变集合的创建，默认在scala包下
          var mySet=Set(1,2,3,4,"asd")
          println(mySet)//无序Set(1, 2, asd, 3, 4)
          //不可变集合创建
          var mySet1=scala.collection.immutable.Set(6,7,8,9,9)//有去重操作
          println(mySet1)//Set(6, 7, 8, 9)
        }
      }
      
      ~~~

   2. 集 Set-可变集合的元素添加和删除

      ~~~ java
      //set可变集合添加元素的三种方式
          mySet +=2//第一种方式，直接按值添加
          mySet +=(999)//第二种
         // mySet.add(333)//第三种方式
          //如果元素已经存在，那么不会重复添加，也不会报错
      ~~~

   3. 可变集合的元素删除

      ~~~ java
      //集合删除元素
          mySet -=2//按照值进行删除
          mySet -=(999)
          //mySet.remove(999)直接删除元素
          //说明：说明：如果删除的对象不存在，则不生效，也不会报错
      ~~~

   4. set集合的遍历

      ~~~ java
      val set03 = Set(1,9,-10)
      println("max=" + (set03 max))
      println("max=" + (set03 min))
      ~~~

## 第七章，集合（下）

### 7.1，集合元素的映射-`map`映射操作

1. 看一个实际需求

   - 请将List(3,5,7) 中的所有元素都 * 2 ，将其结果放到一个新的集合中返回，即返回一个新的List(6,10,14), 请编写程序实现.

     ~~~ java
     package qq.com.rzf.collection
     
     object MyExercise {
       def main(args: Array[String]): Unit = {
         /**
          * 请将List(3,5,7) 中的所有元素都 * 2 ，将其结果放到一个新的集合中返回，
          * 即返回一个新的List(6,10,14), 请编写程序实现.
          */
         var list01=List(3,5,7)
         //新创建集合，存放元素
         var list02=List[Int]()
         for(item <- list01)
           {
             list02= list02 :+item*2
           }
         println(list02)
     
         /**
          * 小结：
          * 优点：
          * 1 处理方式直接，很好理解
          * 2缺点：
          * 1 不够简洁，不高效
          * 2 没有体现函数式编程
          * 集合-->函数 =》新集合-->函数....
          * 3 不利于处理复杂的数据业务
          */
       }
     }
     ~~~

   - 传统方法优点分析:

     1. 简单，很好理解

   - 传统方法缺点分析:

     1. 不够简洁，不利于模块化编程2.如果我们需要对返回的集合进行下一步处理，不方便

### 7.2，集合元素的映射-map映射操作

#### 7.2.1，map映射操作

1. map映射操作

   - 上面提出的问题，其实就是一个关于集合元素映射操作的问题。

     - 在Scala中可以通过map映射操作来解决：将集合中的每一个元素通过指定功能（函数）映射（转换）成新的结果集合这里其实就是所谓的将函数作为参数传递给另外一个函数,这是函数式编程的特点

     - 以HashSet为例说明
       def map[B](f: (A) ⇒ B): HashSet[B]   //map函数的签名
       这个就是map映射函数集合类型都有
       [B] 是泛型
       map 是一个高阶函数(可以接受一个函数的函数，就是高阶函数)，可以接收 函数 f: (A) => B 后面详解(先简单介绍下.)
       HashSet[B] 就是返回的新的集合

2. 高阶函数案例

     ~~~ java
     package qq.com.rzf.collection
     
     object HighFunction {
       def main(args: Array[String]): Unit = {
         //高阶函数的使用
         var res = test(sum, 3.5)
         println(res)
         //在scala中，可以把一个函数赋值给一个变量
         //把一个下划线给一个函数，标示把函数赋值给一个变量，但是并不执行函数
         var res1=print _
         //现在用变量res1执行函数
         res1()
         //res标示是一个函数，res()标示是一个值
       }
       def print():Unit={
         println("hello word")
       }
     
       /**
        * test是一个高阶函数
        * f:Double => Double,n1:Double标示一个函数，该函数可以接受一个double类型，返回double类型
        * n1:Double标示普通参数
        * f(n1)标示执行你传入的参数
        */
       def test(f:Double => Double,n1:Double)={
         f(n1)
       }
       def sum(d:Double):Double={
         println("sum函数被执行了")
         d+d
       }
     }
     def main(args: Array[String]): Unit = {
         test2(sayOK _)
     }
     
     def test2(f: () => Unit) = {
         f()
     }
     
     def sayOK() = {
         println("sayOKKK...")
     }
     ~~~

3. `map`模拟实现

   ~~~ java
   //模拟map的底层原理
   class MyList{
     var list1=List(3,5,7)
     //新创建集合
     var list2=List[Int]()
     //map函数
     def map(f:Int => Int):List[Int]={
       for(item <- list1){
         list2 =list2 :+ f(item)
       }
       list2
     }
   }
   //伴生对象
   object MyList{
     def apply(): MyList = new MyList()
   }
   ~~~

   1. 练习：

      - 请将 val names = List("Alice", "Bob", "Nick") 中的所有单词，全部转成字母大写，返回到新的List集合中.

        ~~~ java
        package qq.com.rzf.collection
        
        object LowToupper {
          def main(args: Array[String]): Unit = {
            /**
             * 请将 val names = List("Alice", "Bob", "Nick") 中的所有单词，
             * 全部转成字母大写，返回到新的List集合中.
             */
            val names=List("Alice", "Bob", "Nick")
            var name1=names.map(lowToupper)
            println(name1)
        
          }
          def lowToupper(name:String):String={
            name.toUpperCase
          }
        }
        
        ~~~

#### 7.2.2，flatmap映射：flat即压扁，压平，扁平化映射

- flatmap：flat即压扁，压平，扁平化，效果就是将集合中的每个元素的子元素映射到某个函数并返回新的集合。

  ~~~ java
  package qq.com.rzf.collection
  
  import qq.com.rzf.collection.LowToupper.lowToupper
  
  object FlatMap {
    def main(args: Array[String]): Unit = {
      val names=List("Alice", "Bob", "Nick")
      //把集合中的元素全部打散输出
      var name1=names.flatMap(lowToupper)
      println(name1)
    }
    def lowToupper(name:String):String={
      name.toUpperCase
    }
  }
  ~~~

#### 7.2.3，集合元素的过滤-filter

- filter：将符合要求的数据(筛选)放置到新的集合中

- 案例：应用案例：将  val names = List("Alice", "Bob", "Nick") 集合中首字母为'A'的筛选到新的集合。

  ~~~ java
  package qq.com.rzf.collection
  
  object MyFilter {
    def main(args: Array[String]): Unit = {
      /**
       * val names = List("Alice", "Bob", "Nick")输出以大写字母a开头的单词
       */
      val names = List("Alice", "Bob", "Nick")
      //输入函数的要求是传入参数string类型，返回boolean类型
      var name=names.filter(startWithA)
      println(name)
    }
    def startWithA(s:String):Boolean={
      s.startsWith("A")
    }
  }
  ~~~

#### 7.2.4，化简, 也称归约

1. 看一个需求:
   val list = List(1, 20, 30, 4 ,5) , 求出list的和.

2. 化简：
   化简：将二元函数引用于集合中的函数,。
   上面的问题当然可以使用遍历list方法来解决，这里我们使用scala的化简方式来完成。

   ~~~ java
   package qq.com.rzf.collection
   
   object ReducerLift {
     def main(args: Array[String]): Unit = {
       /**
        * 函数执行流程
        * 1，先把(1，2)的和计算出来
        * 2，把第一次计算的两个数的个作为第二次计算的第一个参数
        * 3, ((1,2),3)=和
        * 4，(((1,2),3),4)=和
        * 以此类推，调用n-1此函数，n是集合中元素个数
        */
       var list=List(1,2,3,4,5,6,7,8,9)
       var s=list.reduceLeft(sum)
       println(s)
     }
     def sum(n1:Int,n2:Int):Int={
       n1+n2
     }
   }
   ~~~

3. 执行过程

   ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/202427-331951.png)

4. 练习

   1. 代码分析：

   ~~~ java
   package qq.com.rzf.collection
   
   object exerciseReducerLift {
     def main(args: Array[String]): Unit = {
       val list = List(1, 2, 3, 4 ,5)
       def minus( num1 : Int, num2 : Int ): Int = {
         num1 - num2
       }
       println(list.reduceLeft(minus)) //-13
       println(list.reduceRight(minus))//3
       println(list.reduce(minus))//-13
   
     }
   }
   ~~~

   2. 使用化简的方法求出 List(3,4,2,7,5) 最小的值

      ~~~ java
      package qq.com.rzf.collection
      
      object getMin {
        def main(args: Array[String]): Unit = {
          var list=List(2,3,6,8,9,1,23,4,67,98)
          var num=list.reduceLeft(getMin)
          println(num)
        }
        def getMin(n1:Int,n2:Int):Int={
          if(n1>n2) n2 else n1
        }
      }
      ~~~

#### 7.2.5，折叠

- 基本介绍

  fold函数将上一步返回的值作为函数的第一个参数继续传递参与运算，直到list中的所有元素被遍历。

-  可以把reduceLeft看做简化版的foldLeft. 源码如下

  ~~~ java
  def reduceLeft[B >: A](@deprecatedName('f) op: (B, A) => B): B =    if (isEmpty) throw new UnsupportedOperationException("empty.reduceLeft")    else tail.foldLeft[B](head)(op)
  ~~~

  大家可以看到. reduceLeft就是调用的foldLeft[B](head)，并且是默认从集合的head元素开始操作的。

- 相关函数：fold，foldLeft，foldRight，可以参考reduce的相关方法理解

  ~~~ java
  package qq.com.rzf.collection
  
  object Zhedie {
    def main(args: Array[String]): Unit = {
      /**
       * 折叠的理解和化简运行机制一样
       * list.foldLeft(5)(minus)理解为List(5，1, 2, 3, 4) ,list.reducer(minus)
       * 即把head元素添加到list集合的头部，在做运算
       * 1,(5,1)
       * 2,((5,1),2)
       * 以此类推
       */
      val list = List(1, 2, 3, 4)
      def minus( num1 : Int, num2 : Int ): Int = {
        num1 - num2}
      println(list.foldLeft(5)(minus))
  
      /**
       * 右折叠：可以看成List(1, 2, 3, 4，5) ,list.reducer(minus)
       */
      println(list.foldRight(5)(minus))
  
    }
  }
  
  ~~~

- foldLeft和foldRight 缩写方法分别是：/:和:\

  ~~~ java
  val list4 = List(1, 9, 2, 8)
  def minus(num1: Int, num2: Int): Int = { 
      num1 - num2
  }
  var i6 = (1 /: list4) (minus)
  println(i6) 
  
  i6 = (100 /: list4) (minus) 
  println(i6) 
  
  i6 = (list4 :\ 10) (minus) 
  println(i6) 
  ~~~

#### 7.2.6，扫描

- 基本介绍

  扫描，即对某个集合的所有元素做fold操作，但是会把产生的所有中间结果放置于一个集合中保存

  ~~~ java
  package qq.com.rzf.collection
  
  object MyScan {
    def main(args: Array[String]): Unit = {
      def minus( num1 : Int, num2 : Int ) : Int = {
        num1 - num2
      }
  
      /**
       * (1 to 5).scanLeft(5)(minus)标示：5(1,2,3,4,5)
       * 每一次计算的结果保存在集合当中
       * 5-1=4  (4)
       * 4-2=2  (4,2)
       * 2-3=-1 (4,2,-1)
       * -1-4=-5  (4,2,-1,-5)
       * -5-5=-10 (4,2,-1,-5,-10)
       * 最后返回集合：(4,2,-1,-5,-10)
       */
      val i8 = (1 to 5).scanLeft(5)(minus)
      //最后返回的i8是indexseq类型
      println(i8)
  
      def add( num1 : Int, num2 : Int ) : Int = {
        num1 + num2
      }
  
      /**
       * (1 to 5).scanRight(5)(minus)标示：(1,2,3,4,5)5
       */
      val i9 = (1 to 5).scanRight(5)(add)
      println(i9)
  
    }
  }
  ~~~


1. 练习

   1. val sentence = "AAAAAAAAAABBBBBBBBCCCCCDDDDDDD"，将sentence 中各个字符，通过foldLeft存放到 一个ArrayBuffer中
      目的：理解flodLeft的用法.

      ~~~ java
      package qq.com.rzf.collection
      
      import scala.collection.mutable.ArrayBuffer
      
      object Exercise01 {
        def main(args: Array[String]): Unit = {
          val sentence = "AAAAAAAAAABBBBBBBBCCCCCDDDDDDD"
          val arrayBuffer=new ArrayBuffer[Char]()
          //左折叠必须传入头在左边作为参数
          sentence.foldLeft(arrayBuffer)(putArray)
          println(arrayBuffer)
      
      
        }
        def putArray(arr:ArrayBuffer[Char],c:Char): ArrayBuffer[Char] ={
          arr.append(c)
          arr
        }
      }
      ~~~

   2. val sentence = "AAAAAAAAAABBBBBBBBCCCCCDDDDDDD",使用映射集合，统计一句话中，各个字母出现的次数

      ~~~ java
      package qq.com.rzf.collection
      
      import scala.collection.mutable
      
      object WordCount {
        def main(args: Array[String]): Unit = {
          val sentence = "AAAAAAAAAABBBBBBBBCCCCCDDDDDDD"
          var map= Map[Char,Int]()//存放中间结果,但是是不可变的map,所以拿不到结果
          var map1=sentence.foldLeft(map)(wordCount)
          println(map1)//Map(A -> 10, B -> 8, C -> 5, D -> 7)
          //可变的map
          var map2= mutable.Map[Char,Int]()//存放中间结果,这里是可变的map,可以直接拿到中间结果
          //整个过程只对map2一个集合做操作
         sentence.foldLeft(map2)(wordCount1)
          println(map2)//Map(D -> 7, A -> 10, C -> 5, B -> 8)
        }
        //不可变的map
        def wordCount(map:Map[Char,Int],c:Char):Map[Char,Int]={
          //对每一个字符，首先判断其是否存在于map集合中，如果不存在，把字符添加到map中
          //如果存在，取出值做更新操作
          map+(c -> (map.getOrElse(c,0)+1))
        }
        //可变的map,效率高，map是无序不重复
        def wordCount1(map:mutable.Map[Char,Int], c:Char):mutable.Map[Char,Int]={
          //对每一个字符，首先判断其是否存在于map集合中，如果不存在，把字符添加到map中
          //如果存在，取出值做更新操作
          map +=(c -> (map.getOrElse(c,0)+1))
        }
      }
      
      ~~~

   3. 大数据中经典的wordcount案例

      - val lines = List("atguigu liu hello ddd", "atguigu liu aaa aaa aaa ccc ddd uuu“, “liu”,”ddd”)
        使用映射集合，list中，各个单词出现的次数，并按出现次数排序

        ~~~java
        
        ~~~

#### 7.2.7，扩展-拉链(合并)

- 基本介绍
  在开发中，当我们需要将两个集合进行 对偶元组合并，可以使用拉链。

  ~~~ java
  package qq.com.rzf.collection
  
  object Myzip {
    def main(args: Array[String]): Unit = {
     val list1 = List(1, 2 ,3)
      val list2 = List(4, 5, 6)
      val list3 = list1.zip(list2)
      println("list3=" + list3)//list3=List((1,4), (2,5), (3,6))
      val list4 = list2.zip(list1)
      println("list3=" + list4)//list3=List((4,1), (5,2), (6,3))
  
    }
  }
  ~~~

- 注意事项

  - 拉链的本质就是两个集合的合并操作，合并后每个元素是一个 对偶元组。

  - 操作的规则下图:

    ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/202433-83584.png)

  - 如果两个集合个数不对应，会造成数据丢失。

  - 集合不限于List, 也可以是其它集合比如 Array

  - 如果要取出合并后的各个对偶元组的数据，可以遍历

    ~~~ java
    for (item<-list3) {
        print(item._1 + " " + item._2) //取出时，按照元组的方式取出即可   
    }
    ~~~

#### 7.2.8，扩展-迭代器

- 基本说明
  通过iterator方法从集合获得一个迭代器，通过while循环或for表达式对集合进行遍历.

  ~~~ java
  val iterator1 = List(1, 2, 3, 4, 5).iterator // 得到迭代器
  println("--------遍历方式1 -----------------")
  while (iterator1.hasNext) {
      println(iterator1.next())
  }
  
  println("--------遍历方式2 for -----------------")
  val iterator2 = List(11, 22, 33, 44, 55).iterator // 得到迭代器
  for(enum <- iterator2) {
      println(enum) //
  }
  ~~~

- 小结

  -   iterator 的构建实际是 AbstractIterator 的一个匿名子类，该子类提供了以下方法。

    ~~~ java
    def iterator: Iterator[A] = new AbstractIterator[A] {
            var these = self
            def hasNext: Boolean = !these.isEmpty
            def next(): A =
    
    ~~~

  - 该AbstractIterator 子类提供了  hasNext  next 等方法.

  - 因此，我们可以使用 while的方式，使用hasNext和next方法遍历

#### 7.2.9，扩展-流 Stream

- 基本说明
  stream是一个集合。这个集合，可以用于存放无穷多个元素，**但是这无穷个元素并不会一次性生产出来，而是需要用到多大的区间，就会动态的生产，末尾元素遵循lazy规则(即：要使用结果才进行计算的) 。**

1. 创建Stream对象

   ~~~ java
   def numsForm(n: BigInt) : Stream[BigInt] = n #:: numsForm(n + 1)
   val stream1 = numsForm(1)
   ~~~

   - 说明
     Stream 集合存放的数据类型是BigInt
     numsForm 是自定义的一个函数，函数名是程序员指定的。
     创建的集合的第一个元素是 n , 后续元素生成的规则是 n + 1
     后续元素生成的规则是可以程序员指定的 ，比如 numsForm( n * 4)...

2. 使用tail，会动态的向stream集合按规则生成新的元素

   ~~~ java
   //创建Stream
   def numsForm(n: BigInt) : Stream[BigInt] = n #:: numsForm(n + 1)
   val stream1 = numsForm(1)
   println(stream1) 
   println("head=" + stream1.head) 
   println(stream1.tail) //需要用到tail的时候才产生
   println(stream1)
   //注意：如果使用流集合，就不能使用last属性，如果使用last集合就会进行无限循环
   ~~~

3. 使用map映射stream的元素并进行一些计算

   ~~~ java
   //创建Stream
   def numsForm(n: BigInt) : Stream[BigInt] = n #:: numsForm(n + 1)
   
   def multi(x:BigInt) : BigInt = {
       x * x
   }
   println(numsForm(5).map(multi))
   ~~~

#### 7.2.10，扩展-视图 View(优化手段：推迟执行)

- 基本介绍
  Stream的懒加载特性，也可以对其他集合应用view方法来得到类似的效果，具有如下特点：
  view方法产出一个总是被懒执行的集合。view产生的结果是一个集合。
  view不会缓存数据，每次都要重新计算，比如遍历View时。

- 应用案例
  请找到1-100 中，数字倒序排列 和它本身相同的所有数。(1, 2, 3, 4, 5, 6, 7, 8, 9, 11, 22, 33 ...)

  ~~~ java
  package qq.com.rzf.collection
  
  object MyView {
    def main(args: Array[String]): Unit = {
      def multiple(num: Int): Int = {
        num
      }
  
      def eq(i: Int): Boolean = {
        println("eq方法被调用了")
        i.toString.equals(i.toString.reverse)
      }
  
      //说明: 没有使用view
      val viewSquares1 = (1 to 100).map(multiple).filter(eq)
      println(viewSquares1)
  
      //使用view
      //在这里其实打印viewSquares2结果的时候没有执行eq函数，而是在遍历
      //的时候才加载函数进行执行
      val viewSquares2 = (1 to 100).view.map(multiple).filter(eq)
      //eq函数被调用100次，filter进行过滤操作仅仅过滤复合条件的值
      println(viewSquares2)
      for(item <- viewSquares2)
        println(item)
  
    }
  }
  ~~~

#### 7.2.11，扩展-并行集合

- 基本介绍

  所有线程安全的集合都是以Synchronized开头的集合

  ~~~ java
  SynchronizedBuffer
  SynchronizedMap
  SynchronizedPriorityQueue
  SynchronizedQueue
  SynchronizedSet
  SynchronizedStack
  ~~~

- 基本介绍

  Scala为了充分使用多核CPU，提供了并行集合（有别于前面的串行集合），用于多核环境的并行计算。
  主要用到的算法有： Divide and conquer : 分治算法，Scala通过splitters(分解器)，combiners（合并集）等抽象层来实现，主要原理是将计算工作分解很多任务，分发给一些处理器去完成，并将它们处理结果合并返回Work stealin算法，主要用于任务调度负载均衡（load-balancing），通俗点完成自己的所有任务之后，发现其他人还有活没干完，主动（或被安排）帮他人一起干，这样达到尽早干完的目的

- 应用案例 
  parallel(pærəlel 并行)

- 打印1~5

  ~~~ java
  package qq.com.rzf.collection
  
  object ParDemo {
    def main(args: Array[String]): Unit = {
      (1 to 5).foreach(println(_))//标示把遍历1-5中的数字，然后作为参数传递给println函数
      //等价于下面的写法，有序输出
      (1 to 5).foreach(println)//foreach函数会把1-5遍历，然后传递给println函数
      println()
      (1 to 5).par.foreach(println(_))//无序输出，标示把println的输出分配到不同的cpu执行
  
    }
  }
  ~~~

- 查看并行集合中元素访问的线程

  ~~~ java
  val result1 = (0 to 100).map{case _ => Thread.currentThread.getName}
  val result2 = (0 to 100).par.map{case _ => Thread.currentThread.getName}
  println(result1)
  println(result2)
  ~~~

#### 7.2.12，扩展-操作符

1. 基本介绍
   这部分内容没有必要刻意去理解和记忆，语法使用的多了，自然就会熟练的使用，该部分内容了解一下即可。

   1. 操作符扩展
      如果想在变量名、类名等定义中使用语法关键字（保留字），可以配合反引号反引号(飘号)  val `val` = 42

   2. 中置操作符：A 操作符 B 等同于 A.操作符(B)  

      ~~~java
      package qq.com.rzf.collection
      
      object Operator {
        def main(args: Array[String]): Unit = {
          val n1 = 1
          val n2 = 2
          //以下两种写法等同
          val r1 = n1 + n2
          val r2 = n1.+(n2)
          println("r1=" + r1 + " r2=" + r2)
      
        }
      }
      ~~~

   3. 后置操作符：A操作符 等同于 A.操作符，如果操作符定义的时候不带()则调用时不能加括号 

      ~~~ java
      //后置操作符
         package qq.com.rzf.collection
      
      object Operator {
        def main(args: Array[String]): Unit = {
          val n1 = 1
          val n2 = 2
          //以下两种写法等同
          val r1 = n1 + n2
          val r2 = n1.+(n2)
          println("r1=" + r1 + " r2=" + r2)
          var mon=new Monster
          mon+10
          println(mon.money)
          //中置操作符
          mon.+(20)
          println(mon.money)
          //后置操作符
          mon++10
          println(mon.money)
          //取反操作
          !mon
          println(mon.money)
        }
      }
      class Monster{
        var money:Int= _
        //中置操作符
        def +(n:Int):Unit={
          this.money+=n
        }
        //后置操作符
        def ++(m:Int):Unit={
          this.money+=m
        }
        //前置操作符，一元运算符
        def unary_!():Unit={
          this.money = -this.money
        }
      
      }
      ~~~

   4. 前置操作符，+、-、！、~等操作符A等同于A.unary_操作符

      ~~~ java
      //取反操作
        package qq.com.rzf.collection
      
      object Operator {
        def main(args: Array[String]): Unit = {
          val n1 = 1
          val n2 = 2
          //以下两种写法等同
          val r1 = n1 + n2
          val r2 = n1.+(n2)
          println("r1=" + r1 + " r2=" + r2)
          var mon=new Monster
          mon+10
          println(mon.money)
          //中置操作符
          mon.+(20)
          println(mon.money)
          //后置操作符
          mon++10
          println(mon.money)
          //取反操作
          !mon
          println(mon.money)
        }
      }
      class Monster{
        var money:Int= _
        //中置操作符
        def +(n:Int):Unit={
          this.money+=n
        }
        //后置操作符
        def ++(m:Int):Unit={
          this.money+=m
        }
        //前置操作符，一元运算符
        def unary_!():Unit={
          this.money = -this.money
        }
      
      }
      ~~~

   5. 赋值操作符，A 操作符= B 等同于 A = A 操作符 B  ，比如 A += B 等价 A = A + B

## 第八章，模式匹配（`match`)

### 8.1，match

1. 基本介绍

   - Scala中的模式匹配类似于Java中的switch语法，但是更加强大。

   - 模式匹配语法中，采用match关键字声明，每个分支采用case关键字进行声明，当需要匹配时，会从第一个case分支开始，如果匹配成功，那么执行对应的逻辑代码，如果匹配不成功，继续执行下一个分支进行判断。如果所有case都不匹配，那么会执行case _ 分支，类似于Java中default语句。

   - _:三层含义：默认值，函数赋值给变量，case中默认

2. Scala的模式匹配

   ~~~ java
   package qq.com.rzf.collection
   
   object MyMatch {
     def main(args: Array[String]): Unit = {
       val oper = '#'
       val n1 = 20
       val n2 = 10
       var res = 0
   
       /**
        * 说明：match类似于java中的switch语句
        * 如果匹配成功，就执行=>后面的代码，=>后面可以是代码块
        * 匹配的顺序是从上到下，匹配到一个就执行后面的代码块
        * =>后面不需要break语句，会自动的结束
        * 如果一个都没有匹配到，就执行case _代码块
        *
        **/
       oper match {
         case '+' => res = n1 + n2
         case '-' => res = n1 - n2
         case '*' => res = n1 * n2
         case '/' => res = n1 / n2
         case _ => println("oper error")
       }
       println("res=" + res)
   
     }
   }
   ~~~

3. match的细节和注意事项

   1. 如果所有case都不匹配，那么执行case _ 分支，类似于Java中default语句

   2. 如果所有case都不匹配，又没有写case _ 分支，那么会抛出MatchError

   3. 每个case中，不用break语句，自动中断case

   4. 可以在match中使用其它类型，而不仅仅是字符,可以是表达式
   5. => 类似于 java swtich 的 :
   6. => 后面的代码块到下一个case, 是作为一个整体执行，可以使用{} 括起来，也可以不括。 

4. 守卫

   - 基本介绍
     如果想要表达匹配某个范围的数据，就需要在模式匹配中增加条件守卫

     ~~~java
     package qq.com.rzf.collection
     
     object MyMatch01 {
       def main(args: Array[String]): Unit = {
         for(ch <- "db66c")
           {
             ch match {
               case 'd' =>println("匹配到字符d")
               case 'b'=>println("匹配到字符b")
                 //如果case后面有守卫即if语句，那么标示不是默认匹配
                 //标示忽略传入的ch
                 //if后面可以添加复合条件
               case _ =>if(ch>10)println("ch:"+ch)
               case _ =>if(ch.toString.eq("c"))println("字符串c")
            }
           }
       }
     }
     ~~~

5. 练习

   ~~~java
   for (ch <- "+-3!") {
       var sign = 0
       var digit = 0
       ch match {
           case '+' => sign = 1
           case '-' => sign = -1
           case _  => digit = 3//有多个默认匹配，但是仅第一个默认匹配会生效，后面的会失效
           case _  => sign = 2
       }
       println(ch + " " + sign + " " + digit)
   }
   or (ch <- "+-3!") {
       var sign = 0
       var digit = 0
       ch match {
           case _  => digit = 3 //全部默认匹配，不会看后面的匹配
           case '+' => sign = 1
           case '-' => sign = -1
       }
       println(ch + " " + sign + " " + digit)
   }
   //如果在匹配中有if条件匹配，那么如果没有case _默认匹配，那么执行的时候会报错。
   ~~~

### 8.2，模式中的变量及返回值

- 基本介绍
  如果在case关键字后跟变量名，那么match前表达式的值会赋给那个变量

  ~~~ java
  package qq.com.rzf.collection
  
  object MatchCh {
    def main(args: Array[String]): Unit = {
      var ch='+'
      //把匹配到的结果返回给res
      //match是一个表达式，因此可以由返回值
      //返回值就是匹配代码块的最后一句话的值
      var res=ch match {
        case '+'=> println("hello")
          //下面句子的含义是mychar=ch
        case mychar =>println("ok"+mychar)
  
      }
     println(res)
    }
  }
  ~~~

### 8.3，类型匹配

- 基本介绍
  可以匹配对象的任意类型，这样做避免了使用isInstanceOf和asInstanceOf方法

- 案例：

  ~~~ java
  package qq.com.rzf.collection
  
  object Match01 {
    def main(args: Array[String]): Unit = {
      // 类型匹配, obj 可能有如下的类型
      val a = 8
      val obj = if(a == 1) 1
      else if(a == 2) "2"
      else if(a == 3) BigInt(3)
      else if(a == 4) Map("aa" -> 1)
      else if(a == 5) Map(1 -> "aa")
      else if(a == 6) Array(1, 2, 3)
      else if(a == 7) Array("aa", 1)
      else if(a == 8) Array("aa")
      /**
       * 根据obj类型进行匹配， case a : Int => a标示把obj赋值给a,然后判断类型是否是int,
       * 是的话就返回对象
       */
      val result = obj match {
        case a : Int => a
        case b : Map[String, Int] => "对象是一个字符串-数字的Map集合"
        case c : Map[Int, String] => "对象是一个数字-字符串的Map集合"
        case d : Array[String] =>d //"对象是一个字符串数组"
        case e : Array[Int] => "对象是一个数字数组"
        case f : BigInt => Int.MaxValue
        case _ => "啥也不是"
      }
      println(result)
  
    }
  }
  
  ~~~

- 类型匹配注意事项

  - Map[String, Int] 和Map[Int, String]是两种不同的类型，其它类推。list[int]和list[string]也是不同的类型。

  - 在进行类型匹配时，编译器会预先检测是否有可能的匹配，如果没有则报错。

    ~~~ java
    val obj = 10//obj是int类型，已经写死
    val result = obj match {
        case a : Int => a
        case b : Map[String, Int] => "Map集合" //报错，obj不可能是string类型，提前编译
        case _ => "啥也不是"
    }
    
    ~~~

  - 一个说明:

    ~~~ java
    val result = obj match {    
        case i : Int => i
    }  //case i : Int => i 表示 将 i = obj (其它类推)，然后再判断类型
    ~~~

  - 如果 case _ 出现如下情况(忽略匹配的变量值)，则表示隐藏变量名，即不使用,而不是表示默认匹配。

    ~~~java
    val a = 7
    val obj = if(a == 1) 1
        else if(a == 2) "2"
        else if(a == 3) BigInt(3)
        else if(a == 4) Map("aa" -> 1)
        else if(a == 5) Map(1 -> "aa")
        else if(a == 6) Array(1, 2, 3)
        else if(a == 7) Array("aa", 1)
        else if(a == 8) Array("aa")
    val result = obj match {
        case a : Int => a
        case _ : BigInt => Int.MaxValue 
        case b : Map[String, Int] => "一个字符串-数字的Map集合"
        case c : Map[Int, String] => "一个数字-字符串的Map集合"
        case d : Array[String] => "一个字符串数组"
        case e : Array[Int] => "一个数字数组"
        case _ => "啥也不是"
    }
    println(result)
    
    ~~~

### 8.4，数组匹配

- 基本介绍
  Array(0) 匹配只有一个元素且为0的数组。
  Array(x,y) 匹配数组有两个元素，并将两个元素赋值为x和y。当然可以依次类推Array(x,y,z) 匹配数组有3个元素的等等....
  Array(0,_*) 匹配数组以0开始  

- 案例：

  ~~~java
  package qq.com.rzf.collection
  
  object ArrayMatch {
    def main(args: Array[String]): Unit = {
      var arrs=Array(Array(0),Array(1,0),Array(0,1,0),Array(1,2,0),Array(1,1,0,1))
      for(item <- arrs){
        val res=item match {
          case Array(0) => "0"
          case Array(x, y) => x + "=" + y
          case Array(0, _*) => "以0开头和数组"
          case _ => "什么集合都不是"
  
        }
        println("result = " + res)
      }
  
  
    }
  }
  ~~~

### 8.5，匹配列表

~~~ java
package qq.com.rzf.collection

object ListMatch {
  def main(args: Array[String]): Unit = {
    for (list <- Array(List(0), List(1, 0), List(88),List(0, 0, 0), List(1, 0, 0))) {
      val result = list match {
        case 0 :: Nil => "0" //匹配list(0),Nil标示list(),把0元素添加到列表
        case x :: y :: Nil => x + " " + y//匹配list(x,y)
        case 0 :: tail => "0 ..."//List(0, 0, 0)匹配
          //在匹配list(88),并且返回列表中的值
        case x ::  Nil=> x
        case _ => "something else"
      }
      println(result)
    }
  }
}
~~~

### 8.6，匹配元组

~~~ java
package qq.com.rzf.collection

object YuanzuMatch {
  def main(args: Array[String]): Unit = {
    for (pair <- Array((0, 1), (1, 0), (2, 1),(1,0,2))) {
      val result = pair match {
        case (0, _) => "0 ..."//匹配以0开头的二元组，而且第二个元素忽略不要
        case (y, 0) => y//二元组第二个元组必须是0
        case (a,b) => (b,a)//匹配对偶元组
        case _ => "other"//默认其他
      }
      println(result)
    }
  }
}
~~~

### 8.7，对象匹配

1. 基本介绍
   对象匹配，什么才算是匹配呢？，规则如下:
   case中对象的unapply方法(对象提取器)返回Some集合则为匹配成功
   返回none集合则为匹配失败

   ~~~ java
   package qq.com.rzf.collection
   
   object MatchObject {
     def main(args: Array[String]): Unit = {
   
   
       // 模式匹配使用：
       val number: Double = 36.0
       number match {
         /**
          * Square(n) 运行机制说明
          * 1，当匹配到Square(n)的时候
          * 2，首先调用Square的apply(z: Double)，其中z的值就是number的值
          * 3，如果对象提取器apply(z: Double)返回some(6),则表示匹配成功，同时系统将some中的6返回给
          * Square(n)中的n
          *
          */
         case Square(n) => println("匹配成功"+n)//主要看对象提取器返回some还是none
         case _ => println("nothing matched")
       }
   
     }
     object Square {
       /**
        * 接受一个double类型，返回一个Option[Double]类型
        * 返回的值是：Some(math.sqrt(z))，返回z的开平方，并放入到some集合中
        *
        */
       def unapply(z: Double): Option[Double] = Some(math.sqrt(z))//对象提取器
       def apply(z: Double): Double = z * z//new一个对象，相当于构造器
     }
   }
   
   ~~~

   - 应用案例1的小结

     构建对象时apply会被调用 ，比如 val n1 = Square(5)

     当将 Square(n) 写在 case 后时[case Square(n) => xxx]，会默认调用unapply 方法(对象提取器)

     number 会被 传递给def unapply(z: Double) 的 z 形参

     如果返回的是Some集合，则unapply提取器返回的结果会返回给n这个形参

     case中对象的unapply方法(提取器)返回some集合则为匹配成功

     返回none集合则为匹配失败


### 8.8，变量声明中的模式

- 基本介绍
  match中每一个case都可以单独提取出来，意思是一样的.

  ~~~ java
  package qq.com.rzf.collection
  
  object VariableMatch {
    def main(args: Array[String]): Unit = {
      //表示声明两个变量，x=1,y=2
      val (x, y) = (1, 2)
  //声明两个变量，r=q=BigInt(10) /% 3
      val (q, r) = BigInt(10) /% 3  // 包含了2个连续的运算符
      println("q = " + q)
      println("r = " + r)
  
      val arr = Array(1, 7, 2, 9)
      //表示把arr数组中第一个元素给first,第二个给second,其他给别的
      val Array(first, second, _*) = arr//提取arr中前两个元素
      println(first, second)
  
    }
  }
  
  ~~~

### 8.9，for表达式中的模式

- 基本介绍
  for循环也可以进行模式匹配.

- 案例

  ~~~ java
  package qq.com.rzf.collection
  
  object MatchFor {
    def main(args: Array[String]): Unit = {
      val map = Map("A"->1, "B"->0, "C"->3)
      //依次遍历map
      for ( (k, v) <- map ) {
        println(k + " -> " + v)
      }
  //遍历map,只赛选value=0的键值对
      for ((k, 0) <- map) {
        println(k + " --> " + 0)
      }
  //是第二种写法的等价
      for ((k, v) <- map if v == 0) {
        println(k + " ---> " + v)
      }
  
    }
  }
  ~~~

### 8.10，样例类

1. 实例

   ~~~ java
   package qq.com.rzf.telp
   
   object CaseClass {
     def main(args: Array[String]): Unit = {
       println("hello")
     }
   }
   //空的抽象类，可以不带{}，应为是空的
   abstract class Amount
   case class Dollar(value: Double) extends Amount
   case class Currency(value: Double, unit: String) extends Amount
   case object NoAmount extends Amount
   ~~~

   - 每一个样例类在底层会生成两个类

     ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/202500-440498.png)

   - 说明: 这里的 Dollar，Currencry, NoAmount  是样例类。

   - 可以这样理解样例类，就是样例类会默认其它很多的方法，供程序员直接使用 

   - 底层代码说明

     ~~~ java
     //Currency样例类
     public class Currency extends Amount
       implements Product, Serializable
     {
     //构造函数的两个参数在底层会变为只读的属性
       private final double value;
       private final String unit;
     //unapply方法
       public static Option<Tuple2<Object, String>> unapply(Currency paramCurrency)
       {
         return Currency..MODULE$.unapply(paramCurrency);
       }
     //apply方法
       public static Currency apply(double paramDouble, String paramString)
       {
         return Currency..MODULE$.apply(paramDouble, paramString);
       }
     
       public static Function1<Tuple2<Object, String>, Currency> tupled()
       {
         return Currency..MODULE$.tupled();
       }
     
       public static Function1<Object, Function1<String, Currency>> curried()
       {
         return Currency..MODULE$.curried();
       }
     //只读方法
       public double value()
       {
         return this.value; } 
       public String unit() { return this.unit; } 
       public Currency copy(double value, String unit) { return new Currency(value, unit); } 
       public double copy$default$1() { return value(); } 
       public String copy$default$2() { return unit(); } 
       public String productPrefix() { return "Currency"; } 
       public int productArity() { return 2; } 
       public Object productElement(int x$1) { int i = x$1; switch (i) { default:
           throw new IndexOutOfBoundsException(BoxesRunTime.boxToInteger(x$1).toString());
         case 1:
           break;
         case 0: } return BoxesRunTime.boxToDouble(value()); } 
       public Iterator<Object> productIterator() { return ScalaRunTime..MODULE$.typedProductIterator(this); } 
       public boolean canEqual(Object x$1) { return x$1 instanceof Currency; } 
       public int hashCode() { int i = -889275714; i = Statics.mix(i, Statics.doubleHash(value())); i = Statics.mix(i, Statics.anyHash(unit())); return Statics.finalizeHash(i, 2); } 
       public String toString() { return ScalaRunTime..MODULE$._toString(this); } 
       public boolean equals(Object x$1) { if (this != x$1) { Object localObject = x$1;
           int i;
           if ((localObject instanceof Currency)) i = 1; else i = 0; if (i == 0) break label97; Currency localCurrency = (Currency)x$1; if (value() == localCurrency.value()) { str = localCurrency.unit();
             String tmp55_45 = unit(); if (tmp55_45 == null) { tmp55_45; if (str == null) break label76; tmpTernaryOp = tmp55_45; break label89; }  }  }  } 
       public Currency(double value, String unit) { Product.class.$init$(this); }
     
     }
     //真正定义apply和unapply方法的地方
     import scala.None.;
     import scala.Option;
     import scala.Serializable;
     import scala.Some;
     import scala.Tuple2;
     import scala.runtime.AbstractFunction2;
     import scala.runtime.BoxesRunTime;
     
     public final class Currency$ extends AbstractFunction2<Object, String, Currency>
       implements Serializable
     {
       public static final  MODULE$;
     
       static
       {
         new ();
       }
     
       public final String toString()
       {
         return "Currency"; } 
       public Currency apply(double value, String unit) { return new Currency(value, unit); } 
       public Option<Tuple2<Object, String>> unapply(Currency x$0) { return x$0 == null ? None..MODULE$ : new Some(new Tuple2(BoxesRunTime.boxToDouble(x$0.value()), x$0.unit())); } 
       private Object readResolve() { return MODULE$; } 
       private Currency$() { MODULE$ = this; }
     }
     ~~~

2. 基本介绍

   1. 样例类仍然是类
   2. 样例类用case关键字进行声明。
   3. 样例类是为模式匹配(对象)而优化的类
   4. 构造器中的每一个参数都成为val——除非它被显式地声明为var
   5. 在样例类对应的伴生对象中提供apply方法让你不用new关键字就能构造出相应的对象
   6. 提供unapply方法让模式匹配可以工作
   7. 将自动生成toString、equals、hashCode和copy方法(有点类似模板类，直接给生成，供程序员使用)
      除上述外，样例类和其他类完全一样。你可以添加方法和字段，扩展它们

3. 案例

   - 当我们有一个类型为Amount的对象时，可以用模式匹配来匹配他的类型，并将属性值绑定到变量(即：把样例类对象的属性值提取到某个变量)

   ~~~ java
   package qq.com.rzf.telp
   
   object CaseClass01 {
     def main(args: Array[String]): Unit = {
       for (amt <- Array(Dollar(1000.0), Currency(1000.0, "RMB"), NoAmount)) {
         val result = amt match {
             //此处调用unapply()方法
           case Dollar(v) => "$" + v
           case Currency(v, u) => v + " " + u
           case NoAmount => ""
         }
         println(amt + ": " + result)
       }
   
     }
   }
   //空的抽象类，可以不带{}，应为是空的
   abstract class Amount
   case class Dollar(value: Double) extends Amount
   case class Currency(value: Double, unit: String) extends Amount
   case object NoAmount extends Amount
   
   ~~~

4. 样例类的copy方法和带名参数
   创建一个与现有对象值相同的新对象 ，并可以通过带名参数来修改某些属性。

   ~~~ java
   package qq.com.rzf.telp
   
   object CaseClass02 {
     def main(args: Array[String]): Unit = {
       var tmp=new Currency1(5000,"rmb")
       //实现对象的克隆
       var tmp1=tmp.copy()
       println(tmp1)//Currency1(5000.0,rmb)
       //修改参数值
       var tmp2=tmp.copy(8000)
       println(tmp2)//Currency1(8000.0,rmb)
     }
   }
   abstract class Amount1
   case class Dollar1(value: Double) extends Amount1
   case class Currency1(value: Double, unit: String) extends Amount1
   case object NoAmount1 extends Amount1
   ~~~

5. case语句的中置(缀)表达式

   - 基本介绍
     什么是中置表达式？1 + 2，这就是一个中置表达式。如果unapply方法产出一个元组，你可以在case语句中使用中置表示法。比如可以匹配一个List序列

     ~~~java
     package qq.com.rzf.telp
     
     object zhongzhibiaodashi {
       def main(args: Array[String]): Unit = {
         var list = List(1, 3, 5, 9)
         list match {
           /**
            * 两个元素之间：：叫做中置表达式，至少first和second两个匹配才可以
            * first匹配第一个，second匹配第二个，rest匹配5,9
            */
           case first :: second :: rest => println(first + "**"+ second + "**" + rest)
           case _ => println("匹配不到...")
         }
     
       }
     }
     ~~~




## 第九章，函数式编程高级

### 9.1，先看一个需求

- 给你一个集合val list = List(1, 2, 3, 4, "abc") ，请完成如下要求:
  将集合list中的所有数字+1，并返回一个新的集合 
  要求忽略掉 非数字 的元素，即返回的 新的集合 形式为 (2, 3, 4, 5)

- 案例解法：

  ~~~ java
  package qq.com.rzf.partial
  
  object PartialFun {
    def main(args: Array[String]): Unit = {
      /**
       * 给你一个集合val list = List(1, 2, 3, 4, "abc") ，请完成如下要求:
       * 将集合list中的所有数字+1，并返回一个新的集合
       * 要求忽略掉 非数字 的元素，即返回的 新的集合 形式为 (2, 3, 4, 5)
       *这种方式解决问题比较麻烦
       */
      val list = List(1, 2, 3, 4, "abc")
      //，思路一：先过滤，在进行map操作
      println(list.filter(f1).map(f2).map(f3))
      //方式二：模式匹配
      println(list.map(f4))//List(2, 3, 4, 5, ())输出不合题意
  
    }
    //模式匹配
    def f4(n:Any):Any={
      n match{
        case n:Int => n+1
        case _ =>
      }
    }
    //过滤操作
    def f1(n:Any):Boolean={
      n.isInstanceOf[Int]//如果是int类型，就返回true
    }
    //将any类型转换为int类型
    def f2(n:Any):Int={
      n.asInstanceOf[Int]
    }
    //map进行逻辑操作
    def f3(n:Int):Int={
      n+1
    }
  }
  ~~~

### 9.2，偏函数

- 基本介绍
  在对符合某个条件，而不是所有情况进行逻辑操作时，使用偏函数是一个不错的选择
  将包在大括号内的一组case语句封装为函数，我们称之为偏函数，它只对会作用于指定类型的参数或指定范围值的参数实施计算，超出范围的值会忽略.
  偏函数在Scala中是一个特质PartialFunction

  ~~~ java
  package qq.com.rzf.partial
  
  object PartialFun01 {
    def main(args: Array[String]): Unit = {
      //定义一个偏函数
      /**
       * PartialFunction[Any,Int]标示此偏函数接收一个any类型参数，返回int类型
       */
      var pf=new PartialFunction[Any,Int] {
        //如果次函数的条件满足被触发，就会调用apply函数构建对象实例，如果返回false,不执行apply()函数
        //
        override def isDefinedAt(x: Any): Boolean = {
          if(x.isInstanceOf[Int])
            true
          else
            false
        }
        //apply函数写业务逻辑
        //构造器，对传入的值进行+1操作，并返回到新的集合中去
        override def apply(v1: Any): Int = {
          v1.asInstanceOf[Int]+1
        }
      }
      var list=List(1,2,3,4,"hello")
      //使用偏函数
      //map不支持偏函数，偏函数使用collect,偏函数也是一个对象
      /**
       * 偏函数的执行流程：
       * 1，在底层对list集合的元素进行遍历，isDefinedAt调用次数是集合的大小
       * 2，apply()函数调用的次数与规定的条件有关
       * 3，isDefinedAt()函数如果返回true,就调用apply方法，否则过滤掉刚才的数值，不调用apply()方法
       * 4，返回的值取决于apply()的业务逻辑
       * 5，每次得到一个新的数值，apply函数会把他放到新的集合中返回
       */
      println(list.collect(pf))//List(2, 3, 4, 5)
  
    }
  }
  ~~~

- 偏函数的小结

  - 使用构建特质的实现类(使用的方式是PartialFunction的匿名子类)
  -  PartialFunction 是个特质(看源码)
  -  构建偏函数时，参数形式   [Any, Int]是泛型，第一个表示传入参数类型，第二个  表示返回参数
  - 当使用偏函数时，会遍历集合的所有元素，编译器执行流程时先执行isDefinedAt()如果为true ,就会执行 apply, 构建一个新的Int 对象返回
  - 执行isDefinedAt() 为false 就过滤掉这个元素，即不构建新的Int对象.
  - map函数不支持偏函数，因为map底层的机制就是所有循环遍历，无法过滤处理原来集合的元素
  - collect函数支持偏函数

- 偏函数简化形式
  声明偏函数，需要重写特质中的方法，有的时候会略显麻烦，而Scala其实提供了简单的方法

  ~~~ java
  package qq.com.rzf.partial
  
  object SimplePartial {
    def main(args: Array[String]): Unit = {
      //第一种简化形式
      //在这里也可以写隐式函数将浮点型转化成整形类型
      //偏函数的简化形式
      def f:PartialFunction[Any,Int]={
        case i:Int => i+1
          //此处不是完全的模式匹配，不需要有默认语句
        //如果是小数类型，就先*2然后转换成int类型
        case j:Double => (j*2).toInt
        case k:Float => (k*10).toInt
      }
      //函数调用
      var list=List(1,2,3,4,3.5,6,8.5,"hello")
      println(list.collect(f))//List(2, 3, 4, 5)
      //第二种简写形式
      var list1=list.collect{
        case x:Int => x+1  //省略了偏函数的定义
      }
      println(list1)
    }
  }
  ~~~

### 9.3，作为参数的函数

- 基本介绍
  函数作为一个变量传入到了另一个函数中，那么该作为参数的函数的类型是：function1.。，即：(参数类型) => 返回类型

- 快速入门

  ~~~ java
  package qq.com.rzf.partial
  
  object FunFields {
    def main(args: Array[String]): Unit = {
  
      def plus(x: Int) = {
        3 + x
      }
  //plus(_)将plus函数作为参数传递给map函数，而不是执行函数
      val result1 = Array(1, 2, 3, 4).map(plus(_))
      println(result1.mkString(","))
      //打印出plus函数的类型,plus _标示打印此函数的类型或者引用，而不是执行函数
      println("plus的函数类型是："+plus _)//plus的函数类型是：<function1>
    }
  }
  ~~~

- 应用实例小结

  - map(plus(_)) 中的plus(_) 就是将plus这个函数当做一个参数传给了map，_这里代表从集合中遍历出来的一个元素。_
  - plus(_) 这里也可以写成 plus 表示对 Array(1,2,3,4) 遍历，将每次遍历的元素传给plus的 x
    进行 3 + x 运算后，返回新的Int ，并加入到新的集合 result1中
  - def map[B, That] (f: A => B) 的声明中的 f: A => B 一个函数

### 9.4，匿名函数

1. 基本介绍

   没有名字的函数就是匿名函数，可以通过**函数表达式**，来设置匿名函数

2. 案例

   ~~~ java
   package qq.com.rzf.partial
   
   object AnomyFun {
     def main(args: Array[String]): Unit = {
       //匿名函数不需要写def 函数名
       //也不需要写返回值，使用类型推导
       //=编程了=>
       //多行函数体可以用{}
       var triple=(x:Int)=>3*x//匿名函数
       println(triple(3))//9
       //不写参数，triple变量可以打印出函数的类型
       println(triple)//<function1>，应为函数有一个参数，所以是function1
     }
   }
   
   ~~~

3. 练习

   ~~~ java
   val triple =  (x: Double) => 3 * x
   pritnln(triple) // 类型 
   println(triple(3))
   说明
   (x: Double) => 3 * x 就是匿名函数 
   (x: Double) 是形参列表， => 是规定语法表示后面是函数体， 3 * x 就是函数体，如果有多行，可以 {} 换行写.
   triple 是指向匿名函数的变量。
   
   
   //如果是将一个函数，直接传递给一个方法，是如何写的
   package com.atguigu.chapter13
   
   object NoNameFunction {
     def main(args: Array[String]): Unit = {
   
       //请编写一个匿名函数，可以返回2个整数的和，并输出该匿名函数的类型
       //如果我们定义一个函数，则变量名字要写
       val f1 = (x1:Int,x2:Int) => {
         x1 + x2
       }
       println(f1(10, 30)) // 40
       println(f1) // <function2>
   
       //调用f2 完成一个运算
       println(f2(f1,30,40)) // 70 // f = f1
     }
   
     //方法，可以接受一个函数，该函数返回两个数的差
     //这时，我们只是写一个函数的格式(签名)
     def f2(f:(Int,Int) => Int, n1:Int,n2:Int): Int = {
       f(n1,n2)
     }
   }
   ~~~

### 9.5，高阶函数

1. 基本介绍

   能够接受函数作为参数的函数，叫做高阶函数 (**higher-order** function)。可使应用程序更加健壮。 高阶函数可以返回一个匿名函数.

2. 入门

   ~~~ java
   package qq.com.rzf.partial
   
   object HighFun {
     def main(args: Array[String]): Unit = {
       //函数test接收一个函数参数为double类型的函数，此函数返回值也是double类型
       //可以传入多个函数，函数之间还可以相互调用
       def test(f: Double => Double,f1: Double=>Int, n1: Double) = {
         f(f1(n1)) //调用f函数
       }
       def fun(d:Double):Int={
         d.toInt%2
       }
       def sum(d: Double): Double = {
         d + d
       }
       val res = test(sum,fun, 7.0)
       println("res=" + res)
   
     }
   }
   package qq.com.rzf.partial
   
   object HighOrder {
     def main(args: Array[String]): Unit = {
       def minusxy(x: Int) = {
         (y: Int) => x - y // 函数表达式, 返回的是一个匿名函数
       }
       //说明
       //minusxy 高阶函数
       //  返回的是 (y: Int) => x – y 匿名函数
       //返回的匿名函数可以用变量接收
         minusxy(3)// => 返回的就是一个具体的匿名函数 (y: Int) => 3 – y
       //函数的柯里化
       minusxy(3)(5) //=》3 – 5 = -2
       //先返回3-y的匿名函数，在把5带入计算
       val result3 = minusxy(3)(5)
       println(result3)
   
     }
   }
   
   ~~~

3. 小结

   说明: def minusxy(x: Int) = (y: Int) => x - y

   1)函数名为 minusxy

   2)该函数返回一个匿名函数

   ​        (y: Int) = > x -y    

   3)说明val result3 = minusxy(3)(5)

   minusxy(3)执行minusxy(x: Int)得到 (y: Int) => 3 - y 这个匿名函

   minusxy(3)(5)执行 (y: Int) => 3 - y 这个匿名函数

   也可以分步执行: val f1 = minusxy(3);   val res = f1(90)

### 9.6，参数(类型)推断

1. 基本说明

   参数推断省去类型信息（在某些情况下[需要有应用场景]，参数类型是可以推断出来的，如list=(1,2,3) list.map()   map中函数参数类型是可以推断的)，同时也可以进行相应的简写。

2. 参数类型推断写法说明/规则

   1.  参数类型是可以推断时，可以省略参数类型
   2. 当传入的函数，只有单个参数时，可以省去括号
   3. 如果变量只在=>右边只出现一次，可以用_来代替

3. 案例

   ~~~ java
   package qq.com.rzf.partial
   
   object ParamaterInfer {
     def main(args: Array[String]): Unit = {
       val list = List(1, 2, 3, 4)
       println(list.map((x: Int) => x + 1)) // List(2,3,4,5)
       //第一种简写：应为传入整数，所以可以省略int类型
       println(list.map((x) => x + 1)) // List(2,3,4,5)
       //应为传入的只有单个参数，所以可以省略（）
       println(list.map(x => x + 1)) // List(2,3,4,5)
       //x只在=>右边出现一次，所以x可以用_代替
       println(list.map(_ + 1)) // List(2,3,4,5)
       println("******************")
       println(list.reduceLeft(_ + _)) // 对list求和 10
   
       println(list.reduceLeft(f))
       //反推标准的写法.
       println(list.reduceLeft(_ + _))
       //用匿名函数简写
       println(list.reduceLeft((n1:Int,n2:Int) => n1+n2))
       //应为可以传入参数全部是整数，所以可以省略int类型
       println(list.reduceLeft((n1,n2) => n1+n2))
       //传入的参数是单个参数，可以省略（）
       //println(list.reduceLeft(n1,n2 => n1+n2))
       println(list.reduceLeft(_ + _ ))
     }
     def f(n1:Int,n2:Int): Int = {
       n1 + n2
     }
   }
   
   ~~~

### 9.7，闭包(closure)

- 基本介绍：闭包就是一个函数和与其相关的引用环境（变量/值）组合的一个整体(实体)。

- 案例

  ~~~ java
  def minusxy(x: Int) = (y: Int) => x – y
  //minusxy 他会返回一个匿名函数 (y: Int) => x – y
  //匿名函数，他使用了一个外部的变量 x
  //f函数就是闭包.
  val f = minusxy(20) 
  println("f(1)=" + f(1)) // 
  println("f(2)=" + f(2)) // 
  ~~~

- 小结：

  - 第1点
    (y: Int) => x – y

    返回的是一个匿名函数 ，因为该函数引用到到函数外的 x,那么  该函数和x整体形成一个闭包
    如：这里 val f = minusxy(20) 的f函数就是闭包 

    你可以这样理解，返回函数是一个对象，而x就是该对象的一个字段，他们共同形成一个闭包
    当多次调用f时（可以理解多次调用闭包），发现使用的是同一个x, 所以x不变。
    在使用闭包时，主要搞清楚返回函数引用了函数外的哪些变量，因为他们会组合成一个整体(实体),形成一个闭包

- 闭包的最佳实践

  ~~~ java
  package qq.com.rzf.partial
  
  object ClusterDemo {
    def main(args: Array[String]): Unit = {
      /**
       * 请编写一个程序，具体要求如下
       * 编写一个函数 makeSuffix(suffix: String)  可以接收一个文件后缀名(比如.jpg)，并返回一个闭包
       * 调用闭包，可以传入一个文件名，如果该文件名没有指定的后缀(比如.jpg) ,则返回 文件名.jpg , 如果已经有.jpg后缀，则返回原文件名。
       * 要求使用闭包的方式完成
       * String.endsWith(xx)
       */
      val f=makeSuffix(".jpg")
      println(f("dog"))
      println(f("cat.jpg"))
    }
    def  makeSuffix(suffix: String){
      //返回一个匿名函数
      (fileName:String) => {
        if(fileName.endsWith("jpg")){
          fileName
        }else{
          fileName+suffix
        }
      }
    }
  }
  ~~~

- 体会闭包的好处

  - 返回的匿名函数和 makeSuffix (suffix string) 的 suffix 变量 组合成一个闭包,因为 返回的函数引用到suffix这个变量
  - 我们体会一下闭包的好处，如果使用传统的方法，也可以轻松实现这个功能，但是传统方法需要每次都传入 后缀名，比如 .jpg ,而闭包因为可以保留上次引用的某个值，所以我们传入一次就可以反复使用。大家可以仔细的体会一把！

### 9.8，函数柯里化

1. 函数柯里化快速入门
   编写一个函数，接收两个整数，可以返回两个数的乘积，要求:

   使用常规的方式完成
   使用闭包的方式完成
   使用函数柯里化完成
   注意观察编程方式的变化。[案例演示]

   ~~~ java
   package qq.com.rzf.partial
   
   object CurryDeomo {
     def main(args: Array[String]): Unit = {
       def mul(x: Int, y: Int) = x * y
       println(mul(10, 10))
   
       def mulCurry(x: Int) = (y: Int) => x * y
       println(mulCurry(10)(9))
   
       def mulCurry2(x: Int)(y:Int) = x * y
       println(mulCurry2(10)(8))
   
     }
   }
   ~~~

2.  函数柯里化最佳实践

   -  比较两个字符串在忽略大小写的情况下是否相等，注意，这里是两个任务：

   1) 全部转大写（或小写

   2) 比较是否相等

   针对这两个操作，我们用一个函数去处理的思想，其实也变成了两个函数处理的思想（**柯里化**）

   函数柯里化其实就是将完成一件事的步骤逐个完成

   ~~~java
   object CurryDemo02 {
     def main(args: Array[String]): Unit = {
   
       //方法,比较s1和s2是否相等
       def eq(s1: String, s2: String): Boolean = {
         s1.equals(s2)
       }
   
       //隐式类
       //1. checkEq 方法
       //2. 接收 ss: String ， f: (String, String) => Boolean
       //3. 任务分成了两个部分 1. 转成小写 2. 比较
   
       //一个隐式类在底层会生成两个部分
       //1. 隐式类对应的.class  CurryDemo02.TestEq$2.class
       //2. 生成一个方法 TestEq$1(s: String)
       implicit class TestEq(s: String) {
         //这里体现
         def checkEq(ss: String)(f: (String, String) => Boolean): Boolean = {
           f(s.toLowerCase, ss.toLowerCase)
         }
       }
       //可以调用测试
       val str1 = "hello"
       val str2 = "HELLu"
       println(str1.checkEq(str2)(eq)) //TestEq$1(str1).checkEq(str2)(eq)
       //第二种简写形式
       println(str1.checkEq(str2)(_.equals(_))) // true
     }
   }
   ~~~

### 9.9，控制抽象

1. 控制抽象是这样的函数，满足如下条件
   参数是函数
   函数参数没有输入值也没有返回值

2. 案例

   ~~~ java
   package qq.com.rzf.partial
   
   object AbCon {
     def main(args: Array[String]): Unit = {
       //接收一个函数，没有参数，也没有返回值
       def myRunInThread(f1: () => Unit) = {
         new Thread {//new一个匿名类，实现其中的run方法
           override def run(): Unit = {
             f1()//执行f1，相当于把myRunInThread那一段代码放进来执行
             //但是这样写太麻烦
           }
         }.start()
       }
       myRunInThread {
         () => println("干活咯！5秒完成...")
           Thread.sleep(5000)
           println("干完咯！")
       }
   println("简化形式***********************************")
       //接收一个函数，没有参数，也没有返回值
       def myRunInThread1(f1: => Unit) = {
         new Thread {//new一个匿名类，实现其中的run方法
           override def run(): Unit = {
             f1 //执行f1，相当于把myRunInThread那一段代码放进来执行
             //但是这样写太麻烦
           }
         }.start()
       }
       myRunInThread1 {
           println("干活咯！5秒完成...")
           Thread.sleep(5000)
           println("干完咯！")
       }
   
     }
   }
   ~~~

3. 进阶用法：实现类似while的myWhile函数

   ~~~ java
   object ControlAbstractDemo02 {
     def main(args: Array[String]): Unit = {
   
       //传统的while循环
       //    var n = 10
       //    while (n > 0) {
       //      println("n=" + n)
       //      n -= 1
       //    }
   
       var n = 10
       myWhile( n > 0) {
           println("n=" + n)
           n -= 1
       }
   
     }
   
     //使用控制抽象的特点，完成myWhile
     def myWhile(condition:  => Boolean)(block: => Unit): Unit = {
       if (condition) {
         block
         myWhile(condition)(block)
       }
     }
   }
   ~~~

### 第十章，泛型

1. 基本介绍

   如果我们要求函数的参数可以接受任意类型。可以使用泛型，这个类型可以代表任意的数据类型。 
   例如 List，在创建 List 时，可以传入整型、字符串、浮点数等等任意类型。那是因为 List 在 类定义时引用了泛型。比如在Java中：public interface List<E> extends Collection<E>

2. Scala泛型应用案例

   - 请设计一个EnglishClass (英语班级类)，在创建EnglishClass的一个实例时，需要指定[ 班级开班季节(spring,autumn,summer,winter)、班级名称、班级类型]
     开班季节只能是指定的，班级名称为String, 班级类型是(字符串类型 "高级班", "初级班"..) 或者是 Int 类型(1, 2, 3 等)
     请使用泛型来完成本案例.

     ~~~ java
     package qq.com.rzf.fanxing
     
     import qq.com.rzf.fanxing.SeasonEnum.SeasonEnum
     
     object Demo01 {
       def main(args: Array[String]): Unit = {
         /**
          * 要求
          * 请设计一个EnglishClass (英语班级类)，在创建EnglishClass的一个实例时，需要指定[ 班级开班季节(spring,autumn,summer,winter)、班级名称、班级类型]
          * 开班季节只能是指定的，班级名称为String, 班级类型是(字符串类型 "高级班", "初级班"..) 或者是 Int 类型(1, 2, 3 等)
          * 请使用泛型来完成本案例.
          */
         val value = new EnglishClass[SeasonEnum, String, String]("SeasonEnum.spring", "0705", "高级")
       println(value.className+value.classSeason+value.classType)
       }
     }
     
     class EnglishClass[A,B,C](val classSeason:String,val className:String,val classType:String){
     
     }
     //季节是枚举类型
     object SeasonEnum extends Enumeration {
         type SeasonEnum=Value//seasonEnum的类型是值类型，可以使用下面的4个值
         val spring,autumn,summer,winter=Value
     }
     ~~~

3. 定义一个函数，可以获取各种类型的 List 的中间index的值
   使用泛型完成

   ~~~ java
   package qq.com.rzf.fanxing
   
   object Dem02 {
     def main(args: Array[String]): Unit = {
       /**
        * 定义一个函数，可以获取各种类型的 List 的中间index的值
        * 使用泛型完成
        */
       var list=List("aaa","bbb","ccc")
       var list1=List(1,2,3,4,5)
       println(midList[String](list))
       println(midList[Int](list1))
   
     }
     def midList[T](l:List[T])={
       l(l.length/2)
     }
   }
   ~~~

   


​     

​     

​     