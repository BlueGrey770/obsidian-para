---
type: memo
date: 2026-11-06
aliases:
  - "01-jie-kou-nei-bu-lei-yu-lambda-ji-shu-yao-dian"
  - "01 接口、内部类与 Lambda 技术要点"
---

# 01 接口、内部类与 Lambda 技术要点

> 本章围绕「接口定义规则 → 函数式接口 → 匿名内部类 / Lambda 简化 → 接口多态落地」这条主线，梳理 Java 面向接口编程的完整链条。

## 一、技术概念说明

### 1. 接口（Interface）：规则的集合

接口是 Java 中定义**行为规范**的机制——它只声明「能做什么」，不关心「怎么做」。理解接口要抓住三个事实：

- **接口与类是「实现」关系**：类用 `implements` 承诺遵守接口规则；一个类可以实现**多个**接口（多对一），这是 Java 弥补单继承局限的核心手段。
- **接口与接口是「继承」关系**：接口可以 `extends` 另一个接口（如 `Mouse extends USB`），把通用规则细化成专用规则。
- **接口成员有强约束**：成员变量默认 `public static final`（常量，可 `接口名.变量名` 直接访问）；抽象方法默认 `public abstract`；接口**没有构造方法**，不能 `new`。

### 2. 函数式接口（Functional Interface）

**有且仅有一个抽象方法**的接口。它是 Lambda 存在的唯一前提——Lambda 只能创建函数式接口的实例。用 `@FunctionalInterface` 标注可让编译器校验（多一个抽象方法就报错），属于防御性编程的好习惯。

### 3. 匿名内部类：一次性的实现类

当某个接口/类的实现**只用一次**时，不值得单独写类文件，可就地用匿名内部类；本质是「没有名字的子类」，编译后生成真实的 `.class` 文件（如 `Test$1.class`）。

### 4. Lambda 表达式：匿名内部类的极致简化

目标接口是函数式接口时，匿名内部类的模板代码（`new 接口() { @Override ... }`）全部可省，只保留**参数 + 方法体**：

```java
// 匿名内部类写法
Phone p = new Phone() {
    @Override public void sendMessage(String name) {
        System.out.println("给" + name + "发送了一条信息");
    }
};
// Lambda 写法 —— 一行搞定
Phone p = name -> System.out.println("给" + name + "发送了一条信息");
```

### 5. JDK 8 接口增强：默认方法与静态方法

| 类型 | 格式 | 调用方式 | 使用场景 |
|-|-|-|-|
| 默认方法 | `[public] default 返回值 方法名(参数){...}` | 实现类对象调用，可重写 | 给接口「无痛升级」新能力，老实现类不用改 |
| 静态方法 | `[public] static 返回值 方法名(参数){...}` | **只能接口名调用** | 接口相关的工具方法 |

静态方法禁止用实现类调用，根本原因是**多实现冲突**：一个类可实现多个接口，若多个接口有同名静态方法，用实现类调用就无法确定走哪一个。

## 二、技术点之间的联系

```
接口（定义规则）
  │
  ├─ 成员约束：常量 + 抽象方法（无构造方法）
  │
  ├─ 特例：函数式接口（仅一个抽象方法）── @FunctionalInterface 校验
  │        │
  │        └─ 实例化方式的演进：
  │             实现类 → 匿名内部类 → Lambda → 方法引用
  │             （模板代码逐级减少，本质不变）
  │
  ├─ JDK8 增强：默认方法 / 静态方法（接口从「纯规则」变成「规则 + 部分实现」）
  │
  └─ 应用层：接口 + 实现类 + 多态 = 可扩展的业务架构
```

三条关键关系链：

1. **「接口 → 函数式接口 → Lambda」是递进关系**：不是所有接口都能用 Lambda，只有函数式接口才行；Lambda 不是新语法实体，只是函数式接口实例的简写。
2. **匿名内部类与 Lambda 是替代关系**：Lambda 是匿名内部类在函数式接口场景下的子集。区别有二——匿名内部类编译生成物理 `.class` 文件，Lambda 在内存中完成（`invokedynamic`），效率略高；匿名内部类可用于任意类/接口，Lambda 只认函数式接口。
3. **方法引用是 Lambda 的再简化**：当 Lambda 体只是「调用一个已存在的方法」时，可写成 `类名::方法名`（如 `Number3::number` 替代 `num -> Number3.number(num)`）。

## 三、方法与实现要点

### 1. 面向接口编程的三步法

USB 外设、商品结算、员工薪资三个案例结构完全同构，掌握模板即可举一反三：

- **第 1 步 · 定契约**：把「所有角色共有的行为」抽成接口（`USB.connect/disconnect`、`Product.calculateTotalPrice`、`SalaryCalculable.calculateSalary`）。
- **第 2 步 · 做实现**：每个具体角色实现接口、补全各自逻辑；有层级关系时先继承再实现（`MemberProduct extends NormalProduct`）。
- **第 3 步 · 面向接口使用**：声明接口类型的数组/参数（`Product[]`、`SalaryCalculable[]`），统一遍历调用——**多态在这一步生效**：同一行 `calculateTotalPrice()`，普通商品按单价 × 数量算，会员商品自动乘折扣。

### 2. 写 Lambda 的思考顺序

1. 先确认目标接口是不是函数式接口（只有一个抽象方法）；
2. 只看抽象方法签名：参数列表照搬到 `->` 左边，方法体写到右边；
3. 能省则省：参数类型可省、单参数括号可省、单行方法体的大括号和 `return` 可省；
4. 方法体若只是调用现成方法，升级为方法引用。

### 3. 匿名内部类作为返回值

方法返回类型写接口 `Inter`，方法体里 `return new Inter() { ... }`，调用方 `Outer.method().show()` 链式触发——这是工厂方法的雏形。

### 4. 把算法封装成可复用策略

`StringHandler` 这类案例（反转 / 去空格 / 首字母大写）练的是**同一接口 + 不同实现 = 不同行为**，正是策略模式的思想；后续 Stream API 的 `map/filter` 全部建立在这个思想上。

## 四、重点与难点分析

### 重点 1：接口与抽象类的四大区别

| 维度 | 接口 | 抽象类 |
|-|-|-|
| 与类的关系 | 实现（implements），可多实现 | 继承（extends），单继承 |
| 构造方法 | 没有 | 有 |
| 成员变量 | 只能是常量（public static final） | 可以有普通成员变量 |
| 设计意图 | 定义规则，约束所有实现类遵守；类可**选择**实现哪些接口 | 抽取子类共性，强制子类重写抽象方法，作用范围在继承体系内 |

**记忆锚点**：抽象类是「我是什么样的东西」（is-a，纵向血统），接口是「我能做什么事」（can-do，横向能力）。

### 重点 2：接口静态方法的调用限制

只能用 `接口名.静态方法()`，不能用实现类或对象调用。原因：多实现下同签名静态方法会产生歧义，Java 直接从语法层面掐断歧义来源。

### 难点 1：多态场景下的方法分派

`Product product = products[i]` 后调用 `product.calculateTotalPrice()`，JVM 在**运行期**根据对象实际类型（`NormalProduct` 还是 `MemberProduct`）决定执行哪个方法体——编译看左边（接口有这方法即可通过编译），运行看右边（实际对象的方法生效）。`MemberProduct` 的折扣字段 `vip` 只在子类存在，接口引用访问不到，但被重写的方法内部可以正常使用。

### 难点 2：Lambda 的边界

- 不能用于有两个抽象方法的接口（双方法的 `Phone` 必须用匿名内部类）；
- Lambda 中使用的局部变量必须是**事实 final**（不能二次赋值），因为 Lambda 捕获的是值的副本；
- `this` 在 Lambda 中指外部类对象，在匿名内部类中指匿名类自己——嵌套场景极易踩坑。

### 易错点清单

- 误以为接口可以多继承类——错，接口只能继承接口；
- 用实现类调用接口静态方法——编译报错；
- 在需要「同时重写多个方法」的场景误用 Lambda——只能退回匿名内部类；
- 忘记接口成员变量是常量，试图给它二次赋值。

## 五、示例

### 示例 1：接口继承 + 多实现（USB 设备族）

`USB` 定义通用插拔规则，`Mouse` / `Keyboard` 各自 `extends USB` 追加专属能力，实现类只需补全自己那部分。

```java
public interface USB {
    void connect();
    void disconnect();
}

public interface Mouse extends USB { void click(); }
public interface Keyboard extends USB { void input(); }

public class MouseImpl implements Mouse {
    @Override public void connect()    { System.out.println("连接上了鼠标..."); }
    @Override public void click()      { System.out.println("使用了鼠标点击桌面.."); }
    @Override public void disconnect() { System.out.println("拔出了鼠标..."); }
}

public class KeyboardImpl implements Keyboard {
    @Override public void connect()    { System.out.println("连接上了键盘...."); }
    @Override public void input()      { System.out.println("使用了键盘输入了 HelloWorld"); }
    @Override public void disconnect() { System.out.println("拔出了键盘...."); }
}
```

要点：继承来的抽象方法（`connect/disconnect`）同样必须实现，接口链上的所有规则一份都不能漏。

### 示例 2：接口 + 多态完成商品结算

`Product` 只保留「算总价」「取名称」两个能力，普通商品与会员商品各自实现，结算类只面向 `Product` 编程。

```java
public interface Product {
    double calculateTotalPrice();
    String getName();
}

public class NormalProduct implements Product {
    private String name; private double price; private int quantity;
    // 构造、getter/setter 略
    @Override public double calculateTotalPrice() { return price * quantity; }
    @Override public String getName() { return name; }
}

public class MemberProduct extends NormalProduct {
    private double vip;   // 折扣系数，如 0.9
    @Override public double calculateTotalPrice() {
        return getPrice() * getQuantity() * vip;
    }
}

public class Settlement {
    public void settleAccount(Product[] products) {
        double money = 0;
        for (Product product : products) {
            double productMoney = product.calculateTotalPrice();  // 多态分派
            System.out.println("商品名称:" + product.getName() + ",总价：" + productMoney);
            money += productMoney;
        }
        System.out.println("应付总额：" + money);
    }
}
```

要点：新增商品类型时只加实现类，`Settlement` 一行不用改——这就是「面向接口编程」带来的可扩展性。

### 示例 3：接口 + 抽象类协同（员工薪资）

接口定「能算薪资、能报信息」，抽象类收拢姓名 / 工号 / 底薪等共性字段，子类只写各自算法。

```java
public interface SalaryCalculable {
    double calculateSalary();
    String getEmployeeInfo();
}

public abstract class Employee implements SalaryCalculable {
    private String name; private String id; private double basepay;
    @Override public String getEmployeeInfo() { return name + "（" + id + "）"; }
    @Override public abstract double calculateSalary();   // 交给子类
}

public class FullTimeEmployee extends Employee {
    private double bonus;
    @Override public double calculateSalary() { return getBasepay() + bonus; }
}

public class PartTimeEmployee extends Employee {
    private double hour; private double wage;
    @Override public double calculateSalary() { return hour * wage; }
}
```

要点：**接口管「有没有这个能力」，抽象类管「共有的东西放哪」**，两者叠加才能既约束又复用。

### 示例 4：Lambda 实现字符串处理策略

`StringHandler` 只有一个抽象方法，属函数式接口，同一个接口传不同 Lambda 即得到不同行为。

```java
public interface StringHandler {
    String process(String s);
}

public class Test {
    public static void main(String[] args) {
        // 策略一：反转
        StringHandler reverse = s -> new StringBuilder(s).reverse().toString();
        // 策略二：去空格
        StringHandler trim = s -> s.replace(" ", "");
        // 策略三：单词首字母大写
        StringHandler capitalize = s -> {
            String[] words = s.split(" ");
            StringBuilder sb = new StringBuilder();
            for (String w : words) {
                sb.append(w.substring(0, 1).toUpperCase()).append(w.substring(1)).append(" ");
            }
            return sb.toString().trim();
        };
        System.out.println(reverse.process("asdfggss"));
        System.out.println(trim.process("  sa dad ggfs  "));
        System.out.println(capitalize.process("hello word"));
    }
}
```

### 示例 5：方法引用替代 Lambda

当 Lambda 体只是「把参数原样传给某个已有方法」时，直接用 `类名::方法名`。

```java
public interface NumberProcessor {
    int process(int num);
}

public class Number3 {
    public static int number(int num) { return num * num * num; }
}

// Lambda 写法：NumberProcessor p = num -> Number3.number(num);
NumberProcessor p = Number3::number;
System.out.println(p.process(10));   // 1000
```

### 示例 6：匿名内部类作为返回值

```java
public class Demo4 {
    public static void main(String[] args) {
        Outer.method().show();          // 链式调用，输出 HelloWorld
    }
}

interface Inter { void show(); }

class Outer {
    public static Inter method() {
        return new Inter() {
            @Override public void show() { System.out.println("HelloWorld"); }
        };
    }
}
```

要点：返回类型是接口、返回的是匿名单例实现——这是「工厂方法」最朴素的形态，也是回调 / 事件监听器的常见写法。

### 示例 7：接口常量的访问方式

```java
interface Inter {
    int num = 10;                  // 等价于 public static final int num = 10;
    void method();
}

public class Demo {
    public static void main(String[] args) {
        System.out.println(Inter.num);          // 不需要创建对象
        Inter inter = new Inter() {
            @Override public void method() { System.out.println("method 方法被调用"); }
        };
        inter.method();
    }
}
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 接口、内部类与 Lambda / 01 接口、内部类与 Lambda 技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/X6rawpyyfisAQHkPSONcWd8CnPK)
