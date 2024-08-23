##### 笔记

- Shuffle机制  { 分区规则：![1654131429157](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/220602-1654131429157.png)

  ​	哈希值取正，对reduce个数取值

  }

##### hadoop配置文件

- 注意配置maven配置文件使用java8

```xml
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>


    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.1.3</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.30</version>
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

