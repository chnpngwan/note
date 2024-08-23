##### 笔记

1. 存储对象与存储序列化的文件：序列化之后可以压缩， 占用的空间小，但是频繁的使用会频繁的压缩与解压缩，消耗CPU
2. 归并排序， 局部有序的文件，合并为全局有序的文件。
3. 栈内存中存储的是栈帧，栈帧中存储指令， 局部变量
4. 堆内存中存储的是程序运行过程中动态创建的对象，
   1. new对象
   2. 反射
   3. 反序列
   4. 克隆：直接在内存复制（数组的扩容：arraycopy，本地方法），是操作系统的方法，
5. 方法区内存处处的是类的全部信息，（反射获得类的全部信息）
6. 不是只有堆内存才可以存储对象，栈内存也可以存储对象，
   - 栈上分配&逃逸分析，不能确定方法弹栈之后，该对象对否还会使用，可能返回值为对象，可能使用的参数是外部传入的对象，
7. Java内存管理，内存回收，【GC-CMS：并行标记清除，默认就是这个】
   - CMS:面向单核的垃圾回收，（Stop The Java Word）垃圾回收的时候，停止所有操作，性能不高
   - G1-GC：面向多核的CPU
8. Spark 的内存管理，
   - 栈内存中的方法， 静态方法区中的静态内容可以访问堆内存中的对象，使用可达性分析，对象不可达的时候就会被回收，
   - finalize（）方法（Object 类中的方法）， 这个方法被重写的话，对象不会被回收，只能被Java 虚拟机认可一次

##### shuffle

![1659089848210](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659089848210.png)

##### 内存管理

![1659089934080](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659089934080.png)

![1659089950238](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659089950238.png)

![1659090084693](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090084693.png)

![1659090095981](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090095981.png)

#### 数仓相关

##### 概念

![1659090222802](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090222802.png)

![1659090241519](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090241519.png)

![1659090252715](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090252715.png)

![1659090480591](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090480591.png)

##### 整体架构

![1659090545864](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090545864.png)

![1659090567696](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090567696.png)

![1659090591814](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090591814.png)

![1659090602736](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090602736.png)

![1659090620436](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090620436.png)

![1659090639946](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090639946.png)

![1659090684397](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090684397.png)

##### 数据仓库-分层架构

![1659090721529](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090721529.png)

![1659090732256](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090732256.png)

![1659090751820](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090751820.png)

![1659090764885](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090764885.png)

![1659090785379](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090785379.png)

![1659090793853](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090793853.png)

![1659090814535](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090814535.png)

![1659090830328](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090830328.png)

![1659090842884](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090842884.png)

![1659090854512](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090854512.png)

![1659090863427](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090863427.png)

![1659090881977](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090881977.png)

![1659090892411](https://gitee.com/chnpngwng/typora-image/raw/master/assets/note/1659090892411.png)

















