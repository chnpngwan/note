##### 交集并集差集拉链

- 并集不会去重， 并集后使用set集合，会有去重效果
- 差集函数名： `rdd1.subtract(rdd2)`

```scala
package rdd.opration.transform

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Scala12_Transform {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("RDDInstance")

    val sc = new SparkContext(conf)

    val rdd1 = sc.makeRDD(
      List(1, 2, 3, 4), 2
    )

    val rdd2 = sc.makeRDD(
      List(3, 4, 5, 6), 2
    )

    // 交集， 并集， 差集
    //    -- 交集并集差集 画图
    //    交集： 两个数据集中交叉的部分，;
    println(rdd1.intersection(rdd2).collect().mkString(", "))


    // 并集 不会将数据去重，使用set集合会去重，
    println(rdd1.union(rdd2).collect().mkString(", "))

    // 差集【函数名称与scala不一样】
    println(rdd1.subtract(rdd2).collect().mkString(", "))


    // zip 拉链:相同的位置拉在一块
    println(rdd1.zip(rdd2).collect().mkString(", "))

    println("-------------------")


    sc.stop()

  }

}
```

##### 拉链

1. 拉链 zip 要满足条件， 分区数相同， 区内元素个数相同
2. Spark中的 zip 比Scala中的要严格一些

```scala
package rdd.opration.transform

import org.apache.spark.{SparkConf, SparkContext}

object Scala12_Transform_1 {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("RDDInstance")

    val sc = new SparkContext(conf)

    val rdd1 = sc.makeRDD(
      List(1, 2, 3, 4), 2
    )

    val rdd2 = sc.makeRDD(
      List(3, 4, 5, 6), 2
    )


    // zip 拉链:相同的位置拉在一块
    // Spark中能够 分区数量要相同， 分区没数据数量要相同;
    println(rdd1.zip(rdd2).collect().mkString(", "))

    println("-------------------")


    sc.stop()

  }

}
```

##### 函数中的泛型问题

1. 一些函数会进行二次编译
2. 需要检查泛型， 根据泛型的内容激进行编译重载

```scala
package rdd.opration.transform2

import org.apache.spark.{SparkConf, SparkContext}

object Scala12_Transform_2 {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("RDDInstance")

    val sc = new SparkContext(conf)

    val rdd1 = sc.makeRDD(
      List(1, 2, 3, 4), 2
    )

    val rdd2 = sc.makeRDD(
      List(3, 4, 5, 6), 2
    )

    val rdd3 = sc.makeRDD(
      List("3", "4", "5", "6"), 2
    )
    // 交集， 并集， 差集 两个运算的RDD 的泛型要相同, 类型不一致不能运算
    // zip: 拉链 中对泛型没有约束;
    //    -- 交集并集差集 画图
    //    交集： 两个数据集中交叉的部分，;
    println(rdd1.intersection(rdd2).collect().mkString(", "))


    // 并集 不会将数据去重，使用set集合会去重，
    println(rdd1.union(rdd2).collect().mkString(", "))

    // 差集【函数名称与 scala 不一样】
    println(rdd1.subtract(rdd2).collect().mkString(", "))

    // zip 拉链:相同的位置拉在一块
    // Spark中能够 分区数量要相同， 分区没数据数量要相同;
    // println(rdd1.zip(rdd2).collect().mkString(", "))

    // 【泛型不可变，】
    // 所谓的泛型是内部的数据类型约束，


    println("-------------------")

    sc.stop()

  }

}
```

##### 泛型原理

1. 方法能不能调用，看类型
2. 方法的具体执行，看对象
3. 二次编译，隐式转换，关注泛型

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Scala13_Transform {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("RDDInstance")

    val sc = new SparkContext(conf)


    val rdd1 = sc.makeRDD(
      List(3, 4, 5, 6), 2
    )

    // 方法谁提供 类型就声明为谁，
    // 类型决定 他有哪些方法， 能用哪些方法
    // 能不能用看类型， 怎么用看方法；
    val value: RDD[(Int, Int)] = rdd1.map((_, 1))

    // 二次编译： 隐式转换, 其中泛型不可变，只有相同的泛型才会传递过去，完成隐式转换
    value.partitionBy(null)

    println("-------------------")
    sc.stop()

  }

}
```

##### partitionBy: 中的分区规则HashMap扩容原理

![1658231037912](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658231037912.png)

1. 默认分区器是HashMap
2. 通过文件展示的图标辨别文件内容，类型
3. HashMap扩容两倍==他的数据存放规则，key的Hash值 与表的长度进行位运算得到存储位置

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{HashPartitioner, RangePartitioner, SparkConf, SparkContext}

object Scala14_Transform {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("RDDInstance")

    val sc = new SparkContext(conf)


    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(("a", 1), ("b", 2),
        ("c", 3), ("d", 4)),
      2
    )

    // TODO repartition : 改变数据的分区数量， 侧重分区数量
    // partitionBy： 侧重数据在哪里， 数据的位置 ；


    //    rdd1.repartition()

    // partitionBy： 需要传递一个 patitioner 参数 分区器对象
    // Ctrl + H 快捷键 查看类的继承；列表
    // patitioner 是一个抽象类， 需要构建子类， 默认Spark提供了三个子类
    // 能用的就两个子类 HashPartitioner & RangePartitioner;
    // 文件中有访问权限， 文件中的小图标标识文件中的内容

    // TODO Spark 中计算的时候， 默认使用 HashPartitioner进行分区:
    //      底层中使用 key 的hash值， 对分区取余之后 确定具体的分区;
    rdd1.saveAsTextFile("output1")
    rdd1.partitionBy(new HashPartitioner(2)).saveAsTextFile("output2")

    //    排序: RangePartitioner 对数据有要求，数据能排序，使用不方便

    // HashMap 扩容为什么会扩容 2 倍: 默认数据的存放策略
    // 位运算（&）与模（%）运算
    // 扩容两倍才能保证 最低的几位都是1 ， 保证为运算之后的数据是连续的
    // % 运算可以不考虑数据的分区数量 也能保证数据均衡；

    /*
    16
    HashMap   => hash(key.HashCode) & (length-1)

    partitionner => key.hashCode  % length

    01110000
  &
    00001111

    */


    sc.stop()

  }

}
```

##### PartitionBy：自定义分区器

1. 模式匹配的使用

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{HashPartitioner, Partitioner, SparkConf, SparkContext}

object Scala14_Transform_1 {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("RDDInstance")

    val sc = new SparkContext(conf)


    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(("nba", 1), ("wnba", 2),
        ("nba", 3), ("cba", 4)),
      2
    )

    // TODO : 自己定义分区规则
    // nba => 0
    // wba => 1
    // cba => 2;

    rdd1.partitionBy(new MyPartitioner()).saveAsTextFile("output")

    sc.stop()

  }

  //TODO 自定义分区器对象
  // 继承类： partitioner
  class MyPartitioner extends Partitioner{

    // TODO 分区数量
    override def numPartitions: Int = {
      3
    }
    // TODO 根据key的值 指定 确定分区
    override def getPartition(key: Any): Int = {
      key match {
        case "nba" => 0
        case "cba" => 1
        case "wnba" => 2
      }
    }
  }

}
```

##### reduceByKey实现WordCount

![1658231148370](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658231148370.png)

1. 存在shuffle 过程
2. 内部的匿名函数是针对相同的key 的value进行的操作， 将多个value reduce为一个值

![1658231315241](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658231315241.png)

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{Partitioner, SparkConf, SparkContext}

object Scala15_Transform {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("RDDInstance")

    val sc = new SparkContext(conf)


    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(("a", 1), ("a", 2),
        ("a", 3), ("a", 4)),
      2
    )

    // reduceByKey: 算子表示 将数据集中的数据两两聚合
    //    TODO 是分布式计算， 应该考虑分区的处理，存在分区内计算，分区之间计算
    //    分区内与分区之间的计算逻辑是相同的
    //    reduceByKey 存在shuffle 过程， 可以改变分区数量
    //    TODO 数据的分区使用默认的 hashPartitoner ;
    //    TODO 可以实现的 wordCount （2 / 10） ;

    rdd1.reduceByKey(
      (x, y) =>{

        println(s"$x + $y =  ${x+y}")
        x+y
      }
    ).collect().foreach(println)

    sc.stop()

  }

}
```

##### groupbykey与reduceByKey-两者实现wordcount分析

![1658231371737](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658231371737.png)

1. reduceByKey可以实现预聚合 combiner，减少 shuffle中的数据量，效率高
2. group by：按照规则进行分类，可自定义规则，返回结果生成新数据 `RDD[(String, Iterable[(String, Int)])]`
3. groupbykey： 直接将数据按照key进行分组，`RDD[(String, Iterable[Int])]`

![1658231411071](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658231411071.png)

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Scala16_Transform {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("RDDInstance")

    val sc = new SparkContext(conf)


    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(("a", 1), ("a", 2),
        ("a", 3), ("a", 4)),
      2	·	
    )

    // TODO groupby 按照指定的规则进行分组
    //  groupByKey： 只能按照数据的 key 进行分组；
    // 按照key 直接对书籍进行分组
    val rdd12: RDD[(String, Iterable[(String, Int)])] = rdd1.groupBy(_._1)

    // groupByKey: 等同于将 key 与 v 分开了 按照key 对 v 进行分组
    // TODO 可以实现 wordCount （3 / 10）；
    // -----画图 groupbykey 与 reducebykey 哪一个效率更高
    // shuffle的性能决定了当前作业的性能
    //    优化shuffle =>  1.更换硬件
    //                    2. 提升磁盘的缓冲区
    //                    3. 保证数据结果不变的情况下，减少shuffle的次数【小文件传输慢，时间花在了打开关闭文件上了】
    //                    4. 保证数据结果不变的情况下，减少数据量，；

    /*
    -- reduceByKey： 可以在shuffle之前 将相同的key 进行聚合操作，减少shuffle的数据量
                    1. 可以在shuffle之前可以进行预聚合， 称之为 combine，

    */
    val rdd13: RDD[(String, Iterable[Int])] = rdd1.groupByKey()

    rdd1.reduceByKey(_+_)
    sc.stop()

  }

}
```

##### 分区最大值再求和

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}


// 20220717 --- 下午
object Scala17_Transform_test {
  def main(args: Array[String]): Unit = {

    //1.创建SparkConf并设置App名称
    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")

    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)

    val rdd1: RDD[Int] = sc.makeRDD(
      List(1, 2, 3, 4, 5, 6),
      3

    )

    // 求分区最大值值和
    val maxRDD: RDD[Int] = rdd1.mapPartitions(
      iter => {
        List(iter.max).iterator
      }
    )
    val axRDD: RDD[(String, Int)] = maxRDD.map(("a", _))

    axRDD.reduceByKey(_ + _).map(_._2).collect().foreach(println)

    val newRDD: RDD[(String, Iterable[Int])] = axRDD.groupByKey()

    newRDD.mapValues(_.sum).map(_._2)
    
    //4.关闭连接
    sc.stop()

  }


}
```

##### aggregateByKey

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}


// 20220717 --- 下午
object Scala17_Transform_test_1 {
  def main(args: Array[String]): Unit = {

    //1.创建SparkConf并设置App名称
    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")

    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)

    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(
        ("a", 1), ("a", 4), ("b", 8), ("c", 9),
        ("c", 3), ("d", 3), ("a", 4), ("b", 5)
      ),
      2

    )

    // TODO 分区内的计算逻辑 与分区之间的计算逻辑不一样
    //  TODO Spark提供了aggregateByKey 这个算子， 可以区分 分区内外的计算逻辑；
    // aggregateByKey： 存在函数柯里化调用，
    //  第一个参数表示：zeroValue（零值）， 计算的初始值，
    // 第二个参数列表有两个参数；
    //        第一个参数表示分区内，对相同 key  value的计算逻辑，
    //        第二个参数表示分区之间，对相同的key   value计算逻辑
    // ---------画图理解过程。 Spark-算子-2；
    rdd1.aggregateByKey(100)(
      (x, y) => {
        math.max(x, y)
      },
      (x, y) => {
        x + y
      }

    ).collect().foreach(println)

    //4.关闭连接
    sc.stop()

  }


}
```



##### aggregateByKey： 实现WordCount

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}


// 2022-07-17 --- 下午
object Scala17_Transform_test_2 {
  def main(args: Array[String]): Unit = {

    //1.创建SparkConf并设置App名称
    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")

    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)

    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(
        ("a", 1), ("a", 4), ("b", 8), ("c", 9),
        ("c", 3), ("d", 3), ("a", 4), ("b", 5)
      ),
      2

    )

    // TODO 分区内的计算逻辑 与分区之间的计算逻辑不一样
    //  TODO Spark提供了aggregateByKey 这个算子， 可以区分 分区内外的计算逻辑；
    // aggregateByKey： 寸在函数柯里化调用，
    //  第一个参数表示：zeroValue（零值）， 计算的初始值，
    // 第二个参数列表有两个参数；
    //        第一个参数表示分区内，对相同 key  value的计算逻辑，
    //        第二个参数表示分区之间，对相同的key   value计算逻辑
    // ---------画图理解过程。 Spark-算子-2
    // TODO aggregateByKey: 可以实现 wordcount （4 / 10）；
    // 下划线， 按照顺序只是用一次， 可以使用；
    rdd1.aggregateByKey(0)(_ + _, _ + _).collect().foreach(println)

    //  aggregateByKey: 如果分区内外的逻辑一样， 可以使用简化写法，更换一个算子
    //  TODO  wordCount (5 / 10)
    rdd1.foldByKey(0)(_+_).collect().foreach(println)


    //4.关闭连接
    sc.stop()

  }


}
```



##### combineByKey：实现WordCount

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}


// 2022-07-17 --- 下午
object Scala18_Transform_test {
  def main(args: Array[String]): Unit = {

    //1.创建SparkConf并设置App名称
    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")

    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)

    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(
        ("a", 1), ("a", 4), ("b", 8), ("c", 9),
        ("c", 3), ("d", 3), ("a", 4), ("b", 5)
      ),
      2

    )
    // TODO 求每个 key 的平均值 ；
    /*

        rdd1.map {
          case (word, count) => {
            (word, (count, 1))
          }
        }.reduceByKey(
          (t1, t2) => {
            (t1._1 + t2._1, t1._2 + t2._2)
          }
        ).mapValues{
          case (total, cnt) => {
            total / cnt
          }
        }
    */


    // TODO Spark 提供了可以转换数据格式的算子
    //  combineByKey: 提供了三个参数
    //    第一个参数表示： 转换第一个元素的数据格式
    //    第二个参数表示：分区内计算逻辑
    //    第三个参数表示：分区之间计算逻辑
    //
    //    泛型的擦除， 泛型只在编译的时候有用， 在代码执行的时候，泛型会被擦除，
    //    当运行之后才产生的数据以及类型， 泛型不可以省略;
    /*
        rdd1.combineByKey(
          v => {
            (v, 1)
          },
          (t: (Int, Int), v) => {
            (t._1 + v, t._2 + 1)
          },
          (t1: (Int, Int), t2: (Int, Int)) => {
            (t1._1 + t2._1, t2._2 + t2._2)
          }
        ).mapValues {
          t => t._1 / t._2
        }.collect().foreach(println)
    */


    // TODO   (6 / 10)
    rdd1.combineByKey(
      v => {
        v
      },
      (t: Int, v) => {
        t + v
      },
      (t1: Int, t2: Int) => {
        t1 + t2
      }
      )

        //4.关闭连接
        sc.stop()

  }


}
```

##### 几种WordCount比较

![1658231861437](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658231861437.png)

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}


// 2022-07-17 --- 下午
object Scala19_Transform_test {
  def main(args: Array[String]): Unit = {

    //1.创建SparkConf并设置App名称
    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")

    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)

    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(
        ("a", 1), ("a", 4), ("b", 8), ("c", 9),
        ("c", 3), ("d", 3), ("a", 4), ("b", 5)
      ),
      2

    )

    rdd1.reduceByKey(_+_)
    // 有combiner 操作， 对相同的key 的value 进行操作，不能设置初始值
    rdd1.aggregateByKey(0)(_+_, _+_)
      // 能设置初始值， 分区之间与分区内逻辑可以不一样
    rdd1.foldByKey(0)(_+_)
      // 可以设置初始值， 分区之间 与 之内的逻辑一样的时候使用
    rdd1.combineByKey(v=>v, (x:Int, v) => {x+v}, (x: Int, y:Int) => x+y )
      // 可以在函数内对数据进行预处理

    /*
    第一个参数： TODO 数据转换
    第一个参数： TODO 分区内的计算逻辑
    第一个参数： TODO 分区之间的计算逻辑


    */
    

        //4.关闭连接
        sc.stop()

  }


}
```

##### sortByKey

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}


// 2022-07-17 --- 下午
object Scala20_Transform {
  def main(args: Array[String]): Unit = {

    //1.创建SparkConf并设置App名称
    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")

    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)

    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(
        ("a", 1), ("a", 4), ("b", 8), ("c", 9),
        ("c", 3), ("d", 3), ("a", 4), ("b", 5)
      ),
      2

    )

    //    TODO： sortByKey算子：将 key 进行排序

    val newRDD: RDD[(String, Int)] = rdd1.sortByKey()
    newRDD.collect().foreach(println)


    //4.关闭连接
    sc.stop()

  }


}
```

##### mapValues

![1658231957502](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658231957502.png)

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}


// 2022-07-17 --- 下午
object Scala21_Transform {
  def main(args: Array[String]): Unit = {

    //1.创建SparkConf并设置App名称
    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")

    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)

    val rdd1: RDD[(String, Int)] = sc.makeRDD(
      List(
        ("a", 1), ("a", 4), ("b", 8), ("c", 9),
        ("c", 3), ("d", 3), ("a", 4), ("b", 5)
      ),
      2

    )

    rdd1.mapValues(_*2).collect().foreach(println)

    //4.关闭连接
    sc.stop()

  }


}
```



##### WordCount第一种方法

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}


// 2022-07-17 --- 下午
object Scala22_Requrment {
  def main(args: Array[String]): Unit = {

    //1.创建SparkConf并设置App名称
    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")

    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)


    // TODO 省份  + 点击广告(WordCount)   TOP3
    //  一行：时间戳 省份 城市 用户 广告;
    //    TODO 1. 读数据

    // TODO 问题1， groupby 只是分组用groupby， 有后续操作 选择使用 reducebykey
    //          2. 大量的聚合操作都是在单节点中完成， 不适合分布式环境；

    val lineDatas: RDD[String] = sc.textFile("data/data.txt")

    // TODO 2. 将相同省份的数据进行分组
    val prevGroupRDD: RDD[(String, Iterable[String])] = lineDatas.groupBy(
      line => {
        val datas: Array[String] = line.split(" ")
        datas(1)
      }
    )

    // 3. 在组内对广告点击次数 进行统计
    val top3: RDD[(String, List[(String, Int)])] = prevGroupRDD.mapValues(
      list => {
        val adToOne: Iterable[(String, Int)] = list.map(
          line => {
            val datas: Array[String] = line.split(" ")
            (datas(4), 1)
          }
        )


        val adToLists: Map[String, Iterable[(String, Int)]] = adToOne.groupBy(_._1)
        val adToCounts: Map[String, Int] = adToLists.mapValues(_.size)
        // 统计广告的点击次数后， 对数据进行排序
        adToCounts.toList.sortBy(_._2)(Ordering.Int.reverse).take(3)

      }
    )
    top3.collect().foreach(println)


    //4.关闭连接
    sc.stop()

  }

}
```

##### 小练习优化

```scala
package rdd.opration.transform2

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}


// 2022-07-17 --- 下午
object Scala22_Requrment_1 {
  def main(args: Array[String]): Unit = {

    //1.创建SparkConf并设置App名称
    val conf: SparkConf = new SparkConf().setAppName("SparkCoreTest").setMaster("local[*]")

    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)


    // TODO 省份  + 点击广告(WordCount)   TOP3
    //  一行：时间戳 省份 城市 用户 广告;
    //         0     1   2   3   4

    // TODO 先聚合再分组, 将数据聚合，减少数据量
    //  先统计广告点击次数， 再按照省份分组；


    //    TODO 1. duqu 读数据
    val lineDatas: RDD[String] = sc.textFile("data/data.txt")

    val reduceRDD: RDD[((String, String), Int)] = lineDatas.map(
      line => {
        val datas: Array[String] = line.split(" ")
        ((datas(1), datas(4)), 1)
      }
    ).reduceByKey(_ + _)

    // 将统计结果转换结构后 按照省份分组
      //  使用模式匹配，改变数据结构

    val mapRDD: RDD[(String, Iterable[(String, Int)])] = reduceRDD.map {
      case ((prv, ad), cnt) => {
        (prv, (ad, cnt))
      }
    }.groupByKey()

    // 将组内的数据进行排序 取得前三名
    val top3: RDD[(String, List[(String, Int)])] = mapRDD.mapValues(
      iter => {
        iter.toList.sortBy(_._2)(Ordering.Int.reverse).take(3)
      }
    )
    top3.collect().foreach(println)

    //4.关闭连接
    sc.stop()

  }


}
```

#### Scala三个重点

1. 函数式编程

1. 集合函数
2. 模式匹配（用的多）