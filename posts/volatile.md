### 验证volatile保证可见性

```java
public class App {

    public static void main(String[] args) {
        final Data data = new Data();

        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            data.setNumber();
        }).start();

        while (data.number == 0) {

        }

        System.out.println(Thread.currentThread() + "\t" + data.number);
    }
}

class Data {
    // 加上volatile后main线程会结束while循环
    // 去掉volatile后while进入死循环
    volatile int number = 0;

    public void setNumber() {
        System.out.println(Thread.currentThread());
        number = 1;
    }
}
```

### 验证volatile不保证原子性

```java
public class App {

    public static void main(String[] args) throws InterruptedException {
        final Data data = new Data();

        int size = 20;
        final CountDownLatch latch = new CountDownLatch(size);
        for (int i = 0; i < size; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    data.setNumber();
                }
                latch.countDown();
            }).start();
        }

        latch.await();

        // 结果小于20000, 就证明了volatile不保证原子性
        System.out.println(Thread.currentThread() + "\t" + data.number);
    }
}

class Data {
    volatile int number = 0;

    public void setNumber() {
        number++;
    }
}
```

**解决方法**

1. 使用`synchronized`关键字

```java
    public synchronized void setNumber() {
        number++;
    }
```

2. 使用`AtomicInteger`类

```java
class Data {
    AtomicInteger number = new AtomicInteger();

    public void setNumber() {
        number.incrementAndGet();
    }
}
```

### 指令重排举例

```java
public class App {

    public static void main(String[] args) {

    }
}

class Data {
    int i = 0;
    boolean flag = false;

    public void m1() {
        i = 1;          // 语句1
        flag = true;    // 语句2
    }

    // 如果m1与m2方法同时都有多个线程在执行
    // 没有禁止指令重排, 假如m1方法中的语句2先执行, m2方法再执行, 打印的值就是5
    public void m2() {
        if (flag) {
            i = i + 5;  // 语句3
            System.out.println("i: " + i);
        }
    }
}
```

### 单例模式中的volatile

**instance = new Singleton(); 可以分为下面三个步骤:** 

```bash
memory=allocate(); // 1.分配对象内存空间
instance(memory);  // 2.初始化对象
instance = memory; // 3.设置instance指向刚分配的内存地址, 此时instance != null
```

`步骤2`于`步骤3`不存在数据依赖关系, 而且无论`重排前`还是`重排后`程序的执行结果在单线程中没有区别,  因此这种重排是允许的。

但是在`多线程环境下`, 如果是重排后, `先执行步骤3`, 此时`instance != null`, 但是`对象还没有初始化完成`, 在别的线程使用时就会出问题。**这种指令重排是不希望的**, 使用`volatile`禁止指令重排

```java
// 成员变量必须声明为volatile
private static volatile Singleton singleton = null;
```

