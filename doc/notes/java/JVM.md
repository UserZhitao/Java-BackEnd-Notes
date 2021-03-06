# TODO
* ASM框架
* 程序计数器，本地方法区
* 字节码增强
* 类加载机制，双亲委派模型
* JVM的内存模型  
* 内存泄漏和内存溢出
* java的垃圾回收机制   
* 讲一下JVM的内存分区 
* 《深入理解JVM虚拟机》垃圾回收原理 
* Java的逻辑分区 
* 方法区啥的?一个成员变量存在哪里?如果是局部变量是一个对象的引用呢?存在哪里?

# Jvm

## 堆和栈的区别



| 类型 | 堆  | 栈 |  
|---| ----- | -------- | 
|功能| 堆存储Java对象(成员变量,局部变量，类变量) | 存储局部变量和方法 | 
|共享性| 线程共有 | 线程私有  |
|异常错误| StackOverFlowError |OutOfMemoryError |
<!-- * 功能不同
    * 堆存储Java中的对象（成员变量，局部变量，类变量）。
    * 栈用来存储局部变量（方法内部的变量）和方法。
* 共享性不同
    * 栈的内存是线程私有的。(方法相关的当然私有啊！！)
    * 堆的内存是线程共有的。
* 异常错误不同
    * 栈空间不足：java.lang.StackOverFlowError。 经典！
    * 堆空间不足：java.lang.OutOfMemoryError。对象存满了 -->

### 栈的组成

栈三部分:
* 局部变量区
    * 结构:以一个字长为单位，从0开始计数的数组。
    * 类型为short、byte和char会被转换成为int
        long和double占据两个连续的元素。
    * 实例方法只是多了一个隐藏的this。
    * 获取数据直接取索引。
* 操作数栈
    * 和局部变量一样，也是字长为1的数组，不是通过索引是用出栈和入栈来决定的。还记得迪杰斯特拉吗？！
    ![](https://iamjohnnyzhuang.github.io/public/upload/4.png)
* 帧数据区
    * Java栈帧还需要一些数据来支持常量池的解析，正常方法的返回。
    * 处理方法的正常结束和异常终止。通过return来正常结束的话，就弹出当前的栈帧，恢复发起调用方法的栈。如果方法有返回值JVM就会将返回值压入调用方法的操作数栈中。
    * 处理异常:保存了一个对此方法异常引用表的引用。
* 栈的整个结构

    ```java
    public class Main {
    public static void addAndPrint(){
        double result = addTwoTypes(1, 88.88);
        System.out.println(result);
    }

    public static double addTwoTypes(int i, double d) {
        return i + d;
        }
    }
    ```
    
    
    过程快照：
    ![](https://iamjohnnyzhuang.github.io/public/upload/5.png)


如果是方法中的局部变量就存储在堆中。

### 堆
当一颗二叉树的每个节点都大于等于它的两个子节点时，称作堆有序。
用执政表示一个二叉树的话就是一个完全二叉树。

* 由上到下的堆有序:
    * 上浮:只要记住位置k节点的父亲节点是位置k/2
        * 插入元素：将新元素上浮到适合位置
    * 下沉:
        * 删除最大元素：从数组顶端删除最大元素，选择数组最后节点。



### 方法区
是一个各个线程共享的内存区域，用于存储已经被虚拟机加载的类信息，常量，静态变量，及时编译器编译后的代码等数据。别名`Non-Heap`，目的是与Java堆区分开来。

### 运行时常量池
运行时候常量池是方法区的一部分，Class文件除了类的版本，字段，方法等描述信息，还有一个信息是(Constant Pool Table),用于存放字面量和符号引用，这部分将在类加载后进入方法区的运行常量池中。 
类的信息，方法名，方法参数信息。

## GC
`finalize()`：用来发现对象是否存在有没有没有被清除的部分。一旦垃圾回收器准备好释放对象占用的空间时，首先调用这个方法，并且在下一次垃圾回收动作发生时，才会真正回收内存。

基于DFS的垃圾回收技术：对于活的对象，一定可以从堆栈或者静态存储区中找到它的引用，然后遍历找到对象，在寻找对象的引用，往复如此，就能得到所有活的对象。

* "停止"----"复制":暂停程序运行，将存活对象复制到另外一个堆，没有复制的都是垃圾。
 搬运的同时，所有指向对象的引用都必须修正，位于 堆和静态存储区的引用可以直接被修正。
    * 需要两个堆，按需从堆分配几块较大的内存，复制动作发生在大块内存之间。
    * 程序稳定没有垃圾时， 切换到"标记 ----清扫"模式，在垃圾少的时候速度很快。
* "标记 ----清扫":DFS遍历所有活的对象，并设置标记， 结束后，没有标记的对象释放， 但是剩下的堆空间是不连续的 ，所以得重新整理对象。

Java虚拟机会监控垃圾回收器的效率， 如果效率低就采用"标记 ----清扫"，如果内存中碎片较多就会采用"停止 ---- 复制"，这样的自适应策略。
    

## 对象存在的周期

### 对象创建
1. 首先先再常量池定位某个类的符号引用
2. 内存所需要的大小需要类加载完成后才能知道 
3. 对对象必要设置

### 垃圾回收器的工作原理
垃圾回收器工作时一面回收空间，一面使堆中的对象紧凑排列，这样堆指针就能够更容易的移动到传送带开始的地方。

## 反射
**Java 反射机制**：在程序运行的时候，对于任意的一个类，都能够知道这个类的所有属性和方法，对于任意一个对象可以调用它的任意属性和方法这种动态获取信息以及动态调用对象的方法的功能叫做Java反射(reflect)。

