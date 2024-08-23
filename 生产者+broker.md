## 3.5 生产经验——生产者如何提高吞吐量

![1655859444924](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655859444924.png)

设置参数

```java
public class CustomProducerParameters {

    public static void main(String[] args) throws InterruptedException {

        // 1. 创建kafka生产者的配置对象
        Properties properties = new Properties();

        // 2. 给kafka配置对象添加配置信息：bootstrap.servers
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop102:9092");
        
        // key,value序列化（必须）：key.serializer，value.serializer
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");

        // batch.size：批次大小，默认16K
        properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);

        // linger.ms：等待时间，默认0
        properties.put(ProducerConfig.LINGER_MS_CONFIG, 1);

        // RecordAccumulator：缓冲区大小，默认32M：buffer.memory
        properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);

        // compression.type：压缩，默认none，可配置值gzip、snappy、lz4和zstd
properties.put(ProducerConfig.COMPRESSION_TYPE_CONFIG,"snappy");

        // 3. 创建kafka生产者对象
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<String, String>(properties);

        // 4. 调用send方法,发送消息
        for (int i = 0; i < 5; i++) {

            kafkaProducer.send(new ProducerRecord<>("first","atguigu " + i));

        }

        // 5. 关闭资源
        kafkaProducer.close();
    }
} 
```

## 3.6 生产经验——数据可靠性

**0）回顾发送流程**

![1655860794845](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655860794845.png)

**1）ack应答原理**

![1655860835425](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655860835425.png)

![1655861181698](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655861181698.png)

![1655861674918](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655861674918.png)

##### 代码配置

```java
package com.atguigu.kafka.producer;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class CustomProducerAck {

    public static void main(String[] args) throws InterruptedException {

        // 1. 创建kafka生产者的配置对象
        Properties properties = new Properties();

        // 2. 给kafka配置对象添加配置信息：bootstrap.servers
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop102:9092");
        
        // key,value序列化（必须）：key.serializer，value.serializer
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // 设置acks
        properties.put(ProducerConfig.ACKS_CONFIG, "all");

        // 重试次数retries，默认是int最大值，2147483647
        properties.put(ProducerConfig.RETRIES_CONFIG, 3);

        // 3. 创建kafka生产者对象
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<String, String>(properties);

        // 4. 调用send方法,发送消息
        for (int i = 0; i < 5; i++) {

            kafkaProducer.send(new ProducerRecord<>("first","atguigu " + i));

        }

        // 5. 关闭资源
        kafkaProducer.close();
    }
} 
```

## 3.7 生产经验——数据去重

### 3.7.1 数据传递语义

![1655862980020](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655862980020.png)

### 3.7.2 幂等性

**1）幂等性原理**

![1655863148882](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655863148882.png)

**幂等性，去重同一个分区内的数据**

**生产者生的重复数据，幂等性解决不了**

### 3.7.3 生产者事务

**1）Kafka事务介绍**
		0.11版本的Kafka同时引入了事务的特性，为了实现跨分区跨会话的事务，需要引入一个全局唯一的Transaction ID，并将Producer获得的PID和Transaction ID绑定。这样当Producer重启后就可以通过正在进行的Transaction ID获得原来的PID。
		为了管理Transaction，Kafka引入了一个新的组件Transaction Coordinator。Producer就是通过和Transaction Coordinator交互获得Transaction ID对应的任务状态。Transaction Coordinator还负责将事务所有写入Kafka的一个内部Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。
		**注意：提前开启幂等性!!!**

## 3.8 生产经验——数据有序

HQL去重三种方式：

- group by
- distinct
- 开窗后排序（row number）

![1655864584922](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655864584922.png)

## 3.9 生产经验——数据乱序

![1655864859896](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655864859896.png)

- max.in.flight.request.per.connection=1;:[request中的请求的个数]

# 第4章 Kafka Broker

## 4.1 Kafka Broker工作流程

### 4.1.1 Zookeeper存储的Kafka信息

（1）启动Zookeeper客户端。

![1655866569788](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655866569788.png)

### 4.1.2 Kafka Broker总体工作流程

![1655867124942](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655867124942.png)

## 4.2 Kafka 副本

### 4.2.1 副本基本信息

（1）Kafka副本作用：提高数据可靠性。
（2）Kafka默认副本1个，生产环境一般配置为2个，保证数据可靠性；太多副本会增加磁盘存储空间，增加网络上数据传输，降低效率。
（3）Kafka中副本分为：Leader和Follower。Kafka生产者只会把数据发往Leader，然后Follower找Leader进行同步数据。
（4）**Kafka分区中的所有副本统称为AR**（Assigned Repllicas）。
 AR = ISR + OSR
		**ISR，表示和Leader保持同步的Follower集合**。如果Follower长时间未向Leader发送通信请求或同步数据，则该Follower将被踢出ISR。该时间阈值由replica.lag.time.max.ms参数设定，默认30s。Leader发生故障之后，就会从ISR中选举新的Leader。
	OSR，表示Follower与Leader副本同步时，延迟过多的副本。

### 4.2.2 Leader选举流程

​		Kafka集群中有一个broker的Controller会被选举为Controller Leader，负责管理集群broker的上下线，所有topic的分区副本分配和Leader选举等工作。
​		Controller的信息同步工作是依赖于Zookeeper的。

![1655878094871](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655878094871.png)

### 4.2.3 Leader和Follower故障处理细节

![1655879027872](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655879027872.png)

![1655879557327](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655879557327.png)

## 4.3 文件存储

### 4.3.1 文件存储机制

1）Topic数据的存储机制

![1655881907477](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655881907477.png)

3）index文件和log文件详解

![1655884427772](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655884427772.png)

| 参数                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| log.segment.bytes        | Kafka中log日志是分成一块块存储的，此配置是指log日志划分 成块的大小，默认值1G。 |
| log.index.interval.bytes | 默认4kb，kafka里面每当写入了4kb大小的日志（.log），然后就往index文件里面记录一个索引。 稀疏索引。 |

### 4.3.2 文件清理策略

​	Kafka中默认的日志保存时间为7天，可以通过调整如下参数修改保存时间。

- log.retention.hours，最低优先级小时，默认7天。
-  log.retention.minutes，分钟。
- log.retention.ms，最高优先级毫秒。
- log.retention.check.interval.ms，负责设置检查周期，默认5分钟。
  那么日志一旦超过了设置的时间，怎么处理呢？
  Kafka中提供的日志清理策略有delete和compact两种。
  1）delete日志删除：将过期数据删除
- log.cleanup.policy = delete    所有数据启用删除策略
		（1）基于时间：默认打开。以segment中所有记录中的最大时间戳作为该文件时间戳。
	（2）基于大小：默认关闭。超过设置的所有日志总大小，删除最早的segment。
		log.retention.bytes，默认等于-1，表示无穷大。
  思考：如果一个segment中有一部分数据过期，一部分没有过期，怎么处理？

![1655884864478](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655884864478.png)

2）compact日志压缩

![1655885045841](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655885045841.png)

## 4.4 高效读写数据

1）Kafka本身是分布式集群，可以采用分区技术，并行度高
2）读数据采用稀疏索引，可以快速定位要消费的数据
3）顺序写磁盘
		Kafka的producer生产数据，要写入到log文件中，写的过程是一直追加到文件末端，为顺序写。官网有数据表明，同样的磁盘，顺序写能到600M/s，而随机写只有100K/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。

![1655885286149](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655885286149.png)

4）页缓存 + 零拷贝技术

![1655885390894](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655885390894.png)

| 参数                        | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| log.flush.interval.messages | 强制页缓存刷写到磁盘的条数，默认是long的最大值，9223372036854775807。一般不建议修改，交给系统自己管理。 |
| log.flush.interval.ms       | 每隔多久，刷数据到磁盘，默认是null。一般不建议修改，交给系统自己管理。 |























