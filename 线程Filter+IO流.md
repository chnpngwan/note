##### 笔记

- 生产者消费者代码案例
- 线程的生命周期
- File类

多线程&IO

- 今日内容
  - 生产者与消费者案例
  - 线程通信方法
  - 生产者消费者案例优化
  - 线程生命周期
  - File类构造器
  - File类常用方法
  - File类目录遍历

# 第一章 生产者与消费者

## 1.1 案例介绍

生产者与消费者案例：角色一个生产者（线程），一个消费者（线程），针对于同样的一个资源数据，进行不同方向的操作。最终要达到的目的：生产一个消费一个，不能一次生产多个，也不能一次消费多个。实现的效果：生产1个，消费1个，生产2个，消费2个...

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05131.png)



## 1.2 案例实现

- 生产和消费共用一个对象

```java
/**
 * 资源对象，定义产品的计数器
 */
public class Resource {
    int count;//定义产品的计数器 default = 0
}
```

```java
/**
 * 生产者线程，负责对资源对象中计数器++操作
 */
public class Produce implements Runnable{
    //创建资源对象
    private Resource r ;

    public Produce(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true){
            r.count++;
            System.out.println("生产第"+r.count+"个");
        }
    }
}
```

```java
/**
 * 消费者线程，负责将资源对象中的计数器输出打印
 */
public class Customer implements Runnable{
    //创建资源对象
    private Resource r ;

    public Customer(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true){
            //输出资源对象中的计数器
            System.out.println("消费第"+r.count+"个");
        }
    }
}
```

```java
    public static void main(String[] args) {
        //创建资源对象
        Resource r = new Resource();
        //开启生产者线程和消费者线程
        Runnable product = new Produce(r);
        Runnable customer = new Customer(r);
        new Thread(product).start();//开启生产者线程
        new Thread(customer).start();//开启消费者线程
    }
```

- 数据安全加入同步锁，保证同步对象锁唯一的

```java
/**
 * 生产者线程，负责对资源对象中计数器++操作
 */
public class Produce implements Runnable{
    //创建资源对象
    private Resource r ;
    public Produce(Resource r) {
        this.r = r;
    }
    @Override
    public void run() {
        while (true){
            synchronized (r) {
                r.count++;
                System.out.println("生产第" + r.count + "个");
            }
        }
    }
}
```

```java
/**
 * 消费者线程，负责将资源对象中的计数器输出打印
 */
public class Customer implements Runnable{
    //创建资源对象
    private Resource r ;

    public Customer(Resource r) {
        this.r = r;
    }
    @Override
    public void run() {
        while (true){
            synchronized (r) {
                //输出资源对象中的计数器
                System.out.println("消费第" + r.count + "个");
            }
        }
    }
}
```

## 1.3 线程通信的方法

线程通信的方法：线程的等待和唤醒，定义在Object类中：

- void wait() 线程无限等待
- void notify() 唤醒等待的单个线程

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05132.png)

- 线程等待与唤醒中的异常问题

![](D:/Atguigu/day22源码笔记/笔记/images/3.png)

```properties
IllegalMonitorStateException：无效的监视器状态异常
程序中出现这个异常：线程wait() 和 notify() 造成的异常：监视器就是对象锁
wait(),notify() 必须要出现在同步中，方法的调用者必须是锁对象！！
```

```java
/**
 * 资源对象，定义产品的计数器
 */
public class Resource {
    int count;//定义产品的计数器 default = 0
    /**
     *  定义标志位 default false
     *  flag = false，可以生产，不能消费
     *  flag = true ，可以消费，不能生产
     */
    boolean flag ;
}
```

```java
/**
 * 生产者线程，负责对资源对象中计数器++操作
 */
public class Produce implements Runnable{
    //创建资源对象
    private Resource r ;

    public Produce(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true){
            synchronized (r) {
                //判断变量flag =true ，无限等待
                if (r.flag)
                   try{
                       r.wait();
                   }catch (Exception ex){}

                r.count++;
                System.out.println("生产第..." + r.count + "个");
                //改标志位，可消费
                r.flag = true;
                //唤醒对方线程
                r.notify();
            }
        }
    }
}

```

```java
/**
 * 消费者线程，负责将资源对象中的计数器输出打印
 */
public class Customer implements Runnable{
    //创建资源对象
    private Resource r ;

    public Customer(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true){
            synchronized (r) {
                //判断标记 flag=false，不能消费，无限等
                if (!r.flag)
                   try{
                       r.wait();
                   }catch (Exception ex){}

                //输出资源对象中的计数器
                System.out.println("消费第" + r.count + "个");
                //消费完毕改标志位 flag = false 可以生产
                r.flag = false;
                //唤醒对方线程
                r.notify();
            }
        }
    }
}


```

## 1.4 wait()和sleep()区别

- wait()是Object类的，非静态
- sleep()是Thread类的，静态的
- wait()等待，需要被唤醒
- sleep()休眠，时间到自己就醒了
- wait()方法调用者必须是锁对象
- sleep()方法出现哪里都可以



- **sleep()方法不会释放同步锁**
- **wait()方法会释放同步锁，被唤醒后要重新获取同步锁才能执行**

## 1.5 生产者与消费者案例优化

```java
/**
 * 资源对象，定义产品的计数器
 * 成员私有修饰，提供get set
 */
public class Resource {
   private int count;//定义产品的计数器 default = 0
    /**
     *  定义标志位 default false
     *  flag = false，可以生产，不能消费
     *  flag = true ，可以消费，不能生产
     */
   private boolean flag ;
    /**
     * 定义get方法，被消费者线程调用
     * 完成消费者的工作
     */
    public synchronized void get(){
        //判断标志位
        if (!flag)
            try{this.wait();}catch (Exception ex){}
        //消费
        System.out.println("消费第"+count+"个");
        //改标志位
        flag = false;
        this.notify();
    }

    /**
     * 定义set方法，被生产者线程调用
     * 完成生产者的工作
     */
    public synchronized void set(){
        //判断标志位
        if (flag)
            //等待
            try{this.wait();}catch (Exception ex){}
        //生产
        count++;
        System.out.println("生产第..."+count+"个");
        //修改标志位
        flag = true;
        this.notify();
    }
}

```

```java
/**
 * 生产者线程，负责对资源对象中计数器++操作
 */
public class Produce implements Runnable{
    //创建资源对象
    private Resource r ;

    public Produce(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true)
            //调用资源对象的set方法
            r.set();
    }
}

```

```java
/**
 * 消费者线程，负责将资源对象中的计数器输出打印
 */
public class Customer implements Runnable{
    //创建资源对象
    private Resource r ;

    public Customer(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true)
            //调用资源对象的方法get
            r.get();
    }
}

```



# 第二章 线程生命周期

所谓线程的声明周期：线程的状态，反映出线程从创建到结束的生命历程。API已经给出了线程的生命周期。

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05134.png)

- `java.util.concurrent.ConcurrentHashMap`类：JDK1.5版本
  - Map接口的实现类，键值对的集合
  - 底层的数据结构和HashMap一致
  - ConcurrentHashMap类是一半线程安全，一半线程不安全
  - 如果不修改里面的键值对，线程不安全的
  - 如果动了里面的键值对，线程是安全



# 第三章 File类

## 3.1 File类介绍

- `java.io.File`类
  - 计算机存在路径，目录，文件
  - File类将计算机中的路径，目录，文件做成对象
  - 提供很多的方法，操作路径，目录和文件

- 计算机操作系统
  - 目录就是文件夹（Directory），不能存储数据，是文件的容器
  - 文件（File），可以存储数据
  - 路径（Path），一个文件或者目录，在计算机中的存储位置

> File类，是平台无关性的类

## 3.2 File类构造方法 （非常重要）

- File(String pathname)传递字符串的路径，可以表示目录也可以表示文件

```java
/**
     * File(String pathname)传递字符串的路径，可以表示目录也可以表示文件
     * 路径里面，IDEA自动添加转义字符
     */
public static void testConstructor1(){
    File file = new File("E:\\soft\\apache-maven-3.3.9\\bin\\m2.conf");
    System.out.println("file = " + file);
}

```

- File(String parent,String child) 传递字符串的父路径，字符串的子路径

```java
    /**
     * File(String parent,String child) 传递字符串的父路径，字符串的子路径
     * E:\soft\apache-maven-3.3.9\bin  为参考点
     * E:\soft\apache-maven-3.3.9\  是父路径
     * E:\soft\apache-maven-3.3.9\bin\ 里面的都是子路径
     */
    public static void testConstructor2(){
        File file = new File("e:\\soft\\apache-maven-3.3.9","bin");
        System.out.println("file = " + file);
    }

```

- File(File parent,String child) 传递File类型父类经，字符串的子路径

```java
    /**
     * File(File parent,String child) 传递File类型父类经，字符串的子路径
     * 构造方法的第一个参数是File类型，可以调用File类的方法
     */
    public static void testConstructor3(){
        File parent = new File("E:\\soft\\apache-maven-3.3.9");
        File file = new File(parent,"bin");
        System.out.println("file = " + file);
    }

```

> Windows操作系统：路径，目录，文件的名字，都是不区分大小写。Linux操作系统区分
>
> 路径的写法：Windows \，Linux /   Java中路径，写 \ 或者 / 都可以

## 3.3 File类的创建和删除的方法（非常重要）

- boolean createNewFile() 创建文件的方法，文件的路径和名字定义在File类构造方法中

```java
    /**
     * boolean createNewFile() 创建文件的方法，文件的路径和名字定义在File类构造方法中
     * 创建成功返回true
     * createNewFile() 无论名字怎么写，创建出现来都是文件
     */
    public static void testCreateNewFile() throws IOException {
        //创建File对象，路径和文件名写在构造方法中
        File file = new File("e:/a.txt");
        //创建文件
        boolean b = file.createNewFile();
        System.out.println("b = " + b);
    }

```

- boolean mkdirs() 创建目录，目录的名字和路径写在File类构造方法中

```java
    /**
     * boolean mkdirs() 创建目录，目录的名字和路径写在File类构造方法中
     * 创建成功返回true
     */
    public static void testMkdirs(){
        //创建File对象，目录路径写在构造方法中
        File file = new File("e:/a/b/c/d");
        //调用方法创建文件夹
        boolean b = file.mkdirs();
        System.out.println("b = " + b);
    }

```

- boolean delete() 删除，要删除的文件或者是目录写在File类构造方法中

```java
/**
     * boolean delete() 删除，要删除的文件或者是目录写在File类构造方法中
     * 删除不走回收站，在硬盘中就没有了
     * 删除有风险，执行需谨慎
     * 如果文件夹不是空的，就删除不掉
     */
public static void testDelete(){
    File file = new File("e:/a");
    //boolean b = file.delete();
    //System.out.println("b = " + b);
}

```

## 3.4 File类的判断方法（非常重要）

- boolean exists() 判断File构造方法中路径，是不是存在的，存在返回true

```java
    /**
     * boolean exists() 判断File构造方法中路径，是不是存在的，存在返回true
     */
    public static void testExists(){
        //建立File对象，构造方法中，写路径
        File file = new File("E:\\SOFT\\APACHE-MAVEN-3.3.9\\BIN");
        //判断路径是否存在
        boolean b = file.exists();
        System.out.println("b = " + b);
    }

```

- boolean isFile() 判断构造方法中的路径，是不是文件，是文件返回true
- boolean isDirectory() 判断构造方法中的路径，是不是目录，是目录返回true

```java
    /**
     *  boolean isFile() 判断构造方法中的路径，是不是文件，是文件返回true
     *  boolean isDirectory() 判断构造方法中的路径，是不是目录，是目录返回true
     */
    public static void testIsFileDirectory(){
        File file = new File("e:\\soft\\apache-maven-3.3.9\\bin\\mvn.cmd");
        if (file.exists()) {
            //判断路径是不是目录
            boolean directory = file.isDirectory();
            System.out.println("directory = " + directory);

            //判断是不是文件
            boolean isFile = file.isFile();
            System.out.println("isFile = " + isFile);
        }
    }

```

## 3.5 File类获取方法（非常重要）

- String getName()获取名字

```java
    /**
     * String getName()获取名字
     * 返回的名字，File构造方法中，路径最后部分的名字
     */
    public static void testGetName(){
        File file = new File("E:\\soft\\apache-maven-3.3.9\\bin\\m2.conf");
        //getName()方法获取名字
        String name = file.getName();
        System.out.println("name = " + name);
    }

```

- File getParentFile() 返回File构造方法中，路径的父路径（上一级文件夹）

```java
    /**
     * File getParentFile() 返回File构造方法中，路径的父路径（上一级文件夹)
     * getParentFile()方法的返回值是File对象
     * 当一个方法的返回值是一个对象的时候，继续使用对象调用方法
     * 链式编程，方法调用链
     */
    public static void testGetParentFile(){
        File file = new File("E:\\soft\\apache-maven-3.3.9\\bin");
        //获取构造方法中，路径的父路径
        File parent = file.getParentFile();
        System.out.println("parent = " + parent);
    }

```

- File getAbsoluteFile() 返回File构造方法中个，路径的绝对路径

```java
    /**
     * File getAbsoluteFile() 返回File构造方法中个，路径的绝对路径
     * 绝对路径，在计算机系统中，具有唯一性
     * 例子：E:\soft\apache-maven-3.3.9\bin
     *
     * File file = new File("bin");
     * 构造方法中，路径不是绝对路径
     * 返回的绝对路径：是IDEA创建的工程根目录  E:\0411JavaSe\bin
     */
    public static void testGetAbsoluteFile(){
        File file = new File("bin");
        //获取构造方法中路径的绝对路径
        File absoluteFile = file.getAbsoluteFile();
        System.out.println("absoluteFile = " + absoluteFile);
    }

```

## 3.6 ListFiles (非常重要)

- File[] listFiles() 获取File构造方法中，路径下的所有文件和文件夹 （目录遍历）

```java
    /**
     * File[] listFiles() 获取File构造方法中，路径下的所有文件和文件夹 （目录遍历）
     */
    public static void testListFiles(){
        File file = new File("c:/abc");
        //调用方法listFiles 目录的遍历
        File[] fileArray = file.listFiles();
        System.out.println(fileArray.length);
        for (File f : fileArray){
            System.out.println("f = " + f);
        }
    }

```

- 文件过滤器接口
  - `java.io.FileFiter`接口：
    - 接口的实现类，就是文件过滤器
    - 自定义类实现接口，重写方法 accept

```java
/**
 * 自定义文件过滤器
 */
public class MyFileFilter implements FileFilter {
    //重写抽象方法
    /**
     *
     * accept方法是由listFiles调用，传递遍历到的路径
     * 判断这个路径filename，是.java结尾返回true
     * filename = c:\abc\1.docx
     */
    public boolean accept(File filename){
        //filename.getName() 获取路径中的文件名，返回字符串
        //文件名字符串.toLowerCase()变小写
        return filename.getName().toLowerCase().endsWith(".java");
    }
}

```

```java
    /**
     * listFiles方法，遍历目录，只要java文件
     * listFiles(传递文件过滤器接口实现类对象)
     */
    public static void testListFilesFilter(){
        File file = new File("c:/abc");
        //调用方法listFiles 目录的遍历
        File[] fileArray = file.listFiles( new MyFileFilter() );
        for (File f : fileArray){
            System.out.println("f = " + f);
        }
    }

```

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05135.png)



## 3.7 递归遍历所有目录

```java
public class FileDemo06 {
    public static void main(String[] args) {
        getDir( new File("c:/java") );
    }

    /**
     * 定义方法：遍历指定的目录
     * 将要遍历的目录传递参数
     */
    public static void getDir(File srcDir){
        System.out.println(srcDir);
        //遍历目录
        File[] arrFile =srcDir.listFiles();
        for (File f : arrFile){
            //判断 路径 f 是不是文件夹（目录）
            if (f.isDirectory()){
                //调用自己，传递f 路径
                getDir(f);
            }else {
                System.out.println(f);
            }
        }
    }
}

```

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05136.png)

# 第四章 IO流

## 4.1 IO流对象介绍

IO是两个单词的缩写（Input 输入，Output 输出），流是一种比喻，数据从一个设备流向另一个设备。文件1.txt 可以从硬盘中流向内存中

![](images/7.png)

> IO流应用：数据从一个计算机流向另一个计算机，网络通信

## 4.2 IO流对象的分类（非常重要）

- 按照数据流的方向分类：
  - 输出流：Java程序中的数据，流到其他设备，写入
  - 输入流：数据从其他设备，流到Java的程序中，读取
- 按照操作的数据类型分类：
  - 字节流：可以操作任意类型的数据
  - 字符流：只能操作文本类型的数据 （文本工具，记事本，Notepad，editplus）打开这个文件，文本工具会自动查询编码表，结果人能看懂，就是文本
- 总结归类 （4大类）：
  - 字节输出流：可以写入任何数据文件
    - 字节输出流顶级父类：`java.io.OutputStream`
  - 字节输入流：可以读取任何数据文件
    - 字节输入流顶级父类：`java.io.InputStream`
  - 字符输出流：只能写入文本文件
    - 字符输出流顶级父类：`java.io.Writer`
  - 字符输入流：只能读取文本文件
    - 字符输入流顶级父类：`java.io.Reader`

## 4.3 编码表

编码表：为了让计算机可以直接识别人类的文字，开发出了编码表，生活中的一个字对应十进制的整数

- ASCII 编码表：字母，数字，符号，都是1个字节存储，都是正数
  - a-z   97-122
  - A-Z  65-90
  - 0-9  48-57
- GB系列字符集（多个编码表的集合），简体中文
  - GB2312，简体中文编码表，4000多个汉字
  - GBK，简体中文编码表，21886多个汉字
  - GB18030，简体中文编码表，70244多个汉字
  - 中文版操作系统，默认使用的汉字编码表就是GBK
  - 简体中文编码表中，一个汉字占用2个字节，是负数，第一个字节肯定是负数
- Unicode字符集（多个编码表的集合），万国码
  - UTF-8，最小用1个字节存储，汉字是三个字节，基本上负数，可变长度的编码
  - UTF-16，定长的编码表，无论什么字，都是2个字节存储
    - Java中的char类型，和String类型，都是UTF-16编码存储
    - JDK9开始char没变，String类型变了
      - 如果存储汉字，都是UTF-16编码存储
      - 字符串没有汉字，全是字母，数组，拉丁文编码表存储，1个字节存储
  - 最常用的编码表UTF-8









