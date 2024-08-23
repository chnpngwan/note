#### HDFS上传策略

- 机架选择策略
- 副本节点选择
- hdfs下载流程

### 4.1HDFS写数据流程

![1653993706367](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220531-1653993706367.png)

（1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。

（2）NameNode返回是否可以上传。

（3）客户端请求第一个 Block上传到哪几个DataNode服务器上。

（4）NameNode返回3个DataNode节点，分别为dn1、dn2、dn3。

（5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。

（6）dn1、dn2、dn3逐级应答客户端。

（7）客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。

（8）当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。

#### 4.1.2 副本节点选择

![1653993901842](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220531-1653993901842.png)

### 4.2 HDFS读取文件的流程

![1653993965722](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/2205311653993965722.png)

（1）客户端通过DistributedFileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。

（2）挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。

（3）DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。

（4）客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。

### 第5章 NameNode和SecondaryNameNode

#### 5.1 NN和2NN工作机制

思考：NameNode中的元数据是存储在哪里的？

首先，我们做个假设，如果存储在NameNode节点的磁盘中，因为经常需要进行随机访问，还有响应客户请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的FsImage。

这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新FsImage，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦NameNode节点断电，就会产生数据丢失。因此，引入Edits文件（只进行追加操作，效率很高）。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到Edits中。这样，一旦NameNode节点断电，可以通过FsImage和Edits的合并，合成元数据。

但是，如果长时间添加数据到Edits中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行FsImage和Edits的合并，如果这个操作由NameNode节点完成，又会效率过低。因此，引入一个新的节点SecondaryNamenode，专门用于FsImage和Edits的合并。

![1653994160117](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220531-1653994160117.png)

1）**第一阶段NameNode启动**

（1）第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。

（2）客户端对元数据进行增删改的请求。

（3）NameNode记录操作日志，更新滚动日志。

（4）NameNode在内存中对元数据进行增删改。

2）**第二阶段：Secondary NameNode\工作**

（1）Secondary NameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否检查结果。

（2）Secondary NameNode请求执行CheckPoint。

（3）NameNode滚动正在写的Edits日志。

（4）将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。

（5）Secondary NameNode加载编辑日志和镜像文件到内存，并合并。

（6）生成新的镜像文件fsimage.chkpoint。

（7）拷贝fsimage.chkpoint到NameNode。

（8）NameNode将fsimage.chkpoint重新命名成fsimage。

#### **5.3 C**heckPoint时间设置

***\*1）通常\*******\*情况下，SecondaryNameNode\*******\*每隔\*******\*一小时执行一次\*******\*。\****

```
	[hdfs-default.xml]

<property>

 <name>dfs.namenode.checkpoint.period</name>

 <value>3600s</value>

</property>
```

***\*2）一分钟\*******\*检查一次操作次数，当操作次数达到\*******\*1百万\*******\*时，SecondaryNameNode\*******\*执行\*******\*一次。\****

```
<property>

 <name>dfs.namenode.checkpoint.txns</name>

 <value>1000000</value>

<description>操作动作次数</description>

</property>

 

<property>

 <name>dfs.namenode.checkpoint.check.period</name>

 <value>60s</value>

<description> 1分钟检查一次操作次数</description>

</property>
```

### 第6章 DataNode

#### **6.1** DataNode工作机制

![1653994501763](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220531-1653994501763.png)



（1）一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

（2）DataNode启动后向NameNode注册，通过后，周期性（6小时）的向NameNode上报所有的块信息。

DN向NN汇报当前解读信息的时间间隔，默认6小时；

```
<property>

​	<name>dfs.blockreport.intervalMsec</name>

​	<value>21600000</value>

​	<description>Determines block reporting interval in milliseconds.</description>

</property>
```

DN扫描自己节点块信息列表的时间，默认6小时。

```
<property>

​	<name>dfs.datanode.directoryscan.interval</name>

​	<value>21600s</value>

​	<description>Interval in seconds for Datanode to scan data directories and reconcile the difference between blocks in memory and on the disk.

​	Support multiple time unit suffix(case insensitive), as described

​	in dfs.heartbeat.interval.

​	</description>

</property>
```

（3）心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。

（4）集群运行中可以安全加入和退出一些机器。



![1653994632622](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220531-1653994632622.png)

### 1.8.1本地实操

![1654002616949](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220531-1654002616949.png)

#### 3）环境准备

（1）创建maven工程，MapReduceDemo

（2）在pom.xml文件中添加如下依赖

```
<dependencies>

  <dependency>

​    <groupId>org.apache.hadoop</groupId>

​    <artifactId>hadoop-client</artifactId>

​    <version>3.1.3</version>

  </dependency>

  <dependency>

​    <groupId>junit</groupId>

​    <artifactId>junit</artifactId>

​    <version>4.12</version>

  </dependency>

  <dependency>

​    <groupId>org.slf4j</groupId>

​    <artifactId>slf4j-log4j12</artifactId>

​    <version>1.7.30</version>

  </dependency>

</dependencies>
```

（2）在项目的src/main/resources目录下，新建一个文件，命名为“log4j.properties”，在文件中填入。

```bash
log4j.rootLogger=INFO, stdout  

log4j.appender.stdout=org.apache.log4j.ConsoleAppender  

log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  

log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n  

log4j.appender.logfile=org.apache.log4j.FileAppender  

log4j.appender.logfile.File=target/spring.log  

log4j.appender.logfile.layout=org.apache.log4j.PatternLayout  

log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

（3）创建包名：com.atguigu.mapreduce.wordcount

***\*4\*******\*）编写程序\****

（1）编写Mapper类

```java
package com.atguigu.mapreduce.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * 将数据按行输入，按空格切分，以(word,1)的形式输出
 */
public class WcMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

    private static final IntWritable one = new IntWritable(1);
    private Text word = new Text();

    /**
     * 按空格切分输入的一行数据，并把切分的单词以(word,1)的形式输出给框架
     * @param key 输入数据的行号
     * @param value 这一行内容
     * @param context 整个Job的对象
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context) throws IOException, InterruptedException {
        //获取正在处理的本行数据
        String line = value.toString();

        //将这一行数据按照空格切分
        String[] words = line.split(" ");

        //遍历所有单词，输出(word,1)
        for (String word : words) {

            //将处理完的单词重新打包交给Job
            this.word.set(word);
            context.write(this.word, one);
        }
    }
}

```

（2）编写Reducer类

```java
package com.atguigu.mapreduce.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * 将Mapper输出的(word,1)按照word分组累加
 */
public class WcReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

    private final IntWritable result = new IntWritable();
    /**
     * 将一个单词所有的1进行累加
     * @param key 单词
     * @param values 这个单词对应的所有的1
     * @param context job本身
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
        //针对同一个单词所有的1累加
        int sum = 0;
        for (IntWritable value : values) {
            sum += value.get();
        }
        //sum就是输出的结果

        result.set(sum);
        context.write(key, result);

    }
}

```

（3）编写Driver驱动类

```java
package com.atguigu.mapreduce.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * Driver就是新建Job并对Job进行一系列设置
 */
public class WcDriver {
    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {
        // 新建Job
        Job job = Job.getInstance(new Configuration());

        //如果想让它提交到集群, 需要设置Jar包
//        job.setJar("C:\\Users\\skiin\\IdeaProjects\\mapreduce220411\\target\\mapreduce220411-1.0-SNAPSHOT.jar");
        job.setJarByClass(WcMapper.class);

        //设置mapper和Reducer
        job.setMapperClass(WcMapper.class);
        job.setReducerClass(WcReducer.class);

        //设置输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //设置输入输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        //提交任务到Yarn集群
        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);


    }
}

```

***\*5\*******\*）本地测试\****

（1）需要首先配置好HADOOP_HOME变量以及Windows运行依赖

（2）在IDEA/Eclipse上运行程序

### **1.8.2** **提交到集群测试**

***\*1）集群上测试\****

（1）用maven打jar包，需要添加的打包插件依赖

```bash
<build>

  <plugins>

​    <plugin>

​      <artifactId>maven-compiler-plugin</artifactId>

​      <version>3.6.1</version>

​      <configuration>

​        <source>1.8</source>

​        <target>1.8</target>

​      </configuration>

​    </plugin>

​    <plugin>

​      <artifactId>maven-assembly-plugin</artifactId>

​      <configuration>

​        <descriptorRefs>

​          <descriptorRef>jar-with-dependencies</descriptorRef>

​        </descriptorRefs>

​      </configuration>

​      <executions>

​        <execution>

​          <id>make-assembly</id>

​          <phase>package</phase>

​          <goals>

​            <goal>single</goal>

​          </goals>

​        </execution>

​      </executions>

​    </plugin>

  </plugins>

</build>
```

注意：如果工程上显示红叉。在项目上右键->maven->Reimport刷新即可。

（2）将程序打成jar包

（3）修改不带依赖的jar包名称为wc.jar，并拷贝该jar包到Hadoop集群的/opt/module/hadoop-3.1.3路径。

（4）启动Hadoop集群

```properties
[atguigu@hadoop102 hadoop-3.1.3]sbin/start-dfs.sh

[atguigu@hadoop103 hadoop-3.1.3]$ sbin/start-yarn.sh
```

（5）执行WordCount程序

```properties
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop jar  wc.jar

 com.atguigu.mapreduce.wordcount.WordCountDriver /user/atguigu/input /user/atguigu/output
```

