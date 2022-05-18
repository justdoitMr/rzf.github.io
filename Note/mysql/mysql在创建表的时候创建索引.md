# mysql在创建表的时候创建索引

  在创建数据表时创建索引的基本语法结构：

    CREATE  TABLE  table_name(
    属性名  数据类型[约束条件],
    ……
    属性名  数据类型
    [UNIQUE  |  FULLTEXT   |   SPATIAL  ]   INDEX  |  KEY
    [  别名 ]  (  属性名1   [(  长度  )]   [  ASC  |  DESC  )
    );
   属性值的含义如下:

1. UNIQUE: 可选参数，表示索引为唯一索引。
2.  FULLTEXT:  可选参数，表示索引为全文索引。
3. SPATIAL:  可选参数，表示索引为空间索引。
4.  INDEX  和 KEY 参数用于指定字段为索引的，用户在选择时，只需要选择其中的一种即可。
5.  "别名" : 为可选参数，其作用是给创建的索引取新名称。
6.  属性名1:  指索引对应的字段名称，该字段必须被预先定义。
7.  长度:  可选参数，其指索引的长度，必须是字符串类型才可以使用。
8. ASC/DESC: 可选参数，ASC 表示升序排列，DESC 表示降序排列。

## 普通索引创建

创建普通索引，即不添加  UNIQUE、FULLTEXT 等任何参数。

【例】创建表名为 score 的数据表，并在该表的 id 字段上建立索引，SQL 语句如下:

~~~sql
mysql> CREATE table score(
    -> id  int(11)  AUTO_INCREMENT  primary  key  not  null,
    -> name  varchar(50)  not null,
    -> math  int(5)  not null,
    -> English  int (5)  not null,
    -> Chinese  int (5) not  null,
    -> index(id)
    -> );
~~~

## 创建唯一索引

创建唯一索引时，使用  UNIQUE  参数进行约束。

【例】创建表名为 address  的数据表，并在该表的 id 字段上建立唯一索引，SQL 语句如下:

~~~sql
mysql> CREATE  table address(
    -> id  int(11)  auto_increment  primary  key  not  null,
    -> name  varchar(50),
    -> address  varchar(200),
    -> UNIQUE  INDEX  address(id  ASC)
    -> );
~~~

3. 创建全文索引
   全文索引只能作用在  CHAR、VARCHAR、TEXT、类型的字段上。创建全文索引需要使用  FULLTEXT  参数进行约束。

   【例】创建表名为 cards  的数据表，并在该表的 name 字段上建立全文索引，SQL 语句如下:

~~~sql
mysql> create  table cards(
    -> id int(11)  auto_increment  primary key  not  null,
    -> name  varchar(50),
    -> number  bigint(11),
    -> info  varchar(50),
    -> FULLTEXT  KEY  cards_number(name)
    -> );
~~~

## 创建单列索引

创建单列索引，即在数据表的单个字段上创建索引。创建该类型索引不需要引入约束参数，用户在建立时只需要指定单列字段名，即可创建单列索引。

【例】创建名称为  telephone  的数据表，并指定在  tel  字段上建立名称为  tel_num  的单列索引，SQL  语句如下：

~~~sql
mysql> create  table  telephone(
    -> id  int(11)  primary key auto_increment  not  null,
    -> name  varchar(50)  not  null,
    -> tel  varchar(50)  not null,
    -> index  tel_num(tel(20))
    -> );
~~~

## 创建多列索引

创建多列索引即指定表的多个字段即可实现。

【例】创建名称为  information  的数据表，并指定  name  和  sex  为  多列索引，SQL  语句如下：

~~~sql
mysql> create table  information(
    -> inf_id  int(11)  auto_increment  primary  key  not  null,
    -> name  varchar(50)  not  null,
    -> sex  varchar(5)  not null,
    -> birthday  varchar(50)  not  null,
    -> index  info(name,sex)
    -> );
~~~

需要注意的是，在多列索引中，只有查询条件中使用了这些字段中的第一个字段(即上面示例中的  name 字段)，索引才会被使用。
    触发多列索引的条件是用户必须使用索引的第一字段，如果没有用到第一字段，则索引不起任何作用，用户想要优化查询速度，可以应用该类索引形式。

## 创建空间索引

创建空间索引时，需要设置  SPATIAL 参数。同样，必须说明的是，只有  MyISAM 类型表支持该类型索引。而且，索引字段必须有非空约束。

【例】创建一个名称为 list 的数据表，并创建一个名为  listinfo 的空间索引，SQL语句如下:

~~~sql
mysql> create  table  list(
    -> id  int(11)  primary  key  auto_increment  not null,
    -> goods  geometry  not  null,
    -> SPATIAL  INDEX  listinfo(goods)
    -> )engine=MyISAM;
~~~

goods  字段上已经建立名称为  listinfo 的空间索引，其中  goods  字段必须不能为空，且数据类型是  GEOMETRY，该类型是空间数据类型。空间类型不能用其他类型代替，否则在生成空间素引时会产生错误且不能正常创建该类型索引。


    空间类型除了上述示例中提到的 GEOMETRY 类型外，还包括如  POINT、LINESTRING、POLYGON  等类型，这些空间教据类型在平常的操作中很少被用到。
