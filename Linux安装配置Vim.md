##### 笔记

Linux操作系统

- 今日内容
  - 理解Linux内核版和发行版
  - 独立安装Linux发行版
  - 熟练使用vim编辑器
  - 配置系统IP地址
  - 防火墙设置
  - 远程连接资源

# 第一章 Linux操作系统介绍

## 1.1 Linux系统的由来

  美国贝尔实验室（ 美国的新泽西州），1969年退出了非常著名的操作系统Unix。就是免费而且是开源的。1970贝尔实验室又推出了经久不衰的语言C，C语言从新开发了Unix系统的。不是完美的，出现错误，漏洞，功能不完善。社会上的很多公司或者个人，都对Unix做更新，修补。。。

  贝尔实验室后悔，Unix系统是免费的，但是打算要收费了。贝尔实验室和很多三方公司打官司。

  李纳斯托瓦斯，是芬兰赫尔辛基大学的学生，在校期间学习的就是软件编程，学习Unix操作系统，教授担心版权问题，修改了Unix系统的源码，教授学生！李纳斯认为教授改版的系统不行，自己借助于Unix系统的思想，自己在宿舍里面进行改进，写出了Linux操作系统的内核（操作系统的内核就是和硬件交互的），发布到学校的校园网上。

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05231.png)

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05232.jpg)

## 1.2 Linux系统内核版本和发行版本

- 内核版本
  - 操作内核都是和硬件交互（控制硬件）
  - 李纳斯领导的开发小组，进行维护更新

- 发行版本
  - 在操作系统内核之上，建立起来的一个软件
  - 为用户使用的，内核增加了功能：学习使用的发行版Centos森头

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05233.png)

## 1.3 Linux系统和Windows系统的差异

- 应用软件层面：
  - Windows系统：应用软件非常的多
  - Linux系统：应用软件非常少
- 用户的易用度：
  - Windows系统：用户基本不用学
  - Linux系统：不学习基本不会用
- 安全稳定：
  - Windows系统：不安全，不稳定
  - Linux：安全，稳定
- 应用层面：
  - Windows系统：家用机，办公
  - Linxu系统：服务器市场

## 1.4 虚拟机

- 安装Linux操作系统的发行版的解决办法
  - 格式化Windows，安装Linux
  - 双系统，C盘安装Windows，D盘安装Linux
  - 再买个笔记本，专门安装Linux系统
  - 利用虚拟机实现安装Linux系统
    - 虚拟机--就是一个软件
    - 软件可以在你自己的真机里面，虚拟出另一台计算机，安装Linux
    - `VMware-workstation-full` 虚拟机的软件

> @所有人：打开Windows的任务管理器，上方有个选项卡，性能选项卡，性能里面看CPU

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05234.png)



写禁用的同学，需要修改bios（Basic Input Output System）,bios修改需要再开机之间修改

百度搜索：（联想，戴尔，惠普，外星人） 机器品牌开启虚拟化

9:55回来！，禁用的同学，按照网上的教程，开启你的虚拟化！！

# 第二章 安装Centos发行版

## 2.1 安装虚拟机软件

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05235.png)

![6](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05236.png)

![7](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05237.png)

![8](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05238.png)

![9](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05239.png)

## 2.2 虚拟机软件虚拟出一台计算机

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052310.png)

![11](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052311.png)

![12](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052312.png)

![13](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052313.png)

![14](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052314.png)

![15](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052315.png)

![16](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052316.png)

![17](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052317.png)

![18](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052318.png)

![19](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052319.png)

![20](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052320.png)

![21](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052321.png)

![22](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052322.png)

![23](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052323.png)

![24](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052324.png)

![25](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052325.png)

![26](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052326.png)

![27](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052327.png)

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052328.png)

![29](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052329.png)

> 到这里完成虚拟任务，相当于买了一台新机

## 2.3 安装Linux发行版Centos7

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052330.png)

![31](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052331.png)

![32](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052332.png)

![33](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052333.png)

![34](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052334.png)

![35](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052335.png)

![36](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052336.png)

![37](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052337.png)

![38](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052338.png)

![39](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052339.png)

![40](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052340.png)

![41](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052341.png)

![42](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052342.png)

![43](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052343.png)

![44](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052344.png)

![45](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052345.png)

![46](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052346.png)

![47](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052347.png)

![48](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052348.png)

![49](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052349.png)

![50](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052350.png)

![51](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052351.png)

![52](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052352.png)

![53](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052353.png)

![54](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052354.png)

![55](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052355.png)

![56](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052356.png)

![57](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052357.png)

![58](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052358.png)

![59](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052359.png)

![60](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052360.png)

![61](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052361.png)

![62](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052362.png)

![63](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052363.png)

## 2.4 Linux系统的目录结构

Windows操作系统按照磁盘分区来进行，但是Linux系统没有区分，一切都是文件，任何数据形式都是文件形态，Linux目录的结构

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05234.jpg)

- Linux系统的目录组成
  - / 根目录，整个操作系统的顶级目录
  - bin：二进制的意思，存储Linux系统中的命令
  - boot：操作系统的启动引导区
  - **etc：非常重要的目录，存储的都是配置文件，配置JAVA_HOME，就在etc目录下配置**
  - **home：存储登录操作系统的账户**
  - usr：Unix系统共享目录，目录下的任何内容，无论是什么账户登录的都可以使用
  - **root：超级管理的专属目录**

# 第三章 VIM编辑器（非常重要）

## 3.1 VIM编辑器

编辑文本的工具，可以VIM编辑器编辑我们使用的文本文档。以后我们要部署开发环境，Java环境和Hadoop环境，做配置文件，vim编辑器编辑我们的配置文件，键值对形态，xml形态。

- 编辑器的工作模式
  - 一般模式，也称为命令模式
  - 编辑模式
  - 底行模式

> vim编辑文件命令：vim 文件名

## 3.2 VIM编辑器一般工作模式

| 语法          | 功能描述                      |
| ------------- | ----------------------------- |
| **yy**        | 复制光标当前一行              |
| **y数字y**    | 复制一段（从第几行到第几行）  |
| **p**         | 箭头移动到目的行粘贴          |
| u             | 撤销上一步                    |
| **dd**        | 删除光标当前行                |
| **d数字d**    | 删除光标（含）后多少行        |
| x             | 剪切一个字母，相当于del       |
| X             | 剪切一个字母，相当于Backspace |
| yw            | 复制一个词                    |
| dw            | 删除一个词                    |
| shift+6（^）  | 移动到行头                    |
| shift+4 （$） | 移动到行尾                    |
| 1+shift+g     | 移动到页头，数字              |
| shift+g       | 移动到页尾                    |
| 数字+shift+g  | 移动到目标行                  |

## 3.3 编辑模式

编辑文本文件，命令 vim 文件名，如果文件不存在，编辑保存就创建了

- 编辑模式使用步骤
  - vim 文件名，回车，进入到vim编辑器的命令模式
  - 输入 a , i , o 或者 insert键，进入到编辑模式，可以随意编辑文件
  - 编辑完成，按下esc键，进入到命令模式
  - 如果再次按下 a , i , o 或者 insert键，进入到编辑模式，可以随意编辑文件
  - 命令模式下按 : 进入到底行模式

## 3.4 底行模式

输入wq 回车，写入并保存

输入q! 回车，退出不保存



# 第四章 虚拟机的网络环境位置（非常重要）

## 4.1 配置网络环境目的

配置网络环境，目的是真机（Windows机）和虚拟机的网络要联通，虚拟机之间的网络也要联通。Hadoop课程需要做服务器集群

网络环境配置：VMWare软件上，编辑菜单，虚拟网络编辑器在这里进行配置

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052364.png)

是一个虚拟网卡（net8），作用就是真机和虚拟机的连接机制

## 4.2 虚拟网络环境配置

DHCP：动态的IP地址分配规则

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052365.png)

![66](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052366.png)

![67](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052367.png)

![68](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052368.png)

> IP地址设置问题：IP地址最大值是255。以后我们Hadoop软件环境要服务器集群，多台机器一起执行的，多个机器一起，好管理

## 4.4 Linux的IP地址配置问题

Linux系统中，鼠标右键打开终端：输入命令 ifconfig 看到当前机器的IP地址，IP地址是动态分配的，有可能会改变。固定IP地址，不能改变！

修改Linux系统中的配置文件：/etc/sysconfig/network-scripts/ifcfg-ens33 要修改的配置文件

```properties
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
# IP分配规则，改为static
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=eb0b0cd9-df73-4c6a-9177-85e255c38e39
DEVICE=ens33
ONBOOT=yes
# 设置自己的IP地址
IPADDR=192.168.226.100
# 网关配置
GATEWAY=192.168.226.2
# 域名解析服务器，和网关一样
DNS1=192.168.226.2

```

> 修改完毕后，重启网卡：命令 systemctl restart network

## 4.6 配置hosts文件（Windows和Linux都要配）

- windows系统：`C:\Windows\System32\drivers\etc\hosts`文件
- Linux系统：`/etc下`
- hosts：配置的是IP地址和机器名的映射关系
  - 机器要是通过网络，连接另外一台机器，先搜索hosts文件，找文件中的对应关系

- 配置文件的目的：直接写机器名，就可以找到IP地址了

```properties
192.168.226.100 hadoop100
```

- **修改主机名**

```
vim hostname
// etc/hostname
// 重启才生效（reboot）
```



## 4.7 防火墙配置

### systemctl （）

- **基本语法**

​       systemctl  start | stop | restart | status      服务名

- **案例实操**

（1）查看防火墙服务的状态

[root@hadoop100 桌面]# systemctl status firewalld

（2）停止防火墙服务

[root@hadoop100 桌面]# systemctl stop firewalld

systemctl disable firewalld.service     // 持久性停止防火墙

（3）启动防火墙服务

[root@hadoop100 桌面]# systemctl start firewalld

（4）重启防火墙服务

[root@hadoop100 桌面]# systemctl restart firewalld  

## 4.8 systemctl 设置后台服务的自启配置

systemctl list-unit-files         （功能描述：查看服务开机启动状态）

[root@hadoop100 桌面]# systemctl enable firewalld.service 

[root@hadoop100 桌面]# systemctl disable firewalld.service 

# 第五章 远程登录



提升权限   PASSWD：（ALL）

###### 3.4 权限提升

- sudu 提升权限
- root账户修改配置文件 /etc/sudoers

```properties
zhangsan ALL=(ALL)     NOPASSWD:ALL
```
