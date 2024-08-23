flume

1.上传压缩包到/opt

解压压缩包

tar -xzvf apache-flume-1.9.0-bin.tar.gz -C /opt

重命名

mv apache-flume-1.9.0-bin.tar.gz flume

2.配置环境变量

vim /etc/profile.d/hadoop-eco.sh

FLUME_HOME=/opt/flume PATH=\$PATH:\$FLUME_HOME/bin

3.拷贝flume-env.sh.template 重命名为 flume-env.sh

4.编辑flume-env.sh

取消注释，编辑

export JAVA_HOME=/opt/jdk

5.替换flume的lib目录的低版本java.jar包
<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume1.png"
style="width:4.16236in;height:1.19667in" />
rm -rf guava-11.0.2.jar

cp /opt/hadoop/share/hadoop/common/lib/guava-27.0-jre.jar
/opt/flume/lib

二、监听网络端口发送信息(在slave01上借助44444发送内容给master，master的flume
负责接收内容收集显示到控制台)
1.在slave01的控制台上输入内容，通过网络端口44444发送给master(安装flume的机器)
接收，flume则负责监听端口将内容（模拟日志）进行收集显示到控制台，新建fllume配置文件

方式一：以日志的形式存储数据

Vim conf/console.conf

定义当前 agent(数据收集器) 的名字为 a1，可以随意

定义 agent 中的 sources(数据来源) 名字为 r1 a1.sources = r1

定义 agent 中的 sinks(数据存储) 组件叫做 k1 a1.sink=k1

定义 agent 中的 channels(数据缓存) 组件叫 c1 a1.channels=c1

分别详细设置 \#source类型为网络字节流(数据来源的类型为字节流)
a1.sources.r1.type=netcat

netcat代表网络端口

监听的主机名 a1.sources.r1.bind=master

监听接收数据的端口号 a1.sources.r1.port=44444

设置类型为 logger 日志方式（发送的内容的存储方式为日志）
a1.sinks.k1.type=logger

内存进行数据缓存 a1.channels.c1.type=memory

定义容量 a1.channels.c1.capacity=1000

最大事务容量 a1.channels.c1.transactionCapacity=100

建立关联

a1.sources.r1.channels=c1 a1.sinks.k1.channel=c1

2.在master上启动flume监听：(在bin下面启动)

./flume-ng agent -c conf -f ../conf/console.conf -name a1 -Dflume.root.logger=INFO,console

Dflume.root.logger=INFO.console 输出级别为INFO，输出到控制台

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume2.png"
style="width:5.00069in;height:2.45069in" />

已经成功，强制退出即可

3.在slave01上进行网络信息发送

nc master 44444

然后回车，就可以输入要发送的内容

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume3.png" style="width:5.00069in;height:1.52in" />

输入要发送的内容

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume4.png" style="width:5.00069in;height:1.18in" />

 在master的控制台中可以看到接收的内容

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume5.png"> style="width:5.00069in;height:1.43194in" />
完成之后强制退出即可
方式二：以文件的形式存储
 检测网络端口是否被占用
netstat -ano \| findstr :\[端口号\]
需要改一下端口号，防止被占用，改一下44444
在opt下面创建temp用于接收发送的内容
设置类型为 logger 日志方式（发送的内容的存储方式为文件）

a1.sinks.k1.type=file_roll a1.sinks.k1.sink.directory=/opt/temp
a1.sinks.k1.sink.rollinterval=0

在master上启动flume监听：(在bin下面启动)

./flume-ng agent -c conf -f ../conf/console.conf -name a1 -Dflume.root.logger=INFO,console

在slave01上进行网络信息发送 nc slave01 44445

回车之后输入发送的内容
发现内容没有打印到控制台

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume6.png"> style="width:5.00069in;height:1.63694in" />
输入的内容被保存在文件中了

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume7.png"> style="width:5.00069in;height:1.26167in" />

三、定义监听文件变化收集
1.监听master机器上某个文件的变化，将变化的内容打印到控制台，新建flume配置文件
如下：
vim conf/ckfile.conf
Vim conf/flumetest.txt aaaa
定义当前 agent(数据收集器) 的名字为 a2，可以随意
定义 agent 中的 sources(数据来源) 名字为 r1 a2.sources = r1
定义 agent 中的 sinks(数据存储) 组件叫做 k1 a2.sink=k1
定义 agent 中的 channels(数据缓存) 组件叫 c1 a2.channels=c1
分别详细设置 \#source类型为网络字节流(数据来源的类型为字节流)
a2.source.r1.type=exec
a2.source.r1.command=tail -F /opt/temp/flumetest.txt
command = tail 收集文件最后的变化
设置类型为 logger 日志方式（发送的内容的存储方式）
a2.sinks.k1.type=file_roll a2.sinks.k1.sink.directory=/opt/temp
a2.sinks.k1.sink.rollinterval=0
如果flumetest.txt中的文件发生变化，将会收集到temp中
内存进行数据缓存 a1.channels.c1.type=memory
定义容量 a1.channels.c1.capacity=1000
最大事务容量 a1.channels.c1.transactionCapacity=100
建立关联
a1.sources.r1.channels=c1 a1.sinks.k1.channel=c1
2.在master上启动flume监听：(在bin下面启动)
./flume-ng agent -c conf -f ../conf/file.conf -name a2 -Dflume.root.logger=INFO,console
3.echo "bbbb" \>\>/opt/temp/flumetest.txt



4.查看变化的内容是否被检测
变化一次，将会生成一个新的文件，变化的内容会打印在里面
<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume8.png"> style="width:5.00069in;height:1.98194in" />
将监听的内容上传至HDFS
收集配置文件：
vim conf/hdfs.conf
定义当前 agent(数据收集器) 的名字为 a3，可以随意
定义 agent 中的 sources(数据来源) 名字为 r1 a3.sources = r1
定义 agent 中的 sinks(数据存储) 组件叫做 k1 a3.sink=k1
定义 agent 中的 channels(数据缓存) 组件叫 c1 a3.channels=c1
分别详细设置
source类型为网络字节流(数据来源的类型为字节流)
a3.sources.r1.type=exec
a3.sources.r1.command=tail -F /opt/temp/flumetest.txt
设置类型为hdfs（发送的内容的存储方式）
a3.sinks.k1.type=hdfs
a3.sinks.k1.hdfs.path=hdfs://master:50070/input/flume/%y/%m
a3.sinks.k1.hdfs.filePrefix=events-

a3.sinks.k1.hdfs.round=true a3.sinks.k1.hdfs.roundValue=40
a3.sinks.k1.hdfs.roundUnit=second
a3.sinks.k1.hdfs.useLocalTimeStamp=true
内存进行数据缓存
a3.channels.c1.type=memory
定义容量 a3.channels.c1.capacity=1000
最大事务容量 a3.channels.c1.transactionCapacity=100
建立关联
a3.sources.r1.channels=c1 a3.sinks.k1.channel=c1
2.在master上启动flume监听：(在bin下面启动)
./flume-ng agent -c conf -f ../conf/hdfs.conf -name a3 -Dflume.root.logger=INFO,console
3.echo "hello word" \>\> flumetest.txt
然后去HDFS页面查看
 <img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume9.png"> style="width:5.00069in;height:3.06403in" />
从虚拟机中查看文件
hdfs dfs -cat /input/flume/21/08/events-.1630233437362
<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume10.png"> style="width:5.00069in;height:0.63833in" />
出现乱码，在conf/hdfs.conf文件中加入
a3.sinks.k1.hdfs.fileType=DataStream
<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/flume/flume11.png"
style="width:5.00069in;height:0.71367in" />
再次启动
./flume-ng agent -c conf -f ../conf/hdfs.conf -name a3 -Dflume.root.logger=INFO,console
然后HDFS中会再次生成一个新文件，再次查看即可
