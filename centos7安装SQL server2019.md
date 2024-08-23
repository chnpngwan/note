# centos7安装SQL server2019

1、yum安装

```shell
# 关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service
# 配置 yum 源
yum -y install wget
wget -O /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2019.repo
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

| 配置选项                                    | 描述                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| Agent 启用                                  | 启用或禁用SQL Server代理                                     |
| Collation 设置                              | 设置新的排序规则                                             |
| Customer feedback 选择                      | 选择是否发送反馈给微软                                       |
| Database Mail Profile 设置                  | 设置默认的数据库邮件配置                                     |
| Default data directory 修改                 | 修改新的数据文件的默认路径                                   |
| Default log directory 修改                  | 修改新的日志文件的默认路径                                   |
| Default master database file directory 修改 | 修改master数据库的默认路径                                   |
| Default master database file name 修改      | 修改master数据库文件的名称                                   |
| Default dump directory 修改                 | 修改新的内存DUMP和其他排错文件的默认路径                     |
| Default error log directory 修改            | 修改新的SQL Server错误日志文件、默认跟踪、系统健康会话扩展事件和Hekaton会话扩展事件文件 |
| Default backup directory 修改               | 修改新的备份文件的默认路径                                   |
| Dump type 选择                              | 选择内存DUMP文件收集的DUMP类型                               |
| High availability 启用                      | 启用可用性组                                                 |
| Local Audit directory 配置                  | 配置一个添加本地审核文件的目录                               |
| Locale 配置                                 | 配置SQL Server使用的地区（配置语言环境）                     |
| Memory limit 配置                           | 配置SQL Server内存限制                                       |
| TCP port 修改                               | 修改SQL Server连接监听的端口                                 |
| TLS 配置                                    | 配置TLS（Transport Level Security）                          |
| Traceflags 设置                             | 设置服务使用的跟踪标识                                       |

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

