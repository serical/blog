## 设计模式7大设计原则

#### 一、单一职责原则

1. 降低类的复杂度, 一个类负责一项职责
2. 提高可读性, 可维护性
3. 降低变更引起的风险

可以是类级别的单一职责(如一种交通工具一个类), 也可以是方法级别的单一职责(如一个交通工具类包含具体交通工具的方法)

#### 二、接口隔离原则

`InterfaceImpl1`实现了`Interface`

`InterfaceImpl2`实现了`Interface`

```java
interface Interface{
    void method1();
    void method2();
    void method3();
    void method4();
    void method5();
}
```

1. 客户端不应该依赖它不需要的接口, 即一个类对另一个类的依赖应该建立在最小的接口上
2. 类`A`依赖`InterfaceImpl1`时只需要使用`method1` `method2`, 类`B`依赖`InterfaceImpl2`时只需要使用`method3` `method4`  `method5`, 那么`InterfaceImpl1` `InterfaceImpl2`必须实现他们不需要的方法
3. 这时就应该将`Interface`拆分成独立的2个接口, 分别声明`method1` `method2`与 `method3` `method4`  `method5`, 从而实现接口最小依赖

#### 三、倒转依赖原则

1. 底层模块尽量都要有抽象类或者接口, 或者两者都要有, 程序稳定性更好
2. 变量的声明尽量是抽象类或者接口, 这样我们的变量引用于实际对象之间, 就存在一个缓冲层, 利于程序的扩展
3. 继承时遵循里式替换原则

```java
class Email {}
class Person {
  	// 这里的Email应该声明为更为抽象的IReceiver接口, 就能满足Email, QQ, PHONE等等
    public void receive(Email email) {
    }
}
```

依赖的三种方式:

```java
interface CloseAndOpen {
  	// 通过参数接口传递依赖
    void open(TV tv);
}

interface TV {
    void play();
}

class CloseAndOpenImpl implements CloseAndOpen {
    @Override
    public void open(TV tv) {
        tv.play();
    }
}
```

```java
interface CloseAndOpen {
    void open(TV tv);
}

interface TV {
    void play();
}

class CloseAndOpenImpl implements CloseAndOpen {
    private TV tv;
		// 通过构造方法传递依赖
    CloseAndOpenImpl(TV tv) {
        this.tv = tv;
    }

    @Override
    public void open(TV tv) {
        tv.play();
    }
}
```

```java
interface CloseAndOpen {
    void open(TV tv);
}

interface TV {
    void play();
}

class CloseAndOpenImpl implements CloseAndOpen {
    private TV tv;
		// 通过setter方法传递依赖
    public void setTv(TV tv) {
        this.tv = tv;
    }

    @Override
    public void open(TV tv) {
        tv.play();
    }
}
```



#### 四、里式替换原则

1. 在使用继承时, 子类尽量不要重写父类的方法
2. 如果非要重写父类的某一个方法,
   1. 把这个方法抽象提升为抽象方法或者接口, 再由父类和子类分别实现
   2. 通过依赖、聚合、组合来解决问题
3. 里式替换原则告诉我们, 继承实际上是让两个类的耦合性增强了

#### 五、开闭原则

1. 开闭原则是编程中`最基础`、 `最重要`的设计原则
2. 一个软件实体如类、模块、函数应该对`扩展`开放(提供方), 对`修改`关闭(使用方)。用抽象构建框架, 用实现扩展细节
3. 当软件需要变化时, 应该尽量`通过扩展软件实体的行为`来实现变化, 而`不是通过修改已有的代码`来实现变化

#### 六、迪米特原则

1. 一个对象应该对其他对象保存最少的了解
2. 类与类之间的关系越密切, 耦合度越大
3. 迪米特法则, 又叫`最少知道原则`, 即一个类对于自己依赖的类知道的越少越好。也就是说, 对于被依赖的类不管多么的复杂, 都应该尽量把逻辑封装类的内部。 处理提供对外的`public`方法, 不对外泄露任何信息
4. 迪米特还有个更简单的定义: 只与`直接的朋友`通信
5. 直接的朋友: 每个对象都会与其他对象存在`耦合关系`, 只要两个对象之前存在`耦合关系`, 我就说这两个对象之间是`朋友关系`。耦合的方式有很多种, 依赖、关联、组合、聚合等
   1. 直接的朋友: 成员变量、方法参数、方法返回值
   2. 非直接朋友: 方法中的局部变量
6. 也就是说陌生的类最好不要以局部变量的形式出现在类的内部



#### 七、合成复用原则

1. 尽量使用合成/聚合的方式, 而不是使用继承
2. 例如: `B`要使用`A`中的方法

```java
class A {
    void method1() {
    }

    void method2() {
    }
}
```

```java
// 通过继承(反例)
class B extends A {
}
```

```java
class B {
  	// 通过依赖
    public void xxx(A a) {
    }
}
```

```java
class B {
    private A a;
		// 通过聚合
    public void setA(A a) {
        this.a = a;
    }
}
```

```java
class B {
  	// 通过组合
    private A a = new A();
}
```

