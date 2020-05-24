```java
public class Test {

    private final static Object lock1 = new Object();
    private final static Object lock2 = new Object();

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (lock1) {
                try {
                    // 延迟下容易出现死锁
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (lock2) {
                    System.out.println("thread1");
                }

            }
        }).start();
        new Thread(() -> {
            synchronized (lock2) {
                synchronized (lock1) {
                    System.out.println("thread2");
                }
            }
        }).start();
    }
}
```

### jstack查询死锁

```bash
➜  ~ jps -l
33104 jdk.jcmd/sun.tools.jps.Jps
22724
33053 org.jetbrains.jps.cmdline.Launcher
23390
33054 com.serical.one.Test
➜  ~ jstack -l 33054
2020-05-24 13:06:15
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.241-b07 mixed mode):

"Attach Listener" #16 daemon prio=9 os_prio=31 tid=0x00007fb8c0008800 nid=0x3707 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"DestroyJavaVM" #15 prio=5 os_prio=31 tid=0x00007fb8b78fe800 nid=0xe03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Thread-1" #14 prio=5 os_prio=31 tid=0x00007fb8b6029800 nid=0xa203 waiting for monitor entry [0x00007000105df000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.serical.one.TestSingleton.lambda$main$1(TestSingleton.java:27)
	- waiting to lock <0x0000000715dc8210> (a java.lang.Object)
	- locked <0x0000000715dc8220> (a java.lang.Object)
	at com.serical.one.TestSingleton$$Lambda$2/1734853116.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"Thread-0" #13 prio=5 os_prio=31 tid=0x00007fb8b6028000 nid=0x5903 waiting for monitor entry [0x00007000104dc000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.serical.one.TestSingleton.lambda$main$0(TestSingleton.java:19)
	- waiting to lock <0x0000000715dc8220> (a java.lang.Object)
	- locked <0x0000000715dc8210> (a java.lang.Object)
	at com.serical.one.TestSingleton$$Lambda$1/1165897474.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None

"Service Thread" #12 daemon prio=9 os_prio=31 tid=0x00007fb8b5828000 nid=0xa603 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C1 CompilerThread3" #11 daemon prio=9 os_prio=31 tid=0x00007fb886009000 nid=0xa803 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread2" #10 daemon prio=9 os_prio=31 tid=0x00007fb886008800 nid=0x5603 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread1" #9 daemon prio=9 os_prio=31 tid=0x00007fb8b5825800 nid=0x5503 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread0" #8 daemon prio=9 os_prio=31 tid=0x00007fb8b5824800 nid=0x4303 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"JDWP Command Reader" #7 daemon prio=10 os_prio=31 tid=0x00007fb8b680a000 nid=0x4403 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"JDWP Event Helper Thread" #6 daemon prio=10 os_prio=31 tid=0x00007fb8c0061800 nid=0x4003 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"JDWP Transport Listener: dt_socket" #5 daemon prio=10 os_prio=31 tid=0x00007fb8b780e800 nid=0x4607 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Signal Dispatcher" #4 daemon prio=9 os_prio=31 tid=0x00007fb8b5808800 nid=0x4703 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Finalizer" #3 daemon prio=8 os_prio=31 tid=0x00007fb8b601d000 nid=0x4a03 in Object.wait() [0x000070000f8b5000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x0000000715588ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
	- locked <0x0000000715588ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

   Locked ownable synchronizers:
	- None

"Reference Handler" #2 daemon prio=10 os_prio=31 tid=0x00007fb8c005b800 nid=0x4b03 in Object.wait() [0x000070000f7b2000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x0000000715586c00> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:502)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
	- locked <0x0000000715586c00> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

   Locked ownable synchronizers:
	- None

"VM Thread" os_prio=31 tid=0x00007fb8c0056800 nid=0x4d03 runnable

"GC task thread#0 (ParallelGC)" os_prio=31 tid=0x00007fb8c000a800 nid=0x2207 runnable

"GC task thread#1 (ParallelGC)" os_prio=31 tid=0x00007fb8c0016000 nid=0x2103 runnable

"GC task thread#2 (ParallelGC)" os_prio=31 tid=0x00007fb8b6008800 nid=0x1e03 runnable

"GC task thread#3 (ParallelGC)" os_prio=31 tid=0x00007fb8b6017000 nid=0x2a03 runnable

"GC task thread#4 (ParallelGC)" os_prio=31 tid=0x00007fb8c0016800 nid=0x5303 runnable

"GC task thread#5 (ParallelGC)" os_prio=31 tid=0x00007fb8c0017000 nid=0x2b03 runnable

"GC task thread#6 (ParallelGC)" os_prio=31 tid=0x00007fb8c0017800 nid=0x5103 runnable

"GC task thread#7 (ParallelGC)" os_prio=31 tid=0x00007fb8c0018800 nid=0x5003 runnable

"GC task thread#8 (ParallelGC)" os_prio=31 tid=0x00007fb8b6018000 nid=0x2e03 runnable

"GC task thread#9 (ParallelGC)" os_prio=31 tid=0x00007fb8b6018800 nid=0x3003 runnable

"VM Periodic Task Thread" os_prio=31 tid=0x00007fb8b5810800 nid=0xa403 waiting on condition

JNI global references: 2298


# 发现死锁位置
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007fb8c005d408 (object 0x0000000715dc8210, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007fb8c00614a8 (object 0x0000000715dc8220, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
	at com.serical.one.TestSingleton.lambda$main$1(TestSingleton.java:27)
	- waiting to lock <0x0000000715dc8210> (a java.lang.Object)
	- locked <0x0000000715dc8220> (a java.lang.Object)
	at com.serical.one.TestSingleton$$Lambda$2/1734853116.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
"Thread-0":
	at com.serical.one.TestSingleton.lambda$main$0(TestSingleton.java:19)
	- waiting to lock <0x0000000715dc8220> (a java.lang.Object)
	- locked <0x0000000715dc8210> (a java.lang.Object)
	at com.serical.one.TestSingleton$$Lambda$1/1165897474.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

