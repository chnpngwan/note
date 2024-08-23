##### DIM层

##### 商品维度表： `dim_sku_full`

```sql
-- 维度表
    -- 主维表 & 相关维表
        -- 都是业务中的表，如果维度表是用户，那么业务系统的用户表称之为主维表，和用户相关的表称之为相关维表
    -- ORC & Snappy

-- 商品维度表 ： dim_sku_full
    -- 表字段（维度属性）？
        -- 主维表 - 商品表（SKU, SPU）
        -- 存储策略 - 每日全量(2020-06-14)

-- 问题
    -- join执行出错，使用left join
        -- 原因：Hive中的向量化查询优化，有问题
        -- 解决方案：set hive.vectorized.execution.enabled=false;
    -- HiveServer2是否启动Metastore
        -- hive-site.xml文件配置了hive.metastore.uris参数，必须启动Metastore，否则可以不用启动
    -- 间断性连接中断
        -- Hive内存资源不够
    -- create Spark Session Fail
        -- Yarn资源不够
        -- 超时
            -- set hive.spark.client.connect.timeout=300000
    -- SparkTask fail
        -- mapJoin
            -- hive-site.xml配置文件增加配置
                -- hive.auto.convert.join  => false


DROP TABLE IF EXISTS dim_sku_full;
CREATE EXTERNAL TABLE dim_sku_full
(
    `id`                   STRING COMMENT 'sku_id',
    `price`                DECIMAL(16, 2) COMMENT '商品价格',
    `sku_name`             STRING COMMENT '商品名称',
    `sku_desc`             STRING COMMENT '商品描述',
    `weight`               DECIMAL(16, 2) COMMENT '重量',
    `is_sale`              BOOLEAN COMMENT '是否在售',
    `spu_id`               STRING COMMENT 'spu编号',
    `spu_name`             STRING COMMENT 'spu名称',
    `category3_id`         STRING COMMENT '三级分类id',
    `category3_name`       STRING COMMENT '三级分类名称',
    `category2_id`         STRING COMMENT '二级分类id',
    `category2_name`       STRING COMMENT '二级分类名称',
    `category1_id`         STRING COMMENT '一级分类id',
    `category1_name`       STRING COMMENT '一级分类名称',
    `tm_id`                STRING COMMENT '品牌id',
    `tm_name`              STRING COMMENT '品牌名称',
    `sku_attr_values`      ARRAY<STRUCT<attr_id :STRING,value_id :STRING,attr_name :STRING,value_name:STRING>> COMMENT '平台属性',
    `sku_sale_attr_values` ARRAY<STRUCT<sale_attr_id :STRING,sale_attr_value_id :STRING,sale_attr_name :STRING,sale_attr_value_name:STRING>> COMMENT '销售属性',
    `create_time`          STRING COMMENT '创建时间'
) COMMENT '商品维度表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_sku_full/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

-- 数据加载
    -- 数据来源：ODS - 8张
    -- JOIN
    -- CTE 语法（公共表表达式）: 将查询结果集作为一个临时表使用，但是只对当前SQL有效
    -- VIEW

set hive.vectorized.execution.enabled=false;
explain with
sku as (
    select
        id
        ,spu_id
        ,price
        ,sku_name
        ,sku_desc
        ,weight
        ,tm_id
        ,category3_id
        ,is_sale
        ,create_time
    from ods_sku_info_full
    where dt = "2020-06-14"
),
spu as (
    select
        id
        ,spu_name
    from ods_spu_info_full
    where dt = "2020-06-14"
),
category3 as (
    select
        id
        ,name
        ,category2_id
    from ods_base_category3_full
    where dt = "2020-06-14"
),
category2 as (
    select
        id
        ,name
        ,category1_id
    from ods_base_category2_full
    where dt = "2020-06-14"
),
category1 as (
    select
        id
        ,name
    from ods_base_category1_full
    where dt = "2020-06-14"
),
tm as (
    select
        id
        ,tm_name
    from ods_base_trademark_full
    where dt = "2020-06-14"
),
attr_value as (
    select
        sku_id,
        collect_set(
            named_struct(
                "attr_id", attr_id,
                "value_id", value_id,
                "attr_name", attr_name,
                "value_name", value_name
            )
        ) sku_attr_values
    from ods_sku_attr_value_full
    where dt = '2020-06-14'
    group by sku_id
),
sale_attr_value as (
    select
        sku_id,
        collect_set(
            named_struct(
                "sale_attr_id", sale_attr_id,
                "sale_attr_value_id", sale_attr_value_id,
                "sale_attr_name", sale_attr_name,
                "sale_attr_value_name", sale_attr_value_name
            )
        ) sku_sale_attr_values
    from ods_sku_sale_attr_value_full
    where dt = '2020-06-14'
    group by sku_id
)
insert overwrite table dim_sku_full partition (dt="2020-06-14")
select
    sku.id
    ,price
    ,sku_name
    ,sku_desc
    ,weight
    ,is_sale
    ,spu_id
    ,spu_name
    ,category3_id
    ,category3.name category3_name
    ,category2_id
    ,category2.name category2_name
    ,category1_id
    ,category1.name category1_name
    ,tm_id
    ,tm_name
    ,sku_attr_values
    ,sku_sale_attr_values
    ,create_time
from sku
join spu on spu.id = sku.spu_id
join category3 on sku.category3_id = category3.id
join category2 on category2.id = category3.category2_id
join category1 on category1.id = category2.category1_id
join tm on sku.tm_id = tm.id
join attr_value on sku.id = attr_value.sku_id
join sale_attr_value on sku.id = sale_attr_value.sku_id
```

##### 活动维度表： `dim_activity_full`

```sql
-- 活动维度表
    -- 主维表 - 活动规则（粒度最细）

DROP TABLE IF EXISTS dim_activity_full;
CREATE EXTERNAL TABLE dim_activity_full
(
    `activity_rule_id`   STRING COMMENT '活动规则ID',
    `activity_id`        STRING COMMENT '活动ID',
    `activity_name`      STRING COMMENT '活动名称',
    `activity_type_code` STRING COMMENT '活动类型编码',
    `activity_type_name` STRING COMMENT '活动类型名称',
    `activity_desc`      STRING COMMENT '活动描述',
    `start_time`         STRING COMMENT '开始时间',
    `end_time`           STRING COMMENT '结束时间',
    `create_time`        STRING COMMENT '创建时间',
    `condition_amount`   DECIMAL(16, 2) COMMENT '满减金额',
    `condition_num`      BIGINT COMMENT '满减件数',
    `benefit_amount`     DECIMAL(16, 2) COMMENT '优惠金额',
    `benefit_discount`   DECIMAL(16, 2) COMMENT '优惠折扣',
    `benefit_rule`       STRING COMMENT '优惠规则',
    `benefit_level`      STRING COMMENT '优惠级别'
) COMMENT '活动信息表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_activity_full/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

insert overwrite table dim_activity_full partition (dt = '2020-06-14')
select
    rule.id activity_rule_id
    ,activity_id
    ,activity_name
    ,activity_type
    ,at.dic_name activity_type_name
    ,activity_desc
    ,start_time
    ,end_time
    ,create_time
    ,condition_amount
    ,condition_num
    ,benefit_amount
    ,benefit_discount
    ,case rule.activity_type
        when '3101' then concat('满',condition_amount,'元减',benefit_amount,'元')
        when '3102' then concat('满',condition_num,'件打',10*(1-benefit_discount),'折')
        when '3103' then concat('打',10*(1-benefit_discount),'折')
    end benefit_rule
    ,benefit_level
from (
    select
        id,
        activity_id,
        activity_type,
        condition_amount,
        condition_num,
        benefit_amount,
        benefit_discount,
        benefit_level
    from ods_activity_rule_full
    where dt='2020-06-14'
) rule
left join (
    select
        id,
        activity_name,
        activity_desc,
        start_time,
        end_time,
        create_time
    from ods_activity_info_full
    where dt='2020-06-14'
) info on rule.activity_id = info.id
left join (
    select
        dic_code,
        dic_name
    from ods_base_dic_full
    where dt='2020-06-14' and parent_code = '31'
) at on rule.activity_type = at.dic_code;

-- 地区维度表
DROP TABLE IF EXISTS dim_province_full;
CREATE EXTERNAL TABLE dim_province_full
(
    `id`            STRING COMMENT 'id',
    `province_name` STRING COMMENT '省市名称',
    `area_code`     STRING COMMENT '地区编码',
    `iso_code`      STRING COMMENT '旧版ISO-3166-2编码，供可视化使用',
    `iso_3166_2`    STRING COMMENT '新版IOS-3166-2编码，供可视化使用',
    `region_id`     STRING COMMENT '地区id',
    `region_name`   STRING COMMENT '地区名称'
) COMMENT '地区维度表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_province_full/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

insert overwrite table dim_province_full partition(dt='2020-06-14')
select
    province.id,
    province.name,
    province.area_code,
    province.iso_code,
    province.iso_3166_2,
    region_id,
    region_name
from
(
    select
        id,
        name,
        region_id,
        area_code,
        iso_code,
        iso_3166_2
    from ods_base_province_full
    where dt='2020-06-14'
)province
left join
(
    select
        id,
        region_name
    from ods_base_region_full
    where dt='2020-06-14'
)region
on province.region_id=region.id;

-- 日期维度表
    -- 日期表的数据不是同步采集过来的，而是上传文件来的
    -- 文件一般为文本文件，而表采用ORC的格式，所以不匹配
    -- 可以采用中间表中转一下：text => text table => select => insert => orc table
DROP TABLE IF EXISTS dim_date;
CREATE EXTERNAL TABLE dim_date
(
    `date_id`    STRING COMMENT '日期ID',
    `week_id`    STRING COMMENT '周ID,一年中的第几周',
    `week_day`   STRING COMMENT '周几',
    `day`        STRING COMMENT '每月的第几天',
    `month`      STRING COMMENT '一年中的第几月',
    `quarter`    STRING COMMENT '一年中的第几季度',
    `year`       STRING COMMENT '年份',
    `is_workday` STRING COMMENT '是否是工作日',
    `holiday_id` STRING COMMENT '节假日'
) COMMENT '时间维度表'
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_date/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

DROP TABLE IF EXISTS tmp_dim_date_info;
CREATE EXTERNAL TABLE tmp_dim_date_info (
    `date_id` STRING COMMENT '日',
    `week_id` STRING COMMENT '周ID',
    `week_day` STRING COMMENT '周几',
    `day` STRING COMMENT '每月的第几天',
    `month` STRING COMMENT '第几月',
    `quarter` STRING COMMENT '第几季度',
    `year` STRING COMMENT '年',
    `is_workday` STRING COMMENT '是否是工作日',
    `holiday_id` STRING COMMENT '节假日'
) COMMENT '时间维度表'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/tmp/tmp_dim_date_info/';

insert overwrite table dim_date select * from tmp_dim_date_info;
```

##### 优惠券维度表： `dim_coupon_full;`

```sql
-- 优惠券维度表

-- 建表语句
    -- 主维表 - 业务系统的优惠券表

DROP TABLE IF EXISTS dim_coupon_full;
CREATE EXTERNAL TABLE dim_coupon_full
(
    `id`               STRING COMMENT '购物券编号',
    `coupon_name`      STRING COMMENT '购物券名称',
    `coupon_type_code` STRING COMMENT '购物券类型编码',
    `coupon_type_name` STRING COMMENT '购物券类型名称',
    `condition_amount` DECIMAL(16, 2) COMMENT '满额数',
    `condition_num`    BIGINT COMMENT '满件数',
    `activity_id`      STRING COMMENT '活动编号',
    `benefit_amount`   DECIMAL(16, 2) COMMENT '减金额',
    `benefit_discount` DECIMAL(16, 2) COMMENT '折扣',
    `benefit_rule`     STRING COMMENT '优惠规则:满元*减*元，满*件打*折',
    `create_time`      STRING COMMENT '创建时间',
    `range_type_code`  STRING COMMENT '优惠范围类型编码',
    `range_type_name`  STRING COMMENT '优惠范围类型名称',
    `limit_num`        BIGINT COMMENT '最多领取次数',
    `taken_count`      BIGINT COMMENT '已领取次数',
    `start_time`       STRING COMMENT '可以领取的开始日期',
    `end_time`         STRING COMMENT '可以领取的结束日期',
    `operate_time`     STRING COMMENT '修改时间',
    `expire_time`      STRING COMMENT '过期时间'
) COMMENT '优惠券维度表'
    PARTITIONED BY (`dt` STRING)
    STORED AS ORC
    LOCATION '/warehouse/gmall/dim/dim_coupon_full/'
    TBLPROPERTIES ('orc.compress' = 'snappy');

insert overwrite table dim_coupon_full partition (dt = '2020-06-14')
select
    id
    ,coupon_name
    ,coupon_type
    ,ct.dic_name coupon_type_name
    ,condition_amount
    ,condition_num
    ,activity_id
    ,benefit_amount
    ,benefit_discount
    ,case coupon_type
        when '3201' then concat('满',condition_amount,'元减',benefit_amount,'元')
        when '3202' then concat('满',condition_num,'件打',10*(1-benefit_discount),'折')
        when '3203' then concat('减',benefit_amount,'元')
    end benefit_rule
    ,create_time
    ,range_type
    ,rt.dic_name range_type_name
    ,limit_num
    ,taken_count
    ,start_time
    ,end_time
    ,operate_time
    ,expire_time
from (
    select
        id
        ,coupon_name
        ,coupon_type
        ,condition_amount
        ,condition_num
        ,activity_id
        ,benefit_amount
        ,benefit_discount
        ,create_time
        ,range_type
        ,limit_num
        ,taken_count
        ,start_time
        ,end_time
        ,operate_time
        ,expire_time
    from ods_coupon_info_full
    where dt = '2020-06-14'
) cp
left join (
    select
        dic_code,
        dic_name
    from ods_base_dic_full
    where dt = '2020-06-14' and parent_code = '32'
) ct on cp.coupon_type = ct.dic_code
left join (
    select
        dic_code,
        dic_name
    from ods_base_dic_full
    where dt = '2020-06-14' and parent_code = '33'
) rt on cp.range_type = rt.dic_code
```

