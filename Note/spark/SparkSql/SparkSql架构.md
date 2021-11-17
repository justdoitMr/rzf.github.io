
<!-- TOC -->

- [Spark Sql架构](#spark-sql架构)
- [Spark Sql执行过程](#spark-sql执行过程)
- [Spark Sql工作流程](#spark-sql工作流程)
- [具体工作流程](#具体工作流程)

<!-- /TOC -->

## Spark Sql架构

Spark SQL兼容Hive，这是因为Spark SQL架构与Hive底层结构相似，Spark SQL复用了Hive提供的元数据仓库（Metastore）、HiveQL、用户自定义函数（UDF）以及序列化和反序列工具（SerDes），下面通过下图深入了解Spark SQL底层架构。

![20211115162312](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115162312.png)

从上图中可以看出，Spark SQL架构与Hive架构相比，**除了把底层的MapReduce执行引擎更改为Spark，还修改了Catalyst优化器**，Spark SQL快速的计算效率得益于Catalyst优化器。从HiveQL被解析成语法抽象树起，**执行计划生成和优化的工作全部交给Spark SQL的Catalyst优化器进行负责和管理**。

Catalyst优化器是一个新的可扩展的查询优化器，它是基于Scala函数式编程结构，Spark SQL开发工程师设计可扩展架构主要是为了在今后的版本迭代时，能够轻松地添加新的优化技术和功能，尤其是为了解决大数据生产环境中遇到的问题（例如，针对半结构化数据和高级数据分析），另外，Spark作为开源项目，外部开发人员可以针对项目需求自行扩展Catalyst优化器的功能。下面通过下图描述Spark SQL的工作原理。

![20211115162237](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115162237.png)

## Spark Sql执行过程

Spark要想很好地支持SQL，就需要完成

- 解析（Parser）
- 优化（Optimizer）
- 执行（Execution）
  
三大过程。Catalyst优化器在执行计划生成和优化的工作时候，它离不开自己内部的五大组件，具体介绍如下所示。

**Parse组件**：该组件根据一定的语义规则（即第三方类库ANTLR）将SparkSql字符串解析为一个抽象语法树/AST。

**Analyze组件**：该组件会遍历整个AST，并对AST上的每个节点进行数据类型的绑定以及函数绑定，然后根据元数据信息Catalog对数据表中的字段进行解析。

**Optimizer组件**：该组件是Catalyst的核心，主要分为RBO和CBO两种优化策略，其中RBO是基于规则优化，CBO是基于代价优化。

**SparkPlanner组件**：优化后的逻辑执行计划OptimizedLogicalPlan依然是逻辑的，并不能被Spark系统理解，此时需要将OptimizedLogicalPlan转换成physical plan（物理计划）。

**CostModel组件**：主要根据过去的性能统计数据，选择最佳的物理执行计划。

**小结**

- Parse组件：解析字符串为抽象语法树
- Analyze组件：堆数据类型以及函数类型进行绑定，根据元数据信息堆表中的字段进行解析。
- Optimizer组件：对解析好的语法树进行优化。
- SparkPlanner组件：将优化好的逻辑执行计划转换为物理执行计划
- CostModel组件：选择最佳物理执行计划进行执行。


## Spark Sql工作流程

![20211117143741](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211117143741.png)

在了解了上述组件的作用后，下面分步骤讲解Spark SQL工作流程。

1. 在解析SQL语句之前，会创建SparkSession，涉及到表名、字段名称和字段类型的元数据都将保存在Catalog中；

2. 当调用SparkSession的sql()方法时就会使用SparkSqlParser进行解析SQL语句，解析过程中使用的ANTLR进行词法解析和语法解析；

3. 接着使用Analyzer分析器绑定逻辑计划，在该阶段，Analyzer会使用Analyzer Rules，并结合Catalog，对未绑定的逻辑计划进行解析，生成已绑定的逻辑计划；

4. 然后Optimizer根据预先定义好的规则（RBO）对 Resolved Logical Plan 进行优化并生成 Optimized Logical Plan（最优逻辑计划）；

5. 接着使用SparkPlanner对优化后的逻辑计划进行转换，生成多个可以执行的物理计划Physical Plan；

6. 接着CBO优化策略会根据Cost Model算出每个Physical Plan的代价，并选取代价最小的 Physical Plan作为最终的Physical Plan；

7. 最终使用QueryExecution执行物理计划，此时则调用SparkPlan的execute()方法，返回RDD。

## 具体工作流程

![20211115163259](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115163259.png)

以上是SparkSQL的总体执行逻辑，与传统的SQL语句执行过程类似，大致分为**SQL语句、逻辑计划、物理计划以及物理操作**几个阶段，每个阶段又会做一些具体的事情，我们来具体看下各个阶段具体做了些什么。

1. **Parser阶段**

这个阶段是将 SQL 语句转化为UnResolved Logical Plan(包 含 UnresolvedRelation、 UnresolvedFunction、 UnresolvedAttribute)的过程，期间要经过词法分析和语法分析。效果如下：

![20211115163410](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115163410.png)

该阶段式是将字符串解析为抽象的AST语法树。

**analyzer阶段**

这个阶段是将Unresolved Logical Plan 进一步转化为Analyzed Logical Plan的过程。

![20211115163509](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115163509.png)

经过Analyzer之后，查询中涉及到的表及字段信息都会被解析赋值，该阶段对抽象语法树进行数据类型和函数进行绑定，对表中的列属性进行解析。

![20211115163541](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115163541.png)

**Optimizer阶段**

这个阶段是将Analyzed Logical Plan 转换成Optimized Logical  Plan 。

Optimizer 的主要职责是将 Analyzed Logical Plan 根据不同的优化策略来对语法树进行优化，优化逻辑计划节点(Logical Plan)以及表达式(Expression)。它的工作原理和 Analyzer 一致，也是通过其下的 Batch 里面的 Rule[LogicalPlan]来进行处理的

![20211115163645](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115163645.png)

**Spark Planner阶段**

这个阶段是将Optimized Logical Plan 转换为Spark  Plan

前面讲述的主要是逻辑计划，即 SQL 如何被解析成Logical Plan，以及 Logical Plan 如何被 Analyzer 以及 Optimzer，接下来就是逻辑计划被翻译成物理计划，即 SparkPlan。

![20211115163728](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115163728.png)

**PrepareForExecution阶段**

该阶段主要是将Spark Plan 转换为Executed Plan

在 SparkPlan 中插入 Shuffle 的操作，如果前后 2 个 SparkPlan 的 outputPartitioning 不一样的话，则中间需要插入 Shuffle 的动作，比分说聚合函数，先局部聚合，然后全局聚合，局部聚合和全局聚合的分区规则是不一样的，中间需要进行一次 Shuffle。

![20211115163807](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115163807.png)

**Execute阶段**

该阶段是将Executed Plan 进行执行获取RDD[Row]

![20211115163839](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115163839.png)

Spark最终通过Physical Operation来进行数据读取，如下图函数。

![20211115163912](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211115163912.png)