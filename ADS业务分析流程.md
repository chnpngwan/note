#### 10.1.1 交易域用户商品粒度订单最近1日汇总表

##### 商品设计原则

**设计要点：**

```properties
设计要点：
1）DWS层的设计参考指标体系。
2）DWS层的数据存储格式为orc列式存储+snappy压缩。
3）DWS层表名的命名规范为dws_数据域_统计粒度_业务过程_统计周期（1d/nd/td）
注：1d表示最近1日，nd表示最近n日，td表示历史至今。
```



- 订单表中某个用户购买某个商品的金额

```sql
-- 10.1.1 交易域用户商品粒度订单最近1日汇总表
-- 1）建表语句

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
--
-- 2）数据装载
-- （1）首日装载
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dws_trade_user_sku_order_1d partition(dt)
select
    user_id,
    id,
    sku_name,
    category1_id,
    category1_name,
    category2_id,
    category2_name,
    category3_id,
    category3_name,
    tm_id,
    tm_name,
    order_count_1d,
    order_num_1d,
    order_original_amount_1d,
    activity_reduce_amount_1d,
    coupon_reduce_amount_1d,
    order_total_amount_1d,
    dt
from
(
    select
        dt,
        user_id,
        sku_id,
        count(*) order_count_1d,
        sum(sku_num) order_num_1d,
        sum(split_original_amount) order_original_amount_1d,
        sum(nvl(split_activity_amount,0.0)) activity_reduce_amount_1d,
        sum(nvl(split_coupon_amount,0.0)) coupon_reduce_amount_1d,
        sum(split_total_amount) order_total_amount_1d
    from dwd_trade_order_detail_inc
    group by dt,user_id,sku_id
)od
left join
(
    select
        id,
        sku_name,
        category1_id,
        category1_name,
        category2_id,
        category2_name,
        category3_id,
        category3_name,
        tm_id,
        tm_name
    from dim_sku_full
    where dt='2020-06-14'
)sku
on od.sku_id=sku.id;

show partitions dws_trade_user_sku_order_1d;
-- 首日 6.14 之前的数据，使用leftjoin 会有空数据，

-- 每日装载
insert overwrite table dws_trade_user_sku_order_1d partition(dt='2020-06-15')
select
    user_id,
    id,
    sku_name,
    category1_id,
    category1_name,
    category2_id,
    category2_name,
    category3_id,
    category3_name,
    tm_id,
    tm_name,
    order_count,
    order_num,
    order_original_amount,
    activity_reduce_amount,
    coupon_reduce_amount,
    order_total_amount
from
(
    select
        user_id,
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(split_original_amount) order_original_amount,
        sum(nvl(split_activity_amount,0)) activity_reduce_amount,
        sum(nvl(split_coupon_amount,0)) coupon_reduce_amount,
        sum(split_total_amount) order_total_amount
    from dwd_trade_order_detail_inc
    where dt='2020-06-15'
    group by user_id,sku_id
)od
left join
(
    select
        id,
        sku_name,
        category1_id,
        category1_name,
        category2_id,
        category2_name,
        category3_id,
        category3_name,
        tm_id,
        tm_name
    from dim_sku_full
    where dt='2020-06-15'
)sku
on od.sku_id=sku.id;
```



#### 10.1.2 交易域用户粒度订单最近1日汇总表

- 最近一天某个用户的下单信息

```sql
-- 10.1.2 交易域用户粒度订单最近1日汇总表
-- 1）建表语句

DROP TABLE IF EXISTS dws_trade_user_order_1d;
CREATE EXTERNAL TABLE dws_trade_user_order_1d
(
    `user_id`                   STRING COMMENT '用户id',
    `order_count_1d`            BIGINT COMMENT '最近1日下单次数',
    `order_num_1d`              BIGINT COMMENT '最近1日下单商品件数',
    `order_original_amount_1d`  DECIMAL(16, 2) COMMENT '最近1日最近1日下单原始金额',
    `activity_reduce_amount_1d` DECIMAL(16, 2) COMMENT '最近1日下单活动优惠金额',
    `coupon_reduce_amount_1d`   DECIMAL(16, 2) COMMENT '下单优惠券优惠金额',
    `order_total_amount_1d`     DECIMAL(16, 2) COMMENT '最近1日下单最终金额'
) COMMENT '交易域用户粒度订单最近1日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_user_order_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');

-- 2）数据装载
-- （1）首日装载

insert overwrite table dws_trade_user_order_1d partition(dt)
select
    user_id,
    count(distinct(order_id)),
    sum(sku_num),
    sum(split_original_amount),
    sum(nvl(split_activity_amount,0)),
    sum(nvl(split_coupon_amount,0)),
    sum(split_total_amount),
    dt
from dwd_trade_order_detail_inc
group by user_id,dt;
```

#### 10.1.3 交易域用户粒度加购最近1日汇总表

```sql
-- 10.1.3 交易域用户粒度加购最近1日汇总表
-- 1）建表语句

DROP TABLE IF EXISTS dws_trade_user_cart_add_1d;
CREATE EXTERNAL TABLE dws_trade_user_cart_add_1d
(
    `user_id`           STRING COMMENT '用户id',
    `cart_add_count_1d` BIGINT COMMENT '最近1日加购次数',
    `cart_add_num_1d`   BIGINT COMMENT '最近1日加购商品件数'
) COMMENT '交易域用户粒度加购最近1日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_user_cart_add_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```

#### 10.1.4 交易域用户粒度支付最近1日汇总表

```sql
-- 10.1.4 交易域用户粒度支付最近1日汇总表
-- 1）建表语句

DROP TABLE IF EXISTS dws_trade_user_payment_1d;
CREATE EXTERNAL TABLE dws_trade_user_payment_1d
(
    `user_id`           STRING COMMENT '用户id',
    `payment_count_1d`  BIGINT COMMENT '最近1日支付次数',
    `payment_num_1d`    BIGINT COMMENT '最近1日支付商品件数',
    `payment_amount_1d` DECIMAL(16, 2) COMMENT '最近1日支付金额'
) COMMENT '交易域用户粒度支付最近1日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_user_payment_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```



#### 10.1.5 交易域省份粒度订单最近1日汇总表

```sql
-- 10.1.5 交易域省份粒度订单最近1日汇总表
-- 1）建表语句

DROP TABLE IF EXISTS dws_trade_province_order_1d;
CREATE EXTERNAL TABLE dws_trade_province_order_1d
(
    `province_id`               STRING COMMENT '省份id',
    `province_name`             STRING COMMENT '省份名称',
    `area_code`                 STRING COMMENT '地区编码',
    `iso_code`                  STRING COMMENT '旧版ISO-3166-2编码',
    `iso_3166_2`                STRING COMMENT '新版版ISO-3166-2编码',
    `order_count_1d`            BIGINT COMMENT '最近1日下单次数',
    `order_original_amount_1d`  DECIMAL(16, 2) COMMENT '最近1日下单原始金额',
    `activity_reduce_amount_1d` DECIMAL(16, 2) COMMENT '最近1日下单活动优惠金额',
    `coupon_reduce_amount_1d`   DECIMAL(16, 2) COMMENT '最近1日下单优惠券优惠金额',
    `order_total_amount_1d`     DECIMAL(16, 2) COMMENT '最近1日下单最终金额'
) COMMENT '交易域省份粒度订单最近1日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_province_order_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```

#### 10.1.6 工具域用户优惠券粒度优惠券使用(支付)最近1日汇总表

```sql
-- 10.1.6 工具域用户优惠券粒度优惠券使用(支付)最近1日汇总表
-- 1）建表语句

DROP TABLE IF EXISTS dws_tool_user_coupon_coupon_used_1d;
CREATE EXTERNAL TABLE dws_tool_user_coupon_coupon_used_1d
(
    `user_id`          STRING COMMENT '用户id',
    `coupon_id`        STRING COMMENT '优惠券id',
    `coupon_name`      STRING COMMENT '优惠券名称',
    `coupon_type_code` STRING COMMENT '优惠券类型编码',
    `coupon_type_name` STRING COMMENT '优惠券类型名称',
    `benefit_rule`     STRING COMMENT '优惠规则',
    `used_count_1d`    STRING COMMENT '使用(支付)次数'
) COMMENT '工具域用户优惠券粒度优惠券使用(支付)最近1日汇总表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_tool_user_coupon_coupon_used_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```

#### 10.1.7 互动域商品粒度收藏商品最近1日汇总表

```sql
-- 10.1.7 互动域商品粒度收藏商品最近1日汇总表
-- 1）建表语句
DROP TABLE IF EXISTS dws_interaction_sku_favor_add_1d;
CREATE EXTERNAL TABLE dws_interaction_sku_favor_add_1d
(
    `sku_id`             STRING COMMENT '商品id',
    `sku_name`           STRING COMMENT 'sku名称',
    `category1_id`       STRING COMMENT '一级分类id',
    `category1_name`     STRING COMMENT '一级分类名称',
    `category2_id`       STRING COMMENT '一级分类id',
    `category2_name`     STRING COMMENT '一级分类名称',
    `category3_id`       STRING COMMENT '一级分类id',
    `category3_name`     STRING COMMENT '一级分类名称',
    `tm_id`              STRING COMMENT '品牌id',
    `tm_name`            STRING COMMENT '品牌名称',
    `favor_add_count_1d` BIGINT COMMENT '商品被收藏次数'
) COMMENT '互动域商品粒度收藏商品最近1日汇总表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_interaction_sku_favor_add_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```

#### 10.1.8 流量域会话粒度页面浏览最近1日汇总表

```sql
-- 10.1.8 流量域会话粒度页面浏览最近1日汇总表
-- 1）建表语句
DROP TABLE IF EXISTS dws_traffic_session_page_view_1d;
CREATE EXTERNAL TABLE dws_traffic_session_page_view_1d
(
    `session_id`     STRING COMMENT '会话id',
    `mid_id`         string comment '设备id',
    `brand`          string comment '手机品牌',
    `model`          string comment '手机型号',
    `operate_system` string comment '操作系统',
    `version_code`   string comment 'app版本号',
    `channel`        string comment '渠道',
    `during_time_1d` BIGINT COMMENT '最近1日访问时长',
    `page_count_1d`  BIGINT COMMENT '最近1日访问页面数'
) COMMENT '流量域会话粒度页面浏览最近1日汇总表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_traffic_session_page_view_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');
    
    
-- 维度退化，如果一张事实表做了维度退化，有这张事实表建立的汇总表需要做维度退化【原来的事实表中】

-- 2）数据装载
insert overwrite table dws_traffic_session_page_view_1d partition(dt='2020-06-14')
select
    session_id,
    mid_id,
    brand,
    model,
    operate_system,
    version_code,
    channel,
    sum(during_time),
    count(*)
from dwd_traffic_page_view_inc
where dt='2020-06-14'
group by session_id,mid_id,brand,model,operate_system,version_code,channel;

show partitions dwd_traffic_page_view_inc;
-- 事实表来自爱与日志，日志中的时间有预设值，历史日志，日志一般没有历史数据，
-- 事实表中没有历史浏览行为，；    
```

#### 10.1.9 流量域访客页面粒度页面浏览最近1日汇总表

```sql
-- 10.1.9 流量域访客页面粒度页面浏览最近1日汇总表
-- 1）建表语句

DROP TABLE IF EXISTS dws_traffic_page_visitor_page_view_1d;
CREATE EXTERNAL TABLE dws_traffic_page_visitor_page_view_1d
(
    `mid_id`         STRING COMMENT '访客id',
    `brand`          string comment '手机品牌',
    `model`          string comment '手机型号',
    `operate_system` string comment '操作系统',
    `page_id`        STRING COMMENT '页面id',
    `during_time_1d` BIGINT COMMENT '最近1日浏览时长',
    `view_count_1d`  BIGINT COMMENT '最近1日访问次数'
) COMMENT '流量域访客页面粒度页面浏览最近1日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_traffic_page_visitor_page_view_1d'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```



#### 10.2.1 交易域用户商品粒度订单最近n日汇总表

- 最近 n 天 的计算方法， 使用 `sum(if(dt>=date_add('2020-06-14',-6)`， 使用if判断，大于从现在减去6天，也就是7天前的数据，就是最近7天的数据

```sql
-- 10.2.1 交易域用户商品粒度订单最近n日汇总表
-- 1）建表语句
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


--
-- 2）数据装载
insert overwrite table dws_trade_user_sku_order_nd partition(dt='2020-06-14')
select
    user_id,
    sku_id,
    sku_name,
    category1_id,
    category1_name,
    category2_id,
    category2_name,
    category3_id,
    category3_name,
    tm_id,
    tm_name,
    sum(if(dt>=date_add('2020-06-14',-6),order_count_1d,0)),
    sum(if(dt>=date_add('2020-06-14',-6),order_num_1d,0)),
    sum(if(dt>=date_add('2020-06-14',-6),order_original_amount_1d,0)),
    sum(if(dt>=date_add('2020-06-14',-6),activity_reduce_amount_1d,0)),
    sum(if(dt>=date_add('2020-06-14',-6),coupon_reduce_amount_1d,0)),
    sum(if(dt>=date_add('2020-06-14',-6),order_total_amount_1d,0)),
    sum(order_count_1d),
    sum(order_num_1d),
    sum(order_original_amount_1d),
    sum(activity_reduce_amount_1d),
    sum(coupon_reduce_amount_1d),
    sum(order_total_amount_1d)
from dws_trade_user_sku_order_1d
where dt>=date_add('2020-06-14',-29)
group by  user_id,sku_id,sku_name,category1_id,category1_name,category2_id,category2_name,category3_id,category3_name,tm_id,tm_name;
```

#### 10.2.2 交易域省份粒度订单最近n日汇总表

```sql
-- 10.2.2 交易域省份粒度订单最近n日汇总表
-- 1）建表语句
DROP TABLE IF EXISTS dws_trade_province_order_nd;
CREATE EXTERNAL TABLE dws_trade_province_order_nd
(
    `province_id`                STRING COMMENT '省份id',
    `province_name`              STRING COMMENT '省份名称',
    `area_code`                  STRING COMMENT '地区编码',
    `iso_code`                   STRING COMMENT '旧版ISO-3166-2编码',
    `iso_3166_2`                 STRING COMMENT '新版版ISO-3166-2编码',
    `order_count_7d`             BIGINT COMMENT '最近7日下单次数',
    `order_original_amount_7d`   DECIMAL(16, 2) COMMENT '最近7日下单原始金额',
    `activity_reduce_amount_7d`  DECIMAL(16, 2) COMMENT '最近7日下单活动优惠金额',
    `coupon_reduce_amount_7d`    DECIMAL(16, 2) COMMENT '最近7日下单优惠券优惠金额',
    `order_total_amount_7d`      DECIMAL(16, 2) COMMENT '最近7日下单最终金额',
    `order_count_30d`            BIGINT COMMENT '最近30日下单次数',
    `order_original_amount_30d`  DECIMAL(16, 2) COMMENT '最近30日下单原始金额',
    `activity_reduce_amount_30d` DECIMAL(16, 2) COMMENT '最近30日下单活动优惠金额',
    `coupon_reduce_amount_30d`   DECIMAL(16, 2) COMMENT '最近30日下单优惠券优惠金额',
    `order_total_amount_30d`     DECIMAL(16, 2) COMMENT '最近30日下单最终金额'
) COMMENT '交易域省份粒度订单最近n日汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_province_order_nd'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```

#### 10.3.1 交易域用户粒度订单历史至今汇总表

```sql
-- 10.3.1 交易域用户粒度订单历史至今汇总表
-- 1）建表语句
DROP TABLE IF EXISTS dws_trade_user_order_td;
CREATE EXTERNAL TABLE dws_trade_user_order_td
(
    `user_id`                   STRING COMMENT '用户id',
    `order_date_first`          STRING COMMENT '首次下单日期',
    `order_date_last`           STRING COMMENT '末次下单日期',
    `order_count_td`            BIGINT COMMENT '下单次数',
    `order_num_td`              BIGINT COMMENT '购买商品件数',
    `original_amount_td`        DECIMAL(16, 2) COMMENT '原始金额',
    `activity_reduce_amount_td` DECIMAL(16, 2) COMMENT '活动优惠金额',
    `coupon_reduce_amount_td`   DECIMAL(16, 2) COMMENT '优惠券优惠金额',
    `total_amount_td`           DECIMAL(16, 2) COMMENT '最终金额'
) COMMENT '交易域用户粒度订单历史至今汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_trade_user_order_td'
    TBLPROPERTIES ('orc.compress' = 'snappy');
```

#### 10.3.3 用户域用户粒度登录历史至今汇总表

```sql
-- 10.3.3 用户域用户粒度登录历史至今汇总表
-- 1）建表语句
DROP TABLE IF EXISTS dws_user_user_login_td;
CREATE EXTERNAL TABLE dws_user_user_login_td
(
    `user_id`         STRING COMMENT '用户id',
    `login_date_last` STRING COMMENT '末次登录日期',
    `login_count_td`  BIGINT COMMENT '累计登录次数'
) COMMENT '用户域用户粒度登录历史至今汇总事实表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dws/dws_user_user_login_td'
    TBLPROPERTIES ('orc.compress' = 'snappy');

-- 2）数据装载
-- （1）首日装载
insert overwrite table dws_user_user_login_td partition(dt='2020-06-14')
select
    u.id,
    nvl(login_date_last,date_format(create_time,'yyyy-MM-dd')),
    nvl(login_count_td,1)
from
(
    select
        id,
        create_time
    from dim_user_zip
    where dt='9999-12-31'
)u
left join
(
    select
        user_id,
        max(dt) login_date_last,
        count(*) login_count_td
    from dwd_user_login_inc
    group by user_id
)l
on u.id=l.user_id;
```

#### ADS层各个渠道流量统计

- `yarn app -list`： yarn中关闭任务
- 聚合函数只会生成一个结果，所以 select字段查出来的数据要与之对应

```sql
-- 各个渠道流量统计
-- 1）建表语句
DROP TABLE IF EXISTS ads_traffic_stats_by_channel;
CREATE EXTERNAL TABLE ads_traffic_stats_by_channel
(
    `dt`               STRING COMMENT '统计日期',
    `recent_days`      BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `channel`          STRING COMMENT '渠道',
    `uv_count`         BIGINT COMMENT '访客人数',
    `avg_duration_sec` BIGINT COMMENT '会话平均停留时长，单位为秒',
    `avg_page_count`   BIGINT COMMENT '会话平均浏览页面数',
    `sv_count`         BIGINT COMMENT '会话数',
    `bounce_rate`      DECIMAL(16, 2) COMMENT '跳出率'
) COMMENT '各渠道流量统计'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_traffic_stats_by_channel/';

-- 前边的表是明细数据
-- ADS 层保存的是各种结果， 数据量比较少，
-- 按天分区的话会生成过多的小文件，
-- 文件存储格式：使用默认文本格式，没做列式存储， 压缩-【】
-- 列式存储对于分析统计类查询效率高，ADS层存储的是结果，没必要列式存储；

-- 数据装载
--1. 尝试简化需求，先不考虑 周期与粒度， 写出sql之后添加groupby字段

-- yarn app -list
-- yarn kill 任务id;
-- 分组字段选择： 分组之后使用聚合函数，聚合函数的结果只有一个数据， 要保证所有的字段只能有一个值；
```

##### 数据装载

- `explode`函数的用法
- `where dt>=date_add('2020-06-14',-recent_days+1)`：时间减法的使用，使用recent_days,来区分时间分区

```sql
--方案三

-- UDTF函数，lateral view explode(split(category, ',')) tbl as category_id;
-- 幂等性， 小文件， 重复执行（重复数据）
--      保证数据不重复，将数据字段在查询的时候强转为目标类型

insert overwrite table ads_traffic_stats_by_channel
select * from ads_traffic_stats_by_channel
union
select
    '2020-06-14' dt,
    recent_days,
    channel,
    cast(count(distinct(mid_id)) as bigint) uv_count,
    cast(avg(during_time_1d)/1000 as bigint) avg_duration_sec,
    cast(avg(page_count_1d) as bigint) avg_page_count,
    cast(count(*) as bigint) sv_count,
    cast(sum(if(page_count_1d=1,1,0))/count(*) as decimal(16,2)) bounce_rate
from dws_traffic_session_page_view_1d lateral view explode(array(1,7,30)) tmp as recent_days
where dt>=date_add('2020-06-14',-recent_days+1)
group by recent_days,channel;
```

