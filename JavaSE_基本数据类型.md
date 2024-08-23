## 每日一考_day02

#### 1)写出java语言的8个特性

1) 简单 : 相比于C/C++

2) 面向对象 : 关注具有功能的对象.

3) 分布式 : 基于网络多主机协作

4) 健壮 : 强类型(所有数据都必须要有数据类型约束), 异常处理, GC, 指针的安全化.

5) 安全 : 类加载器负责对每个要加载的类进行检查验证.

6) 跨平台 : java程序只依赖JVM执行.

7) 性能好 : 编译型比解释型快, 编译型:把源文件统一编译成可执行文件, 解释型:直接基于源文件执行

8) 多线程 : 高并发, 最大化利用cpu.



#### 2)描述一下语句,类,和方法之间的关系.

类是java程序的基本单位(一个程序至少得是一个类)

方法是java程序中的一个独立的功能单位, 方法必须隶属于类

语句是java程序中的最小执行单位, 语句必须隶属于方法.



类 {

​	方法 {

​		语句1

​		...

​	}

​	方法2 {}

​	...

}



#### 3)什么是主类, 什么是公共类, 公共类有什么注意点?

主类 : 包含了主方法的类, public static void main(String[] args) {}

java 主类

公共类 : 被public修饰的类, 公共类的类名必须要和文件名一致. 所以一个源文件中只能有一个公共类.

#### 4)变量是什么?

内存中的一块被命名的且被特定数据类型约束空间, 此空间中可以保存一个符合数据类型的数据 . 此空间中的数据也可以随意在数据类型的范围内变化.

#### 5)数据类型的作用是什么?

1.	决定了空间的大小
2.	决定了空间中可以保存什么数据, 及数据的范围
3.	决定了空间中的数据可以做什么.

## 变量 : 

​	数据类型 变量名;



### 变量按照数据类型来分 : 

#### 	1) 基本数据类型(primitive) : 内存空间中保存的就是数据本身(itself)

##### 		1) 数值型 

###### 			整数

​				byte 	1

​				short	2

​				int	     4

​				long	  8

###### 			浮点数

​				float	  4

​				double      8

##### 		2) 布尔型

​				boolean    1

##### 		3) 字符型

​				char	   2



![1649813995473](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1649813995473.png)

​	数据类型的选取 原则是 不要浪费空间, 不要空间不够.	



#### 2) 引用数据类型(reference) : 内存空间中保存的是别的数据的地址(address)



变量  : 

​	内存中的一块空间, 可以保存一个数据

变量声明 : 

​	数据类型 变量名;

变量使用注意事项  : 

​	1) 必须要有数据类型和变量名



### 数值型之间的处理

// 结论 : 如果右面的量值的数据类型的范围小于或等于左面的变量的数据类型的范围, 是可以直接完成
	// 结论 : 如果右面的量值的数据类型的范围大于左面的变量的数据类型的范围, 不可以直接完成, 必须要加上强制类型转换
	// 兼容性从小到大 : byte < short < int < long < float < double 
	// 浮点数字面量默认使用的8个字节double型来存储
	// 整数字面量默认使用4个字节int型来存储 

```java
class DataTypeTest2 {
	
	// 结论 : 只要是非long整数作运算, 至少要有一个变量参与, 结果一定是int型, 因为底层的指令只有iXXX.
	// 如果有多种不同类型的变量的混合运算时, 结果的类型是参与的变量中的范围最大的那个类型
	public static void main(String[] args) {
		byte b1 = 10;
		short s1 = 20;
		//s1 = b1 + s1; // iadd 
		s1 = (short)(b1 + s1); // iadd 
		
		byte b2 = 5 + 8; // 这个虽然有运算, 但是没有变量参与, 所以可以直接完成
		int i1 = 30;
		
		b1 = (byte)(b1 + i1);
		
		long l1 = 40L;
		//i1 = l1 + i1;
		i1 = (int)(l1 + i1); // long 强转为 int 
		i1 = (int)l1 + i1; // 
		
		float f1 = 3.2f;
		double d1 = 5.7;
		
		//l1 = f1 + l1; 右面的结果是float型
		l1 = (long)(f1 + l1);
		
		d1 = d1 + l1; // ?
	}
}

public class DataTypeTest {
	// 结论 : 如果右面的量值的数据类型的范围小于或等于左面的变量的数据类型的范围, 是可以直接完成
	// 结论 : 如果右面的量值的数据类型的范围大于左面的变量的数据类型的范围, 不可以直接完成, 必须要加上强制类型转换
	// 兼容性从小到大 : byte < short < int < long < float < double 
	// 浮点数字面量默认使用的8个字节double型来存储
	// 整数字面量默认使用4个字节int型来存储 
	public static void main(String[] args) {
		// b1和s1在底层其实就是int型, 如果在代码中特别指定了其他类型, 编译时需要注意一下
		byte b1 = 10; // 10是int型, 因为10是常量, 永不改变. 这个值也正好可以被byte兼容, 这个操作编译器认为是可以的.
		// 如果这个值不可以被byte兼容, 编译出错!!
		short s1 = 20;
		int i1 = 30;
		long l1 = 40_0000_0000L; // 40亿, 后缀L的作用就是一个提醒, 提醒编译器这个整数字面量使用8字节的long型空间来保存
		
		//b1 = s1; // 这个有错, 编译器认为这是变量, 不靠谱, 所以不可以直接赋值
		
		byte b2 = b1; // 相同类型的数据之间可以直接赋值
		s1 = (short)b1;
		
		short s2 = b1;
		
		s1 = 150;
		//b1 = s1; 右面的数据类型的范围如果大于左面的变量的数据类型的范围,不可以直接赋值, 必须要使用强制类型转换
		b1 = (byte)s1; // 强制类型转换有风险
		System.out.println(b1);
		
		i1 = s1;
		//s1 = i1;
		s1 = (short)i1;
		
		l1 = i1;
		i1 = (byte)l1; // 可以
		
		int i2 = 50; // 50也是一个数据, 称为常量 : 不允许变化的量, 常量包括字面量, 被final修饰的量
		// 100, 'A', true, 300.2, "alksdjf" 这些都是字面量, 所以即所得, 也称为即时数.
		//50 = 50; // 左面的常量 不允许 写入, 所以出错!!!
		// 结论 : 赋值符号的左面必须是变量.
		
		double d1 = .328; 
		//float f1 = d1; float型变量接不住double型的值
		float f1 = (float)d1;
		// 浮点数字面量默认就是double类型.
		//float f2 = 3.88;
		float f2 = (float)3.88;
		float f3 = 3.88F; // 后缀F的作用就是一个提醒 , 提醒编译器,这个浮点数字面量不要再使用默认的double型, 请使用float型来保存
		
		//l1 = f1; // float型的范围要比long型的大, 所以这个赋值不可以直接完成
		l1 = (long)f1;
		f1 = l1;
		d1 = f1;
		
		// 浮点数转换为整数时, 小数部分直接丢弃!!
		int i3 = (int)5.999;
		System.out.println(i3);
	}
}
```



#### char 数据类型

- char 是基本数据类型, 占用2个字节, 可以保存一个字符, 保存的是这个字符的Unicode编码. 取值范围是0~65535
- char因为保存的就是码值, 可以当作整数来使用. 就是非long整数
- 声明char型并赋值时, 查表, 根据'a' 找到它的码值97, 真正保存在变量中的就是97这个整数.
- 打印char时, 逆操作, 根据里面的码值97再反向查表, 查出对应的字符'a', 再打印'a'

```java
class CharTest2 {
	

	public static void main(String[] args) {
		// 有些字符无法用字面量直接描述.
		char c1 = '\r'; // 13
		char c2 = 10; // '\n'
		char c3 = '\t'; // 9 
		char c4 = '\b'; // 8, 是回退键
		System.out.println("abc");
		System.out.println(c1);
		System.out.println(c2);
		System.out.println(c3);
		
		
		
		System.out.println("xyz\b\b300");//x300
		System.out.println("xyzabc\r300");//300abc
		
		System.out.println("ab\\c\"xyz\"");
	}
}

public class CharTest {
	

	public static void main(String[] args) {
		// char 是基本数据类型, 占用2个字节, 可以保存一个字符, 保存的是这个字符的Unicode编码. 取值范围是0~65535
		// char因为保存的就是码值, 可以当作整数来使用. 就是非long整数
		char c1 = 'a'; // 声明char型并赋值时, 查表, 根据'a' 找到它的码值97, 真正保存在变量中的就是97这个整数.
		char c2 = 'b';
		char c3 = 'A';
		char c4 = '1';
		char c5 = '我';
		char c6 = '你';
		
		System.out.println(c1); // 打印char时, 逆操作, 根据里面的码值97再反向查表, 查出对应的字符'a', 再打印'a'
		System.out.println(c2);
		System.out.println(c3);
		System.out.println(c4);
		System.out.println(c5);
		System.out.println(c6);
		
		System.out.println("**********************");
		System.out.println((int)c1); // 直接把码值升级为int型整数来打印.
		System.out.println((int)c2);
		System.out.println((int)c3);
		System.out.println((int)c4);
		System.out.println((int)c5);
		System.out.println((int)c6);
		
		char c7 = 30000;
		System.out.println(c7);
		
		c7 = (char)(c7 + 200); // 非long整数作运算, 结果会升级为int型.
		System.out.println(c7);
	}
}
```

**能用变量的地方, 绝不用常量(字面量)**

#### String字符串

- 字符串String, 是引用型 , 声明变量, 变量中保存地址.
- 字符串内容不可改变, 但是可以连接任意数据 , 连接的结果就是把任意数据作为新内容串在后面, 并且产生新字符串

```java
class StringTest2 {

	public static void main(String[] args) {
		// String str1 = "" + 4;        //判断对错：
		String str2 = 3.5f + "";         //判断str2对错： true
		System.out.println(str2);        //输出："3.5"
		// 判断+是拼接还是加法, 依据左右2个数的类型
		System.out.println(3 + 4 + "Hello!");      //输出：7Hello!
		System.out.println("Hello!" + 3 + 4);      //输出：Hello!34
		System.out.println('a' + 1 + "Hello!");    //输出：98Hello!
		System.out.println("Hello" + 'a' + 1);         //输出：Helloa1
	}
}

public class StringTest {
	//1 写一个Variable类，main方法中使用double类型声明var1和var2变量，然后用var2保存var1与var2之商。
	//2 声明字符串变量str，用str串接的形式表示上述计算过程并打印输出结果。
	//不要再声明var3, 打印的结果内容要求小学生也能看懂, 也就是说不要出现变量名, 只有表达式即可

	public static void main(String[] args) {
		// 字符串String, 是引用型 , 声明变量, 变量中保存地址.
		String s1 = "abc";
		String s2 = ""; // 空串, 表示有对象, 对象中没有字符. 像空签子
		String s3 = null; // 空, 表示没有对象, 连签子也没有
		
		System.out.println(s1);
		System.out.println(s2);
		System.out.println(s3);
		// 字符串内容不可改变, 但是可以连接任意数据 , 连接的结果就是把任意数据作为新内容串在后面, 并且产生新字符串
		s1 = s1 + 100; // "abc100"
		s1 = s1 + false; // "abc100false"
		double d1 = 3.22;
		s1 = s1 + d1; // "abc100false3.22"
		String s4 = "yyy";
		s1 = s1 + s4; // "abc100false3.22yyy"
		s1 = s1 + '好'; // "abc100false3.22yyy好"
		System.out.println(s1);
		
		
		// int => String : "" + int 
		int n2 = 937;
		String s6 = "" + n2; // "937"
		System.out.println(s6);
				
		double d2 = 92.342;
		String s7 = d2 + "";
		System.out.println(s7);
		
	}
}
```





## 进制

293147

2 * 10 # 5 +

9 * 10 # 4 +

3 * 10 # 3 +

1 * 10 # 2 + 

4 * 10 # 1+ 

7 * 10 # 0



权值 : 

​	用10的n次方 这样的数称为10进制数

10 进制 没有10, 逢10进1  => 9 + 1 => 10

十六进制(hex)

​	权值 : 以16为底的n次方

​	16进制没有16, 逢16进1

八进制

​	权值 : 以8为底的n次方

​	8进制没有8, 逢8进1

二进制(binary)

​	权值 : 以2为底的n次方

​	2进制没有2, 逢2进1.

0x123 =>

1 * 16 # 2 + //权值 

2 * 16 # 1 +

3 * 16 # 0 =  291



0110 1001 =>

0 * 2 # 7 : 128

1 * 2 # 6 : 64

1 * 2 # 5 : 32

0 * 2 # 4 : 16

1 * 2 # 3 : 8

0 * 2 # 2 : 4

0 * 2 # 1 : 2

1 * 2 # 0 = 105



0x69 => 

6 * 16 + 9 = 105



十进制数105和16进制数0x69及2进制的0110 1001, 这3个数是什么关系 ??? 是一回事.



#### 练习 : 

0x6211

6 * 16 # 3

2 * 16 # 2 + 529

1 * 16 # 1 + 

1 * 16 # 0 =

529 + 24,576 =>  25105



0011 1101 

32 + 16 + 8 + 4 + 1 => 61



​     十进制			    二进制			十六进制

​	0				   0000				0

​	1				   0001				1

​	2				   0010				2

​	3				   0011				3

​	4				   0100				4

​	5				   0101				5

​	6				   0110				6

​	7				   0111				7

​	8				   1000				8

​	9				   1001				9

​	10			         1010				A

​	11 				1011				B

​	12				 1100				C

​	13				 1101				D

​	14				 1110				E

​	15				 1111				F



**结论 : 一个16进制数正好可以对应4个bit的二制数, 两个16进制数就可以更方便表示8个bit, 正好一个字节**

0x93 => 

1001 0011

0xE72C

1110 0111 0010 1100

0101 0110 1010 1110 => 

0x56AE



9 : 

8421

0000

1001

0x9C5AD7B2 => 4字节

1001 1100 0101 1010 1101 0111 1011 0010



0101 1101 0101 0011 0110 0101 1001 0100 =>

0x5D536594



练习 : 

0xE3B758C9

1110 0011 1011 0111 0101 1000 1100 1001 



1010 1110 1101 0111 1111 0010 1010 1011

0xAED7F2AB

- **表达式是有值的**