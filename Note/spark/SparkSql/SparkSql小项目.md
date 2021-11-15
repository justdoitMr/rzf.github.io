
<!-- TOC -->

- [小项目](#小项目)
  - [业务](#业务)
  - [流程分析](#流程分析)
  - [读取数据](#读取数据)
  - [数据转换](#数据转换)
  - [异常的处理](#异常的处理)
  - [减掉反常数据](#减掉反常数据)
  - [行政区信息](#行政区信息)
    - [需求分析](#需求分析)
    - [工具介绍](#工具介绍)
  - [最终代码](#最终代码)

<!-- /TOC -->

## 小项目

### 业务

**数据结构**

![1623028757244](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/093838-363777.png)

**业务场景**

![1623029918131](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/093856-684959.png)

**技术分析**

![1623029936833](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/093858-855850.png)

### 流程分析

![1623030565008](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/153955-665969.png)

![1623030863366](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/095426-64231.png)

### 读取数据

数据的schema信息。

![1623034020906](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/104757-694256.png)

### 数据转换

![1623034427231](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/15/154006-965977.png)

### 异常的处理

![1623060467625](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623060467625.png)

![1623060485758](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/180824-529125.png)

![1623060537857](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/180902-364382.png)

![1623062104561](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/183507-564039.png)

**案例**

```java
object EitherTest {
	/**
	 * Either的使用
	 * @param args
	 */
	def main(args: Array[String]): Unit = {
		val res: Either[Double, (Double, Exception)] = safe(process, 2)

		res match {
			case Left(r)=>println(res.left.get)
			case Right((b,e))=>println(b,e)
		}

	}

	/**
	 *Either就相当于left or right,left和right并不代表具体的某一种情况，只是代表左右
	 * Option===>some  None
	 * @param f 代表要传进来的函数
	 * @param b 代表传进来的函数的参数
	 * @return
	 */
	def safe(f:Double => Double,b:Double):Either[Double,(Double,Exception)]={
		try{
			val res: Double = f(b)
			Left(res)
		}catch {
			//		出错误的话，返回right,第一个参数是函数的参数，第二个参数是出错误的e
			case e:Exception=>Right(b,e)
		}
	}

	def process(b:Double):Double={
		val a=10.0
		a/b
	}
}
```

### 减掉反常数据







### 行政区信息

#### 需求分析

![1623112785578](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/083946-268376.png)

![1623112824122](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084024-403266.png)

![1623112894330](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084136-594658.png)

![1623112935319](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084215-356957.png)

![1623112957795](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084238-900562.png)

![1623113118227](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084518-619249.png)

![1623113139182](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084539-629311.png)

![1623113170599](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084612-745342.png)

#### 工具介绍

![1623113388683](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084951-501297.png)

![1623113412493](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/085013-401388.png)

**导入Maven依赖**

```java
<!--json解析库-->
        <dependency>
            <groupId>org.json4s</groupId>
            <artifactId>json4s-native_2.11</artifactId>
            <version>${json4s.version}</version>
        </dependency>
        <!--json4s的jackson继承库-->
        <dependency>
            <groupId>org.json4s</groupId>
            <artifactId>json4s-scalap_2.12</artifactId>
            <version>3.6.6</version>
        </dependency>
```

**案例**

```java
package rzf.qq.com

import org.json4s.jackson.Serialization
import org.json4s.jackson.Serialization.read
object JsonTest {

	def main(args: Array[String]): Unit = {

		//import org.json4s._
		//import org.json4s.jackson.JsonMethods._
		//
		//val product=
		//	"""
		//		|{"name":"Toy","price":35.5}
		//	""".stripMargin
		//
		//implicit val formats=Serialization.formats(NoTypeHints)
		//
		////具体解析为某一个对象
		//val pro: Any = println(parse(product)).extract[Product]
		//
		////可以直接通过一个方法，将字符串转换为对象
		//val value: Any = read[Product](product)
		//println(pro)
		//
		//val pro1: Product = Product("电视", 20)
		//compact(render(parse(pro1)))
		//val value1: Any = write(pro1)

	}


}
case class Product(name:String,price:Double)
```

![1623115079075](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/092128-666625.png)

**小结**

![1623115301150](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/093615-290331.png)

### 最终代码

```java
package rzf.qq.com

import java.text.SimpleDateFormat
import java.util.Locale
import java.util.concurrent.TimeUnit

import com.esri.core.geometry.{Geometry, GeometryEngine, MapGeometry, Point, SpatialReference}
import org.apache.ivy.osgi.updatesite.xml.FeatureParser
import org.apache.spark.broadcast.Broadcast
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.expressions.UserDefinedFunction
import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}
import org.json4s.{JObject, NoTypeHints}
import org.json4s.jackson.Serialization
import org.json4s.jackson.Serialization.read

import scala.io.{BufferedSource, Source}

object ProcessTaxi {

	def main(args: Array[String]): Unit = {
		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._
		import org.apache.spark.sql.functions._
		//1 读取数据
		val source: DataFrame = spark.read
			.option("header", value = true)
			.csv("")
	//	2 数据转换操作,将数据转换为具体的数据类型
	val taxiParse: RDD[Either[Trip, (Row, Exception)]] = source.rdd.map(safe((parse)))
		//可以通过如下方式过滤出所有异常的Row
		//taxiParse.filter(e => e.isRight)
		//	.map(e => e.right.get._1)

		val taxiGood: Dataset[Trip] = taxiParse.map(either => either.left.get).toDS()

	//	绘制时长直方图
	//	编写udf函数，完成时长的计算，将毫秒转换为小时
		val hours=(pickUpTime:Long,dropOffTime:Long)=>{
		val duration=dropOffTime-pickUpTime
		val hours=TimeUnit.HOURS.convert(duration,TimeUnit.MICROSECONDS)
		hours
	}
		val hoursUdf=udf(hours)
	//	具体的统计
		taxiGood.groupBy(hoursUdf($"pickUpTime",$"dropOffTime").as("duration"))
			.count()
			.sort("duration")
			.show()

	//	根据直方图的显示，查看数据分布之后，减掉异常的数据
		spark.udf.register("hours",hours)

		val taxiClean: Dataset[Trip] = taxiGood.where("hours(pickUpTime,dropOffTime) BETWEEN 0 AND 3")

	//	增加行政区信息
		/**
		 * 读取数据
		 * 排序
		 * 广播
		 * 创建udf，完成功能
		 * 统计信息
		 */

		val string: String = Source.fromFile("").mkString
		val collection: FeatureCollection = jsonParse(string)
	//	后期要找到每一个出粗车所在的行政区，拿到经纬度，遍历feature搜索其所在的行政区
	//	在搜索的过程中，面积越大，命中的概率越大，把大的行政区放在前面，减少遍历次数
	val sortedFeat: List[Feature] = collection.features.sortBy(feat => {
		(feat.properties("brooughCode"), feat.getGeometry().calculateArea2D())
	})
		val featBC: Broadcast[List[Feature]] = spark.sparkContext.broadcast(sortedFeat)

		val boroughLookUp=(x:Double,y:Double)=>{
		//	搜索经纬度所在的区域
		val featHit: Option[Feature] = featBC.value.find(feat => {
			GeometryEngine.contains(feat.getGeometry(), new Point(x, y), SpatialReference.create(4326))
		})
			val borough: String = featHit.map(feat => feat.properties("borought")).getOrElse("NA")
			borough
		}

		val brough: UserDefinedFunction = udf(boroughLookUp)
		taxiClean.groupBy(brough('dropOffx,'dropOffy))
			.count()
			.show()

	}


	//解析json对象
	def jsonParse(json:String):FeatureCollection={
		//导入隐式转换
		implicit val formats=Serialization.formats(NoTypeHints)

	//	json--->obj
		val collection: FeatureCollection = read[FeatureCollection](json)
		collection
	}


	/**
	 *
	 * @tparam P 参数的类型
	 * @tparam R 返回值的类型
	 * @return
	 */
	def safe[P,R](f:P=>R):P=>Either[R,(P,Exception)]={
		new Function[P,Either[R,(P,Exception)]] with Serializable{
			override def apply(param: P): Either[R, (P, Exception)] = {
				try{
					Left(f(param))
				}catch {
					case exception: Exception=>Right((param,exception))
				}
			}
		}
	}

	def parse(row:Row):Trip={
		val richRow = new RichRow(row)
		val license=richRow.getAs[String]("hack_license").orNull
		val pickUpTime=parseTime(richRow,"pickup_datetime")
		val dropOffTime=parseTime(richRow,"dropoff_datetime")
		val pickUpX=parseLocation(richRow,"pickup_longitude")
		val pickUpY=parseLocation(richRow,"pickup_latitude")
		val dropOffX=parseLocation(richRow,"dropoff_longitude")
		val dropOffY=parseLocation(richRow,"dropoff_latitude")
		Trip(license,pickUpTime,dropOffTime,pickUpX,pickUpY,dropOffX,dropOffY)
	}

	/**
	 * 时间的转换
	 * @param richRow
	 * @param field
	 * @return
	 */
	def parseTime(richRow:RichRow,field:String):Long={
	//	表示出时间的格式
		var pattern="yyyy-MM-dd HH:mm:ss"
		val formatter = new SimpleDateFormat(pattern, Locale.ENGLISH)
	//	执行转换，获取Data对象，getTime()获取时间戳
		val time: Option[String] = richRow.getAs[String](field)
		//将String类型的时间，转换为Long类型
		val timeoption: Option[Long] = time.map(time => formatter.parse(time).getTime)
		timeoption.getOrElse(0l)
	}

	/*
	地理位置的转换
	 */
	def parseLocation(richRow: RichRow,field:String):Double={
	//	获取数据，获取的数据类型是option类型
		val location: Option[String] = richRow.getAs[String](field)
	//	将String类型转换为Double类型
		val loca: Option[Double] = location.map(loc => loc.toDouble)
		loca.getOrElse(0.0d)
	}

}

/**
 * DataFrame中Row的包装类型，主要是为了包装getAs()方法
 * @param row
 */
class RichRow(row:Row){
	def getAs[T](field:String):Option[T]={
	//	1 判断row.getAs()是否是空，row中对应的field是否是空的
	//	row.fieldIndex(field):根据
		if(row.isNullAt(row.fieldIndex(field))){
		//NULL---->返回None
			None
		}else{
			Some(row.getAs[T](field))
		}

	}
}

case class Trip(
	license:String,
	pickUpTime:Long,
	dropOffTime:Long,
	pickUpX:Double,
	pickUpY:Double,
	dropOffX:Double,
	dropOffY:Double
)

case class FeatureCollection(features:List[Feature])

case class Feature(properties:Map[String,String],geometry:JObject){
	def getGeometry():Geometry={

		import org.json4s._
		import org.json4s.jackson.JsonMethods._
		val geometry1: MapGeometry = GeometryEngine.geoJsonToGeometry(compact(render(geometry)), 0, Geometry.Type.Unknown)
		geometry1.getGeometry
	}
}
```

------
