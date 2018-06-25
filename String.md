## 解析String类型
> String，是除了基本数据类型以外，最为重要的一个类型了，而且在笔试中经常可以遇到。很多人认为String很简单，但事实上很多人容易被String的笔试题弄的晕头转向。

```
Q1: String s = new String("Hello")
A1: 如果常量池中已经存在"Hello"，就会直接引用，这时只会创建一个对象。
	如果常量池中不存在"Hello"，则会先创建后引用，这时会定义两个对象。

Q2: 关于String的intern方法
A1: 当一个String实例调用intern()方法时，JVM会查找常量池中是否有相同Unicode的字符串常量，如果有，则返回其的引用，如果没有，则在常量池中增加一个Unicode等于str的字符串并返回它的引用；
```

两个答案看上去没有任何问题，但是，仔细想想好像哪里不对呀。

按照上面的两个面试题的回答，就是说new String会检查常量池，如果有的话就直接引用，如果不存在就要在常量池创建一个，那么还要intern干啥？难道以下代码是没有意义的吗？
```
String s = new String("Hello").intern();
```
如果，每当我们使用new创建字符串的时候，都会到字符串池检查，然后返回。那么以下代码也应该输出结果都是`true`?
```
	String s1 = "Hello";
    String s2 = new String("Hello");
    String s3 = new String("Hello").intern();

    System.out.println(s1 == s2);    // false (jdk1.8)
    System.out.println(s1 == s3);    // true  (jdk1.8)
```

### 字面量和运行时常量池
JVM为了提高性能和减少内存开销，在实例化字符串常量的时候进行了一些优化。为了减少在JVM中创建的字符串的数量，字符串类维护了一个字符串常量池。

在JVM运行时区域的方法区中，有一块区域是运行时常量池，主要用来存储**编译期**生成的各种**字面量**和**符号引用**。

了解Class文件结构或者做过Java代码的反编译的朋友可能都知道，在java代码被`javac`编译之后，文件结构中是包含一部分`Constant pool`的。比如以下代码：
```
public static void main(String[] args) {
    String s = "Hello";
}
```
经过编译后，常量池内容如下：
```
Constant pool:
   #1 = Methodref    #4.#20      // java/lang/Object."<init>":()V
   #2 = String       #21         // Hello
   #3 = Class        #22         // StringDemo
   #4 = Class        #23         // java/lang/Object
   ...
   #16 = Utf8               s
   ..
   #21 = Utf8               Hello
   #22 = Utf8               StringDemo
   #23 = Utf8               java/lang/Object
```
上面的Class文件中的常量池中，比较重要的几个内容：
```
   #16 = Utf8               s
   #21 = Utf8               Hello
   #22 = Utf8               StringDemo
```
上面几个常量中，`s`就是前面提到的符号引用，而`Hello`就是前面提到的**字面量**。而Class文件中的常量池部分的内容，会在运行期被运行时常量池加载进去。关于字面量，详情参考Java SE Specifications。

### new String创建了几个对象
下面，我们可以来分析下`String s = new String("Hello");`创建对象情况了。

这段代码中，我们可以知道的是，在编译期，**符号引用**`s`和**字面量**`Hello`会被加入到Class文件的常量池中，然后在类加载阶段，这两个常量会进入常量池。

但是，这个“进入”过程，并不会直接把所有类中定义的常量全部都加载进来，而是会做个比较，如果需要加到字符串常量池中的字符串已经存在，那么就不需要再把字符串字面量加载进来了。

所以，当我们说<若常量池中已经存在"Hello"，则直接引用，也就是此时只会创建一个对象>说的就是这个字符串字面量在字符串池中被创建的过程。

说完了编译期的事，该到运行期了，在运行期，`new String("Hello");`执行到的时候，是要在Java堆中创建一个字符串对象的，而这个对象所对应的字符串字面量是保存在字符串常量池中的。但是，`String s = new String("Hello");`，**对象的符号引用`s`是保存在Java虚拟机栈上的，它保存的是堆中刚刚创建出来的的字符串对象的引用**。

所以，你也就知道以下代码输出结果为false的原因了。
```
String s1 = new String("Hollis");
String s2 = new String("Hollis");
System.out.println(s1 == s2);
```
因为，`==`比较的是`s1`和`s2`在堆中创建的对象的地址，当然不同了。但是如果使用`equals`，那么比较的就是字面量的内容了，那就会得到`true`。

