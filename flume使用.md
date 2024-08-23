# 第1章 Flume概述

## 1.1 Flume定义 

​		Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统。Flume基于流式架构，灵活简单。

![1655728830493](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655728830493.png)

## 1.2 Flume基础架构

​			Flume组成架构如下图所示。

![1655686625946](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655686625946.png)

### 1.2.1 Agent

Agent是一个JVM进程，它以事件的形式将数据从源头送至目的地。
Agent主要有3个部分组成，Source、Channel、Sink。

### 1.2.2 Source

​		Source是负责接收数据到Flume Agent的组件。Source组件可以处理各种类型、各种格式的日志数据 

### 1.2.3 Sink

​		Sink不断地轮询Channel中的事件且批量地移除它们，并将这些事件批量写入到存储或索引系统、或者被发送到另一个Flume Agent。

### 1.2.4 Channel

​		Channel是位于Source和Sink之间的缓冲区。因此，Channel允许Source和Sink运作在不同的速率上。Channel是线程安全的，可以同时处理几个Source的写入操作和几个Sink的读取操作。
Flume自带两种Channel：Memory Channel和File Channel。
​		Memory Channel是内存中的队列。Memory Channel在不需要关心数据丢失的情景下适用。如果需要关心数据丢失，那么Memory Channel就不应该使用，因为程序死亡、机器宕机或者重启都会导致数据丢失。
File Channel将所有事件写到磁盘。因此在程序关闭或机器宕机的情况下不会丢失数据。

### 1.2.5 Event

​		传输单元，Flume数据传输的基本单元，以Event的形式将数据从源头送至目的地。Event由Header和Body两部分组成，Header用来存放该event的一些属性，为K-V结构，Body用来存放该条数据，形式为字节数组。

![1655728929685](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655728929685.png)



## 2.2 Flume入门案例

### 2.2.1 监控端口数据官方案例

1）案例需求：
使用Flume监听一个端口，收集该端口数据，并打印到控制台。 
2）实现步骤：
（1）安装netcat工具

```
[atguigu@hadoop102 software]$ sudo yum install -y nc
```

（2）判断44444端口是否被占用

```
[atguigu@hadoop102 flume]$ sudo netstat -nlp | grep 44444
```

（3）在conf文件夹下创建Flume Agent配置文件nc-flume-log.conf。

```
[atguigu@hadoop102 conf]$ vim nc-flume-log.conf
```

（4）在nc-flume-log.conf文件中添加如下内容。

```shell
添加内容如下：
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Use a channel which buffers events in memory
# capacity： 容积，能力， 
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Describe the sink
a1.sinks.k1.type = logger

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

（5）先开启flume监听端口
第一种写法：

```properties
[atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name a1 --conf-file conf/nc-flume-log.conf -Dflume.root.logger=INFO,console
```

第二种写法：

```properties
[atguigu@hadoop102 flume]$ bin/flume-ng agent -c conf/ -n a1 -f conf/nc-flume-log.conf -Dflume.root.logger=INFO,console
```

参数说明：
	--conf/-c：表示配置文件存储在conf/目录
	--name/-n：表示给agent起名为a1
	--conf-file/-f：flume本次启动读取的配置文件是在conf文件夹下的nc-flume-log.conf文件。
	-Dflume.root.logger=INFO,console ：-D表示flume运行时动态修改flume.root.logger参数属性值，并将控制台日志打印级别设置为INFO级别。日志级别包括:log、info、warn、error。日志参数已经在配置文件中修改了，不再需要重复输入。



### 2.2.2 实时监控目录下的多个追加文件

Taildir Source适合用于监听多个实时追加的文件，并且能够实现断点续传。
1）案例需求:使用Flume监听整个目录的实时追加文件，并上传至HDFS 
2）需求分析

![1655729179272](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655729179272.png)

- 配置文件

```shell
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = TAILDIR
a1.sources.r1.filegroups = f1 f2
# 必须精确到文件，可以写匹配表达式匹配多个文件
a1.sources.r1.filegroups.f1 = /opt/module/flume/files1/.*file.*
a1.sources.r1.filegroups.f2 = /opt/module/flume/files2/.*log.*
# 实现断点续传的文件存放位置 不改有默认位置也能实现断点续传
a1.sources.r1.positionFile = /opt/module/flume/taildir_position.json


# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Describe the sink
a1.sinks.k1.type = hdfs
# 地址值可以填写hdfs://hadoop102:9820也可以省略,flume会自动读取hadoop配置文件信息获取地址
a1.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume/%Y%m%d/%H
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = log-

#是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true

#设置文件类型 分为二进制文件SequenceFile和文本文件DataStream(不能压缩) 和CompressedStream(可以压缩)
a1.sinks.k1.hdfs.fileType = DataStream

#多久生成一个新的文件
a1.sinks.k1.hdfs.rollInterval = 30
#设置每个文件的滚动大小大概是128M
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0 


# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

# 第3章 Flume进阶

## 3.1 Flume事务

![1655729387520](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655729387520.png)



```properties
# Describe the sink
a1.sinks.k1.type = hdfs
# 地址值可以填写hdfs://hadoop102:9820也可以省略,flume会自动读取hadoop配置文件信息获取地址
a1.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume/%Y%m%d/%H
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = log-
```



- set nu 添加行号
- / 查找

![1655723132409](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655723132409.png)

本机向端口发送内容

```shell
nc localhost 44444
```

## 3.2 Flume Agent内部原理

![1655729549698](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655729549698.png)

**组件介绍：**
1）ChannelSelector
		ChannelSelector的作用就是选出Event将要被发往哪个Channel。其共有两种类型，分别是Replicating（复制）和Multiplexing（多路复用）。
		ReplicatingSelector会将同一个Event发往所有的Channel，Multiplexing会根据相应的原则，将不同的Event发往不同的Channel。
2）SinkProcessor
		SinkProcessor共有三种类型，分别是DefaultSinkProcessor(默认1对1)、LoadBalancingSinkProcessor(负载均衡)和FailoverSinkProcessor(故障转移)
		DefaultSinkProcessor对应的是单个的Sink，LoadBalancingSinkProcessor和FailoverSinkProcessor对应的是Sink Group，LoadBalancingSinkProcessor可以实现负载均衡的功能，FailoverSinkProcessor可以错误恢复的功能。



## 3.3 Flume企业开发案例

### 3.3.1 复制案例

1）案例需求
		使用Flume-1监控文件变动，Flume-1将变动内容传递给Flume-2，Flume-2负责存储到HDFS。同时Flume-1将变动内容传递给Flume-3，Flume-3负责输出到Local FileSystem。

![1655729661537](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655729661537.png)

- flume1.config

```shell
# Name the components on this agent
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1 c2

# Describe/configure the source
a1.sources.r1.type = TAILDIR
a1.sources.r1.filegroups = f1 f2
a1.sources.r1.filegroups.f1 = /opt/module/flume/files1/.*file.*
a1.sources.r1.filegroups.f2 = /opt/module/flume/files2/.*log.*
a1.sources.r1.positionFile = /opt/module/flume/taildir_position.json


# 将数据流复制给所有channel 默认参数可以不写
a1.sources.r1.selector.type = replicating


# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.channels.c2.type = memory
a1.channels.c2.capacity = 1000
a1.channels.c2.transactionCapacity = 100


# Describe the sink
# sink端的avro是一个数据发送者
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop102 
a1.sinks.k1.port = 4141

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop102
a1.sinks.k2.port = 4142


# Bind the source and sink to the channel
a1.sources.r1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2
```

- flume2.config

```shell
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source

a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop102
a1.sources.r1.port = 4141


# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume1/%Y%m%d/%H
# 文件的前缀
a1.sinks.k1.hdfs.filePrefix = log-

#多久生成一个新的文件
a1.sinks.k1.hdfs.rollInterval = 30
#设置每个文件的滚动大小大概是128M
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0
# 使用本地的时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true

#设置文件类型 分为二进制文件SequenceFile和文本文件DataStream(不能压缩) 和CompressedStream(可以压缩)
a1.sinks.k1.hdfs.fileType = DataStream


# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

- flume3.config

```shell
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source

a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop102
a1.sources.r1.port = 4142


# Describe the sink
a1.sinks.k1.type = file_roll
a1.sinks.k1.sink.directory = /opt/module/flume/flume3datas


# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

### 3.3.2 多路复用及拦截器的使用

1）案例需求
		使用Flume采集服务器本地日志，需要按照日志类型的不同，将不同种类的日志发往不同的分析系统。

![1655730000267](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655730000267.png)

- Avro Sink  与  Avaro Source 是flume之间进行通讯传输的专用source与sinke，
- 内部就是网络传输端口， flume对其进行了包装

##### 自定义拦截器

```java
package com.atguigu.flume;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.util.List;
import java.util.Map;

public class MyInterceptor implements Interceptor {
    @Override
    public void initialize() {

    }

    @Override
    public Event intercept(Event event) {

        byte[] body = event.getBody();
        byte b = body[0];

        Map<String, String> headers = event.getHeaders();

        if (b>'0' && b<'9'){
            headers.put("type", "number");
        }else if ((b>'a' && b<'z') || (b>'A' && b<'Z')){
            headers.put("type", "letter");
        }
        event.setHeaders(headers);

        return event;
    }

    @Override
    public List<Event> intercept(List<Event> list) {

        for (Event event : list) {
            intercept(event);
        }

        return list;
    }

    public static class MyBuilder implements Builder{

        @Override
        public Interceptor build() {
            return new MyInterceptor();
        }

        @Override
        public void configure(Context context) {

        }
    }

    @Override
    public void close() {

    }
}

```

- 将代码打tar包，放在lib路径下边

- flume1

```shell
# Name the components on this agent
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1 c2

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444


a1.sources.r1.selector.type = multiplexing
# 使用headers中的哪些参数
a1.sources.r1.selector.header = type
a1.sources.r1.selector.mapping.number = c1
a1.sources.r1.selector.mapping.letter = c2

# a1.sources.r1.selector.default = c4

# 拦截器配置
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.MyInterceptor$MyBuilder

# Describe the sink

a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop102
a1.sinks.k1.port = 4141

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop102
a1.sinks.k2.port = 4142

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.channels.c2.type = memory
a1.channels.c2.capacity = 1000
a1.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2
```

- flume2.config

```shell
a1.sources = r1
a1.sinks = k1
a1.channels = c1

a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop102
a1.sources.r1.port = 4141

a1.sinks.k1.type = logger

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sinks.k1.channel = c1
a1.sources.r1.channels = c1
```

- flume3.config

```shell
a1.sources = r1
a1.sinks = k1
a1.channels = c1

a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop102
a1.sources.r1.port = 4142

a1.sinks.k1.type = logger


a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sinks.k1.channel = c1
a1.sources.r1.channels = c1
```



### 3.3.3 聚合案例

**1）案例需求：**
		hadoop102上的Flume-1监控文件/opt/module/flume/files1/.*file.*，
		hadoop103上的Flume-2监控某一个端口的数据流，
		Flume-1与Flume-2将数据发送给hadoop104上的Flume-3，Flume-3将最终数据打印到控制台。
**2）需求分析**

![1655724329541](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1655724329541.png)



##### flume1.conf

```shell
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = TAILDIR
a1.sources.r1.filegroups = f1 f2
a1.sources.r1.filegroups.f1 = /opt/module/flume/files1/.*file.*
a1.sources.r1.filegroups.f2 = /opt/module/flume/files2/.*log.*
a1.sources.r1.positionFile = /opt/module/flume/taildir_position.json


# Describe the sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop104
a1.sinks.k1.port = 4141


# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

##### flume2.conf

```shell
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop104
a1.sinks.k1.port = 4141

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

##### flume3.conf

```shell
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source

a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop104
a1.sources.r1.port = 4141


# Describe the sink
a1.sinks.k1.type = logger


# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```





