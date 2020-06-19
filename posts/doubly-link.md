```java
public class App2 {

    public static void main(String[] args) {
        final DoublyLinkList list = new DoublyLinkList();
//        list.add(new Node(1, "zs"));
//        list.add(new Node(2, "ls"));
//        list.add(new Node(2, "ls"));
//        list.add(new Node(3, "ww"));

        list.addSort(new Node(3, "ww"));
        list.addSort(new Node(2, "ls"));
        list.addSort(new Node(2, "ls"));
        list.addSort(new Node(1, "zs"));

        list.list();

        System.out.println("更新操作");
        list.update(new Node(2, "李四"));
        list.list();

        System.out.println("删除操作");
        list.delete(2);
        list.list();
    }
}

class DoublyLinkList {
    private Node head = new Node(0, "");

    /**
     * 添加节点到双向链表最后
     *
     * @param node 节点
     */
    public void add(Node node) {
        for (Node current = head; null != current; current = current.next) {
            if (null == current.next) {
                current.next = node;
                node.pre = current;
                break;
            }
        }
    }

    /**
     * 按ID顺序添加节点
     *
     * @param node 节点
     */
    public void addSort(Node node) {
        for (Node current = head; null != current; current = current.next) {
            if (null == current.next) {
                current.next = node;
                node.pre = current;
                break;
            }
            if (current.next.id == node.id) {
                System.out.println("已经存在ID为\t" + node.id + "\t的元素了");
                break;
            }
            if (current.next.id > node.id) {
                // [下一个节点]的pre指向[将要插入节点]
                current.next.pre = node;
                // [将要插入节点]的next指向[下一个节点]
                node.next = current.next;
                // [将要插入节点]的pre指向[当前节点]
                node.pre = current;
                // [当前节点]的next指向[将要插入节点]
                current.next = node;
                break;
            }
        }
    }

    /**
     * 删除一个节点
     *
     * @param id 节点ID
     */
    public void delete(int id) {
        for (Node tmp, current = head.next; null != current; current = tmp) {
            tmp = current.next;
            if (current.id == id) {
                current.pre.next = current.next;
                if (null != current.next) {
                    current.next.pre = current.pre;
                }

                // 删除当前节点指针
                current.pre = null;
                current.next = null;
            }
        }
    }

    /**
     * 更新节点
     *
     * @param node 节点
     */
    public void update(Node node) {
        for (Node current = head.next; null != current; current = current.next) {
            if (current.id == node.id) {
                current.text = node.text;
            }
        }
    }

    public void list() {
        for (Node current = head.next; null != current; current = current.next) {
            System.out.println(current);
        }
    }
}

class Node {
    public int id;
    public String text;
    public Node pre;
    public Node next;

    public Node(int id, String text) {
        this.id = id;
        this.text = text;
    }

    @Override
    public String toString() {
        return "Node{" +
                "id=" + id +
                ", text='" + text + '\'' +
                '}';
    }
}
```

