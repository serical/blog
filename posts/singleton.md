### 测试方法

```java
public class TestSingleton {

    public static void main(String[] args) throws InterruptedException {
        Set<Singleton> set = new HashSet<>();

        int size = 100;
        final CountDownLatch latch = new CountDownLatch(size);
        for (int i = 0; i < size; i++) {
            new Thread(() -> {
                set.add(Singleton.getInstance());
                latch.countDown();
            }).start();
        }

        latch.await();
        System.out.println(set);
    }
}
```

### 1、饿汉式(静态常量)

* 优点: 写法简单, 在类装载的时候就完成了实例化, 避免县城安全问题
* 缺点: 不一定是`getInstance`方法触发类装载, 没有实现`lazy loading`效果, `可能`造成内存浪费

```java
class Singleton {
  
    private Singleton() {
    }

    private final static Singleton SINGLETON = new Singleton();

    public static Singleton getInstance() {
        return SINGLETON;
    }
}
```

### 2、饿汉式(静态块)

* 优缺点与第1种方式一样

```java
class Singleton {

    private Singleton() {
    }

    private final static Singleton SINGLETON;

    static {
        SINGLETON = new Singleton();
    }

    public static Singleton getInstance() {
        return SINGLETON;
    }
}
```

### 3、懒汉式(线程不安全)

优点: 实现了`lazy loading`效果, 但是只能在单线程环境使用

缺点: 多线程环境下会产生多个实例, 实际开发中不要使用这种方式

```java
class Singleton {

    private Singleton() {
    }

    private static Singleton SINGLETON;

    public static Singleton getInstance() {
        if (null == SINGLETON) {
            SINGLETON = new Singleton();
        }
        return SINGLETON;
    }
}

console out: 
[com.serical.learn.gof.singleton.Singleton@74e13f63, com.serical.learn.gof.singleton.Singleton@4c6aca09, com.serical.learn.gof.singleton.Singleton@73248cf9]
```

### 4、懒汉式(线程安全)

* 优点: 实现了`lazy loading`效果, 实现了线程安全效果
* 缺点: 但是效率太低了, 每次`getInstance`方法时都要执行同步方法, 实际开发中不推荐使用

```java
class Singleton {

    private Singleton() {
    }

    private static Singleton SINGLETON;

    public synchronized static Singleton getInstance() {
        if (null == SINGLETON) {
            SINGLETON = new Singleton();
        }
        return SINGLETON;
    }
}
```

### 5、懒汉式(Double Check)

既实现了`lazy loading`又实现了线程安全同时解决了效率问题, 实际开发中推荐使用

```java
class Singleton {

    private Singleton() {
    }

  	// volatile在对象更新后,刷新到主存中
    private static volatile Singleton SINGLETON;

    public static Singleton getInstance() {
        if (null == SINGLETON) {
          	// 保证多线程环境只有一个线程完成初始化
            synchronized (Singleton.class) {
                if (null == SINGLETON) {
                    SINGLETON = new Singleton();
                }
            }
        }
        return SINGLETON;
    }
}
```

### 6、静态内部类

1. `Singleton`装载时`SingletonInstance`并不会装载, 实现了`lazy loading`效果

2. 调用`getInstance`时使用`JVM`类装载机制(类初始化时只有一个线程), 实现线程安全, 推荐使用

```java
class Singleton {

    private Singleton() {
    }

    private static class SingletonInstance {
        private final static Singleton SINGLETON = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonInstance.SINGLETON;
    }
}
```

### 7、枚举方式

借助`jdk1.5`添加的枚举来实现单利模式, 不仅能避免多线程同步问题, 还能防止反序列化重新创建的对象, `Effective Java`推荐方式

```java
enum Singleton {

    INSTANCE;

    public void foo() {
        System.out.println("foo");
    }
}
```

### jdk中的应用

```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    /**
     * Returns the runtime object associated with the current Java application.
     * Most of the methods of class <code>Runtime</code> are instance
     * methods and must be invoked with respect to the current runtime object.
     *
     * @return  the <code>Runtime</code> object associated with the current
     *          Java application.
     */
    public static Runtime getRuntime() {
        return currentRuntime;
    }

    /** Don't let anyone else instantiate this class */
    private Runtime() {}
}
```

