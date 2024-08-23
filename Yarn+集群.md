# Yarn资源管理

## yarn基础架构

![1654562498714](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654562498714.png)

![1654562528287](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654562528287.png)

- 资源都是以容器形式发放的，RM负责管理
- 一个job（作业），会生成一个APPMaster
- RM单独在一个节点上(普通架构)

## **5.2** Yarn工作机制

![1654562894639](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654562894639.png)

- （1）MR程序提交到客户端所在的节点。

- （2）YarnRunner向ResourceManager申请一个Application。

- （3）RM将该应用程序的资源路径返回给YarnRunner。

-  （4）该程序将运行所需资源提交到HDFS上。

- （5）程序资源提交完毕后，申请运行mrAppMaster。

- （6）RM将用户的请求初始化成一个Task。

-  （7）其中一个NodeManager领取到Task任务。

-  （8）该NodeManager创建容器Container，并产生MRAppmaster。

-  （9）Container从HDFS上拷贝资源到本地。

- （10）MRAppmaster向RM 申请运行MapTask资源。

-  （11）RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

-  （12）MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

- （13）MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

-  （14）ReduceTask向MapTask获取相应分区的数据。

- （15）程序运行完毕后，MR会向RM申请注销自己。

## 5.3 作业提交全过程

### 三者关系

![1654563951960](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654563951960.png)

## 5.4 Yarn调度器和调度算法

### 5.4.1 先进先出调度器（FIFO）

**FIFO调度器（First In First Out）：单队列，根据提交作业的先后顺序，先来先服务。**

![1654564193313](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654564193313.png)

### 5.4.2 容量调度器（Capacity Scheduler）

**Capacity Scheduler是Yahoo开发的多用户调度器。**

![1654564490089](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654564490089.png)

### 5.4.3 公平调度器（Fair Scheduler）

**Fair Schedulere是Facebook开发的多用户调度器。**

![1654564864546](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654564864546.png)

- 同时运行全部任务,{`Fair策略`}

![1654565209960](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654565209960.png)

![1654566035759](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654566035759.png)

## 了解DRF策略

![1654568214036](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654568214036.png)

## 第6章 Yarn案例实操

### 6.1 Yarn生产环境核心参数配置案例

# Yarn常用配置

## ResourceManager性能相关

| 名称                                               | 默认值                                                       | 描述                       |
| -------------------------------------------------- | ------------------------------------------------------------ | -------------------------- |
| yarn.resourcemanager.scheduler.class               | org. apache. hadoop. yarn. server. resourcemanager. scheduler. capacity. CapacityScheduler | Yarn默认调度器的名称       |
| yarn.resourcemanager.scheduler.client.thread-count | 50                                                           | Yarn用来处理客户端的线程数 |

## Yarn集群资源配置（NodeManager）

1. 自动配置

   | 名称                                                        | 默认值 | 描述                                                         |
   | ----------------------------------------------------------- | ------ | ------------------------------------------------------------ |
   | yarn.nodemanager.resource.detect-hardware-capabilities      | false  | 是否让NodeManager自动检测硬件资源，如果开启，内存数量为物理内存*0.7，CPU核数为物理核数 |
   | yarn.nodemanager.resource.count-logical-processors-as-cores | false  | 是否采用虚拟CPU和物理CPU比例                                 |
   | yarn.nodemanager.resource.pcores-vcores-multiplier          | 1.0    | 虚拟核数和物理核数的比例                                     |

2. 手动配置

   | 名称                                 | 默认值 | 描述                                          |
   | ------------------------------------ | ------ | --------------------------------------------- |
   | yarn.nodemanager.resource.memory-mb  | 8192   | 手动配置一台NodeManager可以管理的内存总量(mb) |
   | yarn.nodemanager.resource.cpu-vcores | 8      | 手动配置一台NodeManager可以管理的虚拟CPU核数  |

## Yarn容器资源配置

| 名称                                     | 默认值 | 描述                                                      |
| ---------------------------------------- | ------ | --------------------------------------------------------- |
| yarn.scheduler.minimum-allocation-mb     | 1024   | Yarn允许容器申请的最小内存，小于这个内存的申请会被否决    |
| yarn.scheduler.maximum-allocation-mb     | 8192   | Yarn允许容器申请的最大内存，大于这个内存的申请会被否决    |
| yarn.scheduler.minimum-allocation-vcores | 1      | Yarn允许容器申请的最小CPU核数，小于这个核数的申请会被否决 |
| yarn.scheduler.maximum-allocation-vcores | 8      | Yarn允许容器申请的最大CPU核数，大于这个核数的申请会被否决 |

## Yarn容器内存限制

| 名称                                | 默认值 | 描述                                                         |
| ----------------------------------- | ------ | ------------------------------------------------------------ |
| yarn.nodemanager.pmem-check-enabled | true   | 是否限制容器实际使用的物理内存总数，如果设置为true，当容器实际使用的物理内存超过申请限制时，容器会被杀掉 |
| yarn.nodemanager.vmem-check-enabled | true   | 是否限制容器实际使用的虚拟内存总数，如果设置为true，当容器实际使用的虚拟内存超过申请限制的一定比例时，容器会被杀掉 |
| yarn.nodemanager.vmem-pmem-ratio    | 2.1    | 虚拟内存和实际申请内存的比例                                 |

# 容量调度器相关配置

| 名称                                                | 默认值 | 描述                                          |
| --------------------------------------------------- | ------ | --------------------------------------------- |
| yarn.scheduler.capacity.maximum-am-resource-percent | 0.1    | 集群最多有多少比例的资源可以用来运行AppMaster |
| yarn.scheduler.capacity.maximum-applications        | 10000  | 集群正在运行和排队中的APP最多多少个           |

多队列配置

| 名称                                                         | 默认值  | 描述                                  |
| ------------------------------------------------------------ | ------- | ------------------------------------- |
| yarn.scheduler.capacity.root.queues                          | default | 队列总数，默认只有default一条队列     |
| yarn.scheduler.capacity.root.**xxx**.capacity                |         | xxx队列占总资源比例                   |
| yarn.scheduler.capacity.root.**xxx**.user-limit-factor       | 1       | xxx队列单用户可以最多使用多少比例资源 |
| yarn.scheduler.capacity.root.**xxx**.maximum-capacity        |         | xxx队列最大资源比例                   |
| yarn.scheduler.capacity.root.**xxx**.state                   | RUNNING | xxx队列状态，取值范围[RUNNING, STOP]  |
| yarn.scheduler.capacity.root.**xxx**.acl_submit_applications | *       | 规定谁可以向xxx队列提交任务           |
| yarn.scheduler.capacity.root.**xxx**.acl_administer_queue    | *       | 规定谁可以管理xxx队列                 |
| yarn.scheduler.capacity.root.**xxx**.acl_application_max_priority | *       | 规定谁可以向xxx队列提交带优先级的任务 |
| yarn.scheduler.capacity.root.**xxx**.maximum-application-lifetime | -1      | 规定在xxx队列上的APP最长运行多久      |
| yarn.scheduler.capacity.root.**xxx**.default-application-lifetime | -1      | 规定在xxx队列上的APP默认运行多久      |

多队列提交

```bash
hadoop jar \
/opt/module/hadoop-3.1.3\
/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar \
wordcount \
-Dmapreduce.job.queuename=hive \
/input \
/xxx4
```



# 第1章 HDFS—多目录

## 1.1 DataNode多目录配置

- 1）DataNode可以配置成多个目录，每个目录存储的数据不一样（数据不是副本）

![1654583078332](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654583078332.png)

### 2）具体配置如下

**在hdfs-site.xml文件中添加如下内容。**

```bash
<property>
     <name>dfs.datanode.data.dir</name>
     <value>file://${hadoop.tmp.dir}/dfs/data1,file://${hadoop.tmp.dir}/dfs/data2</value>
</property>
```

## 1.2 集群数据均衡之磁盘间数据均衡

**生产环境，由于硬盘空间不足，往往需要增加一块硬盘。刚加载的硬盘没有数据时，可以执行磁盘数据均衡命令。（Hadoop3.x新特性）。**

（1）生成均衡计划（我们只有一块磁盘，不会生成计划）
		`hdfs diskbalancer -plan hadoop103`
（2）执行均衡计划
		`hdfs diskbalancer -execute hadoop103.plan.json`
（3）查看当前均衡任务的执行情况
		`hdfs diskbalancer -query hadoop103`
（4）取消均衡任务

​		`hdfs diskbalancer -cancel hadoop103.plan.json`

# 第2章 HDFS—集群扩容及缩容

## 2.1 添加白名单

​	白名单：表示在白名单的主机IP地址可以，用来存储数据。
​	企业中：配置白名单，可以尽量防止黑客恶意访问攻击。

![1654584074353](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654584074353.png)

​	**配置白名单步骤如下：**



**1）在NameNode节点的/opt/module/hadoop-3.1.3/etc/hadoop目录下分别创建whitelist 和blacklist文件**
（1）创建白名单</u>
`[atguigu@hadoop102 hadoop]$ vim whitelist`
在whitelist中添加如下主机名称，假如集群正常工作的节点为102 103 

```properties
hadoop102
hadoop103
```

（2）创建黑名单
`[atguigu@hadoop102 hadoop]$ touch blacklist`
	保持空的就可以。
2）在hdfs-site.xml配置文件中增加dfs.hosts配置参数

```properties
<!-- 白名单 -->
<property>
     <name>dfs.hosts</name>
     <value>/opt/module/hadoop-3.1.3/etc/hadoop/whitelist</value>
</property>

<!-- 黑名单 -->
<property>
     <name>dfs.hosts.exclude</name>
     <value>/opt/module/hadoop-3.1.3/etc/hadoop/blacklist</value>
</property>
```

3）分发配置文件whitelist，hdfs-site.xml
	`[atguigu@hadoop104 hadoop]$ xsync hdfs-site.xml whitelist`
4）第一次添加白名单必须重启集群，不是第一次，只需要刷新NameNode节点即可
	`[atguigu@hadoop102 hadoop-3.1.3]$ myhadoop.sh stop`
	`[atguigu@hadoop102 hadoop-3.1.3]$ myhadoop.sh start`
5）在web浏览器上查看DN，http://hadoop102:9870/dfshealth.html#tab-datanode

![1654584718377](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654584718377.png)

6）二次修改白名单，增加hadoop104
		`[atguigu@hadoop102 hadoop]$ vim whitelist`
修改为如下内容

```properties
hadoop102
hadoop103
hadoop104
```

7）刷新NameNode
	`[atguigu@hadoop102 hadoop-3.1.3]$ hdfs dfsadmin -refreshNodes`
	`Refresh nodes successful`
8）在web浏览器上查看DN，http://hadoop102:9870/dfshealth.html#tab-datanode

![1654584823269](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654584823269.png)



## 2.2 服役新服务器

##### 1）需求

随着公司业务的增长，数据量越来越大，原有的数据节点的容量已经不能满足存储数据的需求，需要在原有集群基础上动态添加新的数据节点。

##### 2）环境准备

**（1）在hadoop100主机上再克隆一台hadoop105主机**

**（2）修改IP地址和主机名称**

```properties
[root@hadoop105 ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
[root@hadoop105 ~]# vim /etc/hostname
```

**（3）拷贝hadoop102的/opt/module目录和/etc/profile.d/my_env.sh到hadoop105**

```properties
[atguigu@hadoop102 opt]$ scp -r module/* atguigu@hadoop105:/opt/module/

[atguigu@hadoop102 opt]$ sudo scp /etc/profile.d/my_env.sh root@hadoop105:/etc/profile.d/my_env.sh
[atguigu@hadoop105 hadoop-3.1.3]$ source /etc/profile
```

**（4）删除hadoop105上Hadoop的历史数据，data和log数据**

​	`[atguigu@hadoop105 hadoop-3.1.3]$ rm -rf data/ logs/`

**（5）配置hadoop102和hadoop103到hadoop105的ssh无密登录**

​	`[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop105`

​	`[atguigu@hadoop103 .ssh]$ ssh-copy-id hadoop105`

##### 3）服役新节点具体步骤

（1）直接启动DataNode，即可关联到集群

```properties
[atguigu@hadoop105 hadoop-3.1.3]$ hdfs --daemon start datanode
[atguigu@hadoop105 hadoop-3.1.3]$ yarn --daemon start nodemanager
```

![1654585060080](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654585060080.png)

##### 4）在白名单中增加新服役的服务器

**（1）在白名单whitelist中增加hadoop104、hadoop105，并重启集群**
`[atguigu@hadoop102 hadoop]$ vim whitelist`
修改为如下内容

```properties
hadoop102
hadoop103
hadoop104
hadoop105
```

**（2）分发**
`[atguigu@hadoop102 hadoop]$ xsync whitelist`
**（3）刷新NameNode**

```properties
[atguigu@hadoop102 hadoop-3.1.3]$ hdfs dfsadmin -refreshNodes
Refresh nodes successful
```

**5）在hadoop105上上传文件**
`[atguigu@hadoop105 hadoop-3.1.3]$ hadoop fs -put /opt/module/hadoop-3.1.3/LICENSE.txt /`

![1654585254474](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220607-1654585254474.png)



# 磁盘格式化

1. VMware里面添加磁盘，重启虚拟机

2. 新建分区，并进行磁盘格式化

   ```
   sudo fdisk /dev/sdb
   n 多个回车
   w
   ```

   ```bash
   #格式化
   sudo mkfs.ext4 /dev/sdb1
   #挂载
   sudo mount /dev/sdb1 /opt/module/hadoop-3.1.3/data/dfs/data2
   ```

# HDFS服役新节点

1. 在新节点105上准备好Java和Hadoop环境

   将102的Java和Hadoop目录拷贝到105

   删除102的脏数据

2. 添加105的/etc/hosts信息

3. *配置免密（为了将来群起集群）和workers文件

4. 将集群的配置文件同步到105

   ```bash
   rsync -av /opt/module/hadoop-3.1.3/etc hadoop105:/opt/module/hadoop-3.1.3
   ```

5. 在105上启动Datanode

   ```bash
   hdfs --daemon start datanode
   ```

# HDFS退役旧节点

1. 添加黑白名单

   - 修改hdfs-site.xml配置

     ```xml
       <!-- 白名单 -->
       <property>
         <name>dfs.hosts</name>
         <value>/opt/module/hadoop-3.1.3/etc/hadoop/whitelist</value>
       </property>
     
       <!-- 黑名单 -->
       <property>
         <name>dfs.hosts.exclude</name>
         <value>/opt/module/hadoop-3.1.3/etc/hadoop/blacklist</value>
       </property>
     ```

   - 在相应的位置新建黑白名单文件

     ```bash
     touch /opt/module/hadoop-3.1.3/etc/hadoop/whitelist
     touch /opt/module/hadoop-3.1.3/etc/hadoop/blacklist
     echo "hadoop102" >> /opt/module/hadoop-3.1.3/etc/hadoop/whitelist
     echo "hadoop103" >> /opt/module/hadoop-3.1.3/etc/hadoop/whitelist
     echo "hadoop104" >> /opt/module/hadoop-3.1.3/etc/hadoop/whitelist
     echo "hadoop105" >> /opt/module/hadoop-3.1.3/etc/hadoop/whitelist
     ```

   - 重启namenode

     ```bash
     hdfs --daemon stop namenode
     hdfs --daemon start namenode
     ```

2. 通过黑名单退役节点

   - 如果想退役105节点，首先在黑名单中加入hadoop105

     ```bash
     echo "hadoop105" >> /opt/module/hadoop-3.1.3/etc/hadoop/blacklist
     ```

   - 刷新节点

     ```bash
     hdfs dfsadmin -refreshNodes
     ```

   - 等待105退役完毕，在105上关闭datanode

     ```bash
     hdfs --daemon stop datanode
     ```

3. 通过改变白名单，限制和NN通讯的DN

   - 将hadoop105从白名单文件中删除

     ```bash
     sed -i '/hadoop105/d' /opt/module/hadoop-3.1.3/etc/hadoop/whitelist
     ```

   - 刷新节点

     ```bash
     hdfs dfsadmin -refreshNodes
     ```

# 集群安全模式修复

1. 当集群的数据块出现全部丢失的情况，丢失数据过多时，集群重启后就会进入安全模式无法退出

2. 修复方案：首先强制退出安全模式

   ```
   hdfs dfsadmin -safemode forceExit
   ```

3. 删除丢失的数据块

   ```bash
   hdfs fsck -delete /
   ```

   

