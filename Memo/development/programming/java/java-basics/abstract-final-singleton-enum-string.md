---
type: memo
date: 2026-11-09
aliases:
  - "模板方法设计模式"
  - "单例设计模式"
  - "枚举类"
---

# 抽象类、final、单例、枚举与 String

提到抽象类，我们还是以动物、猫、狗为例。如果说猫在叫唤，大家都能想到叫声是“喵喵”，如果说狗在叫唤，大家都能想到叫声是“汪汪”，但如果说一个动物在叫唤，你能想象出是怎么叫唤的吗？此时，未指定是具体哪种类型的动物，肯定不知道应该如果叫唤，所以动物类中的叫唤其实应该是没具体功能的，具体功能应该放在子类中。

具体到Java的动物类中，就是只有一个方法名，但无功能。而这个方法就称为抽象方法，此时该类也应该是抽象类。接下来，我们来学习抽象类的具体语法。

## 认识抽象类

在java中有一个关键字：abstract，它就是抽象的意思，可以用它修饰类、成员方法：

- abstract修饰类，这个类就是抽象类。

```Java
修饰符 abstract class 类名{
  修饰符 abstract 返回值类型 方法名称(形参列表)；
}
```

- abstract修饰方法，这个方法就是抽象方法。

```Java
public abstract class A {
    // 抽象方法：必须abstract修饰，只有方法签名，不能有方法体
    public abstract void test();
 }
```

抽象类、抽象方法必须遵循以下规则：

1. 有抽象方法的类必须是抽象类
2. 抽象类中不一定要有抽象方法
3. 类有的成员：成员变量、方法、构造器，抽象类都可以有。
4. 抽象类最主要的特点：抽象类不能创建对象，仅作为一种特殊的父类，让子类继承并实现。
5. 一个类继承抽象类，必须重写完抽象类的全部抽象方法，否则这个类也必须定义成抽象类。

下面通过代码，演示抽象类的写法：

```Java
// 定义一个抽象类 Animal
public abstract class Animal {
    // 抽象方法，没有具体的实现
    public abstract void makeSound();

    // 具体方法，有具体的实现
    public void eat() {
        System.out.println("动物正在吃东西。");
    }

    // 具体方法，有具体的实现
    public void sleep() {
        System.out.println("动物正在睡觉。");
    }
}

// 定义一个具体的子类 Dog 继承自抽象类 Animal
public class Dog extends Animal {
    // 重写抽象方法 makeSound
    @Override
    public void makeSound() {
        System.out.println("狗叫：汪汪！");
    }

    // 可以选择性地重写具体方法
    @Override
    public void eat() {
        System.out.println("狗正在吃狗粮。");
    }
}

// 定义另一个具体的子类 Cat 继承自抽象类 Animal
public class Cat extends Animal {
    // 重写抽象方法 makeSound
    @Override
    public void makeSound() {
        System.out.println("猫叫：喵喵！");
    }

    // 可以选择性地重写具体方法
    @Override
    public void sleep() {
        System.out.println("猫正在一个舒适的地方睡觉。");
    }
}

//使用抽象类
public class Main {
    public static void main(String[] args) {
        // 创建 Dog 对象
        Animal myDog = new Dog();
        myDog.makeSound();  // 输出: 狗叫：汪汪！
        myDog.eat();        // 输出: 狗正在吃狗粮。
        myDog.sleep();      // 输出: 动物正在睡觉。

        // 创建 Cat 对象
        Animal myCat = new Cat();
        myCat.makeSound();  // 输出: 猫叫：喵喵！
        myCat.eat();        // 输出: 动物正在吃东西。
        myCat.sleep();      // 输出: 猫正在一个舒适的地方睡觉。
    }
}
```

## 抽象类的好处

抽象类的主要目的是为子类提供一个通用的模板，并强制子类实现某些特定的方法。父类知道每个子类都要做某个行为，但每个子类要做的情况不一样，父类就定义成抽象方法，交给子类去重写实现，我们设计这样的抽象类，就是为了更好的支持多态。

比如：动物都会叫唤，但每种动物的叫唤方式不同，所以不能在父类中实现具体的叫唤功能，而需要在各个不同的子类中才能实现。但无论哪种动物，都会叫唤，所以我们可以在父类Animal类中定义一个叫唤的方法，但是又不确定又该如何叫唤，所以可以将方法声明成抽象的。这样一来，所有继承该类的子类， 都得提供这个方法，就实现了强制子类实现特定方法的目的。

假设我们有如下需求：

- 某宠物游戏，需要管理猫、狗的数据。
- 猫的数据有：名字；行为是：喵喵喵的叫\~
- 狗的数据有：名字；行为是：汪汪汪的叫\~
- 请用面向对象编程设计该程序。

我们可以使用抽象类，进行如下编码：

```Java
public abstract class Animal{
    private String name;

    public abstract void cry();

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
 }

 public class Dog extends Animal{

    @Override
    public void cry(){
        System.out.println(getName()+ "汪汪汪的叫~~~");
    }

 }

 public class Cat extends Animal{

    @Override
    public void cry(){
        System.out.println(getName()+ “喵喵喵的叫~~~");
    }

 }
```

以上代码，通过在父类Animal中声明抽象方法cry()，强制它的各个子类必须提供该方法的功能。而且因为父类中有了抽象方法，所以父类必须是抽象类。

## 模板方法设计模式

模板方法设计模式是一种行为设计模式，它定义了一个算法的骨架，将一些步骤延迟到子类中实现。通过这种方式，模板方法允许子类在不改变算法整体结构的情况下重定义算法的某些特定步骤。

![IZonbftr0oHcjdxTqmScAowrnuc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/IZonbftr0oHcjdxTqmScAowrnuc.png)

**主要特点：**

1. **算法框架**：父类中定义了一个模板方法，包含算法的基本步骤，而每个具体步骤由子类实现。
2. **代码复用**：公共的算法部分可以在父类中实现，子类只需实现特定的细节，从而避免代码重复。

下面代码演示了模板方法设计模式：

```Java
package com.itheima.abstract3;
public abstract class People {
   // 1、模板方法设计模式。
    public final void write(){
        System.out.println("\t\t\t《我的爸爸》");
        System.out.println("\t我爸爸是一个好人，我特别喜欢他，他对我很好，我来介绍一下：");
        // 2、模板方法知道子类一定要写这个正文，但是每个子类写的信息是不同的，父类定义一个抽象方法
        // 具体的实现交给子类来重写正文。
        writeMain();
        System.out.println("\t我爸爸真好，你有这样的爸爸吗？");
    }

    public abstract void writeMain();
}

package com.itheima.abstract3;
public class Student extends People{
    @Override
    public void writeMain() {
        System.out.println("\t我爸爸很牛逼，是个管理者，我开车不用看红绿灯的，有这样的爸爸真好，下辈子还要做他儿子！");
    }
}

package com.itheima.abstract3;
public class Teacher extends People {
    @Override
    public void writeMain() {
        System.out.println("\t我爸爸经常让我站在这里别动，他要去买几斤橘子~~柚子我也吃，毕竟是亲生的！");
    }
}

package com.itheima.abstract3;
public class Test {
    public static void main(String[] args) {
        // 目标：理解抽象类的使用场景之二：模板方法设计模式。
        // 学生和老师都要写一篇作文：《我的爸爸》
        //         第一段是一样的：我爸爸是一个好人，我特别喜欢他，他对我很好，我来介绍一下：
        //         第二段是不一样：老师和学生各自写各自的。
        //         第三段是一样的：我爸爸真好，你有这样的爸爸吗？
        // 解决：抽出一个父类。父类中还抽取一个模板方法给子类直接用。
        Student s = new Student();
        s.write();
    }
}
```

<callout emoji="💡">
多学一招：**建议使用final关键字修饰模板方法, 可以学习final修饰方法后再来优化**
模板方法是给子类直接使用的，不能被子类重写。
一旦子类重写了模板方法，模板方法就失效了。
</callout>

## final

### 认识final

final 关键字是最终的意思，在Java中，final 关键字可以用来修饰类、方法和变量。它的主要作用是限制被修饰的实体不能被修改或覆盖，具体来说以以下3种情况：

- 修饰类：该类被称为最终类，特点是不能被继承了。
- 修饰方法：该方法被称为最终方法，特点是不能被重写了。
- 修饰变量：该变量有且仅能被赋值一次。

下面是 final 关键字的用法及其示例：

1. 修饰变量

当 final 修饰一个基本类型变量时，该变量的值将不能被改变

```Java
final int speed = 50; // 基本类型，值不能改变
```

1. 修饰方法

final 方法不能被子类重写（override）。这通常用于确保某个方法的行为不会被子类改变，以保持一致性和安全性。

```Java
class SuperClass {
    final void show() { // final 方法
        System.out.println("This is a final method.");
    }
}

class SubClass extends SuperClass {
    // 下面的尝试会编译错误，因为不能重写 final 方法
    // @Override
    // void show() {}
}
```

1. 修饰类final 类不能被继承。这常用于防止某些设计良好的类结构被破坏，比如 String 类就是一个典型的 final 类。

   ```Java
   final class FinalClass {
       // 类体
   }

   // 下面的尝试会编译错误，因为不能继承 final 类
   // class AnotherClass extends FinalClass {}
   ```

final修饰变量的注意事项：

- java中的变量分类：基本类型、引用类型（指向对象）
- final修饰基本类型的变量，变量存储的数据不能被改变。
- final修饰引用类型的变量，变量存储的地址不能被改变，但地址所指向对象的内容是可以被改变的。

```Java
final int speed = 50; // 基本类型，值不能改变

final int[] arr = {1,2,3};// 引用类型, 是地址值不能改变但是地址中的内容可以修改
//arr = new int[10];// 错误的
arr[0] = 1000;// 这个是可以的
```

### 常量

在Java中，常量是指其值在整个程序执行期间不会改变的变量。通常使用 `static final` 修饰符来定义一个常量，这样做的好处是：

- final：确保该变量的值一旦被赋值后就不能再更改。
- static：使得这个变量成为类级别的变量，而不是实例级别的变量，这意味着不需要创建类的实例就可以访问到这个变量，并且所有实例共享同一份数据。

简而言之：在java中，使用了 static final 修饰的成员变量就被称为常量，常用于记录系统的配置信息。如以下代码所示：

```Java
public class Constant {
    public static final String SCHOOL_NAME  = "传智教育";
}
```

上面类中的 SCHOOL_NAME就是一个常量，它的值不能被修改。

<callout emoji="💡">
多学一招
常量名的命名规范：建议使用大写英文单词，多个单词使用下划线连接起来。
</callout>

为什么使用常量呢？常量主要用于记录系统的一些配置信息，使用常量记录这些不变的信息，有以下优势：

- 代码可读性更好，可维护性也更好。
- 程序编译后，常量会被“宏替换”：出现常量的地方全部会被替换成其记住的字面量，这样可以保证使用常量和直接用字面量的性能是一样的。

常量定义之后，就可以在程序任何地方使用。由于常量是静态的，所以你可以通过类名直接访问它们，而无需创建类的对象：

```Java
public class Main {
    public static void main(String[] args) {
        System.out.println("学校名称: " + Constant.SCHOOL_NAME);
    }
}
```

## 单例类

### 认识设计模式

一个问题通常有n种解法，其中肯定有一种解法是最优的，这个最优的解法被人总结出来了，称之为设计模式。设计模式可以理解为解决特定问题的一套通用解决方案模板。

通俗一点说：

- 经验总结：设计模式是从许多开发者的经验中总结出来的，提供了一些通用的方法来解决特定类型的问题。
- 提高效率：使用设计模式可以避免重复造轮子，让开发者更快速地解决问题，因为这些模式已经被广泛使用并且证明有效。

在Java中有20多种设计模式，对应20多种软件开发中会遇到的问题。java中的设计模式通常分为三大类：

1. 创建型模式（Creational Patterns）：关注于对象的创建方式，如单例模式（Singleton）、工厂模式（Factory）等。
2. 结构型模式（Structural Patterns）：处理类或对象组合的方式，如适配器模式（Adapter）、装饰者模式（Decorator）等。
3. 行为型模式（Behavioral Patterns）：涉及对象之间的职责分配以及它们之间的通信，如观察者模式（Observer）、策略模式（Strategy）等。

在软件开发中，我们要解决某个问题，也需要适当的解决方案，那么这20多种现成的解决方案可以供我们使用，不必从头自己琢磨，所以学习设计模式可以快速提升程序员解决问题的能力。

我们在学习设计模式时，需要重点关注两个方面：

1. 解决什么问题？

每个设计模式，都有其对应的使用场景，我们需要知道各个设计模式主要是用来解决什么问题的。

1. 怎么写？

知道了某个设计模式能解决哪类问题之后，我们还需要知道如果遵循这个设计，具体应该如何写代码。在每个设计模式中都非常明确的规定了该模式下代码的编写规则。我们需要学习这些编码规则。

### 单例设计模式(饿汉式单例)

单例设计模式是一种创建型设计模式，其主要作用是确保一个类只有一个实例，并提供一个全局访问点来获取这个实例。

在Java中，实现单例模式有多种方法，包括懒汉式、饿汉式、双重检查锁定、静态内部类方式、枚举方式。我们今天先来学习前面两种方式：懒汉式单例、饿汉式单例。

饿汉式单例模式是一种在类加载时就创建单例实例的设计模式。由于实例是在类加载时就创建好的，所以它被称为“饿汉式”，因为它很“急切”地创建了实例，而不是等到实际需要时才创建。

饿汉式单例模式的规则如下：

- 把类的构造器私有。
- 定义一个静态变量记住类的一个对象。
- 定义一个静态方法，返回对象。

下面是一个饿汉式单例模式的Java实现：

```Java
 // 单例类
public class A {
    // 2、定义一个静态变量记住类的一个对象
    private static A a = new A();

    // 1、私有构造器
    private A(){
    }

    // 3、定义一个静态方法返回对象
    public static A getInstance(){
        return a;
    }
 }
```

那么应该如何使用这个单例类呢？你可以在你的应用程序中通过 getInstance() 方法来获取 A 类的唯一实例，并调用其方法，而且无论何时调用 `getInstance()` 方法，都会返回相同的 `EagerSingleton` 实例。：

```Java
public class Main {
    public static void main(String[] args) {
        // 获取单例实例并调用方法
        A a1= A.getInstance();
        a1.showMessage();

        // 再次尝试获取实例，验证是否为同一个实例
        A a2= A.getInstance();
        System.out.println(a1== a2);  // 输出 true
    }
}
```

### 单例设计模式(懒汉式单例)

接下来我们学习单例模式的第二种写法：懒汉式单例。

懒汉式单例模式是一种延迟初始化的单例实现方式，即单例实例在第一次被请求时才创建。这种方式的好处是如果单例对象从未被使用，则不会浪费资源去创建它。

懒汉式单例模式的规则如下：

- 把类的构造器私有。
- 定义一个静态变量用于存储对象。
- 提供一个静态方法，保证返回的是同一个对象。

下面是一个懒汉式单例模式的Java实现：

```Java
public class B {
    // 2、定义一个类变量量用于存储对象
    private static B b ; // null
    // 1、单例必须私有构造器
    private B(){
    }

    // 3、提供一个类方法返回类的一个对象
    public static B getInstance(){
        if(b == null){
            b = new B();
        }
        return b;
    }
 }
```

代码说明：

- 第3行：声明了B类型的静态变量，但未实例化
- 第9行：提供了getInstance()方法，以供外部来获取实例对象
- 第10-13行：判断如果尚未初始化过B对象，则new B()创建对象，如果已经创建过，则跳过第11行的创建代码，直接return，即：单例实例在第一次被请求时才创建。

上面代码演示的是一个最基本的懒汉式单例实现，但它不是线程安全的，在多线程环境下可能会导致创建多个实例，这个问题在后面学习了线程知识后再进行优化。

## 枚举类

### 认识枚举类

在Java中，枚举（`enum`）是一种特殊的类型，用于定义一组常量。它可以使代码更加清晰、可读。

枚举类的写法如下：

![Lxh2b3SVJoH3EBxx6zRcT9StnIf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Lxh2b3SVJoH3EBxx6zRcT9StnIf.png)

语法说明：

- 枚举类中的第一行，只能写枚举类的对象名称，且要用逗号隔开。
- 这些名称，本质是常量，每个常量都记住了枚举类的一个对象。
- 枚举中可以有成员变量，也可以没有成员变量

下面通过代码演示一下枚举类的写法：

示例1： 不带成员变量的枚举

```Java
public enum Day {
    SUNDAY,
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY;
}
```

代码说明：

- 这个枚举类用于表示星期几
- 没有成员变量：该枚举类仅定义了星期几的常量，没有附加的属性或方法。

示例2： 带成员变量的枚举

```Java
public enum Season {
    WINTER("冬季"),
    SPRING("春季"),
    SUMMER("夏季"),
    FALL("秋季");

    private final String description;

    // 构造函数
    Season(String description) {
        this.description = description;
    }

    // 获取描述的方法
    public String getDescription() {
        return description;
    }
}

```

代码说明：

- 这个枚举类表示不同的季节，每个季节都有一个描述性字符串作为成员变量。
- 成员变量：description，用于存储每个季节的描述。
- 构造函数：私有构造函数（Season(String description)），确保每个枚举常量都有相应的描述
- 方法：getDescription()，用于获取季节的描述信息。

虽然在写枚举时，没有用到class关键字，但其实枚举的本质依然是类。我们可以通过对java代码进行反编译，观察其底层代码。如下所示，有一个简单的枚举：

```Java
public enum A{
    X , Y , Z;
 }
```

可以使用javap命令对枚举进行反编译，得到如下结果：

```Java
Compiled from “A.java"
public final class A extends java.lang.Enum<A> {
    public static final A X = new A();
    public static final A Y = new A();
    public static final A Z = new A();

    public static A[] values();
    public static A valueOf(java.lang.String);
 }
```

这就是枚举类被编译以后的最终代码，我们可以看到枚举最终被转写成了final class，而且继承了java.lang.Enum类。其实所有的枚举类在底层都有如下特点：

- 枚举都是最终类，不可以被继承，枚举类都是继承java.lang.Enum类的。
- 枚举类的第一行只能罗列一些名称，这些名称都是常量，并且每个常量会记住枚举类的一个对象。
- 枚举类的构造器不能是（写不写都只能是私有的），因此，枚举类对外不能创建对象。
- 编译器为枚举类新增了几个方法（values、valueOf）

### 枚举的应用场景

学习了枚举的编写语法之后，我们来看一下到底在哪些场景需要用到枚举。Java枚举（`enum`）非常适合用于表示一组固定的、预定义的常量。以下是一些适合使用枚举的常见场景：

1. 表示固定集合的状态或选项

当你需要表示一组有限且明确的状态或选项时，枚举是非常合适的。例如：

- 星期几（`MONDAY`, `TUESDAY`, `WEDNESDAY` 等）
- 方向（`NORTH`, `SOUTH`, `EAST`, `WEST`）
- 月份（`JANUARY`, `FEBRUARY`, `MARCH` 等）
- 颜色（`RED`, `GREEN`, `BLUE` 等）

1. 类型安全的常量

使用枚举可以确保类型安全，避免在代码中使用错误的字符串或其他类型的常量。例如：

- 操作系统类型（`WINDOWS`, `LINUX`, `MAC_OS`）
- 用户角色（`ADMIN`, `USER`, `GUEST`）

1. 包含额外信息和行为

枚举不仅可以表示简单的常量，还可以包含额外的数据和方法。这使得枚举非常灵活，可以处理更复杂的逻辑。例如：

- 一个表示行星的枚举，每个行星都有质量和半径等属性。
- 一个表示日历事件类型的枚举，每个事件类型都有特定的行为。

示例：假设我们要开发一款石头迷阵游戏，在这个游戏中玩家可以通过方向键上下左右移动石块，最终将数字恢复成按大小顺序排列。

![BbWybGmzxoE1EVxeLaFcRyzgnIc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/BbWybGmzxoE1EVxeLaFcRyzgnIc.png)

在这个游戏中，我们需要识别用户的移动方向，而移动方向共有4个：上、下、左、右。那么为了编码方便，我们就可以使用枚举来表示各个方向：

```Java
public enum Direction {
    UP, DOWN, LEFT, RIGHT
}
```

其实在jdk自带的好多类中也大量使用了枚举，比如：`RoundingMode` 是 Java 中的一个枚举类，位于 `java.math` 包中。它定义了在数值进行舍入操作时的各种舍入模式。`RoundingMode` 通常与 `BigDecimal` 类一起使用，以控制 `BigDecimal` 对象的舍入行为。

`RoundingMode` 提供了多种舍入模式，每种模式都有其特定的行为和用途。这些模式包括：

- UP：远离零的方向舍入。
- DOWN：向零的方向舍入。
- CEILING：向正无穷大的方向舍入。
- FLOOR：向负无穷大的方向舍入。
- HALF_UP：四舍五入（默认模式）。
- HALF_DOWN：向最近的邻居舍入，如果两个邻居等距，则向零的方向舍入。
- HALF_EVEN：向最近的邻居舍入，如果两个邻居等距，则向偶数舍入（银行家舍入法）。
- UNNECESSARY：断言请求的操作具有精确的结果，因此不需要舍入

下面示例演示了使用BigDecimal进行数学计算时，如何指定舍入模式：

![O4S6btxrcoODpYxcfl9cncafnhf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/O4S6btxrcoODpYxcfl9cncafnhf.png)

## String

概述:

<callout emoji="🌷">
String 是 Java 中最常用的类之一，用于表示字符串文本, String是一个不可改变的字符序列, 一旦创建无法修改
</callout>

**特点: String一旦创建无法改变**

### 字符串对象的获取:

第一种方式: 通过传统new 的方式构造

```Java
 String();   创建一个空白的字符串, 类似于  String str = ""; 字符串没有内容
 String(String str) 创建一个有内容的字符串
 String(byte[] arr) 将字节数组的数据转成字符串对象
 String(char[] arr) 将字符数组的数据转成字符串对象
```

第二种方式: 字符串常量方式, 在java中所有字符串的常量, 都是字符串的对象

```Java
String s = "字符串内容";
```

例如:

```Java

private static void demo1() {
    //1.创建字符串
    String s1 = new String();
    String s2 = new String("梅军建真帅");
    char[] arr = {'a', 'b', 'c'};
    String s3 = new String(arr);
    byte[] arr1 = {97, 98, 99};
    String s4 = new String(arr1);
    String s5 = "梅军建马上给大家表演一段舞蹈";
    System.out.println(s1);
    System.out.println(s2);
    System.out.println(s3);
    System.out.println(s4);
    System.out.println(s5);
}
```

### 字符串常见的功能

```Java
字符串查询操作
int length(); 求出字符串的字符个数
char charAt(int index)：获取指定索引位置的字符
int indexOf(String str)：查找子字符串首次出现索引位置
int lastIndexOf(String str)：查找子字符串最后出现索引位置
boolean contains(CharSequence s)：判断是否包含指定字符序列
字符串修改操作
String substring(int beginIndex)：截取子字符串
String  replace(char oldChar, char newChar)：字符替换
String  trim()：去除首尾空白字符
字符串比较操作
boolean equals(Object anObject)：内容相等性比较
boolean equalsIgnoreCase(String anotherString)：忽略大小写比较
boolean startsWith(String prefix)：前缀匹配检查
boolean endsWith(String prefix)：前缀匹配检查
字符串转换操作
String toUpperCase()：转换为大写
String  toLowerCase()：转换为小写
String[] split(String regex)：按正则表达式分割
```

案例: 演示功能

```Java
public class Test {

    public static void main(String[] args) {
        //alt + ctrl + M
        demo8();
    }
    public static void demo8() {
    //1.先创建字符串对象
    String s = "abcdAAA梅军建asdfdasd梅军建asdfasdf";
    //2. int indexOf(String str); 查询 str在本字符串第一次出现的位置, 默认返回首字符位置
    int index = s.indexOf("梅军建");
    System.out.println(index);
    //3.int lastIndexOf(String str)查询 str在本字符串最一次出现的位置, 默认返回首字符位置
    int index1 = s.lastIndexOf("梅军建");
    System.out.println(index1);
    //4.String toUpperCase() String toLowerCase()  原串不变, 返回新串!!!
    String upperCase = s.toUpperCase();
    System.out.println(upperCase);
    String lowerCase = s.toLowerCase();
    System.out.println(lowerCase);
    String[] arr = {"d:/aaa/a/a/a/asdf.txt", "d:/s/d/f/sdf/asdf.jpg", "d:/asdf/sdfdsf/asdf.mp4", "d:/asdf/sdfdsf/梅军建演唱会.mp3", "e:/s/s/s/asdf.mp3", "asdf.doc", "asdf.txt", "a.java"};
    for (int i = 0; i < arr.length; i++) {
        String s1 = arr[i];
        //查询最后一个/的位置
        int loc = s1.lastIndexOf("/");
        //从/+1的位置截取到最后, 取出文件名
        String name = s1.substring(loc + 1);
        //判断name是否以.mp3结尾
        if (name.endsWith(".mp3")) {
            System.out.println(name);
        }
    }
}

    private static void demo7() {
        String s = "....aaa.bbb.ccc.ddd...";
        /*
            String[] split(String str); 按照str将本字符串切割成若干份
            (1) 切割点不保留
            (2) 切割点如果是.的话要注意, 必须使用\\.转义,因为这个方法时一个正则表达式的方法
                .代表所有字符
            (3) 切割点尽量不要在前后,跟你想的不一样, 前面算内容后面不算内容
         */
        String[] arr = s.split("\\.");
        //遍历数组看内容
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
        System.out.println(arr.length);

    }

    private static void demo6() {
        //1.准备一个字符串
        String s1 = "abcdefg";
        //boolean contains(String obj); 判断当前字符串是否包含obj
        boolean cd = s1.contains("ce");
        System.out.println(cd);
        // boolean startsWith(String other); 判断当前字符串是否以other字符串开头
        boolean b = s1.startsWith("ac");
        System.out.println(b);
        // boolean endsWith(String other); 判断当前字符串是否以other字符串结尾
        boolean b1 = s1.endsWith("fg");
        System.out.println(b1);

    }

    public static void demo5() {
        String s1 = "abacdaef";
        //截取: String substring(int 开始索引, int 结束索引); 含头不含尾, 原串不变返回截取后的新串
        String result = s1.substring(1, 4);
        System.out.println(result);
        String substring = s1.substring(1);//从开始截取到末尾
        System.out.println(substring);
        //替换: 我将字符串中所有的a都替换成梅军建 String replace(CharSequence target, CharSequence replacement);原串不变返回新串
        String replace = s1.replace("a", "梅军建");
        System.out.println(replace);

    }

    public static void demo4() {
        //1.得到字符串
        String s1 = "abcdef";
        //2.访问char[] toCharArray();
        for (int i = 0; i < s1.length(); i++) {
            char c = s1.charAt(i);
            System.out.println(c);
        }

    }

    public static void demo3() {
        //1.得到字符串
        String s1 = "abcdef";
        //2.访问char[] toCharArray();
        char[] charArray = s1.toCharArray();
        for (int i = 0; i < charArray.length; i++) {
            System.out.println(charArray[i]);
        }
        String s2 = new String("abcdef");
        //3. 使用boolean equals(Object obj);
        boolean equals = s1.equals("abcdef");
        System.out.println(s1 == s2);
        boolean b = s1.equalsIgnoreCase("ABCdef");
        System.out.println(b);

    }

    /**
     * 学会字符串的方法
     */
    public static void demo2() {
        //1.得到字符串
        String s1 = "abcdef";
        //2.使用其 int length()功能,获取字符串的长度
        int length = s1.length();
        System.out.println(length);
        //3.使用 char charAt(int index); 找出索引对应的字符
        char c = s1.charAt(3);
        System.out.println(c);

    }

    /**
     * 学会创建字符串
     */
    private static void demo1() {
        //1.创建字符串
        String s1 = new String();
        String s2 = new String("梅军建真帅");
        char[] arr = {'a', 'b', 'c'};
        String s3 = new String(arr);
        byte[] arr1 = {97, 98, 99};
        String s4 = new String(arr1);
        String s5 = "梅军建马上给大家表演一段舞蹈";
        System.out.println(s1);
        System.out.println(s2);
        System.out.println(s3);
        System.out.println(s4);
        System.out.println(s5);
    }
}
```

注意:

<callout emoji="🌷">
**String是不可改变的字符串序列, 所有修改字符串的操作底层都是采用new的方式创建了一个新的字符串**
</callout>

### 案例

#### 案例1

*题目: 键盘录入一个字符串, 统计大写字母的个数和小写字母的个数*

分析:

1. 创建键盘录入对象, 接受一个字符串
2. 定义变量记录大写字母个数和小写字母个数
3. 通过 chartAt 和 length 方法遍历字符串,得每一个字符
4. 字符在底层也是数字, 所以大写字母的范围:  >='A'   &&  <='Z' 字符,  小写字母的范围:  >='a' &&  <= 'z'

实现:

```Java
public static void demo1() {
    //1.键盘录入一个字符串
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入字符串");
    String str = sc.next();
    int bigCaseCount = 0;
    int smallCaseCount = 0;
    //2.遍历字符串, 拿到每一个字符
    for (int i = 0; i < str.length(); i++) {
        char c = str.charAt(i);
        //3.判断字符是否是大小字母
        if (c >= 'A' && c <= 'Z') {
            bigCaseCount++;
        } else if (c >= 'a' && c <= 'z') {
            smallCaseCount++;
        }
    }
    //4.输出结果
    System.out.println("大写字母是:" + bigCaseCount);
    System.out.println("小写字母是:" + smallCaseCount);
}
```

#### 案例2

题目: *键盘录入身份证, 得到生日信息*

分析:

1. 创建键盘录入对象, 接受身份证信息
2. 生日信息在索引6\~13位索引处, 我们可以采用substring方法截取生日信息, 但是这个方法含头不含尾所以 6\~14才行

实现:

```Java
public static void demo2() {
    //1.键盘输入身份证的内容
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入身份证");
    String str = sc.next();
    //2.从身份证获取生日信息 ---> substring
    String birthday = str.substring(6, 14);
    System.out.println(birthday);
}
```

#### 案例3

题目: 现在已知脏话表如下: String[] arr = {"tmd", "sb", "ndy"}; 请键盘录入模拟用户发出内容, 将用户录入的内容中的脏话采用\*\*\*替换

分析:

1. 键盘输入字符串数据
2. 遍历脏话数组, 将刚才输入的字符串内容采用replace替换\*\*\*即可

实现:

```Java
public static void demo3() {
    String[] arr = {"tmd", "sb", "ndy"};

    Scanner sc = new Scanner(System.in);
    String data = sc.next();
    for (int i = 0; i < arr.length; i++) {
        String s = arr[i];
        //将data中的s都替换成***
        data =  data.replace(s,"***");
    }
    System.out.println(data);

}
```

### 常量方式和new方式的区别

结论:

<callout emoji="🌷">
*我们采用常量赋值的方式的话, 字符串对象是堆中的常量池中创建的, 而常量创建方式, 首先检查常量池是否有该字符串如果有可以复用地址, 如果没有则创建新的*
*我们采用new的方式, 只要new就会在堆中开辟新的空间!!!!*
</callout>

面试题:

```Java
public class Test1 {
    public static void main(String[] args) {
        //1.第一种常量赋值
        String s1 = "abc";
        String s3 = "abc";
        //2.第二种通过构造创建
        String s2 = new String("abc");
        String s4 = "ab";
        String s5  =s4 +"c";//"abc"
        String s6  ="ab" +"c";//"abc" "abc"

        System.out.println(s1 == s2); // false  s1在常量方式, s2是new的方式
        System.out.println(s1 == s3); // true   s1和s2都在常量方式, 内容一致可以复用
        System.out.println(s1 == s5);// false  字符串是不可改变的字符序列, 一旦改变了就会创建新的字符串, s5是new的方式, s1常量方式
        System.out.println(s1 == s6); // true, 因为s6后面都是常量在编译期间就已经合并了, 所以在运行期间都是常量方式创建

    }
}
```

结论:

<callout emoji="🌷">
以后引用类型比较不能采用==符号进行对比, 应该在保证重写equals方法的时候,使用equals方法进行对比才行
</callout>

## 相关笔记

- [[Memo/development/programming/java/oop/07-oop-advanced-polymorphism.md|07 多态·课堂问答（含抽象类与 String）]]
- [[Memo/development/programming/java/oop/10-polymorphism-abstract-final-singleton-enum.md|多态、抽象类、final、单例与枚举精讲]]
- [[Memo/development/programming/java/java-syntax-and-control-flow/09-java-string-class.md|String 类精讲]]

---

> 来源：飞书《讲义》知识库 · day08-面向对象高级(抽象类、String) · [原文](https://baxsu7o100e.feishu.cn/wiki/YUoAwpn8hi1OhZkWbGz1rKiN8Tg)
