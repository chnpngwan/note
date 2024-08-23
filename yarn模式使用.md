##### 笔记

1. 进程与线程，
   1. 进程有机会抢占CPU， 分配内存资源
   2. 线程是进程的一部分
   3. 启动一个java程序等同于启动一个进行， 进程名字是类名
   4. local： 本机， 当前进程能够获得的资源：cpu +内存 
   5. sc: Spark Context

##### Spark（计算） On Yarn（资源）

![1657707760819](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1657707760819.png)

1. 计算任务（task）： 计算逻辑+ 数据
2. 计算是独立性较强的操作，完成计算自动关闭， 资源一致运行， 提供服务

原子操作：原子性，==> 事务

- java虚拟机中的long 与 double （32 + 32）： 计算64位的时候 分两步， 计算两个32位的操作， 之后整合起来， 虚拟机只能处理32位的数据

- 埋点数据：
- 高可用： master故障后， 集群可用
- 高可用中选举机制， 网络延迟等原因， 可能出现多个master， 脑裂现象

##### SparkonSpark

![1657707814693](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1657707814693.png)

![1657707826043](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1657707826043.png)





- 注意yarn配置文件的路径

- map：一一对应， 不可能减少

##### Master与Worker的区别

![1657708099141](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1657708099141.png)

##### Driver与Excluder

![1657708145152](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1657708145152.png)

##### WordCount代码解析

1. 绝对路径与相对路径
2. 对偶元组
3. mapvalues：只处理二元组的values， key不变， 处理values

```scala
package com.atguigu.bigdata.spark.core.word.wordcount

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
object WordCount03 {
  def main(args: Array[String]): Unit = {

    // 1. 创建配置文件， 集群， 程序名称
    val conf: SparkConf = new SparkConf().setMaster("local[2]").setAppName("SparkEnv")
    // SparkContext环境对象， 可以简单理解为Driver对象
    val sc = new SparkContext(conf)

    // 2. 读取文件, 如果要使用分布式， 就要使用框架提供的功能， 而不是自己开发
    // 相对路径与绝对路径的概念：
    //    绝对路径： 不可改变的路径
    //              协议：// IP: 端口号/路径
    //                http://192.11.21.21:80/test/test.html
    //                file:///opt/model/xxxx
    //                file://c:/test/test.xml；
    //    相对路径： 可以改变的路径， 存在基准路径，
    //              IDEA中的基准路径：是项目的根路径
    //              Local中的基准路径： Spark的解压缩目录；
    val fileDatas: RDD[String] = sc.textFile("data/word.txt")

    // 3. 对读取到的信息 进行处理， 扁平化操作，
    val wordDatas: RDD[String] = fileDatas.flatMap(_.split(" "))

    // 对处理后的元组进行处理
    val wordGroup: RDD[(String, Iterable[String])] = wordDatas.groupBy(word => word)
    //        fileDatas.flatMap(
    //            (line:String) => {
    //                line.split(" ")
    //            }
    //        )
    //
    //        fileDatas.flatMap(
    //            (line:String) => line.split(" ")
    //        )
    //        fileDatas.flatMap(
    //            (line) => line.split(" ")
    //        )
    //        fileDatas.flatMap(
    //            line => line.split(" ")
    //        )
    //        fileDatas.flatMap(_.split(" "))

    // TODO 将相同的单词分在一个组内
    // 分组后的数据是：
    // (word, list(word, word, word, word))

    // map: 转换映射，  A-> B： 一一对应，
    //    val wordCount: RDD[(String, Int)] = wordGroup.map(
    //      {
    //        t => {
    //          (t._1, t._2.size)
    //        }
    //      }
    //    )


    // 两个元素的元组成为 对偶元组， 也成为 k- v 键值对
    // 如果数据转换的时候， 数据为键值对， 只针对 v 改变， 可以采用特殊方法: mapValues；

    //    wordGroup.mapValues(_.size)

    val wordCount: RDD[(String, Int)] = wordGroup.map {
      case (word, list) => {
        (word, list.size)
      }
    }

    wordCount.collect().foreach(println)

    sc.stop()


  }
}
```

##### Spark中函数

1.  `mapDatas.reduceByKey(_+_)`  ： 一步到位， 完成相同key值中的value操作

```scala
package com.atguigu.bigdata.spark.core.word.wordcount

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

object wordCount04 {
  def main(args: Array[String]): Unit = {

    // 1. 创建配置文件， 集群， 程序名称
    val conf: SparkConf = new SparkConf().setMaster("local[2]").setAppName("SparkEnv")
    // SparkContext环境对象， 可以简单理解为Driver对象
    val sc = new SparkContext(conf)
    val fileDatas: RDD[String] = sc.textFile("data/word.txt")

    // 3. 对读取到的信息 进行处理， 扁平化操作，
    val wordDatas: RDD[String] = fileDatas.flatMap(_.split(" "))

    // (word, 1), (word, 1),  (word, 1),  (word, 1)
    val mapDatas: RDD[(String, Int)] = wordDatas.map(
      word => (word, 1)
    )
    val woordcount: RDD[(String, Int)] = mapDatas.reduceByKey(_+_)


    // 对处理后的元组进行处理
    val wordCount: RDD[(String, Iterable[String])] = wordDatas.groupBy(word => word)

    wordCount.collect().foreach(println)

    sc.stop()

  }
}
```

##### Spark中的Standalone模式步骤

2）再解压一份Spark安装包，并修改解压后的文件夹名称为spark-standalone

```properties
[atguigu@hadoop102 sorfware]$ tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module/
[atguigu@hadoop102 module]$ mv spark-3.0.0-bin-hadoop3.2 spark-standalone
```

3）进入Spark的配置目录/opt/module/spark-standalone/conf
`[atguigu@hadoop102 spark-standalone]$ cd conf`
4）修改slave文件，添加work节点：

```properties
[atguigu@hadoop102 conf]$ mv slaves.template slaves
[atguigu@hadoop102 conf]$ vim slaves
hadoop102
hadoop103
hadoop104
```

5）修改spark-env.sh文件，添加master节点

```properties
[atguigu@hadoop102 conf]$ mv spark-env.sh.template spark-env.sh
[atguigu@hadoop102 conf]$ vim spark-env.sh

SPARK_MASTER_HOST=hadoop102
SPARK_MASTER_PORT=7077
```

6）分发spark-standalone包
`[atguigu@hadoop102 module]$ xsync spark-standalone/`
7）启动spark集群
`[atguigu@hadoop102 spark-standalone]$ sbin/start-all.sh`
查看三台服务器运行进程（xcall.sh是以前数仓项目里面讲的脚本）。

```
[atguigu@hadoop102 spark-standalone]$ xcall.sh jps
================atguigu@hadoop102================
3238 Worker
3163 Master
================atguigu@hadoop103================
2908 Worker
================atguigu@hadoop104================
2978 Worker
```

注意：如果遇到 “JAVA_HOME not set” 异常，可以在sbin目录下的spark-config.sh 文件中加入如下配置。
export JAVA_HOME=XXXX
8）网页查看：hadoop102:8080（master web的端口，相当于yarn的8088端口）
	目前还看不到任何任务的执行信息。
9）官方求PI案例

```bash
[atguigu@hadoop102 spark-standalone]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077 \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

参数：--master spark://hadoop102:7077指定要连接的集群的master。
10）页面查看http://hadoop102:8080/，发现执行本次任务，默认采用三台服务器节点的总核数24核，每个节点		内存1024M。
		8080：master的webUI
		4040：application的webUI的端口号

资源和计算的关系

分布式计算基础架构， Master  ---> Slaver

##### Spark中的yarn模式

0）停止Standalone模式下的spark集群

```basic
[atguigu@hadoop102 spark-standalone]$ sbin/stop-all.sh
[atguigu@hadoop102 spark-standalone]$ zk.sh stop
[atguigu@hadoop103 spark-standalone]$ sbin/stop-master.sh
```

1）为了防止和Standalone模式冲突，再单独解压一份spark
`[atguigu@hadoop102 software]$ tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module/`
2）进入到/opt/module目录，修改spark-3.0.0-bin-hadoop3.2名称为spark-yarn
`[atguigu@hadoop102 module]$ mv spark-3.0.0-bin-hadoop3.2/ spark-yarn`
3）修改hadoop配置文件/opt/module/hadoop-3.1.3/etc/hadoop/yarn-site.xml，添加如下内容
			因为测试环境虚拟机内存较少，防止执行过程进行被意外杀死，做如下配置
			`[atguigu@hadoop102 hadoop]$ vim yarn-site.xml`

```xml
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
     <name>yarn.nodemanager.pmem-check-enabled</name>
     <value>false</value>
</property>

<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
     <name>yarn.nodemanager.vmem-check-enabled</name>
     <value>false</value>
</property>
```

4）分发配置文件
`[atguigu@hadoop102 conf]$ xsync /opt/module/hadoop-3.1.3/etc/hadoop/yarn-site.xml`
5）修改/opt/module/spark-yarn/conf/spark-env.sh，添加YARN_CONF_DIR配置，保证后续运行任务的路径都变成集群路径

```basic
[atguigu@hadoop102 conf]$ mv spark-env.sh.template spark-env.sh
[atguigu@hadoop102 conf]$ vim spark-env.sh

YARN_CONF_DIR=/opt/module/hadoop-3.1.3/etc/hadoop
```

6）启动HDFS以及YARN集群

```basic
[atguigu@hadoop102 hadoop-3.1.3]$ sbin/start-dfs.sh
[atguigu@hadoop103 hadoop-3.1.3]$ sbin/start-yarn.sh
```

7）执行一个程序

```basic
[atguigu@hadoop102 spark-yarn]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

**参数：--master yarn，表示Yarn方式运行；--deploy-mode表示客户端方式运行程序**



##### 简单总结

- 主从架构

RM -- NN

NameNode ---- DataNode

Controller --- Broker   :  kafka

- 分布式特点

分布式计算： 数据规模的切分

HDFS： 切片

Kafka ： Partition

Spark与Hadoop的关系 

Hadoop  MR慢， 数据一次性计算落盘， 

​		Spark基于 MR： 



##### 数据模型

数据结构： 一种管理和组织数据的方式

1. 数组： 连续的内存空间（有限制），

![1657707866429](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1657707866429.png)

2. 链表

![1657707886141](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1657707886141.png)



