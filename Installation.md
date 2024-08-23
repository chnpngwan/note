# 前置准备

1. <a id="fire">关闭防火墙</a>

   1.1.查看防火墙状态

   ```
   systemctl status firewalld
   ```

   1.2. 暂时关闭防火墙

   ```properties
   systemctl stop firewalld
   ```

   1.3. 永久关闭防火墙

   ```properties
   systemctl disable firewalld
   ```

   1.4. 查看防火墙状态

   ```properties
   systemctl status firewalld		#提示 Active: inactive (dead)
   则关闭成功
   ```

   

2. <a id="SELINUX">关闭SELINUX</a>

   - ```properties
     SELINUX=disabled
     sed -i 's/SELINUX=enforcing/SELINUX=disabled/g'  /etc/selinux/config
     sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
     swapoff -a
     ```

3. <a id="date">时间同步(可选)(5911)</a>

   - `tzselect`

# JDK安装

1、确认当前Linux中是否自带安装jdk

```
检查是否自带JDK：rpm -qa | grep java
如果出现了内容，则进行如下命令卸载：
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64
【--nodeps后面的内容则是rpm -qa 查询出来的内容复制过来】
```

2、正式开始安装Jdk

2.1、解压jdk压缩文件到指定目录

```
tar -zxvf jdk...tar.gz -C /opt
```

2.2、进入/usr/local/src目录，对解压后的jdk目录重命名【改短一点】

```
cd /usr/local/src
mv jdk1.8.0_212/   jdk
```

2.3、配置Jdk环境变量【linux能够很好的识别到Java相关的命令】

2.3.1、针对root账户生效的环境变量文件：vim /root/.bash_profile

2.3.2、针对所有账户生效的环境变量文件：vim /etc/profile

```
编辑环境变量配置文件加入下列内容： 
export JAVA_HOME=/usr/local/src/jdk
export PATH=$PATH:$JAVA_HOME/bin
```

2.4、然后环境变量马上生效： source  /root/.bash_profile

2.5、测试jdk是否安装成功：

```
java -version 查看jdk版本
java或者javac
```

# Scala安装

- 解压

  `tar -zxvf /opt/soft/scala-2.11.0.tgz -C /usr/local/src/`

- 重命名

- `mv scala-2.11.0 scala`

- 分发

  `scp -r /usr/local/src/scala slave1:/usr/local/src/`

  `scp -r /usr/local/src/scala slave1:/usr/local/src/`

# MySQL安装

1、因centos7中自带了mariadb数据库，会和mysql发生冲突，先将其卸载了

```
rpm -qa | grep mariadb
如果查询出来有内容，则将其卸载，命令如下：
rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
```

2、创建mysql用户组和用户

```
groupadd mysql
useradd -r -g mysql mysql
```

3、解压mysql安装包（mysql-5.7.40-linux-glibc2.12-x86_64.tar.gz）到指定位置

```
 tar -zxvf /mysoft/mysql-5.7.40-linux-glibc2.12-x86_64.tar.gz -C /usr/local/src/
```

4、进入/usr/local/src下，将解压后的目录改个名字 

```
mv mysql-5.7.40-linux-glibc2.12-x86_64/ mysql5.7
```

5、修改mysql5.7目录及子目录权限：

```
chown -R mysql:mysql /usr/local/src/mysql
chmod -R 755 /usr/local/src/mysql/
```

6、进入/usr/local/src/mysql5.7/bin/下，执行初始化mysql命令： 

```
 ./mysqld --initialize --user=root --datadir=/usr/local/src/mysql/data --basedir=/usr/local/src/mysql
【注意：将初始化最后生成的mysql第一次默认登录密码复制出来】
```

7、新建编辑/etc/my.cnf

```
[mysqld]
datadir=/usr/local/src/mysql/data
port=3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
symbolic-links=0
max_connections=400
innodb_file_per_table=1
#表名大小写不明感，敏感为
lower_case_table_names=1


将/etc/my.cnf进行授权：chmod -R 775 /etc/my.cnf
```

8、修改/usr/local/src/mysql5.7/support-files/目录下的mysql.server文件，将 if test-z "$basedir"

```
if test -z "$basedir"
then
  basedir=/usr/local/src/mysql5.7      （1个）
  bindir=/usr/local/src/mysql5.7/bin   （2个）
  if test -z "$datadir"
  then
    datadir=/usr/local/src/mysql5.7/data（3个）
  fi
  sbindir=/usr/local/src/mysql5.7/bin（4个）
  libexecdir=/usr/local/src/mysql5.7/bin（5个）
```

9、启动服务：进入到/usr/local/src/support-files/目录下执行：

```
/usr/local/src/support-files/mysql.server start
【不出意外会显示SUCCESS成功】
```

10、创建软连接方便后续启动mysql

    ln -s /usr/local/src/mysql5.7/support-files/mysql.server /etc/init.d/mysql
    ln -s /usr/local/src/mysql5.7/bin/mysql /usr/bin/mysql
    测试重新启动mysql：service mysql restart

11、登录mysql修改初始密码： 

```
mysql -u root -p  
输入前面初始化后的第一次默认密码

在mysql的命令行中执行：set password for root@localhost = password('123456');
```

12、远程授权，在mysql命令行下执行：

```
授权mysql远程访问（否则其他机器无法连接上该mysql）
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.xx.xx' IDENTIFIED BY '123456' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY '123456' WITH GRANT OPTION;
flush privileges;
```



# Hive安装

- 解压

  `tar -zxvf /opt/soft/apache-hive-2.3.4-bin.tar.gz -C /usr/local/src/`

- 重命名

  `mv apache-hive-2.3.4-bin hive`

- 配置文件(一个,需复制 ` hive-default.xml.template `        改五  删三)

  ` cp hive-default.xml.template hive-site.xml`

  `vi hive-site.xml`

  - 改动五处(数据库链接地址,驱动 , 用户名 , 密码,版本效验)

    ```xml
      <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://localhost:3306/hiveNo12?createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8&amp;useSSL=false</value>
      </property>
    
      <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
      </property>
    
      <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
        <description>Username to use against metastore database</description>
      </property>
    
      <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
      </property>
    
     <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
      </property>
    ```
    
  - 删除三处
  
    ```xml
      <property>
        <name>hive.querylog.location</name>
        <value></value>
        <description>Location of Hive run time structured log file</description>
      </property>
    
      <property>
        <name>hive.downloaded.resources.dir</name>
        <value></value>
        <description>Temporary local directory for added resources in the remote file system.</description>
      </property>
    
      <property>
        <name>hive.exec.local.scratchdir</name>
        <value></value>
        <description>Local scratch space for Hive jobs</description>
      </property>
    
    
    ```
  
- 将mysql的驱动包拷贝至 hive/lib下

- 初始化hive元数据

  ` schematool -dbType mysql -initSchema`

- 进入hive验证

# Hadoop安装

## 1、单机

### 前置

1. <a id="hostname">修改主机名</a>

   - `vi /etc/hostname`(对应master)

2. <a id="hosts">添加映射(IP <--> 主机名)</a>

   - `vi /etc/hosts`

     ```properties
      xxx.xxx.xxx master
     ```

     

3. <a id="reboot">重启机器(使修改的主机名生效)</a>

   - `reboot`

<a id="PATH">4 : PATH</a>

- <a id="profile">配置`/etc/profile`</a>

  ```properties
  # Java、Scala环境配置
  export JAVA_HOME=/opt/jdk
  
  # Hadoop、hive环境配置
  export HADOOP_HOME=/opt/hadoop
  export HIVE_HOME=/opt/hive
  
  export PATH=$PATH:$JAVA_HOME/bin:$SCALA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin
  ```
  
  
  
- <a id="env">创建并编辑`/etc/profile.d/`下`env.sh`文件</a>

  ```properties
  # E:阶段环境配置
  export JAVA_HOME=/usr/local/src/jdk
  export SCALA_HOME=/usr/local/src/scala
  
  # F:阶段环境配置
  export HADOOP_HOME=/usr/local/src/hadoop
  export SPARK_HOME=/usr/local/src/spark
  export HIVE_HOME=/usr/local/src/hive
  export FLINK_HOME=/usr/local/src/flink
  
  # 其他组件
  export ZOOKEEPER_HOME=/usr/local/src/zookeeper
  export KAFKA_HOME=/usr/local/src/kafka
  export FLUME_HOME=/usr/local/src/flume
  
  
  export PATH=$PATH:$JAVA_HOME/bin:$SCALA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$SPARK_HOME/bin:$HIVE_HOME/bin:$FLINK_HOME/bin:$ZOOKEEPER_HOME/bin:$KAFKA_HOME/bin:$FLUME_HOME/bin
  
  ```

- 生效

  - `. /etc/profile`
  - `soure /etc/profile`

- 验证

  - `echo $XXXX_HOME`

### 配置Hadoop


- 解压

  `tar -zxvf /opt/soft/hadoop-2.7.7.tar.gz -C /usr/local/src/`

- 重命名

  `mv hadoop-2.7.7 hadoop`

- 配置文件(六个)

  - `hadoop-env.sh`

    ```properties
    #原内容
    # The java implementation to use.
    export JAVA_HOME=${JAVA_HOME}
    #修改为(jdk的绝对路径)
    # The java implementation to use.
    export JAVA_HOME=/usr/local/src/jdk
    ```

    

  - `core-site.xml`

    ```xml
    <property>
      <!--namenode的URL地址-->
      <name>fs.defaultFS</name>
      <value>hdfs://master:9000</value>
    </property>
    
    <property>
      <!--hadoop临时文件路径-->
      <name>hadoop.tmp.dir</name>
      <value>/opt/data/hadoop</value>
    </property>
    ```

    

  - `hdfs-site.xml`

    ```xml
    <property>
      <name>dfs.replication</name>
      <value>1</value>
    </property>
    ```

    

  - `yarn-site.xml`

    ```xml
    <property>
    <name>yarn.resouremanager.hostname</name>
    <value>master</value>
    </property>
    
    <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
    </property>
    
    <!--关闭虚拟内存验证-->
    <property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
    </property>
    
    <!--关闭物理内存验证-->
    <property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
    </property>
    ```

    

  - `mapred-site.xml`(需复制`mapred-site.xml.template`)

    ```xml
    <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
    </property>
    ```

    - 修改`slaves`

      ```
      master
      ```

      

- 分发

  `scp -r /usr/local/src/scala slave1:/usr/local/src/`

  `scp -r /usr/local/src/scala slave1:/usr/local/src/`

- 格式化

  `hdfs namenode -format`

- 查看`jps`进程

  ```properties
  #master
  2304 ResourceManager
  2736 Jps
  2137 SecondaryNameNode
  1963 DataNode
  1821 NameNode
  2591 NodeManager
  
  
  ```

- 运行自带案例

  ```properties
  hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.7.jar pi 10 10
  ```


## 2、集群

### 前置

1. <a id="hostname">修改主机名</a>

   - `vi /etc/hostname`(对应master slave1 slave2)

2. <a id="hosts">添加映射(IP <--> 主机名)</a>

   - `vi /etc/hosts`

     ```properties
      xxx.xxx.xxx master
      xxx.xxx.xxx slave1
      xxx.xxx.xxx slave2
     ```

     

3. <a id="reboot">重启机器(使修改的主机名生效)</a>

   - `reboot`

<a id ="SSH">4: `ssh`</a>

4.1. 三台机器同时生成公钥

   - `ssh-keygen -t rsa`

4.2. 拷贝公钥到同一台机器上(如master),三台机器同时拷贝

   - `ssh-copy-id master`
4.3. 复制第二步上的(master)的认证到其他机器上

   - `scp -r /root/.ssh/authorized_keys slave1:/root/.ssh`

   - `scp -r /root/.ssh/authorized_keys slave2:/root/.ssh`
4.4. 主节点(master)生成公钥

   - `ssh-keygen -t rsa`
4.5. 拷贝至从机且自身拷贝

   - `ssh-copy-id XXX`

<a id="PATH">5 : PATH</a>

- <a id="profile">配置`/etc/profile`</a>

  ```properties
  # Java、Scala环境配置
  export JAVA_HOME=/usr/local/src/jdk
  export SCALA_HOME=/usr/local/src/scala
  
  # Hadoop、hive环境配置
  export HADOOP_HOME=/usr/local/src/hadoop
  export HIVE_HOME=/usr/local/src/hive
  
  export PATH=$PATH:$JAVA_HOME/bin:$SCALA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin
  ```
  
  
  
- <a id="env">创建并编辑`/etc/profile.d/`下`env.sh`文件</a>

  ```properties
  # E:阶段环境配置
  export JAVA_HOME=/usr/local/src/jdk
  export SCALA_HOME=/usr/local/src/scala
  
  # F:阶段环境配置
  export HADOOP_HOME=/usr/local/src/hadoop
  export SPARK_HOME=/usr/local/src/spark
  export HIVE_HOME=/usr/local/src/hive
  export FLINK_HOME=/usr/local/src/flink
  
  # 其他组件
  export ZOOKEEPER_HOME=/usr/local/src/zookeeper
  export KAFKA_HOME=/usr/local/src/kafka
  export FLUME_HOME=/usr/local/src/flume
  
  
  export PATH=$PATH:$JAVA_HOME/bin:$SCALA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$SPARK_HOME/bin:$HIVE_HOME/bin:$FLINK_HOME/bin:$ZOOKEEPER_HOME/bin:$KAFKA_HOME/bin:$FLUME_HOME/bin
  
  ```

- 生效

  - `. /etc/profile`
  - `soure /etc/profile`

- 验证

  - `echo $XXXX_HOME`

### 配置Hadoop


- 解压

  `tar -zxvf /opt/soft/hadoop-2.7.7.tar.gz -C /usr/local/src/`

- 重命名

  `mv hadoop-2.7.7 hadoop`

- 配置文件(六个)

  - `hadoop-env.sh`

    ```properties
    #原内容
    # The java implementation to use.
    export JAVA_HOME=${JAVA_HOME}
    #修改为(jdk的绝对路径)
    # The java implementation to use.
    export JAVA_HOME=/usr/local/src/jdk
    ```

    

  - `core-site.xml`

    ```xml
    <property>
      <!--namenode的URL地址-->
      <name>fs.defaultFS</name>
      <value>hdfs://master:9000</value>
    </property>
    
    <property>
      <!--hadoop临时文件路径-->
      <name>hadoop.tmp.dir</name>
      <value>/opt/data/hadoop</value>
    </property>
    ```

    

  - `hdfs-site.xml`

    ```xml
    <property>
      <!--hadoop的副本数量，默认为3-->
      <name>dfs.replication</name>
      <value>3</value>
    </property>
    ```

    

  - `yarn-site.xml`

    ```xml
    <property>
    <name>yarn.resouremanager.hostname</name>
    <value>master</value>
    </property>
    
    <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
    </property>
    
    <!--关闭虚拟内存验证-->
    <property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
    </property>
    
    <!--关闭物理内存验证-->
    <property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
    </property>
    ```

    

  - `mapred-site.xml`(需复制`mapred-site.xml.template`)

    ```xml
    <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
    </property>
    ```

    - 修改`slaves`

      ```
      master
      slave1
      slave2
      ```

      

- 分发

  `scp -r /usr/local/src/scala slave1:/usr/local/src/`

  `scp -r /usr/local/src/scala slave1:/usr/local/src/`

- 格式化

  `hdfs namenode -format`

- 查看`jps`进程

  ```properties
  #slave1
  7234 NodeManager
  7362 Jps
  5931 DataNode
  
  
  #slave2
  7057 NodeManager
  7185 Jps
  5732 DataNode
  
  
  #master
  2304 ResourceManager
  2736 Jps
  2137 SecondaryNameNode
  1963 DataNode
  1821 NameNode
  2591 NodeManager
  
  
  ```

- 运行自带案例

  ```properties
  hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.7.jar pi 10 10
  ```

  
