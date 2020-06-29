[深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.htm)

```java
public class App {

    public static void main(String[] args) {
        String s = new String("hello ") + new String("world");
        System.out.println(System.identityHashCode(s));

        // 位置1
        System.out.println(System.identityHashCode(s));
        s.intern();
        System.out.println(System.identityHashCode(s));

        // 位置2
        String s2 = "hello world";
        System.out.println(System.identityHashCode(s2));

        System.out.println(s == s2);
    }
}

// 控制台输出
1870252780
1870252780
1870252780
1870252780
true
```

