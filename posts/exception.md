### java.util.ConcurrentModificationException

```java
public class App {

    public static void main(String[] args) {
        List<String> list = new ArrayList<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString());
                System.out.println(list);
            }).start();
        }
    }
}

// 控制台输出
Exception in thread "Thread-15" java.util.ConcurrentModificationException
    at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
    at java.util.ArrayList$Itr.next(ArrayList.java:859)
    at java.util.AbstractCollection.toString(AbstractCollection.java:461)
    at java.lang.String.valueOf(String.java:2994)
    at java.io.PrintStream.println(PrintStream.java:821)
    at com.serical.learn.gof.App.lambda$main$0(App.java:15)
    at java.lang.Thread.run(Thread.java:748)
```



### java.lang.StackOverflowError

```java
public class App2 {

    public static void m1() {
        m1();
    }

    public static void main(String[] args) {
        m1();
    }
}

// 控制台输出
Exception in thread "main" java.lang.StackOverflowError
	at com.serical.learn.gof.App2.m1(App2.java:6)
```



### java.lang.OutOfMemoryError: Java heap space

```java
public class App2 {

    public static void main(String[] args) {
        byte[] bytes = new byte[20 * 1024 * 1024];
    }
}

// JVM参数 -Xms10M -Xmx10M -XX:+PrintGCDetails
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.serical.learn.gof.App2.main(App2.java:6)
```

