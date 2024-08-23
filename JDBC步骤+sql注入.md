JDBC

- 今日内容
  - 理解JDBC技术
  - JDBC操作数据库步骤
  - JDBC实现数据CRUD
  - JDBC工具类
  - SQL注入攻击

# 第一章 JDBC技术

## 1.1 JDBC技术介绍

JDBC：（Java DataBase Connectivity）Java的数据库连接技术，实现Java语言和数据库之间的连接技术。通过网络连接到数据库，Java技术向数据库发送SQL语句，获取数据库中的数据！

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05191.png)

> 学习成本上升：关注Java程序，还要关注数据库

## 1.2 数据库驱动程序（非常重要）

我们开发人员是直接写程序开发数据库吗，肯定不是的，数据库的工作原理我们并不清楚

> 数据库驱动程序本质上就是：Sun公司接口的实现了，实现类（驱动）由数据库厂家提供。数据库驱动都是以 jar包形式提供

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05192.png)

## 1.3 JDBC开发步骤（非常重要）

- Java开发数据库步骤：固定的
  - 注册驱动程序
  - 获取数据库连接（网络连接数据库，TCP协议）
  - 获取SQL语句的执行对象
  - 执行SQL语句
  - 处理查询的结果集
  - 释放资源

## 1.4 Sun公司API介绍

- Sun公司的JDBC开发API -- java.sql包中
  - `java.sql.DriverManager`类：管理数据库驱动程序的
    - 负责管理驱动
  - `java.sql.Connection`接口：该接口的实现类表示Java连接数据库的对象
    - 负责连接数据库
  - `java.sql.Statement`接口：该接口的实现类表示要执行SQL语句对象
    - 负责执行SQL语句
  - `java.sql.ResultSet`接口：该接口实现类表示数据查询结果集对象
    - 负责处理查询结果集

> 同学需要注意：接口的是由数据库厂家提供，都是由数据库驱动程序提供！！
>
> JDBC开发是纯粹的面向接口编程思想！

# 第二章 JDBC数据操作 （非常重要）

## 2.1 向数据表插入数据（insert语句）

```java
    /**
     * JDBC技术，连接数据库
     * 实现数据表新增数据
     * - 注册驱动程序
     * - 获取数据库连接（网络连接数据库，TCP协议）
     * - 获取SQL语句的执行对象
     * - 执行SQL语句
     * - 处理查询的结果集
     * - 释放资源
     */
@Test
public void testInsert() throws SQLException {
    //注册驱动程序：告知JVM，要使用哪个数据库驱动
    //DriverManager类静态方法 static void registerDriver(Driver d)
    //Driver参数：Driver是接口传递实现类的对象，接口实现类：数据库驱动实现
    DriverManager.registerDriver(new Driver());

    //获取数据库连接
    //DriverManager类静态方法 static Connection getConnection(String url,String u,String p)
    //方法返回数据库连接对象Connection接口实现类
    //数据库连接地址：连接方式:数据库厂商名://数据库服务器IP地址:端口号/要连接数据库
    String url = "jdbc:mysql://localhost:3306/my3";
    String user = "root";
    String password = "root";
    //调用方法连接数据，传递连接地址，用户名，密码，返回连接对象
    //能拿到连接对象，Java程序和MySQL已经连接成功
    Connection con = DriverManager.getConnection(url,user,password);
    System.out.println("con = " + con);
    //获取SQL语句的执行对象
    //SQL语句执行对象：是接口Statement的实现类对象
    //数据库连接对象方法 ： Statement createStatement() 返回SQL语句执行对象
    Statement stat = con.createStatement();
    System.out.println("stat = " + stat);
    //执行SQL语句
    String sql = "insert into product values(null,'蛋糕',10.5,1555,2)";
    //SQL语句执行对象 stat 的方法 executeUpdate(String sql)
    //返回int类型，表示受影响的行数
    int row = stat.executeUpdate(sql);
    System.out.println("row = " + row);
    //释放资源
    stat.close();
    con.close();
}

```

## 2.2 向数据表更新数据（update语句）

```java
/**
      * JDBC技术，连接数据库
      *  实现数据表更新数据
      */
@Test
public void testUpdate()throws Exception{
    //注册驱动,数据库5.x版本
    //5.x版本 Class.forName("com.mysql.jdbc.Driver");
    Class.forName("com.mysql.cj.jdbc.Driver");
    //数据库连接，获取连接对象Connection接口实现类
    String url = "jdbc:mysql://localhost:3306/my3";
    String user = "root";
    String password = "root";
    //调用方法连接数据，传递连接地址，用户名，密码，返回连接对象
    //能拿到连接对象，Java程序和MySQL已经连接成功
    Connection con = DriverManager.getConnection(url,user,password);
    //System.out.println("con = " + con);
    //获取SQL语句执行对象
    Statement stat = con.createStatement();
    //update语句
    String sql = "update product set pname = '饼干',price = 5.5,num = 1999 where pid = 13";
    //执行SQL语句
    int row = stat.executeUpdate(sql);
    System.out.println("row = " + row);
    stat.close();
    con.close();
}
```

> JDBC操作数据库中的数据：insert,update,delete 三个操作，代码上除了SQL语句不同，其他一样！

## 2.3 JDBC工具类抽取-优化

JDBC操作很多的代码都是一样的，没有必要每次都写，包括每次复制。将共性代码抽取，当需要使用的时候直接调用！

```java
    /**
     * 定义方法，共性抽取
     * 返回数据库连接对象
     * 方法是否加静态：方法是否访问过非静态的成员
     * 如果没有访问过，加static
     */
    public static Connection getConnection()throws Exception{
        Class.forName("com.mysql.cj.jdbc.Driver");
        //数据库连接，获取连接对象Connection接口实现类
        String url = "jdbc:mysql://localhost:3306/my3";
        String user = "root";
        String password = "root";
        //调用方法连接数据，传递连接地址，用户名，密码，返回连接对象
        //能拿到连接对象，Java程序和MySQL已经连接成功
        Connection con = DriverManager.getConnection(url,user,password);
        return con;
    }
}
```

> 工具类抽取测试后没有问题，但是有不好的地方，数据库连接四大信息（驱动类，连接地址，用户名，密码）不能写死，写在配置文件里面，读取文件！

- 配置文件：db.properties 键值对，放在src下

```properties
driverClass=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/my3
user=root
password=root
```

- 工具类优化

```java
/**
 * 自定义的工具类
 * 保存JDBC操作的共性代码
 */
public class JdbcUtils {
    //定义存储数据库连接信息的字符串，静态修饰
    private static String driverClass;
    private static String url;
    private static String user;
    private static String password;

    /**
     * 配置文件读取，程序做一次就行了
     * 利用static代码块读取配置文件
     */
    static {
        try {
            //类加载器，返回字节输入流
            InputStream in =
                JdbcUtils.class.getClassLoader().getResourceAsStream("db.properties");
            Properties prop = new Properties();
            //集合IO关联
            prop.load(in);
            in.close();
            //从集合中，取出键值对
            driverClass = prop.getProperty("driverClass");//驱动类名
            url = prop.getProperty("url");//数据库连接地址
            user = prop.getProperty("user");//连接用户名
            password = prop.getProperty("password");//连接密码
            //注册驱动
            Class.forName(driverClass);
        }catch (Exception ex){}
    }

    /**
     * 定义方法，共性抽取
     * 返回数据库连接对象
     * 方法是否加静态：方法是否访问过非静态的成员
     * 如果没有访问过，加static
     */
    public static Connection getConnection()throws Exception{
        //调用方法连接数据，传递连接地址，用户名，密码，返回连接对象
        //能拿到连接对象，Java程序和MySQL已经连接成功
        Connection con = DriverManager.getConnection(url,user,password);
        return con;
    }
}
```

## 2.4 查询数据表数据

```java
    /**
     * JDBC技术，连接数据库
     *  实现数据表查询数据
     */
@Test
public void testSelect() throws Exception {
    //工具类获取数据库连接对象
    Connection con = JdbcUtils.getConnection();
    //连接对象con的方法，获取Sql语句执行
    Statement stat = con.createStatement();
    //拼写查询的SQL语句
    String sql = "select * from product";
    //执行查询语句,stat对象执行查询语句的方法 executeQuery(String sql)
    //方法 executeQuery返回结果集对象：ResultSet接口
    ResultSet rs = stat.executeQuery(sql);
    //结果集对象rs方法：boolean next() 如果有下一行结果，返回true
    //boolean next()方法和迭代器 hasNext() 一回事
    while (rs.next()){
        //取出数据表中列的数据：结果集对象rs方法getXXX(列名),返回列值
        //取出列 pid的值
        int pid = rs.getInt("pid");
        //取出列 pname的值
        String pname = rs.getString("pname");
        //取出列 price的值
        double price = rs.getDouble("price");
        //取出列 num的值
        int num = rs.getInt("num");
        //取出列 cid的值
        int cid = rs.getInt("cid");
        System.out.println(pid+"\t"+pname+"\t"+price+"\t"+num+"\t"+cid);
    }
    rs.close();
    stat.close();
    con.close();
}
```

## 2.5 JavaBean对象

```
具备私有字段，无参数构造方法，get/set
```

```java
/**
 * Product 类 对应 -- 数据表
 * Product 类中字段 对应 -- 数据表的列
 * Product 类对象 对应 -- 表的行数据
 *
 * 以后这样的类，没有特殊要求的时候，字段类型都写包装类
 *
 * Product类：具备私有字段，无参数构造方法，get/set
 * 具备上述条件的类，称为JavaBean对象
 */
public class Product {
    private Integer pid;
    private String pname;
    private Double price;
    private Integer num;
    private Integer cid;

    public Integer getPid() {
        return pid;
    }

    public void setPid(Integer pid) {
        this.pid = pid;
    }

    public String getPname() {
        return pname;
    }

    public void setPname(String pname) {
        this.pname = pname;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    public Integer getNum() {
        return num;
    }

    public void setNum(Integer num) {
        this.num = num;
    }

    public Integer getCid() {
        return cid;
    }

    public void setCid(Integer cid) {
        this.cid = cid;
    }

    @Override
    public String toString() {
        return "Product{" +
                "pid=" + pid +
                ", pname='" + pname + '\'' +
                ", price=" + price +
                ", num=" + num +
                ", cid=" + cid +
                '}';
    }
}
```

## 2.6 lombok插件

IDEA开发工具，有些功能没有，添加小软件就有了（插件）

lombok现在非常流行插件，自动生成get/set方法

- IDEA安装插件，联网下载

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05194.png)

![5](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05195.png)

- 添加lombok.jar

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05196.png)

- 开启IDEA的注解编译功能

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05197.png)

## 2.7 查询数据存储JavaBean对象

```java
/**
     * 查询数据存储JavaBean对象
     * 数据表有14行数据，将每一行数据，都存储JavaBean对象
     * JavaBean对象，有14个，再存储List集合
     * 习惯：不是new对象，调用方法，现在都是通过一个方法获取对象，对象再来获取对象
     */
@Test
public void testSelectJavaBean()throws Exception{
    //工具类获取数据库连接对象
    Connection con = JdbcUtils.getConnection();
    //通过数据库连接对象，获取SQL语句执行对象
    Statement stat = con.createStatement();
    //拼写查询的SQL语句
    String sql = "select * from product";
    //执行查询语句,stat对象的方法 executeQuery(String sql)
    //返回ResultSet接口实现类，查询数据表的结果集
    ResultSet rs = stat.executeQuery(sql);
    //List集合，存储JavaBean对象
    List<Product> list = new ArrayList<>();
    //循环遍历结果集
    while (rs.next()){
        //如果存在结果集，数据行，进入循环
        //数据表的每行数据，对象JavaBean对象
        Product product = new Product();
        //取出数据表列值，存储product对象中
        product.setPid( rs.getInt("pid") );
        product.setPname(rs.getString("pname"));
        product.setPrice( rs.getDouble("price"));
        product.setNum( rs.getInt("num") );
        product.setCid( rs.getInt("cid") );
        //JavaBean对象存储集合
        list.add(product);
    }
    for (Product p : list){
        System.out.println("p = " + p);
    }
    rs.close();
    stat.close();
    con.close();
}
```

# 第三章 SQL注入攻击

## 3.1 什么是SQL注入攻击

利用数据库SQL语句漏洞进行攻击的行为。注入攻击的实例：用户登录功能实现中，就会出现注入攻击行为。

```java
/**
     * 模拟用户登录，实现SQL语句注入攻击
     */
public  static void userLogin() throws Exception {
    //创建对象Scanner，键盘输入
    Scanner sc = new Scanner(System.in);
    //sc对象方法：String nextLine() 接收控制台输入，字符串
    System.out.println("请输入用户名");
    String user =sc.nextLine();
    System.out.println("请输入密码");
    String pass =sc.nextLine();


    //工具类获取数据库连接对象
    Connection con = JdbcUtils.getConnection();
    //获取SQL语句的执行对象
    Statement stat = con.createStatement();
    //拼写登录查询SQL
    String sql = "select * from user where username = '"+user+"' and password = '"+pass+"' ";

    //select * from user where username = 'tom' and password = '123 or 1=1'
    //select * from user where username = 'tom' and password = '123' or '1=1'
    //select * from user where username = 'sdfgh' and password = 'sdrfgh' or'1=1'
    System.out.println("sql = " + sql);
    //执行查询语句，返回查询的结果集
    ResultSet rs = stat.executeQuery(sql);
    while (rs.next()){
        //取出数据表中的列值
        int id = rs.getInt("id");
        String username = rs.getString("username");
        String password = rs.getString("password");
        System.out.println(id+"\t"+username+"\t"+password);
    }
    rs.close();
    stat.close();
    con.close();
}
```

## 3.2 PreparedStatement接口（非常重要）

- `java.sql.PreparedStatement`接口：执行SQL语句的接口
  - PreparedStatement 子接口继承Statement接口 （接口实现类，数据驱动包里面的）
  - PreparedStatement 接口：防止注入攻击
  - PreparedStatement 接口：执行查询语句，同一个查询语句，速度快

> 执行SQL语句，推荐使用子接口 PreparedStatement 做

- PreparedStatement 接口实现类怎么获取
  - 数据库连接对象con：方法 prepareStatement(传递SQL语句)

```java
    /**
     * 子接口PreparedStatement实现数据表登录查询
     * 父接口Statement，退出历史舞台
     */
    public static void testPreparedStatement() throws Exception {
        //工具类获取数据库连接对象
        Connection con = JdbcUtils.getConnection();
        //先拼SQL语句,SQL语句中的所有参数使用 ？ 占位符
        String sql = "select * from user where username = ? and password = ?";
        //连接对象con的方法 prepareStatement()获取执行SQL语句的对象 （子接口）
        PreparedStatement pst = con.prepareStatement(sql);

        //设置SQL语句中 ？ 占位符的实际参数
        //pst对象的方法 setObject(int 第几个问号，Object 实际值)
        pst.setObject(1,"tom");
        pst.setObject(2,"123");

        //执行SQL语句，pst对象的方法 executeQuery() 不再传递SQL语句
        ResultSet rs = pst.executeQuery();
        while (rs.next()){
            //取出数据表中的列值
            int id = rs.getInt("id");
            String username = rs.getString("username");
            String password = rs.getString("password");
            System.out.println(id+"\t"+username+"\t"+password);
        }
        rs.close();
        pst.close();
        con.close();
    }
```







