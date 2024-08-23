# Linux系统回顾

### |-- 1：安装版本

  |-- 安装图形界面版本，可以删除了
  |-- 安装基础设施服务器版本，稳定安全
  |-- 安装的是Linux版本+发行版本Centos7

### |-- 2:命令问题

####   |-- 目录类

​    |-- cd 进入目录
​    |-- cd .. 退出当前目录
​    |-- cd~ 回到自己的账户目录
​    |-- mkdir 创建目录
​    |-- rm 删除目录，文件也删除
​    |-- Linux系统没有判断，最顶级目录就是 /

####   |-- 文件类

​    |-- cat 看文件
​    |-- vim 文本编辑器
  |-- 复制命令 cp -r递归



####   |-- 操作系统中的命令

​    |--防火墙操作
​      |-- systemctrl status firewalld 查看防火墙
​      |-- systemctrl start  firewalld 启动防火墙
​      |-- systemctrl stop   firewalld 关闭防火墙
​      |-- systemctrl disable firewalld 永久关闭防火墙
​    

```properties
|-- 账户部分
  |-- useradd 创建账户
  |-- passwd 设置账户密码
  |-- su 切换账户
  |-- sudo 提升账户权限
  |-- groupadd 创建账户组
  |-- chown 改变文件所有者
  |-- chgrp 改变文件所属组
  |-- chmod 修改权限
```

   |-- tar 解压缩
   |-- yum 远程安装软件命令
     |-- -y 不提示，直接确认
     |-- install 安装软件





### JavaSE

  |-- JavaEE技术线
  |-- 大数据技术线
    |-- 大数据技术Hadoop生态圈，程序开发就是Java写

#### MySQL

  |-- 存储数据的服务器
  |-- MySQL技术和大数据技术之间没有什么关系
  |-- 大数据领域中，MySQL存储字典数据
  |-- 学SQL语句

Maven -- 创建项目，管理jar包

#### Linux系统

  |-- 大数据技术以Linux为运行平台
  |-- Hadoop软件运行Linux系统中
  |-- HBASE
  |-- Redis

#### Shell脚本编程

  |-- Shell针对运维技术的
  |-- 搭建大数据的开发环境

### Hadoop课程介绍（综合性的框架）

 1. #### Hadoop是个软件，Java语言编写软件。
    
    Hadoop软件解决海量数据的存储和计算，利用图书馆思想
    图书馆书，分类（数学，年级，作者，出版社）
    当我们在软件存储数据的时候，先记录了数据的元信息
    还要记住文件的存储位置
    
 2. #### Hadoop软件组成部分，三大组件
    
      A: HDFS Hadoop Distributed File System
      Hadoop 分布式文件存储系统
      分布式：一台机器完不成，多台机器一起

   B: MapReduce 数据的分析计算工具
      本质就是Java写的代码
      高德地图：发布明天上午10点中，颐和园周边出现大量客流，减少前往
      移动，联通电话销售
      分片技术：将一个大的计算任务切割进行，减少每台机器压力
      计算后的数据汇总起来

   C: Yarn 是资源分配平台，为MapReduce程序分配硬件资源的
   以上的三个个部分：都学会了，Hadoop技术就毕业了

 3. #### Zookeeper 动物管理员
    
    去中心化设置的，在服务器的集群中，没有首领的说法
    所有的机器都是平等，去中心化设置，目的就是为了搭建一个
    高可用服务器集群（7*24）永不间断
    Zookeeper搭建集群



###  --------------Hadoop环境----------------

#####  1: 虚拟机安装Centos7版本系统，内存4G(2G),硬盘建议50G（20G），安装基础设置服务器

#####  2：关闭防火墙，永久关闭

   关闭：        systemctl stop firewalld
   关闭不在启动：systemctl disable firewalld
   查看防火墙的状态：systemctl status firewalld

#####  3： 修改配置文件 /etc/sysconfig/network-script/ifcfg-ens33

   固定IP地址，修改 dhcp为static，配置IP地址，网关，DNS1 
   重启网卡: systemctl restart network

#####  4: 配置机器名 配置文件 /etc/hostname

   配置为hadoop101,重启虚拟机才能生效（reboot）

#####  5: 创建账户atguigu，密码为123

   useradd atguigu
   passwd atguigu

#####  6: 提升atguigu账户权限，免密执行

   修改配置文件 /etc/sudoers
   zhangsan ALL=(ALL)     NOPASSWD:ALL

#####  7: 创建目录

   /opt/software 目录，存储软件包的
   /opt/module 目录，软件包的解压目录

#####  8：目录software和module

   修改拥有者和所属组，改为atguigu
   chgrp atguigu software
   chown atguigu software
   chown atguigu:atguigu software/

#####  9: 修改hosts文件

   windows要改，Linux也要改
   192.168.254.101 hadoop101
   192.168.254.102 hadoop102
   192.168.254.103 hadoop103
   192.168.254.104 hadoop104
   修改Windows的配置文件，使用xshall连接Linux的时候，直接写机器名
   修改Linux的配置文件，做服务器集群（Linux系统之间找到对方）

   从连接Linux，使用atguigu登录

#####  10：克隆，从101机器克隆三台机器，102,103,104

   102,103,104  IP地址修改，修改机器名
------------Hadoop101主机，以上的配置，模版机-------

### ------------Hadoop102主机，搭建hadoop环境 单机-------

 1：软件包jar包，传递到102机器的 /opt/software目录下
 2：解压缩jdk.tar.gz,解压到 /opt/module下
    tar -xvf jdk-8u212-linux-x64.tar.gz -C /opt/module/
 3：配置JDK的环境变量 JAVA_HOME
    修改配置文件：/etc/profile，最好别改，配置环境变量自定义shell脚本
    脚本文件放到目录 profile.d下
    创建文	件 my_env.sh

```properties
    JAVA_HOME=/opt/module/jdk1.8.0_212
     export JAVA_HOME
     PATH=$PATH:$JAVA_HOME/bin
```

​     **重写加载配置文件：Linux系统命令 source 从新加载 /etc/profile**


   重写加载配置文件：Linux系统命令 source 从新加载 /etc/profile

 4：解压缩hadoop.tar.gz 解压到 /opt/module下
    
```properties
HADOOP_HOME=/opt/module/hadoop-3.1.3
export HADOOP_HOME
PATH=$PATH:$HADOOP_HOME/bin
PATH=$PATH:$HADOOP_HOME/sbin
```

  5: 测试hadoop能否正常工作
     hadoop解压目录下，创建文件夹input，保存了文件word.txt
     /opt/module/hadoop-3.1.3/share/hadoop/mapreduce
       目录下有jar包hadoop-mapreduce-examples-3.1.3.jar 运行jar包，统计word.txt文件中词出现次数
命令：hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount input output