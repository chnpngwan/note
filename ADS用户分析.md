#### 11.1.2 路径分析

使用桑基图，分析用户的页面浏览行为

![1660304634731](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1660304634731.png)



排序规则

![1660269732189](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1660269732189.png)

##### 建表语句

```sql
-- 11.1.2 路径分析

-- 1）建表语句
DROP TABLE IF EXISTS ads_page_path;
CREATE EXTERNAL TABLE ads_page_path
(
    `dt`         STRING COMMENT '统计日期',
    `source`     STRING COMMENT '跳转起始页面ID',
    `target`     STRING COMMENT '跳转终到页面ID',
    `path_count` BIGINT COMMENT '跳转次数'
) COMMENT '页面浏览路径分析'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_page_path/';
DROP TABLE IF EXISTS ads_user_change;
```

##### 数据装载

- 数据都在 `dwd_traffic_page_view_inc` 表中， 查询表中的数据，按照session-id进行分区排序，将下一行的数据的page-id当做trget，避免source-id出现空值， 对两个数据进行排序，组装stage，避免出现环状图形

```sql
-- 2）数据装载
-- 桑基图的具体要求
--      1. source 不能为空：【涉及到跨行的情况会使用到开窗函数】；
--      2. 数据中不能成环
--          1. 用户回跳操作，每个阶段添加序号，使；

-- 按照会话分区， 同一个会话中，下一个会话 是不同的任务，用户
--      注意数据中加标记的标记规则， 第二列顺序是 2， 3， 4， 5， 6， 第一列： 1,2， 3， 4， 5；

insert overwrite table ads_page_path
select *
from ads_page_path
union
select '2020-06-14' dt,
       source,
       nvl(target, 'null'),
       count(*)     path_count
from (
         select concat('step-', rn, ':', page_id)          source,
                concat('step-', rn + 1, ':', next_page_id) target
         from (
                  select page_id,
                         lead(page_id, 1, null) over (partition by session_id order by view_time) next_page_id,
                         row_number() over (partition by session_id order by view_time)           rn
                  from dwd_traffic_page_view_inc
                  where dt = '2020-06-14'
              ) t1
     ) t2
group by source, target;
```

#### 用户变动分析

- 用户的流失㔿回流

##### 建表语句

```sql
-- 用户变动分析
-- 1）建表语句
DROP TABLE IF EXISTS ads_user_change;
CREATE EXTERNAL TABLE ads_user_change
(
    `dt`               STRING COMMENT '统计日期',
    `user_churn_count` BIGINT COMMENT '流失用户数',
    `user_back_count`  BIGINT COMMENT '回流用户数'
) COMMENT '用户变动统计'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_user_change/';
```

##### 数据装载

- 在表 `dws_user_user_login_td` 中查到今天的数据，并且其中用户的最后登录日期正好为7天之前，就是流式用户数
- 昨天数据的用户最后登录与今天用户的最后登录时间的差值超过 7 天， 说明用户连续7 天没有登录， 今天登录了，用户回归

```sql
insert overwrite table ads_user_change
select *
from ads_user_change
union
select churn.dt,
       user_churn_count,
       user_back_count
from (
         select '2020-06-14' dt,
                count(*)     user_churn_count
         from dws_user_user_login_td
         where dt = '2020-06-14'
           and login_date_last = date_add('2020-06-14', -7)
     ) churn
         join
     (
         select '2020-06-14' dt,
                count(*)     user_back_count
         from (
                  select user_id,
                         login_date_last
                  from dws_user_user_login_td
                  where dt = '2020-06-14'
              ) t1
                  join
              (
                  select user_id,
                         login_date_last login_date_previous
                  from dws_user_user_login_td
                  where dt = date_add('2020-06-14', -1)
              ) t2
              on t1.user_id = t2.user_id
         where datediff(login_date_last, login_date_previous) >= 8
     ) back
     on churn.dt = back.dt;
```

#### 用户留存率

##### 数据表的定义

- 

```sql
-- 用户留存率
-- 1）建表语句

DROP TABLE IF EXISTS ads_user_retention;
CREATE EXTERNAL TABLE ads_user_retention
(
    `dt`              STRING COMMENT '统计日期',
    `create_date`     STRING COMMENT '用户新增日期',
    `retention_day`   INT COMMENT '截至当前日期留存天数',
    `retention_count` BIGINT COMMENT '留存用户数量',
    `new_user_count`  BIGINT COMMENT '新增用户数量',
    `retention_rate`  DECIMAL(16, 2) COMMENT '留存率'
) COMMENT '用户留存率'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_user_retention/';
```

##### 数据装载

- 用户在之前 7 天内注册的新用户， 在今天登录了，就是用户的 n 天留存
- 注意用户的留存天数的是动态的， 在sql中计算出来的

```sql
select
    '2020-06-14',
    date_id,
    datediff('2020-06-14',date_id),
    sum(if(login_date_last='2020-06-14',1,0)),
    count(*),
    sum(if(login_date_last='2020-06-14',1,0))/count(*)
from
(
    select
        user_id,
        date_id
    from dwd_user_register_inc
    where dt>=date_sub('2020-06-14',7)
    and dt<'2020-06-14'
)t1
join
(
    select
        user_id,
        login_date_last
    from dws_user_user_login_td
    where dt='2020-06-14'
)t2
on t1.user_id=t2.user_id
group by date_id;
```

#### 11.2.3 用户新增活跃统计

##### 建表

```sql
-- 11.2.3 用户新增活跃统计
-- 需求说明如下
--      统计周期	        指标	        指标说明
--      最近1、7、30日		新增用户数		略
--      最近1、7、30日		活跃用户数		略
-- 1）建表语句
DROP TABLE IF EXISTS ads_user_stats;
CREATE EXTERNAL TABLE ads_user_stats
(
    `dt`                STRING COMMENT '统计日期',
    `recent_days`       BIGINT COMMENT '最近n日,1:最近1日,7:最近7日,30:最近30日',
    `new_user_count`    BIGINT COMMENT '新增用户数',
    `active_user_count` BIGINT COMMENT '活跃用户数'
) COMMENT '用户新增活跃统计'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_user_stats/';
```

##### 数据装载

- 使用 explode 函数对数据进行预处理
- 直接查询登录与活跃的用户数
- 注意where条件， 可以先筛选出最近30天的数据，对数据进行预处理
- hive有谓词下推 优化， 

```sql
select
    recent_days,
    count(*)
from dwd_user_register_inc lateral view explode(array(1,7,30)) tmp as recent_days
where dt>=date_sub('2020-06-14',29)
and dt>=date_sub('2020-06-14',recent_days-1)
group by recent_days;

select
    recent_days,
    count(*)
from dws_user_user_login_td lateral view explode(array(1,7,30)) tmp as recent_days
where dt='2020-06-14'
and login_date_last>=date_sub('2020-06-14',29)
and login_date_last>=date_sub('2020-06-14',recent_days-1)
group by recent_days;
```



```sql
insert overwrite table ads_user_stats
select * from ads_user_stats
union
select
    '2020-06-14' dt,
    t1.recent_days,
    new_user_count,
    active_user_count
from
(
    select
        recent_days,
        sum(if(login_date_last>=date_add('2020-06-14',-recent_days+1),1,0)) new_user_count
    from dws_user_user_login_td lateral view explode(array(1,7,30)) tmp as recent_days
    where dt='2020-06-14'
    group by recent_days
)t1
join
(
    select
        recent_days,
        sum(if(date_id>=date_add('2020-06-14',-recent_days+1),1,0)) active_user_count
    from dwd_user_register_inc lateral view explode(array(1,7,30)) tmp as recent_days
    group by recent_days
)t2
on t1.recent_days=t2.recent_days;
```



#### 11.2.4 用户行为漏斗分析

```sql
 -- 11.2.4 用户行为漏斗分析
-- 漏斗分析是一个数据分析模型，它能够科学反映一个业务流程从起点到终点各阶段用户转化情况。由于其能将各阶段环节都展示出来，故哪个阶段存在问题，就能一目了然。
--
-- 该需求要求统计一个完整的购物流程各个阶段的人数，具体说明如下：
-- 统计周期	指标	说明
-- 最近1 日	首页浏览人数	略
-- 最近1 日	商品详情页浏览人数	略
-- 最近1 日	加购人数	略
-- 最近1 日	下单人数	略
-- 最近1 日	支付人数	支付成功人数
--
-- 1）建表语句
         DROP TABLE IF EXISTS ads_user_action;
CREATE EXTERNAL TABLE ads_user_action
(
    `dt`                STRING COMMENT '统计日期',
    `home_count`        BIGINT COMMENT '浏览首页人数',
    `good_detail_count` BIGINT COMMENT '浏览商品详情页人数',
    `cart_count`        BIGINT COMMENT '加入购物车人数',
    `order_count`       BIGINT COMMENT '下单人数',
    `payment_count`     BIGINT COMMENT '支付人数'
) COMMENT '漏斗分析'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_user_action/';
```



##### 数据装载

- 直接查表之后join就行
- 注意一个表中含有两个字段的时候，只查询一次表，使用sum-if 嵌套查询统计字段

```sql
insert overwrite table ads_user_action
select * from ads_user_action
union
select
    '2020-06-14' dt,
    home_count,
    good_detail_count,
    cart_count,
    order_count,
    payment_count
from
(
    select
        1 recent_days,
        sum(if(page_id='home',1,0)) home_count,
        sum(if(page_id='good_detail',1,0)) good_detail_count
    from dws_traffic_page_visitor_page_view_1d
    where dt='2020-06-14'
    and page_id in ('home','good_detail')
)page
join
(
    select
        1 recent_days,
        count(*) cart_count
    from dws_trade_user_cart_add_1d
    where dt='2020-06-14'
)cart
on page.recent_days=cart.recent_days
join
(
    select
        1 recent_days,
        count(*) order_count
    from dws_trade_user_order_1d
    where dt='2020-06-14'
)ord
on page.recent_days=ord.recent_days
join
(
    select
        1 recent_days,
        count(*) payment_count
    from dws_trade_user_payment_1d
    where dt='2020-06-14'
)pay
on page.recent_days=pay.recent_days;
```



#### 11.2.5 新增下单用户统计

```sql
-- 11.2.5 新增下单用户统计
-- 需求说明如下
-- 统计周期	指标	说明
-- 最近1、7、30日	新增下单人数	略
-- 1）建表语句
DROP TABLE IF EXISTS ads_new_order_user_stats;
CREATE EXTERNAL TABLE ads_new_order_user_stats
(
    `dt`                   STRING COMMENT '统计日期',
    `recent_days`          BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `new_order_user_count` BIGINT COMMENT '新增下单人数'
) COMMENT '新增交易用户统计'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/warehouse/gmall/ads/ads_new_order_user_stats/';
```

##### 数据装载

- 使用explode 函数将数据进行预处理
- 当表中的最早下单字段的值在范围之内的时候，说明该用户值在该范围之内首次下单的

```sql
insert overwrite table ads_new_order_user_stats
select * from ads_new_order_user_stats
union
select
    '2020-06-14' dt,
    recent_days,
    sum(if(order_date_first>=date_add('2020-06-14',-recent_days+1),1,0)) new_order_user_count
from dws_trade_user_order_td lateral view explode(array(1,7,30)) tmp as recent_days
where dt='2020-06-14'
group by recent_days;
```









