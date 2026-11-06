---
type: memo
date: 2026-11-06
aliases:
  - "Java 多态、抽象类、final、单例与枚举精讲"
---

# Java 多态、抽象类、final、单例与枚举精讲

## 一、概念说明

### 1. 多态

多态是在继承/实现情况下的一种现象，表现为**对象多态、行为多态**。代码体现：

```java
People p1 = new Student();
People p2 = new Teacher();
p1.run();
p2.run();
```

多态的前提：有继承/实现关系；存在父类引用指向子类对象；存在方法重写。

注意：**多态是对象、行为的多态，Java 中的属性（成员变量）不谈多态**——访问成员变量时看的是编译期类型（父类），没有"运行时绑定"这回事。

### 2. 多态的好处与问题

好处：

- 多态形式下，右边对象是解耦合的，便于扩展和维护（换右边 new 的子类即可，左边类型和调用代码不动）；
- 定义方法时使用父类类型作形参，可以接收一切子类对象，扩展性强。

问题：多态下**不能调用子类的独有功能**——编译看左边，父类没有的方法编译就报错。

### 3. 多态下的类型转换

- 自动类型转换（向上转型）：`父类 变量名 = new 子类();`
- 强制类型转换（向下转型）：`子类 变量名 = (子类) 父类变量;`，用于恢复子类独有功能的调用。

强转的注意：存在继承/实现关系编译期就不报错；但运行时若对象的真实类型与强转类型不符，抛 `ClassCastException`：

```java
People p = new Teacher();
Student s = (Student) p;   // 编译通过，运行时报 ClassCastException
```

强转前建议用 `instanceof` 判断真实类型：

```java
if (p instanceof Student) {
    Student s = (Student) p;
}
```

### 4. 抽象类

`abstract` 可以修饰类和方法：

```java
public abstract class A {
    // 抽象方法：必须 abstract 修饰，只有方法签名，不能有方法体
    public abstract void test();
}
```

抽象类的特点：

1. 抽象类中不一定要有抽象方法，但**有抽象方法的类必须是抽象类**；
2. 类有的成员（成员变量、方法、构造器）抽象类都可以有；
3. 抽象类**不能创建对象**，仅作为一种特殊的父类，让子类继承并实现；
4. 一个类继承抽象类，必须重写完抽象类的全部抽象方法，否则这个类自己也必须定义成抽象类。

使用抽象类的好处：父类知道每个子类都要做某个行为，但每个子类的做法不一样，父类就把该行为定义成抽象方法交给子类重写——**抽象类是为了更好地支持多态**。

### 5. final 关键字

final 意为"最终"，可以修饰类、方法、变量：

- 修饰类：该类为最终类，不能被继承；
- 修饰方法：该方法为最终方法，不能被重写；
- 修饰变量：该变量有且仅能被赋值一次。

final 修饰变量的细化规则：修饰基本类型变量，存储的数据不能改变；修饰引用类型变量，存储的**地址**不能改变，但地址指向的对象内容可以改变。

### 6. 常量

`static final` 修饰的成员变量称为常量，常用于记录系统的配置信息：

```java
public class Constant {
    public static final String SCHOOL_NAME = "示例学校";
}
```

优势：代码可读性、可维护性更好；执行原理上，编译后常量会被"宏替换"——所有用到常量的地方直接替换成字面量，性能与直接写字面量一样。命名规范：大写英文单词，多词用下划线连接。

### 7. 单例设计模式

设计模式：一个问题通常有 n 种解法，被总结出来的最优解法称为设计模式。单例模式确保**某个类只能创建一个对象**。

饿汉式（拿对象时，对象早就创建好了）：

```java
public class A {
    // 2、定义一个静态变量记住类的一个对象
    private static A a = new A();

    // 1、私有构造器
    private A() {
    }

    // 3、定义一个静态方法返回对象
    public static A getObject(){
        return a;
    }
}
```

懒汉式（拿对象时才开始创建，延迟加载）：

```java
public class B {
    // 2、定义一个类变量用于存储对象
    private static B b;   // null

    // 1、单例必须私有构造器
    private B() {
    }

    // 3、提供一个类方法，保证返回同一个对象
    public static B getObject(){
        if (b == null) {
            b = new B();
        }
        return b;
    }
}
```

应用场景：任务管理器、运行时对象等只需要一个实例的场景，避免浪费内存。

### 8. 枚举类

枚举是一种特殊类：

```java
public enum A {
    X, Y, Z;
}
```

特点：

- 第一行只能写枚举对象名称，逗号隔开；这些名称本质是常量，每个常量记住枚举类的一个对象；
- 枚举都是最终类（不可被继承），都继承 `java.lang.Enum`；
- 构造器都是私有的（写不写都只能是私有），对外不能创建对象；
- 编译器会为枚举类新增 `values()`、`valueOf(String)` 等方法。

编译后等价于：

```java
public final class A extends java.lang.Enum<A> {
    public static final A X = new A();
    public static final A Y = new A();
    public static final A Z = new A();

    public static A[] values();
    public static A valueOf(java.lang.String);
}
```

应用场景：做信息分类和标志，比传统常量更安全，不会出现非法选项。例如方向控制：

```java
public enum Direction {
    UP, DOWN, LEFT, RIGHT
}

public void move(Direction direction){
    // 根据用户要求的方向控制图块移动
}
```

## 二、技术点之间的联系

- 继承是多态的前提，方法重写是多态的开关：没有重写，父类引用调用的还是父类行为，多态失去意义；抽象类/接口则是把"必须重写"用语法强制下来，是多态的强化形态。
- 多态的"编译看左边、运行看右边"与类型转换互为补充：向上转型获得解耦，向下转型（配 instanceof）恢复子类特性。
- final 是"继承与重写的刹车"：final 类不能被继承、final 方法不能被重写、final 变量不能被改——它与继承/多态构成一张一弛的关系。
- 单例模式是 static（静态变量持唯一对象）与封装（私有构造器）的组合应用；枚举则是语法层面的"限定实例集合"，可视为单例思想的推广（多例且封闭）。
- 常量（static final）替代散落在代码各处的魔法值，与枚举替代"伪选项常量"是同一治理思路的两个粒度。

## 三、方法与实现要点

### 1. 多态的标准形态

```java
// 父类引用指向子类对象，调用被重写的方法
People p1 = new Student();
p1.run();   // 编译看 People（要有 run 方法），运行执行 Student 重写的 run
```

定义通用方法时用父类作形参：

```java
public static void go(People p) {
    p.run();   // 可以接收 Student、Teacher 等一切子类对象
}
```

### 2. 抽象类的标准用法

```java
public abstract class Animal {
    private String name;

    public abstract void cry();   // 每种动物都要叫，但叫法不同

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}

public class Dog extends Animal {
    @Override
    public void cry(){
        System.out.println(getName() + "汪汪汪的叫~~~");
    }
}

public class Cat extends Animal {
    @Override
    public void cry(){
        System.out.println(getName() + "喵喵喵的叫~~~");
    }
}
```

### 3. 单例的实现步骤（背诵级）

1. 把类的构造器私有；
2. 定义一个静态变量记住类的唯一对象（饿汉直接 new，懒汉先置 null）；
3. 定义一个静态方法返回该对象（懒汉要判空创建）。

### 4. 枚举替代常量的判断

当一组选项数量固定且有限（方向、性别、状态码），用枚举而不是 int/String 常量：传参时编译器就能挡住非法值，可读性和安全性都更好。

## 四、重点与难点分析

### 重点 1：多态的"三前提一注意"

继承/实现 + 父类引用子类对象 + 方法重写，三者缺一不可。注意"成员变量不谈多态"是高频考点：通过父类引用访问同名成员变量，永远拿到父类的。

### 重点 2：抽象类不能 new

`new Animal()` 编译报错。抽象类的价值在于"规定子类必须做什么"，它是设计的契约层，不是实例的来源。

### 难点 1：ClassCastException 的发生时机

存在继承/实现关系，强转在编译期一律放行，真正的类型校验发生在运行时。`People p = new Teacher(); Student s = (Student) p;` 编译没问题，运行才炸。防御手段就是强转前 `instanceof`。

### 难点 2：饿汉与懒汉的权衡

饿汉式类加载时就创建对象，简单且天然线程安全，但不用时也占内存；懒汉式延迟加载省内存，但多线程下需要额外处理并发问题。入门阶段记住形态差异即可，并发细节到多线程阶段再深入。

### 难点 3：final 引用变量的"不可变"边界

`final Student s = new Student();` 之后不能再 `s = new Student();`，但 `s.setName("x")` 完全合法。final 锁的是引用本身，不是对象内容。

### 易错点清单

- 用多态引用调用子类独有方法，编译报错；
- 忘记重写抽象类的某个抽象方法，子类编译报错（要么补重写，要么子类也声明为抽象）；
- 枚举第一行写了成员变量定义——枚举对象名称必须位于第一行；
- 试图继承枚举类或用 new 创建枚举对象。

## 五、示例

### 示例 1：多态下的类型转换与 instanceof

```java
People p = new Teacher();

// p.teach();   // 报错：多态下不能调用子类独有功能

// 强制类型转换恢复子类类型
if (p instanceof Teacher) {
    Teacher t = (Teacher) p;
    t.teach();   // 安全调用子类独有功能
}
```

### 示例 2：抽象类支持多态

```java
public static void main(String[] args) {
    Animal a1 = new Dog();
    a1.setName("旺财");
    a1.cry();   // 旺财汪汪汪的叫~~~

    Animal a2 = new Cat();
    a2.setName("咪咪");
    a2.cry();   // 咪咪喵喵喵的叫~~~
}
```

### 示例 3：常量记录系统配置

```java
public class Constant {
    public static final String SCHOOL_NAME = "示例学校";
}

// 使用处
System.out.println(Constant.SCHOOL_NAME);
// 编译后会被"宏替换"为字面量，性能与直接写字面量一致
```

### 示例 4：饿汉式单例的完整使用

```java
public class A {
    private static A a = new A();
    private A() {
    }
    public static A getObject(){
        return a;
    }
}

public class Test {
    public static void main(String[] args) {
        A a1 = A.getObject();
        A a2 = A.getObject();
        System.out.println(a1 == a2);   // true：两次拿到的是同一个对象
    }
}
```

### 示例 5：枚举做方向控制

```java
public enum Direction {
    UP, DOWN, LEFT, RIGHT
}

public static void move(Direction direction){
    switch (direction) {
        case UP:
            System.out.println("向上移动");
            break;
        case DOWN:
            System.out.println("向下移动");
            break;
        // ...
    }
}

// 调用：move(Direction.UP); 传枚举之外的值编译期就会报错，比 int 常量安全
```
