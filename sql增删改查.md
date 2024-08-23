##### 笔记

IO流对象

- 今日内容
  - MySQL数据库安装
  - MySQL登录
  - 数据库创建
  - 数据表创建
  - 数据CRUD
  - 数据基本查询
  - 数据条件查询
  - SQL语句运算符
  - 聚合函数
  - 排序查询

# 第一章 数据库介绍

## 1.1 为什么要学习数据库

以前存储数据都是临时性，变量，数组，集合。IO流可以读写文件，数据存储在文件中，持久化了。文件后期操作会带来无穷无尽的烦恼，数据不能共享，不能实时更新。为了解决数据的存储问题，数据的共享问题，数据的更新问题，开发出了数据库软件！

## 1.2 数据库软件介绍

可以存储数据的仓库都是数据库。数据库软件是按照特定的格式，和自己特定的算法将数据存储的软件。数据库软件的全名应该是数据库服务器（DataBase Server）。

- 市场上常见到的数据库
  - 甲骨文公司产品：Oracle，收费的数据库，价格极其昂贵，每个CPU核心 7万$
  - 微软公司产品：SqlServler,收费数据库，价格极其昂贵，哈斯达克（每秒20万条股票记录）
  - IBM公司产品：DB2，收费数据库，国内的银行系统比较多
  - 瑞典MySQLAB公司：MySQL Server，免费，C语言编写，MySQL产品被sun公司收购，sun被oracle收购！

## 1.3 MySQL数据库安装

需要一个软件VC++：  VC_redist.x64.exe 安装包，2015-2022 安装 后没有任何反应

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05171.png)

- MySQL是否已经安装了

命令行：输入 services.msc 开启服务窗口，找服务名字：MySQL，找不到没有装过

桌面：此电脑，右键，选择管理，服务

- MySQL安装

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05172.png)

![3](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05173.png)

![4](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05174.png)

![5](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05175.png)

![6](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05176.png)

![7](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05177.png)

![8](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05178.png)

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05179.png)

![10](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051710.png)

![11](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051711.png)

![12](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051712.png)

![13](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051713.png)

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051715.png)

![16](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051716.png)



## 1.4 MySQL登录

windows开始菜单中，出现MySQL数据库的命令行

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051717.png)

![18](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051718.png)

![19](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051719.png)



- 直接cmd命令行登录

mysql -uroot -p 回车，输入你的密码

执行上面的命令，配置Windows环境变量

不配置环境变量也可以，直接开始菜单，点击MySQL命令行也可以



![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051720.png)

#  第二章 SQL语句

## 2.1 DBMS

DBMS：DataBase Manager System 数据库管理系统，是一个软件，管理数据库的软件。不需要单独安装，安装MySQL的时候直接就安装进去了。我们用户对MySQL数据库的任何操作，都是由DBMS来完成的（库管角色）。

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051721.png)

## 2.2 SQL语句

SQL语句是数据库的操作指令，对数据库的任何操作都要使用SQL语言来实现

SQL：Structured Query Language 结构化查询语言，本身SQL语言是一个标准，目前为止所有的数据库都支持SQL语言标准。但是每个数据库又有一些差异

- Data Definition Language (DDL数据定义语言) 如：操作数据库，操作表
- **Data  Manipulation Language**(DML数据操纵语言)，如：对表中的记录操作增删改
- **Data Query Language**(DQL 数据查询语言)，如：对表中数据的查询操作
- Data Control Language(DCL 数据控制语言)，如：对用户权限的设置

## 2.3 SQL语言的语法（语法）

- 不区分大小写
  - 关键字不区分 
  - 名字，数据库名字，数据表名字，表中列名
    - 名字：在Windows里面不区分 
    - 名字：在Linux系统中就区分大小写
- 命名采用Java标识符
- SQL语言没有注释
- 语言是分号结尾
- SQL语言中，数字可以不写引号，其他的数据类型写引号，写单引号

## 2.4 数据库和数据表

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051722.png)

## 2.5 数据库创建

- 创建数据库关键字 : create database 

  - 格式：create database 数据库名

  ```sql
   create database mydb1;
  ```

  - 格式：create database character set 编码表名

  ```sql
  create database mydb2 character set utf8; # 不选择utf8方式
  ```

- 查看当前机器上的所有数据库：show

  - 格式：show databases

- 查看数据库的创建结构

  - 格式：show create database 数据库名

- 创建数据库的时候：MySQL版本问题

- 5.x：

  - 5.0和5.5 - 创建数据库的时候，使用哪个编码表，安装数据库的时候决定的（拉丁文）
  - 5.0和5.5 - 安装MySQL的时候，是可以选择使用默认编码表的，不选择就是拉丁文
  - 5.7 - 安装MySQL的时候，没有选择编码表的过程，和MySQL8版本是一样的，默认的编码表是utf8mb4

- 8.x 默认的编码表是utf8mb4

> 如果同学版本：5.x 版本：创建数据库 create database mydb2 character set utf8mb4;
>
> 版本：8.x版本 create database mydb2

## 2.6 删除数据库

MySQL安装目录：ProgramData目录，MySQL目录，存储数据库的数据

```sql
drop database 数据库名   # 没有是否确认删除，命令执行了，就已经删除了
```

## 2.7 数据库的图形界面

在MySQL命令行中写程序不方便，使用图形界面操作数据库，常见到的图形界面NativeCat，SqlYog，MySQLFront，都是收费的

使用SqlYog

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051726.png)

![27](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051727.png)

![28](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051728.png)

![29](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051729.png)

> Error 图形化工具，连接MySQL服务器报错，MySQL8版本，加密方式变了，如果同学们的机器上安装的是MySQL5.x版本，不存在报错了
>
> 安装MySQL8.x同学们，修改密码的加密方式：
>
> **ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你自己的密码';**

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051730.png)

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051731.png)

# 第三章 数据表操作

## 3.1 创建数据表（非常重要）

数据库中的数据表，是最终存储数据的地方，本质上是一个二维表格（有行，有列，类似excel）

- 创建数据表关键字 ：create table
- 创建数据表的格式：

```sql
create table 表名(
   列名1 数据类型 [约束],
   列名2 数据类型 [约束],
   列名3 数据类型 [约束]
)
```

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051723.png)

> 数字数据类型：int

![24](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051724.png)

> 日期类型：datetime类型，对应java中就是Date类型

![25](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/051725.png)

> 数据库中没有字符和字符串的概念，数据库中是字符和可变字符
>
> 可变字符类型：char，varchar，对应java中的String类型

- 创建学生信息表

```sql
# 创建学生信息表，学生编号，学生姓名，学生班级，年龄
# 数据库的SQL语句，可以使用关键字作为名字
# 引起误读，可以添加反引号 ``
CREATE TABLE student(
    id INT,
    `name` VARCHAR(20),
    class VARCHAR(50),
    age INT
);
```

## 3.2 修改表结构

在已经创建好的数据表上，对表的列进行修改：关键字 alter table

> 修改表结构的操作：开发中别做

- 添加列

```sql
-- 修改表    表名    添加   列名    数据类型
ALTER TABLE student ADD address VARCHAR(50)
```

- 修改列名

```sql
-- 修改表    表名     改变   旧列名     新列名    数据类型
ALTER TABLE student CHANGE address newaddress VARCHAR(50)
```

- 修改列数据类型

```sql
-- 修改表    表名     改变     列名      新数据类型
ALTER TABLE student MODIFY newaddress INT
```

- 删除列

```sql
-- 修改表     表名    丢弃     列名   
ALTER TABLE student DROP newaddress

```

> 保存机制：保存的不是数据库，保存的是代码  Ctrl+s

## 3.3 插入数据（非常重要）

数据操作CRUD：C create，R read，U update，D delete

- 向数据表插入数据关键字：insert into values

  - 插入数据格式一：

  ```properties
  insert [into] 表名 (列名1,列名2,列名3) values (值1,值2,值3)
  
  ```

  ```sql
  INSERT INTO student (id,NAME,class,age) VALUES (1,'张三','一年级一班',6)
  
  ```

  - 插入数据格式二：

  ```properties
  insert [into] 表名 values (全列值)
  
  ```

  ```sql
  INSERT INTO student VALUES(2,'李四','二年级一班',7)
  
  ```

  - 插入数据格式三：

  ```sql
  批量添加
  insert [into] 表名 values (全列值),(全列值),(全列值)
  
  ```

  ```sql
  INSERT INTO student VALUES(3,'刘德华','年级未知',66),
  (4,'张学友','一样未知',68),(5,'任贤齐','也不清楚',55)
  
  INSERT INTO student (id,NAME)VALUES(6,'中岛美雪'),
  (7,'邓丽君')
  
  ```

  ## 3.3 更新和删除数据（非常重要）

- 更新数据：关键字 update set where

  - 语法格式

    ```properties
    update 表名 set 列=值,列=值 where 条件
    
    ```

    ```sql
    UPDATE student SET NAME = '李师师', class = '特殊班级', age = '1500' WHERE id = 2
    
    ```

- 删除数据：关键字 delete from where

  - 语法格式

  ```properties
  delete from 表名 where 条件
  
  ```

  ```sql
  DELETE FROM student WHERE id = 1
  
  ```

> 删除数据表： drop table 表名

## 3.4 数据基本查询（非常重要）

- 查询数据关键字：select from 

```sql
# product 产品表，商品表
create table product(
    # 列名pid primary key主键（数据唯一）  auto_increment（自动增长 i++）
	pid int primary key auto_increment,
    # 商品名
	pname varchar(40),
    # 商品价格
	price double,
    # 库存数
	num int
);
insert into product values(null,'苹果电脑',18000.0,10);
insert into product values(null,'华为5G手机',30000,20);
insert into product values(null,'小米手机',1800,30);
insert into product values(null,'iPhonex',8000,10);
insert into product values(null,'iPhone7',6000,200);
insert into product values(null,'iPhone6s',4000,1000);
insert into product values(null,'iPhone6',3500,100);
insert into product values(null,'iPhone5s',3000,100);

insert into product values(null,'方便面',4.5,1000);
insert into product values(null,'咖啡',11,200); 
insert into product values(null,'矿泉水',3,500);

```

- 全部数据查询

```properties
select * from 表名    # * 代表所有的列，效率很低，实际开发不让写 * 
select 列名 from 表名

```

```sql
-- 查询全表数据
SELECT * FROM product

-- 查询数据，指定列
SELECT pid,pname,price,num FROM product

```

- 去掉重复数据：函数distinct(列名)


```sql
SELECT  DISTINCT(num)  FROM product

```

- 计算查询

```sql
-- 查询数据表，所有的商品价格，上涨20%
SELECT pid,pname,price + price * 0.2 ,num FROM product

```

> 只要是select查询语句，无论怎么查询，原始数据不会改变

- 重命名查询：查询后的列，可以重命名，关键字 as

```sql
-- 查询数据表，所有的商品价格，上涨20%，修改列名
SELECT pid,pname,price + price * 0.2 AS 'price',num FROM product

-- 查询数据表，所有的商品价格，上涨20%，修改列名，简单写法
SELECT pid,pname,price + price * 0.2 price ,num FROM product

```

## 3.5 MySQL中的运算符

- 比较运算符
  - `=,<,>,<=,>=,<>不等于  != 不等于`

- 逻辑运算符
  - `&& ,||, !` Java语言写法
  - `and, or, not`  SQL语句写法

- 区间范围运算符
  - `between and ` 数据的区间运算符
  - `between 1 and 10` 查询数据在1-10之间，包含开头，包含结尾
- 包含运算符
  - `in` 包含的意思
  - `in(1,6,2,5,7)` 查询数据，是1,6,5,2,7 其中一个就可以
  - `not in` 不包含的意思

- 模糊运算符：like

## 3.6 条件查询（非常重要）

- 查询商品价格是3000元

```sql
SELECT * FROM product WHERE price = 3000

```

- 查询商品价格大于3000元

```sql
SELECT * FROM product WHERE price > 3000

```

- 查询商品价格不是3000元

```sql
SELECT * FROM product WHERE price <> 3000 

```

- 查询商品价格在3000元到8000元之间的


```sql
-- 查询商品价格在3000元到8000元之间的
SELECT * FROM product WHERE price >= 3000 AND price <= 8000 
-- 需求可以使用between
SELECT * FROM product WHERE price BETWEEN 3000 AND 8000 
-- between 查询 数据写法：小数在前面，大数在后面
-- 如果查询的是日期区间：小的日期在前，大日期在后面 between '2022-1-1' and '2022-2-2'
SELECT * FROM product WHERE price BETWEEN 8000 AND 3000 -- 查询不到数据

```

- 查询商品名字查询有字母p的


```sql
-- 查询商品名字查询有字母p的，模糊查询like运算符
-- 模糊查询要使用通配符的 % 通配任意字符
-- 数据只记得一部分
SELECT * FROM product WHERE pname LIKE '%p%'
SELECT * FROM product WHERE pname LIKE '%机%'

```

- 查询商品名字是IPh开头

```sql
SELECT * FROM product WHERE pname LIKE 'iph%'

```

> like模糊查询：会造成数据表索引失效

- 范围查询，查询商品的pid号，1,5,7,9

```sql
-- 范围查询，查询商品的pid号，是1,5,7,9 其中一个就可以
SELECT * FROM product WHERE pid = 1 OR pid = 5 OR pid = 7 OR pid = 9
-- 范围查询，推荐使用in 查询
SELECT * FROM product WHERE pid IN(1,7,5,9)
-- 范围查询，查询商品的pid号，不是1,5,7,9 
SELECT * FROM product WHERE pid NOT IN(1,7,5,9)

```

- 查询商品数据，商品库存数量是null


```sql
-- 查询商品数据，商品库存数量是null
-- 查询null值，关键字 is
SELECT * FROM product WHERE num IS NULL
-- 如果查询数据是非null
SELECT * FROM product WHERE num  IS NOT NULL

```

## 3.7 排序查询

- 排序查询关键字 order by
  - 升序关键字 asc 默认值可不写
  - 降序关键字 desc 就必须写

- 查询商品表，按照价格升序

```sql
SELECT * FROM product ORDER BY price  

```

- 查询商品表，按照价格降序

```sql
SELECT * FROM product ORDER BY price  desc

```

- 查询商品表，找出所有的IPhone手机，按照价格降序

```sql
SELECT * FROM product WHERE pname LIKE 'Iph%' ORDER BY price DESC

```

## 3.8 聚合函数

- sum(列名) 对这个列值求和
- max(列名) 找出列的最大值
- min(列名) 找出列的最小值
- avg(列名) 计算列的平均数
- count(列名) 统计数据有多少行

```sql
-- 所有商品价格求和，不参与计算
SELECT SUM(price) FROM product

-- 所有商品价格平均值，遇到null，不参与计算
SELECT AVG(price) FROM product

-- 查询商品表，共有几行数据
SELECT COUNT(1) FROM product

```

