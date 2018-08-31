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
