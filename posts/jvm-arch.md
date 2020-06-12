### JVM架构图

* class files --> 类加载子系统
* 运行时数据区(Runtime Data Area)
  1. Java虚拟机栈
  2. 本地方法栈
  3. 程序计数器
  4. 方法区
  5. 堆
* 执行引擎
* 本地方法接口
* 本地方法库



### 类加载器

* 几种加载器
  * BootstrapClassLoader, 启动类加载器
  * ExtClassLoader, 扩展类加载器
  * AppClassLoader, 应用程序类加载器
* 双亲委派
  * 当收到一个类加载请求时, 首先不会尝试自己去加载这个类, 而是把这个请求委派给父类加载器去完成, 每一个层次的加载器都是如此, 因此所有的加载请求都会转到启动类加载器, 只有当父类加载器反馈在它的加载路径下没有找到所需要的Class, 子类加载器才会自己去加载。
  * 好处就是这样保证了使用不同类加载器最终得到的都是同一个对象

```java
public class App2 {

    public static void main(String[] args) throws InterruptedException {
        final Data data = new Data();
        System.out.println(data.getClass().getClassLoader());
        System.out.println(data.getClass().getClassLoader().getParent());
        System.out.println(data.getClass().getClassLoader().getParent().getParent());
    }
}

class Data {
}

// 控制台输出
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@67117f44
null
```



### Java虚拟机栈

* 线程私有
* 栈管运行、堆管存储

* 数据结构对比
  * 队列 FIFO
  * 栈 FILO
* 在线程创建时创建, 线程结束也就释放, 对于栈来说不存在垃圾回收问题, 生命周期与线程一致
* 存储数据
  * 本地变量(Local Variables): 输入输出参数、方法内的变量
  * 栈操作(Operand Stack): 记录入栈、出栈操作
  * 栈帧数据(Frame Data): 包括类文件、方法等等

**栈运行原理**

1. 当一个方法`A`被调用时, 就会产生一个栈帧`F1`, 并被压入栈中
2. `A`方法调用了`B`方法时, 于是产生一个栈帧`F2`, 也被压入栈中
3. `B`方法调用了`C`方法时, 于是产生一个栈帧`F3`, 也被压入栈中
4. `C`方法执行完毕后, 弹出`F3`栈帧, 再弹出`F2`栈帧, 再弹出`F1`栈帧, 遵循`先进后出`/`后进先出`

每个方法执行时都会创建一个栈帧, 用于存储局部变量、操作数栈、动态链接、方法出口等信息, **每一个方法从调用直至执行完毕的过程, 就是对应着一个栈帧在虚拟机中入栈到出栈的过程**。栈的具体大小与虚拟机实现有关, 通常几百Kb到1M之间



### 本地方法栈

* 线程私有
* 登记标记为`native`的方法, 在`执行引擎`执行时加载本地方法库



### 程序计数器

* 线程私有
* 每个线程都有一个程序计数器, 用来指向下一条指令的地址, 也即将要执行的指令代码, 由执行引擎读取
* 如果要执行的方法是一个`native`, 那这个计数器是空的
* 用来完成分支、循环、跳转、异常处理、线程恢复等基础功能, 不会发生OOM错误



### 方法区

* 供各线程共享的运行时内存区域
* 存储每一个类的结构信息
  * 运行时常量池
  * 字段和方法数据、构造方法、普通方法的字节码内容
* 方法区是规范(接口)
  * 1.7中为永久代, 使用JVM内存
  * 1.8中为元空间, 不在虚拟机中**而是使用本机物理内存, 受本地内存限制**。类的元数据放入本地内存, `字符串池`与`类的静态变量`放入`Java堆`中, 这样可以加载多少类的元数据就不再由`MaxPermSize`控制, 而由系统的实际可用空间控制



### 堆(Heap)

* 新生代(Young/New)
  * Eden
  * S0
  * S1
* 老年代(Old/Tenure)
* 方法区
  * 永久代(Permanent Space)
  * 元空间



### 常用JVM参数

* -Xms

  * 堆的初始大小, 默认物理内存的`1/64`

* -Xmx

  * 堆的最大内存, 默认物理内存的`1/4`

* -XX:+PrintGCDetails

  * 输出详细GC处理日志

* -XX:MaxTenuringThreshold

  * 主要控制新生代需要经历多少次GC晋升到老年代, 最大值是15

  * > [Java HotSpot VM](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)
    >
    > Sets the maximum tenuring threshold for use in adaptive GC sizing. The current largest value is 15. The default value is 15 for the parallel collector and is 4 for CMS.

* -XX:TargetSurvivorRatio=50

  * 默认值50%, 当一个S区所有的age对象的大小如果大于等于该比例, 则重新计算`threshold`, 以`age`和`MaxTenuringThreshold`两者的`最小值`为准

  * > Desired percentage of survivor space used after scavenge.

  * [-XX:TargetSurvivorRatio](https://www.jianshu.com/p/2e69aa552b01)



```java
public class App2 {

    public static void main(String[] args) {
        System.out.println(Runtime.getRuntime().availableProcessors());
        final long totalMemory = Runtime.getRuntime().totalMemory();
        final long maxMemory = Runtime.getRuntime().maxMemory();
        System.out.println("-Xms:" + totalMemory / 1024D / 1024D + "MB");
        System.out.println("-Xmx:" + maxMemory / 1024D / 1024D + "MB");
    }
}

// 没添加JVM参数输出
12
-Xms:491.0MB
-Xmx:7282.0MB

// 添加JVM参数输出 -Xms1024M -Xmx1024M -XX:+PrintGCDetails
  12
-Xms:981.5MB
-Xmx:981.5MB
Heap
 PSYoungGen      total 305664K, used 20971K [0x00000007aab00000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 262144K, 8% used [0x00000007aab00000,0x00000007abf7afb8,0x00000007bab00000)
  from space 43520K, 0% used [0x00000007bd580000,0x00000007bd580000,0x00000007c0000000)
  to   space 43520K, 0% used [0x00000007bab00000,0x00000007bab00000,0x00000007bd580000)
 ParOldGen       total 699392K, used 0K [0x0000000780000000, 0x00000007aab00000, 0x00000007aab00000)
  object space 699392K, 0% used [0x0000000780000000,0x0000000780000000,0x00000007aab00000)
 Metaspace       used 3083K, capacity 4556K, committed 4864K, reserved 1056768K
  class space    used 324K, capacity 392K, committed 512K, reserved 1048576K
```

