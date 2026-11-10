---
type: memo
date: 2026-11-06
aliases:
  - "10-duo-tai-chou-xiang-lei-final-dan-li-mei-ju"
  - "Java 多态、抽象类、final、单例与枚举精讲"
---

# Java 多态、抽象类、final、单例与枚举精讲

> **定位**：只记跨知识点的联系、重点难点与易错点，不重复教程正文；完整讲解与案例见下方教程，题目练习见对应问答。

## 二、技术点之间的联系

- 继承是多态的前提，方法重写是多态的开关：没有重写，父类引用调用的还是父类行为，多态失去意义；抽象类/接口则是把"必须重写"用语法强制下来，是多态的强化形态。
- 多态的"编译看左边、运行看右边"与类型转换互为补充：向上转型获得解耦，向下转型（配 instanceof）恢复子类特性。
- final 是"继承与重写的刹车"：final 类不能被继承、final 方法不能被重写、final 变量不能被改——它与继承/多态构成一张一弛的关系。
- 单例模式是 static（静态变量持唯一对象）与封装（私有构造器）的组合应用；枚举则是语法层面的"限定实例集合"，可视为单例思想的推广（多例且封闭）。
- 常量（static final）替代散落在代码各处的魔法值，与枚举替代"伪选项常量"是同一治理思路的两个粒度。

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

## 相关

- 教程：[[Memo/development/programming/java/java-basics/07-polymorphism.md|多态]]
- 教程：[[Memo/development/programming/java/java-basics/08-abstract-final-singleton-enum-string.md|抽象类、final、单例、枚举与 String]]
- 自测：[[Memo/development/programming/java/java-basics/07-polymorphism-quiz.md|07 多态·问答]]
- 自测：[[Memo/development/programming/java/java-basics/08-abstract-final-singleton-enum-string-quiz.md|08 抽象类与 String·问答]]
