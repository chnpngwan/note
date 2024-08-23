关闭防火墙

```shell
systemctl stop firewalld.service
systemctl disable firewalld.service
```

# docker

1. yum包更新到最新

```shell
yum update
```

2. 安装需要的软件包，yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

3. 设置yum源

```shel
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

4. 安装docker，出现输入的界面都按 y

```shell
yum install -y docker-ce
```

5. 查看docker版本，验证是否成功

```shell
docker -v
```

6.配置镜像加速

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://eob5atc6.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

7.docker服务

- 启动docker 服务：

  - ```shell
    systemctl start docker
    ```

- 停止docker 服务：

  - ```shell
    systemctl stop docker
    ```

- 重启docker 服务：

  - ```shell
    systemctl restart docker
    ```

- 查看docker 服务状态：

  - ```shell
    systemctl status docker
    ```

- 设置开机启动docker：

  - ```shell
    systemctl enable docker
    ```

# dockerUI

```shell
docker run -d --name=dockerui -v /var/run/docker.sock:/var/run/docker.sock -p 8999:8999 jonnyan404/docker-ui 
```

# Redis

1.拉取redis镜像

```shell
docker pull redis:latest
```

2.运行redis镜像

```shell
docker run -itd --name redis-test -p 6379:6379 redis
```

3.进入redis验证

```shell
docker exec -it redis-test /bin/bash
```

# SQL server

1.拉取SQL server镜像

```shell
docker pull mcr.microsoft.com/mssql/server:2022-latest
```

2.运行SQL server镜像

```shell
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=abc123!@#" -e "TZ=Asia/Shanghai" -p 1433:1433 --name sqlserver2022 -d mcr.microsoft.com/mssql/server:2022-latest
```

3.进入SQL server验证

```shell
docker exec -it sqlserver2022 /bin/bash

/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'abc123!@#'
select name from sys.Databases
```

# Rabbitmq

1.拉取Rabbitmq镜像

```shell
docker pull rabbitmq
```

2.运行Rabbitmq镜像

```shell
docker run -d --hostname rabbitmq --name rabbitmq -p 15672:15672 -p 5673:5672 rabbitmq
```

3.进入Rabbitmq验证

```shell
docker exec -it 容器id /bin/bash

rabbitmq-plugins enable rabbitmq_management
http://192.168.100.10:15672 docker pull mysql:8.0.20
```

# MySQL

1.拉取mysql镜像

```shell
docker pull mysql:8.0.20
```

2.运行mysql镜像

```shell
docker run -p 3306:3306 --name mysql820 -e MYSQL_ROOT_PASSWORD=123456  -d mysql:8.0.20
```

3.进入mysql验证

```shell
docker exec -it 容器id /bin/bash

mysql -u root -p123456
grant all PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;
ALTER user 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
FLUSH PRIVILEGES;
```

# Postgresql
1.拉取Postgresql镜像

```shell
docker pull postgres
```

2.运行Postgresql镜像

```shell
docker run -it --name postgresql --restart always -e POSTGRES_PASSWORD='123456' -e ALLOW_IP_RANGE=0.0.0.0/0 -v /root/postgres/data:/var/lib/postgresql -p 55433:5432 -d postgres
```

3.进入Postgresql验证

```shell
docker exec -it 容器id /bin/bash

su postgres
docker cp postgresql:/var/lib/postgresql/data/pg_hba.conf /root  
docker cp /root/pg_hba.conf postgres:/var/lib/postgresql/data
docker cp postgresql:/var/lib/postgresql/data/postgresql.conf /root
```



# centos7安装SQL server2019

1、yum安装

```shell
# 配置 yum 源
yum -y install wget
wget -O /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2019.repo

wget -O /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2017.repo
yum clean all
yum makecache fast

# 安装 SQLserver
yum install mssql-server
# 初始化配置 SQLserver
/opt/mssql/bin/mssql-conf setup
```

![sqlserver](https://gitee.com/chnpngwng/typora-image/raw/master/assets/SQL server/SQL server1.png)

![sqlserver](https://gitee.com/chnpngwng/typora-image/raw/master/assets/SQL server/SQL server2.png)

2、启动 SQLserver

```shell
systemctl start mssql-server
systemctl enable mssql-server

# 验证
systemctl status mssql-server

# SQLserver 的安装路径在/var/opt/mssql，配置文件在 /var/opt/mssql/mssql.conf。
```

3、配置 SQLserver

mssql-conf是在Linux上安装SQL Server 2017后的一个配置脚本。使用命令 /opt/mssql/bin/mssql-conf 可以对 SQLserver 的配置进行修改优化。支持设置以下参数：



示例1：启用 SQLServer 代理

```shell

/opt/mssql/bin/mssql-conf set sqlagent.enabled true

# 需要重启
systemctl restart mssql-server
```

示例2：修改默认数据和日志目录位置

filelocation.defaultdatadir和filelocation.defaultlogdir 设置修改新的数据和日志文件创建的位置。默认路径是：/var/opt/mssql/data。修改到：/home/msdata，操作如下：

```shell

mkdir -p /home/mssql/data
mkdir -p  /home/mssql/log
chown -R mssql:mssql /home/mssql

# 使用mssql-conf脚本执行set命令修改默认数据目录
/opt/mssql/bin/mssql-conf set filelocation.defaultdatadir /home/mssql/data
# 修改日志目录
/opt/mssql/bin/mssql-conf set filelocation.defaultlogdir /home/mssql/log

# 需要重启
systemctl restart mssql-server
```

![SQL server](https://gitee.com/chnpngwng/typora-image/raw/master/assets/SQL server/SQL server3.png)

4、docker方式安装

```shell
docker run -d -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=abc123!@#" -p 1433:1433 -v mssql:/var/opt/mssql --name mssql --hostname mssql -d mcr.microsoft.com/mssql/server
```

5、安装 SQLserver 客户端 sqlcmd

```shell
yum install -y mssql-tools unixODBC-devel

echo "export PATH=$PATH:/opt/mssql-tools/bin" >> /etc/profile
source /etc/profile

# 连接 SQLserver
sqlcmd -S <SQLserver_IP> -U SA -p
```

6、新建数据库

```shell

CREATE DATABASE erpdata
ON PRIMARY
(
    NAME = 'MyDatabase_data',
    FILENAME = '/var/opt/mssql/data/erpdata_data.mdf',
    SIZE = 64MB,
    MAXSIZE = 4GB,
    FILEGROWTH = 10%
)
LOG ON
(
    NAME = 'MyDatabase_log',
    FILENAME = '/var/opt/mssql/data/erpdata_log.ldf',
    SIZE = 32MB,
    MAXSIZE = 1GB,
    FILEGROWTH = 500MB
);
GO
```

