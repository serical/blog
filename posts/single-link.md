```java
public class App2 {

    public static void main(String[] args) {
        final SingleLinkList list = new SingleLinkList();

//        list.add(new HeroNode(1, "zs"));
//        list.add(new HeroNode(2, "ls"));
//        list.add(new HeroNode(3, "ww"));

        list.addSort(new HeroNode(3, "ww"));
        list.addSort(new HeroNode(2, "ls"));
        list.addSort(new HeroNode(1, "zs"));
        list.addSort(new HeroNode(1, "zs"));

        list.list();
        System.out.println("操作之后");
        list.update(new HeroNode(1, "张三"));
        list.list();
//        System.out.println(list.length());
//        list.delete(1);
//        System.out.println(list.length());
//        list.delete(2);
//        System.out.println(list.length());
//        list.delete(3);
//        System.out.println(list.length());

//        int k = 10;
//        final HeroNode res = list.getLastIndex(k);
//        System.out.println("倒数第\t" + k + "\t 为 \t" + res);

//        list.reverse();
//        System.out.println("反转之后");
//        list.list();

        System.out.println("逆序打印");
        list.reversePrint();
    }
}

class SingleLinkList {
    private HeroNode head = new HeroNode(0, "");

    public void add(HeroNode node) {
        for (HeroNode temp = head; ; temp = temp.next) {
            if (null == temp.next) {
                temp.next = node;
                break;
            }
        }
    }

    /**
     * 顺序插入
     *
     * @param node 节点
     */
    public void addSort(HeroNode node) {
        for (HeroNode tmp = head; null != tmp; tmp = tmp.next) {
            if (null == tmp.next) {
                tmp.next = node;
                break;
            }
            if (tmp.next.id == node.id) {
                System.out.println("已经存在ID为\t" + node.id + "\t的元素了");
                break;
            }
            if (tmp.next.id > node.id) {
                node.next = tmp.next;
                tmp.next = node;
                break;
            }
        }
    }

    public void update(HeroNode node) {
        for (HeroNode tmp = head; null != tmp; tmp = tmp.next) {
            if (tmp.id == node.id) {
                tmp.name = node.name;
            }
        }
    }

    public void delete(Integer id) {
        for (HeroNode tmp = head; null != tmp && null != tmp.next; tmp = tmp.next) {
            if (tmp.next.id == id) {
                tmp.next = tmp.next.next;
            }
        }
    }

    public int length() {
        int length = 0;
        for (HeroNode tmp = head; null != tmp && null != tmp.next; tmp = tmp.next) {
            length++;
        }
        return length;
    }

    /**
     * 返回倒数第K个
     *
     * @param k 倒数第K个
     */
    public HeroNode getLastIndex(int k) {
        final int length = length();
        if (k <= 0 || k > length) {
            return null;
        }
        HeroNode res = head.next;
        for (int i = 0; i < length - k; i++) {
            res = res.next;
        }
        return res;
    }

    /**
     * 反转链表
     */
    public void reverse() {
        if (length() == 0) {
            return;
        }
        final HeroNode newHead = new HeroNode(0, "");
        for (HeroNode next, current = this.head.next; null != current; current = next) {
            // 记录原始节点下一个节点
            next = current.next;

            // 插入到新链表的head后第一个位置
            current.next = newHead.next;
            newHead.next = current;
        }
        this.head = newHead;
    }

    /**
     * 逆序打印
     * 方式1: 先SingleLinkList#reverse()再打印, 但是这样改变了原始数据
     * 方式2: 先压入Stack中, 再打印
     */
    public void reversePrint() {
        if (length() == 0) {
            return;
        }
        final Stack<HeroNode> stack = new Stack<>();
        for (HeroNode next = head.next; null != next; next = next.next) {
            stack.push(next);
        }
        while (stack.size() > 0) {
            System.out.println(stack.pop());
        }
    }

    public void list() {
        for (HeroNode temp = head.next; null != temp; temp = temp.next) {
            System.out.println(temp);
        }
    }
}

class HeroNode {
    int id;
    String name;
    HeroNode next;

    public HeroNode(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "HeroNode{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

