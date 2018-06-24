## i++的线程安全问题

看的这个问题的时候？我的第一反应是：i++还有线程安全问题？？

先看一下下面的一个例子，来看看i++到底是不是线程安全的：
```
// 1000个线程，每个线程对共享变量count进行1000次++操作
static int count = 0;
static CountDownLatch cdl = new CountDownLatch(1000);

public static void main(String[] args) throws Exception {
    CountRunnable countRunnable = new CountRunnable();
    for (int i = 0; i < 1000; i++) {
        new Thread(countRunnable).start();
    }
    cdl.await();
    System.out.println(count);
}

static class CountRunnable implements Runnable {
    private void count() {
        for (int i = 0; i < 1000; i++) {
            count++;
        }
    }

    @Override
    public void run() {
        count();
        cdl.countDown();
    }
}
```
上面的例子我们期望的结果应该是 1000000，但运行 N 遍，你会发现总是不为 1000000，至少你现在知道了 i++ 操作它不是线程安全的了。

下面是JMM模型中对共享变量的读写原理
![JMM](https://github.com/toheng/JavaSatck/blob/master/images/jmm.jpg)

**每个线程都有自己的工作内存，每个线程都需要对共享变量操作时，必须要先将共享变量从主内存中加载到自己的工作内存，等到完成共享变量的操作时，再保存到主内存中。**

问题就在这里，如果一个线程运算完成后，没有保存到主内存中，此时这个共享变量的值被另一个线程从主内存中读取到了，这时候读取的数据就是脏数据了，它会覆盖其他线程计算完的值。

如果把count加上`volatile`让内存可见，这种方式也不能解决，因为`volatile`**只能保证可见性**，**不能保证原子性**。多个线程同时读取到这个共享变量的值，就算保证了其他线程修改的可见性，也不能保证线程之间读取到同样的值，并相互覆盖对方的值的情况。

**解决方案：**
1. 对i++操作的方法加同步锁，同时只能有一个线程执行i++操作；
2. 使用原子操作的类，如`java.util.concurrent.atomic.AtomicInteger`，它使用的是CAS算法，效率优于第一种。

