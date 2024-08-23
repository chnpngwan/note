##### 集合高阶函数01

```scala
package chapter07

object Scala_12_HIghtFunc1 {
  def main(args: Array[String]): Unit = {
    // TODO 高级函数的使用的前提， 每一个函数都会内置一个循环
    val list1 = List(1, 2, 3, 6, 7, 9, 8, 4, 0)

    // 类似于MR中的map阶段，
    //    1）过滤
    //    遍历一个集合并从中获取满足指定条件的元素组成一个新的集合。
    // 将集合中的偶数过滤出来
    val list11: List[Int] = list1.filter((p: Int) => {
      p % 2 == 0
    })
    val list12: List[Int] = list1.filter(_ % 2 == 0)


    println(list11)
    //    （2）转化/映射（map）
    //    将集合中的每一个元素映射到某一个函数。
    // TODO 转化映射（map）：流程： 将集合中的每一个元素作为参数 传递给一个函数， 最后按照映射的规则进行计算，
    //  将结果返回， map映射 一定是一一对应的关系, 映射之后可以改变数据类型
    // 将集合中的元素 *2
    val list2: List[String] = list1.map(i => "" + i * 2)
    println(list2)
     val list21: List[List[Int]] = list1.map(i => List(i, i))
    println(list21)

    //    （3）扁平化
    // 将数据格式化， 针对嵌套式集合
    // 将 原始集合
    val list3: List[Int] = list21.flatten
    println("lists===========>"+ list3)
//    lise21 ==  List(List(1, 1), List(2, 2), List(3, 3), List(6, 6), List(7, 7), List(9, 9), List(8, 8)
//    lists===========>List(1, 1, 2, 2, 3, 3, 6, 6, 7, 7, 9, 9, 8, 8, 4, 4, 0, 0)

    // TODO 扁平化特例，原因： String 底层就是有个char类型的数组
    val list31 = List("hello hai", "wang wu")
    val list32: List[Char] = list31.flatten
    println(list32)
    // List(h, e, l, l, o,  , h, a, i, w, a, n, g,  , w, u)

    // 希望的效果 List("hello",  "hai", "wang",  "wu")
    //（4）扁平化+映射 注：flatMap相当于先进行map操作，在进行flatten操作。
    // 首先： map
    val list4: List[List[String]] = list31.map(data => {
      val strings: Array[String] = data.split(" ")
      strings.toList
    })
    println(list4)
    // List(List(hello, hai), List(wang, wu))

    val list41: List[String] = list4.flatten
    println(list41)
    // List(hello, hai, wang, wu)

    // TODO 映射加扁平化 == > 简化方案
    val list42: List[String] = list31.flatMap(data => data.split(" "))
    val list43: List[String] = list31.flatMap(_.split(" "))
    println(list42)
    // List(hello, hai, wang, wu)


    // TODO Shuffle阶段，


    //    集合中的每个元素的子元素映射到某个函数并返回新集合。
    //    （5）分组（groupBy）
    //    按照指定的规则对集合的元素进行分组。
    val list5 = List("hello", "hadoop", "hive", "flume", "kafka", "scala", "spark", "hbase", "flink")
    // 按照相同的首字母分组
    val list51: Map[Char, List[String]] = list5.groupBy(data => data.charAt(0))
    println(list51)

    //    （6）简化（归约）
    //    （7）折叠
  }
}
```

##### 集合高阶函数02

```scala
package chapter07

object Scala_013_HightFunc2 {
  def main(args: Array[String]): Unit = {
    // TODO : 高级函数都自带循环， 将集合中的每个元素循环操作
    // TODO 类似于Reduce阶段

    val list1 = List(1, 2, 4, 6, 8, 3, 5, 4, 7)
    var res: Int = 0
    for (elem <- list1) {
      res += elem * 2
    }
    println(res)

    //    （6）简化（归约）
    // 参数：reduce(op: (A1, A1) => A1)
    // 1.  传入的第一个参数是每次运算的结果， 第二个元素是集合中的每一个元素，   返回结果值
    // 2. 第一个参数，  第二个参数， 返回结果值， 类型就是一样的
    val result: Int = list1.reduce((res, elem) => {
      res + elem * 2
    })
    println(result)

    val i1: Int = list1.reduceLeft((res, elem) => {
      res + elem * 2
    })
    println(i1)

    val i2: Int = list1.reduceRight((res, elem) => {
      res + elem * 2
    })
    println(i2)

    //    （7）折叠(其实也是做规约，相当于是 reduce的一种优化和扩展)fold
    // reduce默认会使用集合中的第一个值作为初始值， 在计算逻辑中会直接跳过第一个元素
    // 循环list1中的数据进行操作
    val i3: Int = list1.fold(0)((res, elem) => {res + elem * 2})
    println(i3)

    // reduce与fold 都不能改变返回值的类型
    // 基于上面的需求， 返回一个二元组
    val tup1: (String, Int) = list1.foldLeft(("元素 * 2 为： ", 0))((res, elem) => (res._1, res._2 + elem * 2))
    println(tup1)

    // 小结：
    // 1.如果不考虑结果初始化的值， 并且保证 传入的数据与返回值类型一致， reduce/reduceLeft
    // 2.如果考虑结果初始化的值， 并且保证 传入的数据与返回值类型一致， fold
  }
}
```

##### WordCount案例

```scala
package chapter07

object Scala_014_WordCount {
  def main(args: Array[String]): Unit = {
    // 单词计数：将集合中出现的相同的单词，进行计数，取计数排名前三的结果
    val stringList = List("Hello Scala Hbase kafka", "Hello Scala Hbase", "Hello Scala", "Hello")

    // 1. 将长字符串 映射+ 扁平化
    val list1: List[String] = stringList.flatMap(data => data.split(" "))
    println(list1)
    // List(Hello, Scala, Hbase, kafka, Hello, Scala, Hbase, Hello, Scala, Hello)

    // 2. 根据相同的单词， 进行分组操作 groupBy
    val stringToList: Map[String, List[String]] = list1.groupBy(word => word)
    println(stringToList)
    // Map(Hello -> List(Hello, Hello, Hello, Hello), Hbase -> List(Hbase, Hbase), kafka -> List(kafka), Scala -> List(Scala, Scala, Scala))

    // 3.将同一个组内的元素进行数量统计， 映射为一个二元组的结构
    val stringToInt: Map[String, Int] = stringToList.map(tuple => (tuple._1, tuple._2.size))
    println(stringToInt)
    // Map(Hello -> 4, Hbase -> 2, kafka -> 1, Scala -> 3)

    // 4. 根据单词的数量进行倒序排列
    val toList: List[(String, Int)] = stringToInt.toList
    println(toList)
    // List((Hello,4), (Hbase,2), (kafka,1), (Scala,3))

    val tuple1: List[(String, Int)] = toList.sortWith((x, y) => x._2>y._2)
    println(tuple1)
    // List((Hello,4), (Scala,3), (Hbase,2), (kafka,1))

    // 5. 获取前三个
    val tup1: List[(String, Int)] = tuple1.take(3)
    println(tup1)
    // List((Hello,4), (Scala,3), (Hbase,2))

    // 一步到位写法(更加体现迭代式开发)

    val tup4: List[(String, Int)] = stringList.flatMap(_.split(" "))
      .groupBy(s => s)
      .mapValues(_.size)
      .toList
      .sortWith((_._2 > _._2))
      .take(3)
    println(tup4)
  }
}
```

##### 队列

```scala
package chapter07

import scala.collection.immutable.Queue
import scala.collection.mutable

object Scala_015_Queue {
  def main(args: Array[String]): Unit = {
    // 不可变队列【了解】
    val que1 = Queue(1, 2, 3, 4, 5)

    // 添加内容
    val que11: Queue[Int] = que1.enqueue(9)
    println(que11)

    //  删除内容, 出队， 返回值是一个二元组， 第一个是出队列的元素， 第二个元素是出队列操作之后的队列
    val dequeue: (Int, Queue[Int]) = que11.dequeue
    println(dequeue)

    // TODO 可变队列
    val que2: mutable.Queue[Int] = mutable.Queue(1, 2, 3, 4, 5)
    que2.enqueue(12)
    println(que2)

    val i1: Int = que2.dequeue()
    println(i1)
    println(que2)

  }
}
```

##### 并行编程

```scala
package chapter07

object Scala_016_Par {
  def main(args: Array[String]): Unit = {
    // 并行集合；状态模拟
    val inclusive1: Range.Inclusive = 1 to 100

    // 单线程处理
    //    for (elem <- inclusive1) {
    //      println(elem + "---------->" + Thread.currentThread().getName)
    //    }

    // 体现并行线程

    inclusive1.par.foreach(i => println(i + "----->" + Thread.currentThread().getName)
    )

  }
}
```

##### Mach语法

- 匹配标识符

```scala
package chapter08

object Scala01_Match {
  def main(args: Array[String]): Unit = {
    // 基本语法
    // 声明变量
    val a = 10
    val b = 20
    val c = '*'
    // 如果匹配到 c 将 a 与 b 进行对应运算

    c match {
      case '+' => println(a + b)
      case '-' => println(a - b)
      case '*' => println(a * b)
      case '/' => println(a / b)
      case _ => println("没有规则")

    }
    
  }
}

```

##### Math匹配对象

```scala
package chapter08

object Scala02_TyoeMath {
  def main(args: Array[String]): Unit = {
    // 匹配类型
    //    TODO 需求： 提供任意数据类型，根据数据类型进行匹配，返回不同的结果
    // 传统if + else 也能完成
//    def func1(x: Any): String = {
//      if(x.isInstanceOf[String]){
//        "String"
//      }self(x.isInstanceOf[Int]){
//        "Int"
//      }
//      "d"
//    }

  }
}

```

```scala
package chapter08

object Scala03_ObjectMath {
  def main(args:5 Array[String]): Unit = {
    // 声明对象

    val array1 = Array(1, 2, 3)
    // 针对数组进行匹配
    array1 match {
      case Array(1, 2, 3) =>println(s"shuzu OLK")
      case _ => println("shuzu NONONO")
    }

    val p1 = new Person03("jerry", 12)

    p1 match {
      case Person03("jerry", 12) =>println("ddddddddddddd")
    }
    

  }
}

case class Person03(val name1: String, val age1: Int){
//  var name: String = "xhangsan"
//  var age: Int = 12
}
//object Person03{
//  def apply(name: String, age: Int): Person03 = new Person03(
//    name, age
//  )
//
//  def unapply(arg: Person03): Option[(String, Int)] = {
//    if (arg == null){
//      None
//    }else{
//      Some(Person03.name, Person03.age)
//    }
//  }
//}
```

##### 偏函数

```scala
package chapter08

object Scala04_MatchPartial {
  def main(args: Array[String]): Unit = {
    // 全函数: 会把集合中的所有元素都进行处理
    // 偏函数: 部分函数 调用一个函数来处理集合数据， 可以根据业务需求只处理一部分

    val list1 = List(1, 2, 3, 4, 5, 6, 7, 'a', 'b', "hello", "nihao")
    // 判断List中的元素， 如果是int类型就加一， 如果是String就拼接   //
    // 传统实现方式
    val list2: List[Any] = list1.map(x => {
//      if (x.isInstanceOf[Int]) {
//        x.asInstanceOf[Int] + 1
//
//      } else if (x.isInstanceOf[String]) {
//        x.asInstanceOf[String] + "//"
//      }else{
//        x
//      }

      // 使用模式匹配实现

      x match {
        case s: String =>s + "//"
        case s: Int => s + 1
        case _ => x
      }
    })
    println(list2)

    // 使用函数实现
    val list4: List[Any] = list1.map({
      case s: String => s + "//"
      case s: Int => s + 1
      case s => s
    })
    println(list4)

    val list5 = List(1, 2, 3, 4, 5, "hadoop", "flume")
    // 需求2； 如果是Int类型 加一， 如果是String 就过滤掉
    // 传统方案处理: 先过滤 后 映射
    // 1. 过滤
    val list51: List[Any] = list5.filter(x => {x.isInstanceOf[Int]})
    println(list51)

    // 2. 完成的映射
    val list52: List[Int] = list51.map(x => x.asInstanceOf[Int] + 1)
    println(list52)

    // 以上功能是否能否一步搞定

    // 报错： map是一个全函数， 不能处理部分操作
//    val list53: List[Int] = list5.map({
//      case x: Int => x + 1
//
//    })
//    println(list53)

    // TODO Scala中偏函数的处理
    val list53: List[Int] = list5.collect({
      case x: Int => x + 100
    })
    println(list5)
    println(list53)

  }
}
```

##### 异常处理

```scala
package chapter09

object Scala01_ExceptionTest {
  def main(args: Array[String]): Unit = {
    // Scala中的异常

    try {
      1 / 0
    } catch {
      case e: ArithmeticException => println("异常信息222： " + e.getMessage)
      case e: RuntimeException => println("异常信息111： " + e.getMessage)

    } finally {
      println("关闭资源")
    }

    println("其他diamante在运行......")


  }
}
```

##### 隐式转换

1. `implicit`： 修饰的变量， 对象， 类中的方法，编译器会在找默认方法的时候，自动找到自己定义的默认隐式转换

```scala
package Chapter10

object Scala01_Implament {
  def main(args: Array[String]): Unit = {
    // TODO 隐式函数的转换
    val a: Int = 100

    // 通过一个隐式函数的使用让 扩展类和 Int 产生关联
    // 当编译器第一次编译失败的时候，会在当前的环境中查找能让代码编译通过的方法，
    // 用于将类型进行转换，实现二次编译，用于拓展类的方法。

    implicit  def changeMyInt(i: Int): MyRichInt = {
      new MyRichInt(i)
    }

    val i1: Int = a.max(200)
    val i2: Int = a.myMax(200)
    println(i1)
    println(i2)

    // TODO 隐式参数
    implicit var str: String = "init WangWu"
    def sayHi(implicit name:String = "lisi") = {
      println(s"hai, $name")
    }
    sayHi("zhangsan !!!!")
    sayHi()
    sayHi

  }


  // 定义扩展类：
  class MyRichInt(val self: Int){
    def myMax(x: Int):Int = {
      if (x > self) x else self
    }
  }
}
```

##### 泛型（了解）

```scala
package chapter11

object Scala01_Generic {
  def main(args: Array[String]): Unit = {
    // 泛型： 了解就行
    // 不变： T   【指的是Son类 是Father的子类， MyList[Son] 和MyList[Father] 】： 相当于解除了父子关系
    // +T ：协变： Son类 是Father的子类， MyList[Son] 和MyList[Father]：  两者是父子关系, 和前边关系保持一致
    // -T 逆变： Son类 是Father的子类， MyList[Son] 和MyList[Father]：  两者是父子关系, 父子关系反转
    val f: MyList[Father] = new MyList[Father]
    val s: MyList[son] = new MyList[son]

    // 通过多态的方式实现

	// val s1: MyList[Father] = new MyList[son]

    val s2: MyList[son] = new MyList[Father]

  }
}

class Father {

}

class son extends Father {

}

// 泛型写法：
// T 不变
// +T : 协变
// -T ： 逆变
class MyList[-T] {

}

```



















