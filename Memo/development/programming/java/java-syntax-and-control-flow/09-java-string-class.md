---
type: memo
date: 2026-11-06
aliases:
  - "09-string-lei-jing-jiang"
  - "Java String 类精讲"
---

# Java String 类精讲

## 一、概念说明

### 1. String 是什么

String 代表字符串，它的对象封装字符串数据，并提供了很多方法完成对字符串的处理。String 是引用数据类型，但书写体验接近基本类型。

### 2. 创建字符串对象的两种方式

方式一：直接使用双引号——Java 程序中所有字符串文字（如 `"abc"`）都是 String 类的对象：

```java
String name = "小黑";
String schoolName = "示例学校";
```

方式二：调用 String 类的构造器：

| 构造器 | 说明 |
| --- | --- |
| `public String()` | 创建一个空白字符串对象，不含任何内容 |
| `public String(String original)` | 根据传入的字符串内容创建字符串对象 |
| `public String(char[] chars)` | 根据字符数组的内容创建字符串对象 |
| `public String(byte[] bytes)` | 根据字节数组的内容创建字符串对象 |

```java
String s1 = new String();
String s2 = new String("abc");
char[] chars = {'a', 'b', 'c'};
String s3 = new String(chars);
byte[] bytes = {97, 98, 99};
String s4 = new String(bytes);   // "abc"
```

### 3. String 的常用方法

| 方法名 | 说明 |
| --- | --- |
| `public int length()` | 获取字符串的长度（字符个数） |
| `public char charAt(int index)` | 获取某个索引位置处的字符 |
| `public char[] toCharArray()` | 把字符串转换成字符数组返回 |
| `public boolean equals(Object anObject)` | 判断内容是否一样 |
| `public boolean equalsIgnoreCase(String anotherString)` | 判断内容是否一样（忽略大小写） |
| `public String substring(int beginIndex, int endIndex)` | 按开始、结束索引截取（包前不包后） |
| `public String substring(int beginIndex)` | 从索引处截取到末尾 |
| `public String replace(CharSequence target, CharSequence replacement)` | 用新值替换旧值，返回新字符串 |
| `public boolean contains(CharSequence s)` | 判断是否包含某个字符串 |
| `public boolean startsWith(String prefix)` | 判断是否以某个字符串开头 |
| `public String[] split(String regex)` | 按指定内容分割，返回字符串数组 |

## 二、技术点之间的联系

- String 是"第一个认真接触的引用类型"：它有构造器、有方法、创建的对象存堆中——面向对象的一切规则都适用，同时又有字面量直接赋值的特殊待遇。
- 字符串不可变性（String 对象内容创建后不可改）解释了为什么 `replace`、`substring` 等方法都是**返回新字符串**而不是改原字符串——所有"修改"方法都是产生新对象。
- 字符串比较必须理解 `==` 与 `equals` 的分工：`==` 比较的是地址，`equals` 比较的是内容。这为后续"引用类型比较"立下规矩。
- `charAt` + 循环可以把字符串当"字符数组"遍历；`split` 把字符串拆成数组；`toCharArray` 直接转成数组——String 与数组、循环的结合是大量文本处理题的基本功。
- 登录校验、验证码生成这两个经典案例，把 String 的方法（equals、charAt、length）与分支、循环、Random 串成完整的小程序。

## 三、方法与实现要点

### 1. 字符串比较的正确姿势

```java
String a = "abc";
String b = new String("abc");

System.out.println(a == b);         // false：比较地址
System.out.println(a.equals(b));    // true：比较内容
```

开发中判断字符串内容是否一致，**一律用 equals**，不用 `==`。

### 2. 遍历字符串

```java
String str = "hello";
for (int i = 0; i < str.length(); i++) {
    char c = str.charAt(i);
    System.out.println(c);
}
```

### 3. 截取与替换

```java
String str = "hello world";
String s1 = str.substring(0, 5);    // hello（包前不包后）
String s2 = str.substring(6);       // world
String s3 = str.replace("world", "java");  // hello java，原字符串不变
```

### 4. 分割

```java
String data = "张三,18,男";
String[] arr = data.split(",");
System.out.println(arr[0]);   // 张三
```

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

## 五、示例

### 示例 1：登录功能（String + 循环 + 分支）

需求：系统正确的登录名和密码是 `admin / 123456`，接收用户输入并判断，最多给三次机会。

```java
Scanner sc = new Scanner(System.in);
for (int i = 0; i < 3; i++) {
    System.out.println("请输入登录名：");
    String loginName = sc.next();
    System.out.println("请输入密码：");
    String passWord = sc.next();

    // 字符串内容比较必须用 equals
    if ("admin".equals(loginName) && "123456".equals(passWord)) {
        System.out.println("欢迎进入系统!");
        break;   // 登录成功，结束循环
    } else {
        System.out.println("登录名或密码错误，请重新输入");
    }
}
```

### 示例 2：随机验证码（String + Random + 循环）

需求：生成 5 位验证码，每位可能是数字、大写字母、小写字母。

```java
String code = "";
String data = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
Random r = new Random();

// 循环5次，每次得到一个字符范围内的随机索引，根据索引提取字符并连接
for (int i = 0; i < 5; i++) {
    int index = r.nextInt(data.length());
    code += data.charAt(index);
}
System.out.println("验证码：" + code);
```

### 示例 3：两种创建方式的对照

```java
// 方式一：字面量直接赋值（推荐，简洁）
String name1 = "abc";

// 方式二：构造器创建
String name2 = new String("abc");

System.out.println(name1 == name2);        // false
System.out.println(name1.equals(name2));   // true
```

### 示例 4：字符串信息拆解

```java
String info = "Java是一门面向对象的编程语言";
System.out.println(info.length());           // 长度
System.out.println(info.charAt(0));          // J
System.out.println(info.contains("面向对象")); // true
System.out.println(info.startsWith("Java")); // true
```
