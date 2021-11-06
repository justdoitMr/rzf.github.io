
# Scala 中级部分
<!-- TOC -->

- [Scala 中级部分](#scala-中级部分)
    - [第三章 包](#第三章-包)
        - [3.1，`Java`包](#31java包)
        - [3.2，`Scala`包](#32scala包)
            - [3.2.1，快速入门](#321快速入门)
            - [3.2.2，Scala包的特点概述](#322scala包的特点概述)
        - [第四章，面向对象部分](#第四章面向对象部分)
        - [4.1，面向对象编程三大特征](#41面向对象编程三大特征)
            - [4.1.1，封装](#411封装)
            - [4.1.2，继承](#412继承)
            - [4.1.3，覆写字段](#413覆写字段)
            - [4.1.5，匿名子类](#415匿名子类)
            - [4.1.6，静态属性和静态方法](#416静态属性和静态方法)
            - [4.1.7，接口](#417接口)
            - [4.1.8，嵌套类](#418嵌套类)
        - [第五章，隐式转换和隐式参数](#第五章隐式转换和隐式参数)
            - [5.1.1，提出问题](#511提出问题)
            - [5.1.2，隐式函数基本介绍](#512隐式函数基本介绍)
            - [5.1.3，隐式转换的注意事项和细节](#513隐式转换的注意事项和细节)
            - [5.1.4，隐式转换丰富类库功能](#514隐式转换丰富类库功能)
            - [5.1.5，隐式值](#515隐式值)
            - [5.1.6，隐式类](#516隐式类)
            - [5.1.7，隐式的转换时机](#517隐式的转换时机)
            - [5.1.8，隐式解析机制](#518隐式解析机制)
            - [5.1.9，隐式转换的前提](#519隐式转换的前提)

<!-- /TOC -->

## 第三章 包

### 3.1，`Java`包

1. 包的作用

   - 区分相同名字的类
   - 当类很多时,可以很好的管理类
   - 控制访问范围

2. 打包命令

   ~~~ java
   package com.atguigu;
   ~~~

3. 打包的本质分析

   - 实际上就是创建不同的文件夹来保存类文件

     ![](../img/scala/java包.png)

4. Java如何引入包

   ~~~java
   import  包;
   比如 import java.awt.*;
   我们引入一个包的主要目的是要使用该包下的类
   比如 import java.util.Scanner;  就只是引入一个类Scanner。
   ~~~

   - java中包名和源码所在的系统文件目录结构要一致，并且编译后的字节码文件路径也和包名保持一致。

### 3.2，`Scala`包

- **和Java一样，Scala中管理项目可以使用包，但Scala中的包的功能更加强大，使用也相对复杂些，下面我们学习Scala包的使用和注意事项。**

#### 3.2.1，快速入门

- 在两个包下面创建两个`Tiger`类，分别在主函数中使用类

~~~ java
object Test {
  def main(args: Array[String]): Unit = {
    var tiger1=new qq.com.rzf.xh.Tiger
    var tiger2=new qq.com.rzf.xm.Tiger
    println(tiger1)
    println(tiger2)
  }
}
//输出
qq.com.rzf.xh.Tiger@ea4a92b
qq.com.rzf.xm.Tiger@3c5a99da
~~~

#### 3.2.2，Scala包的特点概述

~~~ java
基本语法package 包名
~~~

1. `scala`包的作用

   1. Scala包的三大作用(和Java一样)， 区分相同名字的类。
   2.  当类很多时,可以很好的管理类
   3. 控制访问范围

2. Scala中包名和源码所在的系统文件目录结构要可以不一致，但是编译后的字节码文件路径和包名会保持一致(这个工作由编译器完成)。即`scala`源文件的路径和包名的路径无关。

3. Scala包的命名

   1. 命名规则：

      ~~~ JAVA
      //只能包含数字、字母、下划线、小圆点.,但不能用数字开头, 也不要使用关键字。
      demo.class.exec1  //错误 , 因为class是关键字
      demo.12a    // 错误，因为不能以数字开头
      ~~~

   2. 命名规范：

      一般是小写字母+小圆点
      机构类型名.机构或公司名.项目名.业务模块名

4. Scala会自动引入的常用包

   ~~~ java
   java.lang.*  
   scala包
   Predef包
   //查看包内容
    import scala.io//光标定位io，按ctrl+b
   ~~~

5. scala进行package 打包时，可以有如下形式。

   ![](../img/scala/打包方式.png)

   ~~~ java
   //第三种形式演示
   //包名com.rr，{}标示包里面的内容
   package com.rr{
     //在com.rr包路径下还有一个包：com.rr.scala
     package scala{
       //在com.rr.scala包下有一个类：Person
       //也就是说，在scala中可以在一个包中在创建一个包，类或者object对象
       class Person{
         val name = "Nick"
         def play(message: String): Unit ={
           println(this.name + " " + message)
         }
       }
     }
   
    object mypac{
      def main(args: Array[String]): Unit = {
        println("**************")
      }
    }
   }
   ~~~

6. 包也可以像嵌套类那样嵌套使用（包中有包）, 这个在前面的第三种打包方式已经讲过了，在使用第三种方式时的好处是：程序员可以在同一个文件中，将类(class / object)、trait 创建在不同的包中，这样就非常灵活了

   ~~~ java
   
   package qq.rr{
     package stu{
       class People{
         var name:String= _
         var age:Int= _
       }
     }
     class People{
       var name:String= _
       var age:Int= _
     }
     object fun{
       def main(args: Array[String]): Unit = {
       println("**********")
       }
     }
   }
   ~~~

   ![](../img/scala/包.png)

   - 从图中我们可以看到`qq.rr`包下面还有一个包`qq.rr.stu`包，两个包中相同的类名`people`并不重复，应为他们所属的包不同。

7. 作用域原则：可以直接向上访问。即: Scala中子包中直接访问父包中的内容, 大括号体现作用域。(提示：Java中子包使用父包的类，需要import)。在子包和父包 类重名时，默认采用就近原则，如果希望指定使用某个类，则带上包名即可。

   ~~~ java
   package com.atguigu{
     //这个类就是在com.atguigu包下
     class User{
     }
     //这个类对象就是在Monster$ , 也在com.atguigu包下
     object Monster {
     }
     class Dog {
     }
     package scala {
       //这个类就是在com.atguigu.scala包下
       class User{
       }
       //这个Test 类对象
       object Test {
         def main(args: Array[String]): Unit = {
             //子包可以直接访问父包的内容
             var dog = new Dog()
             println("dog=" + dog)
             //在子包和父包 类重名时，默认采用就近原则.
             var u = new User()
             println("u=" + u)
             //在子包和父包 类重名时，如果希望指定使用某个类，则带上包路径
             var u2 = new com.atguigu.User()
             println("u2=" + u2)
         }
       }
   ~~~

8. 父包要访问子包的内容时，需要import对应的类等.

   ~~~ java
   package com.atguigu{
     //引入在com.atguigu 包中希望使用到子包的类Tiger,因此需要引入.
     import com.atguigu.scala.Tiger
     //这个类就是在com.atguigu包下
     class User{
     }
     package scala {
       //Tiger 在 com.atguigu.scala 包中
       class Tiger {}
     }
     object Test2 {
       def main(args: Array[String]): Unit = {
           //如果要在父包使用到子包的类，需要import
           val tiger = new Tiger()
           println("tiger=" + tiger)
       }}}
   ~~~

9. 可以在同一个`.scala`文件中，声明多个并列的package(建议嵌套的`pakage`不要超过3层) 。

10. 包名可以相对路径也可以绝对路径，比如，访问BeanProperty的绝对路径是：_root_.scala.beans.BeanProperty ，在一般情况下：我们使用相对路径来引入包，只有当包名冲突时，使用绝对路径来处理。

    ~~~ java
    package com.atguigu.scala2
    class Manager( var name : String ) {
      //第一种形式,相对路径
      //@BeanProperty var age: Int = _
      //第二种形式, 和第一种一样，都是相对路径引入
      //@scala.beans.BeanProperty var age: Int = _
      //第三种形式, 是绝对路径引入，可以解决包名冲突
      @_root_. scala.beans.BeanProperty var age: Int = _
    }
    object TestBean {
      def main(args: Array[String]): Unit = {
        val m = new Manager("jack")
        println("m=" + m)
      }
     }
    ~~~

11. 包对象

    基本介绍：包可以包含类、对象和特质trait，但不能包含函数/方法或变量的定义。这是Java虚拟机的局限。为了弥补这一点不足，scala提供了包对象的概念来解决这个问题。

    ~~~ java
    package qq.rr{
      //在包中不可以直接声明方法和对象，为了解决这个问题，需要引入包对象
      //package object scala标示一个scala包对象，它是包qq.rr下面的一个包对象
      //每一个包都可以有一个包对象，包对象的名字和子包的名字一样
      //在包对象中可以定义变量和方法
      //在包中定义的变量和方法可以在对应的包中使用
      package object stu{
        //定义变量
        var name:String="aaaa"
        def print():Unit={
          printf("在包对象中定义的方法")
        }
      }
      package stu{
        class People{
          var name:String= _
          var age:Int= _
        }
        //应为fun在stu对应的包下面，所以可以使用包对象中的变量和方法
        object fun{
          def main(args: Array[String]): Unit = {
            println("**********")
            println("name="+name)
            println(print())
          }
        }
      }
      class People{
        var name:String= _
        var age:Int= _
      }
    
    }
    ~~~

    - 包对象的底层实现机制分析(重点)

      - 当创建包对象后，在该包下生成 public final class package 和 public final class package$

      - 通过 package$ 的一个静态实例完成对包对象中的属性和方法的调用。

        ![](../img/scala/调用机制.png)

        ![](../img/scala/包类.png)

        - 一个包对象在底层会生成两个类，`public final class package`和`public final class package$`,但是调用包对象的变量和方法用的是`public final class package$`类。

    - 定义的包对象一定在父包的下面，包对象和子包是平行关系。

    - 每个包都可以有一个包对象。你需要在父包中定义它。

    - 包对象名称需要和子包名一致，一般用来对包的功能补充

12. 包的可见性

    1. Java访问修饰符基本介绍

       - java提供四种访问控制修饰符号控制方法和变量的访问权限（范围）:

       - 公开级别:用public 修饰,对外公开

       - 受保护级别:用protected修饰,对子类和同一个包中的类公开

       - 默认级别:没有修饰符号,向同一个包的类公开.

       - 私有级别:用private修饰,只有类本身可以访问,不对外公开.

         ![](../img/scala/访问类型.png)

         注意：在子类中可以访问受保护的属性和方法，并不要求子类和本类在同一个包中。

         ​	修饰符可以用来修饰类中的属性，成员方法以及类
         ​	只有默认的和public才能修饰类！，并且遵循上述访问权限的特点。

    2. Scala中包的可见性介绍:

       - 在Java中，访问权限分为: public，private，protected和默认。在Scala中，你可以通过类似的修饰符达到同样的效果。但是使用上有区别。

         ~~~ java
         object Visit {
           def main(args: Array[String]): Unit = {
             val  c = new Clerk()
             c.showInfo()
             Clerk.test(c)
         }
           //当一个文件中同时出现class clerk类和object clerk类，
           //1 我们把class clerk叫做半生类，object clerk叫做伴生对象
           //应为scala的设计值去掉了static关键字，而引入了伴生的概念
           //将非静态内容放在伴生类中，静态内容放在伴生对象中
           //半生对象中可以访问私有属性，伴生类也可以访问，但是不可以在外部函数访问
           class Clerk {
             var name : String = "jack"//底层也是私有的，并且自动生成可读取可写的方法
             private var sal : Double = 9999.9//显示私有，只会生成可读的方法
             def showInfo(): Unit = {
               println(" name " + name + " sal= " + sal)
             }}
           object Clerk{
             def test(c : Clerk): Unit = {
               //这里体现出在伴生对象中，可以访问c.sal
               println("test() name=" + c.name + " sal= " + c.sal)
             }
           }
         
         ~~~

         - Scala中包的可见性和访问修饰符的使用

           - 当属性访问权限为默认时，从底层看属性是private的，但是因为提供了xxx_$eq()[类似setter]/xxx()[类似getter] 方法，因此从使用效果看是任何地方都可以访问)
           - 当方法访问权限为默认时，默认为public访问权限
           - private为私有权限，只在类的内部和伴生对象中可用 
           - **protected为受保护权限，scala中受保护权限比Java中更严格，只能子类访问，同包无法访问** ，这个是编译器在语法层面的控制，就像后面父类定义的受保护的方法，在底层虽然受保护的方法权限为`public`但是在同一个包中依旧无法访问，只可以在子类中访问，这个是编译器确定的。
           - 在scala中没有public关键字,即不能用public显式的修饰属性和方法。

         - 小结：scala设计者将访问的方式分成三大类(1) 处处可以访问public (2) 子类和伴生对象能访问protected (3) 本类和伴生对象访问 private

           - 包访问权限（表示属性有了限制。同时包也有了限制），这点和Java不一样，体现出Scala包使用的灵活性。

             ~~~java
             package qq.com.rzf.mypackage
             
             object Visit {
               def main(args: Array[String]): Unit = {
                 val c = new Clerk()
                 c.showInfo()
                 //Clerk.test(c)
                 val student = new Student
                 student.name//错误应为name是私有的
                 student.age//可以访问age属性
               }
               //类
             class Student {
               private val name:String=" "
               //在这里增加一个包的访问权限，1 属性仍然是private,
               // 2，扩大属性使用范围，age属性在mypackage包下面可以使用
               private [mypackage] val age:Int= _
               //在rzf包以及所有子包下面可以访问
               private [rzf] val salary:Int= _
               ///增加包访问权限后，1.private同时起作用。不仅同类可以使用 2. 同时rzf中包下其他类也可以使用
             }
             //说明：private也可以变化，比如protected[atguigu], 非常的灵活。
             ~~~

       - 访问权限的本质问题

         ~~~ java
         //源代码
         class monster{
           var name:String=""
           protected var age:Int= _
           private var salary:Double= _
         }
         //底层编译后的代码
         public class monster
         {
           //发现不同的访问权限在底层实现全部是private,不同的访问权限是通过方法体现出来的。
           private String name = "";
           private int age;
           private double salary;
         
           public String name()
           {
             return this.name; 
           } 
           public void name_$eq(String x$1) { this.name = x$1; } 
           public int age() { return this.age; } 
           public void age_$eq(int x$1) { this.age = x$1; } 
           private double salary() { return this.salary; } 
           private void salary_$eq(double x$1) { this.salary = x$1; }
         
         }
         ~~~

         - 为何这样设计，应为在马丁看来，所有的属性对外要不不可以访问，要不可以访问，只有这两种方式，所以在的层全部转换为私有的，对不同的访问权限是通过方法来进行的。即如果属性可以被别人访问，就提供共有的方法，如果不让别人访问，就提供私有的方法。访问修饰的最大影响就是对底层方法的影响。

13. Scala引入包基本介绍

    - Scala引入包也是使用import, 基本的原理和机制和Java一样，但是Scala中的import功能更加强大，也更灵活。
    - 因为Scala语言源自于Java，所以java.lang包中的类会自动引入到当前环境中，而Scala中的scala包和Predef包的类也会自动引入到当前环境中，即起其下面的类可以直接使用。
    - 如果想要把其他包中的类引入到当前环境中，需要使用import语言

    1. Scala引入包的细节和注意事项

       1. 在Scala中，import语句可以出现在任何地方，并不仅限于文件顶部，import语句的作用一直延伸到包含该语句的块末尾。这种语法的好处是：在需要时在引入包，缩小import 包的作用范围，提高效率。

          ~~~ java
          class User {
            //在user类中导入的包仅仅限于user类中使用，在其他的类中不可以使用
            import scala.beans.BeanProperty
            @BeanProperty var  name : String = ""
          }
          class Dog {
            //报错
            //@BeanProperty var  name : String = ""
          }
          ~~~

       2. Java中如果想要导入包中所有的类，可以通过通配符*，Scala中采用下划线_

          ~~~ java
          import scala.beans._//下划线标示将该包下所有内容导入
          ~~~

       3. 如果不想要某个包中全部的类，而是其中的几个类，可以采用选取器(大括号)

          ~~~ java
          import scala.collection.mutable.{HashMap, HashSet}//选择引入
          ~~~

       4. 如果引入的多个包中含有相同的类，那么可以将类进行重命名进行区分，这个就是重命名。

          ~~~ java
           def test():Unit={
              //为何重命名，应为java包下有些类和scala包下有些类名字相同
              //将java.util包下面的HashMap类重命名为javaHashMap
              import java.util.{HashMap=>javaHashMap,List}
              val kToV = new javaHashMap[Int]()
            }
          ~~~

       5. 如果某个冲突的类根本就不会用到，那么这个类可以直接隐藏掉。

          ~~~ java
          import java.util.{ HashMap=>_, _} 
          var map = new HashMap()
          //第一个_表示隐藏，就是将HashMap类隐藏起来，第二个_标示将java.util包中除了HashMap类其他所有的类全部引入
           //现在引入的HashMap是scala包下面的HashMap类
           val aToB = new mutable.HashMap[]()
          ~~~

### 第四章，面向对象部分

### 4.1，面向对象编程三大特征

- 面向对象编程有三大特征：封装、继承和多态。

- 快速入门

  ~~~ java
  object BankDemo {
    def main(args: Array[String]): Unit = {
      var bank=new Bank("11111",1000,"111")
      bank.query("111")
    }
  
  }
  //编写account类，还有主构造方法
  class  Bank(inAccount:String,inBalance:Double,inPwd:String){
    val accountNum=inAccount
      var balance=inBalance
    var pwd=inPwd
    //查询
    def query(pwd:String):Unit={
    if(!pwd.equals(this.pwd)){
      println("密码错误")
      return
    }
      printf("当前账号:%s 当前余额:%.2f",this.accountNum,this.balance)
    }
    //取款
    def withDraw(pwd:String,money:Double):Any={
      //密码错误或者余额不足返回unit,但是函数要求返回double,所以这里使用返回类型推断
      //如果把:Any=去掉，那么return语句就不起作用，但是这里需要return语句结束程序
      if(!pwd.equals(this.pwd)){
        println("密码错误")
        return
      }else if(money>this.balance) {
        println("余额不足")
        return
      }else{
         this.balance -=money
        this.balance
      }
    }
  }
  ~~~

#### 4.1.1，封装

1. 封装介绍

   封装(encapsulation)就是把抽象出的数据/属性和对数据的操作/方法封装在一起,数据被保护在内部,程序的其它部分只有通过被授权的操作(成员方法),才能对数据进行操作。

   - 封装的理解和好处

     - 隐藏实现细节
     - 提可以对数据进行验证，保证安全合理

   - 封装的实现步骤 

     ~~~ java
     将属性进行私有化
     提供一个公共的set方法，用于对属性判断并赋值
     def  setXxx(参数名 : 类型) : Unit = {	
     //加入数据验证的业务逻辑	属性 = 参数名   
     }
     
     提供一个公共的get方法，用于获取属性的值//def getXxx() [: 返回类型] = {    
     //权限，需要自己.
     	return 属性
     }
     ~~~

   - 封装细节

     - Scala中为了简化代码的开发，当声明属性时，本身就自动提供了对应setter/getter方法，如果属性声明为private的，那么自动生成的setter/getter方法也是private的，如果属性省略访问权限修饰符，那么自动生成的setter/getter方法是public的
     - 因此我们如果只是对一个属性进行简单的set和get ，只要声明一下该属性(属性使用默认访问修饰符) 不用写专门的getset，默认会创建，访问时，直接对象.变量。这样也是为了保持访问一致性 
     - 从形式上看 dog.food 直接访问属性，其实底层仍然是访问的方法,  看一下反编译的代码就明白
     - 有了上面的特性，目前很多新的框架，在进行反射时，也支持对属性的直接反射

#### 4.1.2，继承

1. Java继承的简单回顾

   ~~~ java
   class 子类名 extends 父类名 { 类体 }
   子类继承父类的属性和方法
   ~~~

2. 继承可以解决代码复用,让我们的编程更加靠近人类思维.当多个类存在相同的属性(变量)和方法时,可以从这些类中抽象出父类(比如Student),在父类中定义这些相同的属性和方法，所有的子类不需要重新定义这些属性和方法，只需要通过extends语句来声明继承父类即可。

   ![](../img/scala/继承.png)

3. 和Java一样，Scala也支持类的单继承

   1. Scala继承的基本语法

      ~~~ java
      class 子类名 extends 父类名  { 类体 }
      ~~~

   2. Scala继承给编程带来的便利

      1. 代码的复用性提高了
      2. 代码的扩展性和维护性提高了
         当我们修改父类的方法和属性的时候，子类也相应的继承了父类应有的属性和方法，可以在每一级的父类中添加方法和属性

   3. scala子类继承了什么,怎么继承了?

      - 子类继承了所有的属性，只是私有的属性不能直接访问，需要通过从父类继承的公共的方法去访问。

        ~~~ java
        //实例
        package qq.com.rzf.oop
        
        object ExtendsMy {
          def main(args: Array[String]): Unit = {
            val sub = new Sub()
            sub.sayOk()//子类可以访问父类的public方法
            //protected为受保护权限，scala中受保护权限比Java中更严格，只能子类访问，同包无法访问
            //比java更加严格
           // sub.test200()报错，子类无法访问父类的受保护的方法
          }
        }
        //父类
        class Base {
          var n1: Int = 1 //底层是私有属性
          protected var n2: Int = 2//受保护的属性
          private var n3: Int = 3//显示私有属性
          def test100(): Unit = {//共有方法
            println("base 100")
          }
          protected def test200(): Unit = {//受保护的方法
            println("base 200")
          }
          private def test300(): Unit = {//私有方法
            println("base 300")
          }
        }
        //子类继承
        class Sub extends Base {
          def sayOk(): Unit = {
            this.n1 = 20//这里的访问在底层其实还是调用的方法
            this.n2 = 40
            test200()//父类受保护的方法可以访问
            //test300()父类的私有方法子类可以继承，但是不可以访问
            //this.n3//在这里不可以使用n3,应为在父类方法中n3是私有的子类可以继承，但是不可以访问
            println("范围" + this.n1 + this.n2)
          }
          }
        //底层源码
        public class Base
        {
         //所有属性在底层全部是私有
          private int n1 = 1;
          private int n2 = 2;
          private int n3 = 3;
        
          public int n1()
          {
            return this.n1; } 
          public void n1_$eq(int x$1) { this.n1 = x$1; } 
            //受保护的属性底层自动生成公共方法，只是scala在设计的时候不允许子类使用父类受保护额方法
          public int n2() { return this.n2; } 
          public void n2_$eq(int x$1) { this.n2 = x$1; } 
          //私有属性自动生成私有方法
          private int n3() { return this.n3; } 
          private void n3_$eq(int x$1) { this.n3 = x$1; } 
          public void test100() {
            Predef..MODULE$.println("base 100");
          }
          public void test200() {
            Predef..MODULE$.println("base 200");
          }
          private void test300() {
            Predef..MODULE$.println("base 300");
          }
        }
        //子类
        public class Sub extends Base
        {
          public void sayOk()
          {
            n1_$eq(20);
            n2_$eq(40);
            test200();
        
            Predef..MODULE$.println(new StringBuilder().append("范围").append(BoxesRunTime.boxToInteger(n1())).append(BoxesRunTime.boxToInteger(n2())).toString());
          }
        }
        ~~~

        ![](../img/scala/私有属性.png)

        - 通过调试，我们发现子类继承了父类的私有属性，只是不可以访问。受保护的属性底层自动生成公共方法，但是在同一个包中别的方法仍旧不可以使用，但是`java`可以，这是因为scala在设计的时候不允许同一个包下面的其他类使用，但是子类可以使用。这是编译器在语法层面做的。虽然编译器编译完成后受保护的方法是公共的，但是在没有编译前，编译器禁止这种语法的存在。

        - 小结

          **如果属性和方法没有修饰符修饰，那么属性和方法可以再任何地方使用，如果属性和方法添加受保护的修饰，那么只可以在子类中使用，如果添加为私有的修饰符，只有在本类或者伴生对象可以使用。**

   4. 重写方法(方法覆盖)

      - 说明: scala明确规定，重写一个非抽象方法需要用override关键字修饰，调用超类的方法使用super关键字 

        ~~~ java
        class Person {
          var name : String = "tom"
          def printName() {
            println("Person printName() " + name)
          }
        }
        class Person {
          var name : String = "tom"
          def printName() {
            println("Person printName() " + name)
          }
        }
        ~~~

   5. Scala中类型检查和转换

      - **要测试某个对象是否属于某个给定的类，可以用isInstanceOf方法。用asInstanceOf方法将引用转换为子类的引用。classOf获取对象的类名。**

        ~~~ java
        //使用classOf函数可以获取类名
            println(classOf[String])
        //类型检查
        package qq.com.rzf.oop
        
        object GetClass {
          def main(args: Array[String]): Unit = {
            //使用classOf函数可以获取类名
            println(classOf[String])
            //向下转型
            var dog=new Dog
            //anm1是animal类型
            var anm1:animals=new Dog
            var anm=new animals
            //现在把子类的引用给父类,也就是现在anm指向子类对象
            anm=dog
            //如何把把父类引用给子类引用，即向上转型
            var dog1=dog.asInstanceOf[animals]
            //小疑问？这个算是转型么？但是这一anm1本身就是anm1类型
            anm1=dog.asInstanceOf[animals]
            //现在dog1是animals类型，可以调用子类的方法
            dog1.print()//调用子类方重载法
            //dog1.sayHello()报错，父类不可以调用子类独有的方法
          }
        }
        class animals{
           var name:String="tom"
          def print():Unit={
            println("父类名字是:"+this.name)
          }
        }
        class Dog extends animals {
          override def print():Unit={
            println("子类，名字是:"+this.name)
          }
          def sayHello():Unit={
            println("say hello")
          }
        }
        ~~~

        - classOf[String]就如同Java的 String.class 。
        - obj.isInstanceOf[T]就如同Java的obj instanceof T 判断obj是不是T类型。
        - obj.asInstanceOf[T]就如同Java的(T)obj 将obj强转成T类型。

      - 类型转换实例

        类型检查和转换的最大价值在于：可以判断传入对象的类型，然后转成对应的子类对象，进行相关操作，这里也体现出多态的特点。

        ~~~ java
        package qq.com.rzf.oop
        
        object yinyongDemo {
          def main(args: Array[String]): Unit = {
            var stu01:student01=new student01
            var emp01:employee01=new employee01
            print(stu01)
            println()
            print(emp01)
          }
          def printInfo(peo:people01):Unit={
            //如果对传进来的参数不进行判断，那么现在peo只可以调用people01的方法
            //现在我们可以强制转型，向下转型，对参数进行判断，调用相应子类的方法
            //这里体现的是多肽性
            if(peo.isInstanceOf[student01]){
              //peo.asInstanceOf[student01],这里对peo引用没有任何影响，只是返回了一个新的类型
              //如果传进来的是student01类型，那么调用student01方法
              peo.sayHello()
              peo.print()
            }else if(peo.isInstanceOf[employee01]){
              peo.sayHello()
              peo.sayHello()
            }
          }
        
        }
        class people01{
          def sayHello():Unit={
            println("我是people01类中的sayHello（）方法")
          }
          def print():Unit={
            println("打印姓名")
          }
        }
        class student01 extends  people01 {
          var name:String="tom"
          def sayHello01():Unit={
            println("我是student01类中的sayHello（）方法")
          }
          override def print():Unit={
            println("学生姓名是:"+this.name)
          }
        }
        
        class employee01 extends  people01 {
          var name:String="simth"
          def sayHello01():Unit={
            println("我是employee01类中的sayHello（）方法")
          }
          override def print():Unit={
            println("雇员姓名是:"+this.name)
          }
        }
        ~~~

        ![](../img/scala/转型.png)

   6. 回顾-Java中超类的构造

      ~~~ java
      package qq.com.rzf.oop;
      
      public class BaseDemo {
          public static void main(String[] args) {
              //在这里创建对象，首先会调用父类构造器，然后会调用子类构造器
              B b = new B();
              //在这里创建对象，首先会调用父类构造器，然后会调用子类构造器,与scala对比
              B b1 =new B("tom");
          }
      }
      class A{
          public A(){
              System.out.println("A的无参数构造器");
          }
          public  A(String name){
              System.out.println("姓名是" + name);
          }
      }
      class B extends A{
          public B(){
              //父类的构造器在这里会默认调用
              super();
              System.out.println("B的无参数构造器");
          }
          public B(String name){
              super(name);
              System.out.println("姓名是:" + name);
          }
      }
      //结果
      A的无参数构造器
      B的无参数构造器
      姓名是tom
      姓名是:tom
      ~~~

   7. Scala超类的构造说明

      - 类有一个主构器和任意数量的辅助构造器，而每个辅助构造器都必须先调用主构造器(也可以是间接调用)

      - **只有主构造器可以调用父类的构造器。辅助构造器不能直接调用父类的构造器。在Scala的构造器中，你不能调用super(params)**

        ~~~ java
        //主构造器演示
        package qq.com.rzf.oop
        
        object MainCon {
          def main(args: Array[String]): Unit = {
            val stu = new stu("111",100,"xiaorui",24)
            stu.showInfo()
          }
        }
        class per(inName:String,inAge:Int){
          println("我是per类的主构造器")
          var name:String=inName
          var age:Int=inAge
        
          def showInfo():Unit={
            println("姓名是:"+this.name+"\t"+"年龄是:"+this.age)
          }
        }
        //在这里通过子类的主构造器吧inName:String,inAge:Int参数传递给父类的主构造器
        //从子类到父类传递参数的出口只有一个：
        //class stu(inStuNo:String,inGrade:Double,inName:String,inAge:Int) extends per(inName:String,inAge:Int)
        //没有super(参数)这种语法，不管是在子类的柱构造器还是辅助构造器中，都不可以这样用
        class stu(inStuNo:String,inGrade:Double,inName:String,inAge:Int) extends per(inName:String,inAge:Int){
          //super(inName,inAge)这种写法错误
          //  this.name=inName//在主类中要调用父类构造器
          // this.age=inAge
          println("我是stu类的主构造器")
          var stuNo:String=inStuNo
          var grade:Double=inGrade
          //重写方法,需要添加关键字
          override def showInfo():Unit={
            println("学号是:"+this.stuNo+"\t"+"姓名是:"+this.name+"\t"+"成绩是:"+this.grade)
          }
        }
        //构造器调用顺序，先调用父类主构造器，在调用子类主构造器
        我是per类的主构造器
        我是stu类的主构造器
        学号是:111	姓名是:xiaorui	成绩是:100.0
        class Person(name: String) {
        
        }
        //将子类参数传递给父类构造器
        class Emp (name: String) extends Person(name) {
         
           // super(name) //没有这种写法
        
           def  this() {
                //super("abc") 不可以在辅助类构造器中调用父类构造器
           }
        }
        
        ~~~


#### 4.1.3，覆写字段

   1. 基本介绍

      在Scala中，子类改写父类的字段，我们称为覆写/重写字段。覆写字段需使用 override修饰。

   2. 在Java中只有方法的重写，没有属性/字段的重写，准确的讲，是隐藏字段代替了重写。

      ~~~ java
      //实例
      package qq.com.rzf.oop;
      
      public class FieldsDemo {
          public static void main(String[] args) {
              /*
              总结：
              对于父类和子类有相同的字段的时候，用子类的引用去获取值，获取的是子类字段的值
              用父类的引用去获取字段的值，获取的是父类的字段的值
               */
         		 Sub01 s=new Sub01();
              System.out.println(s.str);//输出sub
              //1 使用父类的引用来隐藏子类的同名属性
              Super01 sub01 = new Sub01();
              System.out.println(((Super01)s).str);//输出super
              //2 使用强制类型转换隐藏子类的同名属性
              System.out.println(sub01.str);//输出super
          }
      }
      class Super01{
           String str="super";
      }
      class Sub01 extends Super01{
           String str="sub";
      }
      
      ~~~

      - 我们创建了两个Sub对象，但是为什么第二个对象打印出来的结果是"Super"呢？
        - 隐藏字段代替了重写。我们可以看出成员变量不能像方法一样被重写。当一个子类定义了一个跟父类相同 名字的字段，子类就是定义了一个新的字段。这个字段在父类中被隐藏的，是不可重写的。
      - 如何访问隐藏字段？
        - 采用父类的引用类型，这样隐藏的字段就能被访问了，像上面所给出的例子一样。 
        -  将子类强制类型转化为父类类型，也能访问到隐藏的字段。
      - 小结：
        - 这个主要涉及到java里面一个字段隐藏的概念，父类和子类定义了一个同名的字段，不会报错。**但对于同一个对象，用父类的引用去取值****(字段)****，会取到父类的字段的值**，**用子类的引用去取值****(字段)****，****则取到子类字段的值**。**在实际的开发中，****要尽量避免子类和父类使用相同的字段名，否则很容易引入一些不容易发现的bug**。

   3. 回顾-Java另一重要特性: 动态绑定机制

      ~~~ java
      public class Dynamic {
          public static void main(String[] args) {
              AA a = new BB();
              //当不把子类中的这两个方法注销掉，结果是40,30
              //注销子类sum()方法：结果是：30,30
              //注销子类sum1()方法：结果是：30,20(getI()方法返回的是父类的i，应为调用的直接是属性)
              System.out.println(a.sum());
              System.out.println(a.sum1());
              /*
              java动态绑定机制小结：
              如果调用的是方法，那么jvm会将该方法和该对象的内存地址绑定
              如果调用一个属性，那么没有动态绑定机制，在哪里调用，就返回在哪里的值
               */
          }
      }
      class AA{
          public int i = 10;
          public int sum() {
              //getI()在这里调用的是方法，会议对象的内存地址动态绑定，所以返回20
              return getI() + 10;
          }
          public int sum1() {
              //这里直接调用属性，所以使用的是10
              return i + 10;
          }
          public int getI() {
              return i;
          }
      
      }
      class BB extends AA {
          public int i = 20;
          @Override
      //    public int sum() {
      //        return i + 20;
      //    }
      
          public int getI() {
              return i;
          }
      //    @Override
      //    public int sum1() {
      //        return i + 10;
      //    }
      }
      ~~~

   4. `Scala`覆写字段

      - Scala覆写字段快速入门

        ~~~ java
        //案例
        package qq.com.rzf.oop
        
        object ScalaOverride {
          def main(args: Array[String]): Unit = {
            val obj1 : AAA = new BBB()
            val obj2 : BBB = new BBB()
            //obj1.age=>obj1.age()，动态绑定机制
            //不管是哪一个类的引用，都指向对象的地址
            //在底层每一个类都有相应的方法
            println(obj1.age)
            println(obj2.age)
        
          }
        }
        class AAA {
          val age : Int = 10
        }
        class BBB extends AAA {
          override val age : Int = 20
        }
        ~~~

        - 覆写字段的注意事项和细节

          - def只能重写另一个def(即：方法只能重写另一个方法)

          - val只能重写父类的一个val属性 或父类的不带参数的def方法 （如果方法带参数就变成了了方法的重写）

            ~~~ java
            //代码正确吗?
            class AAAA {
              var name: String = ""
            }
            class BBBB extends AAAA {
              override  val name: String = "jj"//报错
            }
            //应为var属性在底层会生成共有的get和set方法，但是val会生成只读的方法
            //，重写方法，以属性的方式
            class A {
               def sal(): Int = { 
                  return 10
              }
            }
            
            class B extends A {
               //重写方法不可以有参数
             override val sal : Int = 0
            }
            val a=new B()
            a.sal//返回0，应为有动态绑定机制，不管哪一个类的引用都会指向对象的地址
            ~~~

          - var只能重写另一个抽象的var属性

            抽象属性：声明未初始化的变量就是抽象的属性,抽象属性在抽象类

            ~~~ java
            package qq.com.rzf.oop
            
            object AbstractDEMO {
            
            }
            abstract class abstractClass{
              //scala的熟悉必须进行初始化，如果不进行初始化，那就是默认的抽象属性，抽象属性不进行初始化
              //但是此时类也要声明为初始化
              //对于抽象的属性，在底层不会有属性的声明，而是生成两个抽象的方法
              var name:String
              //属性已经初始化，不是抽象属性，所以在子类中不能够进行继承
              var age:Int= _
            }
            //接下来我们进行重写var属性
            class sub_Abstractclass extends abstractClass {
              //在子类中要对继承的var进行初始化操作
              //1，如果我们在子类中重写父类的抽象方法，本质上是实现了抽象方法
              //2 重写属性可以使用override关键字，也可以不使用
              override var name: String = _
            }
            //抽象属性底层源码实现
            public abstract class abstractClass
            {
              private int age;
            //抽象属性在底层并没有声明，只是生成了两个公共方法
              public abstract String name();
            
              public abstract void name_$eq(String paramString);
            
              public int age()
              {
                return this.age; } 
              public void age_$eq(int x$1) { this.age = x$1; }
            
            }
            ~~~

          - var重写抽象的var属性小结

            - 一个属性没有初始化，那么这个属性就是抽象属性
            - 抽象属性在编译成字节码文件时，属性并不会声明，但是会自动生成抽象方法，所以类必须声明为抽象类
            - 如果是覆写一个父类的抽象属性，那么override 关键字可省略(本质上是实现)
            - `scala`延续了`java`的动态绑定机制。

   #### 4.1.4，抽象类

1. 在Scala中，通过abstract关键字标记不能被实例化的类。方法不用标记abstract，只要省掉方法体即可。抽象类可以拥有抽象字段，抽象字段/属性就是没有初始值的字段

2. 在抽象类中可以由抽象属性，也可以由普通属性,也可以由普通方法（实现方法）。

3. 抽象类的价值更多是在于设计，是设计者设计好后，让子类继抽象类抽象类本质上就是一个模板设计

   ~~~ java
   abstract class Animal {
     var name : String //抽象的属性
     var age : Int // 抽象的属性
     var color : String = "black"
     def cry() // 抽象方法
   }
   ~~~

4. Scala抽象类使用的注意事项和细节讨论

   1. 抽象类不能被实例化，但是在实例化的时候实现其抽象方法是可以的，

      ~~~ java
      object AbstractDEMO {
        def main(args: Array[String]): Unit = {
          //实例化抽象类对象，并动态实现其所有抽象方法
         var tmp= new abstractClass {
            override var name: String = _
      
            override def print(): Unit = ???//抽象方法的实现
          }
        }
      }
      abstract class abstractClass{
        //scala的熟悉必须进行初始化，如果不进行初始化，那就是默认的抽象属性，抽象属性不进行初始化
        //但是此时类也要声明为初始化
        //对于抽象的属性，在底层不会有属性的声明，而是生成两个抽象的方法
        var name:String
        //属性已经初始化，不是抽象属性，所以在子类中不能够进行继承
        var age:Int= _
        def print()//抽象方法
      }
      ~~~

   2. 抽象类不一定要包含abstract方法。也就是说,抽象类可以没有abstract方法

   3. 一旦类包含了抽象方法或者抽象属性,则这个类必须声明为abstract

   4. 抽象方法不能有主体，不允许使用abstract修饰。

   5. 如果一个类继承了抽象类，则它必须实现抽象类的所有抽象方法和抽象属性，除非它自己也声明为abstract类。

   6. 抽象方法和抽象属性不能使用private、final 来修饰，因为这些关键字都是和重写/实现相违背的。
      抽象类中可以有实现的方法.

   7. 子类重写抽象方法不需要override，写上也不会错(另一种叫法是实现).

#### 4.1.5，匿名子类

1. 基本介绍

   和Java一样，可以通过包含带有定义或重写的代码块的方式创建一个匿名的子类.

2. `java`中匿名子类

   ~~~ java
   package qq.com.rzf.oop;
   
   public class AnnomyClass {
       public static void main(String[] args) {
           MyAnomyClass myAnomyClass=new MyAnomyClass() {
               //代码块里面的内容看做匿名子类
               @Override
               public void print() {
                   System.out.println("我是匿名子类");
               }
   
               @Override
               public void sayHello() {
                   System.out.println("say Hello");
               }
           };
           myAnomyClass.sayHello();
       }
   }
   abstract class MyAnomyClass{
       abstract public void print();//抽象方法
       abstract public void sayHello();
   }
   ~~~

3. `scala`匿名子类的语法和`java`语法一样。

   ~~~ java
   package qq.com.rzf.oop
   
   object ScalaAnnoy {
     def main(args: Array[String]): Unit = {
       var monster:Monster=new Monster {
       //override关键字可以拿掉
         override def cry(): Unit = {
           println("我是Monster抽象类实现方法")
         }
       }
       monster.cry()
     }
   }
   abstract class Monster{
     def cry()//抽象方法
   }
   
   ~~~

   ![](../img/scala/类层次图.png)

   - subtype : 子类型(继承关系)
   - implicit conversion: 隐式转换
   - class hierarchy : 类的层级关系
   - 继承层级图小结
     - 在scala中，所有其他类都是AnyRef的子类，类似Java的Object。
     - AnyVal和AnyRef都扩展自Any类。Any类是根节点/根类型
     - Any中定义了isInstanceOf、asInstanceOf方法，以及哈希方法等。
     - Null类型的唯一实例就是null对象。可以将null赋值给任何引用，但不能赋值给值类型的变量[案例演示]。
     - Nothing类型没有实例。它对于泛型结构是有用处的，举例：空列表Nil的类型是List[Nothing]，它是List[T]的子类型，T可以是任何类。
#### 4.1.6，静态属性和静态方法

1. Java的静态概念

   ~~~ java
   public static 返回值类型  方法名(参数列表) {方法体}
         静态属性...
   ~~~

   -  说明: Java中静态方法并不是通过对象调用的，而是通过类对象调用的，所以静态操作并不是面向对象的。

2. Scala中静态的概念-伴生对象Scala语言是完全面向对象(万物皆对象)的语言，所以并没有静态的操作(即在Scala中没有静态的概念)。也没有static关键字, 但是为了能够和Java语言交互(因为Java中有静态概念)，就产生了一种特殊的对象来模拟类对象，我们称之为类的伴生对象。这个类的所有静态内容都可以放置在它的伴生对象中声明和调用

3. 快速入门

   ![](../img/scala/伴生类.png)

4. 伴生对象的小结

   1. Scala中伴生对象采用object关键字声明，伴生对象中声明的全是 "静态"内容，可以通过伴生对象名称直接调用。

   2. 伴生对象对应的类称之为伴生类，伴生对象的名称应该和伴生类名一致。

   3. 伴生对象中的属性和方法都可以通过伴生对象名直接调用访问

   4. 从语法角度来讲，所谓的伴生对象其实就是类的静态方法和静态变量的集合

   5. 从技术角度来讲，scala还是没有生成静态的内容，只不过是将伴生对象生成了一个新的类，实现属性和方法的调用。

      ~~~ java
      //源码
      package qq.com.rzf.oop
      
      object Employee {
        def main(args: Array[String]): Unit = {
          var e:emp=new emp("1111","xiaorui","工程师")
          e.print()
          emp.department//直接用半生对象调用静态属性
        }
      }
      //伴生类
      class emp(inEmp:String,Iname:String,Incareer:String){
        var empNum:String =inEmp
        var name : String =Iname
        var career:String=Incareer
        def print():Unit={
          //使用静态属性
          println("工号:"+empNum+"\t"+"姓名:"+name+"\t"+"职业:"+career+"部门"+"\t"+emp.department)
        }
      }
      //伴生对象
      object emp{
        var department:String="开发部"
      }
      //反编译后的半生对象
      package qq.com.rzf.oop;
      
      public final class emp$
      {
        public static final  MODULE$;
        private String department;
      
        static
        {
          new ();
        }
      //对静态属性的访问底层还是通过方法
        public String department()
        {
          return this.department; } 
        public void department_$eq(String x$1) { this.department = x$1; } 
        private emp$() { MODULE$ = this;
      
          this.department = "开发部";
        }
      }
      ~~~

   6. 从底层原理看，伴生对象实现静态特性是依赖于 public static final MODULE$ 实现的。

   7. 伴生对象的声明应该和伴生类的声明在同一个源码文件中(如果不在同一个文件中会运行错误!)，但是如果没有伴生类，也就没有所谓的伴生对象了，所以放在哪里就无所谓了。

   8. 如果 class A 独立存在，那么A就是一个类， 如果 object A 独立存在，那么A就是一个"静态"性质的对象[即类对象], 在 object A中声明的属性和方法可以通过 A.属性 和 A.方法 来实现调用

   9. 当一个文件中，存在伴生类和伴生对象时，文件的图标会发生变化

   10. 伴生对象-apply方法

       1. 在伴生对象中定义apply方法，可以实现： 类名(参数) 方式来创建对象实例. 

          ![](../img/scala/apply.png)

####   4.1.7，接口

1. Java接口

   ~~~java
   声明接口
   interface 接口名
   实现接口
   class 类名 implements 接口名1，接口2
   ~~~

2. 特点

   1. 在Java中, 一个类可以实现多个接口。
   2. 在Java中，接口之间支持多继承
   3. 接口中属性都是常量
   4. 接口中的方法都是抽象的 

3. Scala接口的介绍

   - 从面向对象来看，接口并不属于面向对象的范畴，Scala是纯面向对象的语言，在Scala中，没有接口, 也没有implements关键字。
   - Scala语言中，采用trait（特质，特征）来代替接口的概念，也就是说，多个类具有相同的特征（特征）时，就可以将这个特质（特征）独立出来，采用关键字trait声明。

4. trait 的声明

   ~~~ java
   trait 特质名 {
   	trait体
   }
   ~~~

   - trait 命名 一般首字母大写.

   - Cloneable , Serializable

   - object T1 extends Serializable {}
   - Serializable： 就是scala的一个特质。
   - 在scala中，java中的接口可以当做特质使用

5. Scala中trait 的使用

   - 一个类具有某种特质（特征），就意味着这个类满足了这个特质（特征）的所有要素，所以在使用时，也采用了extends关键字，如果有多个特质或存在父类，那么需要采用with关键字连接

     ~~~ java
     没有父类class  类名   extends   特质1   with    特质2   with   特质3 ..
     
     有父类class  类名   extends   父类   with  特质1   with   特质2   with 特质3
     ~~~

6. 特质的快速入门案例

   - 可以把特质可以看作是对继承的一种补充
     Scala的继承是单继承,也就是一个类最多只能有一个父类,这种单继承的机制可保证类
     的纯洁性,比c++中的多继承机制简洁。但对子类功能的扩展有一定影响.所以
     我们认为: Scala引入trait特征 第一可以替代Java的接口,  第二个也是对单继承机制
     的一种补充 

     ![](../img/scala/特质.png)

     ~~~ java
     package qq.com.rzf.oop
     
     object MyTrait {
       def main(args: Array[String]): Unit = {
         var t1=new C1
         t1.getConnect("root","root")
       }
     }
     //特质的定义
     trait Trait1 {
       def getConnect(user: String, pwd: String): Unit //声明方法，抽象的.
       def test(n1:Int)
     }
     class A1 {}
     class B1 extends A1 {}
     class C1 extends A1 with Trait1 {
       override def getConnect(user: String, pwd: String): Unit = {
         println("c连接mysql")
       }
     
       override def test(n1: Int): Unit = ???
     }
     class D1 {}
     class E1 extends D1 with Trait1 {
       def getConnect(user: String, pwd: String): Unit = {
         println("e连接oracle")
       }
     
       override def test(n1: Int): Unit = ???
     }
     //反编译源码
     public abstract interface Trait1
     {
       public abstract void getConnect(String paramString1, String paramString2);
     
       public abstract void test(int paramInt);
     }//从底层看，我们发现在底层特质中的为实现的方法都是抽象方法
     //c1底层代码
     public class C1 extends A1//继承a1，实现trait1
     implements Trait1
     {
       public void getConnect(String user, String pwd)
       {
         Predef..MODULE$.println("c连接mysql");
       }
       public void test(int n1) {
         throw Predef..MODULE$.$qmark$qmark$qmark();
       }
     }
     //e1底层代码
     public class E1 extends D1
       implements Trait1
     {
       public void getConnect(String user, String pwd)
       {
         Predef..MODULE$.println("e连接oracle");
       }
       public void test(int n1) {
         throw Predef..MODULE$.$qmark$qmark$qmark();
       }
     }
     ~~~

   - 特质trait 的再说明

     - Scala提供了特质（trait），特质可以同时拥有抽象方法和具体方法，一个类可以实现/继承多个特质。
     - 特质中没有实现的方法就是抽象方法。类通过extends继承特质，通过with可以继承多个特质
     - 所有的java接口都可以当做Scala特质使用 

   - 带有具体实现的特质

     - 说明：和Java中的接口不太一样的是特质中的方法并不一定是抽象的，也可以有非抽象方法(即：实现了的方法)。

       ~~~ java
       package qq.com.rzf.xh
       
       object NonAbstrictTrait {
         def main(args: Array[String]): Unit = {
           var a1:animal=new cat
           a1.printName()
           a1.sayHello()
           println("%%%%%%%%%%%%%%%%%%%%%%")
           var a2:animal=new Dog
           a2.printName()
           a2.sayHello()
         }
       }
       trait animal{
         def sayHello():Unit={//实现的方法，可以重写
           println("动物会说话")
         }
         def printName():Unit//抽象方法
       }
       class cat extends animal{
         override def printName(): Unit = {
           println("我是小猫")
         }
       
         override def sayHello(): Unit = {
           println("我是小猫，我会说喵喵喵")
         }
       }
       class Dog extends animal{
         override def sayHello(): Unit = {
           println("我是小狗，我会说汪汪汪")
         }
       
         override def printName(): Unit = {
           println("我的小狗")
         }
       }
       //源码
       //声明的特质在底层为抽象类
       public abstract class animal$class
       {
         public static void sayHello(animal $this)
         {
           Predef..MODULE$.println("动物会说话");
         }
       
         public static void $init$(animal $this)
         {
         }
       }
       //dog
       public class Dog
         implements animal
       {
         public void sayHello()
         {
           Predef..MODULE$.println("我是小狗，我会说汪汪汪");
         }
       
         public void printName() {
           Predef..MODULE$.println("我的小狗");
         }
       
         public Dog()
         {
           animal.class.$init$(this);
         }
       }
       //cat 
       public class cat
         implements animal
       {
         public void printName()
         {
           Predef..MODULE$.println("我是小猫");
         }
       
         public void sayHello() {
           Predef..MODULE$.println("我是小猫，我会说喵喵喵");
         }
       
         public cat()
         {
           animal.class.$init$(this);
         }
       }
       ~~~

7. 带有特质的对象，动态混入

   1. 除了可以在类声明时继承特质以外，还可以在构建对象时混入特质，扩展对象的功能

   2. 此种方式也可以应用于对抽象类功能进行扩展

   3. 动态混入是Scala特有的方式（java没有动态混入），可在不修改类声明/定义的情况下，扩展类的功能，非常的灵活，耦合性低 。

   4. 动态混入可以在不影响原有的继承关系的基础上，给指定的类扩展功能。

      ~~~ java
      package qq.com.rzf.max
      
      object MyMax {
        def main(args: Array[String]): Unit = {
          //现在在不想改变OracleDB和MySQL3类定义的情况下改变其方法
          //1 用with就叫做动态混入，创建对象的时候，不想修改类定义，就用with混入特质
          //对普通类进行扩展
          val o1=new OracleDB with Operate3
          //现在调用特质方法
          o1.insert(100)
          //抽象类,抽象类不可以实例化，但是这里相当于实现了你名字类，只是抽象类为空
          val o2=new MySQL3  with Operate3
          o2.insert(200)
      
        }
      
      }
      trait Operate3 {
        def insert(id: Int): Unit = {
          println("插入数据 = " + id)
        }
      }
      //下面有两个类
      class OracleDB {
      }
      //抽象类中没有方法
      abstract class MySQL3 {
      }
      ~~~

   5. 如果抽象类中有 抽象的方法，如何动态混入特质?

      ~~~ java
      package qq.com.rzf.max
      
      object MyMax {
        def main(args: Array[String]): Unit = {
          //现在在不想改变OracleDB和MySQL3类定义的情况下改变其方法
          //1 用with就叫做动态混入，创建对象的时候，不想修改类定义，就用with混入特质
          //对普通类进行扩展
          val o1=new OracleDB with Operate3
          //现在调用特质方法
          o1.insert(100)
          //抽象类,抽象类不可以实例化，但是这里相当于实现了你名字类，只是抽象类为空
          val o2=new MySQL3  with Operate3
          o2.insert(200)
          //抽象类中有抽象方法
          val o3=new MySql with Operate3 {
            override def say(): Unit = {
              println("say hello")
            }
          }
      
        }
      
      }
      trait Operate3 {
        def insert(id: Int): Unit = {
          println("插入数据 = " + id)
        }
      }
      //下面有两个类
      class OracleDB {
      }
      abstract class MySQL3 {
      }
      abstract class MySql{
        def say()//抽象类中带有抽象方法
      }
      ~~~

8. 在Scala中创建对象共有几种方式？

   1. new 对象
   2. applay 方法，创建对象
   3. 动态混入(可以不改变类定义而改变类属性)
   4. 匿名子类创建对象
   5. 反射
   6. 反序列化
   7. 工具类(Unsafe.allocate)

9. 叠加特质

   1. 基本介绍

      1. 构建对象的同时如果混入多个特质，称之为叠加特质，
      2. 那么特质声明顺序从左到右，方法执行顺序从右到左。

      ~~~ java
      package qq.com.rzf.xh
      
      object AddTraits {
        def main(args: Array[String]): Unit = {
          //创建对象动态混入两个特质
          //1 当我们创建一个动态对象混入时，创建顺序是怎样的？
          /**
           * scala在叠加特质的时候，会首先从后面的特质开始执行（即从左到右）
           * 1，Operate4
           * 2，Data4
           * 3，DB4
           * 4，File4
           */
          val mysql=new MySQL4 with DB4 with File4
          println(mysql)
      
          /**
           * 当我们执行一个动态混入的对象方法时，其执行顺序如何？
           * 1 从右到左执行，当执行super()是，指的是左边的特质，也就是在栈中，File4父类是DB4
           * 2，如果左边没有特质，则super()指的是其父特质
           * 1，向文件
           * 2，向数据库
           * 3，插入数据
           */
          mysql.insert(100)
        }
      }
      trait Operate4 {//构建特质
        println("Operate4...")//底层生成一个构造器，此语句在构造器中
        def insert(id : Int)
      }
      trait Data4 extends Operate4 {
        println("Data4")
        override  def insert(id : Int): Unit = {//实现或者重写Operate4的insert方法
          println("插入数据 = " + id)
        }
      }
      trait DB4 extends Data4 {
        println("DB4")
        override def insert(id : Int): Unit = {//重写Data4的insert方法
          print("向数据库")
          super.insert(id)//调用insert方法，调用哪个呢？
          //左边没有特质，顾此处的super()指的是其父特质，即Date4
        }
      }
      trait File4 extends  Data4 {
        println("File4")
        override def insert(id : Int): Unit = {//重写
          print("向文件")
          super.insert(id)//调用父类insert方法，动态混入不一定是其父类,此处调用其左边的特质
        }}
      class MySQL4 {}
      
      ~~~

   2. 叠加特质注意事项和细节

      1. 特质声明顺序从左到右。

      2. Scala在执行叠加对象的方法时，会首先从后面的特质(从右向左)开始执行

      3. Scala中特质中如果调用super，并不是表示调用父特质的方法，而是向前面（左边）继续查找特质，如果找不到，才会去父特质查找

      4. 如果想要调用具体特质的方法，可以指定：super[特质].xxx(…).其中的泛型必须是该特质的直接超类类型

         ~~~ java
         //如果我们想调用Date4的insert方法，可以用泛型
             //super[？]问号必须是当前特质的直接父特质（或者超类）
             super[Data4].insert(id)
         ~~~

   3. 在特质中重写抽象方法特例

      - 针对调用父类的`super`抽象方法解决问题
        - 方式1 : 去掉 super()... 
        - 方式2:  调用父特质的抽象方法，那么在实际使用时，没有方法的具体实现，无法编译通过，为了避免这种情况的发生。可重写抽象方法，这样在使用时，**就必须考虑动态混入的顺序问题。**

      ~~~ java
      package qq.com.rzf.xh
      
      object TraitOverride {
        def main(args: Array[String]): Unit = {
          //这种顺序动态混编没有问题
          val o1=new Mysql02 with DB5 with File5
          o1.insert(100)
          //改变顺序
          val o2=new Mysql02 with File5 with DB5
          //为何报错，因为在这里file5调用父类的insert方法，其右边没有特质，只能调用父类特质，但是父类的抽象的
          o2.insert(200)
           //val o3=new Mysql02 with File5 无法通过编译，应为file5的父类insert方法是抽象的
          //其左边没有特质，调用的是其父类的方法
          val o3=new Mysql02 with DB5//正确，没有调用父类方法
        }
      
      }
      trait Operate5 {
        def insert(id : Int)
      }
      
      /**
       * 1 如果我们在子特质中实现或者重写富特质的抽象方法，同时又调用父类的抽象方法
       * 2 这个时候我们实现的方法不是完全实现的，因此需要声明为abstract override
       * 3，这个时候子类实现的方法在子类中依旧是抽象的
       */
      trait File5 extends Operate5 {
        abstract override def insert( id : Int ): Unit = {
          println("将数据保存到文件中..")
          super.insert(id)//在这里调用父类的抽象方法会报错，如果把子类实现的抽象方法前面添加abstract override
          //关键字即可
        }
      }
      trait DB5 extends Operate5{
        def insert(id : Int):Unit={//正常实现抽象方法
          println("将数据写入数据库中")
        }
      }
      class Mysql02{
      
      }
      ~~~

      - 练习

      ![](../img/scala/方法重写.png)

      - 理解 abstract override 的小技巧分享：可以这里理解，当我们给某个方法增加了abstract override 后，就是明确的告诉编译器，该方法确实是重写了父特质的抽象方法，但是重写后，该方法仍然是一个抽象方法（因为没有完全的实现，需要其它特质继续实现[通过混入顺序]）
      - 当作富接口使用的特质
        - 富接口：即该特质中既有抽象方法，又有非抽象方法

10. 特质中的具体字段

    特质中可以定义具体字段，如果初始化了就是具体字段，如果不初始化就是抽象字段。混入该特质的类就具有了该字段，字段不是继承，而是直接加入类，成为自己的字段。

    特质中未被初始化的字段在具体的子类中必须被重写。

    ~~~ java
    package qq.com.rzf.xh
    
    object TraitFields {
      def main(args: Array[String]): Unit = {
        //创建对象对特质中未进行初始化的属性进行重写
      val p1:p=new p with myTrait {
        override var age: Int = _
      }
      }
    }
    class p{
    
    }
    trait myTrait{
      var name:String=""
      var age:Int
    }
    //底层代码
    package qq.com.rzf.xh;
    
    import scala.runtime.TraitSetter;
    
    public final class TraitFields$
    {
      public static final  MODULE$;
    
      static
      {
        new ();
      }
    
      public void main(String[] args)
      {
          //动态混入特质，给对象添加属性
        p p1 = new p() { private int age;
          private String name;
    
          public String name() { return this.name; } 
          @TraitSetter
          public void name_$eq(String x$1) { this.name = x$1; } 
          public int age() { return this.age; } 
          public void age_$eq(int x$1) { this.age = x$1; } } ;
      }
    
      private TraitFields$() {
        MODULE$ = this;
      }
    }
    ~~~

11. 特质构造顺序

    ~~~ java
    trait AA {
      println("A...")
    }
    trait BB extends  AA {
      println("B....")
    }
    trait CC extends  BB {
      println("C....")
    }
    trait DD extends  BB {
      println("D....")
    }
    class EE {
      println("E...")
    }
    class FF extends EE with CC with DD {
      println("F....")
    }
    class KK extends EE {
      println("K....")
    }
    
    ~~~

    - 特质也是有构造器的，构造器中的内容由“字段的初始化”和一些其他语句构成。具体实现请参考“特质叠加”
    - 第一种特质构造顺序(声明类的同时混入特质)
      - 调用当前类的超类构造器
      - 第一个特质的父特质构造器
      - 第一个特质构造器
      - 第二个特质构造器的父特质构造器, 如果已经执行过，就不再执行
      - 第二个特质构造器
      - .......重复4,5的步骤(如果有第3个，第4个特质)
      - 当前类构造器   [案例演示]
    - 第2种特质构造顺序(在构建对象时，动态混入特质)
      - 先创建对象，在混入
      - 调用当前类的超类构造器
      - 当前类构造器
      - 第一个特质构造器的父特质构造器
      - 第一个特质构造器.
      - 第二个特质构造器的父特质构造器, 如果已经执行过，就不再执行
      - 第二个特质构造器
      - .......重复5,6的步骤(如果有第3个，第4个特质)
      - 当前类构造器   
    - 分析两种方式对构造顺序的影响第1种方式实际是构建类对象, 在混入特质时，该对象还没有创建。第2种方式实际是构造匿名子类，可以理解成在混入特质时，对象已经创建了。

12. 扩展类的特质

    - 特质可以继承类，以用来拓展该类的一些功能(和java的不同)，特质具有接口和类的特性。

      ~~~ java
      trait LoggedException extends Exception{
        def log(): Unit ={
          println(getMessage()) // 方法来自于Exception类
        }
      ~~~

    - 所有混入该特质的类，会自动成为那个特质所继承的超类的子类

      ~~~ java
      trait LoggedException extends Exception{
        def log(): Unit ={
          println(getMessage()) // 方法来自于Exception类
        }
      }
      //UnhappyException 就是Exception的子类.
      class UnhappyException extends LoggedException{
          // 已经是Exception的子类了，所以可以重写方法
          override def getMessage = "错误消息！"
      }
      
      ~~~

    - 如果混入该特质的类，已经继承了另一个类(A类)，则要求A类是特质超类的子类，否则就会出现了多继承现象，发生错误。

      ![](../img/scala/特质类.png)

13. 自身类型

    - 说明

      - 自身类型(self-type)：主要是为了解决特质的循环依赖问题，同时可以确保特质在不扩展某个类的情况下，依然可以做到限制混入该特质的类的类型。
      - 相当于对特质的使用增加了严格性限制

      ~~~ java
      //Logger就是自身类型特质
      trait Logger {
        // 明确告诉编译器，我就是Exception,如果没有这句话，下面的getMessage不能调用
        this: Exception =>
        def log(): Unit ={
          // 既然我就是Exception, 那么就可以调用其中的方法
          println(getMessage)
        }
      }
      class Console extends  Logger {} //对吗?
      //不对，应为Console类不是Exception类的子类，所以不可以继承Logger特质，Logger特质已经明确自己为Exception子类
      class Console extends Exception with Logger//对吗? 
      //对，因为Console是Exception的子类
      ~~~

14. type关键字

    使用type关键字可以定义新的数据类型名称
    本质上就是类型的一个别名

    ~~~ java
    type S = String
    var v : S = “abc”
    def test() : S = “xyz”
    ~~~

#### 4.1.8，嵌套类

- 在Scala中，你几乎可以在任何语法结构中内嵌任何语法结构。如在类中可以再定义一个类，这样的类是嵌套类，其他语法结构也是一样。
- 嵌套类类似于Java中的内部类。
- `java`类中共有五大类型成员
  - 属性，方法，内部类，构造器，代码块。

1. Java内部类的简单回顾

   - 在Java中，一个类的内部又完整的嵌套了另一个完整的类结构。被嵌套的类称为内部类(inner class)，嵌套其他类的类称为外部类。内部类最大的特点就是可以直接访问私有属性，并且可以体现类与类之间的包含关系

     ~~~ java
     Java内部类基本语法
     
     class Outer{	//外部类        
     	class Inner{	//内部类       
     }
     }class Other{	
     //外部其他类
     }
     ~~~

   - Java内部类的分类

     - 从定义在外部类的成员位置上来看，

       1. 成员内部类（没用static修饰）
       2. 和静态内部类（使用static修饰）
     - 定义在外部类局部位置上（比如方法内）来看：
       1. 分为局部内部类（有类名）  
       2. 匿名内部类（没有类名）

     ~~~ java
     package com.atguigu.chapter02;
     public class TestJavaClass {
         public static void main(String[] args) {
             //创建一个外部类对象
             OuterClass outer1 = new OuterClass();
             //创建一个外部类对象
             OuterClass outer2 = new OuterClass();
             // 创建Java成员内部类
             // 说明在Java中，将成员内部类当做一个属性，因此使用下面的方式来创建 outer1.new InnerClass().
             OuterClass.InnerClass inner1 = outer1.new InnerClass();
             OuterClass.InnerClass inner2 = outer2.new InnerClass();
     
             //下面的方法调用说明在java中，内部类只和类型相关，也就是说,只要是
             //OuterClass.InnerClass 类型的对象就可以传给 形参 InnerClass ic
             inner1.test(inner2);
             inner2.test(inner1);
     
             // 创建Java静态内部类
             // 因为在java中静态内部类是和类相关的，使用 new OuterClass.StaticInnerClass()
             OuterClass.StaticInnerClass staticInner = new OuterClass.StaticInnerClass();
         }}
     class OuterClass { //外部类
         class InnerClass { //成员内部类
             public void test( InnerClass ic ) {
                 System.out.println(ic);
             }}
     
         static class StaticInnerClass { //静态内部类
         }}
     
     ~~~

     - 内部类最大的特点就是可以直接访问私有属性，并且可以体现类与类之间的包含关系 

2. Scala嵌套类的使用1

   ~~~ java
   package qq.com.rzf.xh
   
   import qq.com
   import qq.com.rzf
   import qq.com.rzf.xh
   
   object ScalaInnerClass {
     def main(args: Array[String]): Unit = {
       //创建外部类
       val outerClass1 = new outerClass
       val outerClass2=new outerClass
       //创建成员内部类，和java语法不一样,默认情况下内部实例和外部类的对象关联
       val innerClas1 = new outerClass1.innerClass
       val innerClass2 = new outerClass2.innerClass
       //创建静态内部类，和java语法一样
       val innerStaticClass1 = new rzf.xh.outerClass.innerStaticClass
       val innerStaticClass2 = new com.rzf.xh.outerClass.innerStaticClass
     }
   
   }
   class outerClass{
     class innerClass{//成员内部类
   
     }
   }
   //伴生对象
   object outerClass{
     class innerStaticClass{//静态内部类
   
     }
   }
   
   ~~~

3. 在内部类中访问外部类的属性和方法两种方法。

   1. 内部类如果想要访问外部类的属性，可以通过外部类对象访问。即：访问方式：外部类名.this.属性名  

      ~~~ java
      package qq.com.rzf.xh
      
      import qq.com
      import qq.com.rzf
      import qq.com.rzf.xh
      
      object ScalaInnerClass {
        def main(args: Array[String]): Unit = {
          //创建外部类
          val outerClass1 = new outerClass
          val outerClass2=new outerClass
          //创建成员内部类，和java语法不一样,默认情况下内部实例和外部类的对象关联
          val innerClas1 = new outerClass1.innerClass
          val innerClass2 = new outerClass2.innerClass
          //创建静态内部类，和java语法一样
          val innerStaticClass1 = new rzf.xh.outerClass.innerStaticClass
          val innerStaticClass2 = new com.rzf.xh.outerClass.innerStaticClass
          innerClas1.info()//调用方法
        }
      
      }
      class outerClass{
        var name : String = "scott"
        private var sal : Double = 1.2//私有属性
        class innerClass{//成员内部类
      
          def info() = {
            // 访问方式：外部类名.this.属性名
            // 怎么理解 ScalaOuterClass.this 就相当于是 ScalaOuterClass 这个外部类的一个实例,
            // 然后通过 ScalaOuterClass.this 实例对象去访问 name 属性
            // 只是这种写法比较特别，学习java的同学可能更容易理解 ScalaOuterClass.class 的写法.
            //在成员内部类中可以访问外部类的的私有属性
            println("name = " + outerClass.this.name
              + " age =" + outerClass.this.sal)
          }
      
        }
      }
      
      //伴生对象
      object outerClass{
        class innerStaticClass{//静态内部类
      
        }
      }
      ~~~

   2. 内部类如果想要访问外部类的属性，也可以通过外部类别名访问(推荐)。即：访问方式：外部类名别名.属性名   【外部类名.this  等价 外部类名别】

      ~~~ java
      class ScalaOuterClass {
        myOuter =>  //这样写，你可以理解成这样写，myOuter就是代表外部类的一个对象.
        class ScalaInnerClass { //成员内部类
          def info() = {
            println("name = " + ScalaOuterClass.this.name
              + " age =" + ScalaOuterClass.this.sal)
            println("-----------------------------------")
            println("name = " + myOuter.name
              + " age =" + myOuter.sal)
          }}
        // 当给外部指定别名时，需要将外部类的属性放到别名后.
        var name : String = "scott"
        private var sal : Double = 1.2
      }
      object ScalaOuterClass {  //伴生对象
        class ScalaStaticInnerClass { //静态内部类
        }}
      inner1.info()
      
      ~~~

   3. 类型投影

      ~~~ java
      class ScalaOuterClass3 {
        myOuter =>
        class ScalaInnerClass3 { //成员内部类
          def test(ic: ScalaInnerClass3): Unit = {
            System.out.println(ic)
          }
        }
      }
      object Scala01_Class { 
          def main(args: Array[String]): Unit = {    
              val outer1 : ScalaOuterClass3 = new ScalaOuterClass3();    
              val outer2 : ScalaOuterClass3 = new ScalaOuterClass3();    
              val inner1 = new outer1.ScalaInnerClass3()    
                  val inner2 = new outer2.ScalaInnerClass3()        inner1.test(inner1) // ok   
                  inner1.test(inner2) // error 原因是scala内部类对象和外部类对象相关.
         //这时可以使用类型投影来屏蔽类型不同  }}
      //说明下面调用test 的 正确和错误的原因：
      //1.Java中的内部类从属于外部类,因此在java中 inner.test(inner2) 就可以，因为是按类型来匹配的。
      //2 Scala中内部类从属于外部类的对象，所以外部类的对象不一样，创建出来的内部类也不一样，无法互换使用
      //3. 比如你使用ideal 看一下在inner1.test()的形参上，它提示的类型是 outer1.ScalaOuterClass, 而不是ScalaOuterClassinner1.test(inner1) // ok inner1.test(inner2) // 错误
      //案例
      package qq.com.rzf.xh
      
      import qq.com
      import qq.com.rzf
      import qq.com.rzf.xh
      
      object ScalaInnerClass {
        def main(args: Array[String]): Unit = {
          //创建外部类
          val outerClass1 = new outerClass
          val outerClass2=new outerClass
          //创建成员内部类，和java语法不一样,默认情况下内部实例和外部类的对象关联
          val innerClas1 = new outerClass1.innerClass
          val innerClass2 = new outerClass2.innerClass
          //创建静态内部类，和java语法一样
          val innerStaticClass1 = new rzf.xh.outerClass.innerStaticClass
          val innerStaticClass2 = new com.rzf.xh.outerClass.innerStaticClass
          innerClas1.info()//调用方法
          //在这里调用内部类方法test
          //如果传入参数是自己，不会报错，应为scala内部类对象创建的时候会自动关联外部类对象
          innerClas1.test(innerClas1)
          //innerClas1.test(innerClass2)在这里会报错，innerClas1和外部类对象不一致问题
          //解决方式是在方法的参数前面添加声明即可(ic: outerClass#innerClass)
          innerClas1.test(innerClass2)//此时不会报错
      
        }
      
      }
      class outerClass{
        var name : String = "scott"
        private var sal : Double = 1.2//私有属性
        class innerClass{//成员内部类
          //内部成员方法
          def test(ic: outerClass#innerClass): Unit = {
            System.out.println(ic)
          }
      
      
          def info() = {
            // 访问方式：外部类名.this.属性名
            // 怎么理解 ScalaOuterClass.this 就相当于是 ScalaOuterClass 这个外部类的一个实例,
            // 然后通过 ScalaOuterClass.this 实例对象去访问 name 属性
            // 只是这种写法比较特别，学习java的同学可能更容易理解 ScalaOuterClass.class 的写法.
            //在成员内部类中可以访问外部类的的私有属性
            println("name = " + outerClass.this.name
              + " age =" + outerClass.this.sal)
          }
      
        }
      }
      
      //伴生对象
      object outerClass{
        class innerStaticClass{//静态内部类
      
        }
      }
      
      ~~~

      - 解决方式-使用类型投影

        类型投影是指：在方法声明上，如果使用  外部类#内部类  的方式，表示忽略内部类的对象关系，等同于Java中内部类的语法操作，我们将这种方式称之为 类型投影（即：忽略对象的创建方式，只考虑类型）

### 第五章，隐式转换和隐式参数

#### 5.1.1，提出问题

- 引出隐式转换的实际需要=>指定某些数据类型的相互转化

~~~ java
package com.atguigu.scala.conversion

object Scala01 {
  def main(args: Array[String]): Unit = {
    val num1 : Int = 3.5 
    val num2 : Int = 4.6
    println(num1)
    println(num2)
  }
}
~~~

#### 5.1.2，隐式函数基本介绍

- 隐式转换函数是以implicit关键字声明的带有单个参数的函数。这种函数将会自动应用，将值从一种类型转换为另一种类型

- 入门案例：

  ~~~java
  package qq.com.rzf.xm
  
  object ImplicteConvert {
    def main(args: Array[String]): Unit = {
      //实现将double类型转换为int
      //隐式函数在作用于才可以生效
      implicit def f1(num:Double):Int={//在底层生成f1$1
        num.toInt
      }
      //使用函数
     
      var number:Int=3.5//在底层，编译器帮助我们做了f1$1(number)
      println(number)//输出3
    }
  }
  //底层源码
  package qq.com.rzf.xm;
  
  import scala.Predef.;
  import scala.runtime.BoxesRunTime;
  
  public final class ImplicteConvert$
  {
    public static final  MODULE$;
  
    static
    {
      new ();
    }
  
    public void main(String[] args)
    {
      int number = f1$1(3.5D);//底层编译器自动做了转换
      Predef..MODULE$.println(BoxesRunTime.boxToInteger(number));
    }
  
    private final int f1$1(double num)
    {
      return (int)num;
    }
  
    private ImplicteConvert$()
    {
      MODULE$ = this;
    }
  }
  ~~~

#### 5.1.3，隐式转换的注意事项和细节

1. 隐式转换函数的函数名可以是任意的，隐式转换与函数名称无关，只与函数签名（函数参数类型和返回值类型）有关。

2. 隐式函数可以有多个(即：隐式函数列表)，但是需要保证在当前环境下，只有一个隐式函数能被识别

   ~~~ java
   package qq.com.rzf.xm
   
   object ImplicteConvert {
     def main(args: Array[String]): Unit = {
       //实现将double类型转换为int
       //隐式函数在作用于才可以生效
       implicit def f1(num:Double):Int={//在底层生成f1$1
         num.toInt
       }
       implicit def f2(num:Float):Int={//在底层生成f1$1
         num.toInt
       }
       //使用函数
       var number:Int=3.5//在底层，编译器帮助我们做了f1$1(number)
       println(number)//输出3
       var number1:Int=4.5f//如果没有f2函数，会报错
       println(number1)//输出3
     }
   }
   //实例
   //在当前环境中，不能存在满足条件的多个隐式函数
   implicit def a(d: Double) = d.toInt
   implicit def b(d: Double) = d.toInt 
   val i1: Int = 3.5 //（X）在转换时，识别出有两个方法可以被使用，就不确定调用哪一个，所以出错
   println(i1)
   ~~~

#### 5.1.4，隐式转换丰富类库功能

1. 基本介绍

   如果需要为一个类增加一个方法，可以通过隐式转换来实现。（动态增加功能）比如想为MySQL类增加一个delete方法，不用修改源代码。

2. 分析解决方案

   在当前程序中，如果想要给MySQL类增加功能是非常简单的，但是在实际项目中，如果想要增加新的功能就会需要改变源代码，这是很难接受的。而且违背了软件开发的OCP开发原则 (开闭原则 open close priceple) 
   在这种情况下，可以通过隐式转换函数给类动态添加功能。

   ~~~ java
   package qq.com.rzf.xm
   
   object Implicit {
     def main(args: Array[String]): Unit = {
       implicit def addDelete(mysql:MySQL): DB = {
         new DB //创建一个新对象
       }
       val mysql = new MySQL
       mysql.insert()
       mysql.delete()//现在可以使用delete方法，但是并没有修改源码
     }
   
   }
   class MySQL{
     def insert(): Unit = {
       println("insert")
     }
   }
   class DB {
     def delete(): Unit = {
       println("delete")
     }
   }
   //叠层代码
   package qq.com.rzf.xm;
   
   public final class Implicit$
   {
     public static final  MODULE$;
   
     static
     {
       new ();
     }
   
     public void main(String[] args)
     {
       MySQL mysql = new MySQL();
       mysql.insert();
         //传入我们创建的mysql对象，返回db对象，调用db的方法
       addDelete$1(mysql).delete();
     }
   
     private final DB addDelete$1(MySQL mysql)
     {
       return new DB();
     }
   
     private Implicit$()
     {
       MODULE$ = this;
     }
   }
   ~~~

#### 5.1.5，隐式值

1. 基本介绍

   隐式值也叫隐式变量，将某个形参变量标记为implicit，所以编译器会在方法省略隐式参数的情况下去搜索作用域内的隐式值作为缺省参数

2. 入门：

   ~~~ java
   package qq.com.rzf.xm
   
   object ImplicitFields {
     def main(args: Array[String]): Unit = {
       implicit var name:String="tom"//定义隐式参数
       def printName(implicit name:String):Unit={
         println("姓名是:"+name)
       }
       printName//可以不传入参数，自动传入，直接打印
     }
   }
   //底层代码
   package qq.com.rzf.xm;
   
   import scala.Predef.;
   import scala.collection.mutable.StringBuilder;
   
   public final class ImplicitFields$
   {
     public static final  MODULE$;
   
     static
     {
       new ();
     }
   
     public void main(String[] args)
     {
       String name = "tom";
   
       printName$1(name);
     }
   
     private final void printName$1(String name)
     {
       Predef..MODULE$.println(new StringBuilder().append("姓名是:").append(name).toString());
     }
   
     private ImplicitFields$() {
       MODULE$ = this;
     }
   }
   ~~~

3. 练习

   ~~~ java
   //案例一
   package qq.com.rzf.xm
   
   object ImplicitDemo {
     def main(args: Array[String]): Unit = {
       // 隐式变量（值）
       implicit val name1: String = "Scala"
       implicit val name2: String = "World"
   
       def hello(implicit content: String = "jack"): Unit = {
         println("Hello " + content)
       }
       hello//报错，应为匹配到两个隐式值，无法确定使用哪一个
     }
   }
   //案例二
   //当同时有隐式值和默认值的时候，隐式值优先级高
   object ImplicitDemo {
     def main(args: Array[String]): Unit = {
       // 隐式变量（值）
       implicit val name1: String = "Scala"
       //implicit val name2: String = "World"
   
       def hello(implicit content: String = "jack"): Unit = {
         println("Hello " + content)
       }
       hello//报错，应为匹配到两个隐式值，无法确定使用哪一个
     }
   }
   //案例三
   当没有匹配到隐式值的时候，有默认值的话，默认值优先级高
   package qq.com.rzf.xm
   
   object ImplicitDemo {
     def main(args: Array[String]): Unit = {
       // 隐式变量（值）
       implicit val name1: Int = 10
       //implicit val name2: String = "World"
   
       def hello(implicit content: String = "jack"): Unit = {
         println("Hello " + content)
       }
       hello
     }
   }
   //案例四
   //既没有默认值，也没有隐式值，还没有传值，但如果传值，可以运行
   package qq.com.rzf.xm
   
   object ImplicitDemo {
     def main(args: Array[String]): Unit = {
       // 隐式变量（值）
       implicit val name1: Int = 10
       //implicit val name2: String = "World"
   
       def hello(implicit content: String ): Unit = {
         println("Hello " + content)
       }
       //hello//报错，因为没有隐式值，也没有默认值，也没有传值，就会报错
       hello("rzf")//传值，可以运行
     }
   }
   //案例5
   //有隐式值和传入默认值
   package qq.com.rzf.xm
   
   object ImplicitDemo {
     def main(args: Array[String]): Unit = {
       // 隐式变量（值）
       //implicit val name1: Int = 10
       implicit val name2: String = "World"
   
       def hello(implicit content: String ): Unit = {
         println("Hello " + content)
       }
       //hello//报错，因为没有隐式值，也没有默认值，也没有传值，就会报错
       hello("rzf")//传值，可以运行
     }
   }
   //输出hello rzf
   ~~~

   - 小结：当有隐式值，默认值，传值情况下，优先级为：传值，隐式值，默认值。
   - 隐式值匹配时不可以有二义性（即匹配值有多个）。
   - 如果三个一个都匹配不到，就会报错。

#### 5.1.6，隐式类

1. 在scala2.10后提供了隐式类，可以使用implicit声明类，隐式类的非常强大，同样可以扩展类的功能，比前面使用隐式转换丰富类库功能更加的方便，在集合中隐式类会发挥重要的作用。

2. 隐式类使用有如下几个特点：

   1. 其所带的构造参数有且只能有一个
   2. 隐式类必须被定义在“类”或“伴生对象”或“包对象”里，即隐式类不能是 顶级的(top-level  objects)。
   3. 隐式类不能是case class（case class在后续介绍 样例类）
   4. 作用域内不能有与之相同名称的标识符

   ~~~ java
   package qq.com.rzf.xm
   
   object ImplicitClass {
     def main(args: Array[String]): Unit = {
       //DB1会对应生成隐式类
       /**
        * DB1对应是一个隐式类，
        * 当我们在该隐式类作用范围内创建MySQL1实例，该隐式类就会生效
        * 这样做其实也就是为了在不修改MySQL1类源码情况下给类增加功能
        */
       implicit class DB1(val m: MySQL1) {
         def addSuffix(): String = { //方法
           m + " scala"
         }
       }
       val mysql1 = new MySQL1
       mysql1.sayOk()
       println(mysql1.addSuffix())
     }
   
   }
   class MySQL1 {
     def sayOk(): Unit = {
       println("sayOk")
     }
   }
   
   ~~~

#### 5.1.7，隐式的转换时机

- 当方法中的参数的类型与目标类型不一致时

- 当对象调用所在类中不存在的方法或成员时，编译器会自动将对象进行隐式转换（根据类型）

  ~~~ java
   // test(9.8)报错，因为类型不匹配
      /**
       * 解决方法两种
       * 
       */
        //第一种：
        test(9.8.toInt)
      //第二种
      implicit def f1(num:Double):Int={
        num.toInt
      }
      test(9.8.toInt)
      def test(num:Int):Unit={
        println(num)
      }
  ~~~

#### 5.1.8，隐式解析机制

即编译器是如何查找到缺失信息的，解析具有以下两种规则：
首先会在当前代码作用域下查找隐式实体（隐式方法、隐式类、隐式对象）。
如果第一条规则查找隐式实体失败，会继续在隐式参数的类型的作用域里查找。类型的作用域是指与该类型相关联的全部伴生模块，一个隐式实体的类型T它的查找范围如下(第二种情况范围广且复杂在使用时，应当尽量避免出现)：
a)  如果T被定义为T with A with B with C,那么A,B,C都是T的部分，在T的隐式解析过程中，它们的伴生对象都会被搜索。
b)  如果T是参数化类型，那么类型参数和与类型参数相关联的部分都算作T的部分，比如List[String]的隐式搜索会搜索List的伴生对象和String的伴生对象。
c)  如果T是一个单例类型p.T，即T是属于某个p对象内，那么这个p对象也会被搜索。
d)  如果T是个类型注入S#T，那么S和T都会被搜索。

#### 5.1.9，隐式转换的前提

- 在进行隐式转换时，需要遵守两个基本的前提：

  不能存在二义性
  隐式操作不能嵌套使用 

  ![](../img/scala/隐式嵌套.png )






​    

   

   

   

   

   

   

   

   

   

   

   

   

   

   

