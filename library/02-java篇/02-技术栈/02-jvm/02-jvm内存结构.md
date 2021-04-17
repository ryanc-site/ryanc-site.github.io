# jvm 内存结构

>本文所有内容基于: 	<span style="color:#42B983;font-weight:bold;">HotSpot    jdk1.8</span>	
>
>时间：<span style="color:#42B983;font-weight:bold;">2021年4月7日14:15:21</span>

## 一、简述

java 虚拟机的内存空间可以分为（基于jvm设计规范的概念）：</br>

<span style="color:#42B983;font-weight:bold;">五部分：</span><span style="color:#FC5531; font-weight:bold;">程序计数器、java虚拟机栈、本地方法栈，堆 和 方法区</span> **直接内存**</br>



![jvm-memory-structure.jpg](https://github.com/doocs/jvm/blob/main/docs/images/jvm-memory-structure.jpg?raw=true)

---



![image-20210411165306022](amWiki/images/lib_img/image-20210411165306022.png)

---

![image-20210411165421771](amWiki/images/lib_img/image-20210411165421771.png)

<span style="color:#FC5531; font-weight:bold;">扩展：</span>

<span style="border-bottom: 2px solid #0081EF;">JDK 1.8 同 JDK 1.7 比，最大的差别就是：</span><span style="border-bottom: 2px solid #0081EF;color:#FC5531;">元数据区取代了永久代。</span></br>

<span style="border-bottom: 2px solid #0081EF;">元空间的本质和永久代类似，都是对 JVM 规范中方法区的实现。</span></br>

<span style="border-bottom: 2px solid #0081EF;">不过元空间与永久代之间最大的区别在于：元数据空间并不在虚拟机中，而是使用本地内存。</span></br>



## 二、程序计数器

### 1、定义

<span style="color:#42B983;font-weight:bold;">	程序计数器</span>  是一块较小的内存空间，是当前线程正在执行的那条字节码指令的地址。</br>

若当前线程正在执行的是一个本地方法，那么此时程序计数器为`Undefined`。</br>

每个线程都具有各自独立的程序计数器，所以该区域是  <span style="color:#FC5531; font-weight:bold;">★非线程共享的内存区域★</span></br>

### 2、作用

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制。
- 在多线程情况下，程序计数器记录的是当前线程执行的位置，从而当线程切换回来时，就知道上次线程执行到哪了。

### 3、特点

- 是一块较小的内存空间。
- 线程私有，每条线程都有自己的程序计数器。
- 生命周期：随着线程的创建而创建，随着线程的结束而销毁。
- 是唯一一个  <span style="color:#FC5531; font-weight:bold;">不会出现`OutOfMemoryError`的内存区域。</span>

## 三 、java虚拟机栈

### 1、定义

<span style="color:#42B983;font-weight:bold;">Java 虚拟机栈</span>  是描述 Java 方法运行过程的内存模型。

Java 虚拟机栈会为每一个即将运行的 Java 方法创建一块叫做“栈帧”的区域，用于存放该方法运行过程中的一些信息，如：

- 局部变量表
- 操作数栈
- 动态链接
- 方法出口信息
- ......

![image-20210411165529231](amWiki/images/lib_img/image-20210411165529231.png)

<span style="color:#FC5531; font-weight:bold;">扩展：</span> 

* 栈帧：局部变量表、操作栈、动态链接、方法返回地址

  * <span style="color:#42B983;font-weight:bold;">局部变量表</span>

    存放方法参数和局部变量
    相对于类属性变量的准备阶段和初始化阶段来说，局部变量没有准备阶段，必须显式初始化
    如果是非静态方法，则在index[0]位置上存储的是方法所属对象的实例引用，随后存储的是参数和局部变量
    字节码指令中的STORE指令就是将操作栈中计算完成的局部变量写回局部变量表的存储空间内

  * <span style="color:#42B983;font-weight:bold;">操作栈</span>

    操作栈是一个初始状态为空的桶式结构栈 在方法执行过程中，会有各种指令往栈中写入和提取信息 JVM的执行引擎是基于栈的执行引擎，其中的栈指的就是操作栈 字节码指令集的定义都是基于栈类型的,栈的深度在方法元信息的stack属性中

  * <span style="color:#42B983;font-weight:bold;">动态链接</span>

    每个栈帧中包含一个在常量池中对当前方法的引用，目的是支持方法调用过程的动态连接

  * <span style="color:#42B983;font-weight:bold;">方法返回地址</span>

    方法执行时有两种退出情况 正常退出 正常执行到任何方法的返回字节码指令，如RETURN、IRETURN、ARETURN等 异常退出 无论何种退出情况，都将返回至方法当前被调用的位置。方法退出的过程相当于弹出当前栈帧



### 2、压栈出栈过程

当方法运行过程中需要创建局部变量时，就将局部变量的值存入栈帧中的局部变量表中。

Java 虚拟机栈的栈顶的栈帧是当前正在执行的活动栈，也就是当前正在执行的方法，PC 寄存器也会指向这个地址。只有这个活动的栈帧的本地变量可以被操作数栈使用，当在这个栈帧中调用另一个方法，与之对应的栈帧又会被创建，新创建的栈帧压入栈顶，变为当前的活动栈帧。

方法结束后，当前栈帧被移出，栈帧的返回值变成新的活动栈帧中操作数栈的一个操作数。如果没有返回值，那么新的活动栈帧中操作数栈的操作数没有变化。

> 由于 Java 虚拟机栈是与线程对应的，数据不是线程共享的，因此不用关心数据一致性问题，也不会存在同步锁的问题。



### 3、特点

- 局部变量表随着栈帧的创建而创建，它的大小在编译时确定，创建时只需分配事先规定的大小即可。在方法运行过程中，局部变量表的大小不会发生改变。</br>
- Java 虚拟机栈会出现两种异常：StackOverFlowError 和 OutOfMemoryError。</br>
  - <span style="border-bottom: 2px solid #0081EF;color:#FC5531;">StackOverFlowError</span> <span style="border-bottom: 2px solid #0081EF;">若 Java 虚拟机栈的大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度时，抛出 StackOverFlowError 异常。</br></span>
  - <span style="border-bottom: 2px solid #0081EF;color:#FC5531;">OutOfMemoryError</span> <span style="border-bottom: 2px solid #0081EF;">若允许动态扩展，那么当线程请求栈时内存用完了，无法再动态扩展时，抛出 OutOfMemoryError 异常。</span></br>
- Java 虚拟机栈也是线程私有，随着线程创建而创建，随着线程的结束而销毁。</br>

> 出现 StackOverFlowError 时，内存空间可能还有很多。




## 四、本地方法栈

### 1、定义

<span style="color:#42B983;font-weight:bold;">本地方法栈</span>  是为 JVM 运行 Native 方法准备的空间，由于很多 Native 方法都是用 C 语言实现的，所以它通常又叫 C 栈。它与 Java 虚拟机栈实现的功能类似，只不过本地方法栈是描述本地方法运行过程的内存模型。



### 2、栈帧变化过程

本地方法被执行时，在本地方法栈也会创建一块栈帧，
用于存放该方法的局部变量表、操作数栈、动态链接、方法出口信息等。

方法执行结束后，相应的栈帧也会出栈，并释放内存空间。
也会抛出 StackOverFlowError 和OutOfMemoryError 异常。

> 如果 Java 虚拟机本身不支持 Native 方法，或是本身不依赖于传统栈，那么可以不提供本地方法栈。如果支持本地方法栈，那么这个栈一般会在线程创建的时候按线程分配。

## 五、堆

### 1、定义

<span style="color:#42B983;font-weight:bold;">堆 </span> 是用来存放对象的内存空间，<span style="color:#FC5531; font-weight:bold;"> 几乎 </span>所有的对象都存储在堆中。

### 2、特点

对于大多数应用来说，Java 堆（Java Heap）是Java 虚拟机所管理的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。

堆内存是所有线程共有的，可以分为两个部分：<span style="color:#FC5531; font-weight:bold;"> 年轻代</span> 和 <span style="color:#FC5531; font-weight:bold;"> 老年代</span>。

<span style="border-bottom: 2px solid #0081EF;">Perm代表的是永久代，但是注意永久代并不属于堆内存中的一部分</span>，<span style="border-bottom: 2px solid #0081EF;color:#FC5531;">同时jdk1.8之后永久代已经被移除。</span>

![image-20210411165604127](amWiki/images/lib_img/image-20210411165604127.png)

- 线程共享，整个 Java 虚拟机只有一个堆，所有的线程都访问同一个堆。而程序计数器、Java 虚拟机栈、本地方法栈都是一个线程对应一个。
- 在虚拟机启动时创建。
- 是垃圾回收的主要场所。
- 进一步可分为：新生代（Eden 区：`From Survior`，`To Survivor`）、老年代。

不同的区域存放不同生命周期的对象，这样可以根据不同的区域使用不同的垃圾回收算法，更具有针对性。

堆的大小既可以固定也可以扩展，但对于主流的虚拟机，堆的大小是可扩展的，因此当线程请求分配内存，但堆已满，且内存已无法再扩展时，就抛出 OutOfMemoryError 异常。

> Java 堆所使用的内存不需要保证是连续的。而由于堆是被所有线程共享的，所以对它的访问需要注意同步问题，方法和对应的属性都需要保证一致性。

## 六、方法区

### 1、定义

<span style="color:#42B983;font-weight:bold;">方法区 ：</span>Java 虚拟机规范中定义方法区是堆的一个逻辑部分。方法区存放以下信息：

- 已经被虚拟机加载的类信息
- 常量
- 静态变量
- 即时编译器编译后的代码

### 2、特点

- 线程共享。 方法区是堆的一个逻辑部分，因此和堆一样，都是线程共享的。整个虚拟机中只有一个方法区。
- 永久代。 方法区中的信息一般需要长期存在，而且它又是堆的逻辑分区，因此用堆的划分方法，把方法区称为“永久代”。
- 内存回收效率低。 方法区中的信息一般需要长期存在，回收一遍之后可能只有少量信息无效。主要回收目标是：对常量池的回收；对类型的卸载。
- Java 虚拟机规范对方法区的要求比较宽松。 和堆一样，允许固定大小，也允许动态扩展，还允许不实现垃圾回收。

### 3、运行时常量池

方法区中存放：类信息、常量、静态变量、即时编译器编译后的代码。常量就存放在运行时常量池中。

当类被 Java 虚拟机加载后， .class 文件中的常量就存放在方法区的运行时常量池中。而且在运行期间，可以向常量池中添加新的常量。如 String 类的 `intern()` 方法就能在运行期间向常量池中添加字符串常量。



## 七、直接内存

### 1、定义

<span style="color:#42B983;font-weight:bold;">直接内存 </span> 并不是虚拟机内存的一部分，也不是Java虚拟机规范中定义的内存区域。

jdk1.4中新加入的NIO，引入了通道与缓冲区的IO方式，它可以调用Native方法直接分配堆外内存，这个堆外内存就是本机内存，不会影响到堆内存的大小。

### 2、操作直接内存

在 NIO 中引入了一种基于通道和缓冲的 IO 方式。它可以通过调用本地方法直接分配 Java 虚拟机之外的内存，然后通过一个存储在堆中的`DirectByteBuffer`对象直接操作该内存，而无须先将外部内存中的数据复制到堆中再进行操作，从而提高了数据操作的效率。

<span style="border-bottom: 2px solid #0081EF;color:#FC5531;">直接内存的大小不受 Java 虚拟机控制，但既然是内存，当内存不足时就会抛出 OutOfMemoryError 异常。</span>

### 3、直接内存与堆内存比较

- 直接内存申请空间耗费更高的性能
- 直接内存读取 IO 的性能要优于普通的堆内存。
- 直接内存作用链： 本地 IO -> 直接内存 -> 本地 IO
- 堆内存作用链：本地 IO -> 直接内存 -> 非直接内存 -> 直接内存 -> 本地 IO

> 服务器管理员在配置虚拟机参数时，会根据实际内存设置`-Xmx`等参数信息，但经常忽略直接内存，使得各个内存区域总和大于物理内存限制，从而导致动态扩展时出现`OutOfMemoryError`异常。



## 附录

### 面试经

1、JVM管理的内存结构是怎样的？
2、不同的虚拟机在实现运行时内存的时候有什么区别？
3、运行时数据区中哪些区域是线程共享的？哪些是独享的？
4、除了JVM运行时内存以外，还有什么区域可以用吗？
5、堆和栈的区别是什么？
6、Java中的数组是存储在堆上还是栈上的？
7、Java中的对象创建有多少种方式？
8、Java中对象创建的过程是怎么样的？
9、Java中的对象一定在堆上分配内存吗？
10、如何获取堆和栈的dump文件？

解惑：[JVM内存结构的面试题_Virgil_K2017的博客-CSDN博客](https://blog.csdn.net/virgil_k2017/article/details/98851768)

### 参考

👉 [jvm/01-jvm-memory-structure.md at main · doocs/jvm (github.com)](https://github.com/doocs/jvm/blob/main/docs/01-jvm-memory-structure.md)

👉 [JAVA高级面试题——2019_longzhutengyue的博客-CSDN博客_java 高级面试题](https://blog.csdn.net/longzhutengyue/article/details/95534447)





