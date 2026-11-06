---
type: memo
date: 2026-11-06
aliases:
  - "Java 方法详解与分支结构"
---

# Java 方法详解与分支结构

## 一、概念说明

### 1. 方法：功能的最小单元

方法是一种用于执行特定任务或操作的代码块，代表一个功能，它可以接收数据进行处理，并返回一个处理后的结果。完整定义格式：

```java
修饰符 返回值类型 方法名(形参列表){
    方法体代码(需要执行的功能代码)
    return 返回值;
}

// 示例：求两个数的最大值
public static int max(int a, int b){
    int max = a > b ? a : b;
    return max;
}
```

各部分的含义：

- **修饰符**：入门阶段固定写 `public static`；
- **返回值类型**：方法运算结果的数据类型，告知调用者会产生什么类型的结果（出参）；没有返回值就写 `void`；
- **方法名**：小驼峰命名，是调用者找到方法的依据；
- **形参列表**：本质是定义一堆变量，等着接收调用者传入的数据（入参），提高方法的扩展性和灵活性；
- **return**：两个作用——结束方法、把结果返回给调用者。

### 2. 方法的调用

方法定义后不调用就不会执行。调用格式：

```java
方法名();                    // 无参无返回值
方法名(实参);                // 有参无返回值
返回值类型 变量名 = 方法名(实参);  // 有参有返回值
```

方法的价值：抽取代码封装成方法后，只需定义一次就能重复调用，提高代码复用性；同时把大段代码拆分成独立功能，方便管理。

### 3. 方法重载

同一个类中允许出现多个名称相同的方法，只要**形参列表不同**（个数不同或类型不同，与变量名无关），就构成方法重载。重载让调用者用同一个方法名应对不同形态的数据。

### 4. 分支结构：if 与 switch

程序流程控制分三种结构：顺序结构（自上而下执行）、分支结构（根据条件选择执行）、循环结构（控制代码重复执行）。

- **if 分支**：根据条件的真或假决定执行哪段代码，有三种形式——单分支 `if`、双分支 `if-else`、多分支 `if-else if-else`。
- **switch 分支**：拿表达式的值与各个 case 后的值逐一比较，相等则执行对应代码块，是"值匹配"式的分支。

### 5. Random：随机数工具

`java.util.Random` 用于生成随机数，三步走：导包 → `new Random()` → 调用 `nextInt(n)` 得到 `0 ~ n-1` 的随机数（不含 n）。

## 二、技术点之间的联系

- 方法是代码复用的基本单位；分支和循环则是方法内部组织逻辑的两种骨架。一个实际功能 = 方法外壳 + 内部的分支/循环。
- if 与 switch 都解决"按条件选路"的问题，但判断依据不同：if 判断布尔表达式的真假（天然支持区间），switch 比较值的相等性（天然支持离散选项）。
- 方法重载依赖形参列表区分，这与"形参本质是变量"呼应——变量的个数和类型构成方法的唯一标识之一。
- 运算符是分支条件的原材料：关系运算符产出布尔值、逻辑运算符组合布尔值，最终喂给 if 或三元。
- Random 常与循环、分支搭配构成完整小程序（猜数字、验证码），是"输入处理输出"闭环中制造不确定性的工具。

## 三、方法与实现要点

### 1. 方法的其他形态与注意事项

无参无返回值的方法：

```java
// 打印3行Hello World：无参数无返回值
public static void print(){
    System.out.println("Hello World");
    System.out.println("Hello World");
    System.out.println("Hello World");
}
```

注意事项：

1. 方法不调用不执行；
2. 方法之间是平级关系，不能嵌套定义，定义的前后顺序无所谓；
3. 无返回值的方法中可以用单独的 `return;` 立即结束当前方法；
4. 返回值类型声明为 `void` 时，方法内部不能用 return 返回数据。

### 2. 方法重载实例

```java
public static void printVariable(int a) {
    System.out.println(a);
}

public static void printVariable(String str) {
    System.out.println(str);
}

public static void printVariable(int a, String str) {
    System.out.println(a);
    System.out.println(str);
}
```

三个方法同名，靠参数列表的个数/类型区分，调用时编译器自动匹配。

### 3. if 分支三种形式

```java
if (条件表达式) {
    代码;
}

if (条件表达式) {
    代码1;
} else {
    代码2;
}

if (条件表达式1) {
    代码1;
} else if (条件表达式2) {
    代码2;
} else if (条件表达式3) {
    代码3;
} else {
    代码n;
}
```

多分支的执行流程：先判断条件 1，为 true 执行语句体 1 并结束整个分支；为 false 再判断条件 2……全部不满足才执行 else。

两个注意事项：

- `if(条件)` 的小括号后**不能跟分号**，否则 `{}` 中的代码将不受 if 控制；
- if 控制的语句体只有一行时，大括号可以省略（不建议省略，易读性差）。

### 4. switch 分支

```java
switch(表达式){
    case 值1:
        执行代码...;
        break;
    case 值2:
        执行代码...;
        break;
    ...
    default:
        执行代码n;
}
```

执行流程：先算出表达式的值，再与 case 后的值匹配，匹配成功执行该 case 块，遇到 break 跳出；全部不匹配执行 default。

注意事项：

- 表达式类型只能是 byte、short、int、char，JDK5 起支持枚举，JDK7 起支持 String；**不支持 double、float、long**；
- case 给出的值不允许重复，且只能是字面量，不能是变量；
- 忘记写 break 会出现**穿透现象**——从匹配的 case 开始一路执行到 switch 结束。

### 5. switch 穿透性的合理利用

当多个 case 分支的代码相同时，可以把相同代码集中到一个 case 块，其余 case 块利用穿透性"滑落"到该块执行，从而简化代码：

```java
switch (week) {
    case "周一":
        System.out.println("埋头苦干，解决bug");
        break;
    case "周二":
    case "周三":
    case "周四":
        System.out.println("请求大牛程序员帮忙");
        break;
    case "周五":
        System.out.println("自己整理代码");
        break;
    case "周六":
    case "周日":
        System.out.println("打游戏");
        break;
}
```

### 6. Random 生成指定区间随机数

`nextInt(n)` 只能生成 `0 ~ n-1`。要得到 `65 ~ 91` 之间的随机数，用"减加法"：先减去起点变成 `0 ~ 26`，生成后再加回来：

```java
Random r = new Random();
int number = r.nextInt(27) + 65;   // 65 ~ 91
```

## 四、重点与难点分析

### 重点 1：if 与 switch 的选型

- if 在功能上远强大于 switch，**条件是区间判断时用 if**；
- 条件是与一个一个具体值比较时用 switch——格式良好、性能较好、代码优雅。

### 重点 2：switch 的穿透性

穿透既是坑也是特性。常规用途必须写 break；当多个分支逻辑相同，穿透反而成为简化代码的手段。关键是有意识地使用，而不是遗忘 break。

### 难点 1：多分支条件的顺序

`if-else if` 链从上到下短路执行，区间条件的书写顺序必须覆盖完整且不重叠。例如按成绩分段奖励时，条件要按从高到低（或从低到高）排列，否则会出现"低分段条件先命中，高分段永远进不去"的逻辑错误。另外注意边界条件的严谨性——`score < 0 && score > 100` 这种条件永远为 false（笔误，应为 `||`）。

### 难点 2：重载的判定标准

重载只看方法名 + 形参列表（个数、类型、顺序），**与返回值类型、参数变量名无关**。仅返回值不同的两个同名方法不构成重载，编译直接报错。

### 易错点清单

- `if(条件);` 多分号导致条件失效；
- switch 表达式用了 long/double/float，编译报错；
- case 后跟变量或重复值，编译报错；
- 以为方法按书写顺序执行——方法只有被调用才执行；
- `nextInt(10)` 以为能生成 10——实际范围是 0~9。

## 五、示例

### 示例 1：有返回值方法 vs 无返回值方法

```java
// 定义方法求两个数的和，把结果返回给调用者
public static int getSum(int a, int b) {
    return a + b;
}

// 定义方法打印两个数的和，不需要返回数据
public static void printSum(int a, int b) {
    System.out.println(a + b);
}
```

定义方法时主要考虑两方面：是否需要接收数据（定形参），是否需要返回数据（定返回值类型）。

### 示例 2：BMI 与基础代谢率计算（方法抽取）

```java
public static void main(String[] args) {
    double weight = 70, height = 1.75;
    int sex = 1, age = 25;
    // 调用方法求出bmi
    double bmi = getBMI(weight, height);
    // 调用方法求出bmr
    double bmr = getBMR(weight, height, sex, age);
}

// 定义方法求出bmi值
public static double getBMI(double weight, double height) {
    return weight / (height * height);
}

// 定义方法求出bmr的值
public static double getBMR(double weight, double height, int sex, int age) {
    // 按性别选择不同公式计算基础代谢率
    ...
}
```

### 示例 3：成绩奖励（if 多分支）

```java
private static void scoreSelect() {
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入您的成绩：");
    int score = sc.nextInt();
    if (score < 0 || score > 100) {
        System.out.println("请输入0~100范围内的正确分数！");
    } else if (score >= 95) {
        System.out.println("恭喜您，奖励一辆自行车");
    } else if (score >= 90) {
        System.out.println("表现不错，奖励去游乐园一次");
    } else if (score >= 80) {
        System.out.println("请继续努力，奖励变形金刚一个");
    } else {
        System.out.println("要继续加油哦！");
    }
}
```

### 示例 4：星期安排（switch + 穿透简化）

见上文"switch 穿透性的合理利用"，多个 case 共享同一段代码。

### 示例 5：生成随机数

```java
import java.util.Random;

public class Test {
    public static void main(String[] args) {
        Random r = new Random();
        int number = r.nextInt(10);        // 0 ~ 9，不包含10
        System.out.println("随机生成了：" + number);

        int n2 = r.nextInt(10) + 1;        // 1 ~ 10：先减后加
    }
}
```
