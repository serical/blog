### 两个线程干活, 一个`加1`, 一个`减1`

```java
public class App2 {

    public static void main(String[] args) {
        final Data data = new Data();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

class Data {
    private Integer count = 0;

    public synchronized void increment() throws InterruptedException {
        if (count != 0) {
            this.wait();
        }
        count++;
        System.out.println(Thread.currentThread().getName() + "\t 生产了一个\t" + count);
        this.notifyAll();
    }

    public synchronized void decrement() throws InterruptedException {
        if (count == 0) {
            this.wait();
        }
        count--;
        System.out.println(Thread.currentThread().getName() + "\t 消费了一个\t" + count);
        this.notifyAll();
    }
}

// 控制台输出
A	 生产了一个	1
B	 消费了一个	0
A	 生产了一个	1
B	 消费了一个	0
A	 生产了一个	1
B	 消费了一个	0
A	 生产了一个	1
B	 消费了一个	0
A	 生产了一个	1
B	 消费了一个	0
A	 生产了一个	1
B	 消费了一个	0
A	 生产了一个	1
B	 消费了一个	0
A	 生产了一个	1
B	 消费了一个	0
A	 生产了一个	1
B	 消费了一个	0
A	 生产了一个	1
B	 消费了一个	0
```



### 四个线程干活, 2个`加1`, 2个`减1`(错误)

```java
public class App2 {

    public static void main(String[] args) {
        final Data data = new Data();

        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                for (int j = 0; j < 10; j++) {
                    try {
                        data.increment();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, "Producer" + i).start();
        }

        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                for (int j = 0; j < 10; j++) {
                    try {
                        data.decrement();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, "Consumer" + i).start();
        }
    }
}

class Data {
    private Integer count = 0;

    public synchronized void increment() throws InterruptedException {
        if (count != 0) {
            this.wait();
        }
        count++;
        System.out.println(Thread.currentThread().getName() + "\t 生产了一个\t" + count);
        this.notifyAll();
    }

    public synchronized void decrement() throws InterruptedException {
        if (count == 0) {
            this.wait();
        }
        count--;
        System.out.println(Thread.currentThread().getName() + "\t 消费了一个\t" + count);
        this.notifyAll();
    }
}

// 控制台输出
Producer0	 生产了一个	1
Consumer0	 消费了一个	0
Producer0	 生产了一个	1
Producer1	 生产了一个	2
Consumer1	 消费了一个	1
Consumer1	 消费了一个	0
Consumer0	 消费了一个	-1
Consumer0	 消费了一个	-2
Consumer0	 消费了一个	-3
Consumer0	 消费了一个	-4
Consumer0	 消费了一个	-5
Consumer0	 消费了一个	-6
Consumer0	 消费了一个	-7
Consumer0	 消费了一个	-8
Consumer0	 消费了一个	-9
Producer1	 生产了一个	-8
Consumer1	 消费了一个	-9
Consumer1	 消费了一个	-10
Consumer1	 消费了一个	-11
Consumer1	 消费了一个	-12
Consumer1	 消费了一个	-13
Consumer1	 消费了一个	-14
Consumer1	 消费了一个	-15
Consumer1	 消费了一个	-16
Producer0	 生产了一个	-15
Producer1	 生产了一个	-14
Producer0	 生产了一个	-13
Producer1	 生产了一个	-12
Producer0	 生产了一个	-11
Producer1	 生产了一个	-10
Producer0	 生产了一个	-9
Producer1	 生产了一个	-8
Producer0	 生产了一个	-7
Producer1	 生产了一个	-6
Producer0	 生产了一个	-5
Producer1	 生产了一个	-4
Producer0	 生产了一个	-3
Producer1	 生产了一个	-2
Producer0	 生产了一个	-1
Producer1	 生产了一个	0
```

**多线程环境不能使用`if`判断, 要使用`while`判断条件**

```java
public synchronized void increment() throws InterruptedException {
  	// 注意while
    while (count != 0) {
        this.wait();
    }
    count++;
    System.out.println(Thread.currentThread().getName() + "\t 生产了一个\t" + count);
    this.notifyAll();
}

public synchronized void decrement() throws InterruptedException {
  	// 注意while
    while (count == 0) {
        this.wait();
    }
    count--;
    System.out.println(Thread.currentThread().getName() + "\t 消费了一个\t" + count);
    this.notifyAll();
}
```



### ReentrantLock Condition 实现

```java
public class App2 {

    public static void main(String[] args) {
        final Data data = new Data();

        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                for (int j = 0; j < 10; j++) {
                    try {
                        data.increment();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, "Producer" + i).start();
        }

        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                for (int j = 0; j < 10; j++) {
                    try {
                        data.decrement();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, "Consumer" + i).start();
        }
    }
}

class Data {
    private Integer count = 0;
    private Lock lock = new ReentrantLock();
    private Condition producer = lock.newCondition();
    private Condition consumer = lock.newCondition();

    public void increment() throws InterruptedException {
        try {
            lock.lock();
            while (count != 0) {
                producer.await();
            }
            count++;
            System.out.println(Thread.currentThread().getName() + "\t 生产了一个\t" + count);

            consumer.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void decrement() throws InterruptedException {
        try {
            lock.lock();
            while (count == 0) {
                consumer.await();
            }
            count--;
            System.out.println(Thread.currentThread().getName() + "\t 消费了一个\t" + count);

            producer.signalAll();
        } finally {
            lock.unlock();
        }
    }
}

// 控制台
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer0	 生产了一个	1
Consumer1	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
Producer1	 生产了一个	1
Consumer0	 消费了一个	0
```



### ReentrantLock Condition精准控制线程执行顺序

3个线程A、B、C实现

* `A->B->C`执行顺序
* A打印5次, B打印10次, C打印15次
* 按照上面的要循环10轮

```java
public class App2 {

    public static void main(String[] args) {
        final Data data = new Data();

        final ReentrantLock lock = new ReentrantLock();
        final Condition a = lock.newCondition();
        final Condition b = lock.newCondition();
        final Condition c = lock.newCondition();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.print(lock, a, b, 1, 2, 5);
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.print(lock, b, c, 2, 3, 10);
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.print(lock, c, a, 3, 1, 15);
            }
        }, "C").start();
    }
}

class Data {
    int status = 1;

    public void print(Lock lock, Condition current, Condition next, int targetStatus, int nextStatus, int count) {
        try {
            lock.lock();
            while (status != targetStatus) {
                current.await();
            }

            for (int i = 0; i < count; i++) {
                System.out.println(Thread.currentThread().getName() + "\t" + i);
            }

            status = nextStatus;
            next.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```



### 数组当做容器

```java
public class Test {

    public static void main(String[] args) {
        final Container container = new Container();
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                for (; ; ) {
                    try {
                        container.add(new Object());
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, "producer" + i).start();
        }

        for (int i = 0; i < 8; i++) {
            new Thread(() -> {
                for (; ; ) {
                    try {
                        container.get();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, "consumer" + i).start();
        }
    }
}

class Container {
    Object[] container = new Object[100];
    int index;

    public synchronized void add(Object o) throws InterruptedException {
        while (index == container.length) {
            this.wait();
        }

        container[index++] = o;
        System.out.println(Thread.currentThread() + "生产者生产一个对象, 当前容量为: " + length());
        TimeUnit.MILLISECONDS.sleep(100);

        this.notifyAll();
    }

    public synchronized void get() throws InterruptedException {
        while (index == 0) {
            this.wait();
        }

        final Object o = container[--index];
        System.out.println(Thread.currentThread() + "消费者完成一次消费, 当前容量为: " + length() + " 对象为: " + o);
        container[index] = null;
        TimeUnit.MILLISECONDS.sleep(100);

        this.notifyAll();
    }

    public int length() {
        return index;
    }
}
```

