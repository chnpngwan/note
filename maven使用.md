Maven

- 今日内容
  - 了解Maven软件
  - 三种仓库的概念
  - 创建JavaSE项目
  - 创建JavaWeb项目
  - Maven依赖管理
  - Maven依赖传递
  - 项目生命周期
  - 子父项目

# 第一章 XML语言

## 1.1 什么是XML语言

XML称为扩展标记语言（Extensible  Markup Language）,W3C （万维网联盟组织）在1998年推出的一个标准语言。

HTML超文本标记语言（Hyper Text Markup Language）做网页的语言

XML语言为了保存数据的，在以后会取代Properties配置文件

## 1.2 XML语言的组成部分

### 1.2.1 文档声明

表明当前的文档，是一个扩展标记语言文档！必须写在文档的0行0列位置

```properties
      版本号           文件使用的编码
<?xml version="1.0" encoding="utf-8"?>
```

### 1.2.2 标签

标签又称为标记（Tag）：是XML文档的重要的组成部分，写xml文档，就是在写标签

- 标签的语法：
  - <标签名> </标签名>  开始标签和结束标签
  - 标签名是自定义，不能是数字开头，不能有特殊符号
  - 有开始标签，就要有结束标签
  - 标签里面称为标签体，可以写其他标签，也可以写普通文本
  - 一个XML文档必须有一个根标签

### 1.2.3 标签属性

标签属性是跟随标签的（Attribute）

- 属性的语法：
  - 属性必须写在开始标签里面，不能写在结束标签里面
  - 属性和标签名之间空格分开
  - 写法是键值对 ： 属性名="属性值"
  - 一个标签中，属性的个数不限制，但不能重复
  - 属性之间没有顺序关系

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<person>
  你好xml
    <a>这里是a标签</a>
    <!-- student标签添加属性-->
    <student name="李四" age="20" address="北京市" >Student标签体</student>
</person>

```

> Maven软件，使用xml形式做配置文件，Hadoop(分布式大数据存储)软件使用xml配置文件



# 第二章 Maven介绍

## 2.1 Maven软件介绍

美国Apache软件基金会开发的一个软件，Maven是一款项目管理软件，从项目的构建开始，编译，测试，运行，打包，部署，都交给Maven管理，开发人员的经历放在功能实现上。重复的运用了面向对象的思想管理我们的项目。OO面向对象涉及很多领域（面向对象编程，面向对象项目管理，面向对象的软件测试，面向对象的需求分析）

Maven中有一个非常重要的思想：POM  Project Object Model

## 2.2 Maven仓库（非常重要）

Maven软件帮我们管理需要的jar包，开发一个项目会需要很多jar包，开发人员手动管理jar包不显示，jar包的使用交给Maven处理。还能自动处理jar包和jar包之间的关系。

- Maven仓库就是存储jar包的
  - 本地仓库：Local-repository 就是我们自己机器上的一个文件夹，里面存储的都是jar包
  - 中央仓库：Maven官方的服务器，保存了地球上99%以上的jar包，专门人员来维护，如果我们要用的jar包没有，自动从中央仓库下载
  - 远程仓库：一些公司自己创建服务器，我们可以从远程仓库上下载jar包，阿里云服务器

> 当我们要使用jar包的时候，Maven会找本地仓库中是否有jar，如果没有就会到远程仓库中下载jar包，下载后保存在本地仓库后，以后就不会下载了

## 2.3 Maven的下载和解压

Maven软件下载地址：https://maven.apache.org/

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05211.png)

## 2.4 Maven的配置（非常重要）

- 配置Maven的环境变量：MAVEN_HOME

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05212.png)

![3](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05213.png)

![4](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05214.png)

- Maven的conf目录下，配置settings.xml文件
  - 配置本地仓库文件夹的位置

```xml
<!-- 配置本地仓库路径的，千万不配置错误！！ -->
<localRepository>E:\soft\apache-maven-3.6.1\repository</localRepository>
```

- 配置阿里云服务器地址，不用动了

```xml
<!--
     配置的是远程仓库的地址，配置的阿里云，国内的服务器
     没有的jar包，从远程仓库中下载
     没有这个配置，自动从中央仓库下载jar (国外)
 -->
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

## 2.5 IDEA绑定Maven（非常重要）

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05215.png)

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05216.png)

![7](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05217.png)

![8](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05218.png)

![9](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05219.png)

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052111.png)

```properties
-DarchetypeCatalog=local
-DarchetypeCatalog=internal
两个写法都是可以的
```

# 第三章 创建项目

## 3.1 创建JavaSE项目

使用Maven提供的骨架（就是项目模板，创建好后，项目里面有什么文件）

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052112.png)

![13](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052113.png)

![14](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052114.png)

![15](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052115.png)

![16](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052116.png)

![17](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052117.png)

![18](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052118.png)

- JavaSe项目的目录结构

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052119.png)

## 3.2 pom.xml （非常重要）

maven软件，使用面向对象思想，管理我们创建的项目，每个项目中都有文件pom.xml，是我们项目的核心配置文件。pom:project object model

pom.xml配置的是项目相关信息

```xml
<!-- 根标签，所有的标签都在根里面-->
<project>
      <!-- 公司名 -->
      <groupId>com.atguigu</groupId>
      <!-- 项目名 -->
      <artifactId>javase</artifactId>
      <version>1.0-SNAPSHOT</version>
    
    <properties>
        <!--源码使用编码表-->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!-- 编译版本 -->
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
  </properties>
    
    <!-- 插件区域 -->
    <pluginManagement> 
    
    </pluginManagement>
</project> 
```

## 3.3 Maven的依赖管理（精华部分）

所谓依赖，就是jar包的意思，我们开发项目肯定会用到jar包，使用的jar包称为依赖。依赖管理就是我们项目对jar包的管理！

```xml
<dependencies>
    <!--
       dependency 依赖的意思，单数形态
         groupId  使用的jar包的公司名
         artifactId  使用jar包的项目名
         version 使用jar包版本号
       使用jar包，就是写配置文件，上面的三个标签，jar包的坐标
    -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
```

- jar包坐标的快速写法

```xml
   <!--
      导入德鲁伊连接池jar，快速写法
      <artifactId>写内容</artifactId>
      直接写jar包，即可
    -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.10</version>
    </dependency>

    <!--
      导入lombok.jar 生成get/set
    -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.12</version>
    </dependency>
```

- 编写jar包坐标不提示问题：

  - IDEA 2019.2 版本：几乎不提示，解决办法无！！（卸载，换版本）

  ![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052120.png)



  - IDEA 2019.3版本和以后版本：有提示的，但是，仓库列表是反的！！
  - 写jar包坐标的时候，会提示呢，但是到版本号提示的时候，自动从网上获取该jar包的所有版本号，让我们选一个，会卡一下子！不打算从网获取所有的版本，断网！就会找本地仓库中的jar

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052121.png)

## 3.4 本地仓库没有的jar

上网找：https://mvnrepository.com/

搜索要使用的jar包，选择版本号，复制里面的jar包坐标

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
</dependency>

```

jar包坐标，复制到项目的pom.xml文件中，自动下载jar包到本地仓库 （在阿里云服务器下载）

## 3.5 自动导入包的问题

按钮是刷新按钮：本意是让Maven再次读取配置pom.xml，写的jar才能生效

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052122.png)



![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052123.png)

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052124.png)

> 在pom文件中，定义了jar坐标，自动导入 ， 无需手动刷新

## 3.6 Maven的依赖传递

导入jar包的时候，就会出现依赖的传递问题。例子：导入A.jar包，A包自己不能运行，需要依赖另一个B.jar包才能，A包依赖于B包。Maven会自动处理jar包之间的依赖关系。

单元测试junit，使用@Test注解，需要2个jar包

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052125.png)

## 3.7 创建无骨架项目（重点）

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052126.png)

> 无骨架项目，Maven编译版本为JDK1.5！！

- Maven项目 xml配置里面有编译插件，改变当前项目的编译版本

```xml
 <!--编译插件，改变Maven编译版本-->
    <build>
        <!-- plugins插件的意思 ，可以定义多个插件-->
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <!--Maven的编译插件-->
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <!--配置插件-->
                <configuration>
                    <!--源码 java文件，使用utf-8编码-->
                    <encoding>utf-8</encoding>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

```

## 3.8 Maven项目的生命周期

- clean：清空target目录
- compile ：编译源码为class文件
- test：运行测试类
- package ：将我们的项目打jar包
- install：打的jar包复制到本地仓库中
- 生命周期命令，执行后面的，会自动执行前面的

# 第四章 综合练习-子父项目（必做）

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052127.png)

- 子父项目：按照层的方式拆分为三个子项目，web,service,dao

  - 创建父项目无骨架：名字：parent

    - 父项目作用，为子项目定义jar包坐标的，根本就不写代码，删除src目录
    - 修改父项目的打包方式 （jar默认，war网络程序JavaWeb程序，pom方式，做jar包）

    ```xml
    <!--父项目的打包方式 pom，提供jar包服务的-->
        <packaging>pom</packaging>
    
        <dependencies>
    
            <!-- Apache dbutils-->
            <dependency>
                <groupId>commons-dbutils</groupId>
                <artifactId>commons-dbutils</artifactId>
                <version>1.7</version>
            </dependency>
    
            <!--德鲁伊连接池-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.1.10</version>
            </dependency>
    
            <!--数据库驱动-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.26</version>
            </dependency>
            
             <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
            </dependency>
        </dependencies>
    
        <!--编译插件，改变Maven编译版本-->
        <build>
            <!-- plugins插件的意思 ，可以定义多个插件-->
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <!--Maven的编译插件-->
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                    <!--配置插件-->
                    <configuration>
                        <!--源码 java文件，使用utf-8编码-->
                        <encoding>utf-8</encoding>
                        <source>1.8</source>
                        <target>1.8</target>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    
    ```

  - 创建子项目无骨架：名字：dao，继承父项目

    - 创建子项目的时候，点击一下父项目

    ![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/052128.png)

  - 创建子项目无骨架：名字：service，继承父项目

  - 创建子项目无骨架：名字：web，继承父项目

> jar包父项目已经定义完成，子项目继承直接使用，如果子项目不使用父项目jar包，可以自己定义，近者优先