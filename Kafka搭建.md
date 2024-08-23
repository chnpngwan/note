# Kafka

安装kafka（启动前需要先zookeeper） 

1.上传kafka_2.12-2.4.1.tgz到/opt/soft

2.解压 tar -xzvf kafka_2.12-2.4.1.tgz -C /opt

  重命名mv kafka_2.12-2.4.1.tgz kafka

3.配置环境变量vi /etc/profile.d/hadoop-eco.sh

export KAFKA_HOME=/opt/kafka

export PATH=\$PATH:\$KAFKA_HOME/bin

scp -r /etc/profile.d/hadoop-eco.sh slave01:/etc/profile.d/hadoop-eco.sh
hadoop-eco.sh

scp -r /etc/profile.d/hadoop-eco.sh slave02:/etc/profile.d/hadoop-eco.sh
hadoop-eco.sh

scp -r /etc/profile.d/hadoop-eco.sh slave03:/etc/profile.d/hadoop-eco.sh
hadoop-eco.sh

source /etc/profile.d/hadoop-eco.sh

4.在kafka下面创建目录

mkdir data mkdir logs

5.修改config 目录下面的server.properties 配置文件

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka1.png"
style="width:4.58403in;height:0.49333in" />

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka2.png" style="width:4.21069in;height:0.42in" />

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka3.png"
style="width:4.59069in;height:0.77033in" />

6.更改kafka停止脚本

Kafuka的bin目录下kafka-server-stop.sh有些问题，需要先修改一下官方提供的stop脚本，集群
中的每一台机器都要改（kafka里面的k是小写）

原：

PIDS=\$(ps ax \| grep -i 'kafka\\Kafka' \| grep java \| grep -v grep \|
awk '{print \$1}')

修改为：

PIDS=\$(ps ax \| grep -i 'kafka' \| grep java \| grep -v grep \| awk
'{print \$1}')

7.在bin下面创建一键启动脚本kk_start_all.sh内容如下：

for host in master slave01 slave02 do

ssh \$host "source
/etc/profile.d/hadoop-eco.sh;/opt/kafka/bin/kafka-server-start.sh
-daemon /opt/kafka/config/server.properties"

echo "\$host kafka 服务正在启动….."

done

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka4.png"
style="width:7.62431in;height:1.05833in" />

7.在bin下面创建一键停止脚本kk_stop_all.sh内容如下：

for host in master slave01 slave02 do

ssh \$host "source /etc/profile;/opt/kafka/bin/kafka-server-stop.sh"
echo "\$host kafka 服务正在停止….."

done

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka5.png"
style="width:4.95236in;height:0.92833in" />

然后赋予两个文件权限

chmod 777 kk_start_all.sh 

chmod 777 kk_stop_all.sh

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka6.png"
style="width:3.22361in;height:0.947in" />

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka7.png"
style="width:2.64361in;height:1.07833in" />

8\.拷贝到从机上面

scp -r /opt/kafka slave01:/opt/kafka scp -r /opt/kafka
slave02:/opt/kafka scp -r /opt/kafka slave03:/opt/kafka

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka8.png" style="width:1.55194in;height:0.69in" />

9.修改01,02,03/opt/kafka/config
的server.properties 01的改为1,02的改为2

10.启动kafka（要先启动zookeeper） kk_start_all.sh

没有编写脚本:kafka-server-start.sh -daemon
/opt/kafka/config/server.properties

创建主题：(在bin下面用命令)

./kafka-topics.sh --create \\ \> --topic whj \\

\> --partitions 3 \\

\> --replication-factor 1 \\

\> --zookeeper master:2181,slave01:2181,slave02:2181

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka9.png"
style="width:2.97528in;height:1.03033in" />

解析：

\> --topic 主题名

\> --partitions 分区数

\> --replication-factor 副本数

\> --zookeeper master:2181,slave01:2181,slave02:2181
在zookeeper集群

查看主题(一行一行的输入) ./kafka-topics.sh --list \\

--zookeeper master:2181,slave01:2181,slave02:2181

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka10.png"
style="width:3.48069in;height:0.99333in" />

删除主题：

./kafka-topics.sh --zookeeper localhost:2181 --delete --topic plc

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka11.png"
style="width:3.21361in;height:0.93167in" />

需要server.properties 中设置 delete.topic.enable=true 否则只标记删除

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka12.png"
style="width:3.45403in;height:0.67167in" />

修改分区数：

kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic whj
--partitions 5

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka13.png"
style="width:3.36028in;height:1.73694in" />

注意：一般分区数不超过主机数量

查看分区的详细信息：

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka14.png" style="width:5.90069in;height:0.52in" />kafka-topics.sh
--bootstrap-server master:9092 --describe --topic whj topic-create

kafka主题与分区学习：
[<u>https://blog.csdn.net/zqf787351070/article/details/130291340</u>](https://blog.csdn.net/zqf787351070/article/details/130291340)

开启生产者（master）

kafka-console-producer.sh --broker-list master:9092 --topic whj

<img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka15.png"
style="width:4.15069in;height:0.532in" />

在另一台主机上创建消费者 ./kafka-console-consumer.sh\\

> --from-beginning --topic whj\\
>
> --bootstrap-server master:9092,slave01:9092,slave02:9092
>
> <img src="https://gitee.com/chnpngwng/typora-image/raw/master/assets/kafka/kafka16.png"
> style="width:5.05903in;height:1.53028in" />

kafka消费命令学习
[<u>http://www.mobiletrain.org/about/BBS/119918.html</u>](http://www.mobiletrain.org/about/BBS/119918.html)
