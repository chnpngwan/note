# 面向对象

##### 包的介绍

```scala
// TODO 当前包下有且只能有一个包对象
// TODO 包对象的作用: 综合管理一些当前包下的共享资源
// 保对象中 不能访问其他对象中的资源
package object chapter06 {

  // 定义属性
  val packageName: String = "chapter06"

  // 定义方法
  def sayHi(name: String) = {
    println(s"hi $name")
  }


  def main(args: Array[String]): Unit = {

  }
}

```

##### 包对象的使用

- 包下的所有类可以共享包对象管理的所有资源

```scala
package chapter06

object Scala01_package {
  def main(args: Array[String]): Unit = {
    // 获取当前包下的共享资源
    println(packageName)

  }
}
```

##### 类的介绍和定义

- @BeanProperty：添加注解自动添加get与set方法

```scala
package chapter06

import scala.beans.BeanProperty

object Scala03_classs {
  def main(args: Array[String]): Unit = {

  }
  // 内部类
  class Perdon01{
    def main(args: Array[String]): Unit = {
      val s =  new Person04

    }

  }


}

// 外部类
class Person03{

}
// 外部类
class Person023{

  // 在Scala中 封装属性与方法
  // Scala中默认不提供get与set方法， 但是scala也对属性提供了读写的权限，
  //  底层自己已经实现了读写权限的方法， 属性私有， 提供public的方法（var修饰）
  // @BeanProperty : 注解添加get与set方法

  @BeanProperty
  var name: String = "zhangsan"

  @BeanProperty
  var age : Int = 12

  def sayHi(name: String) = {
    println(s"hi $name")
  }
}

```

##### 类的构造方法

- 辅助构造器第一行会调用主构造器【难点重点】
- 辅助构造器：主构造器方法重载，
- 掌握新建类对象的代码执行顺序

```scala
package chapter06

// 构造器
object Scala04_Constructor {
  def main(args: Array[String]): Unit = {
    //    var person0 = new Person04("zhangsan")
    val person02 = new Person04(12)


  }
}

// 定义一个类
// Scala中提倡完全面向对象，
// TODO Scala中，针对构造器分两类， 分为主构造器和辅助构造器
// 主构造器： 当前类签名， 主构造器的方法名与类名共用一个,
//    针对当前的对象进行实例化或者构造的时候
//    一定会使用到主构造器

// 辅助构造器: 主要作用，用来补充Scala中构造方法的重载特性

// TODO 000这样设计构造器， 是为了满足在实例化类的时候， 首先将类中的内容都加载一次
class Person04(name: String) {
  println(s"主构造器被调用了。。。")

  // 声明辅助构造器, 所有辅助构造器的方法名固定 [this]
  // 直接调用主构造器
  def this() = {
    this("lisi")
    // 辅助构造器有一个原则， 方法体的第一行就要调用主构造器
    println(s"辅助构造器被调用了。。。")
  }

  // 间接调用主构造器
  def this(i: Int) = {
    // 首先间接调用主构造器, 通过调用另一个辅助构造器调用主构造器
    this()
    println(s"辅助构造器2调用了、、、")
  }

}
```

##### 构造器参数变属性

- 需要使用val/var修饰

```scala
package chapter06

// 构造器
object Scala04_ConstructorArgs {
  def main(args: Array[String]): Unit = {
    val person1 = new Person041("张三")
    println(person1.name1)
    println(person1.name)
  }
}


// 定义类
// TODO 如果想让构造器中的参数直接成为当前类的属性， 必须使用val或者var关键字进行修饰
class Person041(val name: String) {
  println(s"$name")

  // 间接将参数转化为当前类的属性
  val name1: String = name
}
```

##### Scala中的继承

- 使用子类辅助构造器。执行顺序为：父类主构造器- ==> 子类主构造器 ==> 子类辅助构造器
- 当显试的调用父类的辅助构造器的时候会调用

```scala
package chapter06

object Scala06_Extendes {
  def main(args: Array[String]): Unit = {
    // 1. 子类继承父类， 子类可以使用父类的属性与方法

//    val student0 = new student06()
//
//    println(student0.name)

    // 2. 当构造子类的时候， 关注代码执行顺序
    val student1 = new student06(144)
  }
}

// 定义父类
class Person06 {
  println(s"父类的主构造器调用。。。。")

  var name: String = _
  var fname: String = "父类的name"


  // 父类辅助构造器
  def this(name: String) {
    this()
    this.name = name
    println(s"name: $name")
    println(s"父类的辅助构造器被调用了。。。。")
  }

}

// 定义子类
// 继承的时候参数的使用注意事项，
// 1. 使用继承的时候，可以将参数传递给父类的构造器
// 2. 子类主构造器的参数的数量 一定不能少于父类构造器中的参数数量
class student06(name: String, age1: Int) extends Person06(name: String) {
  println(s"子类的主构造器调用了。。。")

  var age: Int = _

  // fuzhu 辅助构造器
  def this(age: Int) {
    this("lisi", 100)
    this.age = age
    println(s"子类的辅助构造器调用..。。")

    println(s"$fname")
  }
}
```

##### Scala抽象属性与方法

- 抽象方法中的抽象属性， 子类可以重写， 不考虑【val或者var】修饰
- 匿名子类体现多态

```scala
package chapter06

// 抽象属性与抽象方法
object Scala07_Abs {
  def main(args: Array[String]): Unit = {

    // 抽象类的用法
    // 1. 直接实例化子类， 让子类继承抽象类
    val student0 = new student07

    student0.sayHi("wangwu")
    println(student0.name)
    println(student0.age)

    // 2. 直接实例化抽象类(获取匿名子类)
    // 匿名子类是多态的一种体现（父类指向子类实例）
    // 匿名子类使用的时候要注意：匿名子类中出现独有的属性或方法， 外部无法访问， 内部可以访问
    val person0: Person07 = new Person07 {
      override val name: String = "zjamgsan"
      override var age: Int = 100

      override def sayHi(name: String): Unit = {
        info(name)
        println(s"nihao $name")
      }

      // 当前匿名子类独有的方法
      def info(in: String) = {
        println(s"info , $in")
      }

      //      info("nihao")

    }
    person0.sayHi("大壮")
  }
}

abstract class Person07 {

  // 抽象属性(Scala中独有的属性)
  val name: String
  var age: Int

  // 抽象方法
  def sayHi(name: String): Unit

}

// 定义子类， 继承抽象类
class student07 extends Person07 {
  override val name: String = "zhangsan"
  override var age: Int = 12

  override def sayHi(name: String): Unit = {
    println(s"Hi $name")

  }
}
```

##### Scala中的Override

- 抽象类中的非抽象属性， val修饰的属性，直接重写即可
  - var修饰的属性， 直接重新赋值就行
  - var本身就是可变类型

```scala
package chapter06

// 抽象类与子类的重写相关
object Scala08_OverRide {
  def main(args: Array[String]): Unit = {

  }
}

// dingyi 定义抽象列
abstract class Person08 {
  // 抽象属性
  val name: String
  var age: Int

  // 非抽象属性
  val name1: String = "person08"
  var age1: Int = 18

  // 非抽象方法
  def sayHi(): Unit = {
    println("hai Person08")
  }
  //  抽象方法
  def sayHi1(): Unit

}

class student08 extends Person08 {
  override val name: String = "student08"
  override var age: Int = 20

  override def sayHi1(): Unit = {
    println("Hi  student08")
  }

  // 对父类中的非抽象内容也可以重写
  override val name1 = "fdj"
  // TODO val修饰的参数可以重写。var修饰的变量不可以重写
  //  val修饰的变量是不可变的， 可以通过重写的方式将值进行变更， 但是var本身是可修改的
  //  如果想变更内容， 直接休改内容就可以

  override def sayHi(): Unit = super.sayHi()

}

```

##### Scala中的Apply

- apply方法默认伴生对象中的方法用来实例化对象（类被设置为私有的）
  - apply方法可以自定义， 重写方法，实现自己的逻辑
- apply方法可以简化调用， 直接Person09() 实现 new一个子类

```scala
package chapter06

// Scala中的apply（）方法的使用
object Scala09_Apply {
  def main(args: Array[String]): Unit = {
    // shilih实例化Person09
    //    val person0 = new Person09
    //    println(person0.name)

    val person0: Person09 = Person09.getPerson09()
    println(person0.name)
    // 传统调用
    val person1 = Person09.apply()

    // jianhua 简化调用
    val person02: Person09 = Person09()
    println(person02.name)

    Person09.apply("张三")
    Person09("王五")

    // apply方法在Scala中的具体使用实例很多
    val ints = List(1, 2, 3, 4, 5, 2)

    // TODO 扩展，考虑通过赋值构造器对私有化的主构造器进行调用
    val zhang01 = new Person09("张庄")
    println(zhang01.name)

  }
}

// 定义一个类
class Person09 private() {
  val name: String = "zhangsan"

  def this(name: String) = {
    this()

  }
}

// 定义伴生对象
// 在当前一个类的伴生对象中可以访问当前类中的所有内容
object Person09 {

  // 此方法的作用：用来实例化Person09
  def getPerson09(): Person09 = {
    new Person09
  }

  // 以上功能可以通过apply方法完成
  // TODO apply方法不仅仅局限于创建对象的功能， 也支持方法的重载
  def apply(): Person09 = new Person09()

  def apply(name: String) = {
    println(s"say Hi $name")
  }
}
```

##### Scala中的多态

- Scala中继承之后， 子类会继承父类的属性与方法

```scala
package chapter06

// Scala中的多态
object Scala10_Poly {
  def main(args: Array[String]): Unit = {

    // scala多态的使用中， 父类与子类的同名的属性与方法， 继承后都会使用子类的
    //  与Scala的继承体系有关系
    val peop10:Person10 = new student10
    println(peop10.name)
    peop10.sayHi()
  }
}

// fu父类

class Person10{
  val name: String = "Person10"
  def sayHi() = {
    println(s"Person10")
  }
}
// fu子类
class student10 extends Person10{
  override val name: String = "student10"

  override def sayHi(): Unit = {
    println(s"student10")
  }
}

```

##### Scala中的Trait【特质】->接口

（1）特质可以同时拥有抽象方法和具体方法
（2）一个类可以混入（mixin）多个特质
（3）所有的Java接口都可以当做Scala特质使用
（4）动态混入：可灵活的扩展类的功能
（4.1）动态混入：创建对象时混入trait，而无需使类混入该trait
（4.2）如果混入的trait中有未实现的方法，则需要实现

- 先继承父类， 后with特质
- 父类与特质中出现同名的抽象属性， 子类继承后直接重写就可以使用
  - 如果出现非抽象的同名属性。 val修饰的直接重写才可使用
  - var修饰的 无解， 要去修改原码

```scala
package chapter06

// Scala中的特质使用
object Scala11_Trait {
  def main(args: Array[String]): Unit = {
    val student1 = new Student111
    student1.sayHi1()


    println("==============")
    println(student1.name1)
//    println(student1.agene)

    //  （1）特质可以同时拥有抽象方法和具体方法
    //  （2）一个类可以混入（mixin）多个特质
    //  （3）所有的Java接口都可以当做Scala特质使用
    //  （4）动态混入：可灵活的扩展类的功能
    //  （4.1）动态混入：创建对象时混入trait，而无需使类混入该trait
    //  （4.2）如果混入的trait中有未实现的方法，则需要实现

    // 动态混入==匿名子类的使用
    val stu112: student112 with Age11 = new student112 with Age11 {
      override val name: String = "112"
      override var age: Int = 122

      override def sayHi(): Unit = {
        println(s"112, $name, $age,$sex,  hai!!!")
      }

      val sex: String = "nan"

    }


  }

}

// 定义普通类
class student112{

}

// 特质声明定义
trait Age11 {
  // TODO te特质中可以定义抽象的属性与方法， 也可以定义非抽象的属性，方法
  // 抽象内容
  val name: String
  var age: Int

  def sayHi(): Unit

  // 定义非抽象的内容
  val name1: String = "name1"
  var age1: Int = 10

  def sayHi1() = {
    println(s"Hi $name, $age")
  }
}

trait Weight11 {
  val name1: String = "weight11"
//  var agene: Int = 90
}

// 定义父类
class Parent11 {
  val name1: String = "person11"
//  var agene: Int = 90
}

// 声明子类， 继承特质
// TODO 支持多继承
// meiyou 没有父类
//  （2）一个类可以混入（mixin）多个特质
class Student11 extends Age11 with Weight11 {
  override val name: String = "student11"
  override var age: Int = 20
  override val name1: String = "syu111============="

  override def sayHi(): Unit = {
    println(s"hi---- $name, $age")
  }

//  agene = 99999
}

// 有父类的情况下
// TODO 有父类必须将父类放在最前边
//3）所有的Java接口都可以当做Scala特质使用
// TODO 有父类的情况注意： 当父类中的属性与混入的特质的属性同名, 执行会报错
// 出现以上情况， 如果val修饰变量， 子类重写解决
//               如果是var修饰的变量，无解， ==>不能出现var修饰的同名属性

class Student111 extends Parent11 with Age11 with Weight11 with java.io.Serializable {
  override val name: String = "student11"
  override var age: Int = 20

  override val name1: String = "student11jiji-----"

  override def sayHi(): Unit = {
    println(s"hi---- $name, $age")
  }

//  agene = 99999
}

```

##### 6.6.3特质和抽象类的区别

- （1）优先使用特质。一个类扩展多个特质是很方便的，但却只能扩展一个抽象类。
- （2）如果你需要构造函数参数，使用抽象类。因为抽象类可以定义带参数的构造函数，而特质不行（有无参构造）