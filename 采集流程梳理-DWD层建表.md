##### 笔记

- 数仓上线首日 ==> 含有历史数据 ===>  放到对应的分区，【使用动态分区】
  - 手动执行首日脚本
- 数仓执行每日 ==> 当天数据
  - 自动化调度每日的文件（任务调度器）

### 采集项目数据流转脚本简单总结

#### 行为数据模拟（日志数据）

1. `lg.sh`: 脚本会在 102， 103 两个节点上生成模拟日志文件，为 JSON 格式，
   1. 生成的日志文件在 `/opt/module/applog`  目录下
   2.  `application.yml` ： 模拟业务中日志数据的生成日期【可以根据需求生成对应日期的用户行为日志】，日志生成条数，等
   3. `path.json`，该文件用来配置访问路径， 日志中具体的点击数据的内容，比如在哪个页面跳转，用户的具体行为
   4. `logback.xml` 配置文件， 可配置日志生成路径，修改内容如下，生成的日志文件具体在节点的那个目录下。

#### 行为数据（日志数据）的采集-到kafka

2. 日志数据采集到Kafka， `f1.sh`
   1. 配置Flume的配置文件 `vim job/file_to_kafka.conf `
      1. 使用 `FILEDIR` 与  `KafkaChannel`
   2. 编写 `Flume` 拦截器，部署
      1. 代码对数据进行过滤，将空数据过滤掉
   3. `Flume` 的启动命令： `bin/flume-ng agent -n a1 -c conf/ -f job/file_to_kafka.conf -Dflume.root.logger=info,console`
   4. `Kafka`的启动命令： `bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic topic_log`
   5. Flume 与 Kafka 消费者都启动之后再启动日志模拟脚本 `lg.sh`
   6. 编写日志采集启停脚本 `f1.sh`
      1. 使用 `f1.sh start / stop`

#### 业务数据模拟

1. 创建 `gmall ` 数据库，执行 `gmall.sql`脚本生成业务数据表
2. 在hadoop102的/opt/module/目录下创建db_log文件夹
   1. 修改相关配置 `mkdir db_log/`
      1. 修改日期以及是否重置参数，生成五天的业务数据
      2. 生成业务数据 `java -jar gmall2020-mock-db-2021-11-14.jar`

#### 业务数据采集-到kafka

- 使用Maxwell将数据采集到Kafka

##### Maxwell简单使用

1. 原理： 将自己伪装成Mysql的Slaver 读取Mysql 的Binlog日志，将数据传输到Kafka
2. 修改Mysql数据库配置，开启Binlog功能 `sudo vim /etc/my.cnf`
   1. 启动Binlog功能
   2. 指定Binlog文件名【存储Binlog代码的文件的名字】
   3. 选择存储类型
   4. 指定具体哪个数据库开启Binlog
   5. 配置完成之后重启Mysql `sudo systemctl restart mysqld`
3. 创建Maxwell 需要的数据库和用户
   1. 创建数据库，调整密码级别， 赋予必要的权限
   2. 配置Maxwell `cd /opt/module/maxwell` --> `cp config.properties.example config.properties`
      1. 配置数据的发送地址为 Kafka
      2. 指定集群地址， 指定目标topic
      3. 配置Mysql相关， 指定Maxwell去监控那个数据库
   3. Maxwell的启动停止指令
      1. 启动 `/opt/module/maxwell/bin/maxwell --config /opt/module/maxwell/config.properties --daemon`
      2. 停止 `ps -ef | grep maxwell | grep -v grep | grep maxwell | awk '{print $2}' | xargs kill -9`
   4. 编辑启停脚本 `vim mxw.sh`  --> `mw=xw.sh start / stop / restart`
   5. 增量同步：直接启动 `mxw.sh` 就行
   6. 全量同步：使用 Bootstrap `/opt/module/maxwell/bin/maxwell-bootstrap --database gmall --table user_info --config /opt/module/maxwell/config.properties`

##### 采集通道的Maxwell 的配置

1. 修改配置文件 `vim /opt/module/maxwell/config.properties`
   1. 指定Maxwell 作为Kafka的生产者， 
   2. 指定Kafka的Server 与对应的topic
   3. 指定Maxwell监控的Mysql连接 【Jdbc】， 指定数据库

##### 日志数据（消费数据）同步

1. 使用Flume 直接将Kafka中的数据同步到HDFS，目录中包含一层日期 
2. 使用Flume的三个组件  `KafkaSource、FileChannel、HDFSSink`
3. 日志消费Flume配置
   1. 修改配置文件 `vim job/kafka_to_hdfs_log.conf `
   2. 配置三个组件， 在配置文件中指定HDFS中的文件存储目录
4. 数据飘逸问题解决，编写拦截器
   1. 将Header中的时间戳文件替换为日志生成的时间戳
5. 脚本启停 `f2.sh start / stop`

##### 业务数据同步【DataX】

- 在 104 节点

1. 解压部署时候创建配置文件
   1. `vim /opt/module/datax/job/base_province.json`
   2. 在文件中指定 Reader 与 Writer， 从MySQL读数据到HDFS
   3. DataX执行命令： ` python bin/datax.py job/base_province.json`
   4. 传参命令： `python bin/datax.py -p"-Ddt=2020-06-14" job/base_province.json`
2. 需要为每一张表创建DataX的配置文件
3. 编写DataX中对于每张数据表的配置文件脚本 `vim ~/bin/gen_import_config.py `
   1. 文件中指定了每张表的配置文件的路径，等信息
4. 批量生成多张表的脚本 `vim ~/bin/gen_import_config.sh`
   1. 其中需要调用 Python 脚本，传入参数，生成Datax使用的配置文件
   2. 使用，直接执行 `gen_import_config.sh`， 会将配置文件直接生成到 `/opt/module/datax/job/import/` 文件夹下
5. 全量表数据同步脚本 `vim ~/bin/mysql_to_hdfs_full.sh `
   1. 使用之前步骤生成的DataX的配置文件，启动DataX，将数据同步
   2. 传参执行： `mysql_to_hdfs_full.sh all 2020-06-14`
6. 增量表数据同步：使用Flume ， `f3.sh`
   1. 创建Kafka到HDFS的配置文件 `vim job/kafka_to_hdfs_db.conf`
      1. 三个组件选型 `Flume需要将Kafka中topic_db主题的数据传输到HDFS，故其需选用KafkaSource以及HDFSSink，Channel选用FileChannel。`
   2. 编写拦截器， 将时间戳做一个转换，取消掉原来的Header头
   3. 编写启停脚本 `f3.sh`：
7. 增量表首日全量同步
   1. `vim mysql_to_kafka_inc_init.sh`：使用Maxwell的Bootstrap功能
      1. 使用： `mysql_to_kafka_inc_init.sh all `

##### 集群启停脚本

- `cluster.sh start / stop`

##### DWD层规划：交易域加购事务事实表

- `parent_code = '24'`： 商品的来源类型（通过推广或者广告点击加入的购物车）
- 每日新增数据的时候需要判断是否为加入购物车，判断 update 有数据，并且数量在增加

```sql
-- DWD 事实表
    -- 1. 交易域加购事务事实表
        -- 加购 ： 用户在某一个时间点向购物车中增加了新的商品（全新，数量的新增）

-- 建表语句
    -- 表名
    -- 分区规划 ： 每日分区保存 （首日动态分区，每日静态分区）
DROP TABLE IF EXISTS dwd_trade_cart_add_inc;
CREATE EXTERNAL TABLE dwd_trade_cart_add_inc
(
    `id`               STRING COMMENT '编号',
    `user_id`          STRING COMMENT '用户id',
    `sku_id`           STRING COMMENT '商品id',
    `date_id`          STRING COMMENT '时间id',
    `create_time`      STRING COMMENT '加购时间',
    `source_id`        STRING COMMENT '来源类型ID',
    `source_type_code` STRING COMMENT '来源类型编码',
    `source_type_name` STRING COMMENT '来源类型名称',
    `sku_num`          BIGINT COMMENT '加购物车件数'
) COMMENT '交易域加购物车事务事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dwd/dwd_trade_cart_add_inc/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

-- 加载数据
    -- 首日加载
        -- maxwell - 首日 - type -> bootstrap-start, insert, finish
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_trade_cart_add_inc partition (dt)
select
     id
    ,user_id
    ,sku_id
    ,date_format(create_time, 'yyyy-MM-dd') date_id
    ,create_time
    ,source_id
    ,source_type
    ,dic.dic_name source_type_name
    ,sku_num
    ,date_format(create_time, 'yyyy-MM-dd')
from (
     select
        data.id
        ,data.user_id
        ,data.sku_id
        ,data.create_time
        ,data.source_id
        ,data.source_type
        ,data.sku_num
    from ods_cart_info_inc
    where dt='2020-06-14'
    and type = 'bootstrap-insert'
) cart
left join (
    select
        dic_code,
        dic_name
    from ods_base_dic_full
    where dt='2020-06-14' and parent_code = '24'
) dic on cart.source_type = dic.dic_code;

show partitions dwd_trade_cart_add_inc;

-- 每日数据加载
    -- 2020-06-15
    -- 判断哪些数据是加购
        -- 06-15 新增商品加入到购物车 => insert into => create time => 加购时间
        -- 06-15 修改购物车中商品的数量 => update (sku_num)
            -- 加购数据需要保存 new.sku_num > old.sku_num  (增量表 - update - old)
            -- 减购数据不考虑   new.sku_num < old.sku_num  (增量表 - update - old)
    -- 数据字段类型：结构体和map的区别
        -- map : 字段数量不确定
        -- 结构体 : 不同的字段属性类型不一致

    -- map_keys : 获取map类型字段的所有key，返回的就是key的数组
    -- array_contains: 判断集合中是否包含指定数据
insert overwrite table dwd_trade_cart_add_inc partition (dt='2020-06-15')
select
     id
    ,user_id
    ,sku_id
    ,date_format(from_utc_timestamp(ts*1000, 'GMT+8'), 'yyyy-MM-dd')
    ,date_format(from_utc_timestamp(ts*1000, 'GMT+8'), 'yyyy-MM-dd HH:mm:ss')
    ,source_id
    ,source_type
    ,dic.dic_name source_type_name
    ,sku_num
from (
     select
        data.id
        ,data.user_id
        ,data.sku_id
        ,ts
        ,data.source_id
        ,data.source_type
        ,if( type='insert', data.sku_num, data.sku_num - old['sku_num']) sku_num
    from ods_cart_info_inc
    where dt = '2020-06-15'
    and (
        type = 'insert'
         or
        (type = 'update' and array_contains(map_keys(old), 'sku_num') and data.sku_num > cast(old['sku_num'] as bigint))
    )
) cart
left join (
    select
        dic_code,
        dic_name
    from ods_base_dic_full
    where dt='2020-06-15' and parent_code = '24'
) dic on cart.source_type = dic.dic_code;


// 时间函数练习
// from_unixtime : 获取0时区的时间数据，传递参数为long
// from_utc_timestamp : 获取指定时区的时间数据，传递参数为long,GMT+8表示东8区
 select
    ts,
    from_unixtime(ts, 'yyyy-MM-dd'),
    from_unixtime(ts, 'yyyy-MM-dd HH:mm:ss'),
    date_format(from_utc_timestamp(ts*1000, 'GMT+8'), 'yyyy-MM-dd'),
    date_format(from_utc_timestamp(ts*1000, 'GMT+8'), 'yyyy-MM-dd HH:mm:ss')
from ods_cart_info_inc
where dt = '2020-06-14'
```

##### 交易域下单事务事实表

- 几张表连接成功
- 使用leftjoin 将需要的数据字段查找出来

```sql
-- 交易域下单事务事实表
    -- 建表
        -- 表名
        -- 粒度
        -- 维度
        -- 事实（度量值）

-- dwd_trade_order_add_inc
DROP TABLE IF EXISTS dwd_trade_order_detail_inc;
CREATE EXTERNAL TABLE dwd_trade_order_detail_inc
(
    `id`                    STRING COMMENT '编号',
    `order_id`              STRING COMMENT '订单id',
    `user_id`               STRING COMMENT '用户id',
    `sku_id`                STRING COMMENT '商品id',
    `province_id`           STRING COMMENT '省份id',
    `activity_id`           STRING COMMENT '参与活动规则id',
    `activity_rule_id`      STRING COMMENT '参与活动规则id',
    `coupon_id`             STRING COMMENT '使用优惠券id',
    `date_id`               STRING COMMENT '下单日期id',
    `create_time`           STRING COMMENT '下单时间',
    `source_id`             STRING COMMENT '来源编号',
    `source_type_code`      STRING COMMENT '来源类型编码',
    `source_type_name`      STRING COMMENT '来源类型名称',
    `sku_num`               BIGINT COMMENT '商品数量',
    `split_original_amount` DECIMAL(16, 2) COMMENT '原始价格',
    `split_activity_amount` DECIMAL(16, 2) COMMENT '活动优惠分摊',
    `split_coupon_amount`   DECIMAL(16, 2) COMMENT '优惠券优惠分摊',
    `split_total_amount`    DECIMAL(16, 2) COMMENT '最终价格分摊'
) COMMENT '交易域下单明细事务事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dwd/dwd_trade_order_detail_inc/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

-- 建表语句
    -- 首日
insert overwrite table dwd_trade_order_detail_inc partition (dt)
select
    *,
    date_format(create_time, 'yyyy-MM-dd')
from (
     select
        data.id,
        data.order_id,
        data.source_type,
        data.create_time
    from ods_order_detail_inc
    where dt='2020-06-14'
    and type='bootstrap-insert'
) od
left join (
    select
        data.id
    from ods_order_info_inc
    where dt='2020-06-14'
    and type='bootstrap-insert'
) oi on od.order_id = oi.id
left join (
    select
        data.order_detail_id
    from ods_order_detail_coupon_inc
    where dt='2020-06-14'
    and type='bootstrap-insert'
) odc on odc.order_detail_id = od.id
left join (
    select
        data.order_detail_id
    from ods_order_detail_activity_inc
    where dt='2020-06-14'
    and type='bootstrap-insert'
) oda on oda.order_detail_id = od.id
left join (
    select
        dic_code,
        dic_name
    from ods_base_dic_full
    where dt='2020-06-14' and parent_code = '24'
) dic on dic.dic_code =  od.source_type;

-- 每日
    -- 2020-06-15
insert overwrite table dwd_trade_order_detail_inc partition (dt='2020-06-15')
select
    *
from (
     select
        data.id,
        data.order_id,
        data.source_type,
        data.create_time
    from ods_order_detail_inc
    where dt='2020-06-15'
    and type='insert'
) od
left join (
    select
        data.id
    from ods_order_info_inc
    where dt='2020-06-15'
    and type='insert'
) oi on od.order_id = oi.id
left join (
    select
        data.order_detail_id
    from ods_order_detail_coupon_inc
    where dt='2020-06-15'
    and type='insert'
) odc on odc.order_detail_id = od.id
left join (
    select
        data.order_detail_id
    from ods_order_detail_activity_inc
    where dt='2020-06-15'
    and type='insert'
) oda on oda.order_detail_id = od.id
left join (
    select
        dic_code,
        dic_name
    from ods_base_dic_full
    where dt='2020-06-15' and parent_code = '24'
) dic on dic.dic_code =  od.source_type
```



