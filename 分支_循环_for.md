## 每日一考_day04

##### 列出变量的使用注意事项(至少6点)

1) 必须要有数据类型和变量名 

2) 先声明, 后使用

3) 变量中的数据只能在其数据类型范围内变化.

4) 变量有其作用范围, 由其声明语句所隶属的一对{}决定 

5) 同一个作用范围内, 变量不可以重复声明.

6) 变量必须要初始化后才可以使用.



##### 变量分类有两种分法, 

- 第一种是按数据类型来分,另外一种是按照声明位置来分, 
- 每一种又各分为哪些种类型. 各有什么特点? 并用的16进制形式写出所有整数数据类型的最小值和最大值

按照数据类型来分

​	1) 基本数据类型  保存数据本身

​		1) 数值型 

​			1) 整数

​				byte		1000 0000 ~ 0111 1111 => 0x80 ~ 0x7F

​				short		1000 0000 0000 0000 ~ 0111 1111 1111 1111 => 0x8000 ~ 0x7FFF

​				char		0000 0000 0000 0000 ~ 1111 1111 1111 1111 => 0x0000 ~ 0xFFFF

​				int		    0x80000000 ~ 0x7FFFFFFF

​				long		0x80000000_00000000 ~ 0x7FFFFFFF_FFFFFFFF

​			2) 浮点数

​		2) 布尔型 

​	2) 引用数据类型 保存对象地址, 某个字节的编号, 占用8字节(64位jdk)

按照声明位置来分

​	1) 局部变量 声明在方法中的变量,  寿命短, 范围小

​	2) 成员变量 声明在类中方法外的变量, 范围大, 寿命长

##### 计算下列结果, 分析过程, 只需要计算到十六进制形式即可.

byte a = 0x6B; // 0110 1011
byte b = 0x5D; // 0101 1101
System.out.println(a & b); // 0100 1001 => 0x49
System.out.println(a | b); // 0111 1111 => 0x7F
System.out.println(a ^ b); // 0011 0110 = >



##### 相同字面量十六进制表示的数比十进制要大, 对或错? 为什么?

错, 

当数字是在10以内时, 0x5, 5 是一样的

当数字是在2位或以上时, 0x20, 20 16进制比10大, 权值计算以16为底

当这样的数字出现时 0x80 , 80, 不一定, 如果0x80是int,short,long时 它是128

如果0x80是byte型, 它是-128



##### 运算符%的作用是什么? 有什么实际的应用?

取余或取模

​	1) m % n 结果总是要在0~n-1.

​	2) m % n 结果如果为0, 说明m能被n整除, 结果不为0, 则说明m不可以被n整除

​	3) m % 2 结果如果为0, 说明m是偶数, 结果不为0, 说明m是奇数.







## switch 分支

- 如果没有break语句, 一旦某个case进入, 会导致后面的case不起作用, 相当于无条件的执行所有case中的语句都要执行，如果在执行中又遇到了break, 也会中断.
- fall through, 称为穿透, 这样的分支就是复杂分支...
- 如果完全没有break, 它就不是分支, 但是如果还有一些break, 它也是分支.
- switch用于某个变量的等值判断.， 内在逻辑 就是对变量的值的可能性进行列举或穷举.
- switch 括号中必须是变量
- case 后面必须跟常量, 100, 200, 'a', 字面量和被final修饰的量.
- case 标签值不可以重复. 原因是它要生成goto语句会出问题.

##### 代码实例

```java
class SwitchTest2 {
	
	public static void main(String[] args) {
		// 如果没有break语句, 一旦某个case进入, 会导致后面的case不起作用, 相当于无条件的执行所有case中的语句都要执行
		// 如果在执行中又遇到了break, 也会中断.
		
		// fall through, 称为穿透, 这样的分支就是复杂分支...
		// 如果完全没有break, 它就不是分支, 但是如果还有一些break, 它也是分支.
		int n = Integer.parseInt(args[0]);
		switch (n) {
			case 1 :
				System.out.println("n == 1");
				//break;
			case 2 :
				System.out.println("n == 2");
				//break;
			case 3 :
				System.out.println("n == 3");
				break;
			case 5 :
				System.out.println("n == 5");
				//break;
			case 10 :
				System.out.println("n == 10");
				break;
			default :
				System.out.println("else");
				//break;
		}
	}
}

public class SwitchTest {
	
	public static void main(String[] args) {
		/*
		int n = 2;
		
		if (n == 1) {
			System.out.println("n == 1"); 
		} else if (n == 2) {
			System.out.println("n == 2");
		} else if (n == 3) {
			System.out.println("n == 3");
		} else if (n == 5) {
			System.out.println("n == 5");
		} else if (n == 10) {
			System.out.println("n == 10");
		} else {
			System.out.println("else");
		}
		*/
		int n = 3;
		switch (n) {
			case 1 :
				System.out.println("n == 1");
				break;
			case 2 :
				System.out.println("n == 2");
				break;
			//case 2 :
			//	System.out.println("n == 2.... 2");
			//	break;
			case 3 :
				System.out.println("n == 3");
				break;
			case 5 :
				System.out.println("n == 5");
				break;
			case 10 :
				System.out.println("n == 10");
				break;
			default :
				System.out.println("else");
				break;
		}
		/* switch用于某个变量的等值判断.
		内在逻辑 就是对变量的值的可能性进行列举或穷举.
		switch (变量) { 括号中必须是变量. 数据类型是 非long整数, String, 枚举.
			case 常量1 :  // if (变量 == 常量1)
				语句块1;
				break; // break的作用是中断switch结构, 结束它.
			case 常量2 : // else if (变量 == 常量2)
				语句块2;
				break;
			case 常量3 : 
				语句块3;
				break;
			case ....
			default : 
				语句块n;
		}
		switch 括号中必须是变量
		case 后面必须跟常量, 100, 200, 'a', 字面量和被final修饰的量.
		case 标签值不可以重复. 原因是它要生成goto语句会出问题.
		*/
	}
}
```



## 循环

- 在条件满足的情况下, 反复的执行特定的代码的功能.

##### 循环的四个部分 : 

​	1) 初始化语句(init) : 作准备工作

​	2) 循环条件 (test) : 决定循环生死. 

​	3) 循环体 : 被多次执行的代码

​	4) 迭代语句(alter) : 让循环趋于结束.

##### 使用if实现冒泡排序

- 一层层的判断

~~~java
class Work12 {
	
	public static void main(String[] args) {
		int num1 = Integer.parseInt(args[0]);
		int num2 = Integer.parseInt(args[1]);
		int num3 = Integer.parseInt(args[2]);
		
		// 目标 : 所有数据都是左小右大.
		if (num1 > num2) {
			int tmp = num1;
			num1 = num2;
			num2 = tmp;
		}
		// 此时num2中保存了前2个数中的最大
		if (num2 > num3) {
			int tmp = num2;
			num2 = num3;
			num3 = tmp;
		}
		// 此时num3中保存了所有数中的最大, num3搞定
		
		if (num1 > num2) {
			int tmp = num1;
			num1 = num2;
			num2 = tmp;
		}
		// 此时num2中保存了前2个数中的最大, num2搞定
		
		
		System.out.println(num1 + "," + num2 + "," + num3);
	}
}

~~~

##### while循环

~~~java
class LoopExer2 {
	
	public static void main(String[] args) {
		int n = Integer.parseInt(args[0]);
		
		int i = 0; // 初始化
		while (i < n) { // 循环条件
			System.out.println("********"); // 循环体
			i++; // 迭代语句 
		}
	}
}
~~~

##### do-while循环

~~~java
class LoopExer4 {
	
	public static void main(String[] args) {
		int n = Integer.parseInt(args[0]);
		int i = 0;
		do {
			System.out.println("********");
			i++;
		} while (i < n);
	}
}
~~~

##### for循环+分支

~~~java
// 练习 : 求1000以内能被13整除的数的平均值
class LoopExer7 {
	
	public static void main(String[] args) {
		int sum = 0;
		int count = 0;
		for (int i = 0; i < 1000; i++) {
			if (i % 13 == 0) {
				System.out.println("i : " + i);
				sum += i;
				count++;
			}
		}
		// 统计运算, 它的特点就是只能一个结果. 必须在循环外出最终结果.
		double avg = (double)sum / count * 1.0;
		System.out.println("avg : " + avg);
		
	}
}	
~~~







































