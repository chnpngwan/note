##### 课堂笔记

- 数组的问题
- 集合-新容器对象`java.util.List`接口：继承Collection接口
  - `java.util.ArrayList`类（数组列表）：重写List与Clecction中的方法
  - `java.util.LinkedList`：（链表）：重写List与Clecction中的方法
  - `java.util.Set`：（）继承Cllection接口
- 迭代器：
- Alt+7：查看类中的方法

集合框架

- 今日内容
  - 集合框架介绍
  - 集合与数组的区别
  - Collection接口使用
  - 迭代器Iterator接口
  - List接口特点
  - 数据结构介绍
  - ArrayList集合
  - LinkedList集合

# 第一章 集合介绍

## 1.1 数组存在问题

Java中的数组，存在一个非常严重的问题，就是长度恒定，一旦创建好，就不能在变！

定长数组：开发人员有的时候，无法确定数组的长度！

为什么解决数组的长度问题，Java从1.2版本开始，出现了新的容器 -- 集合框架。集合容器是长度随意可变的，开发人员无需为了长度而苦恼。

## 1.2 集合框架

从1.2版本开始，出现了一套新的容器对象：集合框架

所有的集合内容都在java.util包

- `java.util.Collection`接口：集合中的**根**接口
  - 任何集合中的容器，都继承该接口，或者实现该接口
  - Collection接口中的方法：应该是所有集合容器中的共性方法！！
- `java.util.List`接口：列表
  - List接口，继承Collection接口
- `java.util.ArrayList` 类：List接口的实现类，数组列表
  - ArrayList类重写接口List和Collection的抽象方法
- `java.util.LinkedList`：List接口的实现类，链表
  - LinkedList类重写接口List和Collection的抽象方法

- `java.util.Set`接口：集
  - Set接口，继承Collection接口
- `java.util.HashSet`类：实现接口Set，哈希表
  - HashSet类重写Set接口和Collection接口的方法
- `java.util.TreeSet`类：实现接口Set，树
  - TreeSet类重写Set接口和Collection接口的方法

> 以上都是我们还要学习的集合容器类：每一个类都是可以存储数据的容器
>
> 每一个容器本质上又不同

## 1.3 集合和数组面试题

集合容器和数组容器的区别：

1. 集合和数组都是容器 （共同点）
2. 集合长度可变，数组长度是固定的 （差异点）
3. 集合只能存储引用类型，不存储基本数据类型，数组可以存储引用类型和基本类型（差异点）

# 第二章 Collection接口 （非常的重要）

## 2.1 Collection接口特点

是所有集合中的根，其他的集合容器都是他的子接口或者是他的实现类，最共性的方法

Collection毕竟是接口，找接口的实现类，new对象，使用实现类ArrayList创建对象

```properties
集合容器对象的创建：格式
Collection<集合容器存储的数据类型> 对象名 = new ArrayList<集合容器存储的数据类型>();
```

## 2.2 Collection接口的方法

- boolean add(要存储的数据) 向集合容器中，存储数据

```java
/**
  * - boolean add(要存储的数据) 向集合容器中，存储数据
  * 集合对象使用的是实现类ArrayList（重写）
  */
public static void testCollectionAdd(){
    //创建集合对象，接口Collection和实现类ArrayList
    //集合容器存储的数据是字符串类型
    Collection<String> col = new ArrayList<>();
    //col对象，调用方法add()向集合中存储元素
    col.add("hello");
    col.add("java");
    col.add("hadoop");
    col.add("zookeeper");
    //输出集合对象 col 调用对象的方法toString()
    //将集合容器中存储的元素打出，不是遍历
    System.out.println( col );
}
```

- void clear() 清空集合中的所有元素

```java
 /**
     * void clear() 清空集合中的所有元素
     * 没有毁掉容器，可以继续存储
     */
     public static void testCollectionClear(){
         //创建集合对象，存储的数据是整数
         Collection<Integer> col = new ArrayList<>();
         //col对象的方法add，存储整数
         col.add(1); //整数1，基本类型，自动装箱
         col.add(2);
         col.add(3);
         col.add(4);
         col.add(5);
         System.out.println(col);
         //col对象调用方法 clear() 清空集合中的元素
         col.clear();
         System.out.println(col);
     }
```

- boolean contains(元素) 判断这个元素是否存在于集合中，如果存在返回true

```java
    /**
     * boolean contains(元素) 判断这个元素是否存在于集合中，如果存在返回true
     */
    public static void testCollectionContains(){
        //创建集合对象，接口Collection和实现类ArrayList
        //集合容器存储的数据是字符串类型
        Collection<String> col = new ArrayList<>();
        //col对象，调用方法add()向集合中存储元素
        col.add("hello");
        col.add("java");
        col.add("hadoop");
        col.add("zookeeper");
        //col集合对象的方法 contains 判断 java元素，是否存在
        boolean b = col.contains("java");
        System.out.println(b);
    }
```

- boolean isEmpty() 判断集合的长度，是不是0，长度是0，返回true

```java
    /**
     * boolean isEmpty() 判断集合的长度，是不是0，长度是0，返回true
     */
    public static void testCollectionIsEmpty(){
        //创建集合对象，接口Collection和实现类ArrayList
        //集合容器存储的数据是字符串类型
        Collection<String> col = new ArrayList<>();
        //col对象，调用方法add()向集合中存储元素
       /* col.add("hello");
        col.add("java");
        col.add("hadoop");
        col.add("zookeeper");
        col.clear();*/

        //调用col对象的方法 isEmpty()
        boolean b = col.isEmpty();
        System.out.println(b);
        //System.out.println(" ".isEmpty());
    }
```

- int size() 返回集合的长度，集合中元素的个数

```java
/**
     * int size() 返回集合的长度，集合中元素的个数
     * Java中的所有长度的表示方式到齐！
     * 1：数组.length   数组的元素个数
     * 2: 字符串.length() 方法  字符串中的字符个数
     * 3：集合.size() 方法，集合容器中的元素的个数
     */
    public static void testCollectionSize(){
        Collection<String> col = new ArrayList<>();
        //col对象，调用方法add()向集合中存储元素
        col.add("hello");
        col.add("java");
        col.add("hadoop");
        col.add("Linux");
        col.add("zookeeper");
        //调用col对象的方法 size() 获取集合长度
        int size = col.size();
        System.out.println(size);
    }
```

- boolean remove(元素) 移除集合中指定元素，移除成功返回true

```java
    /**
     * boolean remove(元素) 移除集合中指定元素，移除成功返回true
     */
    public static void testCollectionRemove(){
        Collection<String> col = new ArrayList<>();
        //col对象，调用方法add()向集合中存储元素
        col.add("hello");
        col.add("java");
        col.add("hadoop");
        col.add("Linux");
        col.add("java");
        col.add("zookeeper");
        //打印集合元素
        System.out.println("删除前："+col);
        //col对象的方法remove，删除java元素
        boolean b = col.remove("java");
        System.out.println(b);
        //打印集合元素
        System.out.println("删除后："+col);
    }
```

- Object[] toArray() 集合中的元素，转成数组

```java
    /**
     * Object[] toArray() 集合中的元素，转成数组
     */
    public static void testCollectionToArray(){
        Collection<String> col = new ArrayList<>();
        //col对象，调用方法add()向集合中存储元素
        col.add("hello");
        col.add("java");
        col.add("hadoop");
        col.add("Linux");
        col.add("java");
        col.add("zookeeper");
        //col对象的方法toArray() 集合转数组
        Object[] array =col.toArray();
        //数组遍历
        for (Object o : array){
            System.out.println(o);
        }
    }
```

# 第三章 迭代器 （非常重要）

## 3.1 迭代器介绍

迭代器（Iterator）：迭代器这个对象，是负责对集合容器的遍历的。

> 迭代器对象：是所有Collection集合的通用遍历模式

- 接口：`java.util.Iterator` ：表示迭代器对象
  - 接口方法：boolean hasNext() 如果集合中有元素可以遍历，返回true
  - 接口方法：next() 获取集合中的下一个元素

- 接口的实现类：
  - 获取接口Iterator实现类的对象，通过集合对象的方法获取
  - 集合对象方法： `Iterator iterator()`  获取

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1.png)



## 3.2 迭代器遍历集合

```java
    /**
     * 迭代器遍历集合
     * 集合遍历步骤：
     *   1: 集合对象的方法 iterator() 获取迭代器接口Iterator的实现类对象
     *   2：Iterator接口实现类对象调用方法 hasNext() 判断有没有元素
     *   3：Iterator接口实现类对象调用方法 next() 取出元素
     */
    public static void testIterator(){
        Collection<String> col = new ArrayList<>();
        //col对象，调用方法add()向集合中存储元素
        col.add("hello");
        col.add("java");
        col.add("hadoop");
        col.add("Linux");
        col.add("zookeeper");
        //1: 集合对象的方法 iterator() 获取迭代器接口Iterator的实现类对象
        Iterator<String> it = col.iterator();
        //2：Iterator接口实现类对象调用方法 hasNext() 判断有没有元素
       while ( it.hasNext() ) {
           //3：Iterator接口实现类对象调用方法 next() 取出元素
           String str = it.next();
           System.out.println("str = " + str);
       }
    }
```

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/2.png)

## 3.3 迭代器的for

```java
    /**
     *  遍历集合for写法
     */
    public static void testIteratorFor(){
        Collection<String> col = new ArrayList<>();
        //col对象，调用方法add()向集合中存储元素
        col.add("hello");
        col.add("java");
        col.add("hadoop");
        col.add("Linux");
        col.add("zookeeper");
        Iterator<String> it = col.iterator();
        while (it.hasNext()){
            String str = it.next();
            System.out.println(str);
        }
        System.out.println("===============");
        //迭代器经过while循环后，指针已经走到最后了
        //新建迭代器才可以
        for ( Iterator<String> it2 =col.iterator(); it2.hasNext() ; ){
            System.out.println(it2.next());
        }
    }
```

> 区别：for写法相比于while，节约内存。迭代器对象的获取在for里面，for循环结束，迭代器对象也就会从内存中消失。 while写法，迭代器对象创建在方法里面，只有方法结束了，对象才会消失

## 3.4 一次遍历两次next()方法的错误

```java
    /**
     * 集合遍历的时候，调用多次next方法是否可以
     * 不可以
     */
    public static void testIteratorNext(){
        Collection<String> col = new ArrayList<>();
        //col对象，调用方法add()向集合中存储元素
        col.add("hello");
        col.add("java");
        col.add("hadoop");
        col.add("Linux");
        col.add("zookeeper");
        Iterator<String> it = col.iterator();
        while (it.hasNext()){
            String str = it.next();
            System.out.println( it.next());
        }
    }
```



```java
//异常，是没有元素被取出，异常是RuntimeException异常类的子类
//运行时异常，一旦发生，改代码吧
Exception in thread "main" java.util.NoSuchElementException
	at java.util.ArrayList$Itr.next(ArrayList.java:862)
	at com.atguigu.iterator.IteratorDemo.testIteratorNext(IteratorDemo.java:29)
	at com.atguigu.iterator.IteratorDemo.main(IteratorDemo.java:12)

```

## 3.5 并发修改异常

> 禁止：在迭代器遍历集合的过程中，不能使用集合的方法，改变集合的长度！！

```java
    /**
     * 并发修改异常
     * 迭代器遍历的过程中
     * 判断集合中是否存在 java 这个元素
     * 如果有，向集合中添加元素 JDK
     */
    public static void testIteratorConcurrent(){
        Collection<String> col = new ArrayList<>();
        //col对象，调用方法add()向集合中存储元素
        col.add("hello");
        col.add("java");
        col.add("hadoop");
        col.add("Linux");
        col.add("zookeeper");
        Iterator<String> it = col.iterator();
        while (it.hasNext()){
            String str = it.next();
            //判断集合中是否存在 java 这个元素
            if ("java".equals(str)) {
                //向集合中添加元素
                col.add("JDK");
            }
            System.out.println(str);
        }
    }

```

```java
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at com.atguigu.iterator.IteratorDemo.testIteratorConcurrent(IteratorDemo.java:30)
	at com.atguigu.iterator.IteratorDemo.main(IteratorDemo.java:12)

```

## 3.6 迭代器原理

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/3.png)

- ArrayList为例
  - 接口Iterator
    - 接口方法 hasNext()
    - 接口方法 next()

```java
public class ArrayList {
    //类中定义方法
    //返回迭代器接口 Iterator 的实现类
    public Iterator iterator(){
        //return 接口实现类的对象
        return new Itr();
    }
    class Itr impelements Iteraotr(){
        //重写方法 next() ,hasNext()
    }
}

```

> 所有的集合迭代器，实现类都是内部类原理

# 第四章 List接口

## 4.1 List接口（非常重要）

`java.util.List`接口：继承父接口Collection

- List接口的特点：
  - 有序的集合 （有序：集合容器，元素存储和取出的顺序一致）
    - 不是排序，不是123 abc顺序
  - 具有索引的集合，可以是索引访问集合中的元素
    - 0开始最小索引，集合的最大索引  集合长度.size() - 1
  - 可以存储重复元素的集合
- 总结：List接口的特点  **有序，索引，重复**

> 该接口的所有实现类，都具备上述的三个特点

## 4.2 List接口的方法

List接口继承Collection接口，学习成本很低，Collection接口的方法已经学习过了，学习List接口自己的方法，List接口的方法都是带有索引的方法。

- add(int index,存储元素) 元素存储在指定的索引上

```java
/**
     * add(int index,存储元素) 元素存储在指定的索引上
     * List接口实现类ArrayList建立对象
     * List接口，继承Collection接口
     */
    public static void testListAdd(){
        //创建集合容器对象，存储字符串
        List<String> list = new ArrayList<>();
        //集合对象list方法add存储元素
        list.add("java");
        list.add("C++");
        list.add("Linux");
        list.add("hadoop");
        //输出集合
        System.out.println("list = " + list);
        //向集合的指定索引上，添加元素
        //原有元素，向后瞬移
        list.add(2,"MySQL");
        //输出集合
        System.out.println("list = " + list);
    }

```

- 返回元素 get(int index) 取出指定索引上的元素

```java
/**
     * 返回元素 get(int index) 取出指定索引上的元素
     */
public static void testListGet(){
    //创建集合容器对象，存储字符串
    List<String> list = new ArrayList<>();
    //集合对象list方法add存储元素
    list.add("java");
    list.add("C++");
    list.add("Linux");
    list.add("hadoop");
    //集合list对象方法get(int 索引) 取出元素
    //String str = list.get(1);
    //System.out.println("str = " + str);
    for (int x = 0 ; x < list.size() ; x++){
        String str = list.get(x);
        System.out.println("str = " + str);
    }
}

```

> List集合所有实现类，都可以使用 size() + get() 遍历，针对于有索引的集合可以

-  set(int index, 元素) 修改指定索引上的元素，返回被修改之前的元素

```java
/**
     *  set(int index, 元素) 修改指定索引上的元素，返回被修改之前的元素
     */
public static void testListSet(){
    //创建集合容器对象，存储字符串
    List<String> list = new ArrayList<>();
    //集合对象list方法add存储元素
    list.add("java");
    list.add("C++");
    list.add("Linux");
    list.add("hadoop");
    System.out.println("list = " + list);
    //集合对象list的方法set，修改集合中的元素
    //获取修改之前的元素
    String str = list.set(2,"LINUX");
    System.out.println("str = " + str);
    System.out.println("list = " + list);
}

```

- remove(int index)移除指定索引上的元素，返回被移除之前的元素

```java
    /**
     * remove(int index)移除指定索引上的元素，返回被移除之前的元素
     */
    public static void testListRemove(){
        //创建集合容器对象，存储字符串
        List<String> list = new ArrayList<>();
        //集合对象list方法add存储元素
        list.add("java");
        list.add("C++");
        list.add("Linux");
        list.add("hadoop");
        System.out.println("list = " + list);
        //集合list对象的方法 remove移除0索引上的元素
        String str = list.remove(0);
        System.out.println("str = " + str);
        System.out.println("list = " + list);
    }

```

- 关于remove方法疑问：

```java
boolean b = col.remove("java");
String str = list.remove(0);
remove方法：传递的是基本类型int，按照索引删除
remove方法：传递的是引用类型，按照元素是否存在删除

```

```java
    /**
     * Collection接口方法： remove(元素)
     * List接口方法：remove(int 索引)
     * ArrayList 实现接口List
     * List接口，继承Collection
     */
    public static void testListRemove2(){
        List<Integer> list = new ArrayList<>();
        list.add(1);//add (new Integer(1))  自动装箱
        list.add(2);
        list.add(3);
        list.add(4);
        System.out.println("删除前：list = " + list);
        //按照索引删除，remove(int 类型参数)
        list.remove(1);
        //按照对象删除，元素是否存在参数
        list.remove(new Integer(1));
        System.out.println("删除后：list = " + list);
    }

```

## 4.3 数据结构

数据结构：数据在内存中的存储方式，就是数据结构

1：数组

数组是最常见数据结构，具有索引，可以通过索引快速访问到指定的元素，数组的查找性能还是很快的

数组的定长的，容量不够，创建新的数组，原有数组中的元素复制到新数组去。堆内存，创建数组，复制元素比较消耗内存，数组的结构，增删速度比较慢。

2：链表

链表是最常见的数据结构：内存中采用地址的记录方式存储的

链表增删非常快，不会破坏原有的数据结构

链表查询慢，获取元素，从链表的开头一个个向后找

链表中的对象，记录下一个元素的内存地址，同时记录上一个元素内存地址，称为双向链表

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/4.png)



3：队列

队列是内存中容器，元素先进去的先出来

4：堆栈

堆栈是元素先进去的后出来

## 4.4 ArrayList集合特点 （非常重要）

- ArrayList类实现接口List
  - ArrayList类底层实现是数组
  - 可以进行复制的方式，扩容
  - 查询速度快，增删速度慢
  - 不同步的（又称为线程不安全的）运行速度快

```java
    public static void main(String[] args) {
        //创建集合对象，接口对实现类多态
        List<Student> list = new ArrayList<>();
        list.add(new Student("张三",20));
        list.add(new Student("李四",22));
        list.add(new Student("王五",24));
        //迭代器遍历
        Iterator<Student> it = list.iterator();
        while (it.hasNext()){
            //取出集合中的元素
            Student stu = it.next();
            System.out.println("stu = " + stu);
        }
    }

```

## 4.5 ArrayList源码 提高 

```java
//变量 DEFAULT_CAPACITY 集合ArrayList里面数组，初始化容量
private static final int DEFAULT_CAPACITY = 10;
//Object类型的对象数组，数组的名字 elementData  存储元素的数组
transient Object[] elementData;
//size表示集合长度，不是数组长度
//size是个计数器：数组存一个元素 size ++
private int size;

//MAX_ARRAY_SIZE变量，ArrayList里面数组，最大值，可以存储多少个元素
//Integer.MAX_VALUE 是int类型的最大值 2147483647 - 8
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

//int 数组新容量 = 数组老容量 + 老容量 >> 1
//ArrayList里面数组，扩容数据，新容量 = 老容量 + 老容量的一半，1.5倍
int newCapacity = oldCapacity + (oldCapacity >> 1);
int newCapacity = 10 + 10 / 2

```

## 4.6 LinkedList 集合特点（非常重要）

- LinkedList类，实现List接口
  - LinkedList集合内部是双向链表
  - 查询速度慢，增删速度快
  - 不同步的（又称为线程不安全的）运行速度快
  - LinkedList是无边界集合，没有大小限制，内存以后多大，就能存储多大

> LinkedList类和ArrayList类，存储和迭代器的写法完全一样！！

## 4.7 LinkedList集合特有方法

LinkedList类自己的特有方法：不能使用多态性，特有方法专门用于操作集合的开头和结尾元素的

- addFirst(元素) 向集合开头添加元素
- addLast(元素) 向集合末尾添加元素

```java
    /**
     * - addFirst(元素) 向集合开头添加元素
     * - addLast(元素) 向集合末尾添加元素
     */
    public static void testAdd(){
        LinkedList<String> link = new LinkedList<>();
        link.add("a");
        link.add("b");
        link.add("c");
        System.out.println("link = " + link);
        //集合的开头添加元素
        link.addFirst("d");
        //集合的结尾添加  link.add("c"); 本身就是结尾添加
        link.addLast("e");
        System.out.println("link = " + link);
    }

```

- getFirst() 获取集合开头元素
- getLast() 获取集合末尾元素

```java
    /**
     * - getFirst() 获取集合开头元素
     * - getLast() 获取集合末尾元素
     */
    public static void testGet(){
        LinkedList<String> link = new LinkedList<>();
        link.add("a");
        link.add("b");
        link.add("c");
        link.add("d");
        //获取集合开头元素
        String first = link.getFirst();
        //获取集合结尾元素
        String last = link.getLast();
        System.out.println("first = " + first);
        System.out.println("last = " + last);
        System.out.println("link = " + link);
    }

```

- removeFirst() 移除并返回集合开头元素
- removeLast() 移除并返回集合末尾元素

```java
    /**
     * - removeFirst() 移除并返回集合开头元素
     * - removeLast() 移除并返回集合末尾元素
     */
    public static void testRemove(){
        LinkedList<String> link = new LinkedList<>();
        link.add("a");
        link.add("b");
        link.add("c");
        link.add("d");
        //移除开头元素
        String first = link.removeFirst();
        //移除结尾元素
        String last = link.removeLast();
        System.out.println("first = " + first);
        System.out.println("last = " + last);
        System.out.println("link = " + link);
    }

```

# 第五章 集合模拟斗地主

1. 准备牌

   扑克牌有54张，52个点数牌 + 2个王牌组成，装在牌盒里面

   54个字符串代替牌，字符串存集合中（牌盒）

   真的要定义54个字符串吗？NO

   52张扑克牌：4个花色+13个点数组成，定值

   嵌套循环，内循环和外循环，总的循环次数=内*外

   外循环做花色，内循环做点数

2. 洗牌

   就是将牌的顺序打乱：集合poker中，存储了54个字符串，集合中的54个字符串元素随机排列

   每次运行，顺序都是不同的。`java.util.Collections`类：静态方法 shuffle(集合),将对集合中的所有元素进行随机排列。

3. 发牌

   有三个玩家，依次发牌，每个玩家手里17张，底牌3张

   每个玩家手里是17个字符串，存储集合，需要三个玩家集合和一个底牌集合

   发牌：本质就是从存储54个集合（牌盒中）取出一个字符串，存储在玩家集合中

4. 看牌

   遍历4个集合

```java
/**
 * 集合模拟斗地主案例
 */
public class Poker {
    public static void main(String[] args) {
        //集合作为牌盒容器，存储54个字符串
        List<String> poker = new ArrayList<>();
        //数组，存储花色 4个
        String[] colors = {"♠","♥","♣","♦"};
        //数组，存储点数 13个
        String[] numbers = {"A","2","3","4","5","6","7","8","9","10","J","Q","K",};
        //嵌套循环，遍历数组，外循环花色，内循环点数
        for (String color : colors){
            for (String number : numbers){
                //花色和点数拼接,52个字符串，存储集合
                //System.out.println(color+number);
                poker.add(color+number);
            }
        }
        //向集合中，存储王牌
        poker.add("大王");
        poker.add("小王");
        //System.out.println("poker = " + poker);
        //Collections类的静态方法：shuffle(poker) 随机排列
        Collections.shuffle(poker);
        //System.out.println("poker = " + poker);
        //创建3个玩家集合，1个底牌集合
        List<String> player1 = new ArrayList<>();
        List<String> player2 = new ArrayList<>();
        List<String> player3 = new ArrayList<>();
        List<String> diPai = new ArrayList<>();
        //遍历牌盒集合poker，集合的索引 % 3 ,实现依次发牌
        for (int x = 0 ; x < poker.size() ; x++){
            //先取出前三张为底牌
            if (x < 3){
                diPai.add( poker.get(x) );
            }

            //判断索引%3==0,发给玩家1
            else if ( x % 3 == 0 ){
                //牌盒集合中，取出字符串，存储玩家集合
                player1.add(poker.get(x));
            }
            //判断索引%3==1 发给玩家2
            else if ( x % 3 == 1){
                player2.add(poker.get(x));
            }
            //判断索引%3==2，发给玩家3
            else{
                player3.add(poker.get(x));
            }
        }
        //调用看牌方法，传递4个集合
        lookPoker(player1);
        lookPoker(player2);
        lookPoker(player3);
        lookPoker(diPai);
    }
    /**
     * 定义方法实现看牌
     * 传递4个集合，遍历
     * 调用4次
     */
    public static void lookPoker(List<String> list){
        for(String str : list){
            System.out.print(str+" ");
        }
        System.out.println();
    }
}

```

