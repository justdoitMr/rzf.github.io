# Mysql基础

<!-- TOC -->

- [Mysql基础](#mysql基础)
	- [数据库概述](#数据库概述)
		- [相关概念](#相关概念)
		- [数据库存储数据的特点](#数据库存储数据的特点)
	- [Mysql数据库介绍](#mysql数据库介绍)
		- [Mysql介绍](#mysql介绍)
		- [mysql优点](#mysql优点)
		- [MySQL服务端的安装和卸载](#mysql服务端的安装和卸载)
			- [mysql数据库的卸载](#mysql数据库的卸载)
			- [Mysql服务的配置](#mysql服务的配置)
		- [Mysql的使用](#mysql的使用)
			- [MYSQL服务的启动](#mysql服务的启动)
			- [客户端的登录](#客户端的登录)
		- [常用命令介绍](#常用命令介绍)
		- [Mysql语法规范](#mysql语法规范)
	- [SQL语句](#sql语句)
		- [SQL概述](#sql概述)
			- [什么是sql](#什么是sql)
			- [SQL语法要求](#sql语法要求)
			- [分类](#分类)
		- [DQL语言](#dql语言)
			- [基础查询](#基础查询)
			- [条件查询](#条件查询)
			- [排序查询](#排序查询)
			- [常见函数学习](#常见函数学习)
				- [字符串函数](#字符串函数)
				- [数学函数](#数学函数)
				- [日期函数](#日期函数)
				- [其他函数](#其他函数)
				- [流程控制函数](#流程控制函数)
				- [练习](#练习)
				- [分组函数一](#分组函数一)
				- [分组查询二](#分组查询二)
				- [练习](#练习-1)
			- [链接查询](#链接查询)
			- [sql99语法](#sql99语法)
			- [子查询](#子查询)
				- [**where或having后面**](#where或having后面)
				- [select后面](#select后面)
				- [from后面](#from后面)
				- [exists后面](#exists后面)
- [下面是相关子查询，前面是非相关子查询](#下面是相关子查询前面是非相关子查询)
			- [分页查询](#分页查询)
			- [union联合查询](#union联合查询)
		- [DML(数据操纵语言)](#dml数据操纵语言)
				- [插入语句](#插入语句)
				- [修改语句](#修改语句)
				- [删除操作](#删除操作)
		- [DDL(数据定义语言)](#ddl数据定义语言)
				- [库管理](#库管理)
				- [表管理](#表管理)

<!-- /TOC -->

## 数据库概述

### 相关概念

**DB**

数据库（ database ）：存储数据的“仓库”。它保存了一系列有组织的数据。

**DBMS**

数据库管理系统（ Database Management System ）。数据库是通过 DBMS 创建和操作的容器。

**sql**

结构化查询语言（ Structure Query Language ）：专门用来与数据库通信的语。

### 数据库存储数据的特点

- 将数据放到表中，表再放到库中 
- 一个数据库中可以有多个表，每个表都有一个的名字，用来标识自己。表名具有唯一性。 
- 表具有一些特性，这些特性定义了数据在表中如何存储，类似java中 “类”的设计。
- 表由列组成，我们也称为字段。所有表都是由一个或多个列组成的，每一列类似java 中的”属性”
- 表中的数据是按行存储的，每一行类似于java中的“对象”

> 表——类
>
> 列，字段——属性
>
> 行——对象

## Mysql数据库介绍

### Mysql介绍

**MySQL**是一种开放源代码的关系型数据库管理系统，开发者为瑞典MySQL AB公司。在2008年1月16号被Sun公司收购。而2009年,SUN又被Oracle收购.目前 MySQL被广泛地应用在Internet上的中小型网站中。由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，许多中小型网站为了降低网站总体拥有成本而选择了MySQL作为网站数据库（Facebook, Twitter, YouTube）。阿里提出“去IOE”，更多网站也开始选择MySQL。

### mysql优点

- 成本低：开放源代码，一般可以免费试用 

- 性能高：执行很快 

- 简单：很容易安装和使用 

###  MySQL服务端的安装和卸载

**DBMS**分为两类：

- 基于共享文件系统的DBMS （Access ） 

- 基于客户机——服务器的DBMS   C/S，（**MySQL**、Oracle、SqlServer） 

#### mysql数据库的卸载

1. 在控制面板中卸载即可。
2. 清除残余的文件

**清除安装的残余文件**

![1611895757672](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1611895757672.png)

**清除数据的残余文件**

![1611895781433](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/124942-958663.png)

**清理注册表**

~~~ java
如果前两步做了，再次安装还是失败，那么可以清理注册表
1：HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\MySQL服务 目录删除
2：HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\MySQL服务 目录删除
3：HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\MySQL服务 目录删除
4：HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\MySQL服务 目录删除
5：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\MySQL服务目录删除
6：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\MySQL服务删除
注册表中的ControlSet001,ControlSet002,不一定是001和002,可能是ControlSet005、006之类
~~~

#### Mysql服务的配置

单击“Finish”按钮完成安装过程。如果想马上配置数据库连接，选择“Launch the MySQL Instance Configuration Wizard”复选框。如果现在没有配置，以后想要配置或重新配置都可以在“MySQL Server”的安装目录的bin目录下（例如：D:\ProgramFiles\MySQL5.5\MySQL Server 5.5\bin）找到“MySQLInstanceConfig.exe”打开“MySQL Instance Configuration Wizard”向导。

1. 开始配置Mysql

![1611895949841](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/125231-601513.png)

2. 选择配置类型

![1611895981756](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/23/145842-203818.png)

选择配置方式，“Detailed Configuration（手动精确配置）”、“Standard Configuration（标准配置）”，我们选择“Detailed Configuration”，方便熟悉配置过程。

3. 选择mysql的应用模式

![1611896027612](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1611896027612.png)

Develop Machine（开发机），使用最小数量的内存

Server Machine（服务器），使用中等大小的内存

Dedicated MySQL Server Machine（专用服务器），使用当前可用的最大内存。

4. 选择数据库用途

![1611896069472](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/125431-139552.png)

选择mysql数据库的大致用途：

“Multifunctional Database（通用多功能型，好）”：此选项对事务性存储引擎（InnoDB）和非事务性（MyISAM）存储引擎的存取速度都很快。

“Transactional Database Only（服务器类型，专注于事务处理，一般）”：此选项主要优化了事务性存储引擎（InnoDB），但是非事务性（MyISAM）存储引擎也能用。

“Non-Transactional Database Only（非事务处理型，较简单）主要做一些监控、记数用，对MyISAM数据类型的支持仅限于non-transactional，注意事务性存储引擎（InnoDB）不能用。

5. **配置InnoDB数据文件目录**

![1611896112512](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1611896112512.png)

InnoDB的数据文件会在数据库第一次启动的时候创建，默认会创建在MySQL的安装目录下。用户可以根据实际的空间状况进行路径的选择。

6. 并发连接设置

![1611896149992](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/125556-557557.png)

选择您的网站的一般mysql 访问量，同时连接的数目，“Decision Support(DSS)/OLAP（决策支持系统，20个左右）”、“Online Transaction Processing(OLTP)（在线事务系统，500个左右）”、“Manual Setting（手动设置，自己输一个数）”

7. 网络选项设置

![1611896197602](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/125639-613110.png)

是否启用TCP/IP连接，设定端口，如果不启用，就只能在自己的机器上访问mysql 数据库了，我这里启用，把前面的勾打上，Port Number：3306，还有一个关于防火墙的设置“Add firewall exception ……”需要选中，将MYSQL服务的监听端口加为windows防火墙例外，避免防火墙阻断。

在这个页面上，您还可以选择“启用标准模式”（Enable Strict Mode），这样MySQL就不会允许细小的语法错误。尽量使用标准模式，因为它可以降低**有害数据**进入数据库的可能性。

8. 选择字符集

![1611896241813](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/125735-56459.png)

注意：如果要用原来数据库的数据，最好能确定原来数据库用的是什么编码，如果这里设置的编码和原来数据库数据的编码不一致，在使用的时候可能会出现乱码。

这个比较重要，就是对mysql默认数据库语言编码进行设置，第一个是西文编码，第二个是多字节的通用utf8编码，第三个，手工选择字符集。

提示：

如果安装时选择了字符集和“utf8”，通过命令行客户端来操作数据库时，有时候会出现乱码，

这是因为“命令行客户端”默认是GBK字符集，因此客户端与服务器端就出现了不一致的情况，会出现乱码。

可以在客户端执行：

mysql> set names gbk;  

可以通过以下命令查看：

mysql> show variables like 'character_set_%';

对于客户端和服务器的交互操作，MySQL提供了3个不同的参数：character_set_client、character_set_connection、character_set_results，分别代表客户端、连接和返回结果的字符集。通常情况下，这3个字符集应该是相同的，才能确保用户写入的数据可以正确的读出和写入。“set names xxx;”命令可以同时修改这3个参数的值，但是需要每次连接都重新设置。

9. 安全选择

![1611896346448](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/125909-644215.png)

选择是否将mysql 安装为windows服务，还可以指定Service Name（服务标识名称，例如我这里取名为“MySQL5.5”），是否将mysql的bin目录加入到Windows PATH环境变量中（加入后，就可以直接使用bin下的命令）”，我这里全部打上了勾。

10. 设置密码

![1611896387993](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/125949-558005.png)

这一步询问是否要修改默认root 用户（超级管理）的密码（默认为空），“New root password”如果要修改，就在此填入新密码，“Confirm（再输一遍）”内再填一次，防止输错。（如果是重装，并且之前已经设置了密码，在这里更改密码可能会出错，请留空，并将“Modify Security Settings”前面的勾去掉，安装配置完成后另行修改密码） 

“Enable root access from remotemachines（是否允许root 用户在其它的机器或使用IP地址登陆，如果要安全，就不要勾上，如果要方便，就勾上它）”。如果没有勾选，默认只支持localhost和127.0.0.1连接。

最后“Create An Anonymous Account（新建一个匿名用户，匿名用户可以连接数据库，不能操作数据，包括查询，如果要有操作数据的权限需要单独分配）”，一般就不用勾了

11. 最后点击执行即可

![1611896428835](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/130030-51993.png)

12. MYSQL安装目录

![1611896477053](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/130117-632917.png)

- 经过上面配置之后，我们发现MYSQL安装目录中多出一个my.ini的配置文件

- bin目录中都是可执行文件；

- MySQL的**数据存储**目录为data

- data目录通常在C:\Documents and Settings\All Users\Application Data\MySQL\MySQL Server 5.1\data位置。

- 在data下的每个目录都代表一个数据库。

**Mysql环境变量的配置**

~~~ java
//将C:\Program Files (x86)\MySQL\MySQL Server 5.5\bin;添加到path中
~~~

### Mysql的使用

#### MYSQL服务的启动

**第一种**

~~~ java
以管理员身份运行
net  start  MySQL服务名
net  stop  MySQL服务名
~~~

**第二种**

在windows的程序服务中启动。

在启动mysql服务后，打开windows任务管理器，会有一个名为mysqld.exe的进程运行，所以mysqld.exe才是MySQL服务器程序。

#### 客户端的登录

在启动MySQL服务器后，我们需要使用管理员用户登录MySQL服务器，然后来对服务器进行操作。登录MySQL需要使用MySQL的客户端程序：mysql.exe

~~~ java
登录：mysql -u root -p root -h localhost；
-u：后面的root是用户名，这里使用的是超级管理员root；
-p：后面的root是密码，这是在安装MySQL时就已经指定的密码；
-h：后面给出的localhost是服务器主机名，它是可以省略的，例如：mysql -u root -p root；
退出：quit或exit；
在登录成功后，打开windows任务管理器，会有一个名为mysql.exe的进程运行，所以mysql.exe是客户端程序。
~~~

**第一种**

使用mysql自带的客户端命令行进行登录，直接输入密码即可

![1611896741638](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/130543-303405.png)

**方式二**

命令行登录

~~~ java
mysql -h 主机名 -P 端口号 -u 用户名 -p密码
//例如：mysql -h localhost -P 3306 -u root -proot	
~~~

> -p与密码之间不能有空格，其他参数名与参数值之间可以有空格也可以没有空格
>
> ~~~ java
> mysql -hlocalhost -P3306 -uroot -proot
> ~~~
>
> 密码建议在一行输入
>
> ~~~ java
> mysql -h localhost -P 3306 -u root -p
> 
> Enter password:****
> ~~~
>
> 如果是连本机：-hlocalhost就可以省略，如果端口号没有修改：-P3306也可以省略
>
> ~~~ java
> 简写成：mysql -u root -p
> 
> Enter password:****
> ~~~

**图示**

![1611896989189](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/29/130949-674490.png)

连接成功后，有关于MySQL Server服务版本的信息，还有第几次连接的id标识。也可以在命令行通过以下方式获取MySQL Server服务版本的信息

> 退出mysql服务使用exit命令即可。

**方式三**

使用可视化工具登录



### 常用命令介绍

1. 显示所有数据库

~~~ java
mysql> show databases;
~~~

2. 使用数据库

~~~ java
mysql> use test;
~~~

3. 查看数据库中的表

~~~ java
mysql> show tables;
~~~

4. 切换数据库并且查看

~~~ java
//从test数据库切换到mysql数据库查看
use mysql;
//查看数据库
show tables;
//也可以使用下面语句
mysql> show tables from mysql;//但是现在还是在test数据库中
//查看当前所在的数据库
mysql> select database();
~~~

5. 查看表的结构

~~~ java
desc student;
//查看表中数据
select * from student;
~~~

6. 查看当前数据库的版本

~~~ java
mysql> select version();
//控制台输入：
mysql --version
~~~

### Mysql语法规范

1. mysql不区分大小写，建议关键字小写，表名，列明小写。
2. 每一条命令使用“；”结尾。
3. 每一条命令根据需要如果非常长，可以使用缩进或者换行。
4. 以“#”作为注释，-- 空格+注释。多行注释：/**/。
5. mysql不区分字符和字符串，统一使用单引号。

## SQL语句

### SQL概述

#### 什么是sql

SQL（Structured Query Language）是“结构化查询语言”，它是对关系型数据库的操作语言。它可以应用到所有关系型数据库中，例如：MySQL、Oracle、SQL Server等。SQL标准有：

#### SQL语法要求

1. SQL语句可以单行或多行书写，以分号结尾；

2. 可以用空格和缩进来来增强语句的可读性；

3. 关键字不区别大小写，建议使用大写；

#### 分类

1. DDL（Data Definition Language）：数据定义语言，用来定义数据库对象：库、表、列等；

2. DML（Data Manipulation Language）：数据操作语言，用来定义数据库记录（数据）；

3. DCL（Data Control Language）：数据控制语言，用来定义访问权限和安全级别；

4. DQL（Data Query Language）：数据查询语言，用来查询记录（数据）。

### DQL语言

**语法**

~~~ sql
SELECT 
selection_list /*要查询的列名称*/
FROM 
table_list /*要查询的表名称*/
WHERE 
condition /*行条件*/
GROUP BY 
grouping_columns /*对结果分组*/
HAVING 
condition /*分组后的行条件*/
ORDER BY
sorting_columns /*对结果分组*/
LIMIT 
offset_start, row_count /*结果限定*/
~~~

**使用表说明**

![1611968406117](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/30/090007-661373.png)

#### 基础查询

**语法**

~~~sql
/*
语法：
select 查询列表 from 表名;


类似于：System.out.println(打印东西);

特点：

1、查询列表可以是：表中的字段、常量值、表达式、函数,也可以是以上的组合
2、查询的结果是一个虚拟的表格
*/
-- 注：f12可以格式化代码

-- 也可以为字段起别名
select 字段名 as 别名 from 表名
~~~

**简单查询**

~~~ sql
USE myemployees;

--1.查询表中的单个字段

SELECT last_name FROM employees;

--2.查询表中的多个字段
SELECT last_name,salary,email FROM employees;

--3.查询表中的所有字段

--方式一：
SELECT 
    `employee_id`,
    `first_name`,
    `last_name`,
    `phone_number`,
    `last_name`,
    `job_id`,
    `phone_number`,
    `job_id`,
    `salary`,
    `commission_pct`,
    `manager_id`,
    `department_id`,
    `hiredate` 
FROM
    employees ;
--方式二：  
 SELECT * FROM employees;
 
--4.查询常量值
 SELECT 100;
 SELECT 'john';
 
--5.查询表达式
 SELECT 100%98;
 
--6.查询函数
 
 SELECT VERSION();
 
--7.起别名，给列起名字
 /*
 ①便于理解
 ②连接查询中，如果要查询的字段有重名的情况，使用别名可以区分开来
 
 */
--方式一：使用as
 SELECT 100*20 AS '结果';
 -- 查询过程中给列重命名
 SELECT last_name AS '性',first_name AS '名' FROM employees;
 
--方式二：使用空格，省略as关键字
SELECT last_name 姓,first_name 名 FROM employees;

--案例：查询salary，显示结果为 out put,别名中有特殊符号，需要添加引号
SELECT salary AS "out put" FROM employees;

--8.去重

--案例：查询员工表中涉及到的所有的部门编号
SELECT DISTINCT department_id FROM employees;

--9.+号的作用

/*
java中的+号：
①运算符，两个操作数都为数值型
②连接符，只要有一个操作数为字符串,结果就是字符串的拼接

mysql中的+号：
仅仅只有一个功能：运算符

select 100+90; 两个操作数都为数值型，则做加法运算
select '123'+90;只要其中一方为字符型，试图将字符型数值转换成数值型
			如果转换成功，则继续做加法运算
select 'john'+90;	如果转换失败，则将字符型数值转换成0

select null+10; 只要其中一方为null，则结果肯定为null

*/

-- 小结：如果是一个整数值和一个字符串类型数值相加，字符串转数值型成功的话，则是数值相加，如果转换失败，那么返回结果是null

--案例：查询员工名和姓连接成一个字段，并显示为 姓名

SELECT CONCAT('a','b','c') AS 结果;

SELECT 
	CONCAT(last_name,first_name) AS 姓名
FROM
	employees;
 -- 下面结果转换失败，结果全部是0	
SELECT last_name+first_name FROM employees;	
~~~

**练习**

~~~ sql
--1.	下面的语句是否可以执行成功  正确
SELECT last_name , job_id , salary AS sal
FROM employees; 

--2.下面的语句是否可以执行成功  正确
SELECT  *  FROM employees; 


--3.找出下面语句中的错误 中文状态下的逗号，中文状态下的双引号
SELECT employee_id , last_name,
salary * 12 AS "ANNUAL  SALARY"
FROM employees;

--4.显示表departments的结构，并查询其中的全部数据

DESC departments;
SELECT * FROM `departments`;

--5.显示出表employees中的全部job_id（不能重复）
SELECT DISTINCT job_id FROM employees;

--6.显示出表employees的全部列，各个列之间用逗号连接，列头显示成OUT_PUT
-- commission_pct列为null的话，就把null替换为0
SELECT 
	IFNULL(commission_pct,0) AS 奖金率,
	commission_pct
FROM 
	employees;
	
---------------------------------------------

SELECT
	CONCAT(`first_name`,',',`last_name`,',',`job_id`,',',IFNULL(commission_pct,0)) AS out_put
FROM
	employees;
~~~

#### 条件查询

**语法**

~~~ java
/*
语法：
	select 
		查询列表
	from
		表名
	where
		筛选条件;

分类：条件的分类
	一、按条件表达式筛选
	
		简单条件运算符：> < = != <> >= <=
	
	二、按逻辑表达式筛选
	逻辑运算符：
	作用：用于连接条件表达式
		java:&& || !
		mysql:and or not
		
	&&和and：两个条件都为true，结果为true，反之为false
	||或or： 只要有一个条件为true，结果为true，反之为false
	!或not： 如果连接的条件本身为false，结果为true，反之为false
	
	三、模糊查询
		like，一般和通配符一起使用，_或者是%
		between and：判断是否在某一个区间
		in：用于判断某一字段是否在指定的列表
		is null：判断某一个字段是否是null值
*/
~~~

**案例演示---条件表达式**

~~~ sql
--一、按条件表达式筛选

--案例1：查询工资>12000的员工信息

SELECT 
	*
FROM
	employees
WHERE
	salary>12000;

--案例2：查询部门编号不等于90号的员工名和部门编号,建议不等号使用<>,而不是!=
SELECT 
	last_name,
	department_id
FROM
	employees
WHERE
	department_id<>90;

--二、按逻辑表达式筛选

--案例1：查询工资z在10000到20000之间的员工名、工资以及奖金
SELECT
	last_name,
	salary,
	commission_pct
FROM
	employees
WHERE
	salary>=10000 AND salary<=20000;
--案例2：查询部门编号不是在90到110之间，或者工资高于15000的员工信息
SELECT
	*
FROM
	employees
WHERE
	NOT(department_id>=90 AND  department_id<=110) OR salary>15000;
~~~

**模糊查询-like**

~~~ sql
/*
like
between and
in
is null|is not null
*/

-- 1.like
/*
特点：
①一般和通配符搭配使用
	通配符：
	% 任意多个字符,包含0个字符
	_ 任意单个字符
*/
--查询名字中包含字符a的员工
SELECT 
	*
FROM
	employees
WHERE
	last_name LIKE '%a%'(名字中包含a字符);

-- 案例2：查询员工名中第三个字符为e，第五个字符为a的员工名和工资
SELECT
	last_name,
	salary
FROM
	employees
WHERE
	last_name LIKE '__n_l%';

--案例3：查询员工名中第二个字符为_的员工名

SELECT
	last_name
FROM
	employees
WHERE
	last_name LIKE '_$_%' ESCAPE '$';-- 转义字符，可以随意指定转义字符使用的符号

~~~

**模糊查询-between-and**

~~~ sql
#2.between and
/*
①使用between and 可以提高语句的简洁度
②包含临界值
③两个临界值不要调换顺序,有顺序关系

*/
--案例1：查询员工编号在100到120之间的员工信息

SELECT
	*
FROM
	employees
WHERE
	employee_id >= 120 AND employee_id<=100;
#----------------------
SELECT
	*
FROM
	employees
WHERE
	employee_id BETWEEN 120 AND 100;

~~~

**模糊查询-in**

~~~ sql
#3.in
/*
含义：判断某字段的值是否属于in列表中的某一项
特点：
	①使用in提高语句简洁度
	②in列表的值类型必须一致或兼容
	③in列表中不支持通配符
*/
--案例：查询员工的工种编号是 IT_PROG、AD_VP、AD_PRES中的一个员工名和工种编号

SELECT
	last_name,
	job_id
FROM
	employees
WHERE
	job_id = 'IT_PROT' OR job_id = 'AD_VP' OR JOB_ID ='AD_PRES';


#------------------

SELECT
	last_name,
	job_id
FROM
	employees
WHERE
	job_id IN( 'IT_PROT' ,'AD_VP','AD_PRES');-- in后面不支持通配符
~~~

**模糊查询-is null**

~~~ sql
-- 4、is null
/*
=或<>不能用于判断null值
is null或is not null 可以判断null值
*/
--案例1：查询没有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct IS NULL;
-- =号运算符不可以判断null值

--案例1：查询有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct IS NOT NULL;-- is 就是和null进行搭配

-----------以下为×，下面这样写错误
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE 
	salary IS 12000;
	
~~~

**安全等于<=>**

既可以判断null值，也可以判断普通类型的值

~~~ sql
--案例1：查询没有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct <=>NULL;
	
	
--案例2：查询工资为12000的员工信息
SELECT
	last_name,
	salary
FROM
	employees

WHERE 
	salary <=> 12000;
	

--is null 和 <=>

IS NULL:仅仅可以判断NULL值，可读性较高，建议使用
<=>    :既可以判断NULL值，又可以判断普通的数值，可读性较低
~~~

#### 排序查询

**语法**

~~~ sql
/*
语法：
select 查询列表
from 表名
【where  筛选条件】
order by 排序的字段或表达式;


特点：
1、asc代表的是升序，可以省略
desc代表的是降序

2、order by子句可以支持 单个字段、别名、表达式、函数、多个字段

3、order by子句在查询语句的最后面，除了limit子句

select 查询列表from 表名 where 查询条件 order by 排序依据(sec，desc)
排序字段可以是：单个字段，多个字段，表达式，函数，列的索引，别名，以及以上的组合。

注意:order by子句只能对最终的查询结果进行排序，不可以用在select的子查询中。
*/
~~~

**案例**

~~~ sql
--1、按单个字段排序,默认是从低到高
SELECT * FROM employees ORDER BY salary DESC;--desc表示降序排列

--2、添加筛选条件再排序

--案例：查询部门编号>=90的员工信息，并按员工编号降序

SELECT *
FROM employees
WHERE department_id>=90
ORDER BY employee_id DESC;

--3、按表达式排序
--案例：查询员工信息 按年薪降序
--年薪是表达式计算出来的


SELECT *,salary*12*(1+IFNULL(commission_pct,0))
FROM employees
ORDER BY salary*12*(1+IFNULL(commission_pct,0)) DESC;

--4、按别名排序
--案例：查询员工信息 按年薪升序

SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪
FROM employees
ORDER BY 年薪 ASC;

--5、按函数排序
--案例：查询员工名，并且按名字的长度降序

SELECT LENGTH(last_name),last_name 
FROM employees
ORDER BY LENGTH(last_name) DESC;

--6、按多个字段排序

--案例：查询员工信息，要求先按工资降序，再按employee_id升序
SELECT *
FROM employees
ORDER BY salary DESC,employee_id ASC;
~~~

**练习**

~~~ sql
---1.查询员工的姓名和部门号和年薪，按年薪降序 按姓名升序

SELECT last_name,department_id,salary*12*(1+IFNULL(commission_pct,0)) 年薪
FROM employees
ORDER BY 年薪 DESC,last_name ASC;


--2.选择工资不在8000到17000的员工的姓名和工资，按工资降序
SELECT last_name,salary
FROM employees

WHERE salary NOT BETWEEN 8000 AND 17000
ORDER BY salary DESC;

--3.查询邮箱中包含e的员工信息，并先按邮箱的字节数降序，再按部门号升序

SELECT *,LENGTH(email)
FROM employees
WHERE email LIKE '%e%'
ORDER BY LENGTH(email) DESC,department_id ASC;
~~~

#### 常见函数学习

~~~ sql
/*

概念：类似于java的方法，将一组逻辑语句封装在方法体中，对外暴露方法名
好处：1、隐藏了实现细节  2、提高代码的重用性
调用：select 函数名(实参列表) 【from 表】;当函数中的参数用到表中的字段时候，需要添加from 表
特点：
	①叫什么（函数名）
	②干什么（函数功能）

分类：
	1、单行函数
	如 concat、length、ifnull等
	2、分组函数
	
	功能：做统计使用，又称为统计函数、聚合函数、组函数,输入一组数据，返回一个函数
	
常见函数：
	一、单行函数
	字符函数：
	length:获取字节个数(utf-8一个汉字代表3个字节,gbk为2个字节)
	concat
	substr
	instr
	trim
	upper
	lower
	lpad
	rpad
	replace
	
	数学函数：
	round
	ceil
	floor
	truncate
	mod
	
	日期函数：
	now
	curdate
	curtime
	year
	month
	monthname
	day
	hour
	minute
	second
	str_to_date
	date_format
	
	其他函数：
	version
	database
	user
	
	控制函数
	if
	case
	
	version() 查询mysql版本函数
	SELECT DATABASE()：查询当前使用的是哪一个数据库
	SELECT USER():查询当前的用户和主机
	SELECT LENGTH('aa'):查询当前字段或者字符占的字节长度
*/
~~~

##### 字符串函数

- 字符串函数

~~~sql
--1.lth 获取参数值的字节个数
SELECT LENGTH('john');
SELECT LENGTH('张三丰hahaha');


SHOW VARIABLES LIKE '%char%'

--2.concat 拼接字符串

SELECT CONCAT(last_name,'_',first_name) 姓名 FROM employees;

--3.upper、lower
SELECT UPPER('john');
SELECT LOWER('joHn');

--示例：将姓变大写，名变小写，然后拼接，函数之间可以嵌套使用
SELECT CONCAT(UPPER(last_name),LOWER(first_name))  姓名 FROM employees;

--4.substr、substring
注意：索引从1开始
--截取从指定索引处后面所有字符,sql中索引是从1开始
SELECT SUBSTR('李莫愁爱上了陆展元',7)  out_put;

--截取从指定索引处指定字符长度的字符，双闭区间，指的是字符长度，length（）求的是字节长度
SELECT SUBSTR('李莫愁爱上了陆展元',1,3) out_put;


--案例：姓名中首字符大写，其他字符小写然后用_拼接，显示出来
-- 截取首字符（1,1）
SELECT CONCAT(UPPER(SUBSTR(last_name,1,1)),'_',LOWER(SUBSTR(last_name,2)))  out_put
FROM employees;

--5.instr 返回子串第一次出现的索引，如果找不到返回0

SELECT INSTR('杨不殷六侠悔爱上了殷六侠','殷六侠')AS out_put;

--6.trim
-- 去掉字符串的前后空格
select trim('   dcusbjh   ');

SELECT LENGTH(TRIM('    张翠山    ')) AS out_put;
-- 去掉aa字符，会把aa当做一个整体，trim函数只是去掉的是前后所指定的字符
SELECT TRIM('aa' FROM 'aaaaaaaaa张aaaaaaaaaaaa翠山aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa')  AS out_put;

--7.lpad 用指定的字符实现左填充指定长度，长度超过的话会进行截取操作，截断会从右边

SELECT LPAD('殷素素',5,'*') AS out_put;

--8.rpad 用指定的字符实现右填充指定长度

SELECT RPAD('殷素素',12,'ab') AS out_put;

--9.replace 替换

SELECT REPLACE('周芷若周芷若周芷若周芷若张无忌爱上了周芷若','周芷若','赵敏') AS out_put;

--CHAR_LENGTH 获取字符个数
SELECT CHAR_LENGTH('hello,郭襄');

--SUBSTRING 截取子串
/*
注意：起始索引从1开始！！！
substr(str,起始索引，截取的字符长度)
substr(str,起始索引)
*/
SELECT SUBSTR('张三丰爱上了郭襄',1,3);
SELECT SUBSTR('张三丰爱上了郭襄',7);

--STRCMP 比较两个字符大小
SELECT STRCMP('aec','aec');

~~~

##### 数学函数

~~~ sql
--round 四舍五入
SELECT ROUND(-1.55);
-- 第二个参数表示保留小数位数
SELECT ROUND(1.567,2);

--ceil 向上取整,返回>=该参数的最小整数

SELECT CEIL(-1.02);

--floor 向下取整，返回<=该参数的最大整数
SELECT FLOOR(-9.99);

--truncate 截断，第二个参数表示小数点后保留的位数

SELECT TRUNCATE(1.69999,1);

--mod取余
/*
mod(a,b) ：  a-a/b*b
被除数是正数，结果就是正数
mod(-10,-3):-10- (-10)/(-3)*（-3）=-1
*/
SELECT MOD(10,-3);
SELECT 10%3;
~~~

##### 日期函数

~~~sql
--now 返回当前系统旟+时间
SELECT NOW();

--curdate 返回当前系统日期，不包含时间
SELECT CURDATE();

--curtime 返回当前时间，不包含日期
SELECT CURTIME();

--可以获取指定的部分，年、月、日、小时、分钟、秒
SELECT YEAR(NOW()) 年;
SELECT YEAR('1998-1-1') 年;
SELECT  YEAR(hiredate) 年 FROM employees;

SELECT MONTH(NOW()) 月;
SELECT MONTHNAME(NOW()) 月;


--str_to_date 将字符通过指定的格式转换成日期

SELECT STR_TO_DATE('1998-3-2','%Y-%c-%d') AS out_put;

--查询入职日期为1992--4-3的员工信息
SELECT * FROM employees WHERE hiredate = '1992-4-3';

SELECT * FROM employees WHERE hiredate = STR_TO_DATE('4-3 1992','%c-%d %Y');


--date_format 将日期转换成字符

SELECT DATE_FORMAT(NOW(),'%Y年%m月%d日') AS out_put;

--查询有奖金的员工名和入职日期(xx月/xx日 xx年)
SELECT last_name,DATE_FORMAT(hiredate,'%m月/%d日 %Y年') 入职日期
FROM employees
WHERE commission_pct IS NOT NULL;
~~~

**补充**

![1612145028984](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/100350-765575.png)

![1614065044378](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614065044378.png)

##### 其他函数

~~~ sql
--四、其他函数
SELECT VERSION();
SELECT DATABASE();
SELECT USER();
~~~

##### 流程控制函数

**if 函数**

~~~sql
--五、流程控制函数
--1.if函数： if else 的效果

SELECT IF(10<5,'大','小');

SELECT last_name,commission_pct,IF(commission_pct IS NULL,'没奖金，呵呵','有奖金，嘻嘻') 备注
FROM employees;

~~~

**case函数**

~~~ sql
--2.case函数的使用一： switch case 的效果

/*
java中
switch(变量或表达式){
	case 常量1：语句1;break;
	...
	default:语句n;break;

}

mysql中

case 要判断的字段或表达式
when 常量1 then 要显示的值1或语句1;
when 常量2 then 要显示的值2或语句2;
...
else 要显示的值n或语句n;
end
*/
*/

/*案例：查询员工的工资，要求

部门号=30，显示的工资为1.1倍
部门号=40，显示的工资为1.2倍
部门号=50，显示的工资为1.3倍
其他部门，显示的工资为原工资

*/

-- CASE既可以搭配select使用，也可以单独作为表达式使用，作为表达式使用then后面是显示的值
SELECT salary 原始工资,department_id,
CASE department_id
WHEN 30 THEN salary*1.1-- then后面是值
WHEN 40 THEN salary*1.2
WHEN 50 THEN salary*1.3
ELSE salarEND AS 新工资
FROM employees;

#3.case 函数的使用二：类似于 多重if
/*
java中：
if(条件1){
	语句1；
}else if(条件2){
	语句2；
}
...
else{
	语句n;
}

mysql中：

case 
when 条件1 then 要显示的值1或语句1 如果这里是语句，那么就要添加;，如果是值，不用添加分号
when 条件2 then 要显示的值2或语句2
。。。
else 要显示的值n或语句n
end
*/

--案例：查询员工的工资的情况
如果工资>20000,显示A级别
如果工资>15000,显示B级别
如果工资>10000，显示C级别
否则，显示D级别

SELECT salary,
CASE 
WHEN salary>20000 THEN 'A'
WHEN salary>15000 THEN 'B'
WHEN salary>10000 THEN 'C'
ELSE 'D'
END AS 工资级别
FROM employees;

~~~

##### 练习

~~~ sql
--1.	显示系统时间(注：日期+时间)
SELECT NOW();

--2.	查询员工号，姓名，工资，以及工资提高百分之20%后的结果（new salary）

SELECT employee_id,last_name,salary,salary*1.2 "new salary"
FROM employees;
--3.	将员工的姓名按首字母排序，并写出姓名的长度（length）

SELECT LENGTH(last_name) 长度,SUBSTR(last_name,1,1) 首字符,last_name
FROM employees
ORDER BY 首字符;

--4.	做一个查询，产生下面的结果
<last_name> earns <salary> monthly but wants <salary*3>
Dream Salary
King earns 24000 monthly but wants 72000

SELECT CONCAT(last_name,' earns ',salary,' monthly but wants ',salary*3) AS "Dream Salary"
FROM employees
WHERE salary=24000;

--5.	使用case-when，按照下面的条件：
job                  grade
AD_PRES            A
ST_MAN             B
IT_PROG             C
SA_REP              D
ST_CLERK           E
--产生下面的结果
Last_name	Job_id	Grade
king	AD_PRES	A

SELECT last_name,job_id AS  job,
CASE job_id
WHEN 'AD_PRES' THEN 'A' 
WHEN 'ST_MAN' THEN 'B' 
WHEN 'IT_PROG' THEN 'C' 
WHEN 'SA_PRE' THEN 'D'
WHEN 'ST_CLERK' THEN 'E'
END AS Grade
FROM employees
WHERE job_id = 'AD_PRES';
~~~

##### 分组函数一

~~~ sql
#二、分组函数
/*
功能：用作统计使用，又称为聚合函数或统计函数或组函数

分类：
sum 求和、avg 平均值、max 最大值 、min 最小值 、count 计算个数

特点：
1、sum、avg一般用于处理数值型
   max、min、count可以处理任何类型
2、以上分组函数都忽略null值

3、可以和distinct搭配实现去重的运算

4、count函数的单独介绍
一般使用count(*)用作统计行数

5、和分组函数一同查询的字段要求是group by后的字段

*/
~~~

**案例**

~~~ sql
#二、分组函数
/*
功能：用作统计使用，又称为聚合函数或统计函数或组函数

分类：
sum 求和、avg 平均值、max 最大值 、min 最小值 、count 计算个数

特点：
1、sum、avg一般用于处理数值型
   max、min、count可以处理任何类型
2、以上分组函数都忽略null值

3、可以和distinct搭配实现去重的运算

4、count函数的单独介绍
一般使用count(*)用作统计行数

5、和分组函数一同查询的字段要求是group by后的字段

*/

--1、简单 的使用
SELECT SUM(salary) FROM employees;
SELECT AVG(salary) FROM employees;
SELECT MIN(salary) FROM employees;
SELECT MAX(salary) FROM employees;
SELECT COUNT(salary) FROM employees;


SELECT SUM(salary) 和,AVG(salary) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数
FROM employees;

SELECT SUM(salary) 和,ROUND(AVG(salary),2) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数
FROM employees;

--2、参数支持哪些类型
-- 不支持字符串类型
SELECT SUM(last_name) ,AVG(last_name) FROM employees;
-- 求出的结果没有什么意义
SELECT SUM(hiredate) ,AVG(hiredate) FROM employees;

SELECT MAX(last_name),MIN(last_name) FROM employees;

SELECT MAX(hiredate),MIN(hiredate) FROM employees;

-- 计算某一列值不是null的个数 
SELECT COUNT(commission_pct) FROM employees;
SELECT COUNT(last_name) FROM employees;

--3、是否忽略null，也就是说sum()函数和avg()函数是忽略null值得

SELECT SUM(commission_pct) ,AVG(commission_pct),SUM(commission_pct)/35,SUM(commission_pct)/107 FROM employees;
-- max min 也都忽略null
SELECT MAX(commission_pct) ,MIN(commission_pct) FROM employees;
 -- count()计算非空值得个数
SELECT COUNT(commission_pct) FROM employees;
SELECT commission_pct FROM employees;
--4、和distinct搭配

SELECT SUM(DISTINCT salary),SUM(salary) FROM employees;

SELECT COUNT(DISTINCT salary),COUNT(salary) FROM employees;

--5、count函数的详细介绍

SELECT COUNT(salary) FROM employees;


SELECT COUNT(*) FROM employees;

SELECT COUNT(1) FROM employees;

--效率：
--MYISAM存储引擎下  ，COUNT(*)的效率高 -- 内部维护一个计数器，所以可以直接返回个数
--INNODB存储引擎下，COUNT(*)和COUNT(1)的效率差不多，比COUNT(字段)要高一些 -- 

--6、和分组函数一同查询的字段有限制
-- 这样写没有意义，聚合函数返回一个值，而employee_id返回多个值，没有意义

SELECT AVG(salary),employee_id  FROM employees;

~~~

**案例**

~~~ sql
#1.查询公司员工工资的最大值，最小值，平均值，总和

SELECT MAX(salary) 最大值,MIN(salary) 最小值,AVG(salary) 平均值,SUM(salary) 和
FROM employees;
#2.查询员工表中的最大入职时间和最小入职时间的相差天数 （DIFFRENCE）

SELECT MAX(hiredate) 最大,MIN(hiredate) 最小,(MAX(hiredate)-MIN(hiredate))/1000/3600/24 DIFFRENCE
FROM employees;

SELECT DATEDIFF(MAX(hiredate),MIN(hiredate)) DIFFRENCE
FROM employees;

SELECT DATEDIFF('1995-2-7','1995-2-6');

#3.查询部门编号为90的员工个数

SELECT COUNT(*) FROM employees WHERE department_id = 90;
~~~

##### 分组查询二

**语法**

~~~ sql
/*
语法：

select 查询列表
from 表
【where 筛选条件】
group by 分组的字段
【order by 排序的字段】;

特点：
1、和分组函数一同查询的字段必须是group by后出现的字段
2、筛选分为两类：分组前筛选和分组后筛选
		针对的表			位置		连接的关键字
分组前筛选	原始表				group by前	where
	
分组后筛选	group by后的结果集    		group by后	having

问题1：分组函数做筛选能不能放在where后面
答：不能

问题2：where——group by——having

一般来讲，能用分组前筛选的，尽量使用分组前筛选，提高效率

3、分组可以按单个字段也可以按多个字段
4、可以搭配着排序使用,多个字段之间没有顺序要求，也可以添加排序，排序是放在最后的。
*/
~~~

**案例**

~~~sql
USE `myemployees`;
--引入：查询每个部门的员工个数

SELECT COUNT(*) FROM employees WHERE department_id=90;

-- 查询每一个工种的最高工资
SELECT MAX(salary),job_id
FROM
	employees
GROUP BY
	job_id;

#1.简单的分组

#案例1：查询每个工种的员工平均工资
SELECT AVG(salary),job_id
FROM employees
GROUP BY job_id;

#案例2：查询每个位置的部门个数

SELECT COUNT(*),location_id
FROM departments
GROUP BY location_id;

#2、可以实现分组前的筛选

#案例1：查询邮箱中包含a字符的 每个部门的最高工资

SELECT MAX(salary),department_id
FROM employees
WHERE email LIKE '%a%'
GROUP BY department_id;

#案例2：查询有奖金的每个领导手下员工的平均工资

SELECT AVG(salary),manager_id
FROM employees
WHERE commission_pct IS NOT NULL
GROUP BY manager_id;

#3、分组后筛选

#案例：查询哪个部门的员工个数>5

#①查询每个部门的员工个数
SELECT COUNT(*),department_id
FROM employees
GROUP BY department_id;

#② 筛选刚才①结果

SELECT COUNT(*),department_id
FROM employees

GROUP BY department_id

HAVING COUNT(*)>5;

#案例2：每个工种有奖金的员工的最高工资>12000的工种编号和最高工资

SELECT job_id,MAX(salary)
FROM employees
WHERE commission_pct IS NOT NULL
GROUP BY job_id
HAVING MAX(salary)>12000;

#案例3：领导编号>102的每个领导手下的最低工资大于5000的领导编号和最低工资

manager_id>102

SELECT manager_id,MIN(salary)
FROM employees
GROUP BY manager_id
HAVING MIN(salary)>5000;

#4.添加排序

#案例：每个工种有奖金的员工的最高工资>6000的工种编号和最高工资,按最高工资升序

SELECT job_id,MAX(salary) m
FROM employees
WHERE commission_pct IS NOT NULL
GROUP BY job_id
HAVING m>6000
ORDER BY m ;

#5.按多个字段分组

#案例：查询每个工种每个部门的最低工资,并按最低工资降序

SELECT MIN(salary),job_id,department_id
FROM employees
GROUP BY department_id,job_id
ORDER BY MIN(salary) DESC;

~~~

##### 练习

~~~sql
--1.查询各job_id的员工工资的最大值，最小值，平均值，总和，并按job_id升序

SELECT MAX(salary),MIN(salary),AVG(salary),SUM(salary),job_id
FROM employees
GROUP BY job_id
ORDER BY job_id;

--2.查询员工最高工资和最低工资的差距（DIFFERENCE）
SELECT MAX(salary)-MIN(salary) DIFFRENCE
FROM employees;

--3.查询各个管理者手下员工的最低工资，其中最低工资不能低于6000，没有管理者的员工不计算在内
SELECT MIN(salary),manager_id
FROM employees
WHERE manager_id IS NOT NULL
GROUP BY manager_id
HAVING MIN(salary)>=6000;

--4.查询所有部门的编号，员工数量和工资平均值,并按平均工资降序
SELECT department_id,COUNT(*),AVG(salary) a
FROM employees
GROUP BY department_id
ORDER BY a DESC;

--5.选择具有各个job_id的员工人数
SELECT COUNT(*) 个数,job_id
FROM employees
GROUP BY job_id;
~~~

**小结**

~~~ sql
--分组函数还可以搭配distinct（）一起使用实现去重
Select count(distinct 字段名)from 表。
--分组查询：
Select 分组函数，分组的字段
From 表名
Where 分组前的筛选条件
Group by 分组列表
Having 分组后的筛选条件
Order by 排序列表
--分组列表可以是单个字段和多个字段
--筛选条件分为两类:
--分组后的筛选。先分组，后筛选，
--如果条件放在where子句中，就是分组前的筛选，筛选是基于原始表筛选
--如果条件放在having子句中，那么就是分组后的筛选，筛选是基于分组后的筛选
			筛选的基表		使用关键词		关键词位置
分组前的筛选		原始表			where子句		group by前面
分组后的筛选		分组后的表		having 子句		group by 后面

~~~

#### 链接查询

**语法**

~~~ sql
连接查询
/*
含义：又称多表查询，当查询的字段来自于多个表时，就会用到连接查询

笛卡尔乘积现象：表1 有m行，表2有n行，结果=m*n行

发生原因：没有有效的连接条件
如何避免：添加有效的连接条件

分类：

	按年代分类：
	sql92标准:仅仅支持内连接
	sql99标准【推荐】：支持内连接+外连接（左外和右外）+交叉连接
	
	按功能分类：
		内连接：
			等值连接
			非等值连接
			自连接
		外连接：
			左外连接
			右外连接
			全外连接
		
		交叉连接

*/
~~~

**案例**

~~~ sql
SELECT * FROM beauty;

SELECT * FROM boys;
SELECT NAME,boyName FROM boys,beauty
WHERE beauty.boyfriend_id= boys.id;

#一、sql92标准
#1、等值连接
/*

① 多表等值连接的结果为多表的交集部分
②n表连接，至少需要n-1个连接条件
③ 多表的顺序没有要求
④一般需要为表起别名
⑤可以搭配前面介绍的所有子句使用，比如排序、分组、筛选


*/

#案例1：查询女神名和对应的男神名
SELECT NAME,boyName 
FROM boys,beauty
WHERE beauty.boyfriend_id= boys.id;

#案例2：查询员工名和对应的部门名

SELECT last_name,department_name
FROM employees,departments
WHERE employees.`department_id`=departments.`department_id`;

#2、为表起别名
/*
①提高语句的简洁度
②区分多个重名的字段

注意：如果为表起了别名，则查询的字段就不能使用原来的表名去限定

*/
#查询员工名、工种号、工种名

SELECT e.last_name,e.job_id,j.job_title
FROM employees  e,jobs j
WHERE e.`job_id`=j.`job_id`;

#3、两个表的顺序是否可以调换

#查询员工名、工种号、工种名

SELECT e.last_name,e.job_id,j.job_title
FROM jobs j,employees e
WHERE e.`job_id`=j.`job_id`;

#4、可以加筛选

#案例：查询有奖金的员工名、部门名

SELECT last_name,department_name,commission_pct

FROM employees e,departments d
WHERE e.`department_id`=d.`department_id`
AND e.`commission_pct` IS NOT NULL;

#案例2：查询城市名中第二个字符为o的部门名和城市名

SELECT department_name,city
FROM departments d,locations l
WHERE d.`location_id` = l.`location_id`
AND city LIKE '_o%';

#5、可以加分组

#案例1：查询每个城市的部门个数

SELECT COUNT(*) 个数,city
FROM departments d,locations l
WHERE d.`location_id`=l.`location_id`
GROUP BY city;

#案例2：查询有奖金的每个部门的部门名和部门的领导编号和该部门的最低工资
SELECT department_name,d.`manager_id`,MIN(salary)
FROM departments d,employees e
WHERE d.`department_id`=e.`department_id`
AND commission_pct IS NOT NULL
GROUP BY department_name,d.`manager_id`;
#6、可以加排序

#案例：查询每个工种的工种名和员工的个数，并且按员工个数降序

SELECT job_title,COUNT(*)
FROM employees e,jobs j
WHERE e.`job_id`=j.`job_id`
GROUP BY job_title
ORDER BY COUNT(*) DESC;

#7、可以实现三表连接？

#案例：查询员工名、部门名和所在的城市

SELECT last_name,department_name,city
FROM employees e,departments d,locations l
WHERE e.`department_id`=d.`department_id`
AND d.`location_id`=l.`location_id`
AND city LIKE 's%'

ORDER BY department_name DESC;

#2、非等值连接

#案例1：查询员工的工资和工资级别

SELECT salary,grade_level
FROM employees e,job_grades g
WHERE salary BETWEEN g.`lowest_sal` AND g.`highest_sal`
AND g.`grade_level`='A';

/*
select salary,employee_id from employees;
select * from job_grades;
CREATE TABLE job_grades
(grade_level VARCHAR(3),
 lowest_sal  int,
 highest_sal int);

INSERT INTO job_grades
VALUES ('A', 1000, 2999);

INSERT INTO job_grades
VALUES ('B', 3000, 5999);

INSERT INTO job_grades
VALUES('C', 6000, 9999);

INSERT INTO job_grades
VALUES('D', 10000, 14999);

INSERT INTO job_grades
VALUES('E', 15000, 24999);

INSERT INTO job_grades
VALUES('F', 25000, 40000);

*/

#3、自连接

#案例：查询 员工名和上级的名称

SELECT e.employee_id,e.last_name,m.employee_id,m.last_name
FROM employees e,employees m
WHERE e.`manager_id`=m.`employee_id`;
~~~

**补充**

~~~ sql
---------------------------------------sql92语法-----------------------------------------
(1)	Select 查询列表
(2)	From 表1 别名1，表二 别明二
(3)	Where 连接条件 
(4)	and 筛选条件
(5)	Group by分组列表
(6)	Having 分组后的筛选
(7)	Order by  排序列表；
--执行顺序：2-3-4-5-6-1-7
内连接：
---------------------------------------sql99语法-----------------------------------------
1)	Select 筛选条件
2)	From 表一  别名一 
3)	Inner join 表二 别名二 On 连接条件
Inner join 表二 别名二 On 连接条件
Inner join 表二 别名二 On 连接条件
...........//在这可以添加多个表的连接
4)	Where 筛选条件一
5)	Group by 分组列表
6)	Having 分组后的筛选条件
7)	Order by 排序列表
执行顺序：2-3-4-5-6-7-1

--外连接：
--查询结果为主表中所有的记录，如果从表中有与之相匹配的记录，那么显示匹配记录，如果从表中没有匹配记录，那么显示为null.
--一般用于主表中有，从表中没有的记录。
--特点：外连接分主从表，顺序不能交换。
--左连接：左边的是主表。
--右连接：右边为主表。
语法：
  Select 查询列表
  From 表一 别名一
  Left/right /full outer join 表二 别名二
  Where 筛选条件
  Group by 分组列表
  Having 筛选条件
  Order by 排序列表；
  full代表全外连接
~~~

#### sql99语法

**语法**

~~~ sql
/*
语法：
	select 查询列表
	from 表1 别名 【连接类型】
	join 表2 别名 
	on 连接条件
	【where 筛选条件】
	【group by 分组】
	【having 筛选条件】
	【order by 排序列表】
	
分类：
内连接（★）：inner
	分类：
    等值
    非等值
    自连接
外连接
	左外(★):left 【outer】
	右外(★)：right 【outer】
	全外：full【outer】
交叉连接：cross 

*/
~~~

**内连接**

~~~ sql
#一）内连接
/*
语法：

select 查询列表
from 表1 别名
inner join 表2 别名
on 连接条件;

分类：
  等值
  非等值
  自连接

特点：
①添加排序、分组、筛选
②inner可以省略
③ 筛选条件放在where后面，连接条件放在on后面，提高分离性，便于阅读
④inner join连接和sql92语法中的等值连接效果是一样的，都是查询多表的交集
*/
~~~

**内连接案例**

~~~ sql
#1、等值连接
#案例1.查询员工名、部门名

SELECT last_name,department_name
FROM departments d
 JOIN  employees e
ON e.`department_id` = d.`department_id`;

#案例2.查询名字中包含e的员工名和工种名（添加筛选）
SELECT last_name,job_title
FROM employees e
INNER JOIN jobs j
ON e.`job_id`=  j.`job_id`
WHERE e.`last_name` LIKE '%e%';

#3. 查询部门个数>3的城市名和部门个数，（添加分组+筛选）

#①查询每个城市的部门个数
#②在①结果上筛选满足条件的
SELECT city,COUNT(*) 部门个数
FROM departments d
INNER JOIN locations l
ON d.`location_id`=l.`location_id`
GROUP BY city
HAVING COUNT(*)>3;

#案例4.查询哪个部门的员工个数>3的部门名和员工个数，并按个数降序（添加排序）

#①查询每个部门的员工个数
SELECT COUNT(*),department_name
FROM employees e
INNER JOIN departments d
ON e.`department_id`=d.`department_id`
GROUP BY department_name

#② 在①结果上筛选员工个数>3的记录，并排序

SELECT COUNT(*) 个数,department_name
FROM employees e
INNER JOIN departments d
ON e.`department_id`=d.`department_id`
GROUP BY department_name
HAVING COUNT(*)>3
ORDER BY COUNT(*) DESC;

#5.查询员工名、部门名、工种名，并按部门名降序（添加三表连接）

SELECT last_name,department_name,job_title
FROM employees e
INNER JOIN departments d ON e.`department_id`=d.`department_id`
INNER JOIN jobs j ON e.`job_id` = j.`job_id`

ORDER BY department_name DESC;
~~~

**外连接**

~~~ sql
#二、外连接
 
 /*
 应用场景：用于查询一个表中有，另一个表没有的记录
 
 特点：
 1、外连接的查询结果为主表中的所有记录
	如果从表中有和它匹配的，则显示匹配的值
	如果从表中没有和它匹配的，则显示null
	外连接查询结果=内连接结果+主表中有而从表没有的记录
 2、左外连接，left join左边的是主表
    右外连接，right join右边的是主表
 3、左外和右外交换两个表的顺序，可以实现同样的效果 
 4、全外连接=内连接的结果+表1中有但表2没有的+表2中有但表1没有的
 */
 #引入：查询男朋友 不在男神表的的女神名
 
 SELECT * FROM beauty;
 SELECT * FROM boys;
 
 #左外连接
 SELECT b.*,bo.*
 FROM boys bo
 LEFT OUTER JOIN beauty b
 ON b.`boyfriend_id` = bo.`id`
 WHERE b.`id` IS NULL;
 
 #案例1：查询哪个部门没有员工
 #左外
 SELECT d.*,e.employee_id
 FROM departments d
 LEFT OUTER JOIN employees e
 ON d.`department_id` = e.`department_id`
 WHERE e.`employee_id` IS NULL;
 
 #右外
 
  SELECT d.*,e.employee_id
 FROM employees e
 RIGHT OUTER JOIN departments d
 ON d.`department_id` = e.`department_id`
 WHERE e.`employee_id` IS NULL;
 
 #全外
 
 USE girls;
 SELECT b.*,bo.*
 FROM beauty b
 FULL OUTER JOIN boys bo
 ON b.`boyfriend_id` = bo.id;

 #交叉连接，交叉连接就是一个笛卡尔乘积
 
 SELECT b.*,bo.*
 FROM beauty b
 CROSS JOIN boys bo;
 
 #sql92和 sql99pk
 /*
 功能：sql99支持的较多
 可读性：sql99实现连接条件和筛选条件的分离，可读性较高
 */
~~~

**连接小结**

![1613909351708](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/21/200913-158039.png)

![1613909468895](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/21/201110-789871.png)

**案例**

~~~ sql
#一、查询编号>3的女神的男朋友信息，如果有则列出详细，如果没有，用null填充

SELECT b.id,b.name,bo.*
FROM beauty b
LEFT OUTER JOIN boys bo
ON b.`boyfriend_id` = bo.`id`
WHERE b.`id`>3;
#二、查询哪个城市没有部门

SELECT city
FROM departments d
RIGHT OUTER JOIN locations l 
ON d.`location_id`=l.`location_id`
WHERE  d.`department_id` IS NULL;

#三、查询部门名为SAL或IT的员工信息

SELECT e.*,d.department_name,d.`department_id`
FROM departments  d
LEFT JOIN employees e
ON d.`department_id` = e.`department_id`
WHERE d.`department_name` IN('SAL','IT');

SELECT * FROM departments
WHERE `department_name` IN('SAL','IT');
~~~

#### 子查询

**嵌套查询**

~~~ sql
1．带有in谓词的子查询：子查询的结果往往是一个集合
select * 
from stu
where sdept
in (
select sdept 
from stu 
where sname='liuchen'
);
--在这里可以用=代替in来查询，应为子查询返回的是一个单值，但是如果刘晨有重名，就会报错，也就是返回不是单值得时候用=会报错。

2,带有any,some,all,的谓词查询。
--当内层查询返回单值得时候，用比较运算符，但是当内层查询返回多值集合时候，要用any,some,all等谓词配合运算符进行运算。

>any:大于子查询结构中的某一个值。（最小值）
>all:大于子查询结果中的所有值。（大于最大值）
<any:小于子查询结果中的某个值，（小于最大）
<all:小于子查询结果中的所有值。（小于最小）
>=any
>=all
<=any
<=all
=any:等于子查询结果中的某个值。（=in）
！=all:不等于子查询结果中的任何一个值。（not in）
=any等价于in谓词，<any等价于<max,<>all等价于not in谓词，<all等价于<min谓词。

3,带有exists谓词的子查询
--带有exists的子查询不返回任何数据，只产生逻辑真true或者逻辑假false。可以利用exists来判断x属于S。

4,集合查询（并UNION，交INTERSECT,差EXCEPT）
--参加集合操作的各个查询结果的列数必须相同，对应列的数据类型也必须相同。

5,基于派生表的查询：子查询不仅可以出现在where子句中，还可以出现在from子句中，这时子查询生成的派生表成为主查询的查询对象。
Select子句的一般格式：
Select [all|distinct]<目标列表达式>[别名].......
From <表明或者视图名>[别名]...........（select 语句）as 别名
where <条件表达式>
Group by <列名1....> having<条件表达式>
Order by <列明1......>[asc|desc]

~~~

- 相关子查询：如果子查询的查询条件依赖于父查询，这类查询称为相关子查询，如果不依赖于父查询，那么称为不相管查询。带有比较符的查询是指父查询与子查询之间用比较运算符进行连接。

- 当用户确切知道内层查询返回单个值时候，可以用>,>=,<,<=,!=或者<>等运算符进行运算。

**语法**

~~~ sql
/*
含义：
出现在其他语句中的select语句，称为子查询或内查询
外部的查询语句，称为主查询或外查询

分类：
按子查询出现的位置：
	select后面：
		仅仅支持标量子查询（子查询结果是单行单列）
	
	from后面：
		支持表子查询（子查询结果是多行多列，子表形式）
	where或having后面：★
		标量子查询（单行） √
		列子查询  （多行） √
		行子查询
		
	exists后面（相关子查询）
		表子查询
按结果集的行列数不同：
	标量子查询（结果集只有一行一列）
	列子查询（结果集只有一列多行）
	行子查询（结果集有一行多列）
	表子查询（结果集一般为多行多列）

特点：子查询放在条件中，必须放在条件的右侧。
子查询一般放在小括号中。
子查询的执行优先于主查询。
单行子查询对应了单行操作符：>,<,<=,>=,<>
多行子查询对应多行操作符：any some all in 

In:判断某字段的值是否在指定的列表里面。X in(30,50,90)
Any/some：判断某字段的值是否满足其中任何一个。
X>any(30,50,90)==x>min(30,50,90)
X=any(30,50,90)==x in(30,50,90)
All:判断某字段的值是否满足里面所有的。例如：x>all(30,50,90)==x>max(30,50,90).

*/

~~~

##### **where或having后面**

~~~ sql
/*
1、标量子查询（单行子查询）
2、列子查询（多行子查询）

3、行子查询（多列多行）

特点：
①子查询放在小括号内
②子查询一般放在条件的右侧
③标量子查询，一般搭配着单行操作符使用
> < >= <= = <>

列子查询，一般搭配着多行操作符使用
in、any/some、all

④子查询的执行优先于主查询执行，主查询的条件用到了子查询的结果

*/
~~~

**标量子查询**

子查询返回的结果是一列一行

~~~sql
#1.标量子查询★

#案例1：谁的工资比 Abel 高?

#①查询Abel的工资,标量子查询，查询的结果只有一行一列
USE `myemployees`
SELECT salary
FROM employees
WHERE last_name = 'Abel'

#②查询员工的信息，满足 salary>①结果，属于不相关子查询
SELECT *
FROM employees
WHERE salary>(

	SELECT salary
	FROM employees
	WHERE last_name = 'Abel'

);

#案例2：返回job_id与141号员工相同，salary比143号员工多的员工 姓名，job_id 和工资

#①查询141号员工的job_id
SELECT job_id
FROM employees
WHERE employee_id = 141

#②查询143号员工的salary
SELECT salary
FROM employees
WHERE employee_id = 143

#③查询员工的姓名，job_id 和工资，要求job_id=①并且salary>②
-- 一个查询中可以放两个子查询

SELECT last_name,job_id,salary
FROM employees
WHERE job_id = (
	SELECT job_id
	FROM employees
	WHERE employee_id = 141
) AND salary>(
	SELECT salary
	FROM employees
	WHERE employee_id = 143

);

#案例3：返回公司工资最少的员工的last_name,job_id和salary

#①查询公司的 最低工资
SELECT MIN(salary)
FROM employees

#②查询last_name,job_id和salary，要求salary=①
SELECT last_name,job_id,salary
FROM employees
WHERE salary=(
	SELECT MIN(salary)
	FROM employees
);

#案例4：查询最低工资大于50号部门最低工资的部门id和其最低工资

#①查询50号部门的最低工资
SELECT  MIN(salary)
FROM employees
WHERE department_id = 50

#②查询每个部门的最低工资

SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id

#③ 在②基础上筛选，满足min(salary)>①
SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id
HAVING MIN(salary)>(
	SELECT  MIN(salary)
	FROM employees
	WHERE department_id = 50

);

#非法使用标量子查询

SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id
HAVING MIN(salary)>(
-- 标量子查询没有搭配单行操作符使用，这里子查询的结果需要是一行一列
	SELECT  salary
	FROM employees
	WHERE department_id = 250

);
~~~

**列子查询**

子查询返回的结果是**一列多行**，下面是与多行子查询搭配的操作符。

![1613989831455](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/23/082139-883412.png)

any/some:表示和返回的结果中的某一个值进行比较

- `>`any可以等价于min()函数，`<`any可以等价于max()函数

all：表示和结果集中所有数据进行比较，`>`all可以等价于`>`max

**案例**

~~~ sql
#2.列子查询（多行子查询）★，放在where 或者having 后面，多列梓查询，查询的结果是一列多行
#案例1：返回location_id是1400或1700的部门中的所有员工姓名

#①查询location_id是1400或1700的部门编号，查询结果是单列多行

USE `myemployees`

SELECT DISTINCT department_id
FROM departments
WHERE location_id IN(1400,1700)

#②查询员工姓名，要求部门号是①列表中的某一个

SELECT last_name
FROM employees
WHERE department_id  <>ALL( --<>是不等于,这里IN可以使用=ANY代替，相当于等于里面的任意一个,NOT IN 可以使用<>ALL代替
	SELECT DISTINCT department_id
	FROM departments
	WHERE location_id IN(1400,1700)

);

SELECT last_name
FROM employees
WHERE department_id  IN(
	SELECT DISTINCT department_id
	FROM departments
	WHERE location_id IN(1400,1700)

);

#案例2：返回其它工种中比job_id为‘IT_PROG’工种任一工资低的员工的员工号、姓名、job_id 以及salary

#①查询job_id为‘IT_PROG’部门任一工资，返回结果是一列多行

SELECT DISTINCT salary
FROM employees
WHERE job_id = 'IT_PROG'

#②查询员工号、姓名、job_id 以及salary，salary<(①)的任意一个
SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<ANY( -- 小于子查询结果中的任意一个值
	SELECT DISTINCT salary
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';

#或
SELECT last_name,employee_id,job_id,salary
FROM employees
WHER salary<(
	SELECT MAX( salary)
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';


#案例3：返回其它部门中比job_id为‘IT_PROG’部门所有工资都低的员工   的员工号、姓名、job_id 以及salary

SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<ALL(
	SELECT DISTINCT salary
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';

#或 小于any可以等价于小于min

SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<(
	SELECT MIN( salary)
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';

#3、行子查询（结果集一行多列或多行多列）也就是说结果是一张表

#案例：查询员工编号最小并且工资最高的员工信息


-- 使用行子查询代替
SELECT * 
FROM employees
WHERE (employee_id,salary)=(-- 多个赛选条件都用==，>,<，也就是多个赛选条件使用的符号一样的时候，可以使用这种语法
	SELECT MIN(employee_id),MAX(salary)
	FROM employees
);

#①查询最小的员工编号
SELECT MIN(employee_id)
FROM employees

#②查询最高工资
SELECT MAX(salary)
FROM employees

#③查询员工信息
SELECT *
FROM employees
WHERE employee_id=(
	SELECT MIN(employee_id)
	FROM employees


)AND salary=(
	SELECT MAX(salary)
	FROM employees

);

~~~

##### select后面

~~~ sql
#二、select后面
/*
仅仅支持标量子查询
*/

#案例：查询每个部门的员工个数

SELECT d.*,(

	SELECT COUNT(*)
	FROM employees e
	WHERE e.department_id = d.`department_id`
 ) 个数 -- 子查询作为结果中的一个结果字段出现，也可以使用join链接查询`employees``employees`
 FROM departments d;
 
 
 #案例2：查询员工号=102的部门名
 
SELECT (
	SELECT department_name,e.department_id
	FROM departments d
	INNER JOIN employees e
	ON d.department_id=e.department_id
	WHERE e.employee_id=102
	
) 部门名;

~~~

##### from后面

~~~ sql
#三、from后面
/*
将子查询结果充当一张表，要求必须起别名
*/

#案例：查询每个部门的平均工资的工资等级
#①查询每个部门的平均工资
SELECT AVG(salary),department_id
FROM employees
GROUP BY department_id

SELECT * FROM job_grades;

#②连接①的结果集和job_grades表，筛选条件平均工资 between lowest_sal and highest_sal

SELECT  ag_dep.*,g.`grade_level`
FROM (
	SELECT AVG(salary) ag,department_id
	FROM employees
	GROUP BY department_id
) ag_dep
INNER JOIN job_grades g
ON ag_dep.ag BETWEEN lowest_sal AND highest_sal;

~~~

##### exists后面

~~~ sql
#四、exists后面（相关子查询）exists后的子查询返回的结果是boolean类型，也就是判断子查询是否存在

/*
语法：
exists(完整的查询语句)
结果：
1或0

*/

SELECT EXISTS(SELECT employee_id FROM employees WHERE salary=300000);

#案例1：查询有员工的部门名

#in
SELECT department_name
FROM departments d
WHERE d.`department_id` IN(
	SELECT department_id
	FROM employees

)

#exists
# 下面是相关子查询，前面是非相关子查询
SELECT department_name
FROM departments d
WHERE EXISTS(
	SELECT *
	FROM employees e
	WHERE d.`department_id`=e.`department_id`


);


#案例2：查询没有女朋友的男神信息

#in

SELECT bo.*
FROM boys bo
WHERE bo.id NOT IN(
	SELECT boyfriend_id
	FROM beauty
)

#exists
SELECT bo.*
FROM boys bo
WHERE NOT EXISTS(
	SELECT boyfriend_id
	FROM beauty b
	WHERE bo.`id`=b.`boyfriend_id`

);

~~~



#### 分页查询

**案例**

~~~ sql
#进阶8：分页查询 ★
/*

应用场景：当要显示的数据，一页显示不全，需要分页提交sql请求
语法：
	select 查询列表
	from 表
	【join type join 表2
	on 连接条件
	where 筛选条件
	group by 分组字段
	having 分组后的筛选
	order by 排序的字段】
	limit 【offset,】size;
	
	offset:要显示条目的起始索引（起始索引从0开始）
	size: 要显示的条目个数
特点：
	①limit语句放在查询语句的最后
	②公式
	要显示的页数 page，每页的条目数size
	
	select 查询列表
	from 表
	limit (page-1)*size,size;
	
	size=10
	page  
	1	0
	2  	10
	3	20
	
*/

~~~

**案例**

~~~~ sql
#案例1：查询前五条员工信息


SELECT * FROM  employees LIMIT 0,5;
SELECT * FROM  employees LIMIT 5;


#案例2：查询第11条——第25条
SELECT * FROM  employees LIMIT 10,15;


#案例3：有奖金的员工信息，并且工资较高的前10名显示出来
SELECT 
    * 
FROM
    employees 
WHERE commission_pct IS NOT NULL 
ORDER BY salary DESC 
LIMIT 10 ;
~~~~

#### union联合查询

**语法**

~~~ sql
#进阶9：联合查询
/*
union 联合 合并：将多条查询语句的结果合并成一个结果

语法：
查询语句1
union
查询语句2
union
...


应用场景：
全外链接使用的是union查询

要查询的结果来自于多个表，且多个表没有直接的连接关系，但查询的信息一致时

特点：★
1、要求多条查询语句的 查询列数 是一致的！
2、要求多条查询语句的查询的每一列的 类型 和 顺序 最好一致
3、union关键字默认去重，如果使用union all 可以包含重复项

*/

~~~

**案例**

~~~ sql
#引入的案例：查询部门编号>90或邮箱包含a的员工信息

SELECT * FROM employees WHERE email LIKE '%a%' OR department_id>90;;

-- 使用联合查询，也就是将where后面的条件拆分成多个

SELECT * FROM employees  WHERE email LIKE '%a%'
UNION
SELECT * FROM employees  WHERE department_id>90;

#案例：查询中国用户中男性的信息以及外国用户中年男性的用户信息

SELECT id,cname FROM t_ca WHERE csex='男'
UNION ALL
SELECT t_id,tname FROM t_ua WHERE tGender='male';

~~~

### DML(数据操纵语言)

~~~ sql
/*
数据操作语言：
插入：insert
修改：update
删除：delete

*/
~~~

##### 插入语句

**方式一**

~~~ sql
#方式一：经典的插入
/*
语法：
insert into 表名(列名,...) values(值1,...);

*/
USE `girls`
SELECT * FROM beauty;
#1.插入的值的类型要与列的类型一致或兼容,字符类型需要用引号引起来
INSERT INTO beauty(id,NAME,sex,borndate,phone,photo,boyfriend_id)
VALUES(13,'唐艺昕','女','1990-4-23','1898888888',NULL,2);

#2.不可以为null的列必须插入值。可以为null的列如何插入值？
#方式一：显示指定
INSERT INTO beauty(id,NAME,sex,borndate,phone,photo,boyfriend_id)
VALUES(13,'唐艺昕','女','1990-4-23','1898888888',NULL,2);

#方式二：
-- 不写可以为null的那一列
INSERT INTO beauty(id,NAME,sex,phone)
VALUES(15,'娜扎','女','1388888888');

#3.列的顺序是否可以调换
INSERT INTO beauty(NAME,sex,id,phone)
VALUES('蒋欣','女',16,'110');


#4.列数和值的个数必须一致，也就是相匹配

INSERT INTO beauty(NAME,sex,id,phone)
VALUES('关晓彤','女',17,'110');

#5.可以省略列名，默认所有列，而且列的顺序和表中列的顺序一致

INSERT INTO beauty
VALUES(18,'张飞','男',NULL,'119',NULL,NULL);

~~~

**方式二**

~~~ sql
#方式二：
/*

语法：
insert into 表名
set 列名=值,列名=值,...
*/

INSERT INTO beauty
SET id=19,NAME='刘涛',phone='999';

#两种方式大pk ★


#1、方式一支持插入多行,方式二不支持

INSERT INTO beauty
VALUES(23,'唐艺昕1','女','1990-4-23','1898888888',NULL,2)
,(24,'唐艺昕2','女','1990-4-23','1898888888',NULL,2)
,(25,'唐艺昕3','女','1990-4-23','1898888888',NULL,2);

#2、方式一支持子查询，方式二不支持

INSERT INTO beauty(id,NAME,phone)
SELECT 26,'宋茜','11809866';

INSERT INTO beauty(id,NAME,phone)
SELECT id,boyname,'1234567'
FROM boys WHERE id<3;
~~~

##### 修改语句

~~~ sql

#二、修改语句

/*

1.修改单表的记录★

语法：
update 表名
set 列=新值,列=新值,...
where 筛选条件;

2.修改多表的记录【补充】

语法：
sql92语法：
update 表1 别名,表2 别名
set 列=值,...
where 连接条件
and 筛选条件;

sql99语法：
update 表1 别名
inner|left|right join 表2 别名
on 连接条件
set 列=值,...
where 筛选条件;
*/

~~~

**案例**

~~~ sql

#1.修改单表的记录
#案例1：修改beauty表中姓唐的女神的电话为13899888899

UPDATE beauty SET phone = '13899888899'
WHERE NAME LIKE '唐%';

#案例2：修改boys表中id好为2的名称为张飞，魅力值 10
UPDATE boys SET boyname='张飞',usercp=10
WHERE id=2;

#2.修改多表的记录

#案例 1：修改张无忌的女朋友的手机号为114

UPDATE boys bo
INNER JOIN beauty b ON bo.`id`=b.`boyfriend_id`
SET b.`phone`='119',bo.`userCP`=1000
WHERE bo.`boyName`='张无忌';



#案例2：修改没有男朋友的女神的男朋友编号都为2号

UPDATE boys bo
RIGHT JOIN beauty b ON bo.`id`=b.`boyfriend_id`
SET b.`boyfriend_id`=2
WHERE bo.`id` IS NULL;

SELECT * FROM boys;
~~~

##### 删除操作

~~~ sql
/*

方式一：delete
语法：

1、单表的删除【★】会删除一整行数据
delete from 表名 where 筛选条件

2、多表的删除【补充】

sql92语法：
delete 表1的别名,表2的别名
from 表1 别名,表2 别名
where 连接条件
and 筛选条件;

sql99语法：

delete 表1的别名,表2的别名
from 表1 别名
inner|left|right join 表2 别名 on 连接条件
where 筛选条件;



方式二：truncate
语法：truncate table 表名;

*/

~~~

**案例**

~~~ sql
#方式一：delete
#1.单表的删除
#案例：删除手机号以9结尾的女神信息

DELETE FROM beauty WHERE phone LIKE '%9';
SELECT * FROM beauty;


#2.多表的删除

#案例：删除张无忌的女朋友的信息

DELETE b -- 这里删除的是beauty表中的女生信息
FROM beauty b
INNER JOIN boys bo ON b.`boyfriend_id` = bo.`id`
WHERE bo.`boyName`='张无忌';


#案例：删除黄晓明的信息以及他女朋友的信息
DELETE b,bo
FROM beauty b
INNER JOIN boys bo ON b.`boyfriend_id`=bo.`id`
WHERE bo.`boyName`='黄晓明';



#方式二：truncate语句，清空表中的数据

#案例：将魅力值>100的男神信息删除
TRUNCATE TABLE boys ;



#delete pk truncate【面试题★】

/*

1.delete 可以加where 条件，truncate不能加

2.truncate删除，效率高一丢丢
3.假如要删除的表中有自增长列，
如果用delete删除后，再插入数据，自增长列的值从断点开始，从最初删除的点开始
而truncate删除后，再插入数据，自增长列的值从1开始。因为truncate是清空表中的所有数据
4.truncate删除没有返回值，delete删除有返回值

5.truncate删除不能回滚，delete删除可以回滚.

*/

SELECT * FROM boys;

DELETE FROM boys;
TRUNCATE TABLE boys;
INSERT INTO boys (boyname,usercp)
VALUES('张飞',100),('刘备',100),('关云长',100);
~~~

### DDL(数据定义语言)

~~~ sql
/*

数据定义语言

库和表的管理

一、库的管理
创建、修改、删除
二、表的管理
创建、修改、删除

创建： create
修改： alter
删除： drop

*/
~~~

##### 库管理

~~~ sql
#1、库的创建
/*
语法：
create database  [if not exists]库名;
*/

#案例：创建库Books

CREATE DATABASE IF NOT EXISTS books ;

#2、库的修改

RENAME DATABASE books TO 新库名;

#更改库的字符集

ALTER DATABASE books CHARACTER SET gbk;

#3、库的删除

DROP DATABASE IF EXISTS books;
~~~

##### 表管理

~~~ sql
#1.表的创建 ★

/*
语法：
create table 表名(
	列名 列的类型【(长度) 约束】,
	列名 列的类型【(长度) 约束】,
	列名 列的类型【(长度) 约束】,
	...
	列名 列的类型【(长度) 约束】

)

*/
--添加约束
CREATE TABLE stuinfo(
	stuName VARCHAR(10) UNIQUE,#添加非空约束
	stuId INT PRIMARY KEY,#添加主键约束
	stuEmail VARCHAR(10)
	gender CHAR(1) DEFAULT '男' NOT NULL;#添加默认约束,约束还可以叠加
	age INT CHECK(age BETWEEN 1 AND 100);#age添加约束，但是mysql不支持
	#添加外键约束
	majorid INT ,
	CONSTRAINT fk_stuinfo_major FOREIGN KEY (majorid) REFERENCES major(id);
);

~~~

**案例**

~~~sql
--案例：创建表Book

CREATE TABLE book(
	id INT,#编号
	bName VARCHAR(20),#图书名
	price DOUBLE,#价格
	authorId  INT,#作者编号
	publishDate DATETIME#出版日期

);

DESC book;

--案例：创建表author
CREATE TABLE IF NOT EXISTS author(
	id INT,
	au_name VARCHAR(20),
	nation VARCHAR(10)

)
DESC author;

#2.表的修改

/*
语法
alter table 表名 add|drop|modify|change column 列名 【列类型 约束】;

*/

#①修改列名

ALTER TABLE book CHANGE COLUMN publishdate pubDate DATETIME;


#②修改列的类型或约束
ALTER TABLE book MODIFY COLUMN pubdate TIMESTAMP;

#③添加新列
ALTER TABLE author ADD COLUMN annual DOUBLE; 

#④删除列

ALTER TABLE book_author DROP COLUMN  annual;
#⑤修改表名

ALTER TABLE author RENAME TO book_author;

DESC book;

#3.表的删除

DROP TABLE IF EXISTS book_author;

SHOW TABLES;

#通用的写法：

DROP DATABASE IF EXISTS 旧库名;
CREATE DATABASE 新库名;

DROP TABLE IF EXISTS 旧表名;
CREATE TABLE  表名();

#4.表的复制

INSERT INTO author VALUES
(1,'村上春树','日本'),
(2,'莫言','中国'),
(3,'冯唐','中国'),
(4,'金庸','中国');

SELECT * FROM Author;
SELECT * FROM copy2;
#1.仅仅复制表的结构

CREATE TABLE copy LIKE author;

#2.复制表的结构+数据（使用子查询插入数据）
CREATE TABLE copy2 
SELECT * FROM author;

#只复制部分数据。使用子查询+条件查询
CREATE TABLE copy3
SELECT id,au_name
FROM author 
WHERE nation='中国';

#仅仅复制某些字段

CREATE TABLE copy4 
SELECT id,au_name
FROM author
WHERE 0;
~~~

