# Maven安装与使用

## 1、Maven简介

### 1.1 Maven概念

Apache Maven是一个项目管理合构建工具，它基于项目对象模型（POM）的概念，通过一小段描述信息来管理项目的构建、报告和文档。

Maven官网：[https://maven.apache.org/](https://maven.apache.org/)

### 1.2 Maven的作用

> 标准化的项目结构

+   不同的IDE之间，项目结构不一样，不通用。
    

![image-20220202153701427](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/uByXK6QSkNjFVAt.png)

+   Maven提供了一套标准化的项目结构，所有IDE使用Maven构建的项目结构完全一样，不同的IDE之间可以通用。
    

![image-20220202154020180](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/LKgToOJUxI37HBe.png)

> 标准化的项目构建

Maven提供了一套简单的命令来构建项目。

![image-20220202154113761](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/7aW2NOitgf4yvrX.png)

> 方便的依赖管理

依赖管理就是管理项目所依赖的第三方资源（jar包、插件等）。

+   原生的依赖管理：
    

![image-20220202154430536](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/ZleG4UNALhm8ayn.png)

+   Maven提供的依赖管理：
    
    +   Maven使用标准的坐标配置来管理各种依赖
        
    
    ![image-20220203030500735](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/84mTCqLAz1O2BUx.png)
    

### 1.3 Maven模型

![image-20220202154903260](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/CTvIMFuVjPzR7cq.png)

### 1.4 Maven仓库

Maven仓库依赖信息查询网站：[https://mvnrepository.com/](https://mvnrepository.com/)

#### 1.4.1 Maven仓库概念

Maven仓库是项目中依赖的第三方库，这个库所在的位置叫做仓库。

#### 1.4.2 Maven仓库分类

+   **中央仓库（Central Repository）**
    
    +   由Maven团队维护的全球唯一仓库，地址：[https://repo1.maven.org/maven2/](https://repo1.maven.org/maven2/)
    
+   **本地仓库（Local Repository）**
    
    +   自己计算机上的一个目录
    
+   **远程仓库【私服】（Remote Repository）**
    
    +   由公司或团队搭建的私有仓库
        

> 仓库的访问顺序：本地仓库 --- 远程仓库 --- 中央仓库

![image-20220202160339627](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/dtgsqbhSFNv7YnP.png)

## 2、Maven的下载、安装、配置

### 2.1 Mavan下载

地址：[https://maven.apache.org/](https://maven.apache.org/)

![image-20220202160932347](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/CvZ5Bht49ki7bdH.png)

![image-20220202161118914](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/sjoNq87GUu4CaJ9.png)

### 2.2 Maven安装

将压缩包进行解压即可。

![image-20220202161252798](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/pjeFWux5SKnGHVa.png)

![image-20220202161520993](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/yUAFsSndbhc6Qer.png)

### 2.3 Maven配置

#### 2.3.1 配置环境变量

![image-20220202161720633](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/VJcB1bjZxG3knTD.png)

![image-20220202161818667](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/rMjLxm1B9Td7FHI.png)

+   测试是否配置成功：
    

mvn \-v

![image-20220406124814455](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/GoiAJ9aThvynxDs.png)

> **配置失败原因分析**：
>
> +   未配置`JAVA_HOME`
>
> +   在用户变量和系统变量中同时配置了`JAVA_HOME`和`MAVEN_HOME`
>
> +   JDK信息使用了默认信息
>
>
> **解决方案**：
>
> > 只在系统变量中设置`JAVA_HOME`和`MAVEN_HOME`，删除默认JDK配置信息
> >
> > ![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/SeZnFuyGRz9lC2H.png)
> >
> > ![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/kf5RpyPIdZ1LMnO.png)

#### 2.3.2 配置本地仓库及远程仓库

![image-20220202162700780](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/luOyI6fzUCV3bwo.png)

+   **配置本地仓库**
    
    +   在计算机上新建文件夹：`mvn_repository`
        
    
    ![image-20220202162959382](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/PXlR6AfExzojbND.png)
    
    +   在`setting.xml`中的`<localRepository>`标签中配置本地仓库的路径
        
    
    默认路径为：`C:\Users\Administrator\.m2\repository`
    
    xxxxxxxxxx
    
    <localRepository\>C:\\SoftWare\\Program\\Java\\apache-maven-3.8.4\\mvn\_repository</localRepository\>
    
+   **配置远程仓库**
    
    +   配置远程仓库的意义：中央仓库在国外，访问速度较慢；访问国内企业搭建的私服，可以提升访问速度。
        
    +   配置阿里云私服：
        
        在`setting.xml`中的`<mirrors>`标签中配置子标签
        
        xxxxxxxxxx
        
        <mirror\>
        
            <id\>alimaven</id\>
            
            <name\>aliyun mavne</name\>
            
            <url\>http://maven.aliyun.com/nexus/content/groups/public/</url\>
            
            <mirrorOf\>central</mirrorOf\>
        
        </mirror\>
        

#### 2.3.3 在eclipse中配置Maven

+   配置Maven存放地址
    
    ![image-20220406091522595](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/N9gWb62Mc5enESQ.png)
    
+   配置Maven仓库地址
    
    ![image-20220406091639656](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/e9rEBa6f2C5HDXM.png)
    

## 3、在eclipse中创建Maven项目

### 3.1 Maven项目创建步骤

![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/BzLKPE7hFdWCfj6.png)

![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/KwM2glNhV8Wp9Pq.png)

![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/rahqQ96FZmBNzVE.png)

![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/E9Ih8WrMsHGeoLb.png)

![image-20220406092748197](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/Z74SbGKicDV9rzQ.png)

### 3.2 创建Maven项目常见文件问题

#### 3.2.1 选择原型内容为空

![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/BIigCoLEM23pDUG.png)

【**解决方案**】

在命令对话框输入如下命令下载原型信息：

xxxxxxxxxx

mvn archetype:generate

![image-20220406094623142](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/9u2ZTl5OQJgtH6w.png)

#### 3.2.2 解决项目创建后报错的问题

+   下载打包插件、设置jdk版本
    

> 在pom.xml中配置如下信息：
>
> xxxxxxxxxx
>
> <plugins\>
>
> <plugin\>
>
>   <!-- 下载打包插件 -->
>
>   <groupId\>org.apache.maven.plugins</groupId\>
>
>   <artifactId\>maven-war-plugin</artifactId\>
>
>   <version\>3.3.1</version\>
>
>   <!-- 设置JDK版本 -->
>
>   <configuration\>
>
>       <source\>1.8</source\>
>     
>       <target\>1.8</target\>
>
>   </configuration\>
>
> </plugin\>
>
> </plugins\>
>
> ![image-20220406123222866](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/7cKPdNfJ1jY9yMt.png)

+   更新Maven项目
    
    ![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/5hfsxlupCPLMzJV.png)
    
    ![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/9l1ptke8ioLxbRf.png)
    
+   设置项目JDK，Tomcat，JDK版本等信息
    
    ![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/KXVgBPhfpOkn3yW.png)
    

![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/IzLHvqSBbXd5AZp.png)

![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/pSle4tCWKHbikmq.png)

![](https://gitee.com/ChnpngWang/typora-image/raw/master/assets/maven/kFtTKfVSX83xGHM.png)