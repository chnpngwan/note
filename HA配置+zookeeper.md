# 第4章Hadoop企业优化

- **多目录处理**
- **新节点服役退役**
- **故障处理修复**

## 4.1 MapReduce优化方法

​	MapReduce优化方法主要从六个方面考虑：数据输入、Map阶段、Reduce阶段、IO传输、数据倾斜问题和常用的调优参数。

### 4.1.1 数据输入

![1654648484951](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654648484951.png)

- 大量小文件：CombineTextInputFormat
- 大块不可切分文件：

### 4.1.2 Map阶段

![1654648656755](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654648656755.png)

- Merge次数：增加一次合并的数量，默认值是10
- 增大缓存，增大溢写比例

### 4.1.3 Reduce阶段

![1654648919368](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654648919368.png)

- 共存，默认5%，

![1654649502577](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654649502577.png)

### 4.1.4 I/O传输

![1654649563939](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654649563939.png)

### 4.1.5 数据倾斜问题

![1654649596130](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654649596130.png)

![1654649646481](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654649646481.png)

## 4.2 常用的调优参数

**1）**资源相关参数
	（1）以下参数是在用户自己的**MR**应用程序中配置就可以生效（**mapred-default.xml**）

![1654650087063](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654650087063.png)

![1654650105537](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654650105537.png)

**（2）**应该在YARN启动之前就配置在服务器的配置文件中才能生效（yarn-default.xml）

![1654650275043](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654650275043.png)

![1654650314459](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654650314459.png)

- 重点是yarn的参数，其他参数了解就行

## 4.4 Hadoop小文件优化方法

### 4.4.1 Hadoop小文件弊端

​		HDFS上每个文件都要在NameNode上创建对应的元数据，这个元数据的大小约为150byte，这样当小文件比较多的时候，就会产生很多的元数据文件，一方面会大量占用NameNode的内存空间，另一方面就是元数据文件过多，使得寻址索引速度变慢。
​		小文件过多，在进行MR计算时，会生成过多切片，需要启动过多的MapTask。每个MapTask处理的数据量小，导致MapTask的处理时间比启动时间还小，白白消耗资源。

### 4.4.2Hadoop小文件解决方案

**1）小文件优化的方向：**
	（1）在数据采集的时候，就将小文件或小批数据合成大文件再上传HDFS。**最合理方案**
	（2）在业务处理之前，在HDFS上使用MapReduce程序对小文件进行合并。
	（3）在MapReduce处理时，可采用CombineTextInputFormat提高效率。
	（4）开启**uber模式**，实现jvm重用（申请固定的资源容器，排队跑任务，所有任务跑完后再释放资源）

​			节省了容器资源申请的时间

**2）Hadoop Archive**
		是一个高效的将小文件放入HDFS块中的文件存档工具，能够将多个小文件打包成一个HAR文件，从而达到减少NameNode的内存使用
**3）CombineTextInputFormat**
		CombineTextInputFormat用于将多个小文件在切片过程中生成一个单独的切片或者少量的切片。 
**4）开启uber模式，实现JVM重用。**
		默认情况下，每个Task任务都需要启动一个JVM来运行，如果Task任务计算的数据量很小，我们可以让同一个Job的多个Task运行在一个JVM中，不必为每个Task都开启一个JVM. 
		开启uber模式，在mapred-site.xml中添加如下配置

```xml
<!--  开启uber模式 -->
<property>
	<name>mapreduce.job.ubertask.enable</name>
	<value>true</value>
</property>

<!-- uber模式中最大的mapTask数量，可向下修改  --> 
<property>
	<name>mapreduce.job.ubertask.maxmaps</name>
	<value>9</value>
</property>
<!-- uber模式中最大的reduce数量，可向下修改 -->
<property>
	<name>mapreduce.job.ubertask.maxreduces</name>
	<value>1</value>
</property>
<!-- uber模式中最大的输入数据量，默认使用dfs.blocksize 的值，可向下修改 -->
<property>
	<name>mapreduce.job.ubertask.maxbytes</name>
	<value></value>
</property>
```

# 第5章 Hadoop扩展

## 5.1 集群间数据拷贝

1）scp实现两个远程主机之间的文件复制
	``scp -r hello.txt root@hadoop103:/user/atguigu/hello.txt		// 推 push`
	`scp -r root@hadoop103:/user/atguigu/hello.txt  hello.txt		// 拉 pull`
	`scp -r root@hadoop103:/user/atguigu/hello.txt root@hadoop104:/user/atguigu`   //是通过本地主		机中转实`现两个远程主机的文件复制；如果在两个远程主机之间ssh没有配置的情况下可以使用该方式。
2）采用distcp命令实现两个Hadoop集群之间的递归数据复制

```properties
[atguigu@hadoop102 hadoop-3.1.3]$  bin/hadoop distcp hdfs://hadoop102:8020/user/atguigu/hello.txt hdfs://hadoop105:8020/user/atguigu/hello.txt
```

## 5.2 小文件存档

![1654651046047](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654651046047.png)

**1）案例实操**
（1）需要启动YARN进程

```properties
[atguigu@hadoop102 hadoop-3.1.3]$ start-yarn.sh
```

（2）归档文件
	把/user/atguigu/input目录里面的所有文件归档成一个叫input.har的归档文件，并把归档后文件存储到/user/atguigu/output路径下。

```properties
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop archive -archiveName input.har -p  /user/atguigu/input   /user/atguigu/output
```

（3）查看归档

```properties
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -ls /user/atguigu/output/input.har
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -ls har:///user/atguigu/output/input.har
```

（4）解归档文件

```properties
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -cp har:/// user/atguigu/output/input.har/*    /user/atguigu
```

## 5.3 回收站

​		开启回收站功能，可以将删除的文件在不超时的情况下，恢复原数据，起到防止误删除、备份等作用。

- **了解就行**

​	**1）回收站参数设置及工作机制**

![1654651448994](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654651448994.png)

**2）启用回收站**
		修改core-site.xml，配置垃圾回收时间为1分钟。

```xml
<property>
    <name>fs.trash.interval</name>
	<value>1</value>
</property>
```

**3）查看回收站**
		回收站目录在hdfs集群中的路径：/user/atguigu/.Trash/….
**4）通过程序删除的文件不会经过回收站，需要调用moveToTrash()才进入回收站**

```properties
//因为本地的客户端拿不到集群的配置信息 所以需要自己手动设置一下回收站
conf.set(“fs.trash.interval”,”1”);
conf.set(“fs.trash.checkpoint.interval”,”1”);
//创建一个回收站对象
Trash trash = New Trash(conf);
trash.moveToTrash(path);
```

**5）通过网页上直接删除的文件也不会走回收站。**
**6）只有在命令行利用hadoop fs -rm命令删除的文件才会走回收站。**

```properties
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -rm -r /user/atguigu/input
2020-07-14 16:13:42,643 INFO fs.TrashPolicyDefault: Moved: 'hdfs://hadoop102:8020/user/atguigu/input' to trash at: hdfs://hadoop102:8020/user/atguigu/.Trash/Current/user/atguigu/input
```

**7）恢复回收站数据**

```properties
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mv
/user/atguigu/.Trash/Current/user/atguigu/input    /user/atguigu/input
```

# 第1章 Zookeeper入门

- **中央协调功能的**
- **特点：半数以上，全局一致**
- **ZAB协议：选举机制， 写数据流程**

## 1.1 概述

![1654653269058](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654653269058.png)

## 1.2 特点

![1654653319256](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654653319256.png)

- **半数以上的节点存活，就能正常工作， 全局数据一致**

## 1.3 数据结构

​			ZooKeeper数据模型的结构与**Unix文件系统很类似**，整体上可以看作是一棵树，每个节点称做一个ZNode。每一个ZNode默认能够存储**1MB**的数据，每个ZNode都可以通过其路径唯一标识。

![1654653761884](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654653761884.png)

# Zookeeper安装

1. 将tar包拷贝到集群，解压并改名

   ```bash
   tar -zxf /opt/software/apache-zookeeper-3.5.7-bin.tar.gz -C /opt/module/
   mv /opt/module/apache-zookeeper-3.5.7-bin /opt/module/zookeeper
   xsync /opt/module/zookeeper
   ```

2. 添加环境变量

   ```bash
   vim /etc/profile.d/my_env.sh
   ```

   追加如下内容

   ```bash
   # ZOOKEEPER_HOME
   export ZOOKEEPER_HOME=/opt/module/zookeeper
   export PATH=$PATH:$ZOOKEEPER_HOME/bin
   ```

   完成后同步环境变量

   ```bash
   sudo xsync /etc/profile.d/my_env.sh
   ```

3. 配置Zookeeper

   - 将配置文件模板重命名

     ```bash
     cp /opt/module/zookeeper/conf/zoo_sample.cfg /opt/module/zookeeper/conf/zoo.cfg
     ```

   - 进行配置

     ```bash
     sed -i '/^dataDir/s/.*/dataDir=\/opt\/module\/zookeeper\/zkData/' /opt/module/zookeeper/conf/zoo.cfg
     echo "server.2=hadoop102:2888:3888" >> /opt/module/zookeeper/conf/zoo.cfg
     echo "server.3=hadoop103:2888:3888" >> /opt/module/zookeeper/conf/zoo.cfg
     echo "server.4=hadoop104:2888:3888" >> /opt/module/zookeeper/conf/zoo.cfg
     ```

   - 同步配置文件

     ```bash
     xsync /opt/module/zookeeper/conf/zoo.cfg
     ```

   - 在所有节点新建/opt/module/zookeeper/zkData目录，并在其中新建myid文件，内容是自己的ID

     ```bash
     xcall 'mkdir -p /opt/module/zookeeper/zkData'
     xcall 'echo $(hostname) | cut -c 9 > /opt/module/zookeeper/zkData/myid'
     ```

4. 启动zookeeper

   - 关闭所有shell窗口并重启，从而让环境变量生效

   - zookeeper默认没有群起命令，只能一个一个节点单独启动

     ```bash
     # 在三台节点分别执行
     zkServer.sh start
     # 查询状态，执行
     zkServer.sh status
     
     ```

   - 为了将来方便zookeeper使用，写一个群起脚本

     ```bash
     sudo  vim /opt/module/zookeeper/bin/zk.sh
     
     ```

     内容如下

     ```bash
     #!/bin/bash
     xcall "zkServer.sh $@ 2>/dev/null" | grep -v Client 
     
     ```

     添加执行权限并测试

     ```bash
     chmod +x /opt/module/zookeeper/bin/zk.sh
     
     ```

     ```bash
     zk.sh start/status/stop
     
     ```

     



### 3.1.2 选举机制（面试重点）

![1654655784077](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654655784077.png)

- 数据新旧（比较ZXID， 事物id）：数据越新越牛---->数据一致比较myid，myid越大越牛，

- 在一定时间内启动的zookeper（几百毫秒之内都算作同时）-----会发生选举。



## 3.2 客户端命令行操作（了解）

### 3.2.1 命令行语法

![1654670947944](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654670947944.png)

![1654670981017](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654670981017.png)

![1654671057236](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654671057236.png)

### 3.2.2 znode节点数据信息

**1）查看当前znode中所包含的内容**

![1654671100632](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654671100632.png)

![1654671118990](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654671118990.png)

![1654671140601](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654671140601.png)

## 3.3 写数据流程

![1654671691964](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654671691964.png)

### 3.2.4 监听器原理

​			客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、节点删除、子目录节点增加删除）时，ZooKeeper会通知客户端。监听机制保证ZooKeeper保存的任何的数据的任何改变都能快速的响应到监听了该节点的应用程序。

---

![1654672709211](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654672709211.png)

- 同步通信：打电话--有来有回，（会阻塞进程）
- 异步通信：发消息---发完就去做别的，（不阻塞进程）
- 回调函数

# 第1章 Hadoop HA高可用

## 1.1 HA概述

​		（1）所谓HA（High Availablity），即高可用（7 * 24小时不中断服务）。
​		（2）实现高可用最关键的策略是消除单点故障。HA严格来说应该分成各个组件的HA机制：HDFS的HA和YARN的HA。
​		（3）NameNode主要在以下两个方面影响HDFS集群
​				=>NameNode机器发生意外，如宕机，集群将无法使用，直到管理员重启。
​				=>NameNode机器需要升级，包括软件、硬件升级，此时集群也将无法使用。
​				HDFS HA功能通过**配置多个NameNodes(Active/Standby)**实现在集群中对NameNode的热备来解决上述问题。如果出现故障，如机器崩溃或机器需要升级维护，这时可通过此种方式将NameNode很快的切换到另外一台机器。

## 1.2 HDFS-HA集群搭建

# HDFS HA搭建

![1654686671718](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220608-1654686671718.png)

1. 先确保自己的Zookeeper正常运行，关机照快照

2. 清理集群之前的旧数据

   ```bash
   xcall 'rm -rf /opt/module/hadoop-3.1.3/data /opt/module/hadoop-3.1.3/logs'
   ```

3. 修改配置文件

   - core-site.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
     
     <configuration>
       <!-- 指定NameNode的地址 -->
       <property>
         <name>fs.defaultFS</name>
         <value>hdfs://mycluster</value>
       </property>
     
       <!-- 指定hadoop数据的存储目录 -->
       <property>
         <name>hadoop.tmp.dir</name>
         <value>/opt/module/hadoop-3.1.3/data</value>
       </property>
     
       <!-- 配置HDFS网页登录使用的静态用户为atguigu -->
       <property>
         <name>hadoop.http.staticuser.user</name>
         <value>atguigu</value>
       </property>
       <!-- 指定zkfc要连接的zkServer地址 -->
       <property>
         <name>ha.zookeeper.quorum</name>
         <value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>
       </property>
     </configuration>
     ```

   - hdfs-site.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
     
     <configuration>
     
       <!-- NameNode数据存储目录 -->
       <property>
         <name>dfs.namenode.name.dir</name>
         <value>file://${hadoop.tmp.dir}/name</value>
       </property>
     
       <!-- DataNode数据存储目录 -->
       <property>
         <name>dfs.datanode.data.dir</name>
         <value>file://${hadoop.tmp.dir}/data</value>
       </property>
     
       <!-- JournalNode数据存储目录 -->
       <property>
         <name>dfs.journalnode.edits.dir</name>
         <value>${hadoop.tmp.dir}/jn</value>
       </property>
     
       <!-- 完全分布式集群名称 -->
       <property>
         <name>dfs.nameservices</name>
         <value>mycluster</value>
       </property>
     
       <!-- 集群中NameNode节点都有哪些 -->
       <property>
         <name>dfs.ha.namenodes.mycluster</name>
         <value>nn1,nn2</value>
       </property>
     
       <!-- NameNode的RPC通信地址 -->
       <property>
         <name>dfs.namenode.rpc-address.mycluster.nn1</name>
         <value>hadoop102:8020</value>
       </property>
       <property>
         <name>dfs.namenode.rpc-address.mycluster.nn2</name>
         <value>hadoop103:8020</value>
       </property>
     
       <!-- NameNode的http通信地址 -->
       <property>
         <name>dfs.namenode.http-address.mycluster.nn1</name>
         <value>hadoop102:9870</value>
       </property>
       <property>
         <name>dfs.namenode.http-address.mycluster.nn2</name>
         <value>hadoop103:9870</value>
       </property>
     
       <!-- 指定NameNode元数据在JournalNode上的存放位置 -->
       <property>
         <name>dfs.namenode.shared.edits.dir</name>
         <value>qjournal://hadoop102:8485;hadoop103:8485;hadoop104:8485/mycluster</value>
       </property>
     
       <!-- 访问代理类：client用于确定哪个NameNode为Active -->
       <property>
         <name>dfs.client.failover.proxy.provider.mycluster</name>
         <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
       </property>
     
       <!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
       <property>
         <name>dfs.ha.fencing.methods</name>
         <value>sshfence</value>
       </property>
     
       <!-- 使用隔离机制时需要ssh秘钥登录-->
       <property>
         <name>dfs.ha.fencing.ssh.private-key-files</name>
         <value>/home/atguigu/.ssh/id_rsa</value>
       </property>
       <!-- 启用nn故障自动转移 -->
       <property>
         <name>dfs.ha.automatic-failover.enabled</name>
         <value>true</value>
       </property>
     </configuration>
     ```

   - 保存配置文件，并同步

     ```
     xsync /opt/module/hadoop-3.1.3/etc/hadoop
     ```

4. 第一次启动集群之前需要初始化

   - 初始化NN的元数据

     - 首先启动QJM集群

       ```bash
       # 在三台节点上分别执行
       hdfs --daemon start journalnode
       ```

       或者

       ```bash
       # 在某一台节点执行
       hdfs --workers --daemon start journalnode
       ```

     - 格式化其中一台Namenode

       ```bash
       # 格式化102的Namenode
       hdfs namenode -format
       # 由于JournalNode启动需要一定时间，如果电脑比较慢，执行晚上上一步，多等等再执行这一步！
       # 由于JournalNode启动需要一定时间，如果电脑比较慢，执行晚上上一步，多等等再执行这一步！
       # 由于JournalNode启动需要一定时间，如果电脑比较慢，执行晚上上一步，多等等再执行这一步！
       # 由于JournalNode启动需要一定时间，如果电脑比较慢，执行晚上上一步，多等等再执行这一步！
       ```

     - 另外的NN同步已经格式化的NN的数据

       ```bash
       # 首先启动已经格式化的NN（启动102的NN）
       hdfs --daemon start namenode
       ```

       ```bash
       # 在需要同步NN数据的节点，执行同步指令
       hdfs namenode -bootstrapStandby
       # 完成以后就可以启动NN了
       hdfs --daemon start namenode
       ```

   - 初始化Zookeeper

     - 启动Zookeeper集群

       ```bash
       zk.sh start
       ```

     - 格式化Zookeeper的数据

       ```bash
       # 在任意配置了NN的节点执行
       hdfs zkfc -formatZK
       ```

     - 在配置了NN的节点启动ZKFC

       ```bash
       # 在102，103执行
       hdfs --daemon start zkfc
       ```

   - 启动Datanodes

     ```bash
     hdfs --workers --daemon start datanode
     ```

5. 第二次启动或者关闭HA，可以直接使用群起脚本

   ```bash
   start-dfs.sh/stop-dfs.sh
   ```

   **由于群起指令（start-dfs.sh）是先启动NN，再启动QJM。如果电脑太卡，可能会导致NN启动失败，此时手动重启NN即可。**

   **由于群起指令（start-dfs.sh）是先启动NN，再启动QJM。如果电脑太卡，可能会导致NN启动失败，此时手动重启NN即可。**

   **由于群起指令（start-dfs.sh）是先启动NN，再启动QJM。如果电脑太卡，可能会导致NN启动失败，此时手动重启NN即可。**















