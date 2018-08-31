#理解并发处理Volatile关键字
Java中有一系列用于处理并发的关键字，比如**synchronized**、**volatile**、**final**、**concurren**包等，解决并发编程中存在的原子性、可见性、有序性问题。
本文主要就`volatile`的用法、原理、以及它是如何提供可见性和有序性保障的。
## volatile的用法
`volatile`只能用来修饰变量，无法修饰方法以及代码块等。
只需要在声明一个可能被多线程同时访问的变量时，使用`volatile`进行修饰。
```
// 双重锁校验实现单例
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
        if (singleton == null) {  
            synchronized (Singleton.class) {  
                if (singleton == null) {  
                    singleton = new Singleton();  
                }  
            }  
        }  
    return singleton;  
    }  
}  
```

## volatile的原理
我们经常听到CPU有多少级缓存这个参数，缓存级数越多，说明这个CPU的执行速度越快，多级缓存是介于CPU和内存之间的。由于引进了多级缓存，就会存在缓存数据不一致的问题。
如果某个变量被`volatile`修饰，这个变量进行写操作时，JVM会向处理器发送一条lock前缀的指令，将这个缓存的变量会写到系统内存中。
但是这样也存在一个问题，如果其它处理器缓存的值还是没更新过的，再执行计算，就会出现数据不一致的问题。在多处理器下，为了保证处理器的缓存一致，就会实现`缓存一致性协议`。
> **缓存一致性协议**：每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器要对这个数据进行修改操作的时候，会强制重新从系统内存里把数据读到处理器缓存里。

所以，如果一个变量被`volatile`所修饰的话，在每次数据变化之后，其值都会被强制刷入主存。而其他处理器的缓存由于遵守了缓存一致性协议，也会把这个变量的值从主存加载到自己的缓存中。这就保证了一个`volatile`在并发编程中，其值在多个缓存中是可见的。

## 可见性
> 可见性是指多个线程同时访问同一个变量时，其中一个线程修改了该变量的值，其它线程可以立即看见修改后的值。

Java内存模型规定了，所有变量都存储在主内存中每条线程有自己的工作内存，线程的工作内存中存储了在主内存中变量的副本，线程对变量的操作必须在工作内存中进行，不能直接读写主内存。不同的线程之间不能直接访问对方工作内存中的变量，线程间的变量传递都需要自己的工作内存和主内存之间进行数据同步。这时候就出现了，**某一个线程修改了工作内存中的变量值，而其它线程不可见修改后的变量**。
**如果这个变量被`volatile`修饰了，被修改过的变量会被立即同步到主内存中，被`volatile`修饰后的变量在线程使用之前都会先从主内存中刷新。所以，`volatile`关键字可以保证多线程操作变量时的可见性**。

## 有序性
> 有序性是指程序执行的顺序会按照代码的先后顺序执行。

由于处理器的优化和指令重排，CPU会优化代码的执行顺序，可能会对输入的代码进行乱序执行。
`volatile`可以禁止指令重排序优化。

## 原子性
> 原子性是指一个操作不可中断，要么全部执行完成，要么不执行。
因为CPU有时间片的概念，会根据不同的调度算法进行现场调度，但一个现场获取时间片后开始执行，在时间片用完，就是失去对CPU的使用权，在多线程的场景下，由于时间片在线程中轮换，就会发生原子性问题。
`synchronized`为了保证原子性，需要通过字节码指令`monitorenter`和`monitorexit`，但是`volatile`和这两个指令之间是没有任何关系的。
`volatile`不能保证原子性。

在以下两个场景中可以使用`volatile`来代替`synchronized`：

> 1、运算结果并不依赖变量的当前值，或者能够确保只有单一的线程会修改变量的值。
> 2、变量不需要与其他状态变量共同参与不变约束。

除以上场景外，都需要使用其他方式来保证原子性，如**synchronized**或者**concurrent包**。


我们来看一下volatile和原子性的例子：
```
public class Test {
    public volatile int inc = 0;
    public void increase() {
        inc++;
    }
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
        while(Thread.activeCount()>1)                 //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```
创建10个线程，然后分别执行1000次i++操作。正常情况下，程序的输出结果应该是10000，但是，多次执行的结果都小于10000。这其实就是volatile无法满足原子性的原因。

**因为虽然volatile可以保证inc在多个线程之间的可见性。但是无法inc++的原子性**。