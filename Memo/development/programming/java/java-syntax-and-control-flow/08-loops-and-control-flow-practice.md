---
type: memo
date: 2026-11-06
aliases:
  - "08-xun-huan-jie-gou-yu-liu-cheng-kong-zhi-shi-zhan"
  - "Java 循环结构与流程控制综合实战"
---

# Java 循环结构与流程控制综合实战

## 一、概念说明

循环结构控制某段代码重复执行，是流程控制的第三种结构（顺序、分支、循环）。Java 提供三种循环：for、while、do-while，外加 break、continue 两个循环控制语句。

### 1. for 循环

```java
for (初始化语句; 循环条件; 迭代语句) {
    循环体语句(重复执行的代码);
}

// 输出3次HelloWorld
for (int i = 0; i < 3; i++) {
    System.out.println("Hello World");
}
```

执行流程：初始化语句只执行一次 → 判断循环条件，false 则循环结束 → true 则执行循环体 → 执行迭代语句 → 再判断条件……直到条件为 false。

### 2. while 循环

```java
初始化语句;
while (循环条件) {
    循环体语句;
    迭代语句;
}
```

执行流程与 for 完全一样，只是三个组成部分的位置不同。

### 3. do-while 循环

```java
初始化语句;
do {
    循环体语句;
    迭代语句;
} while (循环条件);
```

特点：**先执行后判断**，循环体至少执行一次。

### 4. 死循环

没有结束条件、没有干预就不会停下来的循环：

```java
for ( ; ; ) { ... }        // 经典写法之一
while (true) { ... }       // 推荐写法
do { ... } while (true);
```

### 5. break 与 continue

- `break`：跳出并结束当前所在循环的执行（也可结束所在 switch 分支）；
- `continue`：跳出当前循环的当次执行，直接进入下一次循环。

## 二、技术点之间的联系

- for 和 while 在功能上完全等价——for 能解决的 while 也能解决，反之亦然；两者都是"先判断后执行"，do-while 是唯一的"先执行后判断"。
- 使用规范：**知道循环次数用 for，不知道循环几次用 while**。复利翻倍、折纸超珠峰这类"循环到某个条件达成"的问题，次数事先不可知，是 while 的典型场景。
- 循环三要素（初始化、条件、迭代）缺一就可能失控：条件恒真就是死循环，迭代语句遗漏同样死循环。
- break/continue 是循环的"中途干预手段"：break 彻底终止，continue 只跳过当次。猜数字游戏靠 `while(true) + break` 实现"猜到为止"。
- 循环内嵌分支（for + if）是最常见的组合：循环负责"遍历所有候选"，分支负责"筛选符合条件的"，求和、计数、筛选类问题都是这个套路。

## 三、方法与实现要点

### 1. for 与 while 的细节区别

除了写法位置不同，还有一个实际差别：**for 循环中声明的控制变量只在循环内有效，循环结束后消失；while 的控制变量定义在循环外，循环结束后还能继续使用**。

```java
int i = 0;
while (i < 3) {
    System.out.println("Hello World");
    i++;
}
System.out.println(i);   // 可以访问，i = 3

for (int j = 0; j < 3; j++) {
    System.out.println("Hello World");
}
// System.out.println(j);   // 报错：j 已超出作用域
```

### 2. 求和类问题的固定套路

循环外定义累加变量，循环内逐个累加，循环结束输出：

```java
int sum = 0;
for (int i = 1; i <= 5; i++) {
    sum += i;
}
System.out.println(sum);   // 15
```

### 3. 筛选类问题的固定套路

```java
// 求 1-10 的奇数和，方式一：全量遍历 + if 筛选
for (int i = 1; i <= 10; i++) {
    if (i % 2 == 1) {
        // i = 1 3 5 7 9
    }
}

// 方式二：让步进语句直接产生目标数据，更高效
for (int i = 1; i <= 10; i += 2) {
    // 直接产生 1 3 5 7 9
}
```

### 4. 计数类问题的固定套路

在循环外定义 `count` 变量，循环内每命中一次目标就 `count++`，循环结束后 count 即为个数。

### 5. while 应对"次数未知"

复利问题：投资 100000 元，年利率 1.7%，问多少年后本金翻倍？

```java
double principal = 1000000;   // 初始本金
double rate = 0.017;          // 年利率
double target = 2 * principal; // 目标金额：本金翻倍
int years = 0;                 // 已经过的年数

while (principal < target) {
    principal += principal * rate;  // 应用复利公式
    years++;
}
System.out.println("需要的年数：" + years);
```

### 6. 三种死循环写法的选择

推荐 `while(true)`：语义最直接。死循环并非错误，服务器程序、游戏主循环、等待用户反复输入的场景都需要它，关键是循环体内要有明确的 break 出口。

## 四、重点与难点分析

### 重点 1：循环执行流程的精确推演

面试与笔试常要求口述 `for (int i = 0; i < 3; i++)` 的逐步过程：i=0 判断通过执行循环体后 i++；i=1 同理；i=2 同理；i=3 判断失败循环结束。能逐轮写出变量值，是排查循环 bug 的基本功。

### 重点 2：for/while/do-while 的选型

| 场景 | 选择 |
| --- | --- |
| 明确知道循环次数 | for |
| 次数未知、按条件终止 | while |
| 循环体至少要执行一次（如先操作再校验） | do-while |

### 难点 1：循环变量的作用域

for 的循环变量出了循环就失效，想保留最终值必须把变量声明提前到循环外（while 风格）。嵌套循环中内外层变量名不能冲突，且要清楚每一层的迭代节奏——外层走一步，内层走一圈。

### 难点 2：break 与 continue 的边界

- break 只能结束**所在的那一层**循环，嵌套循环中想跳出外层，需要配合标签或改变外层条件；
- continue 跳过的是"当次"不是"全部"，跳过当次后迭代语句照常执行（while 中若迭代语句在 continue 之后，会造成死循环）。

### 难点 3：迭代语句遗漏导致死循环

while 循环忘记写 `i++`，条件永远为真。排查死循环的第一反应：检查条件里的变量是否在循环体内被正确更新。

### 易错点清单

- `while (true)` 里没有 break 出口，程序卡死；
- for 的分号误写成逗号；
- 循环外忘记定义累加/计数变量，循环内反复声明导致结果不对；
- `if` 与循环条件中把 `=` 误写成判断，逻辑彻底跑偏。

## 五、示例

### 示例 1：水仙花数（for + if 筛选 + 计数）

需求：输出所有三位数中的"水仙花数"（个位、十位、百位的立方和等于原数），并统计个数。

```java
int count = 0;
for (int i = 100; i <= 999; i++) {
    // 提取个位、十位、百位
    int ge = i % 10;
    int shi = i / 10 % 10;
    int bai = i / 100;
    // 立方和等于原数则是水仙花数
    if (ge * ge * ge + shi * shi * shi + bai * bai * bai == i) {
        System.out.println(i);
        count++;
    }
}
System.out.println("水仙花数个数：" + count);
```

### 示例 2：折纸超珠峰（while，次数未知）

需求：珠峰高 8848.86 米（8848860 毫米），纸张厚 0.1 毫米，折叠多少次能超过珠峰高度？

```java
double peakHeight = 8848860;   // 山峰高度
double paperThickness = 0.1;   // 纸张厚度
int count = 0;

while (paperThickness < peakHeight) {
    paperThickness *= 2;   // 每折一次厚度翻倍
    count++;
}
System.out.println("折叠次数：" + count);
```

### 示例 3：猜数字游戏（死循环 + break）

需求：随机生成 1-100 的数字，反复提示用户猜测，猜大提示过大、猜小提示过小，直到猜中结束。

```java
// 1.随机1-100之间的数字，采用Random解决
Random random = new Random();
int randomNumber = random.nextInt(100) + 1;

Scanner sc = new Scanner(System.in);
// 2.不断的键盘录入要猜的数字
while (true) {
    System.out.println("请输入您猜的数字：");
    int guessNumber = sc.nextInt();
    // 3.对比猜的数字和随机数字的关系
    if (guessNumber > randomNumber) {
        System.out.println("猜大了");
    } else if (guessNumber < randomNumber) {
        System.out.println("猜小了");
    } else {
        System.out.println("恭喜您猜中了！");
        break;   // 猜中后跳出死循环
    }
}
```

### 示例 4：鸡兔同笼（for 穷举）

需求：笼子里有 35 个头、94 只脚，求鸡和兔各多少只。

```java
// 循环表示鸡的数量 0~35只
for (int chicken = 0; chicken <= 35; chicken++) {
    // 动态算出兔子的数量
    int rabbit = 35 - chicken;
    // 算出脚的数量是否正确
    if (chicken * 2 + rabbit * 4 == 94) {
        System.out.println("鸡：" + chicken + "，兔：" + rabbit);
    }
}
```

### 示例 5：验证码生成（for + switch + Random 综合）

```java
String code = "";
String data = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
Random r = new Random();
// 循环5次，每次随机一个索引，从字符池中取出字符拼接
for (int i = 0; i < 5; i++) {
    int index = r.nextInt(data.length());
    code += data.charAt(index);
}
System.out.println("验证码：" + code);
```

### 示例 6：三种死循环写法对照

```java
for ( ; ; ) {
    System.out.println("Hello World1");
}
while (true) {                    // 推荐写法
    System.out.println("Hello World2");
}
do {
    System.out.println("Hello World3");
} while (true);
```
