### 建造者模式

1. 客户端**不需要知道对象内部组成细节, 将对象与对象的创建过程解耦**, 使得相同的创建过程可以产生不同的对象
2. 每一个具体的建造者相对独立, 与其他具体建造者无关, 因此可以很方便的替换具体建造者或者新增新的建造者, **客户端使用不同的具体建造者可以得到不同的对象**
3. **可以更加精细的控制对象的创建过程**, 将复杂的创建步骤分解到不同的方法中, 使得创建对象过程更加清晰, 也更加方便使用程序来控制创建过程
4. **新增新的具体建造者无需修改已存在的类库代码**, 更加方便扩展, 符合**开闭原则**
5. 建造者模式所创建的对象一般具有较多的共同点, 其组成部分相似, **如果对象之间的差异很大, 则不适合使用建造者模式**, 因此其使用范围受到一定的限制

### 简单的建造者模式

```java
public class App {

    public static void main(String[] args) {
        final Person foo = Person.builder().id(1).name("foo").age(20).build();
        System.out.println(foo);
    }
}

class Person {
    private Integer id;
    private String name;
    private Integer age;

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public Person(Builder builder) {
        this.id = builder.id;
        this.name = builder.name;
        this.age = builder.age;
    }

    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        private Integer id;
        private String name;
        private Integer age;


        public Builder id(Integer id) {
            this.id = id;
            return this;
        }

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder age(Integer age) {
            this.age = age;
            return this;
        }

        public Person build() {
            return new Person(this);
        }
    }
}
```

### 按照顺序建造者

```java
public class App {

    public static void main(String[] args) {
        // 存在调用顺序, 只能按照step1, step2, step3顺序执行
        final Character character = Character.builder()
                .withStep1("step1")
                .withStep2("step2")
                .withStep3("step3")
                .noMoreStep()
                .build();
        System.out.println(character);
    }
}

class Character {
    private String step1;
    private String step2;
    private String step3;

    public static StepBuilder builder() {
        return new StepBuilder();
    }

    public void setStep1(String step1) {
        this.step1 = step1;
    }

    public void setStep2(String step2) {
        this.step2 = step2;
    }

    public void setStep3(String step3) {
        this.step3 = step3;
    }

    @Override
    public String toString() {
        return "Character{" +
                "step1='" + step1 + '\'' +
                ", step2='" + step2 + '\'' +
                ", step3='" + step3 + '\'' +
                '}';
    }
}

interface Step1 {
    Step2 withStep1(String step1);
}

interface Step2 {
    Step3 withStep2(String step2);
}

interface Step3 {
    Step3 withStep3(String step3);

    BuildStep noMoreStep();
}

interface BuildStep {
    Character build();
}

class StepBuilder implements Step1, Step2, Step3, BuildStep {
    private String step1;
    private String step2;
    private String step3;

    @Override
    public Step2 withStep1(String step1) {
        this.step1 = step1;
        return this;
    }

    @Override
    public Step3 withStep2(String step2) {
        this.step2 = step2;
        return this;
    }

    @Override
    public Step3 withStep3(String step3) {
        this.step3 = step3;
        return this;
    }

    @Override
    public BuildStep noMoreStep() {
        return this;
    }

    @Override
    public Character build() {
        final Character character = new Character();
        character.setStep1(step1);
        character.setStep2(step2);
        character.setStep3(step3);
        return character;
    }
}
```

