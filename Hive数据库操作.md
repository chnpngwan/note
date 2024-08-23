# HIVE DDL

1. 数据库DDL

   ```sql
   -- 新建数据库并指定文件夹
   create database test comment 'Just for test' location '/abcd' with dbproperties('aaa'='bbb');
   ```

   ```sql
   -- 查询数据库列表
   show databases;
   -- 查询某一个数据库详情
   desc database test;
   -- 查询更详细信息
   desc database extended test;
   ```

   ```sql
   -- 修改数据库（只能修改属性）
   alter database test set dbproperties('aaa'='ccc');
   ```

   ```sql
   -- 删除数据库
   drop database test;
   -- 强制删除（慎用）
   drop database test cascade;
   ```

2. 表的DDL

   ```sql
   -- 普通建表
   create table student2(id int comment 'id', name string COMMENT 'nnnn')
   COMMENT 'student2 shi wo'
   row format delimited fields terminated by '\t'
              collection items terminated by '_'
              map keys         terminated by ':'
   tblproperties('aaa'='bbb');
   
   -- 根据查询结果建表
   create table xxx as 
   select name, 
          friends[0]             friend1, 
          children['xiao song']  xiaosong, 
          address.street 
   from test;
   
   -- 拷贝表结构
   create table stu3 like student2;
   ```

   ```sql
   -- 查询表列表
   show tables;
   
   -- 查询表结构
   desc student2;
   
   -- 查询更详细的信息
   desc formatted student2;
   ```

   ```sql
   -- 重命名表格
   alter table student2 rename to stuxxx;
   
   -- 修改列信息
   alter table stuxxx change column id idx bigint comment 'idxxx';
   
   -- 追加列信息
   alter table stuxxx add columns (age int comment 'age', daughter string comment 'nver');
   
   -- 替换列信息
   alter table stuxxx replace columns (id string comment 'abc', name string comment 'abcd');
   ```

   ```sql
   -- 删除表
   drop table stuxxx;
   
   -- 删除表但不删除数据
   truncate table stuxxx;
   ```

# HIVE DML

1. 插入数据

   ```sql
   -- 创建表格student
   create table student (id int, name string) row format delimited fields terminated by '\t';
   
   -- 从本地文件系统将数据导入表格
   load data local inpath '/opt/module/hive/datas/student.txt' into table student;
   
   -- 从本地文件系统中将数据覆盖导入表格
   load data local inpath '/opt/module/hive/datas/student.txt' overwrite into table student;
   
   -- 从HDFS将数据导入表格（覆盖）(会将数据移动进表格的数据文件夹)
   load data inpath '/datas/student.txt' overwrite into table student;
   
   ```

   ```sql
   -- 创建一张student2表格
   create table student2 (id int, name string) row format delimited fields terminated by '\t';
   
   -- insert插入
   insert into student2 values (1001, 'zhangsan'),(1002, 'lisi');
   
   -- insert 查询结果(最常用，记住)
   insert into student2 select id, name from student where id > 1002;
   
   -- insert 查询结果(覆盖)
   insert overwrite table student2 select id, name from student where id > 1002;
   
   ```

   ```sql
   -- 将查询结果直接建表（不常用）
   create table student3 as select id, name from student;
   
   -- 通过location指定地址加载数据
   -- 首先将student数据上传到/datas
   create external table student4 (id int, name string) 
   row format delimited fields terminated by '\t'
   location '/datas';
   
   ```

2. 导出数据

   ```sql
   -- insert导出(了解即可)
   insert overwrite local directory '/opt/module/hive/datas/export/student' select id, name from student;
   
   -- 指定分隔符
   insert overwrite local directory '/opt/module/hive/datas/export/student2' 
   row format delimited fields terminated by ','
   select id, name from student;
   
   -- 向HDFS导出
   insert overwrite directory '/stu2_out' 
   row format delimited fields terminated by ','
   select id, name from student;
   
   ```

   ```sql
   -- import 和 export(了解即可)
   export table student to '/stu_out';
   import table stuin from '/stu_out';
   
   ```

# 内部表和外部表

```sql
-- 普通的建表(管理表)
create table student (id int, name string) row format delimited fields terminated by '\t';

-- 创建外部表
create external table stu_ex (id int, name string) row format delimited fields terminated by '\t';

-- 向外部表中插入数据
insert into stu_ex select id,name from student;

-- 删除普通表格时，数据会一起删掉
drop table student;

-- 删除外部表，数据不会删
drop table stu_ex;

```

```sql
-- 外部表和内部表转换
create table stu_ex (id int, name string) row format delimited fields terminated by '\t';

-- 通过修改表将内部表转化为外部表(key value都要大写)
alter table stu_ex set tblproperties('EXTERNAL'='TRUE');

-- 将外部表改回管理表
alter table stu_ex set tblproperties('EXTERNAL'='FALSE');

```

- UDAF函数执行的时候NULL不参与运算
  - 注意avg（）函数，
- betwen and  左右闭区间

- 内连接，左连杰，右连接，全连接
- group by having
- left join on
- 排序一般结合limit使用，

# HIVE 查询

1. 基本查询

   ```sql
   -- 创建部门表
   create table if not exists dept(
       deptno int,    -- 部门编号
       dname string,  -- 部门名称
       loc int        -- 部门位置
   )
   row format delimited fields terminated by '\t';
   
   -- 创建员工表
   create table if not exists emp(
       empno int,      -- 员工编号
       ename string,   -- 员工姓名
       job string,     -- 员工岗位（大数据工程师、前端工程师、java工程师）
       sal double,     -- 员工薪资
       deptno int      -- 部门编号
   )
   row format delimited fields terminated by '\t';
   
   -- 导入数据
   load data local inpath '/opt/module/hive/datas/dept.txt' into table dept;
   load data local inpath '/opt/module/hive/datas/emp.txt' into table emp;
   ```

   ```sql
   -- 查询特定列
   select ename, sal from emp;
   
   -- 起别名，as可以省略
   select ename as name, sal salary from emp;
   
   -- 查询中可以放运算符
   select ename, sal + 10 from emp;
   
   -- 常用函数（UDAF）
   select count(1) cnt,               -- 数个数
          sum(sal) sum_sal,           -- 求和
          avg(sal) avg_sal,           -- 求平均
          min(sal) min_sal,           -- 最小值
          max(sal) max_sal            -- 最大值
   from emp;
   
   -- UDAF函数执行时候，NULL不参与运算的
   select avg(children['xiao song']), 
          min(children['xiao song']), 
          max(children['xiao song']), 
          sum(children['xiao song']),
          count(children['xiao song'])
   from test;
   
   -- 显示前5
   select ename from emp limit 5;
   
   -- 显示2-5名
   select ename from emp limit 1,4;
   
   -- 过滤 查询工资大于1000的人
   select ename, sal from emp where sal>1000;
   
   -- 安全等与 <=> 演示，对比下面两句查询
   select ename, job from emp where job = null;
   select ename, job from emp where job <=> null;
   
   -- 字符串（通配符）以张开头的人
   select * from emp where ename like "张%";
   -- 字符串（正则表达式）以张开头的人
   select * from emp where ename rlike "^张";
   ```

2. 分组和分组过滤

   ```sql
   -- 查询各个部门的平均工资
   select deptno, avg(sal) from emp 
   group by deptno;
   
   -- 查询平均工资大于2000的部门和其平均工资
   select deptno, avg(sal) avg_sal from emp 
   group by deptno 
   having avg_sal>2000;
   ```

3. 连接

   ```sql
   -- 查询员工姓名，员工部门名称
   select emp.ename, dept.dname from emp inner join dept on emp.deptno = dept.deptno;
   
   -- 为了演示多种连接区别，向emp和dept表插入数据
   insert into emp values(7360, '死鬼', '失业', null, 50);
   insert into dept values(60, '不存在', 1700);
   -- 左连接
   select emp.ename, dept.dname from emp left join dept on emp.deptno = dept.deptno;
   -- 右连接
   select emp.ename, dept.dname from emp right join dept on emp.deptno = dept.deptno;
   -- 全连接
   select emp.ename, dept.dname from emp full join dept on emp.deptno = dept.deptno;
   
   -- 多表连接
   create table if not exists location(
       loc int,          -- 部门位置id
       loc_name string   -- 部门位置
   )
   row format delimited fields terminated by '\t';
   load data local inpath '/opt/module/hive/datas/location.txt' into table location;
   -- 查询员工姓名，部门名称，单位名称
   select e.ename,
          d.dname,
          l.loc_name
   from emp e
       right join dept d on e.deptno = d.deptno
       left join location l on d.loc = l.loc;
   
   -- 笛卡尔积 (不写连接条件会产生笛卡尔积)
   select * from emp join dept;
   
   -- 注意下面的情况
   insert into dept values(40, 'XXXX', 1800);
   select emp.ename, dept.dname from emp inner join dept on emp.deptno = dept.deptno;
   ```

4. 排序

   ```sql
   -- 按照工资从高到低排序
   select * from emp order by sal desc;
   -- 排序可以用别名
   select ename, sal salary from emp order by salary desc;
   -- 先按照部门编号升序排序，相同部门按照工资降序排序
   select * from emp order by deptno asc, sal desc;
   -- 按照部门平均工资降序给部门排序
   select deptno, avg(sal) avg_sal from emp group by deptno order by avg_sal desc;
   
   -- 全局排序数据量大可能会遇到性能问题，所以排序一般结合limit使用
   select * from emp order by sal desc limit 5;
   
   -- 分区和分区排序
   -- 首先设置reduce数量为3
   set mapreduce.job.reduces=3;
   
   -- distribute by指定查询按照什么字段的hash分区, sort by在分区内部排序
   select * from emp distribute by empno sort by empno desc;
   
   -- 当distribute by和sort by字段相同且为升序时，可以用cluster by代替
   select * from emp distribute by empno sort by empno;
   -- 相当于
   select * from emp cluster by empno;
   ```

   5.子查询
   
   ```sql
   -- 各科成绩排名
   select name,subject,score,
          rank() over(partition by subject order by score desc) `rank`,
          dense_rank() over(partition by subject order by score desc) `dense_rank`,
          row_number() over(partition by subject order by score desc) `row_number`
   from score;
   
   -- 在上表基础上进行查询各科前3
   select name,subject,score
   from (
       select name,subject,score,
          rank() over(partition by subject order by score desc) `rank`,
          dense_rank() over(partition by subject order by score desc) `dense_rank`,
          row_number() over(partition by subject order by score desc) `row_number`
       from score
   ) t1
   where `rank`<=3;
   ```
   