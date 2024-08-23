# 时间相关函数补充

1. date_format

   ```
   date_format(date/timestamp/string, fmt) - 将一个date/timestamp/string变量转化为fmt的格式
   fmt: 'yyyy-MM-dd HH:mm:ss'
   ```

   ```sql
   select date_format(current_date(), 'yyyy/MM');
   ```

2. current_timestamp

   ```
   current_timestamp() - 显示sql执行时的时间戳
   ```

   ```sql
   select current_timestamp();
   ```

3. from_unixtime

   ```
   from_unixtime(unix_time, format) - 将long型的秒数转化为format格式的时间戳
   ```

   ```sql
   select from_unixtime(1655595312, 'yyyy-MM-dd');
   ```

4. from_utc_timestamp

   ```
   from_utc_timestamp(timestamp, string timezone) - 假定timestamp是UTC时间，将之转化到timezone时区
   ```

   ```
   select from_utc_timestamp(current_timestamp(), 'GMT+08:00');
   ```

5. to_unix_timestamp/unix_timestamp

   ```properties
   unix_timestamp(date[, pattern]) - 将时间类型转化为long型，单位为秒
   如果date是标准类型，符合'yyyy-MM-dd HH:mm:ss'，可以直接转化
   否则需要pattern声名类型
   
   ```

   ```SQL
   select to_unix_timestamp(current_timestamp());
   
   ```

6. to_utc_timestamp

   ```
   to_utc_timestamp(timestamp, string timezone) - 假设给定timestamp是timezone时区，将其转化为UTC时间
   
   ```

   ```sql
   select to_utc_timestamp(current_timestamp(), 'GMT+08:00');
   
   ```

   

# SQL练习

```sql
-- 11. 查询每个用户的注册日期、总登录次数以及在2021年的登录次数、订单数和订单总额
with t1 as (
-- 首先查询用户的注册日期和总登录次数
    select user_id,
           min(active_date)                               register_date,
           count(1)                                       login_count,
           count(if(year(active_date) = '2021', 1, null)) login_count_2021
    from user_active
    group by user_id),
     t2 as (
-- 查询订单数和订单总额
         select user_id,
                count(distinct order_id) order_count,
                sum(price)               order_amount
         from order_info_3
         where year(sale_date) = '2021'
         group by user_id)
select t1.*,
       t2.order_count,
       t2.order_amount
from t1
         left join t2 on t1.user_id = t2.user_id;

-- 12. 查询指定日期时候的全部商品价格
select id, nvl(price, 99) price
from (
-- 构建拉链表
         select id,
                new_price        price,
                changeprice_date start_date,
                date_sub(lead(changeprice_date, 1, '9999-12-31') over (partition by id order by changeprice_date),
                         1)      end_date
         from (select id, new_price, changeprice_date
               from price_modification_details
               union all
               select distinct id, 99, date('1970-01-01')
               from price_modification_details) t2) t1
where start_date <= '2022-04-20'
  and end_date >= '2022-04-20';

-- 13. 即时订单比例
select format_number(count(if(order_date = custom_date, 1, null)) / count(1), 2)
from (
         -- 求首单数量和首单中的及时订单数量
         select user_id,
                order_date,
                custom_date,
                rank() over (partition by user_id order by order_date) rnk
         from delivery_info) t1
where rnk = 1;

-- 14. 向用户推荐朋友收藏的商品


-- 14. 首先找到用户1的好友
select distinct fav.id
from user_favorites fav
         join (select if(user1_id = 1, user2_id, user1_id) uid
               from friendship
               where user1_id = 1
                  or user2_id = 1) t1
              on fav.user_id = t1.uid
         left join (select id
                    from user_favorites
                    where user_id = '1') t2 on fav.id = t2.id
where t2.id is null;

-- 15. 查询所有用户的大于等于两天的连续登录区间
select user_id,
       min(active_date),
       max(active_date)
from (select user_id,
             active_date,
             date_sub(active_date, rnk) st
      from (
               -- 首先将用户活跃转化为天为单位
               select user_id,
                      active_date,
                      rank() over (partition by user_id order by active_date) rnk
               from user_active_2
               group by user_id, active_date) t1) t2
group by user_id, st
having count(1) >= 2;


-- 高级1. 各个视频的平均完播率
select k1.video_id,
       count(if(unix_timestamp(end_time) - unix_timestamp(start_time) >= k2.duration, 1, null)) / count(1) finish_rate
from k1_user_video_log k1
         join k2_video_info k2
              on k1.video_id = k2.video_id
group by k1.video_id
order by finish_rate desc;

-- 高级2. 平均播放进度大于60%的视频类别
create table if not exists
    k3_user_video_log
(
    uid        int,
    video_id   int,
    start_time timestamp,
    end_time   timestamp,
    if_follow  tinyint,
    if_like    tinyint,
    if_retweet tinyint,
    comment_id int
)
    row format delimited fields terminated by ','
    stored as textfile;

create table if not exists
    k4_video_info
(
    video_id     int,
    author       int,
    tag          string,
    duration     int,
    release_time timestamp
)
    row format delimited fields terminated by ','
    stored as textfile;

INSERT overwrite table k3_user_video_log
VALUES (101, 2001, '2021-10-01 10:00:00', '2021-10-01 10:00:30', 0, 1, 1, null),
       (102, 2001, '2021-10-01 10:00:00', '2021-10-01 10:00:21', 0, 0, 1, null),
       (103, 2001, '2021-10-01 11:00:50', '2021-10-01 11:01:20', 0, 1, 0, 1732526),
       (102, 2002, '2021-10-01 11:00:00', '2021-10-01 11:00:30', 1, 0, 1, null),
       (103, 2002, '2021-10-01 10:59:05', '2021-10-01 11:00:05', 1, 0, 1, null);

INSERT overwrite table k4_video_info
VALUES (2001, 901, '影视', 30, '2021-01-01 7:00:00'),
       (2002, 901, '美食', 60, '2021-01-01 7:00:00'),
       (2003, 902, '旅游', 90, '2020-01-01 7:00:00');

select tag,
       concat(int(avg(video_progress) * 10000) / 100, '%') avg_pro
from (
-- 首先要查询每个视频的播放进度
         select k3.video_id,
                k4vi.tag,
                if(unix_timestamp(end_time) - unix_timestamp(start_time) >= k4vi.duration,
                   1, (unix_timestamp(end_time) - unix_timestamp(start_time)) / k4vi.duration) video_progress
         from k3_user_video_log k3
                  join k4_video_info k4vi on k3.video_id = k4vi.video_id) t1
group by tag
having avg(video_progress) > 0.6;

-- 高级3. 每类视频近一个月的转发量/率
select k6.tag,
       count(`if`(if_retweet > 0, 1, null))            retweet_count,
       count(`if`(if_retweet > 0, 1, null)) / count(1) retweet_rate
from k5_user_video_log k5
         join k6_video_info k6 on k5.video_id = k6.video_id
where date_format(start_time, 'yyyy-MM-dd') >= date_sub('2021-10-5', 29)
group by k6.tag
order by retweet_rate desc;

-- 高级4. 每个创作者每月的涨粉率及截止当前的总粉丝量
-- 按照up主，视频维度统计涨粉，掉粉，播放数量
select uid,
       month,
       follow_count,
       (follow_count - cancel_count) / total_count,
       sum(follow_count - cancel_count) over (partition by uid order by month) accumulated_fans
from (select date_format(start_time, 'yyyy-MM') month,
             uid,
             count(if(if_follow = 1, 1, null))  follow_count,
             count(if(if_follow = 2, 1, null))  cancel_count,
             count(1)                           total_count
      from k7_user_video_log
      group by date_format(start_time, 'yyyy-MM'), uid) t1
order by uid, accumulated_fans;

-- 高级5. 近一个月发布的视频中热度最高的top3视频
select video_id,
       int((100 * finish_rate + 5 * like_count + 3 * comment_count + 2 * retweet_count) * fresh_rate) hot
from (
-- 首先要求出视频的各个指标
         select k11.video_id,
                count(if(unix_timestamp(end_time) - unix_timestamp(start_time) >= k12.duration, 1, null)) /
                count(1)                                                                     finish_rate,
                sum(if_like)                                                                 like_count,
                count(comment_id)                                                            comment_count,
                sum(if_retweet)                                                              retweet_count,
                1 / (datediff('2021-10-05', date_format(max(end_time), 'yyyy-MM-dd')) + 1) fresh_rate
         from k11_user_video_log k11
                  join k12_video_info k12 on k11.video_id = k12.video_id
-- where date_format(release_time, 'yyyy-MM-dd') >= date_sub('2021-10-05', 29)
         group by k11.video_id) t1
order by hot desc;
```

