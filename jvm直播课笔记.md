# jvm

![image-20200914202054670](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200914202054670.png)

java特性：跨平台

怎样实现跨平台？

通过jvm实现跨平台

JDK：不同版本的jdk都带有自己的jvm

**jvm：从软件层面屏蔽不同操作系统在底层硬件与指令上的区别**

![image-20200914202351753](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200914202351753.png)

jvm组成部分：

   **1.运行时数据区**

   **2.类装载子系统**

   **3.字节码执行引擎**





## 内存模型

### 栈（线程栈）

   存放局部变量……

   只要一个**方法**开始运行，就会在栈内为当前方法在栈内分配内存空间，就会有自己的线程栈，也叫这个方法对应的栈帧。在这个栈中会存放该方法中的局部变量等。

####  栈帧：

​     **在栈中有无数个栈帧，一个方法对应一个栈帧。**

#### 数据结构中的栈：

​    jvm中的栈和数据结构中的栈有什么联系？

​    数据结构中的栈：FIFO。

​    jvm中的栈的结构就是用了数据结构中的栈。

  **内存分配-->入栈**

  **内存释放-->出栈**

  先入栈的方法首先分配内存，但是后入栈的方法首先执行，首先释放内存。

Q/A:

1. 栈帧内部的组成？  

​      答：**局部变量表，操作数栈，动态链接，方法出口。**

  **局部变量表：**可以将其看成一个数组，为局部变量分配内存。

  **操作数栈： **作为程序运行过程中操作数临时做内存中转的内存空间。可理解为java虚拟机栈中一个用于计算的临时数据区域，在进行计算时操作数栈也是通过弹栈/压栈来访问的。

**（首先将局部变量的值压入操作数栈顶，然后将其取出（即弹栈操作）赋值给局部变量表中对应的内存对应的变量）**。

   

  **动态链接：执行方法时，会解析符号对应的代码所在的方法区的地址。**

​                      把符号引用转为直接引用。

​                       符号引用：

​                       直接引用：

  **方法出口：方法返回值的信息，返回给main方法。**



 ps：栈和堆的联系：

​         栈中会有很多局部变量，某一部分局部变量的引用指向堆。 



   ps：程序计数器：为每一个java程序分配一个内存空间。每个线程执行代码对应的位置（行号）。**每执行一次代码程序计数器的值都修改。**

​           代码是由字节码执行引擎来执行，执行的过程中会动态修改程序计数器的值。

​           **为什么要引入程序计数器？**

​           答：jvm支持多线程，当有多个线程在jvm分配内存，之前的线程可能会被挂起，这时需要执行当前线程，当执行完比之后要回到被挂起的线程对应的内存地址。

  



![image-20200914204354715](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200914204354715.png)

​    jdk自带的命令： javap命令

```
javap -c a.class > a.txt
```



### 方法区/元空间

####   1.常量 

####   2.静态变量 ：

​        如果一个静态变量存的内容是对象，那么它存的是静态变量对应的对象在堆内存中的地址，指向堆。

####   3.类信息



### 本地方法栈：

  某些方法底层是c代码写的。   

（使用很少）

​      执行的c/c++代码。

### 堆：

  新new出的对象一般都要存放到eden区域。如果eden满了之后，会开启yong gc。

  ps：GC root（可达性算法）

​         **根节点：线程栈的本地变量，静态变量，本地方法栈的变量等。**

​        从root结点出发，找到所有被引用到的对象，并将被引用的对象全部放入surivivor区中。

​        一个对象只要经历一次minor gc，年龄就会加1.



Q/A

 1.简述jvm对象从年轻代到老年代的过程：

​       

## jvm调优实战

![image-20200914212740710](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200914212740710.png)            

​     arthas-boot.jar   

#### jvm调优的目的？

答：减少full gc的次数（减少停顿）。

​       gc停顿-->STW，减少STW的次数。

​       STW：在执行垃圾收集线程时会停止所有的用户线程的执行，专心做垃圾收集。时间较长。

#### jvm为什么会有STW机制？

答：如果不设置STW机制，在gc过程中会和用户线程一起执行，在gc结束之后可能会有新的垃圾产生，产生多余垃圾。

​      不同的垃圾收集器，底层实现过程是不同的，但是基本思想一样，基本都用到了STW的模式，只不过可能使用STW的阶段不同。

#### 有没有办法对当前系统调优，让几乎不触发full Gc？

有。扩大eden区。减少full gc的次数。(也是一种调优的思路)



#### 对于单机几十万的高并发的系统，需不需要对minor gc进行调优?

需要。



### 用例分析：

   上亿点击的网站-->日活用户500w左右-->每个用户平均点击20~30次。

   ![image-20200914215157049](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200914215157049.png)

![image-20200914215909455](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200914215909455.png)

![image-20200914220257913](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200914220257913.png)

### 私有部分：栈，本地方法栈，程序计数器 

###    共享部分：堆和方法区

- ###  调优-->主要和堆有关系

  idea中进行性能调优的设置

```
# 堆大小，按常规操作，设成相同的，避免自动扩容
-Xms1536m
-Xmx1536m
# 年轻代大小，Sun推荐设置为堆大小的3/8
-Xmn576m
# 在JVM启动时即预初始化堆中的所有页，能够快速利用
-XX:+AlwaysPreTouch

# 设置一个较大的元空间初始值，避免频繁GC扩容
-XX:MetaspaceSize=256m 
# 元空间最大默认不限制，设一个值保护一下
-XX:MaxMetaspaceSize=768m

# 启用CMS GC
-XX:+UseConcMarkSweepGC
# 启用年轻代并行GC，与CMS是好搭档，其实也不用另写
-XX:+UseParNewGC
# CMS并行标记，降低标记阶段停顿时间
-XX:+CMSParallelRemarkEnabled
# 触发CMS GC的堆内存占用比例，调大点以降低GC频率
-XX:CMSInitiatingOccupencyFraction=85
# GC线程数（ParallelGCThreads、ConcGCThreads）用默认值，不再写

# 对象晋升到老年代的年龄，默认15。根据观察，对IDEA来说设成10就足够了
-XX:MaxTenuringThreshold=10

# 压缩普通对象指针
-XX:+UseCompressedOops

# 指定服务器版JIT编译器，其实不用写，默认已经是了
-server
# JIT代码缓存的大小，默认是240M
-XX:ReservedCodeCacheSize=360M
# 打开JIT分层编译，默认是开启的了
-XX:+TieredCompilation
# 每MB堆空间中的软引用能够存活的近似毫秒数
-XX:SoftRefLRUPolicyMSPerMB=50

# OOM时输出堆dump转储文件
-XX:+HeapDumpOnOutOfMemoryError
# 禁止把某些异常的stack trace优化掉，防止信息被吃了找不到问题
-XX:-OmitStackTraceInFastThrow
# 禁用字节码验证。IDEA的代码足够可靠，不用验证
-Xverify:none
# 启用断言机制（enable assertion）
-ea

-Dfile.encoding=UTF-8
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djdk.http.auth.tunneling.disabledSchemes=""

-XX:ErrorFile=$USER_HOME/java_error_in_idea_%p.log
-XX:HeapDumpPath=$USER_HOME/java_error_in_idea.hprof
```



















![image-20200914212927951](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\image-20200914212927951.png)





