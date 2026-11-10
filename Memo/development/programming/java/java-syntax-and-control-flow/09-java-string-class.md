---
type: memo
date: 2026-11-06
aliases:
  - "09-string-lei-jing-jiang"
  - "Java String 类精讲"
---

# Java String 类精讲

> **定位**：只记跨知识点的联系、重点难点与易错点，不重复教程正文；完整讲解与案例见下方教程，题目练习见对应问答。

## 二、技术点之间的联系

- String 是"第一个认真接触的引用类型"：它有构造器、有方法、创建的对象存堆中——面向对象的一切规则都适用，同时又有字面量直接赋值的特殊待遇。
- 字符串不可变性（String 对象内容创建后不可改）解释了为什么 `replace`、`substring` 等方法都是**返回新字符串**而不是改原字符串——所有"修改"方法都是产生新对象。
- 字符串比较必须理解 `==` 与 `equals` 的分工：`==` 比较的是地址，`equals` 比较的是内容。这为后续"引用类型比较"立下规矩。
- `charAt` + 循环可以把字符串当"字符数组"遍历；`split` 把字符串拆成数组；`toCharArray` 直接转成数组——String 与数组、循环的结合是大量文本处理题的基本功。
- 登录校验、验证码生成这两个经典案例，把 String 的方法（equals、charAt、length）与分支、循环、Random 串成完整的小程序。

## 四、重点与难点分析

### 重点 1：equals 而非 ==

这是 String 最高频的坑。字面量赋值的字符串可能指向同一对象（地址相同，`==` 偶尔"碰巧"返回 true），而 `new` 出来的字符串一定是新对象。一旦用 `==` 判断内容，程序就会出现"有时对有时错"的诡异现象。

### 重点 2：substring 的包前不包后

`substring(0, 5)` 取到的是索引 0~4 的字符，不包含索引 5。记错边界会导致截取的字符串多一个或少一个字符。

### 难点 1：字符串不可变带来的性能意识

`String code = ""; code += c;` 这类循环拼接每次都产生新对象，效率低。入门阶段可以接受，但要建立意识：大量拼接应该用可变字符串（StringBuilder）。

### 难点 2：next() 与 equals 的配合

键盘录入得到的字符串与字面量比较时，必须用 equals：

```java
Scanner sc = new Scanner(System.in);
String input = sc.next();
// if (input == "admin") ...   // 错误：录入的字符串是新对象，地址不同
if ("admin".equals(input)) {   // 正确；字面量写前面还能防空指针
}
```

### 易错点清单

- 用 `==` 比较字符串内容；
- `charAt` 索引越界（0 ~ length()-1）；
- 以为 `replace` 会改原字符串；
- `split` 的参数是正则表达式，分割 `"."` 等特殊字符要转义。

## 相关

- 教程：[[Memo/development/programming/java/java-basics/08-abstract-final-singleton-enum-string.md|抽象类、final、单例、枚举与 String]]
- 自测：[[Memo/development/programming/java/java-basics/08-abstract-final-singleton-enum-string-quiz.md|08 抽象类与 String·问答]]
