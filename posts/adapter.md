### 类适配器模式

`Adapter`类继承`src`类, 实现`dst`接口, 完成`src -> dst`的适配

```java
public class App {

    public static void main(String[] args) {
        new Phone().charge(new VoltageAdapter());
    }
}

class Phone {
    public void charge(Voltage5 voltage5) {
        if (voltage5.output5V() == 5) {
            System.out.println("电压为5V, 能够充电");
        } else {
            System.out.println("电压错误, 不能充电");
        }
    }
}

class Voltage220 {
    public int output220V() {
        return 220;
    }
}

interface Voltage5 {
    int output5V();
}

class VoltageAdapter extends Voltage220 implements Voltage5 {

    @Override
    public int output5V() {
        final int output220V = output220V();
        // 转换过程
        return output220V / 44;
    }
}
```

### 对象适配器模式

`Adapter`不继承`src`类, 而是持有`src`类的对象实例, 以解决强耦合关系。即: 持有`src`类, 实现`dst`接口, 完成`src -> dst`的转换

```java
public class App {

    public static void main(String[] args) {
        final Voltage220 voltage220 = new Voltage220();
        final VoltageAdapter adapter = new VoltageAdapter(voltage220);
        new Phone().charge(adapter);
    }
}

class Phone {
    public void charge(Voltage5 voltage5) {
        if (voltage5.output5V() == 5) {
            System.out.println("电压为5V, 能够充电");
        } else {
            System.out.println("电压错误, 不能充电");
        }
    }
}

class Voltage220 {
    public int output220V() {
        return 220;
    }
}

interface Voltage5 {
    int output5V();
}

class VoltageAdapter implements Voltage5 {

    private Voltage220 voltage220;

    public VoltageAdapter(Voltage220 voltage220) {
        this.voltage220 = voltage220;
    }

    @Override
    public int output5V() {
        final int output220V = voltage220.output220V();
        // 转换过程
        return output220V / 44;
    }
}
```

### 接口适配器模式

提供接口空实现, 以解决不想实现全部方法的情景

```java
public class App {

    public static void main(String[] args) {
        new InterfaceAdapter() {
            @Override
            public void do1() {
                System.out.println("do1");
            }
        }.do1();
    }
}

interface Interface1 {
    void do1();

    void do2();

    void do3();

    void do4();

    void do5();
}

abstract class InterfaceAdapter implements Interface1 {
    @Override
    public void do1() {

    }

    @Override
    public void do2() {

    }

    @Override
    public void do3() {

    }

    @Override
    public void do4() {

    }

    @Override
    public void do5() {

    }
}
```

