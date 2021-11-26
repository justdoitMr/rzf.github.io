
<!-- TOC -->

- [`Scala`初级部分笔记](#scala初级部分笔记)
    - [第一章，`Scala`概述](#第一章scala概述)
        - [1.1，`Scala`运行机制概述](#11scala运行机制概述)
        - [1.2，`Scala`运行机制剖析](#12scala运行机制剖析)
        - [1.3，`scala`执行流程](#13scala执行流程)
        - [1.4，`scala`程序开发注意事项](#14scala程序开发注意事项)
        - [1.5，`scala`语言的转义字符](#15scala语言的转义字符)
        - [1.6，`scala`语言的三种输出](#16scala语言的三种输出)
        - [1.7，`scala`关联源代码](#17scala关联源代码)
        - [1.8，`scala`注释](#18scala注释)
        - [1.9，`scala`编程规范](#19scala编程规范)
    - [第二章，`scala`核心编程](#第二章scala核心编程)
        - [2.1，变量](#21变量)
        - [2.2，数据类型](#22数据类型)
            - [2.2.1，整形类型](#221整形类型)
            - [2.2.2，浮点类型](#222浮点类型)
            - [2.2.3，字符类型(`Char`)](#223字符类型char)
            - [2.2.4，布尔类型：`Boolean`](#224布尔类型boolean)
            - [2.2.5，`Unit`类型、`Null`类型和`Nothing`类型](#225unit类型null类型和nothing类型)
        - [2.3，值类型转换](#23值类型转换)
            - [2.3.1，值的隐式类型转换（编译器自动完成）](#231值的隐式类型转换编译器自动完成)
            - [2.3.2，强制类型转换（不属于隐式类型转换，需要手工完成）](#232强制类型转换不属于隐式类型转换需要手工完成)
            - [2.3.3，高级隐式转换和隐式函数](#233高级隐式转换和隐式函数)
            - [2.3.4，值类型和`String`类型的转换](#234值类型和string类型的转换)
        - [2.4，标识符的命名规范](#24标识符的命名规范)
        - [2.5，运算符](#25运算符)
            - [2.5.1，算数运算符](#251算数运算符)
            - [2.52，关系运算符(比较运算符)](#252关系运算符比较运算符)
            - [2.5.3，逻辑运算符](#253逻辑运算符)
            - [2.5.4，赋值运算符](#254赋值运算符)
            - [2.5.5，位运算符](#255位运算符)
            - [2.5.6，运算符优先级](#256运算符优先级)
        - [2.6，键盘输入语句](#26键盘输入语句)
        - [2.7，流程控制](#27流程控制)
            - [2.7.1，顺序控制](#271顺序控制)
            - [2.7.2，分支控制`if-else`介绍](#272分支控制if-else介绍)
            - [2.7.3，循环](#273循环)
        - [2.8，函数式编程（基础部分）](#28函数式编程基础部分)
            - [2.8.1，方法与函数](#281方法与函数)
            - [2.8.2，面向函数编程](#282面向函数编程)
        - [2.9，面向对象编程（基础部分）](#29面向对象编程基础部分)

<!-- /TOC -->




# `Scala`初级部分笔记

[TOC]

## 第一章，`Scala`概述

### 1.1，`Scala`运行机制概述

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195017-151124.png)

1. `Scala`语言的特点：

- `Scala`是一门以`java`虚拟机`（JVM）`为目标运行环境并将面向对象和函数式编程的最佳特性结合在一起的静态类型编程语言。 
- Scala 是一门多范式 (multi-paradigm) 的编程语言，Scala支持面向对象和函数式编程
- Scala源代码(.scala)会被编译成Java字节码(.class)，然后运行于JVM之上，并可以调用现有的Java类库，实现两种语言的无缝对接。
- Scala 单作为一门语言来看， 非常的简洁高效
- Scala 在设计时，马丁·奥德斯基 是参考了Java的设计思想，可以说Scala是源于java，同时马丁·奥德斯基 也加入了自己的思想，将函数式编程语言的特点融合到JAVA中, 因此，对于学习过Java的同学，只要在学习Scala的过程中，搞清楚Scala 和 Java相同点和不同点，就可以快速的掌握Scala这门语言 

2. `Scala`运行环境的搭建

   [`Scala`运行环境搭建]()

### 1.2，`Scala`运行机制剖析

1. 编写`scala`的`hello`版本程序。

~~~ java
/**
 * 只要以后看到有object Hello ，就应该有一种意识，
 * 1 object Hello 对用的是Hello$类型的一个静态对象，名字是MODULE$
 * 2 在程序中是一个单例的对象
 */
object Hello {
  def main(args: Array[String]): Unit = {
    println("helloword");
  }
}
~~~

2. 转化`scala`程序为`java`程序(反编译)

~~~ java
/**
 * 我们可以理解为scala在运行时做了一个包装
 */
public class Hello {
    public static void main(String[] var0) {
     // .MODULE$.main(var0);
        Hello$.MODULE$.main(var0);
    }

}
 final class Hello$ {
    public static final Hello$ MODULE$;

    static {
        MODULE$ = new Hello$();
    }
    public void main(String[] args) {
      System.out.println("helloword");
    }
    private Hello$() {
      ///  MODULE$ = this;
    }
}

~~~

### 1.3，`scala`执行流程

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195019-672059.png)

- 对于`scala`源文件，我们可以先编译为`class`字节码文件，在运行，也可以不进行编译，直接运行，但是速度比较慢。

### 1.4，`scala`程序开发注意事项

1. `Scala`源文件以` “.scala" `为扩展名
2. `Scala`程序的执行入口是`main()`函数,但是实际上是对`main`函数进行了一层的包装。
3. `Scala`语言严格区分大小写。
4. `Scala`方法由一条条语句构成，每个语句后不需要分号(`Scala`语言会在每行后自动加分号)，这也体现出`Scala`的简洁性。
5. 如果在同一行有多条语句，除了最后一条语句不需要分号，其它语句需要分号(尽量一行就写一条语句)。

### 1.5，`scala`语言的转义字符

![1621684330606](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195211-787393.png)

### 1.6，`scala`语言的三种输出

1. 字符串通过+号连接（类似`java`）

~~~ java
println("name=" + name + " age=" + age + " url=" + url)
~~~

2. printf用法 （类似C语言）字符串通过 % 传值。(格式化输出)

~~~ java
printf("name=%s, age=%d, url=%s \n", name, age, url)
~~~

3. 字符串插值：通过$引用(类似PHP）

~~~ java
println(s"name=$name, age=$age, url=$url")
~~~

~~~ java
object PrintDemo {
  def main(args: Array[String]): Unit = {
    var str1:String="xiaoming";
    var str2:String="xiaobai";
    print(str1+str2);
    var name:String="tom";
    var age:Int=10;
    var salar:Float=10.2f;
    var high:Double=178.2;
    //在这里进行格式化输出
    printf("名字%s 年龄%d 薪水%.2f 身高%.3f",name,age,salar,high)
    //scala支持用$符号获取变量的值,编译器会解析$对应的变量
    println(s"个人信息如下:\n 名字$name \n 年龄$age \n 薪水$salar \n 身高$high");
    //支持用{}包起来一个表达式
    println(s"个人信息如下:\n 名字${name} \n 年龄${age+10} \n 薪水${salar*10} \n 身高$high")
  }
}
~~~

### 1.7，`scala`关联源代码

-  查看源码的快捷键`ctrl+b`，格式化代码`ctrl+alt+l`

- 把`scala`源代码包解压一次或者两个到`scala`安装目录下面的`lib`目录下，用快捷键`ctrl+b`快捷键即可关联源代码查看。

### 1.8，`scala`注释

1. 用于注解说明解释程序的文字就是注释，注释提高了代码的阅读性；
2. 注释是一个程序员必须要具有的良好编程习惯。
3. 将自己的思想通过注释先整理出来，再用代码去体现。

~~~ java
1.单行注释：
基本格式
格式： //注释文字
2.多行注释：
基本格式
格式： /*  注释文字 */
3.文档注释
//文档注释演示，源码
object Comment {
  def main(args: Array[String]): Unit = {
    println("hello word")
  }
  /**
   *@deprecated 已经过期
   * @example
   *          输入：num1=10,num2=20,返回num1+num2=20
   * @param num1
   * @param num2
   * @return 求和
   */
  def sum(num1:Int,num2:Int):Int={
    return num1+num2;
  }
}
//进入源代码的存放目录，在控制台输入
scaladoc -d 输出目录 源代码
scaladoc -d i:/ Comment.scala
~~~

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195023-708308.png)

### 1.9，`scala`编程规范

1. 查看源码的快捷键`ctrl+b`，格式化代码`ctrl+alt+l`
2. 正确的注释风格。
3. 使用`tab`键代码整体向右边移动，使用`shift+table`代码整体向左边移动。
4. 运算符两边加上空格。
5. 每一行最多不超过80个字符，学会使用换行符。

- `scala`程序的编写，编译，运行过程是什么，能否直接运行？
  - 编写就是使用工具开发`scala`程序。
  - 编译就是将`.scala`文件编译成`.class`文件，使用命令`scalac`。
  - 运行就是使用`scala`命令将`.class`文件加载到`jvm`上面运行。
  - 可以直接运行`.scala`文件，但是速度比较慢，使用命令`scala xxx.scala`。

## 第二章，`scala`核心编程

### 2.1，变量

1. 变量相当于内存中一个数据存储空间的表示，你可以把变量看做是一个房间的门牌号，通过门牌号我们可以找到房间，而通过变量名可以访问到变量(值)。
2. 语法说明：

~~~ java
变量声明基本语法
var | val 变量名 [: 变量类型] = 变量值
var str:String="name"
//案例
object varDemo {
  def main(args: Array[String]): Unit = {
    var age:Int=20
    var name:String="tom"
    var salary:Double=20.3
    var ispass:Boolean=true
    //在scala中，小数默认为double类型，整数默认为int类型,定义浮点类型要带f
    var high:Float=98.8f
  }
~~~

3. 声明变量时，类型可以省略（就是叫 类型推断）

   ~~~ java
   object varDemo01 {
     def main(args: Array[String]): Unit = {
       var num=10
       //方式一，idea会给出类型提示
       //方式二isInstanceOf函数
       println(num.isInstanceOf[Int])
     }
   ~~~

   - 类型确定后，就不能修改，说明`Scala` 是**强数据类型语言**(在编译期间类型已经确定，运行期间不可以在改变),可以修改值，但是类型不可以修改。

   - 在声明/定义一个变量时，可以使用`var` 或者 `val` 来修饰，` var` 修饰的变量可改变，`val `修饰的变量不可改 

   - `val`修饰的对象属性在编译后，等同于加上`final`，运用反编译工具，我们发现在底层用`val`定义的变量有`final`来修饰。
   -  `var` 修饰的对象引用可以改变，`val `修饰的则不可改变，但对象的状态(值)却是可以改变的。(比如: 自定义对象、数组、集合等等) 
   - 变量声明时，必须有初始值（显示初始化）。

   ~~~ java
   var age=10
       age=40//正确
       val salary=90
       //salary=20 //会报错 reassignment to val
       //salary设计者为何设计val和var
       //应为在实际编程中，我们更多修改或者获取对象的值，但是很少去改变对象本身
       //dog=new dog(),dog.age=10,修改对象，但是不会这样做：dog=new dog()
       //这时我们就可以使用val,没有现线程安全问题，效率比较高
       //如果对象真的需要改变，就用val
       //声明对象一般用val,标示此对象不可以发生本质变化，对象属性用var,属性可以修改
   ~~~

4. 声明变量和定义变量要区分开：声明变量，变量没有初始化，而定义变量，必须初始化。`scala`要求声明变量时必须初始化。

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195027-567406.png)

5. `scala`中`+`使用

- 当`+`两边是数值型类型时，做加法运算，当有一边是字符串时，做字符串拼接运算。

### 2.2，数据类型

1. `Scala` 与` Java`有着相同的数据类型，在`Scala`中数据类型都是对象，也就是说`scala`没有`java`中的原生(基本)
2. Scala数据类型分为两大类` AnyVal`(值类型) 和 `AnyRef`(引用类型)， 注意：不管是`AnyVal`还是`AnyRef` 都是对象。
3. 相对于`java`的类型系统，`scala`要复杂些！也正是这复杂多变的类型系统才让面向对象编程和函数式编程完美的融合在了一起

~~~ java
//查看源码发现Int是一个类，继承了AnyVal，Int的一个实例可以使用很多方法
//在scala中，如果一个方法没有形参，那么可以省略（）
final abstract class Int private extends AnyVal 
~~~

4. 数据类型体系图

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195028-945675.png)

- 实现代表继承，虚线代表隐式转换，遵循`java`类型转换规则，由低精度到高精度的转换。
- `nothing`是所有类型子类，因此`nothing`类型数值可以返回给任何一个类型的变量。`null`可以返回给任何引用类型。

5. 数据类型表

| 数据类型 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| Byte     | 8位有符号补码整数。数值区间为 -128 到 127                    |
| Short    | 16位有符号补码整数。数值区间为 -32768 到 32767               |
| Int      | 32位有符号补码整数。数值区间为 -2147483648 到 2147483647     |
| Long     | 64位有符号补码整数。数值区间为 -9223372036854775808 到 9223372036854775807 |
| Float    | 32 位, IEEE 754标准的单精度浮点数                            |
| Double   | 64 位 IEEE 754标准的双精度浮点数                             |
| Char     | 16位无符号Unicode字符, 区间值为 U+0000 到 U+FFFF             |
| String   | 字符序列                                                     |
| Boolean  | true或false                                                  |
| Unit     | 表示无值，和其他语言中void等同。用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成()。 |
| Null     | null                                                         |
| Nothing  | Nothing类型在Scala的类层级的最低端；它是任何其他类型的子类型。 |
| Any      | Any是所有其他类的超类                                        |
| AnyRef   | AnyRef类是Scala里所有引用类(reference class)的基类           |

#### 2.2.1，整形类型

| 数据类型  | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| Byte [1]  | 8位有符号补码整数。数值区间为 -128 到 127                    |
| Short [2] | 16位有符号补码整数。数值区间为 -32768 到 32767               |
| Int [4]   | 32位有符号补码整数。数值区间为 -2147483648 到 2147483647     |
| Long [8]  | 64位有符号补码整数。数值区间为 -9223372036854775808 到 9223372036854775807 = 2的(64-1)次方-1 |

1. 整型的使用细节
   1. `Scala`各整数类型有固定的表数范围和字段长度，不受具体`OS`的影响，以保证`Scala`程序的可移植性。
   2. `Scala`的整型 常量/字面量  默认为` Int `型，声明`Long`型 常量/字面量 须后加`‘l’’`或`‘L’ `.
   3. `Scala`程序中变量常声明为`Int`型，除非不足以表示大数，才使用`Long`。

~~~ java
//获取long最值
 Long.MaxValue
Long.MinValue
var e = 9223372036854775234807//会报错Integer number is out of range even for type Long
var e = 9223372036854775807L//在末尾添加L即可解决
~~~

#### 2.2.2，浮点类型

- `Scala`的浮点类型可以表示一个小数

| Float  | 32 位, IEEE 754标准的单精度浮点数 |
| ------ | --------------------------------- |
| Double | 64 位 IEEE 754标准的双精度浮点数  |

~~~ java
var f1 : Float = 1.1//报错，默认是double类型
var f2 = 1.2 //不会报错，没有指明类型
var f3 : Double = 1.3 //ok
var f4 : Float = 1.4f //ok
var f5 : Double = 1.5f  //低精度给高精度

~~~

1. 浮点型使用细节
   1. 与整数类型类似，`Scala` 浮点类型也有固定的表数范围和字段长度，不受具体`OS`的影响。
   2. `Scala`的浮点型常量默认为`Double`型，声明`Float`型常量，须后加`‘f’`或`‘F’`。
   3. 浮点型常量有两种表示形式
             十进制数形式：如：`5.12       512.0f        .512`   (必须有小数点）
             科学计数法形式:如：`5.12e2 => 5.12乘以 10的2次方`      `5.12E-2 => 表示5.12 除以 10的2次方 `
   4. 通常情况下，应该使用`Double`型，因为它比`Float`型更精确(小数点后大致7位), 测试数据 ：`2.2345678912f  , 2.2345678912   //  2.7 与 8.1/3`

#### 2.2.3，字符类型(`Char`)

- 字符类型可以表示单个字符,字符类型是`Char`， 16位无符号`Unicode`字符(2个字节), 区间值为` U+0000 `到 `U+FFFF`,`unicode`码包含`ascll`码。
- 字符常量是用单引号`(‘ ’)`括起来的单个字符。例如：`var c1 = 'a‘   var c2 = '中‘  var c3 =  '9' `.
- Scala 也允许使用转义字符‘\’来将其后的字符转变为特殊字符型常量。例如：var c3 = ‘\n’  // '\n'表示换行符 
- 可以直接给Char赋一个整数，然后输出时，会按照对应的unicode 字符输出 ['\u0061' 97].
- char类型可以进行运算的，相当于一个整数，因为有对应的`unicode `码。
- var c2 : Char =  ‘a’ + 1  正确吗? （×）
  修改: var c2 : Int =  'a' + 1   [ok]

~~~ java
object CharTest {
  def main(args: Array[String]): Unit = {
    var ch:Char=97
    var ch1:Char='a'
    //我们输出一个char类型，会在unicode码表中找到对应的字符输出
    println(ch)
    //char可以当做数值进行运算
    var ch1:Char=97
    var num=10+ch1
    print("num="+num)//107
    /*
    原因：
    1，当把计算结果赋值给一个变量，则编译器会进行类型转换及判断（判断类型和范围是否越界）
    2，当把一个字面量或者常量赋给一个变量，编译器只进行范围检查，不判断类型
     */
    var c1:Char='a'+1//报错，行为编译器自动将a字符转换为int,相加后为int类型赋值给char类型变量
    //属于高精度类型变量赋值给低精度变量，所以报错
    var c2:Char=97+1//报错，相加结果是int类型
    var c3:Char=98//正确
    var c3:Char=9999999999//报错，超过char类型的范围，发生越界

  }
}
~~~

#### 2.2.4，布尔类型：`Boolean`

1. 布尔类型也叫`Boolean`类型，`Booolean`类型数据只允许取值`true`和`false`。
2. `boolean`类型占1个字节
3. `boolean` 类型适于逻辑运算，一般用于程序流程控制：
   - `if`条件控制语句                  
   - `while`循环控制语句
   - `do-while`循环控制语句      
   - `for`循环控制语句

#### 2.2.5，`Unit`类型、`Null`类型和`Nothing`类型

| Unit    | 表示无值，和其他语言中void等同。用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成()。 |
| ------- | ------------------------------------------------------------ |
| Null    | null , Null 类型只有一个实例值 null                          |
| Nothing | Nothing类型在Scala的类层级的最低端；它是任何其他类型的子类型。当一个函数，我们确定没有正常的返回值，可以用Nothing 来指定返回类型，这样有一个好处，就是我们可以把返回的值（异常）赋给其它的函数或者变量（兼容性）def f1():Nothing = {       throw new Exception() } |

~~~ java
object UnitNullTest {
  def main(args: Array[String]): Unit = {
    val hello = sayHello
    print("hello="+hello)//输出值是(),()就代表unit
    //Null类只有一个实例对象，null，类似于Java中的null引用。
    // null可以赋值给任意引用类型(AnyRef)，
    // 但是不能赋值给值类型(AnyVal: 比如 Int, Float, Char, Boolean, Long, Double, Byte, Short)
    var dog:Dog=null//null赋值给引用类型
    var char1:Char=null //null赋值给值类型，没有报错，但是运行时候会报错，
  }
  //Unit等价于其他语言中的void,只有一个实例（）
  def sayHello():Unit={

  }
}
class Dog{

}
~~~

1. 使用细节和注意事项

   1. `Null`类只有一个实例对象，`null`，类似于`Java`中的`null`引用。`null`可以赋值给任意引用类型`(AnyRef)`，但是不能赋值给值类型`(AnyVal: 比如 Int, Float, Char, Boolean, Long, Double, Byte, Short)`
   2. `Unit`类型用来标识过程，也就是没有明确返回值的函数。由此可见，`Unit`类似于`Java`里的`void`。`Unit`只有一个实例，`()`，这个实例也没有实质的意义
   3. `Nothing`，可以作为没有正常返回值的方法的返回类型，非常直观的告诉你这个方法不会正常返回，而且由于`Nothing`是其他任意类型的子类，他还能跟要求返回值的方法兼容。

   ~~~java
   //nothing没有正常返回值
     def test() : Nothing={
     throw new Exception()
     }
   ~~~

### 2.3，值类型转换

#### 2.3.1，值的隐式类型转换（编译器自动完成）

- 当Scala`程序在进行赋值或者运算时，精度小的类型自动转换为精度大的数据类型，这个就是自动类型转换(隐式转换` implicit conversion)`。
- 数据类型按精度(容量)大小排序为

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195034-271887.png)

- 细节说明

  - **有多种类型的数据混合运算时，系统首先自动将所有数据转换成容量最大的那种数据类型，然后再进行计算。**

  ~~~ java
  5.6+10---->double
  ~~~

  - 当我们把精度(容量)大 的数据类型赋值给精度(容量)小 的数据类型时，就会报错，反之就会进行自动类型转换。
  - `(byte, short)` 和` char`之间不会相互自动转换。但是`byte`和`short`之间可以互相转换，
  - **特例：`byte，short，char ` 他们三者可以计算，在计算时首先转换为`int`类型。**
  - 自动提升原则： 表达式结果的类型自动提升为 **操作数中最大的类型**。

~~~ java
object DataConvert {
  def main(args: Array[String]): Unit = {
    var byte:Byte=10
   // var ch:Char=byte//报错，不会自动转换
    var ch1:Char=byte.toChar//明确指出转换
    //byte，short，char  他们三者可以计算，在计算时首先转换为int类型。
    var byte1:Byte=20
    var char:Char=90
    var short:Short=char+byte1//报错，相加结果为int,不能给short类型
    var short1:Short=10+90//报错，参与运算，会进行类型和范围的检查，相加为int类型，不可以给short
    var short2:Short=100//ok
  }
}
~~~

#### 2.3.2，强制类型转换（不属于隐式类型转换，需要手工完成）

- 自动类型转换的逆过程，将容量大的数据类型转换为容量小的数据类型。使用时要加上强制转函数，**但可能造成精度降低或溢出,格外要注意。**

~~~ java
  var num:Int=2.8.toInt//和java强制转换有区别,不会进行四舍五入，直接去掉小数部分
~~~

1. 细节问题

   1. 当进行数据的 从 大——>小，就需要使用到强制转换。
   2. 强转符号只针对于最近的操作数有效，往往会使用小括号提升优先级。

   ~~~ java
   var number:Int=10*3.5.toInt+8
   var number1:Int=(10*3.5+8).toInt
   ~~~

   3. `Char`类型可以保存 `Int`的常量值，但不能保存Int的变量值，需要强转。

   ~~~ java
   var char1:Char=1//正确，char类型保存常量
   var num1:Int=3
   var char2:Char=num1//报错，char类型保存变量，会涉及类型的检查，编译器在保存变量时，一定有类型的参与
   ~~~

   3. Byte和Short类型在进行运算时，当做Int类型处理。
   4. 练习

   ~~~ java
   1）var s : Short  = 5 对
         s = s-2      错
   2） var b : Byte  = 3   对
          b = b + 4错
          b = (b+4).toByte       对
   3）var c : Char  = ‘a’ 对
         var  i : Int = 5 对
         var d : Float = .314F 对
         var result : Double = c+i+d     对
   4） var b : Byte  = 5 对
         var s : Short  = 3 对
          var t : Short = s + b  错
          var t2 = s + b   对,使用类型推导
   
   ~~~

#### 2.3.3，高级隐式转换和隐式函数

见函数式编程章节

#### 2.3.4，值类型和`String`类型的转换

- 在程序开发中，我们经常需要将基本数据类型转成`String` 类型。或者将`String`类型转成基本数据类型。

1. 基本类型转`String`类型

~~~ java
var data:Int=10
var str:String=data+" " //直接加上空串即可，有下划线代表编译器做了自动类型转换，破浪线代表此变量没有使用
~~~

2. ` String`类型转基本数据类型

~~~java
var str1="asdf"
var num2=str1.toDouble//使用自带函数转换
~~~

3. 注意：

   - 在将`String `类型转成 基本数据类型时，要确保`String`类型能够转成有效的数据，比如 我们可以把 `"123"` , 转成一个整数，但是不能把` "hello" `转成一个整数

   - 思考就是要把` "12.5"` 转成` Int `? 

     这里会报错，应为`scala`会先把字符串转换为`double`，然后在转化为`int`会报错。但是可以转化为`float,double`类型。

### 2.4，标识符的命名规范

- `Scala`中的标识符声明，基本和`Java`是一致的，但是细节上会有所变化。

  - 首字符为字母，后续字符任意字母和数字，美元符号，可后接下划线_
  - 数字不可以开头。
  - **首字符为操作符(比如`+ - * / `)，后续字符也需跟操作符 ,至少一个**

  ~~~ java
   var ++ ="hello word"//在底层，++就变成了$plus$plus
   var -+=90
  ~~~

  - 操作符(比如+-*/)不能在标识符中间和最后.
  - 用反引号(飘号)``包括的任意字符串，即使是关键字(39个)也可以

  ~~~ java
  `true`//可以作为标识符
  在scala中，Int不是关键字，是预定义标识符，但是不推荐作为标识符
   var Int=90//不报错
  ~~~

- 练习

~~~java
hello  对  
hello12 对
1hello  错
h-b   错
x h   错
h_4   对
_ab   对
Int    对
Float  对
_   错
Abc    对
+*-   对
+a  错
~~~

- 标识符命名注意事项
  - 包名：尽量采取有意义的包名，简短，有意义(`com`.公司名.项目名)
  - 变量名、函数名 、方法名 采用驼峰法。
- `Scala`关键字有39个：

~~~ java
package, import, class, object, trait, extends, with, type, for
private, protected, abstract, sealed, final, implicit, lazy, override
try, catch, finally, throw 
if, else, match, case, do, while, for, return, yield
def, val, var 
this, super
new
true, false, null
~~~

### 2.5，运算符

#### 2.5.1，算数运算符

- 运算符是一种特殊的符号，用以表示数据的运算、赋值和比较等
  - 算术运算符

  - 赋值运算符
  - 比较运算符(关系运算符)

  - 逻辑运算符【与，或，非】

  - 位运算符 (位运算 `~  | ^  >> << >>>...`)

~~~ java
object Test {
  def main(args: Array[String]): Unit = {
    var r1 : Int = 10 / 3
    println("r1=" + r1)//3
    var r2 : Double = 10 / 3
    println("r2=" + r2)//3.0
    var r3 : Double = 10.0 / 3
    println("r3=" + r3 )3.333333
    println("r3=" + r3.formatted("%.2f") )//3.33
    //%的使用：a%b=a-a/b*b
    println(10%3)//1
    println(-10%3)//-1,-10-(-10/3)*3=-1
    println(-10% -3)//-1
    println(10% -3)//1,10-(10/(-3))*(-3)
    //在scala没有++，--，只有+=1，-=1
  }
}
~~~

- 对于除号“/”，它的整数除和小数除是有区别的：整数之间做除法时，只保留整数部分而舍弃小数部分。 例如：var x : Int = 10/3 ,结果是  3

- 当对一个数取模时，可以等价 a%b=a-a/b*b ， 这样我们可以看到取模的一个本质运算(和java 的取模规则一样)。

- 注意：Scala中没有++、--操作符，需要通过+=、-=来实现同样的效果

#### 2.52，关系运算符(比较运算符)

- 关系运算符的结果都是Boolean型，也就是要么是true，要么是false

- 关系表达式 经常用在 if结构的条件中或循环结构的条件中

- 关系运算符的使用和java一样

| **运算符** | **运算                                 范例                                         结果** |
| ---------- | ------------------------------------------------------------ |
| **==**     | 相等于                               4==3                                                false |
| **!=**     | 不等于                               4!=3                                                 true |
| **<**      | 小于                                   4<3                                                   false |
| **>**      | 大于                                   4>3                                                   true |
| **<=**     | 小于等于                           4<=3                                                false |
| **>=**     | 大于等于                           4>=3                                                true |

- 细节
  - 关系运算符的结果都是Boolean型，也就是要么是true，要么是false。

  - 关系运算符组成的表达式，我们称为关系表达式。 a > b 

  - 比较运算符“==”不能误写成“=”

  - 使用陷阱: 如果两个浮点数进行比较，应当保证数据类型一致.

#### 2.5.3，逻辑运算符

- 用于连接多个条件（一般来讲就是关系表达式），最终的结果也是一个Boolean值。 

- 逻辑运算符一览
- 假定变量 A 为 true，B 为 false

| 运算符 | 描述                      | 实例                       |
| ------ | ------------------------- | -------------------------- |
| &&     | 逻辑与 【同样遵守短路与】 | (A && B) 运算结果为 false  |
| \|\|   | 逻辑或  【遵守短路或】    | (A \|\| B) 运算结果为 true |
| !      | 逻辑非                    | !(A && B) 运算结果为 true  |

#### 2.5.4，赋值运算符

- 赋值运算符就是将某个运算后的值，赋给指定的变量
- 赋值运算符的分类 

| 运算符 | 描述                                           | 实例                                  |
| ------ | ---------------------------------------------- | ------------------------------------- |
| =      | 简单的赋值运算符，将一个表达式的值赋给一个左值 | C = A + B 将 A + B 表达式结果赋值给 C |
| +=     | 相加后再赋值                                   | C += A 等于 C = C + A                 |
| -=     | 相减后再赋值                                   | C -= A 等于 C = C - A                 |
| *=     | 相乘后再赋值                                   | C *= A 等于 C = C * A                 |
| /=     | 相除后再赋值                                   | C /= A 等于 C = C / A                 |
| %=     | 求余后再赋值                                   | C %= A 等于 C = C % A                 |

- 说明：这部分的赋值运算涉及到二进制相关知识，其运行的规则和Java一样。

| 运算符 | 描述           | 实例                       |
| ------ | -------------- | -------------------------- |
| <<=    | 左移后赋值     | C <<= 2  等于 C = C << 2   |
| >>=    | 右移后赋值     | C >>= 2  等于 C = C >> 2   |
| &=     | 按位与后赋值   | C &= 2   等于 C = C & 2    |
| ^=     | 按位异或后赋值 | C ^= 2    等于 C = C ^ 2   |
| \|=    | 按位或后赋值   | C \|= 2    等于 C = C \| 2 |

~~~ java
object assignment {
  def main(args: Array[String]): Unit = {
    var num=2
    num<<=2//向左移动两 位，相当于乘以4
    println(num)
    num>>=2//向右边移动两 位，相当于除以4，2相当与移动2次
    println(num)
  }
}
 //在scala中支持代码块返回
    var res={
      if(num>1)
        println("hello")
      else
        println("word")
    }
~~~

- 赋值运算符特点
  - 运算顺序从右往左。

  - 赋值运算符的左边 只能是变量,右边 可以是变量、表达式、常量值/字面量。

  - 复合赋值运算符等价于下面的效果，比如：`a+=3 `等价于` a = a + 3`
- 有两个变量，`a`和`b`，要求将其进行交换，但是不允许使用中间变量，最终打印结果：

~~~ java
var num1=10
var num2=20
num1=num1+num2
num2=num1-num2
num1=num1-num2
~~~

#### 2.5.5，位运算符

| 运算符 | 描述           | 实例                                                         |
| ------ | -------------- | ------------------------------------------------------------ |
| &      | 按位与运算符   | (a & b) 输出结果 12 ，二进制解释： 0000 1100                 |
| \|     | 按位或运算符   | (a \| b) 输出结果 61 ，二进制解释： 0011 1101                |
| ^      | 按位异或运算符 | (a ^ b) 输出结果 49 ，二进制解释： 0011 0001                 |
| ~      | 按位取反运算符 | (~a ) 输出结果 -61 ，二进制解释： 1100 0011， 在一个有符号二进制数的补码形式。 |
| <<     | 左移动运算符   | a << 2 输出结果 240 ，二进制解释： 1111 0000                 |
| >>     | 右移动运算符   | a >> 2 输出结果 15 ，二进制解释： 0000 1111                  |
| >>>    | 无符号右移     | A >>>2 输出结果 15, 二进制解释: 0000 1111                    |

- 说明: 位运算符的规则和`Java`一样
- `Scala`不支持三目运算符 , 在`Scala` 中使用` if – else` 的方式实现。`（a>b?a:b）`

#### 2.5.6，运算符优先级

单目运算，参与运算的数值只有一个。双目运算有两个。

| 类别 | 运算符                                 | 关联性     |
| ---- | -------------------------------------- | ---------- |
| 1    | () []                                  | 左到右     |
| 2    | **! ~**                                | **右到左** |
| 3    | * / %                                  | 左到右     |
| 4    | + -                                    | 左到右     |
| 5    | >> >>> <<                              | 左到右     |
| 6    | > >= < <=                              | 左到右     |
| 7    | == !=                                  | 左到右     |
| 8    | &                                      | 左到右     |
| 9    | ^                                      | 左到右     |
| 10   | \|                                     | 左到右     |
| 11   | &&                                     | 左到右     |
| 12   | \|\|                                   | 左到右     |
| 13   | **= += -= *= /= %= >>= <<= &= ^= \|=** | **右到左** |
| 14   | ,                                      | 左到右     |

- 运算符有不同的优先级，所谓优先级就是表达式运算中的运算顺序。如右表，上一行运算符总优先于下一行。  
-  只有单目运算符、赋值运算符是从右向左运算的。
- 运算符的优先级和Java一样。
- 小结：
  - 小结运算符的优先级
  - () [] 可以直接提示优先级
  - 单目运算 (! ~)
  - 算术运算符.
  - 位移运算
  - 关系运算符(比较运算符)
  - 位运算
  - 逻辑运算
  - 赋值运算

### 2.6，键盘输入语句

- 案例演示

~~~java
object inputdemo {
  def main(args: Array[String]): Unit = {
    println("输入名字：")
    var name=StdIn.readLine()
    println("输入年龄：")
    var age=StdIn.readInt()
    println("name="+name)
    println("age="+age)
  }
}
~~~

### 2.7，流程控制

- Scala语言中控制结构和Java语言中的控制结构基本相同，在不考虑特殊应用场景的情况下，代码书写方式以及理解方式都没有太大的区别
  - 顺序控制
  - 分支控制（单分支, 双分支，多分支）
  - 循环控制

#### 2.7.1，顺序控制

- 注意事项

  - Scala中定义变量时采用合法的前向引用。如：

  ~~~ java
  def main(args : Array[String]) : Unit = {
          var num1 = 12
          var num2 = num1 + 2
  }
  //错误形式：
  def main(args : Array[String]) : Unit = {
          var num2 = num1 + 2
          var num1 = 12
  }
  ~~~

  - 但是方法和类可以前项引用。

#### 2.7.2，分支控制`if-else`介绍

- 查看某一个包下面有哪些类

~~~java
scala.io.StdIn//光标定在io出，按ctrl+b,光标定在Stdin出，看的是源码
~~~

- 让程序有选择的的执行,分支控制有三种:
  - 单分支 
  - 双分支 
  - 多分支

1. 单分支

~~~ java
基本语法
if (条件表达式) {
	执行代码块
}
说明：当条件表达式为ture 时，就会执行 { } 的代码。
~~~

2. 双分支

~~~java
基本语法
     if (条件表达式) {
	执行代码块1
      } else {
	执行代码块2
      }
说明：当条件表达式成立，即执行代码块1，否则执行代码块2.

~~~

3. 多分支

~~~ java
基本语法
      if (条件表达式1) {
	执行代码块1
      }
      else if (条件表达式2) {
	执行代码块2
      }
       ……
       else {
	执行代码块n
       }
//说明：当条件表达式1成立时，即执行代码块1，如果表达式1不成立，才去判断表达式2是否成立，如果表达式2成立，就执行代码块2，以此类推，如果所有的表达式都不成立，则执行 else 的代码块，注意，只能有一个执行入口。
~~~

4. 分支控制`if-else `注意事项

   1. 如果大括号`{}`内的逻辑代码只有一行，大括号可以省略, 这点和`java `的规定一样。
   2. `Scala`中任意表达式都是有返回值的，也就意味着`if else`表达式其实是有返回结果的，具体返回结果的值取决于满足条件的代码体的最后一行内容.
   3. `Scala`中是没有三元运算符，因为可以这样简写

   ~~~ java
   // Java
   int result = flg ? 1 : 0
   // Scala
   val result = if (flg) 1 else 0   // 因为 scala 的if-else 是有返回值的，因此，本身这个语言也不需要三元运算符了(如图)，并且可以写在同一行，类似 三元运算
   ~~~

5. 嵌套分支

   - 在一个分支结构中又完整的嵌套了另一个完整的分支结构，里面的分支的结构称为内层分支外面的分支结构称为外层分支。嵌套分支不要超过3层

~~~ java
基本语法
if(){        
	if(){        }
	else{        }
}

~~~

6. 类似`switch`分支结构
   - 在`scala`中没有`switch`,而是使用模式匹配来处理。模式匹配涉及到的知识点较为综合，因此我们放在后面讲解。`match-case `[有一个专题来讲解...]

#### 2.7.3，循环

1. `for`循环

   - `Scala` 也为`for` 循环这一常见的控制结构提供了非常多的特性，这些`for `循环的特性被称为`for` 推导式（`for comprehension`）或`for `表达式（`for expression`）
   - `for`循环中的`i`默认是`val`类型的。
   - `for`循环`i`变量内部自动会加。
   - **范围数据循环方式1**

   ~~~ java
   object forDemo {
     def main(args: Array[String]): Unit = {
       var start=0
       var end=10
       for (i <- start to end)
         {
           println("hello word:"+i)
         }
     }
   }
   //start是从那个数开始循环，end是那个数结束循环，to是关键字，start to end 前后闭合
   ~~~

   - 说明
     `i` 表示循环的变量， `<-` 规定好 `to `规定
     `i `将会从 `1-3 `循环， 前后闭合 （包括1 和 3）

   - **范围数据循环方式2**

   ~~~ java
   for(j <- start until end){
         println("hello word:"+j)
       }
   //左闭右开
   ~~~

   - 说明:
     这种方式和前面的区别在于` i` 是从`1 `到` (3-1)`
     前闭合后开的范围,和java的arr.length() 类似`for (int i =0; i < arr.lenght(); i++ ){}`
   - **循环守卫**

   ~~~ java
   for(j <- start to end if j%2==0){
         println("hello word:"+j)
       }
   //输出
   hello word:0
   hello word:2
   hello word:4
   hello word:6
   hello word:8
   hello word:10
   ~~~

   - 循环守卫，即循环保护式（也称条件判断式，守卫）。保护式为`true`则进入循环体内部，为`false`则跳过，类似于`continue`
     上面的代码等价

   ~~~ java
   for (i<-1 to 3){	
       if ( i != 2) {	
        println(i+" ")	
       }
   }
   ~~~

   - **引入变量**

   ~~~ java
    for (m <- 1 to 3;n=4-m) {
         print(n)
       }
   //输出
   321
   ~~~

   - 对基本案例说明
     没有关键字，所以范围后一定要加；来隔断逻辑
     上面的代码等价

   ~~~ java
   for (i <- 1 to 3) {
       var j = 4 – i 
       print(j + “ “)
   }
   ~~~

   - **嵌套循环**

   ~~~ java
   for(i <- 1 to 3; j <- 1 to 3) {
     println(" i =" + i + " j = " + j)
   }
   ~~~

   - 对基本案例说明
     没有关键字，所以范围后一定要加；来隔断逻辑
     上面的代码等价

   ~~~ java
   for (i <- 1 to 3) {
       for (j <-1 to 3) {	
           println("ok")
       }
   }
   ~~~

   - **循环返回值**返回一个`vector`集合

   ~~~ java
   val res = for(i <- 1 to 10) yield i * 2
   println(res)
   //将变量i从-1到10循环，yield关键字标示将后面的值放入vector中并且返回给res，下面这种语法体现出scala语法的特点，就是将一个集合中的数据进行处理，返回一个新的集合
   val res = for(i <- 1 to 10) yield{
         if(i%2==0){
           i
         }else{
           0
         "不是偶数"
     }
   }
       println(res)     
   Vector(0, 2, 0, 4, 0, 6, 0, 8, 0, 10)
   ~~~

   - 对基本案例说明
     将遍历过程中处理的结果返回到一个新`Vector`集合中，使用`yield`关键字
   - **使用花括号{}代替小括号()**

   ~~~ java
    for{i <- 1 to 3; j =  i * 2}{
         println(" i= " + i + " j= " + j)
       }
   ~~~

   - 对基本案例说明
     `{}`和`()`对于`for`表达式来说都可以
     `for `推导式有一个不成文的约定：当`for` 推导式仅包含单一表达式时使用圆括号，当其包含多个表达式时使用大括号,当使用`{} `来换行写表达式时，**分号**就不用写了

   ~~~ java
    for{
         i <- 1 to 3
         j = i * 2
       }{
         println(" i= " + i + " j= " + j)
       }
   ~~~

   - 注意事项和细节说明
     - `scala `的`for`循环形式和`java`是较大差异，这点请同学们注意，但是基本的原理还是一样的。
     - `scala `的`for`循环的步长如何控制`! [for(i <- Range(1,3,2)]`
     - 思考题：如何使用循环守卫控制步长
   - `for`循环步长控制，两种方式

   ~~~ java
   //for循环步长控制，默认为1
       for(i <- Range(0,10,2){
         println(i)
       }
   Range(start,end,step)
   val Range = scala.collection.immutable.Range//range是一个不可变的集合
   //构建方法
   def apply(start: Int, end: Int, step: Int): Range = new Range(start, end, step)//源码
   ~~~

   ~~~ java
    //for循环守卫控制步长
       for(i <- 0 to 10 if(i%2==0)){
         println("i="+i)
       }
   //只有当if语句为true时才执行输出语句，相当于步长为2
   ~~~

2. `while`循环控制

   `List`源码中用到了`while`循环。

   ~~~ java
   循环变量初始化 //循环的四个要素
   while (循环条件) {
              循环体(语句)
              循环变量迭代
   }
   
   ~~~

   - 注意事项和细节说明
     - 循环条件是返回一个布尔值的表达式

     - `while`循环是先判断再执行语句
     - 与`If`语句不同,`While`语句本身没有值,即整个`While`语句的结果是`Unit`类型的`()`
     - 因为`while`中没有返回值,所以当要用该语句来计算并返回结果时,就不可避免的使用变量 ，而变量需要声明在`while`循环的外部，那么就等同于循环的内部对外部的变量造成了影响，所以不推荐使用，而是推荐使用`for`循环。

3. `do..while`循环控制

   ~~~ java
   基本语法
   
     循环变量初始化;
      do{
                  循环体(语句)
                   循环变量迭代
      } while(循环条件)
   
   
   do...while循环应用实例
   ~~~

   - 注意事项和细节说明
     - 循环条件是返回一个布尔值的表达式
     - `do..while`循环是先执行，再判断
     - 和`while` 一样，因为`do…while`中没有返回值,所以当要用该语句来计算并返回结果时,就不可避免的使用变量 ，而变量需要声明在`do...while`循环的外部，那么就等同于循环的内部对外部的变量造成了影响，所以不推荐使用，而是推荐使用`for`循环。

4. 多层循环

   1. 将一个循环放在另一个循环体内，就形成了嵌套循环。其中，`for ,while ,do…while`均可以作为外层循环和内层循环。【建议一般使用两层，最多不要超过3层】

   2. 实质上，嵌套循环就是把内层循环当成外层循环的循环体。当只有内层循环的循环条件为`false`时，才会完全跳出内层循环，才可结束外层的当次循环，开始下一次的循环。

   3. 设外层循环次数为`m`次，内层为`n`次， 则内层循环体实际上需要执行 `m * n`次。

5. 循环的中断

   - `Scala`内置控制结构特地去掉了`break`和`continue`，是为了更好的适应函数化编程，推荐使用函数式的风格解决`break`和`contine`的功能，而不是一个关键字。

   - `break()`函数也可以用在`for`循环和`do while`循环，用法和`while`一样。

     ~~~ java
     //源码解读
      def break(): Nothing = { throw breakException }
     //break（）函数使用抛出异常来中断循环
     //实例
     object WhileTest {
       def main(args: Array[String]): Unit = {
         var i=0
         //breakable()
         // 1 是一个高阶函数，参数可以接受函数的函数
         //2 op: => Unit标示接受的参数是一个没有输入，也没有返回值的函数，
         // 简单的理解为可以接受代码块
         //3 breakable对break()函数抛出的异常做了处理，代码可继续执行
         //当我们传入代码块的时候，breakable()中的小括号换成{}
         //breakable()源码
         //    def breakable(op: => Unit) {
         //      try {
         //        op
         //      } catch {
         //        case ex: BreakControl =>
         //          if (ex ne breakException) throw ex
         //      }
         //    }
         breakable(
           while(i <= 20) {
             i+=1
             if(i == 18){
               //中断操作,在scala中用break()函数中断循环
               break()
             }
           }
         )
       }
     }
     
     ~~~

   - `Scala`内置控制结构特地也去掉了`continue`，是为了更好的适应函数化编程，可以使用`if – else` 或是 循环守卫实现`continue`的效果

     ~~~ java
     object MyContinue {
       def main(args: Array[String]): Unit = {
         //当i不等于2和5时，跳过，执行输出条件
         for(i <- 0 to 10 if(i != 2 && i != 5)){
           println("i="+i)
         }
         println("*******************************")
         for(i <- 0 to 10 ){
           if(i != 2 && i != 5)
           println("i="+i)
         }
       }
     }
     ~~~

### 2.8，函数式编程（基础部分）

#### 2.8.1，方法与函数

- 函数式编程基础
  - 函数定义/声明
  - 函数运行机制
  - 递归  
  - 过程
  - 惰性函数和异常
- 函数式编程高级
  - 值函数(函数字面量)  
  - 闭包 
  - 应用函数 
  - 抽象控制
- 在`scala`中，函数式编程和面向对象编程融合在一起，学习函数式编程式需要`oop`的知识，同样学习`oop`需要函数式编程的基础。
- 关系如下图:

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195045-111945.png)

1. 在学习`Scala`中将方法、函数、函数式编程和面向对象编程明确一下：

   1. 在`scala`中，方法和函数几乎可以等同(比如他们的定义、使用、运行机制都一样的)，只是函数的使用方式更加的灵活多样  [方法转函数]。

   2. 函数式编程是从编程方式(范式)的角度来谈的，可以这样理解：函数式编程把函数当做一等公民，充分利用函数、 支持的函数的多种使用方式。比如：在`Scala`当中，函数是一等公民，像变量一样，既可以作为函数的参数使用，也可以将函数赋值给一个变量. ，函数的创建不用依赖于类或者对象，而在`Java`当中，函数的创建则要依赖于类、抽象类或者接口.。

   3. 面向对象编程是以对象为基础的编程方式。

   4. 在`scala`中函数式编程和面向对象编程融合在一起了 。

   5. 在学习Scala中将方法、函数、函数式编程和面向对象编程关系分析图：

   ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195047-975677.png)

   - 一段代码如果在面向`oop`编程，就称为方法，如果在面向函数编程，就叫做函数。

   ~~~ java
   //方法转换成函数
   object Method {
     def main(args: Array[String]): Unit = {
       val dog=new Dog
       var sum=dog.getSum(20,30)
       println(sum)
       //现在把dog中getSum()方法转换成函数
       var f1=dog.getSum _
       println("f1="+f1)
       //使用函数直接调用即可
       println("f1="+f1(50,60))
   
       //函数，求两个数的和，相当于把函数给了f2
       var f2=(num1:Int,num2:Int)=>{
         //函数体
         num1+num2
       }
       println("f2="+f2)
       println("f2="+f2(50,60))
     }
   }
   class Dog{
     def getSum(num1:Int,num2:Int) :Int={
       return num1 + num2
     }
   }
   ~~~

#### 2.8.2，面向函数编程

1. 函数式编程介绍

   1. "函数式编程"是一种"编程范式"（programming paradigm）。

   2. 它属于"结构化编程"的一种，主要思想是把运算过程尽量写成一系列嵌套的函数调用。

   3. 函数式编程中，将函数也当做数据类型，因此可以接受函数当作输入（参数）和输出（返回值）。（增强了编程的粒度）

   4. 函数式编程中，最重要的就是函数。

2. 为什么需要函数

   - 请大家完成这样一个需求: 
     输入两个数,再输入一个运算符(+,-)，得到结果.。

   - 先使用传统的方式来解决，看看有什么问题没有？
     - 代码冗余 
     - 不利于代码的维护

   ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195243-32031.png)

3. 函数/方法的定义

   - 语法

   ~~~ java
   `def` 函数名 ([参数名: 参数类型], ...)[[: 返回值类型] =] {
   	语句... //完成某个功能
   	`return` 返回值
   }
   ~~~

   1. 函数声明关键字为`def  (definition)`
   2. [参数名: 参数类型], ...：表示函数的输入(就是参数列表), 可以没有。 如果有，多个参数使用逗号间隔
   3. 函数中的语句：表示为了实现某一功能代码块
   4. 函数可以有返回值,也可以没有
   5. 返回值形式1:  `// def` 函数名(参数列表) : 数据类型 = {函数体}  // 返回值确定,清晰   
   6. 返回值形式2:  `// def` 函数名(参数列表) = {函数体}  // 有返回值, 类型是推断出来的
   7. 返回值形式3:  `// def `函数名(参数列表) {函数体}  // 无返回值` Unit`
   8. 如果没有`return` ,默认以执行到最后一行的结果作为返回值

   ~~~ java
   //快速入门
   object MyFunction {
     def main(args: Array[String]): Unit = {
       getSum(10,20,'+')
     }
     //如果在这里返回值定义为Int会报错，应为如果输入不是+和-，那么返回为null不是int类型，所以在这里
     //如果省略返回值类型，让编译器自己推断，即可
     def getSum(num1:Int,num2:Int,oper:Char)={
       if(oper == '+'){
         num1 + num2
       }else if(oper == '-'){
         num1- num2
       }else
         {
           null
         }
     }
   50
   f1=<function2>
   f1=110
   f2=<function2>
   f2=110
   ~~~

4. 函数的基本运行原理

   - 结合`jvm`栈理解。

   ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195246-572347.png)

5. 递归算法

   - 函数递归需要遵守的重要原则（总结）:
     - 程序执行一个函数时，就创建一个新的受保护的独立空间(新函数栈桢)
     - 函数的局部变量是独立的，不会相互影响
     - 递归必须向退出递归的条件逼近，否则就是无限递归，会造成死循环
     - 当一个函数执行完毕，或者遇到return，就会返回，遵守谁调用，就将结果返回给谁。

6. 函数/方法注意事项和细节讨论

   1. 函数的形参列表可以是多个, 如果函数没有形参，调用时 可以不带`() `
   2. 形参列表和返回值的数据类型可以是值类型和引用类型。
   3. `Scala`中的函数可以根据函数体最后一行代码自行推断函数返回值类型。那么在这种情况下，`return`关键字可以省略。

   ~~~ java
   def getSum(n1: Int, n2: Int): Int = {
        n1 + n2
   }
   //省略return语句
   def getSum(n1: Int, n2: Int) = {
       n1 + n2
   }
   ~~~

   4. **因为`Scala`可以自行推断，所以在省略`return`关键字的场合，返回值类型也可以省略,前面冒号也要去掉**。

   5. 如果函数明确使用`return`关键字，那么函数返回就不能使用自行推断了,这时要明确写成 : 返回类型 =  ，当然如果你什么都不写，即使有`return `返回值为`()`。即如果写`return`语句，那么函数返回类型一定要写，编译器自动推断失效，如果不写函数返回类型，`return`语句可以省略，让编译器自己推断，但是如果不省略`return`语句，那么`return`语句也会失效，返回空值`()`。

      ~~~ java
      object FunctionTest {
        def main(args: Array[String]): Unit = {
          println(getSum(10,20))
        }
        def getSum(n1:Int,n2:Int){
          return n1+n2
        }
      }
      //输出（）
      ~~~

   6. 如果函数明确声明无返回值（声明Unit），那么函数体中即使使用return关键字也不会有返回值

      ~~~ java
       def getSum1(n1:Int,n2:Int):Unit={
          return n1+n2
        }
      //输出（）
      ~~~

   7. 如果明确函数无返回值或不确定返回值类型，那么返回值类型可以省略(或声明为Any)

      ~~~ java
      def f3(s: String) = {
          if(s.length >= 3)
             s + "123"//返回字符串
          else
             3//返回int
      }
      //返回nothing版本
      def f4(s: String): Any = {
          if(s.length >= 3)
             s + "123"
          else
             3
      }
      ~~~

   8. Scala语法中任何的语法结构都可以嵌套其他语法结构(灵活)，即：函数/方法中可以再声明/定义函数/方法，类中可以再声明类。

      ~~~ java
      //函数嵌套
      object FunctionTest01 {
        def main(args: Array[String]): Unit = {
          //主函数内部定义的函数和外面定义的函数地位是一样的
          //底层源码标示：private final void sayHello$1()
          def sayHello():Unit={
            println("hello")
          }
          println("word")
          //在主函数中还可以定义与外面函数一样的函数
          def sayBye():Unit={//private final sayBye$1()
            //private final void sayBye$1()
            println("bye")
            //函数中还可以定义函数
          //  private final void sayBye$2()
            def sayBye():Unit={//private final sayBye$2()
              println("bye")
            }
          }
        }
        def sayBye():Unit={
          println("bye")
        }
      }
      //底层存储形式
      package qq.com.rzf.chapter02;
      
      import scala.Predef.;
      //类
      public final class FunctionTest01$
      {
        public static final  MODULE$;
      
        static
        {
          new ();
        }
      //主方法
        public void main(String[] args)
        {
          Predef..MODULE$.println("word");
        }
      //类中的方法
        public void sayBye()
        {
          Predef..MODULE$.println("bye");
        }
      //主函数中的方法
        private final void sayHello$1()
        {
          Predef..MODULE$.println("hello");
        }
      //方法中的方法
        private final void sayBye$2()
        {
          Predef..MODULE$.println("bye");
        }
      //主函数中定义的方法和类中的方法一直
        private final void sayBye$1()
        {
          Predef..MODULE$.println("bye");
        }
      
        private FunctionTest01$()
        {
          MODULE$ = this;
        }
      }
      ~~~

   9. 递归函数未执行之前是无法推断出来结果类型，在使用时必须有明确的返回值类型

      ~~~ java
      def f8(n: Int) : Int = { //递归不可以使用类型推断，必须指明返回类型
          if(n <= 0)      
              1  else     
                  n * f8(n - 1)
              }
      
      ~~~

   10. Scala函数的形参，在声明参数时，直接赋初始值(默认值)，这时调用函数时，如果没有指定实参，则会使用默认值。如果指定了实参，则实参会覆盖默认值。

       ~~~ java
        def getName(name:String="tom"):String={
           name+"ok"
         }
       //输出
       tomok
       ~~~

   11. 如果存在多个参数，每一个参数都可以设定默认值，那么这个时候，传递的参数到底是覆盖默认值，还是赋值给没有默认值的参数，就不确定了(默认按照声明顺序[从左到右])。在这种情况下，可以采用带名参数 

       ~~~ java
       object FunctionTest02 {
       
         def main(args: Array[String]): Unit = {
           //默认传参数从左到右
           mysqlCon("localhost",777)
           //但是如果想具体给某一个参数传值，需要指明
           mysqlCon(user="rzf",pwd ="root")
         }
       
         def mysqlCon(url: String = "jdbc:mysql://localhost",
                      port: Int = 3306,
                      user: String = "root",
                      pwd: String = "root"): Unit = {
           println("host=" + host)
           println("port=" + port)
           println("user=" + user)
           println("pwd=" + pwd)
         }
       }
       //面试题
       def f6 ( p1: String = "v1", p2: String ) {//p1有默认值，但是p2没有
             println(p1 + p2);
       }
       f6("v2" ) //报错，应为p2没有默认值，但是也没有传参数
       f6(p2="v2") //输出v1v2,使用了代理参数
       ~~~

   12. `scala`形参默认是`val`类型，因此传入函数不可以进行修改操作。

   13. 可变参数

       ~~~ java
       /支持0到多个参数
       def sum(args: Int*) : Int = { }
       
       //支持1到多个参数
       def sum(n1: Int, args:  Int*) : Int  = { }
       ~~~

       - 说明: 
         - `args` 是集合, 通过` for`循环 可以访问到各个值。
         - 可变参数需要写在形参列表的最后。

       ~~~ java
       object varParameaters {
         def main(args: Array[String]): Unit = {
           println(getSum(1,2,3,4,5,6,7,8,9,10))
         }
         def getSum(num:Int,args:Int*):Int ={
           var sum:Int=num
           for(item <- args){
             sum +=item
           }
         return sum
         }
       }
       ~~~

       - 练习

         ~~~ java
         def f1="asdfgh"和
           def f1()={
             "asdfgh"
           }
           等价，因为参数列表为空，所以可以省略（），应为函数体只有一句话，所以可以省略{}
         ~~~

   14. 过程

       - 将函数返回类型为`unit`的函数称为过程，如果明确函数没有返回值，那么等号也是可以省略的。
       - 如果函数声明时没有返回值类型，但是有`=`，可以进行类型推断最后一段代码，这时这个函数实际是有返回值的，该函数并不是过程，
       - 开发工具的自动代码补全功能，虽然会自动加上Unit，但是考虑到Scala语言的简单，灵活，最好不加.

   15. 惰性函数

       - 看一个应用场景
         惰性计算（尽可能延迟表达式求值）是许多函数式编程语言的特性。惰性集合在需要时提供其元素，无需预先计算它们，这带来了一些好处。首先，您可以将耗时的计算推迟到绝对需要的时候。其次，您可以创造无限个集合，只要它们继续收到请求，就会继续提供元素。函数的惰性使用让您能够得到更高效的代码。Java 并没有为惰性提供原生支持，Scala提供了。

         ~~~ java
         //惰性函数演示
         object LazyFunction {
           def main(args: Array[String]): Unit = {
              val res = sum(10, 20)//这里没有添加lazy声明
             println("-----------------")
             //println("res=" + res)
           }
           def sum(n1 : Int, n2 : Int): Int = {
             println("sum() 执行了..")
             return  n1 + n2
           }
         }
         //执行结果
         sum() 执行了..
         -----------------
         //解释：应为对res变量没有添加lazy声明，可以看到直接运行此程序sum()函数已经被执行，输出 println("sum() 执行了..")语句，
         //现在添加lazy声明
         object LazyFunction {
           def main(args: Array[String]): Unit = {
             lazy val res = sum(10, 20)//添加lazy声明
             println("-----------------")
             //println("res=" + res)
           }
           def sum(n1 : Int, n2 : Int): Int = {
             println("sum() 执行了..")
             return  n1 + n2
           }
         }
         //输出结果
         -----------------
         //添加lazy声明发现此函数也没有执行，因为没有输出 println("sum() 执行了..")语句，解释：这里只是把sum函数加载到缓存，并没有真正的执行函数，等到什么时候需要用到sum()函数的执行结果的时候才真正执行函数
         //现在打印输出结果
         object LazyFunction {
           def main(args: Array[String]): Unit = {
             lazy val res = sum(10, 20)//有lazy声明
             println("-----------------")
             println("res=" + res)//在这里用到res结果
           }
           def sum(n1 : Int, n2 : Int): Int = {
             println("sum() 执行了..")
             return  n1 + n2
           }
         }
         //输出结果
         -----------------
         sum() 执行了..
         res=30
         //解释：执行主函数，但是在执行调用函数语句时，并没有执行调用函数，而是进行缓存，等到println("res=" + res)语句执行需要用到res结果的时候才去执行调用函数，所以才叫做惰性函数。，从返回的结果可以看到情况。
         ~~~

         - 注意事项和细节
           - lazy 不能修饰 var 类型的变量。只能修饰`val`类型，所以是线程安全的。
           - 不但是 在调用函数时，加了 lazy ,会导致函数的执行被推迟，我们在声明一个变量时，如果给声明了 lazy ,那么变量值得分配也会推迟。 比如 lazy val i = 10，此时变量并没有分配空间，只有在使用变量的时候才立即分配空间。
           - 应用场景：可以将耗时间的计算推迟到绝对需要的时候使用。（优化）

       16. 异常

           - Scala提供try和catch块来处理异常。try块用于包含可能出错的代码。catch块用于处理try块中发生的异常。可以根据需要在程序中有任意数量的try...catch块。

           - 语法处理上和Java类似，但是又不尽相同。

           - 在`java`中异常分为运行时异常和编译时异常，但是在`scala`中简化处理，只有运行时异常。

           - `java`异常处理结构

             ~~~ java
             public class Myexception {
                 public static void main(String[] args) {
                     try {
                         // 可疑代码
                         int i = 0;
                         int b = 10;
                         int c = b / i; // 执行代码时，会抛出ArithmeticException异常
                         //小的异常写在前面，大的异常写在后面，不可以相反，但是在scala中可以相反
                     } catch (ArithmeticException ex)
                     {
                         ex.printStackTrace();
                     }catch(Exception e)  {
                         e.printStackTrace();
                     } finally {
                         // 最终要执行的代码
                         System.out.println("java finally");
                     }
                     System.out.println("捕获异常，继续执行");
                 }
             }
             //异常捕获后，保证代码可以继续执行
             ~~~

             - Java异常处理的注意点.

               - java语言按照try—catch-catch...—finally的方式来处理异常。

               - 不管有没有异常捕获，都会执行finally, **因此通常可以在finally代码块中释放资源**

               - 可以有多个catch，分别捕获对应的异常，这时需要把范围小的异常类写在前面，把范围大的异常类写在后面，否则编译错误。会提示 "Exception 'java.lang.xxxxxx' has already been caught"

             - `Scala`异常处理举例

               ~~~ java
               object ScalaException {
                 def main(args: Array[String]): Unit = {
                   //在scala中只有一个catch块，并且在catch中有多个case,每一个case匹配一个异常
                   // => println("捕获了异常")表是捕获异常后对异常的处理，可以为一个代码块，用{}括起来即可
                   try {
                     val r = 10 / 0
                   } catch {
                     case ex: ArithmeticException=> println("捕获了除数为零的算术异常")
                       case ex: Exception => println("捕获了异常")
                       } finally {
                       // 最终要执行的代码
                       println("scala finally...")
                       }
               
                       }
               }
               ~~~

               - 细节问题

                 - 我们将可疑代码封装在try块中。 在try块之后使用了一个catch处理程序来捕获异常。如果发生任何异常，catch处理程序将处理它，程序将不会异常终止。
                 - Scala的异常的工作机制和Java一样，但是Scala没有“checked(编译期或受检)”异常，即Scala没有编译异常这个概念，异常都是在运行的时候捕获处理。
                 - 用throw关键字，抛出一个异常对象。所有异常都是Throwable的子类型。throw表达式是有类型的，就是Nothing，因为Nothing是所有类型的子类型，所以throw表达式可以用在需要类型的地方

                 ~~~ java
                 object MyScalaException {
                   def main(args: Array[String]): Unit = {
                 //    val res = test()
                 //    println(res.toString)
                 //    println("异常发生了")//此种方式知道test()函数发生了异常，但是并没有处理，所以println()语句并不会执行
                     //捕获并处理异常
                     try{
                       val res = test()//可能发生异常，用try包围
                     }catch {
                       //对异常进行处理
                       case ex:Exception=>{println("我把异常处理了")}
                     }
                     println("异常发生了")//现在此打印语句可以正常执行
                   }
                   def test(): Nothing = {
                     throw new Exception("不对")
                   }
                 }
                 ~~~

               - 在`Scala`里，借用了模式匹配的思想来做异常的匹配，因此，在`catch`的代码里，是一系列`case`子句来匹配异常。【前面案例可以看出这个特点, 模式匹配我们后面详解】，当匹配上后` => `有多条语句可以换行写，类似 `java `的` switch case x: `代码块..

               - 异常捕捉的机制与其他语言中一样，如果有异常发生，catch子句是按次序捕捉的。因此，在`catch`子句中，越具体的异常越要靠前，越普遍的异常越靠后，如果把越普遍的异常写在前，把具体的异常写在后，在`scala`中也不会报错，但这样是非常不好的编程风格。

               - `finally`子句用于执行不管是正常处理还是有异常发生时都需要执行的步骤，一般用于对象的清理工作，这点和`Java`一样。

               -  `Scala`提供了`throws`关键字来声明异常。可以使用方法定义声明异常。 它向调用者函数提供了此方法可能引发此异常的信息。 它有助于调用函数处理并将该代码包含在`try-catch`块中，以避免程序异常终止。在`scala`中，可以使`throws`注释来声明异常

               ~~~ java
               def main(args: Array[String]): Unit = {
                   f11()
               }
               //@throws注解表名向使用此函数的对象声明，f11函数会发生NumberFormatException异常
               @throws(classOf[NumberFormatException])
               def f11()  = {
                   "abc".toInt
               }
               
               ~~~

### 2.9，面向对象编程（基础部分）

1. `Scala`语言是面向对象的

   1. `Java`是面向对象的编程语言，由于历史原因，`Java`中还存在着非面向对象的内容:基本类型`(int,float..) `，`null`，静态方法等。 
   2. `Scala`语言来自于`Java`，所以天生就是面向对象的语言，而且`Scala`是纯粹的面向对象的语言，即在`Scala`中，一切皆为对象。 
   3. 在面向对象的学习过程中可以对比着`Java`语言学习

2. 案例：

   ~~~ java
   object Oop {
     def main(args: Array[String]): Unit = {
     //创建对象
       val cat=new Cat
       cat.name="xuaihua"
       //在这里其实不是真正访问属性，而是调用了函数cat.age_$eq(10)
       //cat.name等价于cat.name(),底层调用的是方法
       cat.age=10
       cat.color="white"
       printf("\n信息如下：%s \t %d \t %s",cat.name,cat.age,cat.color)
     }
   }
   class Cat{
     //对象属性默认是私有的，当我们声明 var name:String=""，在底层字节码存储为private name
     //1同时会生成两个public函数public name()函数，对应java中的get()方法
     //2 public name_$eq(),对应java中的set()方法
     //3 如果只有一个calss类，仅仅会生成一个.class文件，如果有object声明，会生成两个.class文件
     var name:String=""//scala中变量必须有初始值
   
     var age:Int= _//_下划线标示给age设置默认初始值，int类型默认为0
     var color:String= _//string默认初始值为""
   }
   //cat类底层源码
   import scala.reflect.ScalaSignature;
   //类默认添加public 
   public class Cat
   {
       //属性在底层全部添加private
     private String name = "";
     private int age;
     private String color;
     //以下方法全部都是自动生成
   //相当与get()方法
     public String name()
     {
       return this.name; } 
       //相当于set()方法
     public void name_$eq(String x$1) { this.name = x$1; } 
     public int age() {
       return this.age; } 
     public void age_$eq(int x$1) { this.age = x$1; } 
     public String color() { return this.color; } 
     public void color_$eq(String x$1) { this.color = x$1; }
   
   }
   ~~~

3. 类的定义

   ~~~ java
   [修饰符] class 类名 {
      类体
   } 
   ~~~

   - 定义类的注意事项

     - scala语法中，类并不声明为public，所有这些类都具有公有可见性(即默认就是public)
     - 一个Scala源文件可以包含多个类.

   - 属性/成员变量

     - 属性的定义语法同变量，示例：[访问修饰符] var 属性名称 [：类型] = 属性值
     - 属性的定义类型可以为任意类型，包含值类型[AnyVal]或引用类型[AnyRef]
     - 给属性设置默认值时用`_`时，必须指出属性的类型。不指明类型的默认值，编译器会根据值的类型去推断属性的类型。
     - Scala中声明一个属性,必须显示的初始化，然后根据初始化数据的类型自动推断，属性类型可以省略(这点和Java不同)。[案例演示] 

     - 如果赋值为null,则一定要加类型，因为不加类型, 那么该属性的类型就是Null类型.

     ~~~ java
     class Cat{
       //对象属性默认是私有的，当我们声明 var name:String=""，在底层字节码存储为private name
       //1同时会生成两个public函数public name()函数，对应java中的get()方法
       //2 public name_$eq(),对应java中的set()方法
       //3 如果只有一个calss类，仅仅会生成一个.class文件，如果有object声明，会生成两个.class文件
       var name=null//scala中变量必须有初始值
       var age:Int= _//_下划线标示给age设置默认初始值，int类型默认为0
       var color:String= null//string默认初始值为""
     }
     //name初始化为null,但是没有指出类型， cat.name="xuaihua"，给name赋值会报错，应为编译器会推断name类型为null类型，但是对于color属性，已经显示指明类型，但是里面的值为null，属性类型依旧是string类型
     ~~~

     - 如果在定义属性时，暂时不赋值，也可以使用符号_(下划线)，让系统分配默认值.

       | **类型**            | **_** **对应的值** |
       | ------------------- | ------------------ |
       | Byte Short Int Long | 0                  |
       | Float Double        | 0.0                |
       | String 和 引用类型  | null               |
       | Boolean             | false              |

     - 不同对象的属性是独立，互不影响，一个对象对属性的更改，不影响另外一个。案例演示+图(Monster) //这点和java完全一样(即对象和对象之间相互独立)。

   - 属性的高级部分

     - 说明：属性的高级部分和构造器(构造方法/函数) 相关，我们把属性高级部分放到构造器那里讲解。

4. 如何创建对象

   ~~~ java
   基本语法
   val | var 对象名 [：类型]  = new 类型()//此处的类型可以省略
   ~~~

   - 说明

     - 如果我们不希望改变对象的引用(即：内存地址), 应该声明为`val` 性质的，否则声明为`var`,` scala`设计者推荐使用`val` ,因为一般来说，在程序中，我们只是改变对象属性的值，而不是改变对象的引用。
     - `scala`在声明对象变量时，可以根据创建对象的类型自动推断，所以类型声明可以省略，**但当类型和后面new 对象类型有继**承关系即多态时，就必须写了.

   - 如何访问属性:对象名.属性名; 

   - 类和对象的内存分配机制(重要)

     ~~~ java
     val p1 = new Person
     p1.name = "jack"
     p1.age = 30
     val p2 = p1   //内存布局?
     //内存布局如下图：在这里先new了一个person对象，此对象的内存空间在堆这种数据结构中分配，而在栈中分配新对象的引用，即存储地址的标识符，即p1实际存储的是对象在堆中的地址，然后又把p1的引用分配给了p2，即把p1的值给了p2,因此p1和p2都指向内存的同一块区域，应为存储的地址相同。通过p1和p2都可以访问堆中的对象。
     ~~~

     ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/195305-292278.png)

5. 方法

   - Scala中的方法其实就是函数，声明规则请参考函数式编程中的函数声明。

     ~~~ java
     基本语法
     def 方法名(参数列表) [：返回值类型] = { 
     	方法体
     }
     ~~~

   - 方法的调用机制原理

     - 当我们scala开始执行时，先在栈区开辟一个main栈。
     - 当scala程序在执行到一个方法时，总会创建一个新的栈桢。
     - 每个栈桢是独立的空间，变量（基本数据类型）是独立的，相互不影响（引用类型除外）
     -   当方法执行完毕后，该方法开辟的栈桢就会出栈。

6. 构造器

   - 构造器(constructor)又叫构造方法，是类的一种特殊的方法，它的主要作用是完成对新对象的初始化。

   - `java`构造器基本语法

     ~~~ java
     回顾-Java构造器基本语法
     [修饰符] 方法名(参数列表){
     	构造方法体
     }
     ~~~

     - 在Java中一个类可以定义多个不同的构造方法，构造方法重载
     - 如果程序员没有定义构造方法，系统会自动给类生成一个默认无参构造方法(也叫默认构造器)，比如 Person (){super();}
     - 一旦定义了自己的构造方法（构造器）,默认的构造方法就覆盖了，就不能再使用默认的无参构造方法，除非显式的定义一下,即:  Person(){};

   - `Scala`构造器的介绍

     - 和Java一样，Scala构造对象也需要调用构造方法，并且可以有任意多个构造方法（即scala中构造器也支持重载）。

     - Scala类的构造器包括： 主构造器（一个） 和 辅助构造器(多个)

     - 如果类有继承关系，构造器的执行顺序是，首先调用其父类构造器，然后调用本类的主构造器，最后调用本类的辅助构造器

     - Scala构造器的基本语法

       ~~~ java
       class 类名(形参列表) {  // 主构造器
          // 类体
          def  this(形参列表) {  // 辅助构造器
          }
          def  this(形参列表) {  //辅助构造器可以有多个...
          }
       } 
        辅助构造器 函数的名称this, 可以有多个，编译器通过不同参数（个数或类型）来区分.
       ~~~

     - 案例

       ~~~java
       object Constructor {
         def main(args: Array[String]): Unit = {
           //用主构造器初始化实例
           var p1=new People("rui",24)
           println(p1)
           //用辅助构造器初始化实例
           var p2=new People("xiaorui")
           println(p2)
         }
       }
       //在声明类的时候直接声明主构造器
       class People(inName:String,inAge:Int){
         //柱构造器其实就是把类中除了函数以外的内容全部执行一遍，从底层源码乐意看到
         var name:String=inName
         var age:Int=inAge
         println(age+10)
         println("***********************")
         //定义辅助构造器
         def this(name:String){
           //辅助构造器第一句话必须调用主构造器
           this("",28)   //这一句话相当于执行一次主构造器吧
           this.name=name
         }
         //重写toString()方法，便于输出
         override def toString: String = {
           "name="+this.name+" "+"age="+this.age
         }
       }
       //源码编译
       public class People
       {
         private String name;
         private int age;
       
         public String name()
         {
           return this.name; } 
         public void name_$eq(String x$1) { this.name = x$1; } 
         public int age() { return this.age; } 
         public void age_$eq(int x$1) { this.age = x$1; }
       
       
         public String toString()
         {
           return new StringBuilder().append("name=").append(name()).append(" ").append("age=").append(BoxesRunTime.boxToInteger(age())).toString();
         }
       //people类构造器，除了类中的方法，其余的全部执行一遍
         public People(String inName, int inAge)
         {
           this.name = inName;
           this.age = inAge;
           Predef..MODULE$.println(BoxesRunTime.boxToInteger(age() + 10));
           Predef..MODULE$.println("***********************");
         }
       //辅助类构造器，我们可以发现主构造器和辅助构造器的本质就是重载，即类的参数类型和个数不同
         public People(String name) {
           this("", 28);
           name_$eq(name);
         }
       }
       ~~~

   - Scala构造器注意事项和细节

     - Scala构造器作用是完成对新对象的初始化，构造器没有返回值。
     - 主构造器的声明直接放置于类名之后 
     - 主构造器会执行类定义中的所有语句（把类中写的语句放入到主构造器），这里可以体会到Scala
     - 函数式编程和面向对象编程融合在一起，即：构造器也是方法（函数），传递参数和使用方法和前面的函数部分内容没有区别
     - 如果主构造器无参数，小括号可省略，构建对象时调用的构造方法的小括号也可以省略
     - 辅助构造器名称为this（这个和Java是不一样的），多个辅助构造器通过不同参数列表进行区分， 在底层就是构造器重载。
     - 如果想让主构造器变成私有的，可以在()之前加上private，这样用户不能直接通过主构造器来构造对象了。但是用户还可以使用辅助构造器，应为辅助构造器为`public`
     - 辅助构造器的声明不能和主构造器的声明一致,会发生错误(即构造器名重复),属性个数不可以一样。

7. 属性高级

   - 构造器参数

     - Scala类的主构造器的形参未用任何修饰符修饰，那么这个参数是局部变量。

     - 如果参数使用val关键字声明，那么Scala会将参数作为类的私有的只读属性使用 

     - 如果参数使用var关键字声明，那么Scala会将参数作为类的成员属性使用,并会提供属性对应的xxx()[类似getter]/xxx_$eq()[类似setter]方法，即这时的成员属性是私有的，但是可读写。

       ~~~ java
       object ConDemo {
         def main(args: Array[String]): Unit = {
           var w1=new Worker("tom")
           w1.name//在这里不可以访问到inName
           var w2=new Worker1("jak")
           w2.inName//在这里可以访问到inName属性
           val worker = new Worker2("cat")
           worker.inName="xiaobai"
           //此时发现name已经被修改
           println(worker.name)
         }
       }
       //如果柱构造器是class Worker(inName:String)这样，那么inName就是一个局部变量，其作用于范围就是
       //主构造器内部
       class Worker(inName:String){
         var name:String=inName
       
       }
       //如果柱构造器是Worker1(val inName:String)这样，inName有val修饰，
       // 那么inName就是类的私有属性，只可以读取，不可以修改使用
       class Worker1(val inName:String){
         //查看底层代码发现此时inName已经变成一个私有的final属性，并且有一个方法可以提供访问属性
         var name:String=inName
       }
       //如果柱构造器是Worker2(var inName:String)这样，inName有var修饰，
       // 那么inName就是类的私有属性，可以读取和修改
       class Worker2(var inName:String){
         //查看底层代码发现此时inName已经变成一个私有的属性，并且生成可读可写的方法
         var name:String=inName
       }
       //反编译后的源代码
       public class Worker
       {
         private String name;
       
         public String name()
         {
           return this.name; } 
         public void name_$eq(String x$1) { this.name = x$1; } 
         public Worker(String inName) { this.name = inName; }
       
       }
       public class Worker1
       {
         private final String inName;//编程私有可读
         private String name;
       
         public String inName()
         {
           return this.inName;
         }
         public String name() { return this.name; } 
         public void name_$eq(String x$1) { this.name = x$1; } 
         public Worker1(String inName) { this.name = inName; }//读取方法
       
       }
       public class Worker2
       {
         private String inName;
         private String name = inName();//可读可写
       
         public String inName()
         {
           return this.inName; } 
         public void inName_$eq(String x$1) { this.inName = x$1; } 
         public String name() {
           return this.name; } 
         public void name_$eq(String x$1) { this.name = x$1; }
       
       
         public Worker2(String inName)
         {
         }
       }
       ~~~

     - Bean属性

       - JavaBeans规范定义了Java的属性是像getXxx（）和setXxx（）的方法。许多Java工具（框架）都依赖这个命名习惯。为了Java的互操作性。将Scala字段加@BeanProperty时，这样会自动生成规范的 setXxx/getXxx 方法。这时可以使用 对象.setXxx() 和 对象.getXxx() 来调用属性。

       - 注意:给某个属性加入@BeanPropetry注解后，会生成getXXX和setXXX的方法
         并且对原来底层自动生成类似xxx(),xxx_$eq()方法，没有冲突，二者可以共存。

         ~~~ java
         import scala.beans.BeanProperty
         
         object BeanDemo {
           def main(args: Array[String]): Unit = {
             val car = new car
             //运用注解生成的方法
             car.setName("baoma")
             //底层的方法还可以使用
             car.color="white"
             println("name="+car.getName+" "+"color="+car.color)
           }
         }
         class car{
           //给name属性添加注解,然后各个属性会自动生成get和set方法，但是并不和底层的方法冲突
           @BeanProperty var name:String= _
           @BeanProperty var color:String= _
         }
         ~~~

     - 对象创建的流程分析

       ~~~ java
       class Person {  
           var age: Short = 90 
           var name: String = _  
       def this(n: String, a: Int) {    
           this()    
           this.name = n    
           this.age = a  
       	}
       }
       var p : Person = new Person("小倩",20)
       ~~~

       - 加载类的信息(属性信息和方法信息), 如果父类也没有加载, 则由父到子加载父类
       - 在内存中(堆)给对象开辟空间
       - 使用父类的构造器(主构造器/辅助构造器)完成父类的初始化 (多个父类)
       - 使用本类的主构造器完成初始化(也就是按照类中的默认值进行初始化)
       - 使用本类的辅助构造器继续初始化（按照new对象时候的赋值进行初始化）
       - 将对象在内存中的地址赋给 p 这个引用





