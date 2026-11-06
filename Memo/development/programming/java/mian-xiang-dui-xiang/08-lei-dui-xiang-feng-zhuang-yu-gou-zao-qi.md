---
type: memo
date: 2026-11-06
aliases:
  - "Java 类、对象、封装与构造器精讲"
---

# Java 类、对象、封装与构造器精讲

## 一、概念说明

### 1. 对象与类

对象是一种特殊的数据结构，用来记住一个事物的数据，从而代表该事物。变量、数组能存的数据形态有限，现实事物（一个学生、一部电影）包含多种维度的数据，对象正是为解决复杂数据的存储问题而生的。

类是对象的设计图（模板），描述了一类事物"有什么数据、能做什么处理"。获取对象的标准路径：

1. 先定义类（描述对象）；
2. 用 `new` 关键字根据类创建对象：`类名 对象名 =  new 类名();`——每 new 一次就得到一个新对象；
3. 使用对象：`对象名.属性名 = 值` 存数据，`对象名.方法名(...)` 调行为。

```java
public class Star {
    String name;
    int age;
    double height;
    double weight;
}

Star s1 = new Star();
s1.name = "赵丽颖";
s1.age = 36;
```

### 2. 类的三大成分

```java
public class 类名 {
    // 1、成员变量：说明对象可以处理什么数据（属性）
    // 2、成员方法：描述对象有什么功能（行为），比入门阶段的方法少了 static
    // 3、构造器：创建对象时完成初始化
}
```

区分两组概念：

- 成员变量：定义在类中、方法外的变量；局部变量：定义在方法中或参数列表中的变量；
- 类泛指一类群体（数据不具体），对象是类产生的具体存在（数据是具体的）。

### 3. 构造器

构造器是创建对象时被调用的特殊方法，常见应用场景是**创建对象的同时完成成员变量的初始化赋值**。

```java
public class Student {
    // 无参构造器
    public Student() {
    }

    // 有参构造器
    public Student(String name, double score) {
        this.name = name;
        this.score = score;
    }
}
```

构造器的规则：

1. 构造器名必须和类名一致，大小写都要一致；
2. 没有返回值类型，连 `void` 都不能写；
3. 如果不提供任何构造器，系统赠送一个无参构造器；
4. **一旦提供了任何构造器，系统不再赠送无参构造器**——还想用无参的，必须自己手写一个；
5. 构造器可以重载（方法名相同、参数列表不同）。

### 4. this 关键字

`this` 是一个变量，用在方法中代表**正在调用方法的当前对象**——哪个对象调用方法，this 就指向哪个对象。主要应用场景：解决成员变量与方法内变量重名导致的访问冲突。

```java
public class Student {
    String name;
    double score;

    public void checkPass(double score){
        // this.score 是成员变量，score 是局部变量（形参）
        if (this.score >= score) {
            System.out.println("恭喜您，考上哈佛，走向巅峰~~~");
        } else {
            System.out.println("不好意思，您没有考上~~~");
        }
    }
}
```

变量访问遵循**就近原则**：默认先找局部变量，非要访问成员变量时用 `this.` 区分。

### 5. 封装

封装是面向对象三大特征（封装、继承、多态）之一。用类设计对象处理某个事物的数据时，应该把**要处理的数据、以及处理这些数据的方法，设计到同一个对象中去**。

封装的设计规范是**合理隐藏、合理暴露**：公开成员用 `public` 修饰，隐藏成员用 `private` 修饰。`private` 修饰的成员只能在本类中使用，外界不能访问。典型做法：私有化成员变量，为每个成员变量提供公开的 setter/getter 方法间接访问——方法里可以编写校验逻辑屏蔽非法数据，提高安全性。

### 6. JavaBean（实体类）

实体类是一种特殊的类，满足两条要求：

1. 成员变量全部私有，并提供 public 修饰的 getter/setter 方法；
2. 必须提供无参数构造器，有参数构造器可选。

```java
public class Person {
    private String name;
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```

实体类的应用场景：实体类对象只负责数据存取，对数据的处理交给其他类的对象完成，实现**数据和数据业务处理相分离**——这是目前软件开发中流行的分层设计基础。

## 二、技术点之间的联系

- 类是模板、对象是实例、构造器是"实例化时的初始化入口"——三者构成 `定义类 → new 对象（走构造器）→ 使用对象` 的完整链路。
- this 与构造器配合最紧密：有参构造器几乎必然写成 `this.name = name;`，因为形参习惯与成员变量同名，this 是区分两者的唯一手段。
- 封装是"为什么成员变量要 private"的答案；JavaBean 是封装规范的具体落地形态；getter/setter 是封装之后对外保留的可控通道。
- static 关键字（见进阶篇）解释了成员方法"少一个 static"的原因：实例成员属于对象，必须创建对象后才有内存空间。
- 对象在内存中的形态：`new Student()` 在堆中开辟空间，成员变量有默认值（String 为 null、int 为 0、double 为 0.0），对象变量里存的是堆中对象的地址——理解这一点才能解释"两个引用指向同一对象时改一个影响另一个"。

## 三、方法与实现要点

### 1. 面向对象编程的心法

万物皆对象，谁的数据谁存储。学生数据交给学生对象存、电影数据交给电影对象存、汽车数据交给汽车对象存。分析需求时先问"这个系统里有几类事物"，每类事物对应一个类。

### 2. 定义一个标准类的检查清单

1. 类用 public 修饰；
2. 成员变量用 private 修饰，并提供 setter/getter 方法；
3. 提供无参构造器；如有需要再提供有参构造器；
4. setter 中编写数据校验逻辑（如年龄不能为负）。

### 3. 构造器的典型用法

```java
// 无参创建，再逐个 set
Student s1 = new Student();
s1.setName("小明");

// 有参创建，初始化一步到位
Student s2 = new Student("小明", 99.5);
```

### 4. this 的验证

```java
public class Student {
    public void printThis(){
        System.out.println(this);
    }
}

public class Test {
    public static void main(String[] args) {
        Student s1 = new Student();
        System.out.println(s1);   // 输出 s1 的地址
        s1.printThis();           // 输出相同的地址——this 就是当前对象

        Student s2 = new Student();
        System.out.println(s2);
        s2.printThis();           // 输出 s2 的地址
    }
}
```

### 5. 封装的实战形态

```java
public class Student {
    private int age;

    // 对age设定一个存的方法和一个取的方法
    public void setAge(int age) {
        if (age >= 0 && age <= 150) {
            this.age = age;
        } else {
            System.out.println("年龄数据非法");
        }
    }

    public int getAge() {
        return age;
    }
}
```

外部只能通过 `stu.setAge(10)`、`stu.getAge()` 间接访问，非法数据在 setter 中被拦截。

## 四、重点与难点分析

### 重点 1：构造器的"赠送规则"

类默认自带无参构造器；一旦写了任何构造器（哪怕是有参的），默认无参构造器立即消失。这是新人最常踩的编译错误来源："以前 `new Student()` 好好的，加了个有参构造器怎么就报错了？"规范做法：写了有参构造器，随手把无参构造器也补上。

### 重点 2：成员变量与局部变量的区别

| 维度 | 成员变量 | 局部变量 |
| --- | --- | --- |
| 定义位置 | 类中、方法外 | 方法中、参数列表中 |
| 默认值 | 有（null/0/0.0/false） | 没有，必须先赋值才能用 |
| 生命周期 | 随对象存在 | 随方法调用存在 |
| 冲突时 | 需要 this 显式访问 | 就近优先被访问 |

### 难点 1：对象的内存机制

`Student s1 = new Student();` 拆成两步理解：堆里创建对象（成员变量按默认值初始化），栈里的变量 s1 保存对象地址。方法被调用时，通过地址找到对象，再把数据写进对象空间。理解"变量存的是地址"，才能理解引用传递、对象共享、以及为什么方法里改对象属性会影响外部。

### 难点 2：数据与业务分离的意识

入门阶段容易把所有代码塞进一个类。实体类（JavaBean）模式的要点是**克制**：实体类只存数据，打印成绩、判断等级这类行为交给专门的操作类（如 `StudentOperator`）。这是后续三层架构（controller/service/dao）的思想雏形。

### 易错点清单

- 构造器误写返回值类型（`public void Student(){}`），编译器会把它当普通方法；
- private 成员在外部直接访问，编译报错；
- 局部变量未初始化就使用；
- 成员方法与成员变量同名冲突时忘记用 this 区分。

## 五、示例

### 示例 1：学生类的完整定义与使用

```java
public class Student {
    String name;      // 姓名
    double chinese;   // 语文成绩
    double math;      // 数学成绩

    public void printTotalScore() {
        System.out.println(name + "的各科总分是:" + (chinese + math));
    }

    public void printAverageScore() {
        System.out.println(name + "的各科平均分是:" + (chinese + math) / 2.0);
    }
}

public class Test {
    public static void main(String[] args) {
        Student s1 = new Student();
        s1.name = "小丽";
        s1.chinese = 100;
        s1.math = 100;
        s1.printTotalScore();
        s1.printAverageScore();

        Student s2 = new Student();
        s2.name = "小明";
        s2.chinese = 59;
        s2.math = 99;
        s2.printTotalScore();
        s2.printAverageScore();
    }
}
```

### 示例 2：手机类——类的最小骨架

```java
public class Phone {
    // 数据(属性) --> 成员变量
    String brand;

    // 功能(行为) --> 成员方法
    public void call(){
        System.out.println("打电话");
    }
}

// 使用
Phone p = new Phone();   // 根据类创建对象
p.brand = "华为";        // 使用手机的属性
p.call();                // 使用手机的行为
```

### 示例 3：构造器初始化与重载

```java
public class Student {
    private String name;
    private double score;

    // 无参构造器：手写出来，防止系统不再赠送
    public Student() {
    }

    // 有参构造器：创建对象的同时完成赋值
    public Student(String name, double score) {
        this.name = name;
        this.score = score;
    }
}
```

### 示例 4：简易电影信息展示系统（实体类 + 业务类分离）

需求：展示全部电影信息；按编号查询某部电影详情。

```java
// 实体类：只负责存数据
public class Movie {
    private int id;      // 编号
    private String name; // 名称
    private double price;// 价格
    // 无参/有参构造器 + getter/setter 略
}

// 业务类：负责数据的处理
public class Cinema {
    private Movie[] movies;

    public void showAllMovies() {
        // 遍历电影数组，展示每部电影
        for (int i = 0; i < movies.length; i++) {
            ...
        }
    }

    public void queryById(int id) {
        // 遍历电影的数组，找到每部电影，取出其编号和要查询的编号进行对比
        int count = 0;
        for (int i = 0; i < movies.length; i++) {
            ...
        }
        // 循环结束后查看标记，判断是否存在该编号
        if (count == 0) {
            System.out.println("没有查询到该电影");
        }
    }
}
```
