### 原型模式简介

1. 创建新的对象比较复制时, 可以使用原型模式简化对象的创建过程, 同时也能提高效率
2. 不用重新初始化对象, 而是动态获取对象运行时的状态
3. 实现深度克隆时可能需要比较实现比较复杂的代码
4. 如果采用重写`clone`方式实现深度克隆, 对于全新的类来说没什么问题, 但是对于已有的类来说, 需要修改其源代码, 违背了OCP原则。



### 一、默认克隆

**引用类型**

```java
class DeepType implements Cloneable {
    private Integer id;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    @Override
    public String toString() {
        return "DeepType{" +
                "id=" + id +
                '}';
    }

    public DeepType(Integer id) {
        this.id = id;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }
}
```

**测试**

```java
public class App {

    public static void main(String[] args) throws CloneNotSupportedException {
        final Sheep sheep = new Sheep(1, "tom", new DeepType(1));
        final Sheep clone = (Sheep) sheep.clone();
        System.out.println(sheep == clone);
        System.out.println(sheep);
        System.out.println(clone);

        System.out.println(sheep.getDeepType() == clone.getDeepType());
        sheep.getDeepType().setId(2);
        System.out.println(sheep);
        System.out.println(clone);
    }
}

class Sheep implements Cloneable {
    private Integer id;
    private String name;
    private DeepType deepType;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    @Override
    public String toString() {
        return "Sheep{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", deepType=" + deepType +
                '}';
    }

    public Sheep(Integer id, String name, DeepType deepType) {
        this.id = id;
        this.name = name;
        this.deepType = deepType;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public DeepType getDeepType() {
        return deepType;
    }

    public void setDeepType(DeepType deepType) {
        this.deepType = deepType;
    }
}
```

**控制台输出**

```bash
false
Sheep{id=1, name='tom', deepType=DeepType{id=1}}
Sheep{id=1, name='tom', deepType=DeepType{id=1}}
true
Sheep{id=1, name='tom', deepType=DeepType{id=2}}
Sheep{id=1, name='tom', deepType=DeepType{id=2}}
```

* 对于成员变量为`基本数据类型`以及`Sting`是直接进行值传递, 也就是将该属性值复制一份给新的对象
* 对于成员变量为`引用类型`的, 如成员变量是`某个类的实例对象`或者`数组`, 进行的是引用传递, 也就是将该属性的引用值(`内存地址`)复制一份给新的对象。这种情况下, 只要一个对象修改该成员变量, 会影响到另一个对象成员变量的值。

### 二、深度克隆

#### 1、重写`Sheep`的`clone`方法

```java
    @Override
    protected Object clone() throws CloneNotSupportedException {
        // 复制基本数据类型
        final Sheep sheep = (Sheep) super.clone();
        // 复制引用数据
        final DeepType deepType = (DeepType) this.getDeepType().clone();
        sheep.setDeepType(deepType);
        return sheep;
    }
```

**控制台输出**

```bash
false
Sheep{id=1, name='tom', deepType=DeepType{id=1}}
Sheep{id=1, name='tom', deepType=DeepType{id=1}}
false
Sheep{id=1, name='tom', deepType=DeepType{id=2}}
Sheep{id=1, name='tom', deepType=DeepType{id=1}}
```

#### 2、在`Sheep`中增加一个`deepClone`方法

```java
    public Sheep deepClone() throws IOException, ClassNotFoundException {
        @Cleanup final ByteArrayOutputStream bos = new ByteArrayOutputStream();
        @Cleanup final ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(this);

        @Cleanup final ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        @Cleanup final ObjectInputStream ois = new ObjectInputStream(bis);
        return (Sheep) ois.readObject();
    }
```

**控制台输出**

```java
false
Sheep{id=1, name='tom', deepType=DeepType{id=1}}
Sheep{id=1, name='tom', deepType=DeepType{id=1}}
false
Sheep{id=1, name='tom', deepType=DeepType{id=2}}
Sheep{id=1, name='tom', deepType=DeepType{id=1}}
```

