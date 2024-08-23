##### 笔记

1. Spark SQL ：是按照行进行读取数据的， 要求一行是一个JSON 格式的文件；
2. 常量的计算是在编译的时候计算的，一个常量可以赋值给 char 类型变量

##### 小练习04-UDAF完成

```scala
package com.atguigu.bigdata.spark.sql

import org.apache.spark.sql._
import org.apache.spark.sql.expressions.Aggregator

import scala.collection.mutable
import scala.collection.mutable.ListBuffer

object SparkSQL13_Req {

    def main(args: Array[String]): Unit = {
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
        spark.udf.register("cityRemark", functions.udaf(new CityRemarkUDAF))

        spark.sql(
            """
              |	select
              |		area,
              |		product_name,
              |		count(*) as clickCnt,
              |     cityRemark(city_name)
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
              |""".stripMargin).show(false)

        spark.stop()

    }
    case class CityRemarkBuffer( var total : Long, var cityMap : mutable.Map[String, Long] )
    // TODO 自定义聚合函数（城市备注）
    // 1. 继承类Aggregator
    // 2. 定义泛型
    //    IN : String(城市名称)
    //    BUFF : (total, cityMap)
    //    OUT :String（备注）
    // 3. 重写方法（4 + 2）
    class CityRemarkUDAF extends Aggregator[String, CityRemarkBuffer,String] {
        override def zero: CityRemarkBuffer = {
            CityRemarkBuffer(0L, mutable.Map[String, Long]() )
        }

        override def reduce(buff: CityRemarkBuffer, city: String): CityRemarkBuffer = {
            buff.total += 1L
            val map: mutable.Map[String, Long] = buff.cityMap
            val oldCnt: Long = map.getOrElse(city, 0L)
            map.update(city, oldCnt + 1L)
            buff.cityMap = map
            buff
        }

        override def merge(b1: CityRemarkBuffer, b2: CityRemarkBuffer): CityRemarkBuffer = {
            b1.total += b2.total

            b2.cityMap.foreach {
                case ( city, cnt2 ) => {
                    // Long, Int => AnyVal
                    val oldCnt: Long = b1.cityMap.getOrElse(city, 0L)
                    b1.cityMap.update(city, oldCnt + cnt2)
                }
            }

            b1
        }

        override def finish(buffer: CityRemarkBuffer): String = {
            val list = ListBuffer[String]()

            val total: Long = buffer.total
            val map: mutable.Map[String, Long] = buffer.cityMap
            val top2: List[(String, Long)] = map.toList.sortBy(_._2)(Ordering.Long.reverse).take(2)

            var rest = 100L

            top2.foreach {
                case (city, cnt) => {
                    val r = cnt * 100 / total
                    rest -= r
                    list.append(s"${city} ${r}%")
                }
            }

            if ( map.size > 2 ) {
                list.append(s"其他 ${rest}%")
            }

            list.mkString(", ")
        }

        override def bufferEncoder: Encoder[CityRemarkBuffer] = Encoders.product
        override def outputEncoder: Encoder[String] = Encoders.STRING
    }

}

```

##### Spark源码提交流程

![1658836022090](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658836022090.png)

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658836143332.png" alt="1658836143332"  />

- SparkSubMit进程