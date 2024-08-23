- 集合内容回顾
- hash值的说明

集合框架

- 今日内容
  - 泛型技术介绍
  - 自定义泛型类，方法，接口
  - 泛型通配符
  - Set接口特点
  - HashSet集合特点
  - 哈希表数据结构
  - TreeSet集合特点
  - 红黑树结构

##### 集合内容回顾

1： 集合框架作用
  |-- 容器存储
  |-- 只能存储对象
  |-- 长度可变
  |-- 集合和数组都是内存数据（临时数据）

2. Collection接口（顶级接口）
    |-- add() 添加元素
    |-- size() 集合长度
    |-- remove() 元素删除
    |-- clear() 清空集合中的所有元素
    |-- iterator() 返回迭代集合的迭代器

3. List接口  继承 Collection接口
    |-- 特点：**有序，索引，重复**  三大特点
    |-- 该接口任何实现类都具备三大特点
    |-- add() 带索引添加
    |-- remove()带索引移除
    |-- get() 指定索引获取元素
    |-- set() 带索引修改元素

4. ArrayList类  实现List接口
    |-- **有序，索引，重复**
    |-- 底层是数组实现
    |-- 查询快，增删速度慢
    |-- **初始化容量10个长度，每次增长1.5倍**
    |-- 线程不安全的集合，运行速度快

5. LinkedList类 实现List接口
    |-- **有序，索引，重复**
    |-- 底层是链表结构实现
    |-- 查询慢，增删速度快
    |-- 线程不安全的集合，运行速度快

6. Iterator接口 遍历集合的迭代器
    |-- hasNext() 判断是否有下一个元素
    |-- next() 取出下一个元素
    |-- remove() 遍历到哪个元素，就删除哪个元素
    |-- 任何一个集合类，都有一个方法 iterator() 获取接口Iterator实现类
    
7. HashSet：

    |-- 底层数数组

    |-- 默认长度16， 扩表因子0.75，   扩展增大2倍， 最大值： << 30  = 1073741824

# 第一章 泛型

## 1.1 泛型技术介绍

泛型（Generic）技术是JDK1.5版本出现的新特性，泛型是一项安全技术。在没有泛型技术之前，使用集合容器是不安全的。

```java
    public static void main(String[] args) {
        //创建不带泛型的集合容器
        List list = new ArrayList();
        //集合中可以存储的数据，是任意类型
        list.add("abc");
        list.add("haha");
        list.add("hehe");
        list.add("xixi");
        list.add("zizi");
        list.add(1);
        //迭代器遍历集合
        Iterator it = list.iterator();
        while (it.hasNext()){
            //取出集合的元素，存储的时候是任意类型
            //取出的时候，也是任意类型
            String obj = (String)it.next();
            System.out.println("obj = " + obj.length());
        }
    }
//程序执行，产生类型转换异常
```

- 泛型技术，就是在集合类的旁边添加<集合要存储的数据类型>

  - 强制集合存储指定的数据类型，存储了其他类型，编译失败

  - 已经强制集合存储指定的类型，迭代器遍历的时候，无需强制转换

  - > 泛型技术，将程序的安全问题，提前暴露在编译时期

## 1.2 泛型技术的标准写法 （非常重要）

```properties
集合类<集合存储的数据类型> 对象名 = new 集合类<集合存储的数据类型>();
//从JDK1.7版本开始，后面的泛型可以不写，减少代码量，钻石操作符
集合类<集合存储的数据类型> 对象名 = new 集合类<>();
//注意事项，泛型前后必须一致
```

## 1.3 API文档的E问题

- Collection<E>
- List<E>
- ArrayList<E>

```java
E：其实就是一个变量名,这种变量接收的不是一个数值，而是一个数据类型
ArrayList<String> array = new ArrayList<>();
注意：创建对象，传递String类型， E --> 变成String类型了

//add存储元素，参数类型是E
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
//get，索引获取元素 返回值是E
public E get(int index) {
	rangeCheck(index);
	return elementData(index);
}
```

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05101.png)



## 1.4 自定义泛型类

```java
/**
 * 自定义的泛型类
 * 模仿ArrayList
 *
 * <里面就是变量名>
 *     QQ  或者是 E 变量名，等着接收一个数据类型
 *
 *  Factory<String> factory = new Factory<>();
 *  QQ 全部变成String类型
 */
public class Factory<QQ> {
    private QQ name;

    public QQ getName() {
        return name;
    }

    public void setName(QQ name) {
        this.name = name;
    }
}
```

```java
   public static void main(String[] args) {
        //创建对象 Factory
        Factory<String> factory = new Factory<>();
        factory.setName("abc");
        String name = factory.getName();
        System.out.println("name = " + name);

        //创建对象 Factory
        Factory<Double> factory2 = new Factory<>();
        factory2.setName(1.5);
        Double d = factory2.getName();
        System.out.println("d = " + d);
    }
```

```java
    /**
     * 定义静态方法
     * 是静态不能直接引用非静态
     * 静态方法中的泛型，不能和类上一样
     * 静态方法自己定义泛型
     * 静态方法的泛型：传递什么，就是什么
     */
    public static <E> void print(E q){
        System.out.println(q);
    }
```

## 1.5 泛型接口

```java
/**
 * 自定义的泛型接口
 */
public interface MyInterface <E>{
    //方法上的泛型，跟随接口走
    public abstract void inter(E e);
}
```

```java
/**
 * 实现接口，指定泛型
 */
public class MyInterfaceImpl implements MyInterface<String>{
    @Override
    public void inter(String s) {
        System.out.println("实现类重写方法"+s);
    }
}

```

```java
    public static void main(String[] args) {
        //创建接口实现类对象
        MyInterfaceImpl my = new MyInterfaceImpl();
        my.inter("haha");
    }

```

## 1.6 泛型的通配符

```java
   public static void main(String[] args) {
        //定义集合，存储字符串
        List<String> stringList = new ArrayList<>();
        stringList.add("abc");
        stringList.add("bcd");

        //定义集合，存储整数
        List<Integer> integerList = new ArrayList<>();
        integerList.add(123);
        integerList.add(456);

        //调用方法遍历集合
        print(stringList);
        print(integerList);
    }
    /**
     *  定义方法：可以同时迭代两个集合
     *  选择使用泛型的通配符 ?  可以通配任意的类型
     */
    public static void print(List<?> list){
        Iterator<?> it = list.iterator();
        while (it.hasNext()){
            // ? 可以通配任何类型，但是取出集合元素
            // 迭代器方法 next() 返回值只能是Object
            Object obj = it.next();
            System.out.println("obj = " + obj);
        }
    }

```

## 1.7 for循环

增强型的for循环，出现在JDK1.5版本，主要作用就是减少代码编写量

`java.lang.Iterable`接口：实现这个接口，可以使用增强型的for循环 (包括数组)

Collection接口，List接口，Set接口，都是这个接口的子接口，集合都可以使用

语法格式：

```java
for (数据类型 变量名 : 集合或者数组){
    
}

```

循环是一个编译特效：集合编译为迭代器，数组编译为传统的for写法

# 第二章 Set接口

## 2.1 Set接口介绍

- `java.util.Set`接口：集

  - Set接口继承Collection接口，  List和Set是兄弟关系
  - Set系列的集合**不能包含重复元素**的
  - Set系列的集合**没有索引**的

  - 该接口的所有实现类，都具有上述2个特征

- Set接口方法：和父接口Collection的方法完全是一样的！！

```java
public static void main(String[] args) {
        //创建Set接口的集合
        Set<String> set = new HashSet<>();
        set.add("abc");
        set.add("bcd");
        set.add("def");
        set.add("hello");

        Iterator<String> it = set.iterator();
        while (it.hasNext()){
            System.out.println(it.next());
        }
        System.out.println("===============");
        for (String s : set){
            System.out.println("s = " + s);
        }
    }

```

## 2.2 HashSet集合特点 （非常重要）

- HashSet类 ： 实现Set接口

  - 底层数据结构是哈希表，单向链表
  - 是无序的集合，元素存储的顺序和取出的顺序不一致
  - 不同步的，线程不安全运行速度快

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05103.png)

- HashSet集合存储字符串

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05104.png)

> 向哈希表集合中，存储元素的时候，集合怎么判定是不是重复的元素
>
> 关键点在于：对象自己的hashCode()值，和equals方法的返回值
>
> 两个对象哈希值相同，equals方法返回true，就是重复元素

```java
    /**
     * HashSet集合存储字符串，遍历
     * = new HashSet<>(); 数组默认长度16，加载因子是0.75
     * = new HashSet<>(30,0.9F); 数组指定长度30，加载因子0.9
     */
    public static void testHashSetString(){
        Set<String> set = new HashSet<>();
        set.add("abc");
        set.add(new String("abc"));
        set.add("通话");
        set.add("重地");
        for (String str : set){
            System.out.println("str = " + str);
        }
    }

```


## 2.3 对象的哈希值（很重要）

每一个类都可以new对象，每一个类又继承Object类：

Object类中定义方法：

```java
/**
   native 修饰符：本地意思
   本地方法：方法是不开源的，方法的源码是C++语言编写
   本地方法的运行是在一个独立内存中：本地方法栈 
   本地方法的作用，调用操作系统的功能
   方法对象的哈希值
*/
public native int hashCode();

```

任何一个子类对象，都可以调用该方法

- toString方法源码

```java
public String toString(){
    return getClass().getName()+"@"+Integer.toHexString( hashCode() );
}
//getClass().getName() 执行结果是类名  com.atguigu.hashcode.Student
//hashCode() 返回对象的哈希值，int类型
//Integer.toHexString( 哈希值 ) 十进制转成十六进制  1b6d3586

```

> 哈希值，就是hashCode()方法，计算出来一个int类型整数而已
>
> 每个对象，都会具有的一个值，JVM赋予每个对象的一个int值

- 自己定义自己的哈希值：重写父类的方法

```java
/**
  * 重写父类的方法 hashCode()
  */
public int hashCode(){
    return 9527;
}

```

- String类对象的哈希值

String类继承Object类，肯定会有哈希值。String类继承Object，重写父类的方法hashCode，定义String对象自己的哈希值！

String类哈希值的计算方式：31*上一次哈希值的结果+字符串中单个字符的ASCII码

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05102.png)

> 为什么要乘以31呢？31是质数，除了1个自己，比他小的都不能整除
>
> 乘以31：降低概率，字符串内容不同，但是计算出的哈希值是相同的！！

```java
    /**
     * String字符串对象哈希值
     * 任何对象都具有哈希值
     *   String s1 = "abc";
     *   String s2 = new String("abc");
     *   没有区别！！
     */
    public static void stringHashCode(){
        String s1 = "abc";
        String s2 = new String("abc");
        System.out.println(s1 == s2);//false  比较内存地址

        //s1对象的哈希值和s2对象的哈希值是一样，哈希值是int类型
        System.out.println(s1.hashCode() == s2.hashCode());//true
        System.out.println(s1.hashCode());// 96354
        System.out.println(s2.hashCode());// 96354

        System.out.println("=============");

        String s3 = "通话";
        String s4 = "重地";
        System.out.println(s3.hashCode()); // 1179395
        System.out.println(s4.hashCode()); // 1179395
        System.out.println(s3.equals(s4)); //false

    }

```

## 2.4 哈希表源码（提高）

HashSet集合，new的时候，里面new另一个对象 HashMap，HashSet类自己没有什么功能， 依靠的是另一个集合HashMap的实现的功能。

```java
//哈希表，数组默认的长度 = 16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
//数组的最大容量 1 << 30  = 1073741824
static final int MAXIMUM_CAPACITY = 1 << 30;
//数组的扩容指标 阈值，加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
内部类：
 static class Node<K,V> {
        final int hash;//对象哈希值
        final K key; // Key 集合中存储的元素
        Node<K,V> next;//下一个存储的对象
 }
 //存储原理 ： Node<K,V>[] tab  对象数组

```

## 2.5 HashSet集合存储自定义对象

```java
   /**
     * HashSet存储自定义对象Student
     * set.add( new Student("李四",21));
     * set.add( new Student("李四",21));
     * 内存的角度上看：堆内存中的两个对象
     * Student对象：如果姓名和年龄一样的，认为是同一个对象
     * Student类：重写父类的方法 hashCode 和 equals
     */
    public static void testHashSetStudent(){
        //创建HashSet集合
        Set<Student> set = new HashSet<>();
        set.add( new Student("张三",20));
        set.add( new Student("李四",21));
        set.add( new Student("李四",21));
        set.add( new Student("王五",23));
        set.add( new Student("赵六",22));
        for (Student stu : set){
            System.out.println("stu = " + stu);
        }
    }

```

## 2.6 树形结构

- 二叉树

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05105.png)

- 自然平衡二叉树，对树的节点加以染色，变成了红黑树结构

红黑树，是一种内存的树形结构

- 节点是红色或者黑色
- 根节点是黑色
- 每个叶子的节点都是黑色的空节点（NULL）
- 每个红色节点的两个子节点都是黑色的。
- 从任意节点到其每个叶子的所有路径都包含相同的黑色节点数量。

https://www.cs.usfca.edu/~galles/visualization/RedBlack.html 

## 2.7 TreeSet集合特点（非常重要）

- `java.util.TreeSet`类：实现Set接口
  - 底层是由TreeMap来实现的
  - 底层是**红黑树结构**
  - 查找速度快，增删慢
  - 线程不安全的集合，运行速度快
  - TreeSet会对存储的元素，进行自然顺序的排序 abc  123
  - 是无序的集合

- TreeSet集合存储字符串

```java
    /**
     * TreeSet集合存储字符串
     * 按照字符串的自然顺序排序： abcd
     */
    public static void testTreeSetString(){
        Set<String> set = new TreeSet<>();
        set.add("hello");
        set.add("world");
        set.add("java");
        set.add("c++");
        for (String str : set){
            System.out.println("str = " + str);
        }
    }

```

- TreeSet集合存储字自定义对象

  ![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05106.png)

  ```java
      /**
       * TreeSet集合存储自定义对象Student
       */
      public static void testTreeSetStudent(){
          Set<Student> set = new TreeSet<>();
          set.add( new Student("李四",21));
          set.add( new Student("张三",20));
          set.add( new Student("赵六",22));
          set.add( new Student("王五",23));
          for(Student stu : set){
              System.out.println("stu = " + stu);
          }
      }
  
  ```

```java
Exception in thread "main" java.lang.ClassCastException: com.atguigu.pojo.Student cannot be cast to java.lang.Comparable
	at java.util.TreeMap.compare(TreeMap.java:1294)
	at java.util.TreeMap.put(TreeMap.java:538)
	at java.util.TreeSet.add(TreeSet.java:255)
	at com.atguigu.set.TreeSetDemo.testTreeSetStudent(TreeSetDemo.java:20)
	at com.atguigu.set.TreeSetDemo.main(TreeSetDemo.java:13)


```

> Student对象存储在TreeSet集合中，集合要对存储的元素排序，根据存储的Student对象的自然顺序进行排序，但是我们的Student没有自然顺序，因此出现了该异常。
>
> 必须让Student具备自然顺序

```java
    /**
     * 重写方法compareTo
     * 返回int类型
     * this对象和参数s对象比较
     * 两个对象的年龄差
     *  set.add( new Student("李四",21)); == 参数 s   已有的
     *  set.add( new Student("张三",20)); == this 对象 后来的
     *  this.age - s.age; 负数，调用者小，后来的对象小，排前面
     */
    public int compareTo(Student s){
        return this.age - s.age;
    }

```

- TreeSet排序的比较器

TreeSet集合对元素排序有2个方式，第一个是元素的自然顺序（Comparable接口实现），第二个是比较器，按照比较器的比较结果来排序。比较器的优先级高

`java.util.Comparator`接口：比较器接口

自定义类，实现接口，重写方法 comare

```java

```

```java
/**
 * TreeSet集合存储自定义对象Student
 */
public static void testTreeSetStudent(){
    //创建TreeSet集合是无参数构造方法，使用元素的自然顺序
    //Set<Student> set = new TreeSet<>();

    //创建TreeSet集合，传递比较器接口实现类的对象，使用比较器排序
    Set<Student> set = new TreeSet<>( new MyComparator() );
    set.add( new Student("李四",21));
    set.add( new Student("张三",20));
    set.add( new Student("赵六",22));
    set.add( new Student("王五",23));
    for(Student stu : set){
        System.out.println("stu = " + stu);
    }
}

```

- 能总结下Comparator和Compareble接口区别么
  - Comparator接口和Compareble接口都是为集合TreeSet准备排序的
  - Compareble接口，是集合中存储对象，自己实现的接口 Student类实现接口
    - 对象的自然顺序
  - Comparator接口，单独定义了类，实现的接口
    - 自定义比较器
  - Student类为例：
    - 如果这个类Student是我们自己写的，用Compareble，自然顺序
    - 类Student不是我们写的，其他同事的，使用比较器排序



