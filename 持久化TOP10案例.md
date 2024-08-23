##### 笔记

1. 方法的重载，多个不同的参数，更加灵活
2. 方法的重写，原有的方法不满足需求，重写拓展功能

- 有向无环图：数据有向无环，

  1）DAG有向无环图
  		DAG（Directed Acyclic Graph）有向无环图是由点和线组成的拓扑图形，该图形具有方向，不会闭环。例如，DAG记录了RDD的转换过程和任务的阶段。

![1658279495864](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658279495864.png)

![1658316331315](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658316331315.png)

![1658316341127](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658316341127.png)

![1658316369569](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658316369569.png)



##### 一个JOB中阶段数量如何确定

- 阶段数量 = shuffle依赖数量（shuffleMapStage） + 1（）

##### 一个阶段中任务的数量如何确定

- 阶段中最后一个RDD中的分区的数量

##### 一个作业中任务的数量如何确定

- 每一个阶段中最后一个RDD中的分区数量之和

**3）RDD任务切分中间分为：Application、Job、Stage和Task**
  		（1）Application：初始化一个SparkContext即生成一个Application；
  		（2）Job：一个Action算子就会生成一个Job；
  		（3）Stage ：Stage等于宽依赖的个数加1；
  		（4）Task：一个Stage阶段中，最后一个RDD的分区个数就是Task的个数。
  		注意：Application->Job->Stage->Task每一层都是1对n的关系。

##### 分区器相关

- 两次调用相同的分区器的时候， 原码内部会判断， 如果两个分区器相等，返回自身，不会重新分区

##### 埋点

- 在页面上触发了某个事件，向服务器发送数据

##### Spark中的RDD文件读写

1. 读文件的时候，object 与 seqynce 要写上泛型

```scala
    //2.创建SparkContext，该对象是提交Spark App的入口
    val sc: SparkContext = new SparkContext(conf)


    val rdd0: RDD[(String, Int)] = sc.makeRDD(
      List(("a", 1), ("b", 2)),
      2
    )
    /*    rdd0.saveAsTextFile("out1")
        rdd0.saveAsObjectFile("out2")
        rdd0.saveAsSequenceFile("out3")
        */

    // 注意 写上泛型， 读取文件要知道文件的格式
    println(sc.textFile("out1").collect().mkString(", "))
    println(sc.objectFile[(String, Int)]("out2").collect().mkString(", "))
    println(sc.sequenceFile[String, Int]("out3").collect().mkString(", "))

    rdd0.sortBy(_._1)
```

##### RDD持久化

![1658316400150](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658316400150.png)

![1658316409207](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1658316409207.png)

1. cache
2. persist： 可以选择存放的位置

```scala
    val rdd0: RDD[String] = sc.makeRDD(
      List("Hello Spark", "Hello Scala"),
      2
    )

    val rdd1: RDD[String] = rdd0.flatMap(_.split(" "))

    val rdd2: RDD[(String, Int)] = rdd1.map(
      word => {
        println("+++++++++++")
        (word, 1)
      }
    )

    // TODO 持久化：  --- 缓存:放在内存中
    // rdd2.cache()

    // TODO persist: 持久化
    // rdd2.persist()

    //    选择存储级别
    //      堆内内存： 受到Java内存管理的内存
    //      堆外内存 ：OFF_HEAP: 不受到Java虚拟机管理的内存；
    //      非堆内存：方法区，栈内存
    //      cache 与 persist 只对当前的应用有效，;
    rdd2.persist(StorageLevel.OFF_HEAP)


    val rdd21: RDD[(String, Int)] = rdd2.reduceByKey(_ + _)
    rdd21.saveAsTextFile("output1")

    println("-----------------")
    val rdd22 = rdd2.groupByKey()
    rdd22.saveAsTextFile("output2")
```

##### 检查点文件

1. `rdd2.checkpoint()`：一般与 cache 联合使用
2. 设置 检查点文件的存储位置

```scala

    // 设置检查点文件存储位置
    sc.setCheckpointDir("cp")

    val rdd0: RDD[String] = sc.makeRDD(
      List("Hello Spark", "Hello Scala"),
      2
    )

    val rdd1: RDD[String] = rdd0.flatMap(_.split(" "))

    val rdd2: RDD[(String, Int)] = rdd1.map(
      word => {
        println("+++++++++++")
        (word, 1)
      }
    )

    // TODO 持久化： 检查点文件
    // 报错： Checkpoint directory has not been set in the SparkContext
    // 检查点文件一般设置为分布式存储系统，存在 HDFS 中
    // 检查点与 cache 一般联合使用： 将结果存在缓存里边， 检查点文件执行的时候会使用缓存，速度快一点;
    rdd2.cache()
    rdd2.checkpoint()

    rdd2.persist(StorageLevel.OFF_HEAP)


    val rdd21: RDD[(String, Int)] = rdd2.reduceByKey(_ + _)
    rdd21.saveAsTextFile("output1")

    println("-----------------")
    val rdd22 = rdd2.groupByKey()
    rdd22.saveAsTextFile("output2")
```

##### cache与检查点文件的区别

1. cache会增加依赖，通过血缘关系找到文件
2. 检查点文件会直接切断血缘关系

```scala
    rdd2.cache()
    // rdd2.checkpoint()
    // cach 会增加依赖，通过血缘关系可以找到 cach 依赖的位置，直接返回
    // 检查点文件会切断血缘关系， 在 HDFS 直接将检查点当做数据源，减少依赖关系，会切断血缘关系
    //  当一个 RDD 需要重复使用的时候 考虑加 cach ;


    // 如果确定缓存之后不需要了 可以释放资源
    rdd2.unpersist()
```

##### 案例001-cogroup-热点广告TOP10

1. 过滤出三类数据
2. cogroup全连接数据
3. 转化数据格式去前十个

```scala
    // TODO 读取数据， 获取原始数据
    val datasRDD: RDD[String] = sc.textFile("data/user_visit_action.txt")

    //  TODO 对数据进行统计分析
    //      1. 统计点击数量(先过滤)
    //      2. 统计下单数量
    //      3. 统计支付数量
    // TODO 对数据进行排序，统计前10名；

    // filter 与 map 没有先后顺序， 切分不能优化
    val clickCnt: RDD[(String, Int)] = datasRDD.filter(
      data => {
        val datas: Array[String] = data.split("_")
        datas(6) != "-1"
      }
    ).map(
      data => {
        val datas: Array[String] = data.split("_")
        (datas(6), 1)
      }
    ).reduceByKey(_ + _)
    // clickCnt.collect().foreach(println)


    // 下单
    val orderCnt: RDD[(String, Int)] = datasRDD.filter(
      data => {
        val datas: Array[String] = data.split("_")
        datas(8) != "null"
      }
    ).flatMap(
      data => {
        val datas: Array[String] = data.split("_")
        val ids: Array[String] = datas(8).split(",")
        ids.map((_, 1))
      }
    ).reduceByKey(_ + _)
    // orderCnt.collect().foreach(println)


    // 支付
    val payCnt: RDD[(String, Int)] = datasRDD.filter(
      data => {
        val datas: Array[String] = data.split("_")
        datas(10) != "null"
      }
    ).flatMap(
      data => {
        val datas: Array[String] = data.split("_")
        val ids: Array[String] = datas(10).split(",")
        ids.map((_, 1))
      }
    ).reduceByKey(_ + _)

    // TODO 获取前十名
    // val top10: Array[(String, Int)] = clickCnt.sortBy(_._2,false).take(10)
    // top10.foreach(println)

    // （品类， 点击数量）；
    // （品类， 下单数量）；
    // （品类， 支付数量）；
    // （品类， (点击， 下单，支付))  ；
    // 使用 join 可以连接==> (笛卡尔积， shuffle， 两个数据集有相同的 key 才可以连接)
    // 左右连接， 全连接 ======> 使用cogroup；
    val coDatas: RDD[(String, (Iterable[Int], Iterable[Int], Iterable[Int]))] = clickCnt.cogroup(orderCnt, payCnt)

    val mapDatas: RDD[(String, (Int, Int, Int))] = coDatas.mapValues {
      case (clinkIter, orderIter, payIter) => {
        val clickCnt = clinkIter.headOption.getOrElse(0)
        val orderCnt = orderIter.headOption.getOrElse(0)
        val payCnt = payIter.headOption.getOrElse(0)

        (clickCnt, orderCnt, payCnt)
      }
    }
    val top10: Array[(String, (Int, Int, Int))] = mapDatas.sortBy(_._2, false).take(10)
    top10.foreach(println)

```

##### 案例02--直接union

1. 改变数据结构，不使用cogroup， 直接union

```scala
        // TODO RDD如果重复使用，尽量采用持久化操作
        val datasRDD: RDD[String] = sc.textFile("data/user_visit_action.txt")
        datasRDD.cache()

        val clickCnt = datasRDD.filter(
            data => {
                val datas = data.split("_")
                datas(6) != "-1"
            }
        ).map(
            data => {
                val datas = data.split("_")
                (datas(6), 1)
            }
        ).reduceByKey(_+_)

        // TODO 下单数据
        val orderCnt = datasRDD.filter(
            data => {
                val datas = data.split("_")
                datas(8) != "null"
            }
        ).flatMap(
            data => {
                val datas = data.split("_")
                val ids = datas(8).split(",")
                ids.map((_, 1))
            }
        ).reduceByKey(_+_)

        // TODO 支付数据
        val payCnt = datasRDD.filter(
            data => {
                val datas = data.split("_")
                datas(10) != "null"
            }
        ).flatMap(
            data => {
                val datas = data.split("_")
                val ids = datas(10).split(",")
                ids.map((_, 1))
                //(datas(8), 1) // ((1,2,3), 1) => (1,1),(2,1),(3,1)
            }
        ).reduceByKey(_+_)

        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(0, 1，0))
        //                => (品类，(1, 1，0))
        // (品类，1)       =>  (品类，(0, 0，1))
        //                => (品类，(1, 1，1))
        // =>
        // (品类，(1, 1，1))
        clickCnt.map {
            case (cid, cnt) => {
                (cid, (cnt, 0, 0))
            }
        }
        .union(
            orderCnt.map {
                case (cid, cnt) => {
                    (cid, (0, cnt, 0))
                }
            }
        )
        .union(
            payCnt.map {
                case (cid, cnt) => {
                    ( cid, (0, 0, cnt) )
                }
            }
        ).reduceByKey(
            (t1, t2) => {
                (t1._1 + t2._1, t1._2 + t2._2, t1._3 + t2._3)
            }
        ).sortBy(_._2, false).take(10).foreach(println)

        sc.stop()
```

##### 案例升级3-直接转换之后聚合

1. 首先处理数据，不聚合，只转换数据格式
2. 之后统一聚合

```scala
        // TODO RDD如果重复使用，尽量采用持久化操作
        val datasRDD: RDD[String] = sc.textFile("data/user_visit_action.txt")

        datasRDD.flatMap(
            data => {
                val datas = data.split("_")
                if ( datas(6) != "-1" ) {
                    // 点击数据
                    List((datas(6), (1, 0, 0)))
                } else if ( datas(8) != "null" ) {
                    // 下单数据
                    val ids = datas(8).split(",")
                    ids.map(
                        id => {
                            (id, (0, 1, 0))
                        }
                    )
                } else if (datas(10) != "null") {
                    // 支付数据
                    val ids = datas(10).split(",")
                    ids.map(
                        id => {
                            (id, (0, 0, 1))
                        }
                    )
                } else {
                    Nil
                }

            }
        ).reduceByKey(
            (t1, t2) => {
                (t1._1 + t2._1, t1._2 + t2._2, t1._3 + t2._3)
            }
        ).sortBy(_._2, false).take(10).foreach(println)

        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(1, 0，0))
        // (品类，1)       => (品类，(0, 1，0))
        // (品类，1)       => (品类，(0, 1，0))
        // (品类，1)       => (品类，(0, 1，0))
        // (品类，1)       => (品类，(0, 1，0))
        // (品类，1)       => (品类，(0, 1，0))
        // (品类，1)       => (品类，(0, 1，0))
        // (品类，1)       => (品类，(0, 1，0))
        // (品类，1)       => (品类，(0, 1，0))
        //                => (品类，(10, 10，0))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        // (品类，1)       =>  (品类，(0, 0，1))
        //                => (品类，(10, 10，10))
        // =>
        // (品类，(10, 10，10))

        sc.stop()

```









