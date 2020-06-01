### 1、CAS

`CAS`全称为`Compare-And-Swap`, 它是一条`CPU并发原语`, 它的功能是**判断内存某个位置的值是否为期望的值, 如果是则更新为新的值**, 这个过程是原子的。

`CAS并发原语`体现在`Java`中就是`sum.misc.Unsafe`类中的各个方法。调用`Unsafe`类中的`CAS`方法, `JVM`会帮我们实现出`CAS汇编指令`。这是一种完全依赖于`硬件`的功能, 通过它实现了`原子操作`。

由于`CAS`是一种`系统原语`, 原语属于操作系统用语范畴, 是由若干条指令组成的, 用于完成某个功能的一个过程, **并且原语的执行必须是连续的, 在执行过程中不允许被中断, 也就是说CAS是一条CPU的原子指令, 不会造成数据不一致的问题**。

**Unsafe**是`CAS`操作的核心类, 由于`Java`方法无法直接访问`底层系统`, 需要通过本地(`native`)方法来访问, 基于该类可以**直接操作特定内存的数据**。`Unsafe`类存在于`sun.misc`包中, 其内部方法可以像`C`的`指针`一样`直接操作内存`, `Java`中`CAS`操作的执行依赖于Unsafe的方法。



### 2、CAS示例

```java
    /**
     * Atomically sets the value to the given updated value
     * if the current value {@code ==} the expected value.
     *
     * @param expect the expected value
     * @param update the new value
     * @return {@code true} if successful. False return indicates that
     * the actual value was not equal to the expected value.
     */
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```

**例子**

```java
public class App {

    public static void main(String[] args) {
        final AtomicInteger atomicInteger = new AtomicInteger(10);
        final boolean first = atomicInteger.compareAndSet(10, 2020);
        System.out.println(first + "\t" + atomicInteger);
        final boolean second = atomicInteger.compareAndSet(10, 1024);
        System.out.println(second + "\t" + atomicInteger);
    }
}
```

**输出**

```java
true	2020
false	2020
```



### 3、底层原理

[Where can I find the source code for native methods in Java library?](https://stackoverflow.com/questions/30251032/where-can-i-find-the-source-code-for-native-methods-in-java-library/30251098)

**sun.misc.Unsafe#getAndAddInt**

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

1. `var1` `AtomicInteger`对象本身
2. `var2`该对象值的引用地址
3. `Var4`需要变动的数值
4. `var5`是通过`var1`, `var2`找出的主内存中真实的值
5. 用当前对象中的值与`var5`比较
   1. 如果相同, 则更新为`var4+var5`, 并且返回`true`
   2. 如果不同, 继续第4、5步, 直到更新完成整个流程

**native source code**

```cpp
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```



### 4、CAS缺点

1. 循环时间长`CPU`开销大
   * 如果`Compare`长时间一直不成功, 可能会给`CPU`带来很大的开销
2. 只能保证`一个变量`的`原子操作`
3. 存在`ABA`问题
   * CAS算法实现一个重要的前提就是取出内存中`某个时刻`的数据与`当下时刻`的数据进行比较, 那么在这个时间差内会导致数据的变化



### 5、ABA问题

比如`线程1`、`线程2`都同时取出值`A`, `线程2`通过一系列操作将值修改为`B,` 然后又修改为`A`, 这个时候`线程1`进行`CAS`操作发现内存中的值任然为`A`, 然后`线程1`操作成功。

**尽管线程1也操作成功了, 但是并不代表这个过程就没有问题。**



### 6、AtomicReference

```java
public class App {

    public static void main(String[] args) {
        AtomicReference<User> userAtomicReference = new AtomicReference<>();

        final User zs = new User(1L, "张三");
        final User ls = new User(2L, "李四");
        userAtomicReference.set(zs);

        final boolean first = userAtomicReference.compareAndSet(zs, ls);
        System.out.println(first + "\t" + userAtomicReference);
        final boolean second = userAtomicReference.compareAndSet(zs, ls);
        System.out.println(second + "\t" + userAtomicReference);
    }
}

@Data
@AllArgsConstructor
class User {
    private Long uid;
    private String username;
}

// 控制台输出
true	User(uid=2, username=李四)
false	User(uid=2, username=李四)
```



### 7、ABA问题解决

**ABA问题演示**

```java
public class App {

    public static void main(String[] args) {
        AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
        new Thread(() -> {
          	// 重现一次ABA
            final boolean first = atomicReference.compareAndSet(100, 101);
            System.out.println(first + "\t" + atomicReference);
            final boolean second = atomicReference.compareAndSet(101, 100);
            System.out.println(second + "\t" + atomicReference);
        }, "t1").start();

        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
						
          	// 依然能够更新成功, 说明AtomicReference不能解决ABA问题
            final boolean first = atomicReference.compareAndSet(100, 2020);
            System.out.println(first + "\t" + atomicReference);
        }, "t2").start();
    }
}

// 控制台输出
true	101
true	100
true	2020
```



**ABA问题解决**

```java
public class App {

    public static void main(String[] args) {
        AtomicStampedReference<Integer> atomicStampedReference =
                new AtomicStampedReference<>(100, 1);
        new Thread(() -> {
            System.out.println(Thread.currentThread()
                    + "\t第1次版本号: " + atomicStampedReference.getStamp()
                    + "\t" + atomicStampedReference.getReference());

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            // 重现一次ABA问题
            atomicStampedReference.compareAndSet(100, 101,
                    atomicStampedReference.getStamp(),
                    atomicStampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread()
                    + "\t第2次版本号: " + atomicStampedReference.getStamp()
                    + "\t" + atomicStampedReference.getReference());
            atomicStampedReference.compareAndSet(101, 100,
                    atomicStampedReference.getStamp(),
                    atomicStampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread()
                    + "\t第3次版本号: " + atomicStampedReference.getStamp()
                    + "\t" + atomicStampedReference.getReference());
        }, "t1").start();

        new Thread(() -> {
            // 以局部变量固化stamp
            final int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread()
                    + "\t第1次版本号: " + stamp
                    + "\t" + atomicStampedReference.getReference());

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            // 按原始版本号更新, 更新失败, 解决ABA问题
            final boolean result = atomicStampedReference.compareAndSet(
                    100, 2020,
                    stamp, stamp + 1);
            System.out.println(Thread.currentThread()
                    + "\t" + result
                    + "\t" + atomicStampedReference.getReference());
        }, "t2").start();
    }
}

// 控制台输出
Thread[t1,5,main]	第1次版本号: 1	100
Thread[t2,5,main]	第1次版本号: 1	100
Thread[t1,5,main]	第2次版本号: 2	101
Thread[t1,5,main]	第3次版本号: 3	100
Thread[t2,5,main]	false	100
```