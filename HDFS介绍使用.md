# 框架运行出错怎么办

1. 定位错误的进程（我们的例子是Datanode）

2. 查看错误进程的日志（去DN挂掉的节点，找到其日志，例子里面我们查看hadoop102的DN日志）

3. 定位运行日志位置

   日志的位置就在Hadoop家目录下logs文件夹里面

4. 查看日志

   ```bash
   tail -n 100 /opt/module/hadoop-3.1.3/logs/hadoop-atguigu-datanode-hadoop102.log
   ```

   ```
   2022-05-30 08:46:50,265 INFO org.eclipse.jetty.server.AbstractConnector: Started ServerConnector@3406472c{HTTP/1.1,[http/1.1]}{localhost:33538}
   2022-05-30 08:46:50,265 INFO org.eclipse.jetty.server.Server: Started @1003ms
   2022-05-30 08:46:50,338 INFO org.apache.hadoop.hdfs.server.datanode.web.DatanodeHttpServer: Listening HTTP traffic on /0.0.0.0:9864
   2022-05-30 08:46:50,341 INFO org.apache.hadoop.util.JvmPauseMonitor: Starting JVM pause monitor
   2022-05-30 08:46:50,359 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: dnUserName = atguigu
   2022-05-30 08:46:50,359 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: supergroup = supergroup
   2022-05-30 08:46:50,392 INFO org.apache.hadoop.ipc.CallQueueManager: Using callQueue: class java.util.concurrent.LinkedBlockingQueue queueCapacity: 1000 scheduler: class org.apache.hadoop.ipc.DefaultRpcScheduler
   2022-05-30 08:46:50,402 INFO org.apache.hadoop.ipc.Server: Starting Socket Reader #1 for port 9867
   2022-05-30 08:46:50,499 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Opened IPC server at /0.0.0.0:9867
   2022-05-30 08:46:50,510 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Refresh request received for nameservices: null
   2022-05-30 08:46:50,514 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Starting BPOfferServices for nameservices: <default>
   2022-05-30 08:46:50,521 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Block pool <registering> (Datanode Uuid unassigned) service to hadoop102/192.168.10.102:8020 starting to offer service
   2022-05-30 08:46:50,525 INFO org.apache.hadoop.ipc.Server: IPC Server Responder: starting
   2022-05-30 08:46:50,525 INFO org.apache.hadoop.ipc.Server: IPC Server listener on 9867: starting
   2022-05-30 08:46:50,676 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Acknowledging ACTIVE Namenode during handshakeBlock pool <registering> (Datanode Uuid unassigned) service to hadoop102/192.168.10.102:8020
   2022-05-30 08:46:50,678 INFO org.apache.hadoop.hdfs.server.common.Storage: Using 2 threads to upgrade data directories (dfs.datanode.parallel.volumes.load.threads.num=2, dataDirs=2)
   2022-05-30 08:46:50,681 INFO org.apache.hadoop.hdfs.server.common.Storage: Lock on /opt/module/hadoop-3.1.3/data/dfs/data/in_use.lock acquired by nodename 2777@hadoop102
   2022-05-30 08:46:50,682 WARN org.apache.hadoop.hdfs.server.common.Storage: Failed to add storage directory [DISK]file:/opt/module/hadoop-3.1.3/data/dfs/data
   java.io.IOException: Incompatible clusterIDs in /opt/module/hadoop-3.1.3/data/dfs/data: namenode clusterID = CID-6521df2b-56c9-4043-929d-01e57a85738e; datanode clusterID = CID-963fc533-014b-407e-a3cc-29021825ed8b
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.doTransition(DataStorage.java:744)
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.loadStorageDirectory(DataStorage.java:294)
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.loadDataStorage(DataStorage.java:407)
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.addStorageLocations(DataStorage.java:387)
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.recoverTransitionRead(DataStorage.java:559)
   	at org.apache.hadoop.hdfs.server.datanode.DataNode.initStorage(DataNode.java:1743)
   	at org.apache.hadoop.hdfs.server.datanode.DataNode.initBlockPool(DataNode.java:1679)
   	at org.apache.hadoop.hdfs.server.datanode.BPOfferService.verifyAndSetNamespaceInfo(BPOfferService.java:390)
   	at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.connectToNNAndHandshake(BPServiceActor.java:282)
   	at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.run(BPServiceActor.java:822)
   	at java.lang.Thread.run(Thread.java:748)
   2022-05-30 08:46:50,683 INFO org.apache.hadoop.hdfs.server.common.Storage: Lock on /opt/module/hadoop-3.1.3/data/dfs/data2/in_use.lock acquired by nodename 2777@hadoop102
   2022-05-30 08:46:50,683 WARN org.apache.hadoop.hdfs.server.common.Storage: Failed to add storage directory [DISK]file:/opt/module/hadoop-3.1.3/data/dfs/data2
   java.io.IOException: Incompatible clusterIDs in /opt/module/hadoop-3.1.3/data/dfs/data2: namenode clusterID = CID-6521df2b-56c9-4043-929d-01e57a85738e; datanode clusterID = CID-963fc533-014b-407e-a3cc-29021825ed8b
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.doTransition(DataStorage.java:744)
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.loadStorageDirectory(DataStorage.java:294)
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.loadDataStorage(DataStorage.java:407)
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.addStorageLocations(DataStorage.java:387)
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.recoverTransitionRead(DataStorage.java:559)
   	at org.apache.hadoop.hdfs.server.datanode.DataNode.initStorage(DataNode.java:1743)
   	at org.apache.hadoop.hdfs.server.datanode.DataNode.initBlockPool(DataNode.java:1679)
   	at org.apache.hadoop.hdfs.server.datanode.BPOfferService.verifyAndSetNamespaceInfo(BPOfferService.java:390)
   	at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.connectToNNAndHandshake(BPServiceActor.java:282)
   	at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.run(BPServiceActor.java:822)
   	at java.lang.Thread.run(Thread.java:748)
   2022-05-30 08:46:50,684 ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: Initialization failed for Block pool <registering> (Datanode Uuid c2c33d95-c940-471a-89e0-e6b734140b06) service to hadoop102/192.168.10.102:8020. Exiting. 
   java.io.IOException: All specified directories have failed to load.
   	at org.apache.hadoop.hdfs.server.datanode.DataStorage.recoverTransitionRead(DataStorage.java:560)
   	at org.apache.hadoop.hdfs.server.datanode.DataNode.initStorage(DataNode.java:1743)
   	at org.apache.hadoop.hdfs.server.datanode.DataNode.initBlockPool(DataNode.java:1679)
   	at org.apache.hadoop.hdfs.server.datanode.BPOfferService.verifyAndSetNamespaceInfo(BPOfferService.java:390)
   	at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.connectToNNAndHandshake(BPServiceActor.java:282)
   	at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.run(BPServiceActor.java:822)
   	at java.lang.Thread.run(Thread.java:748)
   2022-05-30 08:46:50,684 WARN org.apache.hadoop.hdfs.server.datanode.DataNode: Ending block pool service for: Block pool <registering> (Datanode Uuid c2c33d95-c940-471a-89e0-e6b734140b06) service to hadoop102/192.168.10.102:8020
   2022-05-30 08:46:50,785 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Removed Block pool <registering> (Datanode Uuid c2c33d95-c940-471a-89e0-e6b734140b06)
   2022-05-30 08:46:52,785 WARN org.apache.hadoop.hdfs.server.datanode.DataNode: Exiting Datanode
   2022-05-30 08:46:52,788 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: SHUTDOWN_MSG: 
   /************************************************************
   SHUTDOWN_MSG: Shutting down DataNode at hadoop102/192.168.10.102
   ************************************************************/
   ```

5. 定位日志错误

   日志按行分为四个级别：

   | 级别  | 意义                                                     |
   | ----- | -------------------------------------------------------- |
   | INFO  | 框架正常运行的日志，一般不用管                           |
   | WARN  | 警告：需要提起注意的地方，如果其没有导致错误，可以不用管 |
   | ERROR | 错误：框架运行不正常，需要修复                           |
   | FATAL | 致命错误：框架因为这个挂掉了                             |

   如果需要排错，找ERROR和FATAL

   在我们的例子里面：错误是因为NN和DN的集群ID不一致导致

6. 修复错误

   将NN的VERSION文件的clusterID改为和DN一致，重启集群



# 上课过程中常用的脚本

1. xsync 集群文件同步脚本

```sql
xsync /opt/module/hadoop-3.1.3/etc/hadoop/
```

1. xcall 集群执行指令【集群同步执行同一个命令】

   ```sql
   # 先安装pdsh
   sudo yum install -y epel-release
   sudo yum install -y pdsh
   ```

   ```bash
   # pdsh使用帮助
   pdsh -w 'hadoop102,hadoop103,hadoop104' 'jps'
   ```

   ```bash
   # 封装xcall脚本
   sudo vim /bin/xcall
   ```

   粘贴如下内容：

   ```bash
   #!/bin/bash
   pdsh -w 'hadoop102,hadoop103,hadoop104' "$*" | awk -F ":" '{temp=$1;$1=null;array[temp]=array[temp]$0"\n"}END{for (i in array) {print "===========  "i"  ============\n"array[i]}}'
   ```

   ```bash
   # 保存退出，增加执行权限
   sudo chmod +x /bin/xcall
   ```

2. jpsall 快速查看三个节点的Java进程

   ```bash
   # 封装jpsall脚本
   sudo vim /bin/jpsall
   ```

   粘贴如下内容：

   ```bash
   #!/bin/bash
   xcall jps | grep -v Jps
   ```

   ```bash
   # 保存退出，增加执行权限
   sudo chmod +x /bin/jpsall
   ```

   # 历史服务器和日志聚集

   1. mapred-site.xml

      添加如下内容

      ```xml
        <!-- 历史服务器端地址 -->
        <property>
          <name>mapreduce.jobhistory.address</name>
          <value>hadoop102:10020</value>
        </property>
      
        <!-- 历史服务器web端地址 -->
        <property>
          <name>mapreduce.jobhistory.webapp.address</name>
          <value>hadoop102:19888</value>
        </property>
      ```

   2. yarn-site.xml

      ```xml
        <!-- 开启日志聚集功能 -->
        <property>
          <name>yarn.log-aggregation-enable</name>
          <value>true</value>
        </property>
        <!-- 设置日志聚集服务器地址 -->
        <property>  
          <name>yarn.log.server.url</name>  
          <value>http://hadoop102:19888/jobhistory/logs</value>
        </property>
        <!-- 设置日志保留时间为7天 -->
        <property>
          <name>yarn.log-aggregation.retain-seconds</name>
          <value>604800</value>
        </property>
      ```

   3. 同步配置文件

      ```bash
      xsync  /opt/module/hadoop-3.1.3/etc
      ```

   4. 重启集群，并启动历史服务器

      ```bash
      # 关闭集群
      stop-dfs.sh
      stop-yarn.sh
      
      # 启动集群
      start-dfs.sh
      start-yarn.sh
      ```

      ```bash
      # 手动启动历史服务器
      mapred --daemon start historyserver
      ```

   5. 再跑一个mapreduce，测试历史服务器工作是否正常

      ```bash
      hadoop fs -mkdir /input
      hadoop fs -put /opt/module/hadoop-3.1.3/LICENSE.txt /input
      ```

      ```bash
      # 提交新的mr程序
      hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input /output
      ```

      完成以后，在网页上查看刚刚跑过的mr程序的日志信息



# 集群时间同步

1. NTP服务

   Network Time Protocol, 网络时间协议，是一个让本地计算机和网络上一台服务器时间保持一致的服务

   ```bash
   # 首先需要安装ntp服务
   sudo yum install -y ntp
   
   # 安装完成启动ntp服务
   sudo systemctl enable ntpd
   sudo systemctl start ntpd
   ```

2. 手动和网络上的服务器进行时间对齐

   ```bash
   sudo ntpdate ntp.aliyun.com
   ```

   ```bash
   # 如果给三台服务器对时间
   sudo xcall 'ntpdate ntp.aliyun.com'
   ```

3. 如果没有外部网络

   - 将102配置成能够提供时间同步服务的服务器

     ```bash
     # 首先需要安装ntp服务
     sudo yum install -y ntp
     
     # 安装完成启动ntp服务
     sudo systemctl enable ntpd
     sudo systemctl start ntpd
     ```

     ```bash
     # 配置102的ntp服务
     sudo vim /etc/ntp.conf
     ```

     修改如下位置

     将

     ```ini
     #restrict 192.168.10.0 mask 255.255.255.0 nomodify notrap
     ```

     改为

     ```ini
     restrict 192.168.10.0 mask 255.255.255.0 nomodify notrap
     ```

     将

     ```ini
     server 0.centos.pool.ntp.org iburst
     server 1.centos.pool.ntp.org iburst
     server 2.centos.pool.ntp.org iburst
     server 3.centos.pool.ntp.org iburst
     ```

     改为

     ```ini
     #server 0.centos.pool.ntp.org iburst
     #server 1.centos.pool.ntp.org iburst
     #server 2.centos.pool.ntp.org iburst
     #server 3.centos.pool.ntp.org iburst
     
     ```

     在任意位置添加

     ```ini
     server 127.127.1.0
     fudge 127.127.1.0 stratum 10
     
     ```

     保存退出，重启102的时间同步服务

     ```bash
     sudo systemctl restart ntpd
     
     ```

   - 103，104定期同步102时间

     ```bash
     # 在103和104设置定时任务
     sudo crontab -e
     
     ```

     添加

     ```bash
     * * * * * /usr/sbin/ntpdate hadoop102
     
     ```

     保存并退出

   - 改变103，104时间，看看能否自动纠正

     ```
     # 修改103，104时间
     sudo pdsh -w 'hadoop103,hadoop104' 'date -s "2011-01-01"'
     
     ```






   # 节点状态诊断

   1. 联网状态

      - 外网

        ```bash
        ping -c 4 www.baidu.com
        ```

        如果出现类似下面的显示，就没问题

        ```
        PING www.a.shifen.com (110.242.68.3) 56(84) bytes of data.
        64 bytes from 110.242.68.3 (110.242.68.3): icmp_seq=1 ttl=128 time=10.2 ms
        64 bytes from 110.242.68.3 (110.242.68.3): icmp_seq=2 ttl=128 time=10.4 ms
        64 bytes from 110.242.68.3 (110.242.68.3): icmp_seq=3 ttl=128 time=10.3 ms
        64 bytes from 110.242.68.3 (110.242.68.3): icmp_seq=4 ttl=128 time=10.5 ms
        
        --- www.a.shifen.com ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3005ms
        rtt min/avg/max/mdev = 10.262/10.401/10.542/0.144 ms
        ```

      - 内网

        ```bash
        ping -c 4 hadoop102
        ping -c 4 hadoop103
        ping -c 4 hadoop104
        ```

   2. 权限状态

      - atguigu的sudo权限

        ```bash
        sudo ls
        ```

        如果不会报错就没事

      - /opt/module文件夹的所有者和所属组

        ```
        ll /opt/module
        ```

        查看所有者和所属组

        ```
        总用量 4
        drwxrwxr-x.  4 atguigu atguigu 4096 5月  30 09:36 datas
        drwxr-xr-x. 12 atguigu atguigu  190 5月  30 08:45 hadoop-3.1.3
        drwxrwxr-x. 10 atguigu atguigu  184 5月  21 11:16 hive
        drwxr-xr-x.  7 atguigu atguigu  245 4月   2 2019 jdk1.8.0_212
        drwxrwxr-x.  8 atguigu atguigu  160 5月  18 14:22 zookeeper
        ```

        如果不对，用以下指令修复

        ```bash
        sudo chown -R atguigu:atguigu /opt/*
        ```

   3. 免密配置

      - 免密设置

      ```properties
      ssh-keygen -t rsa
      密钥对产生位置：账户目录
      id_rsa 私钥
      id_rsa.pub 公钥
      ssh-copy-id hadoop103
      ```
      
      

      - atguigu免密
      
        ```
        ssh hadoop102
        ```

~~~properties
  ```
    sudo ssh hadoop104
    exit
    ```
  
    如果登录不上，用root用户互相配置一下免密
~~~

###    集群启动步骤

- 1）如果集群是第一次启动，需要在hadoop102节点格式化NameNode
  - 在hadoop的解压目录bin下

```
 hdfs namenode -format
```

- 群起hadoop服务

- web端查看HDFS的NameNode

  - ①浏览器中输入：http://hadoop102:9870

  - ②查看HDFS上存储的数据信息

- （5）Web端查看YARN的ResourceManager

  - ①浏览器中输入：http://hadoop103:8088

  - ②查看YARN上运行的Job信息

- 历史服务器的地址

  - http://hadoop102:19888/jobhistory


## HDFS

#### HDFS优点

![1653909551077](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220530-1653909551077.png)

- 缺点

![1653909586540](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220530-1653909586540.png)

#### HDFS组成架构

![1653909640321](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/2205301653909640321.png)

#### HDFS文件块的大小

![1653909724884](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220530-1653909724884.png)

### 2.3.2上传

1）-moveFromLocal：从本地剪切粘贴到HDFS

```bash
[atguigu@hadoop102 hadoop-3.1.3]$ vim shuguo.txt
输入：
shuguo
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs  -moveFromLocal  ./shuguo.txt  /sanguo
```

2）-copyFromLocal：从本地文件系统中拷贝文件到HDFS路径去

```bash
[atguigu@hadoop102 hadoop-3.1.3]$ vim weiguo.txt
输入：
weiguo
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -copyFromLocal weiguo.txt /sanguo
```

3）-put：等同于copyFromLocal，生产环境更习惯用put

```bash
[atguigu@hadoop102 hadoop-3.1.3]$ vim wuguo.txt
输入：
wuguo
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -put ./wuguo.txt /sanguo
```

4）-appendToFile：追加一个文件到已经存在的文件末尾

```bash
[atguigu@hadoop102 hadoop-3.1.3]$ vim liubei.txt
输入：
liubei

[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -appendToFile liubei.txt /sanguo/shuguo.txt
```



### 2.3.3下载

1）-copyToLocal：从HDFS拷贝到本地

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -copyToLocal /sanguo/shuguo.txt ./
```

2）-get：等同于copyToLocal，生产环境更习惯用get

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -get /sanguo/shuguo.txt ./shuguo2.txt
```



### **2.3.4** **HDFS直接操作**

1）-ls: 显示目录信息

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -ls /sanguo
```

2）-cat：显示文件内容

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -cat /sanguo/shuguo.txt
```

3）-chgrp、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs  -chmod 666  /sanguo/shuguo.txt

[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs  -chown  atguigu:atguigu  /sanguo/shuguo.txt
```

4）-mkdir：创建路径

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mkdir /jinguo
```

5）-cp：从HDFS的一个路径拷贝到HDFS的另一个路径

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -cp /sanguo/shuguo.txt /jinguo
```

6）-mv：在HDFS目录中移动文件

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mv /sanguo/wuguo.txt /jinguo

[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mv /sanguo/weiguo.txt /jinguo
```

7）-tail：显示一个文件的末尾1kb的数据

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -tail /jinguo/shuguo.txt
```

8）-rm：删除文件或文件夹

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -rm /sanguo/shuguo.txt
```

9）-rm -r：递归删除目录及目录里面内容

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -rm -r /sanguo
```

10）-du统计文件夹的大小信息

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -du -s -h /jinguo
27  81  /jinguo
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -du  -h /jinguo
14  42  /jinguo/shuguo.txt
7  21  /jinguo/weiguo.txt
6  18  /jinguo/wuguo.tx
```

