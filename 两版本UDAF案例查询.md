##### 笔记

1. `i++`与 `++i`问题

##### 数据模型

1. RDD + Data-frame + Data-Set
2. 之间的基本转换

##### SQL

1. UDF + UDAF + SQL 
2. UDF ： map方法，处理函数模型中的每个值，处理的是每一行中的数据
3. **UDAF：** 内部有聚合的能力，会保留数据，使用Buffer， 继承类，实现函数， 自定义函数，要能独立写出来方法。 ====>>    强类型函数

Spark 3.0 将强类型 与 弱类型 联合在一起，

##### hive-site.xml [配置文件]

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <!-- jdbc连接的URL -->
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
  </property>

  <!-- jdbc连接的Driver-->
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
  </property>

  <!-- jdbc连接的username-->
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
  </property>

  <!-- jdbc连接的password -->
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>000000</value>
  </property>

  <!-- Hive默认在HDFS的工作目录 -->
  <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/hive/warehouse</value>
  </property>
  <!-- 指定hiveserver2连接的host -->
  <property>
    <name>hive.server2.thrift.bind.host</name>
    <value>hadoop102</value>
  </property>

  <!-- 指定hiveserver2连接的端口号 -->
  <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
  </property>

  <!-- hiveserver2的高可用参数，开启此参数可以提高hiveserver2的启动速度 -->
  <property>
    <name>hive.server2.active.passive.ha.enable</name>
    <value>true</value>
  </property>

</configuration>
```

##### Spark 3.0 UDAF

```scala
package com.atguigu.bigdata.spark.sql

import org.apache.spark.sql.expressions.Aggregator
import org.apache.spark.sql._

object SparkSQL08_UDAF {

    def main(args: Array[String]): Unit = {
        val spark = SparkSession.builder
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        // TODO 构建模型
        val df: DataFrame = spark.read.json("data/user.json")

        df.createTempView("user")

        // TODO 在Spark中添加自定义UDAF函数
        //     UDAF函数需要采用函数类实现
        spark.udf.register("avgAge", functions.udaf( new MyAvgAgeUDAF() ))

        // SQL => 弱类型
        // UDAF => 强类型
        // Spark3.0 => 弱类型 + 强类型
        spark.sql("select avgAge(age) from user").show

        spark.stop()
    }
    case class AvgAgeBuffer( var total : Long, var cnt : Int )
    // TODO 自定义聚合函数（UDAF）
    // 1. 继承类org.apache.spark.sql.expressions.Aggregator
    // 2. 定义泛型
    //    IN : Long
    //    BUFF : 计算缓冲区的类型 AvgAgeBuffer
    //    OUT : Long
    // 3. 重写方法 （4 + 2）
    class MyAvgAgeUDAF extends Aggregator[Long, AvgAgeBuffer, Long]{

        // TODO zero表示初始
        override def zero: AvgAgeBuffer = {
            AvgAgeBuffer(0L, 0)
        }

        // TODO reduce可以理解为聚合数据
        override def reduce(buffer: AvgAgeBuffer, age: Long): AvgAgeBuffer = {
            buffer.total += age
            buffer.cnt += 1
            buffer
        }

        // TODO merge合并缓冲区
        override def merge(b1: AvgAgeBuffer, b2: AvgAgeBuffer): AvgAgeBuffer = {
            b1.total += b2.total
            b1.cnt += b2.cnt
            b1
        }

        // TODO finish完成计算
        override def finish(buffer: AvgAgeBuffer): Long = {
            buffer.total / buffer.cnt
        }

        override def bufferEncoder: Encoder[AvgAgeBuffer] = Encoders.product
        override def outputEncoder: Encoder[Long] = Encoders.scalaLong
    }
}

```

##### Spark 3.0之前的class 强类型 DS 的UDAF

```scala
package com.atguigu.bigdata.spark.sql

import org.apache.spark.sql.{Encoder, _}
import org.apache.spark.sql.expressions.{Aggregator, MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{DataType, LongType, StructField, StructType}

object SparkSQL08_UDAF_Class {

    def main(args: Array[String]): Unit = {
        val spark = SparkSession.builder
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        import spark.implicits._

        // TODO 构建模型
        val df: DataFrame = spark.read.json("data/user.json")
        val ds = df.as[User]

        // TODO Spark3.0版本前，无法将强类型的UDAF应用在弱类型的SQL中
        // TODO Spark3.0版本前，将强类型的UDAF应用在强类型的DSL中
        // 需要将UDAF的结果作为查询列,需要一个转换的过程
        val udaf = new MyAvgAgeUDAFClass()
        ds.select(udaf.toColumn).show


        spark.stop()
    }
    case class User( name:String, age:Long )
    case class AvgAgeBuffer( var total : Long, var cnt : Long )
    // TODO 自定义UDAF函数（旧版本，强类型）
    //   1. 继承类
    //   2. 定义泛型
    //      IN
    //      BUFF
    //      OUT
    //   3. 重写方法 （4 + 2）
    class MyAvgAgeUDAFClass extends Aggregator[User, AvgAgeBuffer, Long]{
        override def zero: AvgAgeBuffer = {
            AvgAgeBuffer(0L, 0L)
        }

        override def reduce(buff: AvgAgeBuffer, user: User): AvgAgeBuffer = {
            buff.total += user.age
            buff.cnt += 1
            buff
        }

        override def merge(b1: AvgAgeBuffer, b2: AvgAgeBuffer): AvgAgeBuffer = {
            b1.total += b2.total
            b1.cnt += b2.cnt
            b1
        }

        override def finish(buff: AvgAgeBuffer): Long = {
            buff.total / buff.cnt
        }

        override def bufferEncoder: Encoder[AvgAgeBuffer] = Encoders.product
        override def outputEncoder: Encoder[Long] = Encoders.scalaLong
    }

}

```

##### 3.0 之前的 弱类型 SQL 中的HDAF

```scala
package com.atguigu.bigdata.spark.sql

import org.apache.spark.sql._
import org.apache.spark.sql.expressions.{Aggregator, MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{DataType, LongType, StructField, StructType}

object SparkSQL08_UDAF_SQL {

    def main(args: Array[String]): Unit = {
        val spark = SparkSession.builder
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        // TODO 构建模型
        val df: DataFrame = spark.read.json("data/user.json")

        df.createTempView("user")

        // TODO 在Spark中添加自定义UDAF函数
        //     UDAF函数需要采用函数类实现
        spark.udf.register("avgAge", new MyAvgAgeUDAF_OldVersion())

        // SQL => 弱类型
        // UDAF => 强类型
        // Spark3.0 => 弱类型 + 强类型
        spark.sql("select avgAge(age) from user").show

        spark.stop()
    }

    // TODO 旧版本的UDAF
    //   1. 继承类UserDefinedAggregateFunction
    //   2. 重写方法 （8）
    class MyAvgAgeUDAF_OldVersion extends UserDefinedAggregateFunction{
        // TODO 输入数据的结构
        override def inputSchema: StructType = {
            StructType(
                Array(
                    StructField(
                        "age",
                        LongType
                    )
                )
            )
        }

        // TODO 缓冲区数据的结构
        override def bufferSchema: StructType = {
            StructType(
                Array(
                    StructField(
                        "total",
                        LongType
                    ),
                    StructField(
                        "cnt",
                        LongType
                    )
                )
            )
        }

        // TODO 输出数据的结构
        override def dataType: DataType = {
            LongType
        }

        // TODO 计算稳定性
        override def deterministic: Boolean = true

        // TODO 初始化
        override def initialize(buffer: MutableAggregationBuffer): Unit = {
            buffer.update(0, 0L)
            buffer.update(1, 0L)
        }

        // TODO 更新
        override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
            buffer.update(0, buffer.getLong(0) + input.getLong(0))
            buffer.update(1, buffer.getLong(1) + 1)
        }

        // TODO 合并
        override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
            buffer1.update(0, buffer1.getLong(0) + buffer2.getLong(0))
            buffer1.update(1, buffer1.getLong(1) + buffer2.getLong(1))
        }

        // TODO 计算
        override def evaluate(buffer: Row): Any = {
            buffer.getLong(0) / buffer.getLong(1)
        }
    }

}

```

##### Spark中的读写数据

```scala
        val spark = SparkSession.builder
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        // TODO SparkSQL的数据读取和保存
        // 读取数据的目的就是为了构建数据模型
        // SparkSQL读取文件和保存文件时，会默认采用Parquet格式
        // data/user.json is not a Parquet file. expected magic number at tail [80, 65, 82, 49]
        // but found [32, 53, 48, 125]
        val frame1: DataFrame = spark.read.format("json").load("data/user.json")
        val frame2: DataFrame = spark.read.json("data/user.json")

        frame1.show()

        spark.stop()
```

##### 保存数据

```scala
        val spark = SparkSession.builder
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        // TODO SparkSQL的数据读取和保存
        val frame2: DataFrame = spark.read.json("data/user.json")

        // TODO 保存数据
        // /output already exists.
        // 保存模式
        frame2.write.mode("overwrite").format("json").save("output")
```

##### JDBC 读写保存

```scala
        val spark = SparkSession.builder
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        // TODO 读取MySQL中的数据
        val df: DataFrame = spark.read.format("jdbc")
            .option("url", "jdbc:mysql://linux1:3306/spark-sql")
            .option("driver", "com.mysql.jdbc.Driver")
            .option("user", "root")
            .option("password", "123123")
            .option("dbtable", "user")
            .load()

        //df.show
        // TODO 将数据写入MySQL
        df.write.format("jdbc")
            //.mode(SaveMode.Append)
            .option("url", "jdbc:mysql://linux1:3306/spark-sql")
            .option("driver", "com.mysql.jdbc.Driver")
            .option("user", "root")
            .option("password", "123123")
            .option("dbtable", "user13")
            .save()

        spark.stop()
```

##### 读取 Hive 中的数据

```scala

        System.setProperty("HADOOP_USER_NAME","root")

        val spark = SparkSession.builder
            .enableHiveSupport()
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        // TODO 将Hive作为数据源
        spark.sql("show tables").show()

        spark.stop()
```

##### 小练习-01-数据准备

```scala
        System.setProperty("HADOOP_USER_NAME","root")

        val spark = SparkSession.builder
            .enableHiveSupport()
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        // TODO 在Hive中创建表，加载数据
        spark.sql("use atguigu220411")

        spark.sql(
            """
              |CREATE TABLE `user_visit_action`(
              |  `date` string,
              |  `user_id` bigint,
              |  `session_id` string,
              |  `page_id` bigint,
              |  `action_time` string,
              |  `search_keyword` string,
              |  `click_category_id` bigint,
              |  `click_product_id` bigint,
              |  `order_category_ids` string,
              |  `order_product_ids` string,
              |  `pay_category_ids` string,
              |  `pay_product_ids` string,
              |  `city_id` bigint
              |)
              |row format delimited fields terminated by '\t'
              |""".stripMargin)

        spark.sql(
            """
              |load data local inpath 'data/user_visit_action.txt' into table user_visit_action
              |""".stripMargin)

        spark.sql(
            """
              |CREATE TABLE `city_info`(
              |  `city_id` bigint,
              |  `city_name` string,
              |  `area` string
              |)
              |row format delimited fields terminated by '\t'
              |""".stripMargin)

        spark.sql(
            """
              |load data local inpath 'data/city_info.txt' into table city_info
              |""".stripMargin)

        spark.sql(
            """
              |CREATE TABLE `product_info`(
              |  `product_id` bigint,
              |  `product_name` string,
              |  `extend_info` string
              |)
              |row format delimited fields terminated by '\t'
              |""".stripMargin)

        spark.sql(
            """
              |load data local inpath 'data/product_info.txt' into table product_info
              |""".stripMargin)

        spark.sql(
            """
              |select * from city_info
              |""".stripMargin).show

        spark.stop()
```

##### 小练习-02-数据聚合

```scala
        System.setProperty("HADOOP_USER_NAME","root")

        val spark = SparkSession.builder
            .enableHiveSupport()
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        // TODO 在Hive中创建表，加载数据
        spark.sql("use atguigu220411")

        // TODO 需求：各区域热门商品Top3

        // 1. 过滤数据，保留所有的点击数据
        // 2. 缺什么， 补什么，多什么，删什么
        // 3  聚合数据，获取点击数量
        // 4. 将数据按照区域进行分组，组内对点击数量进行排序，取前3名
        spark.sql(
            """
              |select
              |	*
              |from (
              |	select
              |		*,
              |		rank() over ( partition by area order by clickCnt desc ) as rank
              |	from (
              |		select
              |			area,
              |			product_name,
              |			count(*) as clickCnt
              |		from (
              |			select
              |				a.*,
              |				city_name,
              |				area,
              |				product_name
              |			from user_visit_action a
              |			join city_info c on a.city_id = c.city_id
              |			join product_info p on a.click_product_id = p.product_id
              |			where a.click_product_id != -1
              |		) t1 group by area, product_name
              |	) t2
              |) t3 where rank <= 3
              |""".stripMargin).show

        spark.stop()
```

##### 小练习-03-数据分步聚合

```scala
        System.setProperty("HADOOP_USER_NAME","root")

        val spark = SparkSession.builder
            .enableHiveSupport()
            .master("local[*]")
            .appName("SparkSQL")
            .getOrCreate()

        // TODO 在Hive中创建表，加载数据
        spark.sql("use atguigu220411")

        // TODO 需求：各区域热门商品Top3

        // 1. 过滤数据，保留所有的点击数据
        // 2. 缺什么， 补什么，多什么，删什么
        spark.sql(
            """
              |	select
              |		a.*,
              |		city_name,
              |		area,
              |		product_name
              |	from user_visit_action a
              |	join city_info c on a.city_id = c.city_id
              |	join product_info p on a.click_product_id = p.product_id
              |	where a.click_product_id != -1
              |""".stripMargin).createOrReplaceTempView("t1")
        // 3  聚合数据，获取点击数量
        spark.sql(
            """
              |	select
              |		area,
              |		product_name,
              |		count(*) as clickCnt
              |	from t1 group by area, product_name
              |""".stripMargin).createOrReplaceTempView("t2")
        // 4. 将数据按照区域进行分组，组内对点击数量进行排序，取前3名
        spark.sql(
            """
              |	select
              |		*,
              |		rank() over ( partition by area order by clickCnt desc ) as rank
              |	from t2
              |""".stripMargin).createOrReplaceTempView("t3")

        spark.sql(
            """
              |select
              |	*
              |from t3 where rank <= 3
              |""".stripMargin).show

        spark.stop()
```

