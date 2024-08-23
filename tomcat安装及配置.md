# **tomcat安装及环境配置**

# 前提条件要有Java环境

## **一、tomcat下载**

官网地址：[Apache Tomcat® - Welcome!](https://tomcat.apache.org/)

选择Tomcat8版本（自己随意，这里我是选择的8版本）

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/030.jpg)

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/031.jpg)

## 二、Tomcat配置环境变量

首先右击此电脑，属性，打开高级系统设置：

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/014.png)

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/015.png)

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/016.png)

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/034.jpg)

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/035.jpg)

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/036.jpg)

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/037.jpg)

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/038.jpg)

## **三、**验证Tomcat配置是否成功

首先win+R输入cmd回车，然后再DOS窗口输入startup.bat回车。

之后会出现Tomcat启动窗口。

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/039.jpg)

代表配置成功。

如果出现报错或者一闪而过，可能是端口号被占用。Tomcat默认窗口时8080。

可以尝试重启电脑，再启动Tomcat试试。不行的话，就修改Tomcat端口号。

如果配置成功，打开浏览器，输入http://localhost:8080/

（刚才那个黑窗口一定不能关闭，否则肯定打不开这个页面）

如果出现如下图，则表示成功。

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/040.jpg)

## 四、在 Eclipse 中配置 Tomcat

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/041.jpg)

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/042.jpg)

指定服务器路径和部署路径

![clipboard.png](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/Java/043.jpg)
