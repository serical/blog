### 工厂模式的意义

将实例化对象的代码提取出来, 放到一个类中统一维护和管理, 降低类与类之间的耦合关系, 提供整个项目扩展性与维护性

### 1、简单工厂模式

```java
public class App {

    public static void main(String[] args) {
        System.out.println(SimpleFactory.create(WeaponType.AXE));
        System.out.println(SimpleFactory.create(WeaponType.BOW));
    }
}

enum WeaponType {AXE, BOW}

interface Weapon {
}

class Axe implements Weapon {
}

class Bow implements Weapon {
}

class SimpleFactory {

    private static final Map<WeaponType, Weapon> map = new HashMap<>();

    static {
        map.put(WeaponType.AXE, new Axe());
        map.put(WeaponType.BOW, new Bow());
    }

    public static Weapon create(WeaponType type) {
        return map.get(type);
    }
}
```

[版本二](https://github.com/serical/java-design-patterns/blob/master/factory-kit/src/main/java/com/iluwatar/factorykit/App.java)

```java
public class App {

    public static void main(String[] args) {
        final SimpleFactory factory = SimpleFactory.factory(builder -> {
            builder.add(WeaponType.AXE, Axe::new);
            builder.add(WeaponType.BOW, Bow::new);
        });
        System.out.println(factory.create(WeaponType.AXE));
        System.out.println(factory.create(WeaponType.BOW));
    }
}

enum WeaponType {AXE, BOW}

interface Weapon {
}

class Axe implements Weapon {
}

class Bow implements Weapon {
}

interface WeaponBuilder {
    void add(WeaponType type, Supplier<Weapon> supplier);
}

interface SimpleFactory {
    Weapon create(WeaponType type);

    static SimpleFactory factory(Consumer<WeaponBuilder> consumer) {
        Map<WeaponType, Supplier<Weapon>> map = new HashMap<>();
        consumer.accept(map::put);

        return type -> map.get(type).get();
    }
}
```



### 2、工厂方法模式

```java
public class App {

    public static void main(String[] args) {
        Blacksmith blacksmith = new ElfBlacksmith();
        System.out.println(blacksmith.createWeapon(WeaponType.AXE));
        System.out.println(blacksmith.createWeapon(WeaponType.BOW));

        blacksmith = new OrcBlacksmith();
        System.out.println(blacksmith.createWeapon(WeaponType.AXE));
        System.out.println(blacksmith.createWeapon(WeaponType.BOW));
    }
}

enum WeaponType {AXE, BOW}

interface Weapon {
}

class ElfWeapon implements Weapon {
    private WeaponType weaponType;

    public ElfWeapon(WeaponType weaponType) {
        this.weaponType = weaponType;
    }

    @Override
    public String toString() {
        return "Elf " + weaponType;
    }
}

class OrcWeapon implements Weapon {
    private WeaponType weaponType;

    public OrcWeapon(WeaponType weaponType) {
        this.weaponType = weaponType;
    }

    @Override
    public String toString() {
        return "Orc " + weaponType;
    }
}

interface Blacksmith {
    Weapon createWeapon(WeaponType type);
}

class ElfBlacksmith implements Blacksmith {

    private static final Map<WeaponType, Weapon> map;

    static {
        map = new HashMap<>(WeaponType.values().length);
        Arrays.stream(WeaponType.values()).forEach(type -> map.put(type, new ElfWeapon(type)));
    }

    @Override
    public Weapon createWeapon(WeaponType type) {
        return map.get(type);
    }
}

class OrcBlacksmith implements Blacksmith {

    public static final Map<WeaponType, Weapon> map;

    static {
        map = new HashMap<>(WeaponType.values().length);
        Arrays.stream(WeaponType.values()).forEach(type -> map.put(type, new OrcWeapon(type)));
    }

    @Override
    public Weapon createWeapon(WeaponType type) {
        return map.get(type);
    }
}
```



### 3、抽象工厂模式

```java
@Data
public class App {

    private King king;
    private Army army;
    private Castle castle;

    private static KingdomFactory makeFactory(KingdomType type) {
        switch (type) {
            case ELF:
                return new ElfKingdomFactory();
            case ORC:
                return new OrcKingdomFactory();
            default:
                throw new RuntimeException("type error");
        }
    }

    private void createKingdom(KingdomFactory factory) {
        this.setKing(factory.createKing());
        this.setArmy(factory.createArmy());
        this.setCastle(factory.createCastle());
    }

    public static void main(String[] args) {
        final App app = new App();
        app.createKingdom(makeFactory(KingdomType.ELF));
        System.out.println("ELF King");
        System.out.println(app.getKing().getDesc());
        System.out.println(app.getArmy().getDesc());
        System.out.println(app.getCastle().getDesc());

        app.createKingdom(makeFactory(KingdomType.ORC));
        System.out.println("ORC King");
        System.out.println(app.getKing().getDesc());
        System.out.println(app.getArmy().getDesc());
        System.out.println(app.getCastle().getDesc());
    }
}

enum KingdomType {ELF, ORC}


// interface King
interface King {
    String getDesc();
}

class ElfKing implements King {

    @Override
    public String getDesc() {
        return "this is elf king";
    }
}

class OrcKing implements King {

    @Override
    public String getDesc() {
        return "this is orc king";
    }
}

// interface Army
interface Army {
    String getDesc();
}

class ElfArmy implements Army {

    @Override
    public String getDesc() {
        return "this is elf army";
    }
}

class OrcArmy implements Army {

    @Override
    public String getDesc() {
        return "this is orc army";
    }
}

// interface Castle
interface Castle {
    String getDesc();
}

class ElfCastle implements Castle {

    @Override
    public String getDesc() {
        return "this is elf castle";
    }
}

class OrcCastle implements Castle {

    @Override
    public String getDesc() {
        return "this is orc castle";
    }
}

// interface KingdomFactory
interface KingdomFactory {
    King createKing();

    Army createArmy();

    Castle createCastle();
}

class ElfKingdomFactory implements KingdomFactory {

    @Override
    public King createKing() {
        return new ElfKing();
    }

    @Override
    public Army createArmy() {
        return new ElfArmy();
    }

    @Override
    public Castle createCastle() {
        return new ElfCastle();
    }
}

class OrcKingdomFactory implements KingdomFactory {

    @Override
    public King createKing() {
        return new OrcKing();
    }

    @Override
    public Army createArmy() {
        return new OrcArmy();
    }

    @Override
    public Castle createCastle() {
        return new OrcCastle();
    }
}
```

