## UML类图

### UML基本介绍

1. `UML——Unified modeling language UML `(统一建模语言)，是一种用于软件系统分析和设计的语言工具，它用于帮助软件开发人员进行思考和记录思路的结果
2. `UML` 本身是一套符号的规定，就像数学符号和化学符号一样，这些符号用于描述软件模型中的各个元素和他们之间的关系，比如类、接口、实现、泛化、依赖、组合、聚合等，如右图:

### UML图分类

1. 用例图(use case)
2. 静态结构图：类图、对象图、包图、组件图、部署图
3. 动态行为图：交互图（时序图与协作图）、状态图、活动图

> 类图是描述类与类之间的关系的，是 `UML `图中最核心的

**类图用于描述系统中的类(对象)本身的组成和类(对象)之间的各种静态关系。**

**类之间的关系：依赖、泛化（继承）、实现、关联、聚合与组合。**

### 类图—依赖关系（Dependence）

只要是在类中用到了对方，那么他们之间就存在依赖关系。如果没有对方，连编绎都通过不了。

**代码说明**

```java
public class Dependence {
    private PersonDao personDao;
    public void save(Person person){}
    public IDCard getIDCard(Integer personID){
        return null;
    }
    public void modify(){
        Department department=new Department();
    }
    public static void main(String[] args) {

    }
}
class PersonDao{}

class IDCard{ }

class Person{}

class Department{}

```

**类图**

![1608448604981](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/design/202012/20/151646-59730.png)

- 在这里也可以把依赖换成聚合关系。

**小结**

> 1)	类中用到了对方
>
> 2) 	如果是类的成员属性
>
> 3)	如果是方法的返回类型
>
> 4)	是方法接收的参数类型
>
> 5)	方法中使用到

### 类图—泛化关系 (generalization）

泛化关系实际上就是继承关系，他是依赖关系的特例

**代码说明**

```java
public abstract class DaoSupport{ public void save(Object entity){
}
public void delete(Object id){
}

}

public class PersonServiceBean extends Daosupport{
}

```

**类图**

![1608448892685](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/design/202012/20/152139-910427.png)

**小结**

> 1)      泛化关系实际上就是继承关系
>
> 2)      如果 A 类继承了 B 类，我们就说 A 和 B 存在泛化关系

### 类图—实现关系（Implementation）

实现关系实际上就是 **A** 类实现 **B** 接口，他是依赖关系的特例

**代码说明**

```java
public interface PersonService { public void delete(Interger id);
}
public class PersonServiceBean implements PersonService { 
  public void delete(Interger id){}
}

```

**类图**

![1608449016439](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/design/202012/20/152338-893737.png)

### 类图—关联关系（Association）

![1608451495287](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/24/165124-607443.png)

### 类图—聚合关系（Aggregation）

**介绍**

聚合关系（`Aggregation`）表示的是**整体和部分**的关系，**整体与部分可以分开**。聚合关系是关联关系的特例，所以他具有关联的导航性与多重性。

如：一台电脑由键盘(`keyboard`)、显示器(`monitor`)，鼠标等组成；组成电脑的各个配件是可以从电脑上分离出来的，使用带空心菱形的实线来表示：

**代码说明**

```java
public class Computer {
    private Monitor monitor;
    private Mouse mouse;

    public void setMonitor(Monitor monitor) {
        this.monitor = monitor;
    }

    public void setMouse(Mouse mouse) {
        this.mouse = mouse;
    }
}
class Monitor{}
class Mouse{}
```

**类图说明**

![1608452075089](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/design/202012/20/161436-77360.png)

### 类图—组合关系（Composition）

**基本介绍**

组合关系：也是整体与部分的关系，**但是整体与部分不可以分开**

- 再看一个案例：在程序中我们定义实体：`Person `与 `IDCard`、`Head`, 那么 `Head` 和 `Person` 就是 组合，`IDCard` 和`Person `就是聚合。
- 但是如果在程序中 `Person` 实体中定义了对` IDCard` 进行级联删除，即删除 `Person` 时连同` IDCard` 一起删除，那么 `IDCard ` 和 `Person` 就是组合了.

**代码说明**

```java
public class Person{ 
  private IDCard card;//身份证可能丢失，所以是聚合关系
	private Head head = new Head();//组合关系
}
class IDCard{} 
class Head{}
```

**类图**

![1608452386196](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/design/202012/20/161948-221855.png)

## 设计模式

### 设计模式介绍

1. 设计模式是程序员在面对同类软件工程设计问题所总结出来的有用的经验，模式不是代码，而是某类问题的通用解决方案，设计模式（Design pattern）代表了最佳的实践。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。
2. 设计模式的本质提高 软件的维护性，通用性和扩展性，并降低软件的复杂度

### 设计模式的分类

设计模式分为三种类型，共 **23** 种

- 创建型模式：单例模式、抽象工厂模式、原型模式（克隆对象）、建造者模式、工厂模式。
- 结构型模式：适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式。
- 行为型模式：模版方法模式、命令模式、访问者模式、迭代器模式、观察者模式、中介者模式、备忘录模式、解释器模式（`Interpreter `模式）、状态模式、策略模式、职责链模式(责任链模式)。

> 注意：不同的书籍上对分类和名称略有差别
>
> - 创建型模式：主要讲如何创建一个对象。
> - 结构性模式：是站在软件结构角度思考问题，如何设计软件结构才具有更好的可扩展性，更加容易维护。
> - 行为型模式：站在方法的角度思考，如何设计方法更加的合理

