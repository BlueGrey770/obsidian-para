---
type: memo
date: 2026-11-06
aliases:
  - "07-fang-fa-xiang-jie-yu-fen-zhi-jie-gou"
  - "Java 方法详解与分支结构"
---

# Java 方法详解与分支结构

> **定位**：只记跨知识点的联系、重点难点与易错点，不重复教程正文；完整讲解与案例见下方教程，题目练习见对应问答。

## 二、技术点之间的联系

- 方法是代码复用的基本单位；分支和循环则是方法内部组织逻辑的两种骨架。一个实际功能 = 方法外壳 + 内部的分支/循环。
- if 与 switch 都解决"按条件选路"的问题，但判断依据不同：if 判断布尔表达式的真假（天然支持区间），switch 比较值的相等性（天然支持离散选项）。
- 方法重载依赖形参列表区分，这与"形参本质是变量"呼应——变量的个数和类型构成方法的唯一标识之一。
- 运算符是分支条件的原材料：关系运算符产出布尔值、逻辑运算符组合布尔值，最终喂给 if 或三元。
- Random 常与循环、分支搭配构成完整小程序（猜数字、验证码），是"输入处理输出"闭环中制造不确定性的工具。

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

## 相关

- 教程：[[Memo/development/programming/java/java-basics/02-java-syntax-basics.md|Java 基础语法]]
- 教程：[[Memo/development/programming/java/java-basics/03-program-flow-control.md|程序流程控制]]
- 自测：[[Memo/development/programming/java/java-basics/02-java-syntax-basics-quiz.md|02 基础语法·问答]]
- 自测：[[Memo/development/programming/java/java-basics/03-program-flow-control-quiz.md|03 流程控制·问答]]
