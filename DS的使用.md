##### 笔记4

1. ADS层可能会出现空文件，插入ADS表中数据的时候使用的是一个 insert-overrite，底层调用的是spark或者mr引擎写入文件， 写入文件的个数有最后一个task的并行度决定， 如果只写入一条数据，开启两个并行度，就会生成空文件。
2. `set nu` ： 设置行号
3. `/etc/ssh/`
4. `echo $JAVA_HOME`
5. `vim 中的 shift+zz`: 保存并
6. `which stop-all.sh`: 检查命令的具体位置
7. `vim 中使用 / `： 进行搜索
8. DataX配置文件将数据有 ADS层导入到MySQL中的自己新建的数据表中，方便之后的全流程调度工具查询展示数据，提高查询效率。
9.  DolphinScheduler 的使用

