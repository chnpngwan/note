##### 课堂笔记

- test-Wrapper---

- 包装类

- 字符串常量，在常量区中

  - ~~~java
    /*class Test{
        toChararr()：System.arrcopy();
        equse（）
        #####indexof（）：
        startWith()
        endWith()
        #####subsstring(int begin, int endbegin): [start, end) //结束索引不包含
        replace(old, new)  //  替换字符【替换所有的字符|字符串】
        replaceall()：  //支持正则表达式替换
        trim()：  //去除字符串首尾逇空白字符
        concate（）：字符串连接
        splite（）： //切割字符串
        String.valueof(....)	// 将任意类型转换为字符串类型
    }
    */
    ~~~

- StringBuder():

  - 链式操作

  ~~~java
  /*
  	append（）
  	insert（）
  	可扩容操作步骤，lenth*2 + 2 (防止扩容失败）
  
  */
  ~~~
  
- Date类

  #### 装箱与拆箱
  
  ##### 笔记
  
  ~~~java
  
  /**
   * 包装 : 把基本数据包装在对象中.
   * byte         Byte
   * short        Short
   * int          Integer
   * long         Long
   * float        Float
   * double       Double
   * char         Character
   * boolean      Boolean
   *
   * 装箱 : 基本数据 => 对象
   *      手工装箱 :
   *          Xxx obj = new Xxx(xxx);
   *      自动装箱
   *          Xxx obj = xxx;
   * 拆箱 : 对象 => 基本数据
   *      手工拆箱 :
   *          xxx = obj.xxxValue();
   *      自动拆箱
   *          xxx = obj;
   * 字符串相关 :
   *      String => 基本值
   *          String str = "xxx";
   *          xxx = Xxx.parseXxx(str);
   *      基本值 => String
   *          String str = "" + xxx; // 野路子
   *          String str = String.valueOf(xxx); 正规做法.
   */
  ~~~
  
  ##### 代码
  
  ```java
  package com.atguigu.javase.test;
  
  import org.junit.Test;
  public class WrapperTest {
  
      @Test
      public void method1() {
          Integer i = new Integer(1);
          Integer j = new Integer(1);
          System.out.println(i == j); // 比较地址 : false
  
          Integer m = 1; // 自动装箱时, 如果数据在[-128~127], 取缓冲区中的现成的对象
          Integer n = 1;
          System.out.println(m == n);
  
          Integer x = 128; // 128大于127, 所以装箱时会直接new Integer(...)
          Integer y = 128;
          System.out.println(x == y);
      }
  
      @Test
      public void test1() {
          Integer obj1 = new Integer(200);
          Integer obj2 = 100;
          Double obj3 = 3.22;
          Float obj4 = 92.22f;
          Long obj5 = 34234L;
  
          int i = obj1.intValue();
          double v = obj3.doubleValue();
          float f = obj4;
          long l = obj5;
      }
  }
  
  ```
  
  #### 字符串
  
  ~~~java
  /**
   * 字符串 是最重要的类, 没有之一.
   * 字符串就是内容不可改变的Unicode字符序列. 字符串在字符串中的位置索引是固定的.
   * 任何的对于字符串内容的修改都必须会产生新对象. 如果要频繁修改字符串内容, 效率非常低.
   * 字符串 底层就是一个char[]
   *
   *               0 2   6   10       16         23       29    35 37
   * String str = "  abc XYZ 我喜欢你, 你喜欢我吗? 我不喜欢你 zzz 123  ";
   *
   * ##public int length() : 获取数组串的长度, 字符数
   *      str.length() => 38
   *
   * ##public char charAt(int index) : 获取指定下标index位置处的字符  arr[index]
   *      str.charAt(7) => 'Y', str.charAt(25) => '喜'
   *
   * public char[] toCharArray() : 把字符串转换为相应的字符数组, 返回的是内部数组的一个副本
   *      第1个参数是源数组对象, 第2个参数是源数组的开始下标, 第3个参数是目标数组对象, 第4个参数是目标数组的开始下标.
   *      第5个参数是要复制的元素个数
   *      System.arraycopy(value, 0, result, 0, value.length);
   *
   * ###public boolean equals(Object anObject) : 比较两个字符串的内容是否相f等, 这是比较字符串必须要使用, 不可以使用==
   * public boolean equalsIgnoreCase(String other) : 比较两个字符串的内容是否相等, 忽略大小写.
   * public int compareTo(String anotherString) : 比较大小
   * public int compareToIgnoreCase(String anotherString) : 比较大小, 忽略大小写
   *
   *               0 2   6   10       16         23       29    35 37
   * String str = "  abc XYZ 我喜欢你, 你喜欢我吗? 我不喜欢你 zzz 123  ";
   * #####
   * public int indexOf(String str) : 获取参数中的子串在当前串中首次出现的下标
   *          str.indexOf("喜欢") => 11
   * public int indexOf(String str ,int fromIndex) : 获取参数中的子串在当前串中首次出现的下标, 从指定下标fromIndex开始检索
   *          str.indexOf("喜欢", 12) => 17
   *          str.indexOf("喜欢", 18) => 25
   *          str.indexOf("喜欢", 26) => -1
   *
   * public int lastIndexOf(String str) : 获取参数中的子串在当前串中首次出现的下标, 从右向左检索
   *          str.lastIndexOf("喜欢") => 25
   * public int lastIndexOf(String str ,int fromIndex)
   *          str.lastIndexOf("喜欢", 24) => 17
   *          str.lastIndexOf("喜欢", 16) => 11
   *          str.lastIndexOf("喜欢", 10) => -1
   *
   *               0 2   6   10        16         23       29    35 37
   * String str = "  abc XYZ 我喜欢你, 你喜欢我吗? 我不喜欢你 zzz 123  ";
   * public boolean startsWith(String prefix) : 判断当前串是否以参数指定的子串为开始
   *          str.startsWith("  abc") => true
   * public boolean endsWith(String suffix) : 判断当前串是否以参数指定的子串为结束
   *          str.endsWith("zzz 123") => false
   *
   * #####
   * public String substring(int beginIndex,int endIndex) : 从当前串中取子串, 从beginIndex(包含)开始到endIndex(不包含)结束
   *          str.substring(10, 21) => "我喜欢你, 你喜欢我吗", 子串长度 = 结束索引-开始索引
   *          str.substring(23, str.length()); 从23开始, 取到末尾
   * public String substring(int beginIndex) : 取子串, 从beginIndex(包含)取到末尾
   *          str.substring(23);
   *
   * ###
   * public String replace(char oldChar,char newChar) : 替换字符串中所有的oldChar为newChar
   *          str.replace(' ', '#');
   * public String replace(CharSequence old, CharSequence replacement) : 替换字符串中所有的old子串为新串replacement
   *
   * public String replaceAll(String old,String new) :  替换全部字符串, 支持正则表达式(专门用于字符串匹配)
   *
   * public String trim() : 修剪字符串首尾的空白字符(码值小于等于32), 产生新串
   *          str.strim() => "abc XYZ 我喜欢你, 你喜欢我吗? 我不喜欢你 zzz 123"
   *
   * public String concat(String str) : 字符串连接, 和 + 一样的效果
   * public String toUpperCase() : 变大写字母
   * public String toLowerCase() : 变小写字母
   *
   * public String[] split(String regex) : 以参数中的子串为切割器, 把字符串切割成多个部分, 切割完成后的切割器就被丢弃.
   *
   * ##
   * public static String valueOf(...) : 把任意数据转换为相应的字符串内容
   */
  ~~~
  
  
  
  
  
  #### StringBuder
  
  ~~~java
  /**
   * StringBuilder : 内容可以改变的Unicode字符序列, 内部也是使用一个char[]来保存所有字符
   * 它像一个容器,里面有大量的字符, 可以保存任意多个字符.
   * <p>
   * StringBuilder append(...) : 在当前串的末尾追加新的内容, 内部内容就会变化. 为了和字符串的+操作保持一致.
   * StringBuilder insert(int index, ...) : 在指定下标index位置处插入任意内容.
   * StringBuilder delete(int beginIndex, int endIndex) : 删除从beginIndex开始到endIndex结束的区间.
   * void setCharAt(int index, char newCh) : 替换指定下标index位置的字符为新字符newCh
   * <p>
   * StringBuffer 是老的API, 效率低, 线程安全
   * StringBuilder 是新的API, 效率高, 线程不安全(首选)
   */
  ~~~
  
  
  
  