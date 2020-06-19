### 使用单向环形链表解决

```java
public class App2 {

    public static void main(String[] args) {
        final CircleSingleLinkList list = new CircleSingleLinkList();
        int n = 10;
        list.init(n);
        list.show();
        System.out.println("出圈");
        list.remove(1, 2, n);
    }
}

class CircleSingleLinkList {
    private Node first;

    public void init(int n) {
        if (n < 1) {
            return;
        }
        Node current = null;
        for (int i = 1; i <= n; i++) {
            final Node node = new Node(i);
            if (i == 1) {
                first = node;
                first.next = first;
                current = first;
            } else {
                current.next = node;
                node.next = first;
                current = node;
            }
        }
    }

    public void show() {
        for (Node next = first; null != next; next = next.next) {
            System.out.println(next);
            if (next.next == first) {
                break;
            }
        }
    }

    /**
     * 从第k个位置开始, 每数m个移除一个
     *
     * @param k 从第几个开始
     * @param m 表示数几下
     * @param n 总个数
     */
    public void remove(int k, int m, int n) {
        if (null == first || k < 1 || k > n) {
            return;
        }

        // 初始化第一个与最后一个指针
        Node last = first, next = first;
        for (; last.next != first; ) {
            last = last.next;
        }

        // 处理k
        for (int i = 0; i < k - 1; i++) {
            next = next.next;
            last = last.next;
        }

        for (; last != next; ) {
            for (int i = 0; i < m - 1; i++) {
                last = last.next;
                next = next.next;
            }

            System.out.println(next);
            last.next = next.next;
            next = next.next;
        }
        System.out.println("最后一个为:" + last);
    }
}

class Node {
    public int id;
    public Node next;

    public Node(int id) {
        this.id = id;
    }

    @Override
    public String toString() {
        return "Node{" +
                "id=" + id +
                '}';
    }
}

// 控制台
Node{id=1}
Node{id=2}
Node{id=3}
Node{id=4}
Node{id=5}
Node{id=6}
Node{id=7}
Node{id=8}
Node{id=9}
Node{id=10}
出圈
Node{id=2}
Node{id=4}
Node{id=6}
Node{id=8}
Node{id=10}
Node{id=3}
Node{id=7}
Node{id=1}
Node{id=9}
最后一个为:Node{id=5}
```

