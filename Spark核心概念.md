# Spark核心概念

## 部署模式

### 4大角色

- 资源管理类

	- Master

		- Local: Local进程本身

		- StandAlone: Master进程

		- YARN: ResourceManager

	- Worker

		- Local: Local进程本身

		- StandAlone: Worker进程

		- YARN: NodeManager进程

- 任务运行类

	- Driver

		- Local: 在Local进行内的Driver线程

		- StandAlone: Driver运行在Master进程内

		- YARN

			- Client

				- Driver运行在客户端进程内

			- Cluster

				- Driver运行在容器内, 和ApplicationMaster在同一容器内部

	- Executor

		- Local: 不存在, 由Driver工作干活

		- StandAlone: 运行在Worker进程中

		- YARN: 运行在容器内部, 以进程存在

## RDD

### 概念

- 分布式弹性数据集

	- 分布式

		- RDD的数据是分散在各个分区上的

	- 弹性

		- RDD的分区数量可以动态增减

	- 数据集

		- 存储数据的集合

### 特点

- RDD是有分区的

- 算子作用在每个分区上

- RDD之间是有血缘关系(依赖链条)

- (可选)针对KV型RDD, 可以自定义分区器

	- 默认分区器是Hash分区规则

- (可能)如有可能, RDD加载数据将就近加载, 在数据所在的机器上启动Executor来加载数据

	- 移动数据不如移动计算

- RDD的数据是只读的

	- 对RDD的修改都会产生一个新的RDD

	- 这种叫做RDD的迭代计算

	- 这种一般就是TransFormation算子

### 算子

- Transformation

	- 返回一个新RDD

		- Transformation用于规划RDD的迭代链条(执行蓝图)

- Action

	- 返回值不是RDD

		- Action算子用于将Transformation组成的执行蓝图进行调用开启工作

- 概念

	- 读取数生成RDD

		- 本地转向分布式计算的开端

	- RDD迭代计算

		- 分布式计算

	- Action算子产出计算结果

		- 分布式计算结果汇聚回本地集合

- Action的两个特殊算子

	- 多数的Action算子都是将结果向Driver进行汇聚

	- 唯独

		- foreach

		- saveAsTextFile

		- 这两个算子是直接由RDD的分区(线程)执行

			- 而不经过Driver

### 缓存

- 概念

	- RDD的数据是过程数据, 一旦产生新的RDD后, 老的RDD数据就不存在了

	- 如果你要多次使用这个RDD对象, 为了给你提供数据, 这个RDD会从头开始生成自己

	- 为了解决这个问题, 思路在于, 将RDD的数据保存起来

	- 这就是缓存技术和CheckPoint技术的主要目标

- 将RDD的数据保存到

	- 内存

		- 分区所在Executor的可用内存中

	- 硬盘

		- 分区所在Executor所在服务器的本地硬盘

			- 不支持HDFS

- 数据保存是分散存储的

	- 各个分区自行将分区内的数据写出到内存或硬盘

- 安全性不够安全

	- 分散存储, 分区越多散的越多, 风险越高, 坏1个全部坏掉

	- 如果存放到内存中, 如果内存不够 缓存可能被清理

- 血缘关系

	- 保留

		- 避免丢失后, 可以通过记录的血缘(代码执行流程)重新计算数据

### CheckPoint

- 保存

	- 硬盘

		- 本地硬盘

			- Local模式

		- HDFS硬盘

			- 什么模式都可以

- 数据存储是集中存储

	- 分区多少风险都一样

	- 由于可以放入HDFS所以安全性较高(软件层面)

- 安全性设计为安全的

	- 不保留血缘关系

	- 如果丢失那就真丢咯

### Spark的执行流程

- 大方向

	- 提交代码

	- 生成Driver

		- DAG Scheduler规划逻辑任务

	- 生成Executor(被Driver生成)

	- Driver内的TaskScheduler就开始给Executor安排逻辑的任务并监控它们干活

- 细节(YARN 集群模式为例)

	- 客户端提交代码到YARN

	- YARN构建第一个容器

		- 启动ApplicationMaster

		- ApplicationMaster启动Driver

		- Driver构建DAG Scheduler规划逻辑任务

		- 它俩通讯, AM得知需要几个容器

		- AM找RM要容器

	- ResourceManager分配剩下的容器

	- Driver在剩余的容器内启动Executor

	- Driver的Task Scheduler分配逻辑任务到Executor中运行, 并监控

- 总结

	- 不管是大方向还是具体细节,无非

		- 先有Driver后有Executor

		- DAG调度器规划任务

			- Task调度器分配并管理任务运行

### DAGScheduler 和 Task Scheduler

- 它俩都是Driver的功能

- DAG调度器规划任务

- Task调度器分配并管理任务

### 内存迭代

- Spark的任务都会产生DAG执行图

- DAG会基于分区关系和宽窄依赖划分出不同的阶段

- 这样每一个阶段的内部都是窄依赖了

- 每一个阶段内部都会被划分逻辑的Task

- 每一个逻辑的Task都是一个PipLine(内存迭代管道)

- 每一个Task都有一个具体的线程执行

- 每一个Task横穿多个RDD的分区处理数据

- 这一条处理线, 就是在一个线程的内部走内存计算

- 多个Task就是多个线的内存计算, 也就是并行内存迭代

- PS

	- 这套回答适用于

		- Spark如何做内存计算

		- DAG是什么?有啥用?

		- 阶段划分有啥用?

		- 为什么要划分宽窄依赖

## DataFrame

### 概念

- 分布式弹性数据集

	- 分布式: 多分区

	- 弹性: 分区可以动态增减

	- 数据集

		- 仅存储结构化数据

### 特点

- 分区

- 算子

	- 作用在每个分区上

- 就近加载

- 分区器

### 和RDD的区别

- 存储

	- RDD存储的是数据, 不限类型和结构, 有啥都能存, 只要你能将它序列化成功 就能被RDD处理存储

	- DataFrame存储的是结构化数据, 仅能存储结构化

- 结构

	- RDD的泛型可以是任意

		- 泛型: 存储的对象类型

	- DataFrame的泛型只能是Row对象

- 优化

	- RDD不能被自动优化

		- 因为存储的太宽泛了无法被针对

	- DataFrame可以被自动优化

		- 因为存储结构单一

			- 二维表结构

- 本质

	- RDD是本质

	- DataFrame是表象, 最终要被翻译成RDD干活

## SparkSQL的执行流程

### 提交代码

### Catalyst优化器优化

- 生成原始AST语法树

	- 也就是原始DAG

- 在AST语法树上做标记, 标记元数据信息

- 对AST语法树执行优化

	- 2大方向

		- 谓词下推\断言下推

			- 将逻辑判断提前

			- 提前进行行过滤

		- 列值裁剪

			- 将列的宽度减少

	- 2个大方向本质上就是

		- 提前进行

			- 行过滤

				- 谓词下推

			- 列过滤

				- 列值裁剪

- 生成优化后的执行计划

- 基于执行计划提供的逻辑关系翻译RDD代码

### RDD代码提交运行

## 层次关系

### Spark环境

- Application1

	- Job1

		- Stage1

			- Task1

				- 负责处理RDD1的1个分区

				- RDD2的1个分区

				- RDD3的1个分区

				- RDD N的1个分区

					- 一个RDD的一个分区, 只能被一个Task处理

						-  

					- 1个Task可以处理多个分区, 这些分区必须是不同RDD的

			- Task2

				- 一个Task是一个并行

			- Task3

				- 一个Task是一个PipLine(内存迭代管道)

			- TaskN

				- 一个Task由一个线程所执行

					-  

			- Task是Spark运行中最小单位

				-  

		- Stage2

			- 一个Job会被划分成多个Stage阶段

				-  

		- StageN

			- Stage基于宽窄依赖来划分

				-  

	- Job2

		- 一个Job对应一个Action

	- Job3

		- 一个Job对应一个DAG

			-  

	- JobN

		- 一个Action产生1个Job, 一个Job会有自己所属的DAG执行图

			-  

- Application2

	- 有自己的Driver

- ApplicationN

	-  

### 宽窄依赖

- 窄依赖

	- 父RDD的一个分区, 全部将数据发给一个子RDD的一个分区

- 宽依赖

	- 父RDD的一个分区, 将数据发送给子RDD的多个分区

	- 宽依赖, 一般也被称之为: shuffle

###  

