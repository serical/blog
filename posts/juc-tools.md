### Callable FutureTask

```java
public class App2 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        final FutureTask<Integer> task = new FutureTask<>(() -> {
            System.out.println("start future task");
            TimeUnit.SECONDS.sleep(3);
            return 1024;
        });

        new Thread(task).start();
        new Thread(task).start();
				
      	// 阻塞
        System.out.println(task.get());
    }
}

// 控制台输出
start future task
1024
```

java.util.concurrent.FutureTask#run多次调用的情况下, 实际只会执行一次

```java
public void run() {
  	// 这个CAS保证一个futureTask只执行一次
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                        null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```



### CountDownLatch

```java
public class App2 {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(6);

        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t" + "完成");
                latch.countDown();
            }, i + "").start();
        }

        latch.await();
        System.out.println("main线程完成");
    }
}

// 控制台输出
0	完成
3	完成
2	完成
4	完成
1	完成
5	完成
main线程完成
```



### CyclicBarrier

```java
public class App2 {

    public static void main(String[] args) {
        final CyclicBarrier barrier = new CyclicBarrier(8, () -> System.out.println("准备完毕"));
        for (int i = 0; i < 8; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t" + "准备中");

                try {
                    barrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }

                System.out.println(Thread.currentThread().getName() + "\t" + " 出发");
            }).start();
        }
    }
}

// 控制台输出
Thread-0	准备中
Thread-2	准备中
Thread-1	准备中
Thread-4	准备中
Thread-3	准备中
Thread-5	准备中
Thread-6	准备中
Thread-7	准备中
准备完毕
Thread-7	 出发
Thread-0	 出发
Thread-1	 出发
Thread-2	 出发
Thread-5	 出发
Thread-3	 出发
Thread-4	 出发
Thread-6	 出发
```



### Semaphore

* acquire
  * 要么成功信号量-1
  * 要么一直等下去直到有线程释放信号量, 或者超时
* release
  * 信号量+1, 唤醒其它正在等待的线程
* 用途
  * 多个共享资源互斥使用 
  * 并发线程数量控制

```java
public class App2 {

    public static void main(String[] args) {
        final Semaphore semaphore = new Semaphore(3);

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "\t抢到了");
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                    System.out.println(Thread.currentThread().getName() + "\t释放了啊");
                }
            }, String.valueOf(i)).start();
        }
    }
}
```



### ReentrantReadWriteLock

* 只要有一个线程`写`, 其他的线程都不能执行`读`和`写`
* 可以多个线程`同时读`

```java
public class App2 {

    public static void main(String[] args) {
        final Data data = new Data();

        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                data.put(Thread.currentThread().getName(), Thread.currentThread().getName());
            }, String.valueOf(i)).start();
        }

        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                data.get(Thread.currentThread().getName());
            }, String.valueOf(i)).start();
        }
    }
}

class Data {
    private Map<String, String> cache = new HashMap<>();
    private ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void put(String key, String value) {
        try {
            readWriteLock.writeLock().lock();
            System.out.println(Thread.currentThread().getName() + "\t" + DateUtil.now() + "\t 准备写入");
            TimeUnit.SECONDS.sleep(1);
            cache.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\t" + DateUtil.now() + "\t 完成写入");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }

    }

    public void get(String key) {
        try {
            readWriteLock.readLock().lock();
            System.out.println(Thread.currentThread().getName() + "\t" + DateUtil.now() + "\t 准备读取");
            cache.get(key);
            System.out.println(Thread.currentThread().getName() + "\t" + DateUtil.now() + "\t 完成读取");
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}

// 控制台输出
1	2020-06-12 11:18:22	 准备写入
1	2020-06-12 11:18:23	 完成写入
0	2020-06-12 11:18:23	 准备写入
0	2020-06-12 11:18:24	 完成写入
2	2020-06-12 11:18:24	 准备写入
2	2020-06-12 11:18:25	 完成写入
3	2020-06-12 11:18:25	 准备写入
3	2020-06-12 11:18:26	 完成写入
4	2020-06-12 11:18:26	 准备写入
4	2020-06-12 11:18:27	 完成写入
0	2020-06-12 11:18:27	 准备读取
0	2020-06-12 11:18:27	 完成读取
1	2020-06-12 11:18:27	 准备读取
2	2020-06-12 11:18:27	 准备读取
3	2020-06-12 11:18:27	 准备读取
1	2020-06-12 11:18:27	 完成读取
3	2020-06-12 11:18:27	 完成读取
2	2020-06-12 11:18:27	 完成读取
4	2020-06-12 11:18:27	 准备读取
4	2020-06-12 11:18:27	 完成读取
```

