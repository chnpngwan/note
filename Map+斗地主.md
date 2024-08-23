##### 课堂记录

- 集合回顾
- map中使用=作为分隔符
- 【增强for循环使用`map.keyset + map.entrySet`】
- 【`LinkedHashMap, LinkedHashSet`】
- 二分查找（有序的数组）

集合框架

- 今日内容
  - Map集合特性
  - Map接口方法
  - Map集合遍历
  - `HashMap`集合特性
  - `TreeMap`集合特性
  - `Hashtable`集合特性
  - Properties类

# 第一章 Map集合

## 1.1 Map集合介绍

Map：映射键值对。集合家族中有2个顶级的接口，**一个是Collection，一个是Map。两个接口之间没有任何关系**

Map系列的集合，每次存储两个对象，一个对象称为键（Key），另一个对象称为值（Value）

> Map系列的集合中，不能包含重复的键，每个键只能对应一个值

Map<K,V> K是用于键的数据类型，V是用于值的数据类型

## 1.2 Map接口方法（非常重要）

Map接口，作为键值对集合的顶级接口，该接口的方法，是所有的子接口和实现类都必须具备的！

- put(键,值) 向集合中存储键值对

```java
    /**
     * put(键,值) 向集合中存储键值对
     * put方法有返回值，返回值的类型是V，值的数据类型
     * 返回值在一般的情况下均返回 null 值
     * 当向集合中存储了重复键的时候，不在返回null
     * 会替换原有的值，put将返回被替换之前的值
     */
    public static void testMapPut(){
        //创建Map集合，多态方式创建
        //字符串为键，整数为值
        Map<String,Integer> map = new HashMap<>();
        //map集合对象的方法put，存储键值对
        map.put("java",10000);
        map.put("c++",20000);
        map.put("hello",30000);
        Integer value = map.put("java",15000);
        System.out.println("value = " + value);
        System.out.println("map = " + map);
    }
```

- V get(K) 传递键，获取该键对应的值

```java
    /**
     * V get(K) 传递键，获取该键对应的值
     * 集合中没有这个键，返回null值
     */
    public static void testMapGet(){
        //创建Map集合，整数作为键，字符串为值
        Map<Integer,String> map = new HashMap<>();
        //集合对象方法put存储键值对
        map.put(1,"hello");
        map.put(3,"java");
        map.put(2,"world");
        System.out.println("map = " + map);
        //map集合对象的方法 get，传递键，获取值
        String value = map.get(3);
        System.out.println("value = " + value);
    }
```

- boolean containsKey(K) 判断集合中是否包含此键，如果包含返回true
- boolean containsValue(V)判断集合中是否包含此值，如果包含返回true

```java
 /**
     * - boolean containsKey(K) 判断集合中是否包含此键，如果包含返回true
     * - boolean containsValue(V)判断集合中是否包含此值，如果包含返回true
     */
    public static void testMapContains(){
        //创建Map集合，整数作为键，字符串为值
        Map<Integer,String> map = new HashMap<>();
        //集合对象方法put存储键值对
        map.put(1,"hello");
        map.put(3,"java");
        map.put(2,"world");
        //判断集合中个，是否有3这个键
        boolean key = map.containsKey(3);
        //判断集合中，是否有 world这个值
        boolean value = map.containsValue("world");
        System.out.println("key = " + key);
        System.out.println("value = " + value);
    }
```

- Collection<V> values() 将Map集合中的所有的值，存储在Collection集合中

```java
/**
     * Collection<V> values() 将Map集合中的所有的值，存储在Collection集合中
     */
    public static void testMapValues(){
        //创建Map集合，整数作为键，字符串为值
        Map<Integer,String> map = new HashMap<>();
        //集合对象方法put存储键值对
        map.put(1,"hello");
        map.put(3,"java");
        map.put(2,"world");
        //map集合对象的方法values() 将值拿出来，存储在Collection集合中
        Collection<String> col = map.values();
        for(String s : col){
            System.out.println("s = " + s);
        }
    }
```

## 1.3 Map集合遍历 （非常重要）

### 1.3.1 Map集合的遍历方式一：键找值

- Map接口中的方法：Set<K> keySet()  Map中的所有的键，存储在Set集合中

```java
  /**
     * Map接口中的方法：Set<K> keySet()  Map中的所有的键，存储在Set集合中
     * 遍历步骤：
     *   1：调用Map集合方法 keySet() ，拿到Set集合
     *   2：迭代器迭代Set集合
     *   3：取出Set集合中的元素：元素是Map集合中的键
     *   4：Map中的键，找Map中的值，Map接口方法 get
     */
    public static void testMapKeySet(){
        //创建Map集合，键是字符串，值是整数
        Map<String,Integer> map = new HashMap<>();
        map.put("java",123);
        map.put("hadoop",124);
        map.put("Linux",125);
        map.put("hive",126);
        map.put("zookeeper",127);
        //1：调用Map集合方法 keySet() ，拿到Set集合
        Set<String> set = map.keySet();
        //2：迭代器迭代Set集合
        Iterator<String> it = set.iterator();
        //3：取出Set集合中的元素：元素是Map集合中的键
        while (it.hasNext()){
            //4：Map中的键，找Map中的值，Map接口方法 get
            String key = it.next();
            Integer value = map.get(key);
            System.out.println(key+"="+value);
        }
    }
```

### 1.3.2 Map集合的遍历方式二：键值对关系遍历

Map集合中存储了多少个键值对，就会出现多少个键值对的对应关系

对应关系也是对象，Entry接口实现类的对象，表示键值对的对应关系

将多个对应关系，存储在Set集合中。

![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05111.png)

Entry接口方法：getKey() 获取键，getValue()获取值

- Map接口方法：  entrySet() 集合中的键值对的对应关系，存储在Set集合中，对应关系就是Entry接口实现类的对象

```java
    /**
     * Map接口方法：  entrySet() 集合中的键值对的对应关系，存储在Set集合中，
     * 对应关系就是Entry接口实现类的对象
     * 遍历步骤：
     *   1: 调用Map集合的方法：Set<Entry> entrySet() Map中的所有键值对的对应关系 Entry接口对象，存储Set集合
     *   2: 迭代器遍历Set集合
     *   3：取出Set集合元素
     *         Set集合中，存储的是Entry接口实现类的对象 （结婚证，键值对的对应关系）
     *   4：Entry接口实现类的对象，调用方法getKey,getValue
     */
public static void testMapEntry(){
    //创建Map集合，键是字符串，值是整数
    Map<String,Integer> map = new HashMap<>();
    map.put("java",123);
    map.put("hadoop",124);
    map.put("Linux",125);
    map.put("hive",126);
    map.put("zookeeper",127);
    //1:调用Map集合的方法：Set<Entry> entrySet() Map中的所有键值对的对应关系 Entry接口对象，存储Set集合
    //Entry接口实现内部接口，通过外部接口名.内部接口名
    Set< Map.Entry<String,Integer> > set = map.entrySet();
    //2: 迭代器遍历Set集合
    Iterator<Map.Entry<String,Integer>> it = set.iterator();
    while (it.hasNext()){
        //3：取出Set集合元素,就是Entry接口实现类的对象
        Map.Entry<String,Integer> entry = it.next();
        //4：Entry接口实现类的对象，调用方法getKey,getValue
        String key = entry.getKey();
        Integer value = entry.getValue();
        System.out.println(key+"="+value);
    }
}
```

### 1.3.3 增强for能不能遍历Map集合

回答：for循环不能直接遍历Map集合！但是可以间接遍历Map集合！

```java
   /**
     * for循环间接遍历Map集合：entrySet方式
     */
    public static void testMapForEntrySet(){
        //创建Map集合，键是字符串，值是整数
        Map<String,Integer> map = new HashMap<>();
        map.put("java",123);
        map.put("hadoop",124);
        map.put("Linux",125);
        map.put("hive",126);
        map.put("zookeeper",127);

        for ( Map.Entry<String,Integer> entry :map.entrySet() ){
            System.out.println(entry.getKey()+"::"+entry.getValue());
        }
    }

    /**
     * for循环间接遍历Map集合：keySet方式
     */
    public static void testMapForKeySet(){
        //创建Map集合，键是字符串，值是整数
        Map<String,Integer> map = new HashMap<>();
        map.put("java",123);
        map.put("hadoop",124);
        map.put("Linux",125);
        map.put("hive",126);
        map.put("zookeeper",127);

        //Set<String> set = map.keySet();
        for (String key : map.keySet()){
            //Integer value = map.get(key);
            System.out.println(key+"::"+map.get(key));
        }
    }
```

# 第二章 HashMap集合

## 2.1 HashMap集合自身特点 （非常重要）

- HashMap类，实现Map接口
  - 底层是哈希表结构：数组 + 单向链表组合
  - **默认长度16，增长翻一倍，加载因子0.75F**
  - 线程不安全的集合，运行速度快
  - 允许存储null值，null键
  - 必须保证键的唯一性，用于键的对象必须实现方法 hashCode()，equals()
- HashMap类在JDK1.7和1.8是不同的
  - JDK1.8进行优化：转红黑树动作
    - 当数组的一个索引上，链表的中元素的数量，达到8个，转换为红黑树结构 
    - 数组中长度达到64，才会转树
  - JDK1.8进行优化尾插法
    - JDK1.7 没转树功能
    - JDK1.7 头插法



![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05115.png)

## 2.2 HashMap练习

- HashMap存储键值对：键字符串，值是自定义对象

```java
    /**
     * HashMap存储键值对，键是字符串，值是Person类型
     * 进行遍历
     */
    public static void testStringPerson(){
        Map<String, Person> map = new HashMap<>();
        map.put("北京市",new Person("杰克",20));
        map.put("上海市",new Person("露丝",20));
        map.put("广州市",new Person("麦克",20));
        map.put("深圳市",new Person("李磊",20));
        map.put("重庆市",new Person("韩梅梅",20));
        //entrySet方式遍历
        Set< Map.Entry<String,Person> > set = map.entrySet();
        Iterator< Map.Entry<String,Person> > it = set.iterator();
        while (it.hasNext()){
            Map.Entry<String,Person> entry =it.next();
            //取出键值对
            String key = entry.getKey();
            Person value = entry.getValue();
            System.out.println(key+"::"+value);
        }
    }
```

- HashMap存储键值对：键是自定义对象，值是字符串

```java
  /**
     * HashMap存储键值对：键是自定义对象，值是字符串
     * map.put(new Person("露丝",20),"上海市");
     * map.put(new Person("露丝",20),"上海市");
     * 在哈希表中，认为一个Person对象，姓名和年龄一样，就是一个对象
     * 但是内存中（两个地方）
     * 用于键的对象，必须重写hashCode,equals
     */
    public static void testPersonString(){
        Map<Person,String> map = new HashMap<>();
        map.put(new Person("杰克",20),"北京市");
        map.put(new Person("露丝",20),"上海市");
        map.put(new Person("露丝",20),"上海市");
        map.put(new Person("麦克",20),"广州市");
        map.put(new Person("李雷",20),"深圳市");
        map.put(new Person("韩梅梅",20),"温州市");
        Set<Map.Entry<Person,String>> set = map.entrySet();
        Iterator<Map.Entry<Person,String>> it = set.iterator();
        while (it.hasNext()){
           Map.Entry<Person,String> entry = it.next();
           Person key = entry.getKey();
           String value = entry.getValue();
            System.out.println(key+"::"+value);
        }
    }
```

> 8个基本数据类型的包装类以及String类，都继承Object，重写方法hashCode(),equals()
>
> Map集合，存储的是8个基本类型包装类和String，都会自动做到键唯一性
>
> Map<String,Object> map = new HashMap<>(); 以后的课程中，项目中，最长见

## 2.3 LinkedHashMap

- `java.util.LinkedHashMap`类：继承HashMap，实现Map接口
  - 底层数据结构是哈希表：数组 + 双向链表组合
  - 其他的特点都和父类是一样的
  - 有序的Map集合，保证存储和取出的顺序是一致的

- `java.util.LinkedHashSet`类：继承HashSet，实现Set接口
  - 本身没有功能，底层调用LinkedHashMap实现类
  - 有序的Set集合，保证存储和取出的顺序是一致的

```java
    public static void main(String[] args) {
        //有序的Map集合
        Map<String,Integer> map = new LinkedHashMap<>();
        map.put("b",2);
        map.put("d",4);
        map.put("a",1);
        map.put("c",3);
        System.out.println("map = " + map);

        //有序的Set集合
        Set<String> set = new LinkedHashSet<>();
        set.add("are");
        set.add("how");
        set.add("hello");
        set.add("you");
        System.out.println("set = " + set);
    }
```

# 第三章 TreeMap集合

## 3.1 TreeMap集合自身特点（非常重要）

- `java.util.TreeMap`类：实现Map接口
  - 底层是红黑树结构
  - 查询速度快，但是增删慢
  - 线程不安全集合，运行速度快
  - 红黑树结构，对存储的元素排序
    - 1：对象的自然顺序，对象必须实现接口Comparable
    - 2：自定义的比较器，必须实现接口Comparator

## 3.2 TreeMap练习

- TreeMap存储键值对：键字符串，值是自定义对象

```java
   /**
     *  TreeMap存储键值对：键字符串，值是自定义对象
     */
    public static void testStringPerson(){
        Map<String, Person> map = new TreeMap<>();
        map.put("beijing",new Person("杰克",20));
        map.put("shanghai",new Person("露丝",20));
        map.put("guangzhou",new Person("麦克",20));
        map.put("shenzhen",new Person("李磊",20));
        map.put("chongqing",new Person("韩梅梅",20));
        Set<Map.Entry<String,Person>> set =map.entrySet();
        Iterator<Map.Entry<String,Person>> it = set.iterator();
        while (it.hasNext()){
            Map.Entry<String,Person> entry =it.next();
            String key = entry.getKey();
            Person value = entry.getValue();
            System.out.println(key+"::"+value);
        }
    }
```

> 8个基本数据类型包装类和String类，都实现了接口Comparable，重写方法compareTo
>
> 8个基本数据类型和String对象，存储到红黑树中，作为键，直接就排序了！

- TreeMap存储键值对：键是自定义对象 ，值是字符串


```java
    /**
     * TreeMap存储键值对：键是自定义对象 ，值是字符串
     */
    public static void testPersonString(){
        Map<Person,String> map = new TreeMap<>( new MyComparator() );
        map.put(new Person("杰克",20),"北京市");
        map.put(new Person("露丝",18),"上海市");
        map.put(new Person("麦克",22),"广州市");
        map.put(new Person("李雷",30),"深圳市");
        map.put(new Person("韩梅梅",25),"温州市");
        Set<Map.Entry<Person,String>> set = map.entrySet();
        Iterator<Map.Entry<Person,String>> it = set.iterator();
        while (it.hasNext()){
            Map.Entry<Person,String> entry = it.next();
            System.out.println(entry.getKey()+"::"+entry.getValue());
        }
    }
```

```java
/**
 * 自定义的比较器：实现接口Comparator<T>
 *     重写方法 compare
 */
public class MyComparator implements Comparator<Person> {
    /**
     *  比较器中的比较方法
     *  o1是后来的对象，o2是先来的对象
     */
    public int compare(Person o1, Person o2) {
        return o1.getAge() - o2.getAge();
    }
}
```

# 第三章 Hashtable集合

## 3.1 Hashtable 集合自身特性（非常重要）

- `java.util.Hashtable`类，实现Map接口
  - 底层数据结构是哈希表结构
  - 这个集合**不能存储null值，null键**
  - 数组初始容量**11个，加载因子0.75F**
  - 这个类是线程安全的集合，运行速度慢
  - Hashtable集合被更加先进的HashMap取代 （郁郁而终）
  - Vector集合，被更加先进的ArrayList取代 （郁郁而终）

## 3.2 Properties类 （非常重要）

- `java.util.Propeties`类，继承Hashtable，实现Map接口
  - 集合自身的特性和父类Hashtable一样的，不在重复
  - Properties集合类可以和IO流对象，结合使用，实现数据的持久化！！

## 3.3 Properties类的特有方法 （非常重要）

Properties不能写多态：这个集合没有泛型，该集合的存储元素的数据类型固定String类型

- 存储键值对：setProperty(String key,String value)

```java
    /**
     * 存储键值对：setProperty(String key,String value)
     */
    public static void set(){
        Properties prop = new Properties();
        //存储键值对
        prop.setProperty("a","123");
        prop.setProperty("b","223");
        prop.setProperty("c","323");
        System.out.println("prop = " + prop);
    }
```

- 取出键值对：String getProperty(String key)

```java
    /**
     * 取出键值对：String getProperty(String key)
     */
    public static void get(){
        Properties prop = new Properties();
        //存储键值对
        prop.setProperty("a","123");
        prop.setProperty("b","223");
        prop.setProperty("c","323");
        System.out.println("prop = " + prop);
        String value = prop.getProperty("a");
        System.out.println("value = " + value);
    }
```

- Set<String>stringPropertyNames() 将集合中的键，存储到Set集合，相当于Map中方法keySet()

```java
    /**
     * Set<String>stringPropertyNames() 将集合中的键，存储到Set集合，
     * 相当于Map中方法keySet()
     */
    public static void propertyNames(){
        Properties prop = new Properties();
        //存储键值对
        prop.setProperty("a","123");
        prop.setProperty("b","223");
        prop.setProperty("c","323");
        Set<String> set = prop.stringPropertyNames();
        for (String key : set){
            System.out.println(key+"::"+prop.getProperty(key));
        }
    }
```

- Properties类中关键的方法： load (IO中的输入流)

# 第四章 数组集合工具类

## 4.1 数组工具类

- `java.util.Arrays`类：数组操作的工具类，全部是静态方法，数组的常见的操作
  - static void sort(数组) 对数组的升序排列，快速排序法
  - static String toString(数组) 将数组的元素，拼接为字符串，返回String类型
  - static int binarySearch(数组，要查找的元素) 二分查找，返回元素出现的索引
    - 查找有共性：查找的结果是引用类型，找不到返回null
    - 查找有共性：查找的结果是基本类型，找不到返回负数
  - asList(T...t) 数组转成集合

```java
 /**
     * List<T>  asList(T...t) 数组转成集合
     * 转成集合，集合长度不能改
     */
    public static void testArraysAsList(){
        List<String> list = Arrays.asList("abc","123","QQ");
        list.set(1,"hello");
        System.out.println("list = " + list);
    }

    /**
     * static int binarySearch(数组，要查找的元素) 二分查找，返回元素出现的索引
     * 找不到元素，返回负数，
     * 计算公式： 返回 (-插入点)-1
     */
    public static void testArraysBinarySearch(){
        int[] arr = {1,3,6,8,12,16,19,22,8,26};
        int index = Arrays.binarySearch(arr, 9);
        System.out.println("index = " + index);
    }

    /**
     * static String toString(数组) 将数组的元素，拼接为字符串，返回String类型
     */
    public static void testArraysToString(){
        int[] arr = {2,1,5,6,9,0,3};
        //String str = Arrays.toString(arr);
        //System.out.println("str = " + str);
        System.out.println(Arrays.toString(arr));
    }

    /**
     * static void sort(数组) 对数组的升序排列，快速排序法
     */
    public static void testArraysSort(){
        int[] arr = {2,1,5,6,9,0,3};
        Arrays.sort(arr);
        for (int i : arr){
            System.out.println("i = " + i);
        }
    }
```

## 4.2 集合工具类

- `java.util.Collections`类：集合操作的工具，全部静态方法
  - sort(List list) 对List集合中的元素，升序排列
  - Comparator reverseOrder(传递比较器)，返回比较器，逆转原有比较器的顺序
  - synchronized    ['sɪŋkrənaɪzd]  开头的方法，可以将线程不安全的集合， 转成线程安全的集合

```java
 /**
     * synchronized开头的方法，可以将线程不安全的集合， 转成线程安全的集合
     */
    public static void testCollectionsSynchronized(){
        List<String> list = new ArrayList<>();
        list.add("java");
        List<String> newList = Collections.synchronizedList(list);
        //newList集合，变成线程安全的集合
        System.out.println("newList = " + newList);
    }

    /**
     *  Comparator reverseOrder(传递比较器)，返回比较器，逆转原有比较器的顺序
     */
    public static void testCollectionsReverseOrder(){

        Comparator com =  Collections.reverseOrder( new MyComparator());

        Map<Person,String> map = new TreeMap<>( com );
        map.put(new Person("杰克",20),"北京市");
        map.put(new Person("露丝",18),"上海市");
        map.put(new Person("麦克",22),"广州市");
        map.put(new Person("李雷",30),"深圳市");
        map.put(new Person("韩梅梅",25),"温州市");
        Set<Map.Entry<Person,String>> set = map.entrySet();
        Iterator<Map.Entry<Person,String>> it = set.iterator();
        while (it.hasNext()){
            Map.Entry<Person,String> entry = it.next();
            System.out.println(entry.getKey()+"::"+entry.getValue());
        }
    }

    /**
     * sort(List list) 对List集合中的元素，升序排列
     */
    public static void testCollectionsSort(){
        List<String> list = new ArrayList<>();
        list.add("hello");
        list.add("world");
        list.add("java");
        list.add("how");
        System.out.println("list = " + list);
        Collections.sort(list);
        System.out.println("list = " + list);
    }
```

# 第五章 斗地主案例排序

斗地主游戏排序，不是自然顺序， 从小到大：3333 4444 5555 ... ... AAAA 2222 小王 大王

玩家手上： 3 3 4 4 5 6 77 8 9 A A 2 2 王  顺序是开发人员自定义的顺序。

使用编号思想：对54张牌（字符串）进行编号的定义，可以是0-53，让每一个排都对应的自己的编号

3333 对应编号 0123 ， 4444 对应的编号 4567 , 5555 对应的编号 891011 ......

大王对应53，小王对应52,  2222对应51 50 49 48

 ![](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/05112.png)

```java
public static void main(String[] args) {
        //Map集合，存储牌编号，和牌的字符串
        Map<Integer,String> pokerMap = new HashMap<>();
        //List集合，专门存储编号
        List<Integer> pokerList = new ArrayList<>();
        //数组，存储花色 4个
        String[] colors = {"♠","♥","♣","♦"};
        //数组，存储点数 13个，按照牌的顺序来写
        String[] numbers = {"3","4","5","6","7","8","9","10","J","Q","K","A","2"};
        //定义变量，用户记录牌的编号，0-53
        int index = 0;
        //遍历数组，花色和点数的拼接
        //点数是外循环，花色是内循环
        for (String number :numbers){
            for (String color : colors){
                //花色和点数拼接字符串，存储Map集合,键是编号
                pokerMap.put(index,color+number);
                //单独存储编号
                pokerList.add(index);
                index++;
            }
        }
        //存储王牌
        pokerMap.put(52,"小王");
        pokerMap.put(53,"大王");
        pokerList.add(52);
        pokerList.add(53);
        /*System.out.println("pokerMap = " + pokerMap);
        System.out.println("pokerList = " + pokerList);*/
        //洗牌，打乱pokerList集合中元素顺序，洗的是编号
        Collections.shuffle(pokerList);
        //System.out.println("pokerList = " + pokerList);
        //定义4个集合，3个玩家，1个底牌  TreeSet存储 （自带排序）
        //如果使用ArrayList也行，Collections.sort() 对ArrayList排序
        Set<Integer> player1 = new TreeSet<>();
        Set<Integer> player2 = new TreeSet<>();
        Set<Integer> player3 = new TreeSet<>();
        Set<Integer> diPai = new TreeSet<>();
        //循环遍历编号集合 pokerList ，% 3的方式发牌
        //发到玩家手中的是 编号
        for (int x = 0 ; x < pokerList.size() ; x++){
            //留下3个底牌
            if ( x < 3){
                diPai.add( pokerList.get(x) );
            }
            else if ( x % 3 == 0){
                player1.add( pokerList.get(x) );
            }
            else if ( x % 3 == 1){
                player2.add( pokerList.get(x) );
            }
            else {
                player3.add( pokerList.get(x) );
            }
        }
        /*System.out.println("player1 = " + player1);
        System.out.println("player2 = " + player2);
        System.out.println("player3 = " + player3);*/
        //调用看牌方法，传递TreeSet集合，Map集合
        lookPoker( "刘德华",player1,pokerMap );
        lookPoker( "张学友",player2,pokerMap );
        lookPoker( "郭富城",player3,pokerMap );
        lookPoker( "底牌",diPai,pokerMap );
    }
    /**
     *  定义看牌的方法： TreeSet集合遍历，集合的元素作为键，到Map集合中找值
     */
    public static void lookPoker(String name,Set<Integer> set,Map<Integer,String> map){
        System.out.print(name+"::");
        for(Integer key : set){
            String value = map.get(key);
            System.out.print(value+" ");
        }
        System.out.println();
    }
```

##### 栈实现算数运算

![1652276166407](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1652276166407.png)

##### 栈括号匹配

![1652276208830](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1652276208830.png)

##### 浏览器前进后退

![1652276249235](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1652276249235.png)

