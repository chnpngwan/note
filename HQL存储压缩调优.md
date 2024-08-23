# HQL练习

```sql
drop table if exists student;
drop table if exists course;
drop table if exists teacher;
drop table if exists score;
create table student (s_id string,s_name string,s_birth string,s_sex string);
create table course  (c_id string,c_name string,t_id string);
create table teacher (t_id string,t_name string);
create table score   (s_id string,c_id string,s_score int);
insert overwrite table student values ("01","赵雷","1990-01-01","男"),("02","钱电","1990-12-21","男"),("03","孙风","1990-05-20","男"),("04","李云","1990-08-06","男"),("05","周梅","1991-12-01","女"),("06","吴兰","1992-03-01","女"),("07","郑竹","1989-07-01","女"),("08","王菊","1990-01-20","女");
insert overwrite table course  values ("01","语文","02"),("02","数学","01"),("03","英语","03");
insert overwrite table teacher values ("01","张三"),("02","李四"),("03","王五");
insert overwrite table score   values ("01","01",80),("01","02",90),("01","03",99),("02","01",70),("02","02",60),("02","03",80),("03","01",80),("03","02",80),("03","03",80),("04","01",50),("04","02",30),("04","03",20),("05","01",76),("05","02",87),("06","01",31),("06","03",34),("07","02",89),("07","03",98);

-- 1. 查询"01"课程比"02"课程成绩高的学生的信息及课程分数:
-- 2. 查询没有学全所有课程的同学的信息:
-- 3. 查询至少有一门课与学号为"01"的同学所学相同的同学的信息:
-- 4. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩:
-- 5. 查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率
--    优秀率:–及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
-- 6. 各科成绩排名
-- 7. 查询所有课程的成绩第2名到第3名的学生信息及该课程成绩:
-- 8. 查询任何一门课程成绩在70分以上的学生姓名、课程名称和分数:
-- 9. 查询选修了全部课程的学生信息:
-- 10.查询各学生的年龄(周岁):
```

- 查询"01"课程比"02"课程成绩高的学生的信息及课程分数:

```sql
select student.s_id, student.s_name, student.s_birth, student.s_sex,
       score.c1, score.sco1, score.c2, score.sco2
from test.student join (
    select s1.s_id, s1.c_id c1, s1.s_score sco1,
           s2.c_id c2, s2.s_score sco2
    from test.score s1
             join test.score s2
                  on s1.s_id=s2.s_id and s1.c_id='01' and s2.c_id='02'
    where s1.s_score>s2.s_score
    ) score on student.s_id = score.s_id;
```

![1655207690795](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220614-1655207690795.png)

- 2. 查询没有学全所有课程的同学的信息:

```sql
select student.*
from test.student
join (
    select  score.s_id,
            sum(1) sum_c
    from test.score
    group by score.s_id
    ) tbl
on student.s_id = tbl.s_id and tbl.sum_c <3;
```

![1655208594153](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220614-1655208594153.png)

- 3 查询至少有一门课与学号为"01"的同学所学相同的同学的信息:

```sql
select student.* from test.student
    where student.s_id in (
        select distinct score.s_id from test.score
        where score.s_score in (
            select score.s_score from test.score where score.s_id='01'
        )
        ) and student.s_id <> '01';
```

![1655209060657](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220614-1655209060657.png)

4. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩:

```sql
select score.* , tbl.avg_score
    from test.score
        join (
        select score.s_id,
               avg(score.s_score) avg_score
        from test.score
        group by score.s_id
    ) tbl on score.s_id=tbl.s_id
order by tbl.avg_score desc ;
```

![1655209676626](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220614-1655209676626.png)

5. 查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率  优秀率:–及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90

```sql
select course.c_id, course.c_name, tbl.max_score, tbl.min_score, tbl.avg_score,
       tbl.jige, tbl.zhongdeng, tbl.youliang, tbl.youxiu
    from course
        join (
        select score.c_id,
               max(score.s_score) max_score,
               min(score.s_score) min_score,
               avg(score.s_score) avg_score,
               (count(`if`(score.s_score>=60, 1, null)) / count(1)) * 100 jige,
               (count(`if`(score.s_score>=70 and score.s_score<80, 1, null)) /count(1)) * 100 zhongdeng,
               (count(`if`(score.s_score>=80 and score.s_score<90, 1, null)) /count(1)) * 100 youliang,
               (count(`if`(score.s_score>=90, 1, null)) / count(1)) * 100 youxiu
        from test.score
        group by score.c_id
        ) tbl on course.c_id = tbl.c_id;
```

![1655210531363](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/20220614-1655210531363.png)

6. 各科成绩排名

```sql
select score.s_id, score.c_id, score.s_score,
       rank() over (partition by score.c_id order by score.s_score desc ) rank
    from test.score;
```

![1655210768002](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220614-1655210768002.png)

7. 查询所有课程的成绩第2名到第3名的学生信息及该课程成绩:

```sql
select student.s_id, student.s_name, student.s_birth, student.s_sex, tbl.c_id, tbl.s_score
    from test.student
        join (
        select score.s_id, score.c_id, score.s_score,
               rank() over (partition by score.c_id order by score.s_score desc ) rank
        from test.score
        )tbl on student.s_id=tbl.s_id
where tbl.rank>=2 and tbl.rank<=3;
```

![1655211182169](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220614-1655211182169.png)

8. 查询任何一门课程成绩在70分以上的学生姓名、课程名称和分数:

```sql
select student.s_name, tbl.c_id, tbl.s_score
    from test.student
        join (
        select *
        from test.score
        where score.s_score>70
        ) tbl on student.s_id=tbl.s_id;
```

![1655211419853](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220614-1655211419853.png)

9. 查询选修了全部课程的学生信息:

```sql
select student.*
    from test.student
        join (
        select score.s_id,
               count(score.s_id) num
        from test.score
        group by score.s_id
        ) tbl on student.s_id=tbl.s_id and tbl.num=3;
```

![1655212156439](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220614-1655212156439.png)

10.查询各学生的年龄(周岁):

```sql
select student.*,
       `floor`(datediff(`current_date`(), student.s_birth) / 365) age
from test.student;
```

![1655212624396](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220614-1655212624396.png)



# SQL-10连解析

```sql
drop table if exists student;
drop table if exists course;
drop table if exists teacher;
drop table if exists score;
create table student (s_id string,s_name string,s_birth string,s_sex string);
create table course  (c_id string,c_name string,t_id string);
create table teacher (t_id string,t_name string);
create table score   (s_id string,c_id string,s_score int);
insert overwrite table student values ("01","赵雷","1990-01-01","男"),("02","钱电","1990-12-21","男"),("03","孙风","1990-05-20","男"),("04","李云","1990-08-06","男"),("05","周梅","1991-12-01","女"),("06","吴兰","1992-03-01","女"),("07","郑竹","1989-07-01","女"),("08","王菊","1990-01-20","女");
insert overwrite table course  values ("01","语文","02"),("02","数学","01"),("03","英语","03");
insert overwrite table teacher values ("01","张三"),("02","李四"),("03","王五");
insert overwrite table score   values ("01","01",80),("01","02",90),("01","03",99),("02","01",70),("02","02",60),("02","03",80),("03","01",80),("03","02",80),("03","03",80),("04","01",50),("04","02",30),("04","03",20),("05","01",76),("05","02",87),("06","01",31),("06","03",34),("07","02",89),("07","03",98);

-- 1. 查询"01"课程比"02"课程成绩高的学生的信息及课程分数:
with t1 as (
  select s_id, c_id, s_score
  from score
  where c_id = '01'
),
t2 as (
  select s_id, c_id, s_score
  from score
  where c_id = '02'
)
SELECT t1.s_id, 
       s.s_name,
       s.s_birth,
       s.s_sex,
       t1.s_score, 
       t2.s_score
FROM t1 join t2 on t1.s_id = t2.s_id
        join student s on t1.s_id = s.s_id
where t1.s_score > t2.s_score;

-- 如果想查询所有成绩
with t1 as (
  SELECT s_id, str_to_map(concat_ws(':',collect_list(concat(c_id, ',', s_score))), ':',',') scores
  FROM score
  group by s_id
)
SELECT s.s_id,
       s.s_name,
       s.s_birth,
       s.s_sex,
       t1.scores['01'] class01,
       t1.scores['02'] class02,
       t1.scores['03'] class03
from t1 join student s
on t1.s_id = s.s_id
where t1.scores['01'] > t1.scores['02'];
-- 2. 查询没有学全所有课程的同学的信息:
select s.s_id, 
       s.s_name,
       s.s_birth,
       s.s_sex
from student s left join 
(
  -- 首先查询选了所有课的人
  select s.s_id
  from score s
  group by s.s_id
  having count(s.c_id) = (select count(c_id) from course)
) t1 on s.s_id = t1.s_id
where t1.s_id is null;  -- left join 以后，t1.s_id 是null的都是没选全的

-- 3. 查询至少有一门课与学号为"01"的同学所学相同的同学的信息:

select distinct st.s_id, 
       st.s_name,
       st.s_birth,
       st.s_sex
from score s join 
(
  -- 首先查询01学生选了几门课
  select c_id
  from score
  where s_id = '01'
) t1
on s.c_id = t1.c_id
join student st on s.s_id = st.s_id
where s.s_id <> '01';


-- 4. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩:
-- 简单方式，开窗求平均
select s_id, c_id, s_score, avg(s_score) over(partition by s_id) avg_score
from score
order by avg_score desc;

-- 如果希望做的漂亮一点
with t1 as (
  SELECT s_id, 
         str_to_map(concat_ws(':',collect_list(concat(c_id, ',', s_score))), ':',',') scores,
         avg(s_score) avg_score
  FROM score
  group by s_id
)
select s_id,
       t1.scores['01'] class01,
       t1.scores['02'] class02,
       t1.scores['03'] class03,
       avg_score
from t1
order by avg_score desc;

-- 5. 查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率
--    优秀率:–及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
SELECT c.c_id,
       c.c_name,
       max(s_score) highest,
       min(s_score) lowest,
       avg(s_score) avg_score,
       count(if(s_score>=60, 1, null)) / count(1) pass_rate,
       count(if(s_score>=70 and s_score<80, 1, null)) / count(1) mid_rate,
       count(if(s_score>=80 and s_score<90, 1, null)) / count(1) good_rate,
       count(if(s_score>=90, 1, null)) / count(1) excellect_rate
FROM score s
join course c on s.c_id = c.c_id
group by c.c_id,c.c_name;

-- 6. 各科成绩排名
SELECT c_id, s_id, s_score, 
       rank() over(partition by c_id order by s_score desc) rnk
from score;

-- 7. 查询 所有课程的成绩第2名到第3名的学生信息及该课程成绩:
SELECT s.*,
       t1.c_id,
       t1.s_score
FROM (
  SELECT c_id, s_id, s_score, 
         rank() over(partition by c_id order by s_score desc) rnk
  from score
) t1
join student s on t1.s_id = s.s_id
where rnk BETWEEN 2 and 3;
-- 思考一下，下午讨论

-- 8. 查询任何 一门课程成绩在70分以上的学生姓名、课程名称和分数:、

select s.*,
       sc.c_id,
       sc.s_score
from (
  -- 首先查询 任何一门课程在70分以上的学生
  select s_id
  from score
  group by s_id
  having min(s_score) >= 70
) t1 join score sc on t1.s_id = sc.s_id
     join student s on t1.s_id = s.s_id;

-- 断句方式改变
select s.*,
       sc.c_id,
       sc.s_score
from (
  -- 首先查询 任意一门课程在70分以上的学生
  select s_id
  from score
  group by s_id
  having max(s_score) >= 70
) t1 join score sc on t1.s_id = sc.s_id
     join student s on t1.s_id = s.s_id;

-- 9. 查询选修了全部课程的学生信息:
select s.s_id, 
       s.s_name,
       s.s_birth,
       s.s_sex
from student s join 
(
  -- 首先查询选了所有课的人
  select s.s_id
  from score s
  group by s.s_id
  having count(s.c_id) = (select count(c_id) from course)
) t1 on s.s_id = t1.s_id;

-- 10.查询各学生的年龄(周岁):

-- 简单粗暴
SELECT s_id, floor(datediff(current_date(), s_birth) / 365) age
FROM student;

-- 正确写法，要按照年月日计算，判断他今年过没过生日，如果过了，

SELECT s_id, 
       if(
       month(current_date()) > month(s_birth) or           -- 过没过生日
       (month(current_date()) = month(s_birth) and day(current_date()) > day(s_birth)),
       year(current_date()) - year(s_birth),               -- 过生日的岁数
       year(current_date()) - year(s_birth) - 1            -- 没过生日的岁数
       ) age
FROM student;
```



# 存储和压缩

1. 测试不同的存储格式

   ```sql
   -- 新建一张未压缩的文本格式表
   create table bigtable_text(
       id bigint, 
       t bigint, 
       uid string, 
       keyword string, 
       url_rank int, 
       click_num int, 
       click_url string
   ) 
   row format delimited fields terminated by '\t';
   
   -- 数据导入
   load data local inpath '/opt/module/hive/datas/bigtable' into table bigtable;
   ```

   ```sql
   -- 创建一张orc存储格式未压缩的表格
   create table bigtable_orc(
       id bigint, 
       t bigint, 
       uid string, 
       keyword string, 
       url_rank int, 
       click_num int, 
       click_url string
   ) 
   row format delimited fields terminated by '\t'
   stored as orc
   tblproperties('orc.compress'='NONE');
   
   -- 数据导入
   insert into bigtable_orc select * from bigtable_text;
   ```

   ```sql
   -- 创建一张parquet存储格式未压缩的表格
   create table bigtable_parquet(
       id bigint, 
       t bigint, 
       uid string, 
       keyword string, 
       url_rank int, 
       click_num int, 
       click_url string
   ) 
   row format delimited fields terminated by '\t'
   stored as parquet;
   
   -- 数据插入
   insert into bigtable_parquet select * from bigtable_text;
   ```

2. 列式存储带压缩

   ```sql
   -- 创建一张orc存储格式zlib的表格
   create table bigtable_orc_zlib(
       id bigint, 
       t bigint, 
       uid string, 
       keyword string, 
       url_rank int, 
       click_num int, 
       click_url string
   ) 
   row format delimited fields terminated by '\t'
   stored as orc
   tblproperties('orc.compress'='ZLIB');
   
   -- 数据导入
   insert into bigtable_orc_zlib select * from bigtable_text;
   ```

   ```sql
   -- 创建一张orc存储格式snappy的表格
   create table bigtable_orc_snappy(
       id bigint, 
       t bigint, 
       uid string, 
       keyword string, 
       url_rank int, 
       click_num int, 
       click_url string
   ) 
   row format delimited fields terminated by '\t'
   stored as orc
   tblproperties('orc.compress'='SNAPPY');
   
   -- 数据导入
   insert into bigtable_orc_snappy select * from bigtable_text;
   ```

   ```sql
   -- 创建一张parquet存储格式GZIP压缩的表格
   create table bigtable_parquet_zlib(
       id bigint, 
       t bigint, 
       uid string, 
       keyword string, 
       url_rank int, 
       click_num int, 
       click_url string
   ) 
   row format delimited fields terminated by '\t'
   stored as parquet
   tblproperties('parquet.compression'='GZIP');
   
   -- 数据插入
   insert into bigtable_parquet_zlib select * from bigtable_text;
   ```

   ```sql
   -- 创建一张parquet存储格式snappy压缩的表格
   create table bigtable_parquet_snappy(
       id bigint, 
       t bigint, 
       uid string, 
       keyword string, 
       url_rank int, 
       click_num int, 
       click_url string
   ) 
   row format delimited fields terminated by '\t'
   stored as parquet
   tblproperties('parquet.compression'='SNAPPY');
   
   -- 数据插入
   insert into bigtable_parquet_snappy select * from bigtable_text;
   ```

   

# 查看执行计划

```sql
explain select * from test.student;
-- explain： 解释说明
```

   1. Fetch Operator

      直接取数据，不用mapreduce

      | 关注重点 | 解释       |
      | -------- | ---------- |
      | limit    | 取几行数据 |

   2. Select Operator

      查询数据

      | 关注重点    | 解释                           |
      | ----------- | ------------------------------ |
      | expressions | 查询的列数                     |
      | Statistics  | 统计信息（行数，文件大小等等） |

   3. Filter Operator

      过滤数据

      | 关注重点  | 解释     |
      | --------- | -------- |
      | predicate | 过滤条件 |

   4. Map Join Operator

      执行MapJoin

   5. Join Operator

      执行ReduceJoin

   6. File Output Operator

      输出结果格式

      | 关注重点   | 解释         |
      | ---------- | ------------ |
      | compressed | 压缩格式     |
      | Statistics | 统计数据     |
      | table      | 最终输出格式 |

   7. Group By Operator

      分组操作

      | 关注重点     | 解释           |
      | ------------ | -------------- |
      | aggregations | 执行的具体操作 |
      | keys         | group by的key  |
      | mode         | 分组方法       |

   8. Reduce Output Operator

      汇总操作，一般指Combiner

      | 关注重点                     | 解释          |
      | ---------------------------- | ------------- |
      | key expressions              | reduce的key   |
      | value expressions            | reduce的value |
      | sort order                   | 升序还是降序  |
      | Map-reduce partition columns | 分区key       |

   9. PTF Operator

      开窗操作

      | 关注重点         | 解释               |
      | ---------------- | ------------------ |
      | partition by     | 分组的列           |
      | order by         | 排序的列           |
      | window functions | 开窗以后执行的操作 |













