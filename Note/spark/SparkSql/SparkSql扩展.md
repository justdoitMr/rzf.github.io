
<!-- TOC -->

- [SparkSql扩展](#sparksql扩展)
  - [数据源与格式](#数据源与格式)
    - [DataFrameReader](#dataframereader)
    - [text数据读取](#text数据读取)
    - [Json数据读取](#json数据读取)
    - [CSV格式](#csv格式)
    - [Parquet数据](#parquet数据)
    - [jdbc数据](#jdbc数据)
    - [数据读取框架案例](#数据读取框架案例)
  - [DataFrameWriter](#dataframewriter)
  - [Parquet文件格式](#parquet文件格式)
    - [Parquet文件](#parquet文件)
    - [写入文件指定分区](#写入文件指定分区)
  - [JSON格式文件](#json格式文件)

<!-- /TOC -->

## SparkSql扩展

在SparkSQL模块，提供一套完成API接口，用于方便读写外部数据源的的数据（从Spark 1.4版本提供），框架本身内置外部数据源：

![1622032473209](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/203435-344568.png)

### 数据源与格式

数据分析处理中，数据可以分为结构化数据、非结构化数据及半结构化数据。

![1622032548537](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153433-774381.png)

**结构化数据（Structured）**

- 结构化数据源可提供有效的存储和性能。例如，Parquet和ORC等柱状格式使从列的子集中提取值变得更加容易。
- 基于行的存储格式（如Avro）可有效地序列化和存储提供存储优势的数据。然而，这些优点通常以灵活性为代价。如因结构的固定性，格式转变可能相对困难。

**非结构化数据（UnStructured）**

- 相比之下，非结构化数据源通常是自由格式文本或二进制对象，其不包含标记或元数据以定义数据的结构。
- 报纸文章，医疗记录，图像，应用程序日志通常被视为非结构化数据。这些类型的源通常要求数据周围的上下文是可解析的。

**半结构化数据（Semi-Structured）**

- 半结构化数据源是按记录构建的，但不一定具有跨越所有记录的明确定义的全局模式。每个数据记录都使用其结构信息进行扩充。 
- 半结构化数据格式的好处是，它们在表达数据时提供了最大的灵活性，因为每条记录都是自我描述的。但这些格式的主要缺点是它们会产生额外的解析开销，并且不是特别为ad-hoc(特定)查询而构建的。

#### DataFrameReader

![1622005563689](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/130712-436516.png)

读取文件，至少设置以下内容：

- 文件地址
- 文件类型(format)
- 读取数据的参数(option)
- 结构信息（schema）

![1622005680164](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/130800-761526.png)

#### text数据读取

SparkSession加载文本文件数据，提供两种方法，返回值分别为DataFrame和Dataset

```java
object Test20 {

	def main(args: Array[String]): Unit = {
		//1.准备SparkSQL开发环境
		val spark: SparkSession = SparkSession.builder().appName("hello").master("local[*]").getOrCreate()
		val sc: SparkContext = spark.sparkContext
		sc.setLogLevel("WARN")

		val frame: DataFrame = spark.read.text("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\person")
		spark.read
				.format("text")
			.load("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\person")
				.show(3)

		//frame.show(3)

		//5.关闭资源
		sc.stop()
		spark.stop()
	}
}
```

可以查看一下底层源码

```java
//text方法返回的是DataFrame
def text(path: String): DataFrame = {
    // This method ensures that calls that explicit need single argument works, see SPARK-16009
    text(Seq(path): _*)
  }

```

![1622033516232](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153443-154824.png)

可以看出textFile方法底层还是调用text方法，先加载数据封装到DataFrame中，再使用as[String]方法将DataFrame转换为Dataset，实际中推荐使用textFile方法，从Spark 2.0开始提供。

无论是text方法还是textFile方法读取文本数据时，一行一行的加载数据，每行数据使用UTF-8编码的字符串，列名称为【value】。

#### Json数据读取

实际项目中，有时处理数据以JSON格式存储的，尤其后续结构化流式模块：StructuredStreaming，从Kafka Topic消费数据很多时间是JSON个数据，封装到DataFrame中，需要解析提取字段的值。以读取github操作日志JSON数据为例，数据结构如下：

![1622032925471](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/204715-700777.png)

使用**textFile**加载数据，对每条JSON格式字符串数据，使用SparkSQL函数库functions中自带**get_json_obejct**函数提取字段：id、type、public和created_at的值。

**函数：get_json_obejct使用说明**

![1622033604262](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153447-879432.png)

```java
object Test21 {

	def main(args: Array[String]): Unit = {
		val spark = SparkSession.builder()
			.appName(this.getClass.getSimpleName.stripSuffix("$"))
			.master("local[*]")
			// 通过装饰模式获取实例对象，此种方式为线程安全的
			.getOrCreate()
		val sc: SparkContext = spark.sparkContext
		sc.setLogLevel("WARN")
		import spark.implicits._

		// TODO: 从LocalFS上读取json格式数据(压缩）
		val jsonDF: DataFrame = spark.read.json("data/input/2015-03-01-11.json.gz")
		//jsonDF.printSchema()
		jsonDF.show(5, truncate = true)

		println("===================================================")
		val githubDS: Dataset[String] = spark.read.textFile("data/input/2015-03-01-11.json.gz")
		//githubDS.printSchema() // value 字段名称，类型就是String
		githubDS.show(5,truncate = true)

		// TODO：使用SparkSQL自带函数，针对JSON格式数据解析的函数
		import org.apache.spark.sql.functions._
		// 获取如下四个字段的值：id、type、public和created_at
		val gitDF: DataFrame = githubDS.select(
			get_json_object($"value", "$.id").as("id"),
			get_json_object($"value", "$.type").as("type"),
			get_json_object($"value", "$.public").as("public"),
			get_json_object($"value", "$.created_at").as("created_at")
		)
		gitDF.printSchema()
		gitDF.show(10, truncate = false)

		// 应用结束，关闭资源
		spark.stop()
	}
}
```

运行结果展示

![1622033732195](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153451-870714.png)

#### CSV格式

在机器学习中，常常使用的数据存储在csv/tsv文件格式中，所以SparkSQL中也支持直接读取格式数据，从2.0版本开始内置数据源。关于CSV/TSV格式数据说明：

![1622033775839](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153454-608480.png)

SparkSQL中读取CSV格式数据，可以设置一些选项，重点选项：

**分隔符：sep** 

默认值为逗号，必须单个字符

**数据文件首行是否是列名称：header** 

默认值为false，如果数据文件首行是列名称，设置为true

**是否自动推断每个列的数据类型：inferSchema** 

默认值为false，可以设置为true

当读取CSV/TSV格式数据文件首行是否是列名称，读取数据方式（参数设置）不一样的

- 行是列的名称，如下方式读取数据文件

```java
  // TODO: 读取TSV格式数据
        val ratingsDF: DataFrame = spark.read
            // 设置每行数据各个字段之间的分隔符， 默认值为 逗号
            .option("sep", "\t")
            // 设置数据文件首行为列名称，默认值为 false
            .option("header", "true")
            // 自动推荐数据类型，默认值为false
            .option("inferSchema", "true")
            // 指定文件的路径
            .csv("datas/ml-100k/u.dat")
        
        ratingsDF.printSchema()
        ratingsDF.show(10, truncate = false)
```

- 首行不是列的名称，如下方式读取数据（设置Schema信息）

```java
  // 定义Schema信息
        val schema = StructType(
            StructField("user_id", IntegerType, nullable = true) ::
                StructField("movie_id", IntegerType, nullable = true) ::
                StructField("rating", DoubleType, nullable = true) ::
                StructField("timestamp", StringType, nullable = true) :: Nil
        )
        
        // TODO: 读取TSV格式数据
        val mlRatingsDF: DataFrame = spark.read
            // 设置每行数据各个字段之间的分隔符， 默认值为 逗号
            .option("sep", "\t")
            // 指定Schema信息
            .schema(schema)
            // 指定文件的路径
            .csv("datas/ml-100k/u.data")
        
        mlRatingsDF.printSchema()
        mlRatingsDF.show(5, truncate = false)
```

- 将DataFrame数据保存至CSV格式文件，演示代码如下：

```java
 						mlRatingsDF
            // 降低分区数，此处设置为1，将所有数据保存到一个文件中
            .coalesce(1)
            .write
            // 设置保存模式，依据实际业务场景选择，此处为覆写
            .mode(SaveMode.Overwrite)
            .option("sep", ",")
            // TODO: 建议设置首行为列名
            .option("header", "true")
            .csv("datas/ml-csv-" + System.nanoTime())
```

**案例**

```java
object Test22 {
	
		def main(args: Array[String]): Unit = {
			val spark = SparkSession.builder()
				.appName(this.getClass.getSimpleName.stripSuffix("$"))
				.master("local[*]")
				// 通过装饰模式获取实例对象，此种方式为线程安全的
				.getOrCreate()
			val sc: SparkContext = spark.sparkContext
			sc.setLogLevel("WARN")
			import spark.implicits._

			/**
			 * 实际企业数据分析中
			 * csv\tsv格式数据，每个文件的第一行（head, 首行），字段的名称（列名）
			 */
			// TODO: 读取CSV格式数据
			val ratingsDF: DataFrame = spark.read
				// 设置每行数据各个字段之间的分隔符， 默认值为 逗号
				.option("sep", "\t")
				// 设置数据文件首行为列名称，默认值为 false
				.option("header", "true")
				// 自动推荐数据类型，默认值为false
				.option("inferSchema", "true")
				// 指定文件的路径
				.csv("data/input/rating_100k_with_head.data")

			ratingsDF.printSchema()
			ratingsDF.show(10, truncate = false)

			println("=======================================================")
			// 定义Schema信息
			val schema = StructType(
				StructField("user_id", IntegerType, nullable = true) ::
					StructField("movie_id", IntegerType, nullable = true) ::
					StructField("rating", DoubleType, nullable = true) ::
					StructField("timestamp", StringType, nullable = true) :: Nil
			)

			// TODO: 读取CSV格式数据
			val mlRatingsDF: DataFrame = spark.read
				// 设置每行数据各个字段之间的分隔符， 默认值为 逗号
				.option("sep", "\t")
				// 指定Schema信息
				.schema(schema)
				// 指定文件的路径
				.csv("data/input/rating_100k.data")

			mlRatingsDF.printSchema()
			mlRatingsDF.show(10, truncate = false)

			println("=======================================================")
			/**
			 * 将电影评分数据保存为CSV格式数据
			 */
			mlRatingsDF
				// 降低分区数，此处设置为1，将所有数据保存到一个文件中
				.coalesce(1)
				.write
				// 设置保存模式，依据实际业务场景选择，此处为覆写
				.mode(SaveMode.Overwrite)
				.option("sep", ",")
				// TODO: 建议设置首行为列名
				.option("header", "true")
				.csv("data/output/ml-csv-" + System.currentTimeMillis())

			// 关闭资源
			spark.stop()
		}
}
```

#### Parquet数据

SparkSQL模块中默认读取数据文件格式就是parquet列式存储数据，通过参数【spark.sql.sources.default】设置，默认值为【parquet】。

```java
object SparkSQLParquet {
    def main(args: Array[String]): Unit = {
        val spark = SparkSession.builder()
          .appName(this.getClass.getSimpleName.stripSuffix("$"))
          .master("local[*]")
          // 通过装饰模式获取实例对象，此种方式为线程安全的
          .getOrCreate()
        val sc: SparkContext = spark.sparkContext
        sc.setLogLevel("WARN")
        import spark.implicits._

        // TODO: 从LocalFS上读取parquet格式数据
        val usersDF: DataFrame = spark.read.parquet("data/input/users.parquet")
        usersDF.printSchema()
        usersDF.show(10, truncate = false)

        println("==================================================")

        // SparkSQL默认读取文件格式为parquet
        val df = spark.read.load("data/input/users.parquet")
        df.printSchema()
        df.show(10, truncate = false)

        // 应用结束，关闭资源
        spark.stop()
    }
}
```

**结果**

![1622034171994](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153504-230756.png)

#### jdbc数据

回顾在SparkCore中读取MySQL表的数据通过JdbcRDD来读取的，在SparkSQL模块中提供对应接口，提供三种方式读取数据：

- 单分区模式

![1622034270194](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153513-75063.png)

- **多分区模式**，可以设置列的名称，作为分区字段及列的值范围和分区数目

![1622034306063](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153507-580717.png)

- **高度自由分区模式**，通过设置条件语句设置分区数据及各个分区数据范围

![1622034329911](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153517-397546.png)

当加载读取RDBMS表的数据量不大时，可以直接使用单分区模式加载；当数据量很多时，考虑使用多分区及自由分区方式加载。

从RDBMS表中读取数据，需要设置连接数据库相关信息，基本属性选项如下：

![1622034358412](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153521-540873.png)

**代码演示**

```java
// 连接数据库三要素信息
        val url: String = "jdbc:mysql://node1.itcast.cn:3306/?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true"
        val table: String = "db_shop.so"
        // 存储用户和密码等属性
        val props: Properties = new Properties()
        props.put("driver", "com.mysql.cj.jdbc.Driver")
        props.put("user", "root")
        props.put("password", "123456")
        
        // TODO: 从MySQL数据库表：销售订单表 so
        // def jdbc(url: String, table: String, properties: Properties): DataFrame
        val sosDF: DataFrame = spark.read.jdbc(url, table, props)
        println(s"Count = ${sosDF.count()}")
        sosDF.printSchema()
        sosDF.show(10, truncate = false)

```

可以使用option方法设置连接数据库信息，而不使用Properties传递，代码如下：

```java
// TODO： 使用option设置参数
        val dataframe: DataFrame = spark.read
            .format("jdbc")
            .option("driver", "com.mysql.cj.jdbc.Driver")
            .option("url", "jdbc:mysql://node1.itcast.cn:3306/?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true")
            .option("user", "root")
            .option("password", "123456")
            .option("dbtable", "db_shop.so")
            .load()
        dataframe.show(5, truncate = false)

```



#### 数据读取框架案例

```java
object Test10 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//第一种读取形式
		spark.read
				.format("csv")//设置文件的格式
			.option("header",true)//设置表头信息
			.option("inferScheme",value = true)//设置自动推断数据类型，也就是说schema是来自csv文件当中的
			.load("")//读取数据的路径
			.show(10)//查看10条数据


	//	第二种读取形式
		spark.read
			.option("header",true)//设置表头信息
			.option("inferScheme",value = true)//设置自动推断数据类型，也就是说schema是来自csv文件当中的
			.csv("")
			.show(10)
	}
}
```

但是这两种方法本质上是一致的，因为类似csv这样的方式只是load()方法的封装。可以从下面的源码看出来：

```java
def csv(paths: String*): DataFrame = format("csv").load(paths : _*)
```

> 如果使用load()方式加载数据，但是没有指定format的话，默认是按照Parquet的格式读取文件。
>
> 也就是说sparksql默认读取文件的格式是Parquet格式。

![1622013939274](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/152539-357155.png)

### DataFrameWriter

![1622013978207](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/152627-708490.png)

![1622014095287](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/152815-241737.png)

**写入模式**

![1622014153714](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/152914-689673.png)

![1622014215443](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/153016-958555.png)

**案例**

```java
object Test11 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//读取一个文件
		val df: DataFrame = spark.read
			.format("csv") //设置文件的格式
			.option("header", true) //设置表头信息
			.load("")//读取数据的路径


	//	写入文件
	val json: Unit = df.write
		.json("")

	//	第二种写入方式
		df.write
			.format("json")
			.save("")
	}
}
```

### Parquet文件格式

#### Parquet文件

![1622017248166](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/162050-259946.png)

**文件的读取**

![1622020881136](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/172122-190055.png)

```java
object Test12 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		val df: DataFrame = spark.read
			.option("header", true)
			.csv("")

	//	数据存储为parquet格式
		df.write//spark默认写入文件的格式是parquet格式
				.mode(SaveMode.Overwrite)//指定写入模式为重写
			.format("parquet").save("")


	//	读取parquet格式的文件
	//	默认的读取格式也是parquet格式文件
	//	可以读取文件夹
		spark.read
			.format("json")
			.load("")

	}
}
```

spark默认读取和写入的都是parquet格式的文件。

在写入的时候，会写入文件夹内，因为spark是支持分区操作单位，每一个分区会写入一个文件夹内。

读取数据的时候也是按照分区进行读取的，逐个文件夹进行遍历读取操作。

#### 写入文件指定分区

为什么要进行分区？

![1622020819885](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/172021-707124.png)

![1622018506792](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622018506792.png)

表分区不仅在parquet格式的文件上面有，在其他格式的文件也有分区。

```java
object Test13 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		val df: DataFrame = spark.read
			.option("header", true)
			.csv("")


	//	写文件，表分区
	//	写分区表的时候，分区列不会包含在生成的文件中，仅仅显示在文件夹上面
	//	读取的时候，自然也读取不到分区信息
		df.write
			//这里是按照列进行分区操作
			.partitionBy("year","month")//按照年和月进行分区
			.save("")

	//	读取文件，自行发现分区
	//	如果读取的时候没有指定具体的分区，而是指定分区的文件夹，那么会自动的发现分区，这个时候
	//	分区列也会输出在文件中
		spark.read
			.parquet("")//这里需要指定具体分区的路径
			.printSchema()//打印表头信息
	}
}
```

### JSON格式文件

![1622021108833](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/173403-591287.png)

![1622021698139](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153544-274367.png)

![1622021725453](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/173527-130371.png)

```java
object Test14 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		val df: DataFrame = spark.read
			.option("header", true)
			.csv("")

	//	写入外部的时候，以json方式写入
	//但是写入的不是标准格式的json文件
		df.write
			.json("")
	}
}
```

**处理json文件格式的小技巧**

- toJSON()：可以将一个DataFrame（里面存储的是对象）转换为JSON格式的DataFrame（也就是json字符串）
  - toJSON()应用场景：处理完数据之后，DataFrame中如果是一个对象，如果其他的系统只支持json格式的数据，sparksql和这种系统进行整合的时候，那么就需要这种形式的转换。

```java
df.toJSON.show()
```

- 可以直接从RDD读取JSON格式的DataFrame

```java
//将json格式的数据转换为一个RDD
		val rdd: RDD[String] = df.toJSON.rdd
		//从rdd中读取出一个dataframe
		val frame: DataFrame = spark.read.json(rdd)
```

![1622022486043](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/174806-270779.png)
