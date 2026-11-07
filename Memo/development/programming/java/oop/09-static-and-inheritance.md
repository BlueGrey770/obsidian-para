---
type: memo
date: 2026-11-06
aliases:
  - "09-static-yu-ji-cheng"
  - "Java static 关键字与继承机制精讲"
---

# Java static 关键字与继承机制精讲

## 一、概念说明

### 1. static 修饰成员变量

成员变量按有无 `static` 修饰分为两种：

- **静态变量（类变量）**：有 static 修饰，属于类，随类加载只初始化一次，在内存中只有一份，被类的全部对象共享；
- **实例变量（对象变量）**：无 static 修饰，属于每个对象，每个对象中各有一份。

访问方式：

```java
类名.静态变量     // 推荐
对象名.静态变量   // 不推荐（能通过编译，但语义上容易误解为属于对象）
对象.实例变量     // 唯一方式
```

```java
public class Student {
    static String name;   // 静态变量
    int age;              // 实例变量
}

public class Test {
    public static void main(String[] args) {
        // 类变量的用法
        Student.name = "张三";          // 类名.静态变量（推荐）

        Student s1 = new Student();
        s1.name = "李四";              // 对象.静态变量（不推荐）

        Student s2 = new Student();
        s2.name = "王五";

        System.out.println(s1.name);      // 王五：静态变量只有一份，被共享覆盖
        System.out.println(Student.name); // 王五

        // 实例变量的用法
        s1.age = 23;
        s2.age = 18;
        System.out.println(s1.age);       // 23
        // System.out.println(Student.age); // 报错：实例变量属于对象
    }
}
```

静态变量的应用场景：某个数据只需要一份，且希望被共享（访问、修改），就定义成静态变量。典型案例：用户类记住自己创建了多少个用户对象。

```java
public class User {
    // 静态变量：记录创建对象的总数
    public static int number;

    // 构造器：每创建一个对象就 +1
    public User(){
        User.number++;
    }
}
```

### 2. static 修饰成员方法

- **静态方法（类方法）**：有 static 修饰，属于类，推荐 `类名.静态方法` 调用；
- **实例方法**：无 static 修饰，属于对象，只能 `对象.实例方法` 调用。

静态方法的常见应用场景是**做工具类**：工具类中的方法都是静态方法，每个方法完成一个功能，供开发者直接调用，提高代码复用和开发效率。实例方法需要创建对象来调用，对象占内存，纯属浪费；静态方法用类名直接调用，方便且省内存。工具类没有创建对象的需求，建议把构造器私有化。

### 3. static 的注意事项

1. 静态方法中可以直接访问静态成员，**不可以直接访问实例成员**；
2. 实例方法中既可以直接访问静态成员，也可以直接访问实例成员；
3. 实例方法中可以出现 this 关键字，静态方法中不可以出现 this。

根本原因：静态成员随类加载就存在，实例成员要等对象创建后才存在，静态方法执行时实例成员可能还没有内存。

### 4. 继承

继承用 `extends` 关键字让两个类建立父子关系，目的是**提高代码的重用性，减少重复代码的书写**。

```java
public class B extends A {
}
```

- 子类能继承父类的**非私有成员**（成员变量、成员方法）；
- 子类的对象是由子类、父类共同完成的（对象中既有父类部分的数据，也有子类部分的数据）。

### 5. 权限修饰符

权限修饰符限制类中成员（成员变量、成员方法、构造器）能被访问的范围：

| 修饰符 | 本类 | 同一个包中的类 | 子孙类 | 任意类 |
| --- | --- | --- | --- | --- |
| private | √ | | | |
| 缺省 | √ | √ | | |
| protected | √ | √ | √ | |
| public | √ | √ | √ | √ |

### 6. 继承的特点

1. **单继承**：一个类只能继承一个直接父类，Java 不支持多继承；
2. **多层继承**：不支持多继承，但支持多层继承（A 继承 B，B 继承 C）；
3. **最终类**：不希望某个类被继承，可以用 final 修饰；
4. **祖宗类**：Java 中所有的类都直接或间接继承了 Object 类，Object 是所有类的祖宗。

为什么不支持多继承？反证：若 `class C extends A, B`，而 A、B 中都有同名方法 `method()`，`c.method()` 不知道该执行哪一个，语义混乱。

### 7. 方法重写（Override）

当子类觉得父类中的某个方法不好用、无法满足自己的需求时，子类可以写一个方法名、参数列表与父类一样的方法去覆盖父类方法。注意事项：

- 建议加 `@Override` 注解，让编译器校验重写格式是否正确，可读性也更好；
- 子类重写方法的访问权限必须**大于或等于**父类方法的权限（public > protected > 缺省）；
- 重写方法的返回值类型必须与被重写方法一样，或者范围更小；
- 私有方法、静态方法不能被重写。

常见应用场景：子类重写 Object 类的 `toString()` 方法，以便返回对象的内容而非地址。

### 8. 子类构造器的特点

**子类的全部构造器都会先调用父类的构造器，再执行自己的方法体。**默认情况下，子类构造器的第一行代码都是 `super()`（写不写都有），调用父类的无参构造器；如果父类没有无参构造器，必须在子类构造器第一行手写 `super(...)` 指定调用父类的有参构造器。

任意类的构造器中还可以通过 `this(...)` 调用本类的其他构造器（兄弟构造器）。注意：`this(...)`、`super(...)` 都只能放在构造器的第一行，因此二者不能同时出现。

## 二、技术点之间的联系

- static 解决"数据只有一份、大家共享"的问题；继承解决"多个类有重复代码"的问题——前者是空间上的共享，后者是代码上的复用。
- 继承建立父子关系后，方法重写让子类能个性化父类的行为，super 让子类能回头访问父类成员，权限修饰符则规定了继承中"什么能被看到"——四者共同构成继承机制的完整规则集。
- 子类构造器先调父类构造器，与"子类对象由父类、子类共同组成"互为因果：对象里有父类的数据，自然要先初始化父类部分。
- 就近原则（局部 → 子类成员 → 父类成员）贯穿成员访问；this 区分"局部与成员"，super 区分"子类与父类"。
- static 的访问限制（静态方法不能碰实例成员）与构造器规则，本质都是**加载顺序**问题：类先于对象存在，父类部分先于子类部分初始化。

## 三、方法与实现要点

### 1. 继承的典型写法

```java
public class People {
    private String name;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}

public class Teacher extends People {
    private String skill;  // 技能

    public String getSkill() {
        return skill;
    }
    public void setSkill(String skill) {
        this.skill = skill;
    }
}

public class Consultant extends People {
    private int number;  // 解答问题的总人数
    // getter/setter 略
}
```

抽取前 Teacher 和 Consultant 各自重复定义 name；抽成父类 People 后，重复代码消失。

### 2. 子类访问成员的就近原则

在子类方法中访问成员（变量、方法），依照就近原则：先子类局部范围找，再子类成员范围找，再父类成员范围找，父类还没有就报错。子父类出现重名成员时优先使用子类的；一定要在子类中访问父类的重名成员，用 `super.父类成员变量 / super.父类成员方法`。

### 3. 子类构造器调用父类构造器的应用

```java
class People {
    private String name;
    private int age;

    public People() {
    }

    public People(String name, int age) {
        this.name = name;
        this.age = age;
    }
    // get + set 方法...
}

class Teacher extends People {
    private String skill;

    public Teacher() {
    }

    public Teacher(String name, int age, String skill) {
        super(name, age);      // 先把父类部分的数据初始化
        this.skill = skill;    // 再初始化子类部分的数据
    }
    // get + set 方法...
}

// 使用
Teacher t = new Teacher("李四", 36, "Java");
```

### 4. this(...) 调用兄弟构造器

```java
public class Student {
    private String schoolName;
    private String name;

    public Student(String name){
        this(name, "默认学校");   // 调用本类的有参构造器
    }

    public Student(String name, String schoolName){
        this.name = name;
        this.schoolName = schoolName;
    }
}
```

### 5. 工具类的标准写法

```java
public class XxxUtil {
    // 构造器私有化，防止无意义地创建对象
    private XxxUtil() {
    }

    public static void doSomething() {
        ...
    }
}
```

## 四、重点与难点分析

### 重点 1：静态成员共享语义

静态变量在内存中只有一份，任何对象修改它都会影响所有访问者。`s1.name = "李四"; s2.name = "王五";` 之后 `s1.name` 也变成了"王五"——因为它俩访问的是同一块内存。共享是特性，用错就是 bug。

### 重点 2：重写的三条硬规则

权限不能更小、返回值类型不能更大、私有/静态方法不能重写。记住口诀：**重写要"平级或更开放"，不能比父类更保守**。

### 难点 1：super() 的隐式存在

子类构造器不写 super() 时编译器自动补一行 `super()`。如果父类只提供了有参构造器，子类构造器就会编译报错——因为自动补的 `super()` 找不到父类无参构造器。排查这类错误的思路：先看父类是否还有无参构造器，再决定子类首行手写 `super(...)` 还是给父类补一个无参构造器。

### 难点 2：静态方法中为什么不能出现 this

this 代表"当前对象"，而静态方法属于类、可能在没有创建任何对象时被调用，此时根本不存在"当前对象"。同理，静态方法访问实例变量也可能面临"对象还没创建"的局面。

### 难点 3：继承后的成员访问链路

就近原则 + this + super 构成完整链路：局部变量直接用 → 子类成员用 this（可省略）→ 父类成员用 super。写复杂继承代码时，脑里要有这条链路，否则容易"以为调用的是父类方法，实际执行的是子类重写后的方法"。

### 易错点清单

- 用对象访问静态成员（能编译但语义误导）；
- 父类没有无参构造器，子类构造器也不写 super(...)，编译报错；
- 重写时权限缩小（父类 public 子类缺省），编译报错；
- this(...) 和 super(...) 写在构造器非第一行，编译报错；
- 试图继承 final 修饰的类。

## 五、示例

### 示例 1：静态变量统计对象创建数量

```java
public class User {
    // 静态变量：所有对象共享一份计数
    public static int number;

    // 每创建一个对象，构造器执行一次，计数+1
    public User(){
        User.number++;
    }
}

public class Test {
    public static void main(String[] args) {
        new User();
        new User();
        new User();
        System.out.println(User.number);   // 3
    }
}
```

### 示例 2：方法重写与 @Override

```java
public class People {
    public void run() {
        System.out.println("人会跑");
    }
}

public class Student extends People {
    @Override   // 让编译器校验重写格式
    public void run() {
        System.out.println("学生跑得很快");
    }
}
```

### 示例 3：super 访问父类重名成员

```java
public class People {
    String name = "父类的name";

    public void print() {
        System.out.println("父类的print方法");
    }
}

public class Student extends People {
    String name = "子类的name";

    public void show() {
        String name = "局部的name";
        System.out.println(name);         // 局部：就近原则
        System.out.println(this.name);    // 子类成员
        System.out.println(super.name);   // 父类成员
        super.print();                    // 指定调用父类方法
    }
}
```

### 示例 4：子类构造器完整流程

见"方法与实现要点"第 3 节：`new Teacher("李四", 36, "Java")` 时，先执行 `super(name, age)` 初始化对象中父类部分的 name、age，再执行 `this.skill = skill` 初始化子类部分。

### 示例 5：继承消除重复代码

```java
// 抽取前：Consultant 和 Teacher 各自维护一份 name 及 getter/setter
// 抽取后：公共成员上升到 People，子类只保留特有成员
public class Consultant extends People {
    private int number;  // 解答问题的人数
    // 只需关心自己特有的成员
}

public class Teacher extends People {
    private String skill;  // 技能
}
```
