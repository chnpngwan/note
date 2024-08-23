# SQL练习

```sql
-- 创建学生表
DROP TABLE IF EXISTS student;
create table if not exists student
(
    stu_id   string COMMENT '学生id',
    stu_name string COMMENT '学生姓名',
    birthday date COMMENT '出生日期',
    sex      string COMMENT '性别'
)
    row format delimited fields terminated by ','
    stored as textfile;

-- 创建课程表
DROP TABLE IF EXISTS course;
create table if not exists course
(
    course_id   string COMMENT '课程id',
    course_name string COMMENT '课程名',
    tea_id      string COMMENT '任课老师id'
)
    row format delimited fields terminated by ','
    stored as textfile;

-- 创建老师表
DROP TABLE IF EXISTS teacher;
create table if not exists teacher
(
    tea_id   string COMMENT '老师id',
    tea_name string COMMENT '学生姓名'
)
    row format delimited fields terminated by ','
    stored as textfile;

-- 创建分数表
DROP TABLE IF EXISTS score;
create table if not exists score
(
    stu_id    string COMMENT '学生id',
    course_id string COMMENT '课程id',
    course    int COMMENT '成绩'
)
    row format delimited fields terminated by ','
    stored as textfile;


-- 昨日思考题
-- 首先查出学生成绩排名, 找出所有成绩都排名2-3之间的学生
select score.*,
       s.*
from (select s_id
      from (select s_id,
                   c_id,
                   s_score,
                   rank() over (partition by c_id order by s_score desc) rnk
            from score) t1
      group by s_id
      having count(if(rnk between 2 and 3, 1, null)) = count(1)) t2
         join score on t2.s_id = score.s_id
         join student s on t2.s_id = s.s_id;

-- 3.2.1 查询各科成绩最高分和最低分
select course_id,
       max(course) max_score,
       min(course) min_score
from score
group by course_id;

-- 3.3.3 查询名字第一个字相同的人数
select substr(stu_name, 1, 1),
       count(1)
from student
group by substr(stu_name, 1, 1);

-- 5.2.7 查询学过“李体音”老师所教的所有课的同学的学号、姓名
-- 让所有学生和李体音老师的所有课程笛卡尔积
select s.stu_id, stu_name, birthday, sex
from student s
         join course c
         join teacher t on c.tea_id = t.tea_id
         left join score s2 on s.stu_id = s2.stu_id and c.course_id = s2.course_id
where tea_name = '李体音'
group by s.stu_id, stu_name, birthday, sex
having count(if(s2.course is null, 1, null)) = 0;


-- 5.2.8 查询选过'李体音'任意课程的同学
select distinct s.*
from score
         join course c on score.course_id = c.course_id
         join teacher t on c.tea_id = t.tea_id
         join student s on score.stu_id = s.stu_id
where tea_name = '李体音';

-- 5.2.9 查询没学过"李体音"老师讲授的任一门课程的学生姓名
select s.*
from student s
         left join (select stu_id
                    from score
                             join course c on score.course_id = c.course_id
                             join teacher t on c.tea_id = t.tea_id
                    where tea_name = '李体音') t1
                   on s.stu_id = t1.stu_id
where t1.stu_id is null;

-- 5.2.10 查询选修“李体音”老师所授课程的学生中成绩最高的学生姓名及其成绩
-- 假设不考虑课程
select stu_name, course
from (select stu_id, course, rank() over (order by course desc) rnk
      from score
               join course c on score.course_id = c.course_id
               join teacher t on c.tea_id = t.tea_id
      where tea_name = '李体音') t1
         join student s on t1.stu_id = s.stu_id
where rnk = 1;

-- 考虑不同课程
select stu_name, course_name, course
from (select stu_id, c.course_name, course, rank() over (partition by score.course_id order by course desc) rnk
      from score
               join course c on score.course_id = c.course_id
               join teacher t on c.tea_id = t.tea_id
      where tea_name = '李体音') t1
         join student s on t1.stu_id = s.stu_id
where rnk = 1;

-- 5.1.3 查询平均成绩大于85的所有学生的学号、姓名和平均成绩
select s.*, avg_score
from student s
         join (
    -- 首先查询平均成绩高于85分的学生
    select stu_id, avg(course) avg_score
    from score
    group by stu_id
    having avg_score > 85) t1
              on s.stu_id = t1.stu_id;

-- 5.1.8 使用sql实现将该表行转列为下面的表结构
select stu_id,
       max(if(course_id = '01', course, null)),
       max(if(course_id = '02', course, null)),
       max(if(course_id = '03', course, null)),
       max(if(course_id = '04', course, null))
from score
group by stu_id;

-- 5.2.3 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
select s.stu_id, s.stu_name, t1.avg_score
from student s
         join (
    -- 查询不及格科目两科以上的人
    select stu_id, avg(course) avg_score
    from score
    group by stu_id
    having count(if(course < 60, 1, null)) >= 2) t1
              on s.stu_id = t1.stu_id;

-- 6.1.3 查询每门课程成绩最好的前两名学生姓名
-- 是否要考虑并列
select s.stu_id, course_id, stu_name, course
from (select stu_id,
             course_id,
             course,
             rank() over (partition by course_id order by course desc)       rnk,
             row_number() over (partition by course_id order by course desc) rn
      from score) t1
join student s on t1.stu_id = s.stu_id
where rnk <=2;

-- 6.1.5 查询各科成绩前三名的记录
select course_id, course
    from (
select course_id,
       course,
       dense_rank() over (partition by course_id order by course desc) dr
from score) t1
where dr<=3;
```

##### 5.1.4写完了，5.1.5还没做呢
