##### 建表过程分析

-- 1. dws层为什么要保存汇总数据？
    -- 汇总一些公用数据，
-- 2. dws层要存放哪些汇总数据；

-- 1）：指标体系，
    -- 看需求，分析需求的计算逻辑，需要用到哪张表，进行怎样的计算====> 找到每一个指标依赖的中间结果====> 公用的中间结果
    -- 指标体系：分析指标逻辑===找中间结果

-- 原子指标：对指标的聚合逻辑进行了定义：
    -- 订单总额，总件数，总次数，
    -- 不是非常明确，辅助定义原子指标的概念【业务过程， 度量值， 聚合逻辑】 ====》订单总额
-- 派生指标：
-- 衍生指标：一个或者多个派生指标

-- 一张汇总表中含有多个派生指标，【业务过程，统计周期，统计粒度相同】
-- 数据域：对数仓中的表进行分类操作，在分类中找表效率高，；
    -- 一般按照业务过程进行分类，；



-- 设计数据表的名
    -- dws_trade_tm_order_1d
        -- 表结构：品牌id，  下单次数， 下单人数， 按天分区--与dwd中的分区对应， ；

##### `dws_trade_tm_order_1d`：的设计

- 按照最近一日的日期分组之后直接聚合，
- 统计用户要去重，
- 左连接表拿到品牌的名字

```sql
drop table if exists dws_trade_tm_order_1d;
create external table dws_trade_tm_order_1d
(
    `tm_id`                     string comment '品牌id',
    `tm_name`                   string comment '品牌名称',
    `order_count`               string comment '下单次数',
    `order_user_count`          string comment '下单人数',
    `order_num`                 string comment '下单件数',
    `order_total_amount`        string comment '下单金额'

) comment '交易域品牌粒度订单最近1日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_tm_order_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');

-- 数据装载
insert overwrite table dws_trade_tm_order_1d partition (dt='2020-16-14')
select tm_id,
       tm_name,
       count(*),
       count(distinct user_id),
       sum(sku_num),
       sum(split_total_amount)
from
(
    select sku_id,
           user_id,
           sku_num,
           split_total_amount
    from dwd_trade_order_detail_inc
    where dt='2020-06-14'
) od
left join
(
    select id,
           tm_id,
           tm_name
    from dim_sku_full
    where dt='2020-06-14'
) sku
on od.sku_id = sku.id
group by tm_id, tm_name;
```

##### `dws_trade_tm_order_nd`：最近 n 天

- 最近30天包含最近7天
- 使用 `sum（if）`： 的组合函数 将包含在最近30天的数据取出来
  - 使用if将最近30天的统计数据做一个转换映射， 想要的映射为原始数据，不想要的映射为 0 在执行sum操作的时候会忽略

```sql
drop table if exists dws_trade_tm_order_nd;
create external table dws_trade_tm_order_nd
(
    `tm_id`                     string comment '品牌id',
    `tm_name`                   string comment '品牌名称',
    `order_count_7d`               string comment '最近7天下单次数',
    `order_user_count_7d`          string comment '最近7天下单人数',
    `order_num_7d`                 string comment '最近7天下单件数',
    `order_total_amount_7d`        string comment '最近7天下单金额',
    `order_count_30d`               string comment '最近30天下单次数',
    `order_user_count_30d`          string comment '最近30天下单人数',
    `order_num_30d`                 string comment '最近30天下单件数',
    `order_total_amount_30d`        string comment '最近30天下单金额'

) comment '交易域品牌粒度订单最近1日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_tm_order_nd'
    TBLPROPERTIES ('orc.compress' = 'snappy');

insert overwrite table dws_trade_tm_order_nd partition (dt='2020-16-14')
select tm_id,
       tm_name,
       sum(`if`(dt>=date_sub('2020-10-14', 6), order_count, 0)),
       sum(`if`(dt>=date_sub('2020-10-14', 6), order_user_count, 0)),
       sum(`if`(dt>=date_sub('2020-10-14', 6), order_num, 0)),
       sum(`if`(dt>=date_sub('2020-10-14', 6), order_total_amount, 0)),
       sum(order_count),
       sum(order_user_count),
       sum(order_num),
       sum(order_total_amount)
from dws_trade_tm_order_1d
where dt>date_sub('2020-06-14', 29)
group by tm_id,tm_name;
```



##### 汇总表的设计原则

- 公用性越强越好
- 不包含需要汇总计算的指标【例如用户等，后边需要在7/30时间段内汇总用户数】
- 只选择维度的组合，不要使用具体的维度的属性， 使用【用户， sku】而不选择品牌，品类等维度的具体属性

##### 按照用户商品建表

- 前一天与 n 天

```sql
DROP TABLE IF EXISTS dws_trade_user_sku_order_1d;
CREATE EXTERNAL TABLE dws_trade_user_sku_order_1d
(
    `user_id`                   STRING COMMENT '用户id',
    `sku_id`                    STRING COMMENT 'sku_id',
    `sku_name`                  STRING COMMENT 'sku名称',
    `category1_id`              STRING COMMENT '一级分类id',
    `category1_name`            STRING COMMENT '一级分类名称',
    `category2_id`              STRING COMMENT '一级分类id',
    `category2_name`            STRING COMMENT '一级分类名称',
    `category3_id`              STRING COMMENT '一级分类id',
    `category3_name`            STRING COMMENT '一级分类名称',
    `tm_id`                     STRING COMMENT '品牌id',
    `tm_name`                   STRING COMMENT '品牌名称',
    `order_count_1d`            BIGINT COMMENT '最近1日下单次数',
    `order_num_1d`              BIGINT COMMENT '最近1日下单件数',
    `order_original_amount_1d`  DECIMAL(16, 2) COMMENT '最近1日下单原始金额',
    `activity_reduce_amount_1d` DECIMAL(16, 2) COMMENT '最近1日活动优惠金额',
    `coupon_reduce_amount_1d`   DECIMAL(16, 2) COMMENT '最近1日优惠券优惠金额',
    `order_total_amount_1d`     DECIMAL(16, 2) COMMENT '最近1日下单最终金额'
) COMMENT '交易域用户商品粒度订单最近1日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_user_sku_order_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');
    
----------------- n 天    
DROP TABLE IF EXISTS dws_trade_user_sku_order_nd;
CREATE EXTERNAL TABLE dws_trade_user_sku_order_nd
(
    `user_id`                    STRING COMMENT '用户id',
    `sku_id`                     STRING COMMENT 'sku_id',
    `sku_name`                   STRING COMMENT 'sku名称',
    `category1_id`               STRING COMMENT '一级分类id',
    `category1_name`             STRING COMMENT '一级分类名称',
    `category2_id`               STRING COMMENT '一级分类id',
    `category2_name`             STRING COMMENT '一级分类名称',
    `category3_id`               STRING COMMENT '一级分类id',
    `category3_name`             STRING COMMENT '一级分类名称',
    `tm_id`                      STRING COMMENT '品牌id',
    `tm_name`                    STRING COMMENT '品牌名称',
    `order_count_7d`             STRING COMMENT '最近7日下单次数',
    `order_num_7d`               BIGINT COMMENT '最近7日下单件数',
    `order_original_amount_7d`   DECIMAL(16, 2) COMMENT '最近7日下单原始金额',
    `activity_reduce_amount_7d`  DECIMAL(16, 2) COMMENT '最近7日活动优惠金额',
    `coupon_reduce_amount_7d`    DECIMAL(16, 2) COMMENT '最近7日优惠券优惠金额',
    `order_total_amount_7d`      DECIMAL(16, 2) COMMENT '最近7日下单最终金额',
    `order_count_30d`            BIGINT COMMENT '最近30日下单次数',
    `order_num_30d`              BIGINT COMMENT '最近30日下单件数',
    `order_original_amount_30d`  DECIMAL(16, 2) COMMENT '最近30日下单原始金额',
    `activity_reduce_amount_30d` DECIMAL(16, 2) COMMENT '最近30日活动优惠金额',
    `coupon_reduce_amount_30d`   DECIMAL(16, 2) COMMENT '最近30日优惠券优惠金额',
    `order_total_amount_30d`     DECIMAL(16, 2) COMMENT '最近30日下单最终金额'
) COMMENT '交易域用户商品粒度订单最近n日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_user_sku_order_nd'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```

##### 首日数据加载

```sql
--dws_trade_user_order_td
DROP TABLE IF EXISTS dws_trade_user_order_td;
CREATE EXTERNAL TABLE dws_trade_user_order_td
(
    `user_id`            STRING COMMENT '用户id',
    `order_date_first`   STRING COMMENT '首次下单日期',
    `order_date_last`    STRING COMMENT '末次下单日期',
    `order_count`        BIGINT COMMENT '下单次数',
    `order_num`          BIGINT COMMENT '下单件数',
    `order_total_amount` BIGINT COMMENT '下单金额'
) COMMENT '交易域用户粒度订单历史至今汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_user_order_td'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```

##### 每日数据加载

- 需要使用到前一天的数据计算结果，避免数据的重复计算
- 使用全外连接 的 join， 注意SQL 中的 if， NVL函数的使用

```sql
--数据装载
--2020-06-14
insert overwrite table dws_trade_user_order_td partition(dt='2020-06-14')
select
    user_id,
    min(date_id) order_date_first,
    max(date_id) order_date_last,
    count(distinct order_id),
    sum(sku_num),
    sum(split_total_amount)
from dwd_trade_order_detail_inc
group by user_id;

--2020-06-15
-- insert overwrite table dws_trade_user_order_td partition(dt='2020-06-15')
explain formatted
select
    nvl(new.user_id,old.user_id),
    if(old.user_id is null,'2020-06-15',old.order_date_first),
    if(new.user_id is null,old.order_date_last,'2020-06-15'),
    nvl(old.order_count,0)+nvl(new.order_count,0),
    nvl(old.order_num,0)+nvl(new.order_num,0),
    nvl(old.order_total_amount,0)+nvl(new.order_amount,0)
from
(
    select
        user_id,
        order_date_first,
        order_date_last,
        order_count,
        order_num,
        order_total_amount
    from dws_trade_user_order_td
    where dt=date_sub('2020-06-15',1)
)old
full outer join
(
    select
        user_id,
        count(distinct order_id) order_count,
        sum(sku_num) order_num,
        sum(split_total_amount) order_amount
    from dwd_trade_order_detail_inc
    where dt='2020-06-15'
    group by user_id
)new
on old.user_id=new.user_id;
```
