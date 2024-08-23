##### 5.1.4写完了，5.1.5还没做呢

改写5.2.7

# SQL练习-中级十道题

```sql
-- 1. 求销量第二的商品，没有返回null
select id
from (select 1) t2
         left join (select id
                    from (select id,
                                 rank() over (order by sum desc) rnk
                          from order_summary) t1
                    where rnk = 2) t3;

-- 2. 连续登录至少三天的用户
-- 套路1
select user_id
from (select user_id,
             active_date,
             --rn和date应该都是连续的
             row_number() over (partition by user_id order by active_date) rn
      from (
               -- 首先将表的粒度转化为以天为单位
               select user_id,
                      active_date
               from user_active
               group by user_id, active_date) t1) t2
group by user_id, date_sub(active_date, rn)
having count(1) >= 3;

-- 套路2
select distinct user_id
from (select user_id,
             active_date,
             -- 两行之后的登录日期今天差值应该正好为2
             lead(active_date, 2, null) over (partition by user_id order by active_date) tmp
      from (
               -- 首先将表的粒度转化为以天为单位
               select user_id,
                      active_date
               from user_active
               group by user_id, active_date) t1) t2
where date_add(active_date, 2) = tmp;

-- 3. 各品类的商品个数及销量最高的商品
INSERT overwrite table category
VALUES ('1', '数码'),
       ('2', '日用'),
       ('3', '厨房清洁');
-- 考虑有并列的情况
select category_name,
       count(id)                           cnt,
       -- min(if(rnk =1, id, null)),
       collect_list(if(rnk = 1, id, null)) most_sales
from (select category_id,
             id,
             rank() over (partition by category_id order by sum desc) rnk
      from (select category_id,
                   id,
                   sum(sum) sum
            from order_summary
            group by category_id, id) t2) t1
         join mid.category on t1.category_id = category.category_id
group by category_name;

-- 4. 查询用户的累计消费总额及VIP等级
select user_id,
       buy_date,
       leiji,
       case
           when leiji < 5000 then 'vip1'
           when leiji >= 5000 and leiji < 10000 then 'vip2'
           else 'vip3' end
from (select user_id,
             buy_date,
             sum,
             sum(sum) over (partition by user_id order by buy_date
                 rows between unbounded preceding and current row) leiji
      from user_consum_details) t1;

-- 5. 查询首次消费后第二天连续消费的用户比率
select count(if(date_add(buy_date, 1) = next_order, 1, null)) / count(1)
-- select *
from (select user_id,
             buy_date,
             lead(buy_date, 1) over (partition by user_id order by buy_date) next_order,
             rank() over (partition by user_id order by buy_date)            rnk
      from user_consum_details) t1
where rnk = 1;

-- 6. 每个销售产品第一年的销售数量、销售年份、销售总额
-- 首先求出每个商品每个年份的统计，再取第一年
select id,
       `year`,
       total_num,
       total_amount
from (select id,
             year(sale_date)                                        `year`,
             sum(num)                                               total_num,
             sum(price)                                             total_amount,
             rank() over (partition by id order by year(sale_date)) rnk
      from order_detail
      group by id, year(sale_date)) t1
where rnk = 1;

-- 7. 筛选去年总销量小于10的商品
select o.id,
       p.name,
       sum(num) total_num
from order_detail o
         join product_attr p on o.id = p.id
-- 筛选上架超过一个月的产品
where date_add(p.from_date, 30) < '2022-01-10'
-- 筛选 去年的销量
  and year(sale_date) = year('2022-01-01') - 1
group by o.id, p.name
-- 筛选去年销量小于10的商品
having total_num < 10;

-- 8. 查询每日新用户数
-- 首先找到所有用户的首次登录
select first_login_date,
       count(user_id) cnt
from (select user_id,
             min(active_date) first_login_date
      from user_login_detail
      group by user_id) t1
-- 过滤最近90天
where date_add(first_login_date, 90) >= '2022-01-10'
group by first_login_date;

-- 9. 每个商品的销售最多的日期

select id,
       min(sale_date) max_sale_date
from (
-- 先求每个商品每天买了多少件, 并按照销售的件数降序排名
         select id,
                sale_date,
                rank() over (partition by id order by sum(num) desc) rnk
         from order_detail2
         group by sale_date, id) t1
-- 过滤销售最多的日期
where rnk = 1
group by id
order by id;

-- 10. 查询销售件数高于品类平均数的商品
-- 首先求出每件商品销售数量
select category_id,
       id,
       total_num,
       category_avg
from (
         -- 在商品销售明细基础上加一列品类平均销量
         select p.category_id,
                t1.id,
                t1.total_num,
                avg(total_num) over (partition by p.category_id) category_avg
         from (
                  -- 先求商品总销售量
                  select id, sum(num) total_num
                  from order_detail_3
                  group by id) t1
                  join product_attr_2 p on t1.id = p.id) t2
where total_num > category_avg;
```
