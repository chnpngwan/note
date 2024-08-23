# centos7安装MySQL

## 一、环境准备

环境：

| centos | MySQL  |
| :----: | :----: |
|  7.9   | 8.0.33 |

## 二、开始安装

### 1、关闭selinux，放行3306端口，修改hosts文件

```shell
#关闭selinux
sed -i 's/SELINUX\=enforcing/SELINUX\=disabled/g' /etc/selinux/config

#放行3306端口
firewall-cmd --add-port=3306/tcp --permanent

#重启防火墙更新配置
firewall-cmd --reload

#验证
firewall-cmd --list-all
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_1.png)

```shell
#修改hosts文件
echo '172.20.38.189 databases' >> /etc/hosts
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_2.png)

### 2、删除MariaDB

```
yum -y remove mariadb-libs.x86_64
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_3.png)

### 3、安装libaio-devel

```
yum install libaio-devel -y
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_4.png)

### 4、删除MySQL的my.cnf配置

```
rm -rf /etc/my.cnf
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_5.png)

### 5、创建用户组和用户

```
id mysql

groupadd -g 54321 mysql

useradd -r -g mysql -s /bin/false -u 54321 mysql

#验证用户和用户组
cat /etc/group | grep mysql
cat /etc/passwd | grep mysql
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_6.png)

### 6、下载、解压缩、创建软链接

```
下载：
	先安装wget：yum -y install wget
	下载MySQL：wget https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.33-linux-glibc2.12-x86_64.tar
解压缩：
	tar -xvf mysql-8.0.33-linux-glibc2.12-x86_64.tar
	tar -xvf mysql-8.0.33-linux-glibc2.12-x86_64.tar.xz -C /usr/local/src/
创建软链接
	ln -s /usr/local/src/mysql-8.0.33-linux-glibc2.12-x86_64/ /usr/local/mysql
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_7.png)

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_8.png)

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_9.png)

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_10.png)

### 7、创建数据目录

```
mkdir -p /data/mysql/data
chown -R mysql:mysql /data/mysql/data
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_11.png)

### 8、参数配置

```
vim /etc/my.cnf


[client]

socket=/data/mysql/data/mysql.sock
[mysql]
prompt="\\u@\\h [\\d]> "

[mysqld]
basedir=/usr/local/mysql
datadir=/data/mysql/data
user=mysql
port=3306
pid-file=/data/mysql/data/mysql.pid
socket=/data/mysql/data/mysql.sock
log-error=/data/mysql/data/mysql.err
log-timestamps=system
 
mysqlx_port=33060
mysqlx_socket=/data/mysql/data/mysqlx.sock
 
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_12.png)

### 9、初始化数据库

```
默认使用/etc/my.cnf参数文件
/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --user=mysql --initialize
 
看到日志输出显示临时密码说明初始化成功
[root@databases src]# more /data/mysql/data/mysql.err | grep temporary
2024-05-15T03:35:41.955762-04:00 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: !ar(xlp#2)P,
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_13.png)

### 10、启动数据库

```
[root@databases src]# /usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf &
[1] 8715
[root@databases src]# 2024-05-15T07:38:09.521126Z mysqld_safe Logging to '/data/mysql/data/mysql.err'.
2024-05-15T07:38:09.544846Z mysqld_safe Starting mysqld daemon with databases from /data/mysql/data

[root@databases src]# ps -ef |grep mysql | grep -v grep
root      8715  8479  0 03:38 pts/0    00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf
mysql     8901  8715  4 03:38 pts/0    00:00:00 /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/data/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/data/mysql.err --pid-file=/data/mysql/data/mysql.pid --socket=/data/mysql/data/mysql.sock --port=3306

```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_14.png)

### 11、配置环境变量

```
[root@databases ~]# vim .bash_profile

# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH


export MYSQL_HOME=/usr/local/mysql
export PATH=$PATH:$MYSQL_HOME/bin

[root@databases ~]# source .bash_profile
```

### 12、登录测试

```
[root@databases ~]# mysql -uroot -p -S /data/mysql/3306/data/mysql.sock
Enter password: 
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/data/mysql/3306/data/mysql.sock' (2)
[root@databases ~]# mysql -uroot -p -S /data/mysql/data/mysql.sock
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.33

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

root@localhost [(none)]> select * from mysql.user\G
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
root@localhost [(none)]> alter user 'root'@'localhost' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

root@localhost [(none)]> 

```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_15.png)

### 13、使用配置文件启动

```
 
修改配置文件/usr/local/mysql/support-files/mysql.server
 
通过查看配置文件/usr/local/mysql/support-files/mysql.server
basedir默认/usr/local/mysql
datadir默认/usr/local/mysql/data，修改为/data/mysql/3306/data
配置文件默认是/etc/my.cnf，不需要修改
 
 
经过测试datadir不需要修改，因为配置文件默认是/etc/my.cnf，参数中指定了datadir的具体值
 
[root@databases ~]#  vim /usr/local/mysql/support-files/mysql.server
 
 
basedir=/usr/local/mysql
datadir=/data/mysql/data
 
if test -z "$basedir"
then
  basedir=/usr/local/mysql
  bindir=/usr/local/mysql/bin
  if test -z "$datadir"
  then
    datadir=/data/mysql/data
  fi
  sbindir=/usr/local/mysql/bin
  libexecdir=/usr/local/mysql/bin
else
  bindir="$basedir/bin"
  if test -z "$datadir"
  then
    datadir="$basedir/data"
  fi
  sbindir="$basedir/sbin"
  libexecdir="$basedir/libexec"
fi
 
使用配置文件/usr/local/mysql/support-files/mysql.server启动
 
[root@databases ~]#  /usr/local/mysql/support-files/mysql.server start
Starting MySQL................ SUCCESS!
 
[root@databases ~]#  /usr/local/mysql/support-files/mysql.server status
 SUCCESS! MySQL running (102015)
 
[root@databases ~]#  /usr/local/mysql/support-files/mysql.server stop
Shutting down MySQL... SUCCESS! 
 
```

![833](https://gitee.com/chnpngwng/typora-image/raw/master/assets/mysql/833_16.png)

