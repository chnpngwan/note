![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656124279367.png)

# 第1章 实时数仓同步数据

```properties
f1.sh: 启动上游flume， 监控log变化， 将数据写到kafka

f2.sh: 启动下游监控日志消费的flume， 将数据从kafka读出来，写入hdfs
```

实时数仓由Flink源源不断从Kafka当中读数据计算，所以不需要手动同步数据到实时数仓。

# 第2章 离线数仓同步数据

## 2.1 用户行为数据同步

### 2.1.1 数据通道

​		用户行为数据由Flume从Kafka直接同步到HDFS，由于离线数仓采用Hive的分区表按天统计，所以目标路径要包含一层日期。具体数据流向如下图所示。

![1656463313246](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656463313246.png)

- 拦截器在sorce中定义书写

## 2.1.2 日志消费Flume配置概述

​		按照规划，该Flume需将Kafka中topic_log的数据发往HDFS。并且对每天产生的用户行为日志进行区分，将不同天的数据发往HDFS不同天的路径。
​		此处选择KafkaSource、FileChannel、HDFSSink。
​		关键配置如下：

![1656463821004](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656463821004.png)

### 2.1.3 日志消费Flume配置实操

1）创建Flume配置文件
在hadoop104节点的Flume的job目录下创建kafka_to_hdfs_log.conf

- a1.sources.r1.kafka.consumer.group.id = kafka_to_dfs_log

- 小文件问题：1  影响NameNode健康
- 2；  启动过多maptask占用过大的内存
  - 设置rollinterval：设置平均多久产生rollsize（128M）， 将时间间隔设置为平均时间【让数据尽可能的等于块大小】

- 配置文件内容

```properties
#定义组件
a1.sources=r1
a1.channels=c1
a1.sinks=k1

#配置source1
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000
a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
a1.sources.r1.kafka.topics=topic_log
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.atguigu.gmall.flume.interceptor.TimestampInterceptor$Builder

#配置channel
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /opt/module/flume/checkpoint/behavior1
a1.channels.c1.dataDirs = /opt/module/flume/data/behavior1
a1.channels.c1.maxFileSize = 2146435071
a1.channels.c1.capacity = 1000000
a1.channels.c1.keep-alive = 6

#配置sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /origin_data/gmall/log/topic_log/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = log
a1.sinks.k1.hdfs.round = false


a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

#控制输出文件类型   CompressedStream: 压缩
a1.sinks.k1.hdfs.fileType = CompressedStream
a1.sinks.k1.hdfs.codeC = gzip

#组装 
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

**注：配置优化**
1）FileChannel优化
		通过配置dataDirs指向多个路径，每个路径对应不同的硬盘，增大Flume吞吐量。
官方说明如下：

```
Comma separated list of directories for storing log files. Using multiple directories on separate disks can improve file channel peformance
```

​		checkpointDir和backupCheckpointDir也尽量配置在不同硬盘对应的目录中，保证checkpoint坏掉后，可以快速使用backupCheckpointDir恢复数据
2）HDFS Sink优化
（1）HDFS存入大量小文件，有什么影响？
​		元数据层面：每个小文件都有一份元数据，其中包括文件路径，文件名，所有者，所属组，权限，创建时间等，这些信息都保存在Namenode内存中。所以小文件过多，会占用Namenode服务器大量内存，影响Namenode性能和使用寿命
​		计算层面：默认情况下MR会对每个小文件启用一个Map任务计算，非常影响计算性能。同时也影响磁盘寻址时间。
​	（2）HDFS小文件处理
​		官方默认的这三个参数配置写入HDFS后会产生小文件，hdfs.rollInterval、hdfs.rollSize、hdfs.rollCount
基于以上hdfs.rollInterval=3600，hdfs.rollSize=134217728，hdfs.rollCount =0几个参数综合作用，效果如下：
​		（1）文件在达到128M时会滚动生成新文件
​		（2）文件创建超3600秒时会滚动生成新文件

3）编写Flume拦截器
（1）数据漂移问题

![1656468566445](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656468566445.png)

- 拦截器类：在com.atguigu.gmall.flume.interceptor包下创建TimestampInterceptor类

```java
package com.atguigu.gmall.flume.interceptor;

import com.alibaba.fastjson.JSONObject;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.nio.charset.StandardCharsets;
import java.util.List;
import java.util.Map;

public class TimestampInterceptor implements Interceptor {
    @Override
    public void initialize() {

    }


    /*
    * 将body数据中的 ts 字段 放到header中的timestamp中
    * */
    @Override
    public Event intercept(Event event) {

        // 获取header与body

        byte[] body = event.getBody();
        String log = new String(body, StandardCharsets.UTF_8);
        Map<String, String> headers = event.getHeaders();

        // 解析body中的ts
        JSONObject jsonObject = JSONObject.parseObject(log);
        String ts = jsonObject.getString("ts");

        // 到ts字段归为timestamp
        headers.put("timestamp", ts);


        return event;
    }

    @Override
    public List<Event> intercept(List<Event> list) {

        for (Event event : list) {
            intercept(event);
        }
        return list;
    }

    @Override
    public void close() {

    }

    public static class Builder implements Interceptor.Builder{

        @Override
        public Interceptor build() {
            return new TimestampInterceptor();
        }

        @Override
        public void configure(Context context) {

        }
    }
}

```

![1656492646857](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656492646857.png)

![1656492664826](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656492664826.png)

![1656492673579](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656492673579.png)

- 首先要启动fi.sh 监控log文件的变化， 将变化情况写到kafka中

![1656492768366](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656492768366.png)

```shell
#!/bin/bash

case $1 in
"start")
        echo " --------启动 hadoop104 日志数据flume-------"
        ssh hadoop104 "nohup /opt/module/flume/bin/flume-ng agent -n a1 -c /opt/module/flume/conf -f /opt/module/flume/job/kafka_to_hdfs_log.conf >/dev/null 2>&1 &"
;;
"stop")

        echo " --------停止 hadoop104 日志数据flume-------"
        ssh hadoop104 "ps -ef | grep kafka_to_hdfs_log | grep -v grep |awk '{print \$2}' | xargs -n1 kill"
;;
esac
```

![1656492791265](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656492791265.png)

- f2.sh是日志消费的fkume脚本， 负责将kafka中的数据读入到hdfs



## 2.2 业务数据同步

### 2.2.1 数据同步策略概述

​		业务数据是数据仓库的重要数据来源，我们需要每日定时从业务数据库中抽取数据，传输到数据仓库中，之后再对数据进行分析统计。
​		为保证统计结果的正确性，需要保证数据仓库中的数据与业务数据库是同步的，离线数仓的计算周期通常为天，所以数据同步周期也通常为天，即每天同步一次即可。
​		数据的同步策略有全量同步和增量同步。
​		全量同步，就是每天都将业务数据库中的全部数据同步一份到数据仓库，这是保证两侧数据同步的最简单的方式。

![1656471727981](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656471727981.png)

![1656471738130](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656471738130.png)

### 2.2.2 数据同步策略选择

​	两种策略都能保证数据仓库和业务数据库的数据同步，那应该如何选择呢？下面对两种策略进行简要对比。

| 同步策略 | 优点                 | 缺点                                               |
| ------------------ | ------------------------------ | ------------------------------------------------------------ |
| 全量同步 | 逻辑简单                       | 在某些情况下效率较低。例如某张表数据量较大，但是每天数据的变化比例很低，若对其采用每日全量同步，则会重复同步和存储大量相同的数据。 |
| 增量同步 | 效率高，无需同步和存储重复数据 | 逻辑复杂，需要将每日的新增及变化数据同原来的数据进行整合，才能使用 |

​		两种策略都能保证数据仓库和业务数据库的数据同步，那应该如何选择呢？下面对两种策略进行简要对比。
根据上述对比，可以得出以下结论：
​		通常情况，业务表数据量比较大，优先考虑增量，数据量比较小，优先考虑全量；具体选择由数仓模型决定，此处暂不详解。 
​		下图为各表同步策略：

![1656472636025](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656472636025.png)

------------------------

​								维度表																			事实表

### 2.2.3 数据同步工具概述

​		数据同步工具种类繁多，大致可分为两类，一类是以DataX、Sqoop为代表的基于Select查询的离线、批量同步工具，另一类是以Maxwell、Canal为代表的基于数据库数据变更日志（例如MySQL的binlog，其会实时记录所有的insert、update以及delete操作）的实时流式同步工具。
​		全量同步通常使用DataX、Sqoop等基于查询的离线同步工具。而增量同步既可以使用DataX、Sqoop等工具，也可使用Maxwell、Canal等工具，下面对增量同步不同方案进行简要对比。

| 增量同步方案   | DataX/Sqoop                           | Maxwell/Canal                  |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 对数据库的要求 | 原理是基于查询，故若想通过select查询获取新增及变化数据，就要求数据表中存在create_time、update_time等字段，然后根据这些字段获取变更数据。 | 要求数据库记录变更操作，例如MySQL需开启binlog。              |
| 数据的中间状态 | 由于是离线批量同步，故若一条数据在一天中变化多次，该方案只能获取最后一个状态，中间状态无法获取。 | 由于是实时获取所有的数据变更操作，所以可以获取变更数据的所有中间状态。 |

### 2.2.5 全量表数据同步

#### 2.2.5.1 数据同步工具DataX部署

# 第1章 DataX简介

## 1.1 DataX概述

​	DataX 是阿里巴巴开源的一个异构数据源离线同步工具，致力于实现包括关系型数据库(MySQL、Oracle等)、HDFS、Hive、ODPS、HBase、FTP等各种异构数据源之间稳定高效的数据同步功能。
源码地址：https://github.com/alibaba/DataX

## 1.2 DataX支持的数据源

​		DataX目前已经有了比较全面的插件体系，主流的RDBMS数据库、NOSQL、大数据计算系统都已经接入，目前支持数据如下图。

![1656493109148](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656493109148.png)

![1656493119310](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656493119310.png)

# 第2章 DataX架构原理

## 2.1 DataX设计理念

​		为了解决异构数据源同步问题，DataX将复杂的网状的同步链路变成了星型数据链路，DataX作为中间传输载体负责连接各种数据源。当需要接入一个新的数据源的时候，只需要将此数据源对接到DataX，便能跟已有的数据源做到无缝数据同步。

![1656493145732](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656493145732.png)

## 2.2 DataX框架设计

​		DataX本身作为离线数据同步框架，采用Framework + plugin架构构建。将数据源读取和写入抽象成为Reader/Writer插件，纳入到整个同步框架中。

![1656493184469](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656493184469.png)

## 2.3 DataX运行流程

​		下面用一个DataX作业生命周期的时序图说明DataX的运行流程、核心概念以及每个概念之间的关系。

![1656499786382](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656499786382.png)

## 2.4 DataX调度决策思路

​		举例来说，用户提交了一个DataX作业，并且配置了总的并发度为20，目的是对一个有100张分表的mysql数据源进行同步。DataX的调度决策思路是：
​		1）DataX Job根据分库分表切分策略，将同步工作分成100个Task。
​		2）根据配置的总的并发度20，以及每个Task Group的并发度5，DataX计算共需要分配4个TaskGroup。
​		3）4个TaskGroup平分100个Task，每一个TaskGroup负责运行25个Task。

![1656499854338](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656499854338.png)

# 第3章 DataX部署

1）下载DataX安装包并上传到hadoop102的/opt/software

![1656499897742](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656499897742.png)

# 第4章 DataX使用

## 4.1 DataX使用概述

### 4.1.1 DataX任务提交命令

​		DataX的使用十分简单，用户只需根据自己同步数据的数据源和目的地选择相应的Reader和Writer，并将Reader和Writer的信息配置在一个json文件中，然后执行如下命令提交数据同步任务即可。

`[atguigu@hadoop102 datax]$ python bin/datax.py path/to/your/job.json`

### 4.2.2 DataX配置文件格式

​		可以使用如下命名查看DataX配置文件模板。
`[atguigu@hadoop102 datax]$ python bin/datax.py -r mysqlreader -w hdfswriter`
​		配置文件模板如下，json最外层是一个job，job包含setting和content两部分，其中setting用于对整个job进行配置，content用户配置数据源和目的地。

![1656500109438](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500109438.png)

## 4.2 同步MySQL数据到HDFS案例

​		案例要求：同步gmall数据库中base_province表数据到HDFS的/base_province目录
​		需求分析：要实现该功能，需选用MySQLReader和HDFSWriter，MySQLReader具有两种模式分别是TableMode和QuerySQLMode，前者使用table，column，where等属性声明需要同步的数据；后者使用一条SQL查询语句声明需要同步的数据。
下面分别使用两种模式进行演示。

## 4.2.1 MySQLReader之TableMode

1）编写配置文件
（1）创建配置文件base_province.json
`[atguigu@hadoop102 ~]$ vim /opt/module/datax/job/base_province.json`
（2）配置文件内容如下

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "column": [
                            "id",
                            "name",
                            "region_id",
                            "area_code",
                            "iso_code",
                            "iso_3166_2"
                        ],
                        "where": "id>=3",
                        "connection": [
                            {
                                "jdbcUrl": [
                                    "jdbc:mysql://hadoop102:3306/gmall"
                                ],
                                "table": [
                                    "base_province"
                                ]
                            }
                        ],
                        "password": "000000",
                        "splitPk": "",
                        "username": "root"
                    }
                },
                "writer": {
                    "name": "hdfswriter",
                    "parameter": {
                        "column": [
                            {
                                "name": "id",
                                "type": "bigint"
                            },
                            {
                                "name": "name",
                                "type": "string"
                            },
                            {
                                "name": "region_id",
                                "type": "string"
                            },
                            {
                                "name": "area_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_3166_2",
                                "type": "string"
                            }
                        ],
                        "compress": "gzip",
                        "defaultFS": "hdfs://hadoop102:8020",
                        "fieldDelimiter": "\t",
                        "fileName": "base_province",
                        "fileType": "text",
                        "path": "/base_province",
                        "writeMode": "append"
                    }
                }
            }
        ],
        "setting": {
            "speed": {
                "channel": 1
            }
        }
    }
}
```

2）配置文件说明
（1）Reader参数说明

![1656500289091](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500289091.png)

![1656500299669](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500299669.png)

**注意事项：**
		HFDS Writer并未提供nullFormat参数：也就是用户并不能自定义null值写到HFDS文件中的存储格式。默认情况下，HFDS Writer会将null值存储为空字符串（''），而Hive默认的null值存储格式为\N。所以后期将DataX同步的文件导入Hive表就会出现问题。
**解决该问题的方案有两个：**
		一是修改DataX HDFS Writer的源码，增加自定义null值存储格式的逻辑，可参考https://blog.csdn.net/u010834071/article/details/105506580。
二是在Hive中建表时指定null值存储格式为空字符串（''），例如：

```shell
DROP TABLE IF EXISTS base_province;
CREATE EXTERNAL TABLE base_province
(
    `id`         STRING COMMENT '编号',
    `name`       STRING COMMENT '省份名称',
    `region_id`  STRING COMMENT '地区ID',
    `area_code`  STRING COMMENT '地区编码',
    `iso_code`   STRING COMMENT '旧版ISO-3166-2编码，供可视化使用',
    `iso_3166_2` STRING COMMENT '新版IOS-3166-2编码，供可视化使用'
) COMMENT '省份表'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/base_province/';
```

（3）Setting参数说明

![1656500476263](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500476263.png)

![1656500505766](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500505766.png)

```tex
2021-10-13 11:13:14.930 [job-0] INFO  JobContainer - 
任务启动时刻                    : 2021-10-13 11:13:03
任务结束时刻                    : 2021-10-13 11:13:14
任务总计耗时                    :                 11s
任务平均流量                    :               66B/s
记录写入速度                    :              3rec/s
读出记录总数                    :                  32
读写失败总数                    :                   0
```

![1656500578620](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500578620.png)

### 4.2.2 MySQLReader之QuerySQLMode

1）编写配置文件

![1656500615333](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500615333.png)

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "connection": [
                            {
                                "jdbcUrl": [
                                    "jdbc:mysql://hadoop102:3306/gmall"
                                ],
                                "querySql": [
                                    "select id,name,region_id,area_code,iso_code,iso_3166_2 from base_province where id>=3"
                                ]
                            }
                        ],
                        "password": "000000",
                        "username": "root"
                    }
                },
                "writer": {
                    "name": "hdfswriter",
                    "parameter": {
                        "column": [
                            {
                                "name": "id",
                                "type": "bigint"
                            },
                            {
                                "name": "name",
                                "type": "string"
                            },
                            {
                                "name": "region_id",
                                "type": "string"
                            },
                            {
                                "name": "area_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_3166_2",
                                "type": "string"
                            }
                        ],
                        "compress": "gzip",
                        "defaultFS": "hdfs://hadoop102:8020",
                        "fieldDelimiter": "\t",
                        "fileName": "base_province",
                        "fileType": "text",
                        "path": "/base_province",
                        "writeMode": "append"
                    }
                }
            }
        ],
        "setting": {
            "speed": {
                "channel": 1
            }
        }
    }
}
```

![1656500663065](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500663065.png)

![1656500689122](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500689122.png)

![1656500702356](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500702356.png)

### 4.2.3 DataX传参

​		通常情况下，离线数据同步任务需要每日定时重复执行，故HDFS上的目标路径通常会包含一层日期，以对每日同步的数据加以区分，也就是说每日同步数据的目标路径不是固定不变的，因此DataX配置文件中HDFS Writer的path参数的值应该是动态的。为实现这一效果，就需要使用DataX传参的功能。
​		DataX传参的用法如下，在JSON配置文件中使用**${param}**引用参数，在提交任务时使用**-p"-Dparam=value"**传入参数值，具体示例如下。
1）编写配置文件
（1）修改配置文件base_province.json
`[atguigu@hadoop102 ~]$ vim /opt/module/datax/job/base_province.json`
（2）配置文件内容如下

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "connection": [
                            {
                                "jdbcUrl": [
                                    "jdbc:mysql://hadoop102:3306/gmall"
                                ],
                                "querySql": [
                                    "select id,name,region_id,area_code,iso_code,iso_3166_2 from base_province where id>=3"
                                ]
                            }
                        ],
                        "password": "000000",
                        "username": "root"
                    }
                },
                "writer": {
                    "name": "hdfswriter",
                    "parameter": {
                        "column": [
                            {
                                "name": "id",
                                "type": "bigint"
                            },
                            {
                                "name": "name",
                                "type": "string"
                            },
                            {
                                "name": "region_id",
                                "type": "string"
                            },
                            {
                                "name": "area_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_code",
                                "type": "string"
                            },
                            {
                                "name": "iso_3166_2",
                                "type": "string"
                            }
                        ],
                        "compress": "gzip",
                        "defaultFS": "hdfs://hadoop102:8020",
                        "fieldDelimiter": "\t",
                        "fileName": "base_province",
                        "fileType": "text",
                        "path": "/base_province/${dt}",
                        "writeMode": "append"
                    }
                }
            }
        ],
        "setting": {
            "speed": {
                "channel": 1
            }
        }
    }
}
```

![1656500826317](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500826317.png)

![1656500842856](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1656500842856.png)





































