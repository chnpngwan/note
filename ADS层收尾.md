#### 11.2.6 最近7日内连续3日下单用户数

1. 三种方法
   1. 开窗使用 lead（）函数取后片的两行的数据放在同一行， 之后判断两个时间相减
   2. 使用开窗函数 rank（） 函数排序， 之后时间与排序相减，同一天的数据分到同一组，判断组内数据条数
   3. 使用 1000 数据相加， 最后使用 like 进行模糊匹配

```sql

-- 11.2.6 最近7日内连续3日下单用户数
-- 1）建表语句
DROP TABLE IF EXISTS ads_order_continuously_user_count;
CREATE EXTERNAL TABLE ads_order_continuously_user_count
(
    `dt`                            STRING COMMENT '统计日期',
    `recent_days`                   BIGINT COMMENT '最近天数,7:最近7天',
    `order_continuously_user_count` BIGINT COMMENT '连续3日下单用户数'
) COMMENT '新增交易用户统计'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_order_continuously_user_count/';
-- 题干分析： 用户下单粒度
--  ；
-- 数据装载: 去重
--      distinct () : 返回的是一个结构体
--      group by（）
--      开窗+过滤：开窗后加一个 rownumber，之后排序 加一个  where过滤 只取分区中的一个;

--      跨行操作---》使用开窗
--          方案一：
select count(distinct user_id)

from (
         select user_id,
                datediff(lead2, dt) diff
         from (
                  select user_id,
                         dt,
                         lead(dt, 2, '9999-12-31') over (partition by user_id order by dt) lead2
                  from dws_trade_user_order_1d
                  where dt >= date_sub('2020-06-14', 6)
              ) t1
     ) t2
where diff = 2;
--          方案二：
--          使用rank 函数： rank() over(partition by user_id order by dt)
--          rank函数是连续的，日期如果也是连续的，那么日期与rank的 差值就是相同的
--          得到diff的差值之后，分组聚合count 之后检查连续个数;

select count(distinct (user_id))
from (
         select user_id,
                diff,
                count(*) cnt
         from (
                  select user_id,
                         date_sub(dt, rk) diff
                  from (
                           select user_id,
                                  dt,
                                  rank() over (partition by user_id order by dt) rk
                           from dws_trade_user_order_1d
                           where dt >= date_sub('2020-06-14', 6)
                       ) t1
              ) t2
         group by user_id, diff
     ) t3
where cnt >= 3;


-- 方案三：
select count(*)
from (
         select user_id,
                sum(num) count
         from (
                  select user_id,
                         case dt
                             when '2020-06-14' then 1
                             when '2020-06-13' then 10
                             when '2020-06-12' then 100
                             when '2020-06-11' then 1000
                             when '2020-06-10' then 10000
                             when '2020-06-09' then 100000
                             when '2020-06-08' then 1000000
                             end num
                  from dws_trade_user_order_1d
                  where dt >= date_sub('2020-06-14', 6)
              ) t1
         group by user_id
     ) t2
where count like '%111%';

-- 间断一天也算连续的算法 ，
select count(distinct user_id)
from (
         select user_id,
                datediff(lead2, dt) diff
         from (
                  select user_id,
                         dt,
                         lead(dt, 2, '9999-12-31') over (partition by user_id order by dt) lead2
                  from dws_trade_user_order_1d
                  where dt >= date_sub('2020-06-14', 6)
              ) t1
     ) t2
where diff <= 3;

```

#### 11.3.1 最近30日各品牌复购率

- 直接查询表中的数据 `dws_trade_user_sku_order_nd`：分组之后求和， 个数大于1个就是购买过得， 大于等于3个就是多次购买的，嵌套子查询 直接计算比值就行

```sql
-- 11.3.1 最近30日各品牌复购率
-- 需求说明如下   (以下单作为购买的人数)
--          统计周期	    统计粒度	    指标	    说明
--          最近30日	    品牌	        复购率	重复购买人数占购买人数比例
-- 1）建表语句
DROP TABLE IF EXISTS ads_repeat_purchase_by_tm;
CREATE EXTERNAL TABLE ads_repeat_purchase_by_tm
(
    `dt`                STRING COMMENT '统计日期',
    `recent_days`       BIGINT COMMENT '最近天数,30:最近30天',
    `tm_id`             STRING COMMENT '品牌ID',
    `tm_name`           STRING COMMENT '品牌名称',
    `order_repeat_rate` DECIMAL(16, 2) COMMENT '复购率'
) COMMENT '各品牌复购率统计'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_repeat_purchase_by_tm/';

-- 数据装载
--      复购率：多次购买 / 购买过
--      计算最近30 日每个用户购买（下单）每个品牌的次数，
--      多次的粒度转换；

select tm_id,
       tm_name,
       sum(`if`(order_count >= 3, 1, 0)) / sum(`if`(order_count >= 1, 1, 0))
from (
         select user_id,
                tm_id,
                tm_name,
                sum(order_count_30d) order_count

         from dws_trade_user_sku_order_nd
         where dt = '2020-06-14'
         group by user_id, tm_id, tm_name
     ) t1
group by tm_id, tm_name;

```



#### 11.3.2 各品牌商品下单统计

1. 直接在对应的表中查询相应的字段 `dws_trade_user_sku_order_nd`： 注意添加不存在的字段， 三个查询直接使用 union
2. 30 与7 天的数据都在nd表中， 可以直接查询表，省去一份 union
3. 直接查询一次使用 case-when-end 将数据分开

```sql
-- 11.3.2 各品牌商品下单统计
-- 需求说明如下
-- 统计周期	        统计粒度	    指标	    说明
-- 最近1、7、30日	    品牌	        订单数	    略
-- 最近1、7、30日	    品牌	        订单人数	    略
-- 1）建表语句
DROP TABLE IF EXISTS ads_order_stats_by_tm;
CREATE EXTERNAL TABLE ads_order_stats_by_tm
(
    `dt`               STRING COMMENT '统计日期',
    `recent_days`      BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `tm_id`            STRING COMMENT '品牌ID',
    `tm_name`          STRING COMMENT '品牌名称',
    `order_count`      BIGINT COMMENT '订单数',
    `order_user_count` BIGINT COMMENT '订单人数'
) COMMENT '各品牌商品交易统计'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_order_stats_by_tm/';


--数据装载
--方案一
--1d
select
    '2020-06-14' dt,
    1,
    tm_id,
    tm_name,
    sum(order_count_1d),
    count(distinct user_id)
from dws_trade_user_sku_order_1d
where dt='2020-06-14'
group by tm_id,tm_name
union all
--7d
select
    '2020-06-14' dt,
    7,
    tm_id,
    tm_name,
    sum(order_count_7d),
    count(distinct if(order_count_7d>0,user_id,null))
from dws_trade_user_sku_order_nd
where dt='2020-06-14'
group by tm_id,tm_name
union all
--30d
select
    '2020-06-14' dt,
    30,
    tm_id,
    tm_name,
    sum(order_count_30d),
    count(distinct if(order_count_30d>0,user_id,null))
from dws_trade_user_sku_order_nd
where dt='2020-06-14'
group by tm_id,tm_name;


--方案二
select
    '2020-06-14' dt,
    recent_days,
    tm_id,
    tm_name,
    sum(order_count),
    count(distinct if(order_count>0,user_id,null))
from
(
    select
        7 recent_days,
        user_id,
        tm_id,
        tm_name,
        cast(order_count_7d as bigint) order_count
    from dws_trade_user_sku_order_nd
    where dt='2020-06-14'
    union all
    select
        30 recent_days,
        user_id,
        tm_id,
        tm_name,
        order_count_30d
    from dws_trade_user_sku_order_nd
    where dt='2020-06-14'
)t1
group by recent_days,tm_id,tm_name;

--方案三
select
    '2020-06-14' dt,
    recent_days,
    tm_id,
    tm_name,
    sum(order_count),
    count(distinct if(order_count>0,user_id,null))
from
(
    select
        recent_days,
        user_id,
        tm_id,
        tm_name,
        case recent_days
            when 7 then order_count_7d
            when 30 then order_count_30d
        end order_count
    from dws_trade_user_sku_order_nd lateral view explode(array(7,30)) tmp as recent_days
    where dt='2020-06-14'
)t1
group by recent_days,tm_id,tm_name;

```



####  11.3.4 各分类商品购物车存量Top3

- 周期快照事实表中直接查到需要的数据， left-join 上需要的字段信息的另一张表，嵌套子查询时候使用开窗排序，添加一列，再在嵌套的子查询中使用条件过滤 出想要的数据
- 分类求 topn 是模式化的操作-记住
-  HiveServer2 报错： too maney open files   :HivrServer2 打开的文件太多了，默认每个进程只能打开4096， 调整配置文件修改文件上限【直接修改配置文件，修改默认的设置，让一个用户能够打开的文件多一点】

```sql
-- 11.3.4 各分类商品购物车存量Top3
-- 1）建表语句
DROP TABLE IF EXISTS ads_sku_cart_num_top3_by_cate;
CREATE EXTERNAL TABLE ads_sku_cart_num_top3_by_cate
(
    `dt`             STRING COMMENT '统计日期',
    `category1_id`   STRING COMMENT '一级分类ID',
    `category1_name` STRING COMMENT '一级分类名称',
    `category2_id`   STRING COMMENT '二级分类ID',
    `category2_name` STRING COMMENT '二级分类名称',
    `category3_id`   STRING COMMENT '三级分类ID',
    `category3_name` STRING COMMENT '三级分类名称',
    `sku_id`         STRING COMMENT '商品id',
    `sku_name`       STRING COMMENT '商品名称',
    `cart_num`       BIGINT COMMENT '购物车中商品数量',
    `rk`             BIGINT COMMENT '排名'
) COMMENT '各分类商品购物车存量Top10'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_sku_cart_num_top3_by_cate/';

--数据装载
select
    '2020-06-14' dt,
    category1_id,
    category1_name,
    category2_id,
    category2_name,
    category3_id,
    category3_name,
    sku_id,
    sku_name,
    cart_num,
    rk
from
(
    select
        category1_id,
        category1_name,
        category2_id,
        category2_name,
        category3_id,
        category3_name,
        sku_id,
        sku_name,
        cart_num,
        rank() over (partition by category1_id,category2_id,category3_id order by cart_num desc) rk
    from
    (
        select
            sku_id,
            sum(sku_num) cart_num
        from dwd_trade_cart_full
        where dt='2020-06-14'
        group by sku_id
    )cart
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
            category3_name
        from dim_sku_full
        where dt='2020-06-14'
    )sku
    on cart.sku_id=sku.id
)t1
where rk<=3;

-- 数据装载： 分组topn
--      开窗， rank 对字段添加序号， 之后过滤
--      周期型对存量的事实做全量同步；

-- HiveServer2 报错： too maney open files   :HivrServer2 打开的文件太多了，默认每个进程只能打开4096， 调整配置文件修改文件上限
```

#### 11.4.1 下单到支付时间间隔平均值

1. 直接在累积性事物事实表中查找想要的字段， 在查询的过程中直接使用 avg 计算平均值，
   1. 注意周期快照表的 dt 分区字段的含义， 某一天的平均时间， 包括昨天的全量表中的数据， 还包括未完成的分区中的数据， 因为 只有确认收货的时候才会将数据移动分区

```sql
-- 11.4.1 下单到支付时间间隔平均值
-- 具体要求：最近1日完成支付的订单的下单时间到支付时间的时间间隔的平均值。
-- 1）建表语句
DROP TABLE IF EXISTS ads_order_to_pay_interval_avg;
CREATE EXTERNAL TABLE ads_order_to_pay_interval_avg
(
    `dt`                        STRING COMMENT '统计日期',
    `order_to_pay_interval_avg` BIGINT COMMENT '下单到支付时间间隔平均值,单位为秒'
) COMMENT '下单到支付时间间隔平均值'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_order_to_pay_interval_avg/';



--数据装载
select
    avg(unix_timestamp(payment_time)-unix_timestamp(order_time))
from dwd_trade_trade_flow_acc
where dt in ('2020-06-14','9999-12-31')
and payment_date_id='2020-06-14';

-- 累积性快照事实表，基于业务流程中的多个关键业务流程中的一个关键业务流程
--      作用：分析业务过程中的【多事物关联统计】；
-- 数据装载

select avg(to_unix_timestamp(payment_time) - to_unix_timestamp(order_time))
from dwd_trade_trade_flow_acc
where dt in ('9999-12-31', '2020-06-14')
  and payment_date_id = '2020-06-14';


```

#### 11.4.2 各省份交易统计

1. 直接在对应的1d表中查询到 1 天的数据， 
2. 然后使用 `lateral view explode(array(7,30))` 对nd 表中的数据进行炸裂操作， 后边使用 case-when-end对想用的数据进行判断，获取目标数据
3. 计算出来的表直接 union 就行， nd 中查询到两类数据的格式相同，可以直接union

```sql
-- 11.4.2 各省份交易统计
-- 需求说明如下
-- 统计周期	统计粒度	指标	说明
-- 最近1、7、30日	省份	订单数	略
-- 最近1、7、30日	省份	订单金额	略
-- 1）建表语句
DROP TABLE IF EXISTS ads_order_by_province;
CREATE EXTERNAL TABLE ads_order_by_province
(
    `dt`                 STRING COMMENT '统计日期',
    `recent_days`        BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `province_id`        STRING COMMENT '省份ID',
    `province_name`      STRING COMMENT '省份名称',
    `area_code`          STRING COMMENT '地区编码',
    `iso_code`           STRING COMMENT '国际标准地区编码',
    `iso_code_3166_2`    STRING COMMENT '国际标准地区编码',
    `order_count`        BIGINT COMMENT '订单数',
    `order_total_amount` DECIMAL(16, 2) COMMENT '订单金额'
) COMMENT '各地区订单统计'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_order_by_province/';

desc dws_trade_province_order_nd;

--数据装载
insert overwrite table ads_order_by_province
select * from ads_order_by_province
union
select
    '2020-06-14' dt,
    1 recent_days,
    province_id,
    province_name,
    area_code,
    iso_code,
    iso_3166_2,
    order_count_1d,
    order_total_amount_1d
from dws_trade_province_order_1d
where dt='2020-06-14'
union
select
    '2020-06-14' dt,
    recent_days,
    province_id,
    province_name,
    area_code,
    iso_code,
    iso_3166_2,
    case recent_days
        when 7 then order_count_7d
        when 30 then order_count_30d
    end order_count,
    case recent_days
        when 7 then order_total_amount_7d
        when 30 then order_total_amount_30d
    end order_total_amount
from dws_trade_province_order_nd lateral view explode(array(7,30)) tmp as recent_days
where dt='2020-06-14';

```

#### 11.5.1 优惠券使用统计

1. 直接按照优惠券进行分组，然后对每个使用数量进行求和，
2. 直接count 数据条数 =  使用人数（原来的数据表的粒度就是每个用户使用的优惠券的情况，一行数据代表一个用户 ）

```sql

-- 11.5.1 优惠券使用统计
-- 需求说明如下
-- 统计周期	统计粒度	指标	说明
-- 最近1日	优惠券	使用次数	支付才算使用
-- 最近1日	优惠券	使用人数	支付才算使用
-- 1）建表语句
DROP TABLE IF EXISTS ads_coupon_stats;
CREATE EXTERNAL TABLE ads_coupon_stats
(
    `dt`              STRING COMMENT '统计日期',
    `coupon_id`       STRING COMMENT '优惠券ID',
    `coupon_name`     STRING COMMENT '优惠券名称',
    `used_count`      BIGINT COMMENT '使用次数',
    `used_user_count` BIGINT COMMENT '使用人数'
) COMMENT '优惠券统计'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_coupon_stats/';


--数据装载
select
    coupon_id,
    coupon_name,
    sum(used_count_1d),
    count(*)
from dws_tool_user_coupon_coupon_used_1d
where dt='2020-06-14'
group by coupon_id,coupon_name;
```

##### DataX的简单回顾