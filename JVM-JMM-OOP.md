# Java内存模型、Java对象模型和JVM内存结构
**Java内存模型**、**Java对象模**型和**JVM内存结构**这三个知识点，可以说是一个Java初中级程序员进阶高级程序员的一个必经过程。这里将这三个不同的概念进行梳理。
    
## JVM内存结构
Java代码是运行在JVM之上，也就是Java为什么可以一次编译，到处运行的主要原因。JVM在执行Java的过程中会把所管理的内存划分为若干个不同的数据区域，每个区域都有不同的用途。下图是JVM运行时的内存结构。
![enter image description here](http://wx2.sinaimg.cn/mw690/006YxpfSgy1fuomv1bju1j30mk0dldg1.jpg)
有些区域随着虚拟机进程的启动而存在，有些区域依赖用户线程的启动和结束而建立和销毁。
- 运行时常量池用于存放编译器生成的各种字面量和符号应用，但并不是所有常量只有在编译期才能产生，比如在运行期，`String.intern`也会把新的常量放入池中。
- 除了以上JVM运行时内存外，还有直接内存可以供使用，Java虚拟机规范没有定义直接内存，不由JVM管理，是利用本地方法库直接在堆外申请的内存区域。
- 堆和栈的数据划分也不是绝对的。

> 总结，JVM内存结构，由Java虚拟机规范定义。描述的是Java程序执行过程中，由JVM管理的不同数据区域。各个区域有其特定的功能。

## 内存模型（JMM）
Java堆和方法区的区域是多个线程共享的数据区域，也就是说，多个线程可能可以操作保存在堆或者方法区中的同一个数据。这也就是我们常说的“Java的线程间通过共享内存进行通信”。
    
JMM是和多线程相关的，他描述了一组规则或规范，这个规范定义了一个线程对共享变量的写入时对另一个线程是可见的。

总结下，**Java的多线程之间是通过共享内存进行通信的**，而由于采用共享内存进行通信，在通信过程中会存在一系列如可见性、原子性、顺序性等问题，而JMM就是围绕着多线程通信以及与其相关的一系列特性而建立的模型。JMM定义了一些语法集，这些语法集映射到Java语言中就是`volatile`、`synchronized`等关键字。
    
![enter](http://wx4.sinaimg.cn/mw690/006YxpfSgy1fuonhhtsllj30br0ah0st.jpg)

## Java对象模型
Java是一种面向对象的语言，而Java对象在JVM中的存储也是有一定的结构的。而这个关于Java对象自身的存储模型称之为Java对象模型。

HotSpot虚拟机中，设计了一个OOP-Klass Model。OOP（Ordinary Object Pointer）指的是普通对象指针，而Klass用来描述对象实例的具体类型。

每一个Java类，在被JVM加载的时候，JVM会给这个类创建一个instanceKlass，保存在方法区，用来在JVM层表示该Java类。当我们在Java代码中，使用new创建一个对象的时候，JVM会创建一个instanceOopDesc对象，这个对象中包含了对象头以及实例数据。
    
![](http://wx3.sinaimg.cn/mw690/006YxpfSgy1fuoo1ncsn2j31840js429.jpg)

### 总结
- JVM内存结构，和Java虚拟机的运行时区域有关。
- Java内存模型，和Java的并发编程有关。
- Java对象模型，和Java对象在虚拟机中的表现形式有关。