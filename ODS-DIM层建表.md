##### Kafka中的零拷贝

- 毫秒使用13位数字，秒使用10位数字。
- 物理删除：直接在硬盘上删除
- 逻辑删除：设置标志位
- 列式编辑， 按住 ALT 鼠标点击滑动

##### 数据仓库ODS

![1659439134776](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659439134776.png)

##### 数据仓库DIM

![1659439185331](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659439185331.png)

##### 分区分桶-SQL基本函数回顾

```sql
-- 分区表
    -- 所谓的分区，其实就是表的虚拟列
    -- 其实执行后，会在HDFS上增加一个文件路径，也就是说分区列不是数据的一部分，是路径的一部分。
create table my_partition_table(
    id int
)
partitioned by ( dt string )
location '/warehouse/test/my_partition_table';

select * from my_partition_table;

// JOB => Spark => 申请资源 => 启动进程
// 静态分区
insert into my_partition_table partition (dt='2022') values (1);
// 动态分区
set hive.exec.dynamic.partition.mode=nonstrict;
insert into my_partition_table partition (dt) values (1, "2021");

select * from my_partition_table;


-- 建表语句
-- 外部表
-- 内部表
create table my_table(
    id int,
    name string
);
create external table my_external_table (
    id int,
    name string
);

-- 创建JSON表
CREATE TABLE my_json_table(id string, name string, age bigint)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE;

-- 查询数据
select * from my_json_table;

-- 字段得类型：array, map, struct（class）
select array(1,2,3);
create table my_array_table(
    ids array<int>
);
insert into my_array_table select array(1,2,3);
// 查询数据
select ids[1] from my_array_table;
select `map`("a", "b", "c", "d");
select `map`("a", 1, "c", 2);
select struct("a", "b", 3);
select named_struct("action_id", "favor_add", "ts", 1585744376605);

--select `array`(1,2,3) from sku_attr_value where sku_id = '1'

-- array操作是将一个行数据的多个列封装为数组返回一个列
-- select array("1", "2", "3") from xxxx
-- group by (聚合操作) (聚合函数)(UDAF)
```

##### 日志建表查询

```sql
-- ODS 日志表建表语句
    -- 1. 外部表 & 内部表
    -- 2. 字段特殊类型
        -- 2.1 array
        -- 2.2 map (K - V)
        -- 2.3 struct ( K - V )
    -- 3. 分区表 partition
    -- 4. JSON表 (数据为JSON格式，表还是文本)
    -- 5. 压缩格式 gzip
    -- 6. 位置 location
-- 问题点：
    -- 1. HiveServer2 日志错误
    -- 2. Hive堆内存
    -- 3. DataGrip 字段和字段解释comment 乱码
DROP TABLE IF EXISTS ods_log_inc;
CREATE EXTERNAL TABLE ods_log_inc
(
    `common`   STRUCT<ar :STRING,ba :STRING,ch :STRING,is_new :STRING,md :STRING,mid :STRING,os :STRING,uid :STRING,vc
                      :STRING> COMMENT '公共信息',
    `page`     STRUCT<during_time :STRING,item :STRING,item_type :STRING,last_page_id :STRING,page_id
                      :STRING,source_type :STRING> COMMENT '页面信息',
    `actions`  ARRAY<STRUCT<action_id:STRING,item:STRING,item_type:STRING,ts:BIGINT>> COMMENT '动作信息',
    `displays` ARRAY<STRUCT<display_type :STRING,item :STRING,item_type :STRING,`order` :STRING,pos_id
                            :STRING>> COMMENT '曝光信息',
    `start`    STRUCT<entry :STRING,loading_time :BIGINT,open_ad_id :BIGINT,open_ad_ms :BIGINT,open_ad_skip_ms
                      :BIGINT> COMMENT '启动信息',
    `err`      STRUCT<error_code:BIGINT,msg:STRING> COMMENT '错误信息',
    `ts`       BIGINT  COMMENT '时间戳'
) COMMENT '活动信息表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
    LOCATION '/warehouse/gmall/ods/ods_log_inc/';

-- 数据装载
load data inpath '/origin_data/gmall/log/gmall_log/2020-06-14' into table ods_log_inc partition(dt='2020-06-14');
load data inpath '/origin_data/gmall/log/gmall_log/2020-06-15' into table ods_log_inc partition(dt='2020-06-15');
load data inpath '/origin_data/gmall/log/gmall_log/2020-06-16' into table ods_log_inc partition(dt='2020-06-16');

-- 查询数据
select common.ar from ods_log_inc;
```

##### ODS表的同步

```sql
-- ODS业务表（MySQL） - 全量表
    -- 全量表（15） & 增量表（13）
    -- 全量表
        -- MySQL表的全部数据（维度分析）
        -- 存储策略：一天一份全量数据（分区表）
        -- 存储格式：TSV, \t
        -- DataX -> select
    -- 增量表
        -- MySQL业务表每日新增和变化的数据（事实统计）
        -- 存储策略：一天一份增量数据（分区表）
        -- 存储格式：JSON
        -- Maxwell -> binlog -> JSON
            -- 首日全量 -> bootstrap
            -- 每日增量 -> insert, update, delete

-- 全量表 - 建表语句
-- 增量表

DROP TABLE IF EXISTS ods_activity_info_full;
CREATE EXTERNAL TABLE ods_activity_info_full
(
    `id`            STRING COMMENT '活动id',
    `activity_name` STRING COMMENT '活动名称',
    `activity_type` STRING COMMENT '活动类型',
    `activity_desc` STRING COMMENT '活动描述',
    `start_time`    STRING COMMENT '开始时间',
    `end_time`      STRING COMMENT '结束时间',
    `create_time`   STRING COMMENT '创建时间'
) COMMENT '活动信息表'
    PARTITIONED BY (`dt` STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    NULL DEFINED AS ''
    LOCATION '/warehouse/gmall/ods/ods_activity_info_full/';
```

##### 维度表建表语句

```sql
with
sku as
(
    select
        id,
        price,
        sku_name,
        sku_desc,
        weight,
        is_sale,
        spu_id,
        category3_id,
        tm_id,
        create_time
    from ods_sku_info_full
    where dt='2020-06-14'
),
spu as
(
    select
        id,
        spu_name
    from ods_spu_info_full
    where dt='2020-06-14'
),
c3 as
(
    select
        id,
        name,
        category2_id
    from ods_base_category3_full
    where dt='2020-06-14'
),
c2 as
(
    select
        id,
        name,
        category1_id
    from ods_base_category2_full
    where dt='2020-06-14'
),
c1 as
(
    select
        id,
        name
    from ods_base_category1_full
    where dt='2020-06-14'
),
tm as
(
    select
        id,
        tm_name
    from ods_base_trademark_full
    where dt='2020-06-14'
),
attr as
(
    select
        sku_id,
        collect_set(named_struct('attr_id',attr_id,'value_id',value_id,'attr_name',attr_name,'value_name',value_name)) attrs
    from ods_sku_attr_value_full
    where dt='2020-06-14'
    group by sku_id
),
sale_attr as
(
    select
        sku_id,
        collect_set(named_struct('sale_attr_id',sale_attr_id,'sale_attr_value_id',sale_attr_value_id,'sale_attr_name',sale_attr_name,'sale_attr_value_name',sale_attr_value_name)) sale_attrs
    from ods_sku_sale_attr_value_full
    where dt='2020-06-14'
    group by sku_id
)
insert overwrite table dim_sku_full partition(dt='2020-06-14')
select
    sku.id,
    sku.price,
    sku.sku_name,
    sku.sku_desc,
    sku.weight,
    sku.is_sale,
    sku.spu_id,
    spu.spu_name,
    sku.category3_id,
    c3.name,
    c3.category2_id,
    c2.name,
    c2.category1_id,
    c1.name,
    sku.tm_id,
    tm.tm_name,
    attr.attrs,
    sale_attr.sale_attrs,
    sku.create_time
from sku
left join spu on sku.spu_id=spu.id
left join c3 on sku.category3_id=c3.id
left join c2 on c3.category2_id=c2.id
left join c1 on c2.category1_id=c1.id
left join tm on sku.tm_id=tm.id
left join attr on sku.id=attr.sku_id
left join sale_attr on sku.id=sale_attr.sku_id;
```

##### 日志表数据装载

![1659440065471](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659440065471.png)

##### 业务表数据装载

```shell
[atguigu@hadoop102 bin]$ vim hdfs_to_ods_db.sh 
```

```shell
#!/bin/bash

APP=gmall220411

if [ -n "$2" ] ;then
   do_date=$2
else 
   do_date=`date -d '-1 day' +%F`
fi

load_data(){
    sql=""
    for i in $*; do
        #判断路径是否存在
        hadoop fs -test -e /origin_data/$APP/db/${i:4}/$do_date
        #路径存在方可装载数据
        if [[ $? = 0 ]]; then
            sql=$sql"load data inpath '/origin_data/$APP/db/${i:4}/$do_date' OVERWRITE into table ${APP}.$i partition(dt='$do_date');"
        fi
    done
    hive -e "$sql"
}

case $1 in
    "ods_activity_info_full")
        load_data "ods_activity_info_full"
    ;;
    "ods_activity_rule_full")
        load_data "ods_activity_rule_full"
    ;;
    "ods_base_category1_full")
        load_data "ods_base_category1_full"
    ;;
    "ods_base_category2_full")
        load_data "ods_base_category2_full"
    ;;
    "ods_base_category3_full")
        load_data "ods_base_category3_full"
    ;;
    "ods_base_dic_full")
        load_data "ods_base_dic_full"
    ;;
    "ods_base_province_full")
        load_data "ods_base_province_full"
    ;;
    "ods_base_region_full")
        load_data "ods_base_region_full"
    ;;
    "ods_base_trademark_full")
        load_data "ods_base_trademark_full"
    ;;
    "ods_cart_info_full")
        load_data "ods_cart_info_full"
    ;;
    "ods_coupon_info_full")
        load_data "ods_coupon_info_full"
    ;;
    "ods_sku_attr_value_full")
        load_data "ods_sku_attr_value_full"
    ;;
    "ods_sku_info_full")
        load_data "ods_sku_info_full"
    ;;
    "ods_sku_sale_attr_value_full")
        load_data "ods_sku_sale_attr_value_full"
    ;;
    "ods_spu_info_full")
        load_data "ods_spu_info_full"
    ;;

    "ods_cart_info_inc")
        load_data "ods_cart_info_inc"
    ;;
    "ods_comment_info_inc")
        load_data "ods_comment_info_inc"
    ;;
    "ods_coupon_use_inc")
        load_data "ods_coupon_use_inc"
    ;;
    "ods_favor_info_inc")
        load_data "ods_favor_info_inc"
    ;;
    "ods_order_detail_inc")
        load_data "ods_order_detail_inc"
    ;;
    "ods_order_detail_activity_inc")
        load_data "ods_order_detail_activity_inc"
    ;;
    "ods_order_detail_coupon_inc")
        load_data "ods_order_detail_coupon_inc"
    ;;
    "ods_order_info_inc")
        load_data "ods_order_info_inc"
    ;;
    "ods_order_refund_info_inc")
        load_data "ods_order_refund_info_inc"
    ;;
    "ods_order_status_log_inc")
        load_data "ods_order_status_log_inc"
    ;;
    "ods_payment_info_inc")
        load_data "ods_payment_info_inc"
    ;;
    "ods_refund_payment_inc")
        load_data "ods_refund_payment_inc"
    ;;
    "ods_user_info_inc")
        load_data "ods_user_info_inc"
    ;;
    "all")
        load_data "ods_activity_info_full" "ods_activity_rule_full" "ods_base_category1_full" "ods_base_category2_full" "ods_base_category3_full" "ods_base_dic_full" "ods_base_province_full" "ods_base_region_full" "ods_base_trademark_full" "ods_cart_info_full" "ods_coupon_info_full" "ods_sku_attr_value_full" "ods_sku_info_full" "ods_sku_sale_attr_value_full" "ods_spu_info_full" "ods_cart_info_inc" "ods_comment_info_inc" "ods_coupon_use_inc" "ods_favor_info_inc" "ods_order_detail_inc" "ods_order_detail_activity_inc" "ods_order_detail_coupon_inc" "ods_order_info_inc" "ods_order_refund_info_inc" "ods_order_status_log_inc" "ods_payment_info_inc" "ods_refund_payment_inc" "ods_user_info_inc"
    ;;
esac
```

![1659440152992](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659440152992.png)

