##### 课堂笔记

- CompareUtil-----ExceptionTest(exception)-----Wrapper------

- 匿名内部类使用：
  - new 父类|接口（）{}；	：隐含的使用了继承---调用父类|接口的构造器
- 比大小 return减法数据
- main方法中一般不thorws异常
- 静态方法没有覆盖性【方法覆盖四条件】
- 捕获再抛出异常---包装：对象关联
  - 异常使用方式选择
- 单元测试【JUnit】
- 常用类
- api：Integer与String简单查看
- 装箱与拆箱：
  - 包装类之间不兼容
- String类：intern()方法：GC区的方法强行放到常量区
  - String中的数组复制：arrcopy







#### 每日一考_day17

1. 什么是匿名内部类? 如何使用?	

   在方法中声明的同时并创建对象的内部类. 因为没有名字,所以必须同时创建,并且不能再创建别的对象.

   new 父类(...) | 接口() {

   ​	类体部分就是父类或接口的子类的类体

   };

   适用于某个类和对象的一次性使用. 通常和接口配合.

   new 接口() {

   ​	实现接口中的抽象方法.

   };



1. 定义一个枚举MyEnum, 包含3个常量对象ONE, TWO, THREE

   获取所有对象的数组, 再获取名为"THREE"的枚举对象

   ```java
   enum MyEnum {
       ONE, TWO, THREE
   }
   
   class Test {
       
       public static void main(String[] args) {
           MyEnum en0 = MyEnum.TWO;
       	MyEnum[] arr = MyEnum.values();
           MyEnum en = MyEnum.valueOf("THREE");
       }
   }
   ```

   



3. 异常按照处理方式分为几种? 各包含哪些类?

   受检异常 : 必须接受检查和处理的异常, 如果不处理, 编译出错, 所以也称为编译时异常

   ​	Exception及其子类(RuntimeException及其子类除外) : 一般严重问题

   

   非受检异常 : 不是必须接受检查和处理的异常, 如果 不处理, 编译不出错, 但是运行时仍然会出问题

   ​			所以也称为运行时异常

   ​	Error及其子类 : 极其严重

   ​	RuntimeException及其子类 : 非常常见, 比较轻微



4. 判断
   1) 非受检异常就是必须不要处理的异常 F
      	2) 受检异常就是可以处理的异常 F
      	3) 非受检异常是不必须处理的异常 T 
      	4) 受检异常可以对其忽略 F
      	5) 无论是什么异常,都必须对其进行处理 F
      	6) 只有受检异常会引起程序中断 F 
      	7) 受检异常是必须对其进行处理的异常 T 
      	8) 只有非受检异常会引起程序中断 F
      	9) 异常处理只适用于受检异常 F
      	10) 异常处理适用于所有异常, 包括Error T

   

5. 异常的处理有几种方式, 各是什么, 如何处理?

   1) 捕获 

   ​	try {

   ​		可能抛出异常的语句;

   ​	} catch (可能的异常类型1 引用) {

   ​		处理异常.		

   ​	} catch (可能的异常类型2 引用) {

   ​		处理异常2;

   ​	} ..... {

   ​	

   ​	} finally {

   ​		必须要执行的语句, 无论前面try, catch中...发生什么都不影响我的执行.

   ​		这里主要做的工作就是释放不在GC区中的属于OS的资源, 防止资源泄露

   ​	}

   2) 抛出 

   ​	在方法声明中添加throws 可能的异常类型列表.



#### idea中导入第三方jar

![1651808314516](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1651808314516.png)





![1651808419291](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1651808419291.png)





![1651808526278](D:/Atguigu/04_Note/imgs/1651808526278.png)

![1651809026029](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1651809026029.png)

### java中的Test包

##### 使用步骤

~~~java
/**
 * 单元测试 : 并不是JDK内置的功能, 是一个第三方的库, 需要导入.
 *
 * 1) 右击项目, 选择new => directory : 在项目下创建新目录 lib
 * 2) 把需要的.jar文件复制到这个目录中
 * 3) 右击每一个.jar文件, 点击 "Add as Library", 把它加为项目的一个库
 *    添加成功的标志就是这个.jar可以展开. 展开以后可以看到里面的豆子.
 * 4) 右击模块, Open Module Settings...
 * 5) 在左面选中 libraries , 在右面的列表再右击 .jar => add to modules
 * 6) 在弹出的窗口中选中目标模块即可.
 *
 * 单元测试的注意点 :
 *      类必须是公共的, 并且没有任何构造器
 *      测试方法是公共的, 无参无返回的非静态方法
 */
~~~

- 实例代码
- 注意方法签名--public void test1()

~~~java
package com.atguigu.javase.test;

import org.junit.Test;=
public class JUnitTest {

    @Test
    public void test1() {
        System.out.println("Hello");
        //new Integer();
    }
}
~~~

### 异常处理收尾

###### 定义

 * 异常 : 程序在运行时出现的非正常状况, 会导致程序的崩溃.
 * 受检异常 : 必须接受检查和处理的异常
 * 非受检异常 : 不是必须接受检查和处理的异常

###### 处理方式

~~~java
 /* 处理方式  :
 *      1) 捕获
 *          try catch finally
 *      2) 抛出
 *          在方法声明中使用throws 抛出异常类型
 *          throws是警告作用
 *          throw是真的抛出
 *      3) 捕获再抛出
 *         在普通方法中 :
 *          try {
 *
 *          } catch(可能的异常类型 引用) {
 *              throw new 自定义异常(引用);
 *          }
~~~

###### 方法覆盖：最终版条件

~~~java
 /* 方法覆盖 : Override, 子类方法重写父类继承来的方法.
 * 条件 :
 *      1) 方法签名要一致, 返回值类型 方法名 参数列表(类型, 个数, 顺序)
 *          如果返回值类型是类类型, 在重写时, 子类方法返回的类型可以是父类方法返回类型的子类类型.
 *      2) 子类方法的访问控制修饰符要大于等于父类方法的访问控制修饰符
 *      3) 父类和子类中的方法都是非static的. 静态方法是共存关系.
 *      4) 子类重写方法抛出的受检异常要小于等于父类类型的.
~~~

###### 异常处理方式选择

~~~java
/* 处理方式的选择 :
 *      功能方法或被调用的方法抛出异常.
 *      主方法或入口方法要捕获异常.
 *      原则 : 方法出问题会不会影响栈的生存. 不会让栈死的方法要抛出异常, 会导致栈死的方法要捕获异常.
 *
 */
// 自定义异常, 1) 继承Exception 2) 提供几个构造器
~~~

###### 实例代码

~~~java
package com.atguigu.javase.exception;

class DividedByZeroException extends Exception {

    public DividedByZeroException(String message) {
        super(message);
    }

    public DividedByZeroException(Throwable cause) {
        super(cause); // 对象关联
    }

}

public class ExceptionTest {

    public static int divide(int x, int y) throws DividedByZeroException {
        try {
            return x / y;
        } catch (ArithmeticException e) {
            // 包装
            throw new DividedByZeroException(e); // 对象关联, 自定义异常对象关联了捕获到的异常
        }
    }

    public static void main3(String[] args) {
        System.out.println("main begin");

        try {
            System.out.println(divide(10, 2));
            System.out.println(divide(10, 0));
        } catch (DividedByZeroException e) {
            e.printStackTrace();
        }

        System.out.println("main end");
    }

    public static int divide2(int x, int y) throws DividedByZeroException {
        if (y == 0) {
            throw new DividedByZeroException("除数不可以为0");
        }
        return x / y;
    }

    //主方法不要抛出异常
    //public static void main(String[] args) throws DividedByZeroException {
    public static void main(String[] args) {
        System.out.println("main begin");

        try {
            System.out.println(divide2(10, 2));
            System.out.println(divide2(10, 0));
        } catch (DividedByZeroException e) {
            //e.printStackTrace();
            System.out.println(e.getMessage());
        }

        System.out.println("main end"); // 核心代码
    }
}
~~~

### wraper：基本数据类型装箱

##### 数据类型包装类

~~~java
 /* 包装类 : 把基本数据包装成对象的特殊类型. 目的是让基本数据也具有对象的特征,比如调用方法.
 * byte     Byte
 * short    Short
 * int      Integer
 * long     Long
 * float    Float
 * double   Double
 * char     Character
 * boolean  Boolean
~~~

##### 包装对象不兼容

~~~java
 /* 包装类对象之间不可像基本值一样兼容
 * Double不能兼容Float
 * Long不能兼容Integer
~~~

##### 装箱与拆箱方式

~~~java
 /* 装箱 : boxing
 *      基本数据值 => 对象
 *      Integer obj = new Integer(200);
 *
 *      手工装箱
 *      Xxx obj = new Xxx(xxx);
 *      Xxx obj = new Xxx("xxx");
 *      Xxx obj = Xxx.valueOf(xxx);
 *      Xxx obj = Xxx.valueOf("xxx");
 *
 *      自动装箱
 *      Xxx obj = xxx;
 *
 * 拆箱 : unboxing
 *      对象 => 基本数据值
 *      int n = obj.intValue();
 *
 *      手工拆箱
 *      xxx = obj.xxxValue();
 *      自动拆箱
 *      xxx = obj;
 *
 * String => 基本数据类型
 *      String s = "xxx";
 *      xxx = Xxx.parseXxx(s);
 *
 * 基本数据类型 => String
 *      String s = "" + xxx;
~~~

##### 实例代码

~~~java
package com.atguigu.javase.test;

import org.junit.Test;
public class WrapperTest {

    @Test
    public void test3() {
        Boolean obj1 = new Boolean(false);
        Boolean obj2 = new Boolean("true");
        Boolean obj3 = Boolean.valueOf(true);
        Boolean obj4 = Boolean.valueOf("false");

        System.out.println("obj1 = " + obj1);
        System.out.println("obj2 = " + obj2);
        System.out.println("obj3 = " + obj3);
        System.out.println("obj4 = " + obj4);

        // 拆箱
        boolean b1 = obj1.booleanValue();
        boolean b2 = obj2;

        String s1 = "true";
        // String => boolean值
        boolean b3 = Boolean.parseBoolean(s1);
        // 基本值 => String
        // "" + 基本值
        String s2 = "" + b3;

        Object[] arr = {3.22, 24.4f, 234L, 342, false, 'c'};
    }

    @Test
    public void test2() {
        Double obj1 = new Double(3.22); // 手工装箱
        Double obj2 = new Double("234234.234234");
        Double obj3 = Double.valueOf(2.322);
        Double obj4 = Double.valueOf("234234.234");

        Double obj5 = 9234.23423; // 自动装箱

        //Double obj6 = 324; 不可以
        //Double obj7 = 2.23f; 不兼容
        Float obj8 = 3.88f;
        double v1 = obj1.doubleValue(); // 手工拆箱
        double v2 = obj2; // 自动拆箱
        String s1 = "234234.23423";
        // String => double值
        double v3 = Double.parseDouble(s1);
        System.out.println(v3);
    }

    @Test
    public void test1() {
        Integer obj1 = new Integer(50); // 手工装箱
        Integer obj2 = 500; // 自动装箱 , Integer obj2 = new Integer(500);
        int n1 = obj1.intValue(); // 手工拆箱
        int n2 = obj2; // 自动拆箱

        Object obj3 = 300;
        System.out.println(obj1);
        System.out.println(obj2);
        System.out.println(n1);
        System.out.println(n2);

        String s1 = "234234";
        // 工具方法, String => int
        int n3 = Integer.parseInt(s1);
        // int = > String
        int n4 = 32842;
        String s2 = "" + n4;

        if (obj1 == 50) { // obj.intValue() == 50
            System.out.println("llllll");
        }

        if (obj2 > 30) { // 自动拆箱

        }
    }
}
~~~

### String类【重点】

##### Strin简介

~~~Java
 /* 字符串 是最重要的类, 没有之一.
 * 字符串就是内容不可改变的Unicode字符序列. 字符串在字符串中的位置索引是固定的.
 * 任何的对于字符串内容的修改都必须会产生新对象. 如果要频繁修改字符串内容, 效率非常低.
 * 字符串 底层就是一个char[]
 *
 * "askdlfj我和234234" => {'a','s','k'.....}
 * String s = "abc";
 * s += 200; // "abc200"
 *
 * Student s = new Student(1, "小明", 3, 90);
 * s.setScore(100); // 内容可以改变
~~~

##### 代码实例

~~~java
package com.atguigu.javase.test;

import org.junit.Test;

/**
 *
 * 所有的字符串常量 都直接保存在 永久区中的常量区中.
 *
 *               0 2   6   10       16         23       29    35 37
 * String str = "  abc XYZ 我喜欢你, 你喜欢我吗? 我不喜欢你 zzz 123  ";
 *
 * public int length() : 获取数组串的长度, 字符数
 *      str.length() => 38
 *
 * public char charAt(int index) : 获取指定下标index位置处的字符  arr[index]
 *      str.charAt(7) => 'Y', str.charAt(25) => '喜'
 *
 * public char[] toCharArray() : 把字符串转换为相应的字符数组, 返回的是内部数组的一个副本
 *      第1个参数是源数组对象, 第2个参数是源数组的开始下标, 第3个参数是目标数组对象, 第4个参数是目标数组的开始下标.
 *      第5个参数是要复制的元素个数
 *      System.arraycopy(value, 0, result, 0, value.length);
 *
 *
 * public boolean equals(Object anObject)
 * public int compareTo(String anotherString)
 * public int indexOf(String str)
 * public int indexOf(String str ,int fromIndex)
 * public int lastIndexOf(String str)
 * public int lastIndexOf(String str ,int fromIndex)
 * public boolean startsWith(String prefix)
 * public boolean endsWith(String suffix)
 * public String substring(int beginIndex,int endIndex)
 * public String substring(int beginIndex)
 * public String replace(char oldChar,char newChar)
 * public String replaceAll(String old,String new)
 * public String trim()
 * public String concat(String str)
 * public String toUpperCase()
 * public String toLowerCase()
 * public String[] split(String regex)
 */
public class StringTest {

    // 练习  : 反转一个字符串
    @Test
    public void exer13() {
        String str = "  abc XYZ 我喜欢你, 你喜欢我吗? 我不喜欢你 zzz 123  ";
        String s2 = "";
        for (int i = 0; i < str.length(); i++) {
            char ch = str.charAt(i);
            s2 = ch + s2;
        }
        System.out.println(s2);
    }

    @Test
    public void exer12() {
        String str = "  abc XYZ 我喜欢你, 你喜欢我吗? 我不喜欢你 zzz 123  ";
        char[] chars = str.toCharArray();
        for (int i = 0; i < chars.length / 2; i++) {
            char tmp = chars[i];
            chars[i] = chars[chars.length - 1 - i];
            chars[chars.length - 1 - i] = tmp;
        }
        String s2 = new String(chars);
        System.out.println(s2);

        System.out.println(str); // 原始字符串有没有被反转??
    }

    @Test
    public void exer1() {
        String str = "  abc XYZ 我喜欢你, 你喜欢我吗? 我不喜欢你 zzz 123  ";
        String s2 = "";
        // 反向遍历字符串
        for (int i = str.length() - 1; i >= 0; i--) {
            // 遍历时把每个字符都串接在s2的后面
            char ch = str.charAt(i);
            s2 += ch;
        }
        System.out.println(s2);
    }


    @Test
    public void test2() {

        //String s1 = "abcd"; // 字符串常量
        final String s1 = "atguigu"; // 常量区
        String s2 = "java"; // 常量区
        String s4 = "java"; // 常量区
        String s3 = new String("java"); // GC区
        System.out.println(s2 == s3); // false
        System.out.println(s2 == s4); // true, 不要错以为字符串就可以这样比较!!!
        System.out.println(s2.equals(s3)); // 比较内容, true

        System.out.println("************************************");
        String s5 = "atguigujava"; // 常量区
        String s6 = s1 + s2; // 字符串拼接时, 如果有变量参与, 它的拼接结果在gc区
        System.out.println(s5 == s6); // false
        System.out.println(s5.equals(s6)); // true

        String s7 = s1 + "java"; // 如果全部都是常量的拼接,结果在常量区
        System.out.println(s5 == s7);

        String s8 = (s1 + s2).intern(); // 把在GC区中的字符串强行放入常量区, 如果常量区中没有此串,则创建新的,
        // 如果有, 则返回已有串的地址.
        // intern方法有可能会导致常量区暴满. java8中保存类模板数据的内存空间独立出来,称为元空间(meta space)
        System.out.println(s5 == s8); // true

        String s9 = new String("yyyzzz");
        //String s10 = new String("yyy");
        String s10 = s9;
        String s11 = new String("yyy" + "zzz");
        System.out.println(s9 == s10);
    }
    
    @Test
    public void test1() {
        // 创建字符串对象
        String s1 = new String(); // String s1 = "";, 空串表示有对象, 但是没有内容.
        String s2 = null; // null表示啥也没有
        String s3 = new String("abc"); // 产生的新对象和参数中的对象不是一回事
        char[] arr = {'a', '我', '1', '3', 'G', 'X', '9', '你'};
        String s4 = new String(arr); // 把字符数组变成字符串
        System.out.println("s4 = " + s4);
        // 第2个参数表示数组的开始下标, 第3个参数表示要选取几个字符
        String s5 = new String(arr, 2, 4); // 字符数组的一部分 创建字符串
        System.out.println("s5 = " + s5);
    }
}
~~~



