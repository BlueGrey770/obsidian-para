---
type: memo
date: 2026-11-09
aliases:
  - "方法与类型转换与运算符"
---

# Java 基础语法

## 方法

<callout emoji="🌷">
**概念: 方法是完成特定功能代码块, 可以简单将方法理解为存储处理数据代码的容器**
</callout>

### 理解方法

一个完整的Java程序是一套复杂的系统，是同各种不同的功能组合而成的，而这些不同的功能就称为一个个方法，我们编写的代码需要放在各个方法中。

我们用汽车来打个比方，以便更好地理解Java中的方法。

假设你有一辆多功能车，这辆车可以完成很多任务，比如开车、停车、加油、打开空调等。每个功能就像是一个独立的操作。在编程中，我们可以把每个操作看作是一个“方法”。方法就是一段被封装起来的代码，用来执行特定的任务或计算，并且可以被多次调用而不需要重复编写相同的代码。

- 开车：想象成一个名为`startDriving`的方法，当你需要启动车辆并开始行驶时，你就调用这个方法。
- 停车：可以视为`stopDriving`方法，当到达目的地后，你会调用它让车辆停下来。
- 加油：可以是`refuel`方法，当油箱快空了的时候，你需要通过调用这个方法给汽车加满油。
- 打开空调：这可能是一个叫`turnOnAC`的方法，天气热时，你可以通过它来开启空调系统。

### main方法

java中有一个特殊的方法，就是名字为main的方法，那它有什么用呢？

现在，考虑一下当你第一次坐进汽车里准备出发的情景。在做任何其他事情之前，你必须先发动引擎。在Java程序中，`main`方法就像发动引擎一样，它是程序启动时第一个被执行的方法。没有`main`方法，你的Java应用程序就无法运行。

- 发动引擎（`main`方法）：在你能够开车（调用`startDriving`）、加油（调用`refuel`）或其他任何操作之前，首先需要做的事情是发动引擎。同样地，在Java程序中，无论你想执行什么功能，都必须从`main`方法开始。`main`方法是整个程序的入口点，一旦程序启动，JVM（Java虚拟机）就会自动寻找并运行`main`方法内的代码。

总结来说，`main`方法好比是你每次上车后的第一步——发动引擎；而其他方法则代表了车辆的各种具体功能。通过这种方式组织代码，可以让程序逻辑更加清晰易懂，同时提高代码的复用性和可维护性。

![ItYGbdjekoTd8mxZlmzcC3i5nNb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ItYGbdjekoTd8mxZlmzcC3i5nNb.png)

### 方法的语法

#### 方法基本语法

在Java中，方法（或称为函数）的基本语法包括了方法的返回类型、方法名、参数列表以及方法体。下面是一个典型的Java方法定义的基本语法：

```Java
[修饰符] 返回值类型 方法名(参数类型1 参数名1, 参数类型2 参数名2, ...) {
    // 方法体 - 实现具体功能的代码
    [return 表达式;]  // 如果方法有返回值，则需要使用 return 语句
}
```

组成部分说明

- 修饰符：可选，用来指定方法的访问级别和其他特性。常用的修饰符包括 `public`（公开的）、`private`（私有的）、`protected`（受保护的）和默认不写时的包级可见性，修饰符在本节先不作详细讲解，在后面讲解面向对象时会专门讲解。
- 返回值类型：指定了方法执行后返回的数据类型。如果方法不需要返回任何值，则应声明为 `void`。比如：我们编写一个方法计算2个数的和，需要在方法中进行计算，然后返回和，则方法需要有返回类型。如果方法只是在控制台输出一些内容，则不需要返回数据。
- 方法名：给方法起的名字，应该具有描述性，以便于理解该方法的作用。
- 参数列表：括号内的变量列表，用于传递给方法的信息。每个参数由其数据类型和名称组成，并且多个参数之间用逗号分隔。如果没有参数，也要保留一对空括号。
- 方法体：大括号 `{}` 内的代码块，包含了实现特定任务的具体逻辑。
- return 语句：如果方法定义了一个非 `void` 的返回类型，那么必须有一个 `return` 语句来返回一个相应类型的值。对于 `void` 类型的方法，`return` 语句是可选的，通常用于提前结束方法。

#### 方法示例

- 无参数无返回值

```Java
public void sayHello() {
    System.out.println("Hello!");
}
```

- 带参数无返回值

```Java
public void greet(String name) {
    System.out.println("Hello, " + name + "!");
}
```

- 带参数有返回值

```Java
public int addNumbers(int a, int b) {
    return a + b;
}
```

- 无参数有返回值

```Java
public String getGreeting() {
    return "Good morning!";
}
```

#### 方法的调用

方法编写完成后，可以在任意地方进行调用，以执行其中的功能。

在Java中，调用一个方法的基本语法相对简单。你需要知道方法的名字以及它需要哪些参数（如果有的话）。方法调用通常包括方法名加上一对圆括号`()`，如果方法有参数，则这些参数会被放在圆括号内，并且多个参数之间用逗号分隔。下面通过示例演示方法的调用语法。

假设我们编写了以下3个方法：

```Java
public class MethodDemo {
    // 无参数无返回值的方法
    public static void sayHello() {
        System.out.println("Hello from Calculator!");
    }

    // 带参数无返回值的方法
    public static void greet(String name) {
        System.out.println("Hello, " + name + "!");
    }

    // 带参数有返回值的方法
    public static int addNumbers(int a, int b) {
        return a + b;
    }
}
```

以上方法，调用的示例如下：

- 调用无参数无返回值的方法

```Java
sayHello();
```

- 调用带参数无返回值的方法

```Java
greet("Alice");
```

- 调用带参数有返回值的方法

```Java
int sum = addNumbers(5, 3);
System.out.println(sum);
```

## 方法的注意事项

### 方法重载

一个类中，出现多个方法的名称相同，但是它们的形参列表是不同的，那么这些方法就称为方法重载。

方法重载的主要作用是：

1. 提供更好的可读性和易用性：通过使用相同的方法名处理不同的输入，可以使代码更加直观和易于理解。例如，你可以有一个`print`方法用于打印不同类型的数据（如整数、字符串等），而不需要为每种类型创建单独的方法名。
2. 增加灵活性：允许用户以多种方式调用同一个方法，根据传入的不同参数执行不同的操作。比如，一个计算面积的方法可以根据传入的是圆的半径还是矩形的长宽来分别计算对应的面积。
3. 减少冗余代码：避免为了处理相似功能但略有不同的情况而编写大量重复的方法。通过重载，可以利用已有方法的名字，只更改参数列表即可。

如以下示例：

```Java
public class Calculator {
    // 加法：整数
    public static int add(int a, int b) {
        return a + b;
    }

    // 加法：浮点数
    public static double add(double a, double b) {
        return a + b;
    }
}
```

在这个例子中，`add`方法被重载了两次，一次接受两个整数参数，另一次接受两个双精度浮点数参数。这样做的好处是，当调用者需要进行加法运算时，可以根据实际的数据类型选择合适的方法，同时保持了方法命名的一致性。

调用示例：

```Java
int sumInt = add(5, 3);      // 使用 int 版本的 add 方法
double sumDouble = add(2.5, 3.7);  // 使用 double 版本的 add 方法
```

总之，方法重载是一种强大的工具，它使得Java程序设计更加灵活且具有良好的扩展性。

### 无返回值的方法中使用return

在有返回值的方法中，return用来返回结果的。

但在Java中，即使一个方法声明为void（即没有返回值），你仍然可以使用return语句。在这种情况下，return的作用是立即终止方法的执行并返回到调用该方法的地方。它不返回任何值，只是简单地结束方法。

为什么在无返回值的方法中使用 return？

- 提前退出：当你需要在满足某个条件时提前结束方法的执行，而不需要等到方法体的最后一行代码执行完毕。
- 简化逻辑：可以帮助简化复杂的控制流，使得代码更加清晰易懂。

示例：

假设有一个方法用于检查数组中的元素是否包含某个特定值，并且一旦找到就立即停止搜索：

```Java
public class Example {

    public static void main(String[] args) {
        printMessage(10); // 调用方法
        printMessage(-5); // 调用方法
    }

    // 无返回值的方法
    public static void printMessage(int number) {
        // 如果输入数字小于0，提前返回
        if (number < 0) {
            System.out.println("输入的数字不能为负数，方法结束.");
            return; // 结束方法的执行
        }

        // 继续执行其他逻辑
        System.out.println("输入的数字是: " + number);
    }
}

```

> 1. `printMessage` 方法是一个无返回值的方法，返回类型为 `void`。
> 2. 在方法内部，如果传入的参数 `number` 小于0，使用 `return` 语句提前结束方法的执行。
> 3. 否则，继续打印输入的数字。

简而言之，在void方法中使用return是为了控制程序流程，特别是在某些条件下希望提前退出方法时非常有用。

## 类型转换

在程序中，有时我们需要将不同的数据放在一起进行计算，但在java中不同类型的数据不能直接进行运算，这是因为Java是强类型语言，要求进行运算的操作数必须是相同类型。

既然不同类型的数据不能直接运算，此时就需要将数据转换成相同的类型，才能完成计算。在java中，类型转换有两大类：自动类型转换、强制类型转换。通过这两个方式，都可以将要计算的数据转换成相同类型，从而进行正确的计算。

### 自动类型转换

在Java中，**自动类型转换**（也称为隐式类型转换）是指将一种数据类型的值自动转换为另一种兼容的数据类型，而无需显式地进行转换。这通常发生在较小或较低精度的数据类型被赋值给较大或更高精度的数据类型时，例如：

- int 可以自动转换为 long、float 或 double
- float 可以自动转换为 double
- char可以自动转换成int 【**这个需要重点注意**】

![Axnjb6xFxoDctkxWqbMcFOoZnwf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Axnjb6xFxoDctkxWqbMcFOoZnwf.png)

为什么要进行类型转换呢？我们分情况来说明：

- 防止数据丢失

当一个较小类型的值被赋给较大类型的变量时，如果直接赋值可能会导致数据丢失。自动类型转换确保了这种赋值是安全的，不会丢失信息。

```Java
int a = 100;
long b = a;  // 自动将 int 转换为 long
```

- 表达式的兼容性

在涉及不同类型的操作数的表达式中，Java会自动将操作数转换为合适的类型，以确保表达式的正确计算。

```Java
int a = 5;
double b = 3.14;
double c = a + b;  // a 自动转换为 double，然后与 b 相加
```

- 方法参数传递

当调用方法时，如果传入的参数类型比方法声明的参数类型小，Java会自动进行类型转换，以匹配方法签名。

```Java
void static printDouble(double d) {
    System.out.println(d);
}

int x = 10;
printDouble(x);  // x 自动转换为 double
```

- 简化编程

自动类型转换使得代码更加简洁，减少了显式类型转换的需要，从而提高了编程效率和代码可读性。比如上面的方法参数传递，就不需要将x强制转换成double再进行传递。

### 强制类型转换

在Java中，强制类型转换（也称为显式类型转换）是指程序员明确地将一个数据类型转换为另一个数据类型。这通常是在从大范围的数据类型向小范围的数据类型转换时进行的，因为这种转换可能会导致数据丢失或精度降低。

强制类型转换的语法如下：

```Java
目标类型 变量 = (目标类型) (原始变量);
```

示例：

```Java
double d = 3.14;
int i = (int) d;  // 将 double 转换为 int，小数部分会被舍去
```

在这个例子中，d 的值 3.14 被转换为 int 类型，结果是 3，因为 int 类型不能存储小数部分。所以丢失了一部分数据。

为什么要进行强制类型转换呢？

- 处理不同类型之间的赋值

当需要将一个较大范围的数据类型赋值给较小范围的数据类型时，必须使用强制类型转换。否则，编译器会报错。

```Java
double d = 3.14;
int i = (int) d;  // 必须进行强制类型转换
```

- 方法调用参数传递

当方法参数类型与实际传入的变量类型不匹配时，可以通过强制类型转换来满足方法签名的要求。

```Java
void static printInt(int num) {
    System.out.println(num);
}

double d = 3.14;
printInt((int) d);  // 强制转换为 int
```

- 性能优化

在某些情况下，为了提高性能，可能需要将较大的数据类型转换为较小的数据类型。例如，在计算密集型应用中，使用 int 类型而不是 long 或 double 类型可以提高运算速度。

### 自动类型提升

在Java中，表达式的自动类型提升是指在涉及多种数据类型的算术运算时，Java会自动将较小的数据类型提升为较大的数据类型，以确保运算的精确性和一致性。

![P4tdb31j3onKuyxTUWice6v7nnb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/P4tdb31j3onKuyxTUWice6v7nnb.png)

注意事项：

- 表达式的最终结果类型由表达式中的最高类型决定。
- 在表达式中，byte、short、char 是直接转换成int类型参与运算的。

## 输入输出

### 了解API

在前面的学习中，我们知道JDK是Java开发工具包，它包含了开发Java应用程序所需的所有工具和库。JRE是jdk的一个重要组成部分，JRE (Java Runtime Environment)也称为运行时环境。而Java API是JRE中的一个重要组成部分，提供了丰富的类和接口，使得开发者可以方便地调用各种功能。

![EWbFbgVHfovOvRxG8S0c8ulbnYc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/EWbFbgVHfovOvRxG8S0c8ulbnYc.png)

在理解Java API之前，先想象一下你有一本工具书，里面包含了各种各样的工具和指南，帮助你完成不同的任务。比如，这本工具书里有教你如何开锁的工具、如何修理自行车的工具、如何做蛋糕的食谱等等。这些工具和指南都是预先准备好的，你可以直接拿来使用，而不需要自己从头开始发明这些工具。

在编程的世界里，Java API就像是这样一本工具书。它是一组预先编写好的代码集合，包含了大量的类、方法和常量。这些代码可以帮助你完成各种常见的任务，比如数据的输入输出、读取文件、连接网络、处理日期时间等。你不需要知道这些代码是如何工作的，只需要按照API提供的规则来使用它们。

Java API的作用

- 简化开发

就像你不需要自己发明一个开锁工具一样，你也不需要自己编写复杂的代码来实现一些基本功能。Java API已经为你准备好了这些“工具”，你可以直接拿来用。

- 提高效率

使用API可以大大加快你的开发速度。比如，如果你想让程序显示当前的时间，你不需要去研究时钟的工作原理，只需要调用java.time包中的几个方法就可以了。

- 确保质量

API中的代码是由经验丰富的开发者编写的，并且经过了广泛的测试。这意味着这些代码是可靠的，不容易出错。就像工具书里的工具已经被验证过很多次，不会出现意外。

- 标准化

所有的Java程序员都可以使用相同的API，这就像是大家都能读懂同一本工具书。这样一来，不同的程序员之间可以更容易地协作，因为他们都在使用相同的标准。

### 输入输出

在JAVA API中提供了各种各样的功能，其中就有数据的输入输出功能。本小节我们来使用java api完成数据的输入输出。

#### 数据输出

我们首先通过java api实现数据的输出，这个比较简单，我们之前一样在用了，比如以下代码：

```Java
System.out.println("Hello World!");
```

以上代码就是使用了java api中的  System的功能，实现了将数据输出到控制台的功能。

#### 数据输入

Scanner类是Java标准库中java.util包提供的一个工具，用于获取用户输入的数据。我们可以使用此类，完成数据输入的功能。

使用Scanner完成数据输入的步骤如下：

1. 导包：告诉程序去JDK的哪个包中找Scanner

```Java
package com.itheima.scanner;
import java.util.Scanner;
```

1. 抄代码：代表得到键盘扫描器对象(东西)

```Java
Scanner sc = new Scanner(System.in);
```

1. 抄代码：等待接收用户输入的数据

```Java
        System.out.println(“请输入您的年龄：”);
        int age = sc.nextInt();
        System.out.println(“年龄是：” + age);

        System.out.println(“请输入您的身高：”);
        double height = sc.nextDouble();
        System.out.println(“身高是：” + height );

        System.out.println(“请输入您的姓名：”);
        String name = sc.next();
        System.out.println("欢迎：" + name);
```

完整代码如下：

```Java
package com.itheima.scanner;
public class Test {
    public static void main(String[] args) {
        //得到键盘扫描器对象
        Scanner sc = new Scanner(System.in);

        System.out.println(“请输入您的年龄：”);
        int age = sc.nextInt();   //接收用户输入的数据
        System.out.println(“年龄是：” + age);

        System.out.println(“请输入您的姓名：”);
        String name = sc.next();   //接收用户输入的数据
        System.out.println("欢迎：" + name);
    }
}
```

程序运行时，会等待用户输入年龄、姓名，输入完成后，会在控制台显示。

## 运算符

接下来，给同学们讲解一个在开发中用得很多的一块内容，叫做运算符。

大家知道计算机是用来处理数据的，处理数据就少不了对数据的计算，想要对数据进行计算就必须用到运算符。

运算符就是参与运算的符号。Java提供的运算符有很多种，可以分为算术下面几种

- 算术运算符
- 赋值运算符
- 关系运算符
- 逻辑运算符
- 三元运算符

### 算术运算符

#### 基本算术运算符

先从最基本的算术运算符开始学习，基本算术运算符有：

- `+`:加法 两数相加
- `-`：减法 两数相减
- `*`：乘法 两数相乘
- `/`：除法 两数相除（两边都是整型时，进行整型运算）
- `%`：取模 两数取余（两个数必须是整数）

<callout emoji="💡">
注意点
- /: 两个整数相除，结果也是一个整数
- %: 表示两个数相除，取余数,取模,求余
- +： 除了用于加法运算，还可以作为连接符。+符号与字符串运算的时候是用作连接符的，其结果依然是一个字符串。
</callout>

下面通过代码演示一下各种算术运算符的运算效果

```Java
public class OperatorDemo1 {
    public static void main(String[] args) {
        // 目标：掌握基本的算术运算符的使用。
        int a = 10;
        int b = 2;
        System.out.println(a + b);
        System.out.println(a - b);
        System.out.println(a * b); // 20
        System.out.println(a / b); // 5
        System.out.println(5 / 2); // 2.5 ==> 2
        System.out.println(5.0 / 2); // 2.5
        int i = 5;
        int j = 2;
        System.out.println(1.0 * i / j); // 2.5

        System.out.println(a % b); // 0
        System.out.println(3 % 2); // 1

        System.out.println("---------------------------------------");

        // 目标2：掌握使用+符号做连接符的情况。
        int a2 = 5;
        System.out.println("abc" + a2); // "abc5"
        System.out.println(a2 + 5); //  10
        System.out.println("itheima" + a2 + 'a'); // "itheima5a"
        System.out.println(a2 + 'a' + "itheima"); // 102itheima
    }
}
```

#### 自增自减运算符

在Java中，自增 (++) 和自减 (--) 运算符用于将一个变量的值增加1或减少1。这两个运算符都有两种形式：**前缀形式**和**后缀形式**。

（1）自增运算符 `++`

- 前缀形式 (`++i`): 先执行自增操作，然后使用更新后的值。例如，如果 `int i = 5;`，那么 `int j = ++i;` 将首先使 `i` 变为6，然后把6赋给 `j`。所以，此时 `i` 和 `j` 的值都是6。
- 后缀形式 (`i++`): 先使用当前值，然后再执行自增操作。如果 `int i = 5;`，那么 `int j = i++;` 将先把 `i` 当前的值5赋给 `j`，之后再让 `i` 增加到6。因此，`j` 的值是5，而 `i` 的值变成了6。

（2）自减运算符 `--`

- 前缀形式 (`--i`): 先执行自减操作，然后使用更新后的值。比如 `int i = 5;`，那么 `int j = --i;` 将首先使 `i` 减少到4，然后把4赋给 `j`。结果 `i` 和 `j` 的值都是4。
- 后缀形式 (`i--`): 先使用当前值，然后再执行自减操作。若 `int i = 5;`，则 `int j = i--;` 会先让 `j` 等于 `i` 当前的值5，之后再让 `i` 减少到4。这样 `j` 的值是5，而 `i` 的值变为4。

总结来说，前缀形式的自增/自减运算符会在表达式计算之前立即改变变量的值，而后缀形式则是在表达式计算之后才改变变量的值。选择使用哪种形式取决于你希望何时对变量进行实际的修改。

自增自减运算符完整示例：

```Java
public class IncrementDecrementDemo {
    public static void main(String[] args) {
        int x1 = 100;
        x1++; //后缀形式
        System.out.println(x1); //此时x1为101
        x1--; //后缀形式
        System.out.println(x1); //此时x1为100

        ++x1; //前缀形式
        System.out.println(x1); //此时x1为101
        --x1; //前缀形式
        System.out.println(x1); //此时x1为100

        int a = 5;
        int b;

        // 前缀自增
        b = ++a; // 先增加a, 然后赋值给b
        System.out.println("After prefix increment: a = " + a + ", b = " + b);

        // 后缀自增
        a = 5; // 重置a的值
        b = a++; // 先赋值给b, 然后增加a
        System.out.println("After postfix increment: a = " + a + ", b = " + b);

        // 前缀自减
        a = 5; // 重置a的值
        b = --a; // 先减少a, 然后赋值给b
        System.out.println("After prefix decrement: a = " + a + ", b = " + b);

        // 后缀自减
        a = 5; // 重置a的值
        b = a--; // 先赋值给b, 然后减少a
        System.out.println("After postfix decrement: a = " + a + ", b = " + b);
    }
}
```

### 赋值运算符

接下来，我们学习赋值运算符。赋值运算符主要用于为变量设置值，java中的赋值运算符有：

- `=`：赋值
- `+=`：加并赋值
- `-=`：减并赋值
- `*=`：乘并赋值
- `/=`：除并赋值
- `%=`：取模并赋值

以上各运算符的使用示例如下：

```Java
public class AssignmentOperatorsDemo {
    public static void main(String[] args) {
        int a = 10; // 基本赋值
        int b = 20;

        System.out.println("Initial values: a = " + a + ", b = " + b);

        // 加法赋值
        a += 3; // 相当于 a = a + 3
        System.out.println("After a += 3, a = " + a);

        // 减法赋值
        b -= 5; // 相当于 b = b - 5
        System.out.println("After b -= 5, b = " + b);

        // 乘法赋值
        a *= 2; // 相当于 a = a * 2
        System.out.println("After a *= 2, a = " + a);

        // 除法赋值
        b /= 4; // 相当于 b = b / 4
        System.out.println("After b /= 4, b = " + b);

        // 取模赋值
        a %= 3; // 相当于 a = a % 3
        System.out.println("After a %= 3, a = " + a);
    }
}
```

以上代码是这样工作的：

- `a += 3` 将 `a` 的当前值（10）加上3，结果是13。
- `b -= 5` 从 `b` 的当前值（20）减去5，结果是15。
- `a *= 2` 将 `a` 的当前值（13）乘以2，结果是26。
- `b /= 4` 将 `b` 的当前值（15）除以4，结果是3（整数除法，舍弃小数部分）。
- `a %= 3` 计算 `a` 的当前值（26）除以3的余数，结果是2。

<callout emoji="🧥">
**注意: 扩展赋值运算符隐含了强制类型转换, ++,-- 也隐含强制类型转换**
</callout>

### 关系运算符

在Java中，关系运算符用于比较两个值，并返回一个布尔结果（`true` 或 `false`）。这些运算符通常用于控制程序根据条件执行不同的代码。以下是Java中的主要关系运算符

- `==`：等于
- `!=`：不等于
- `<`：小于
- `>`：大于
- `<=`：小于等于
- `>=`：大于等于

下面是一个简单的Java程序示例，演示了各种关系运算符的使用：

```Java
public class RelationalOperatorsDemo {
    public static void main(String[] args) {
        int a = 10;
        int b = 20;

        // 等于 (==)
        System.out.println("a == b: " + (a == b)); // 输出 false

        // 不等于 (!=)
        System.out.println("a != b: " + (a != b)); // 输出 true

        // 大于 (>)
        System.out.println("a > b: " + (a > b)); // 输出 false

        // 小于 (<)
        System.out.println("a < b: " + (a < b)); // 输出 true

        // 大于等于 (>=)
        System.out.println("a >= b: " + (a >= b)); // 输出 false

        // 小于等于 (<=)
        System.out.println("a <= b: " + (a <= b)); // 输出 true
    }
}
```

当你运行这个程序时，你将看到以下输出：

![FF4pbgSn2oNk62xRs9lcXuUvnrh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/FF4pbgSn2oNk62xRs9lcXuUvnrh.png)

执行说明如下：

- `a == b` 检查 `a` 是否等于 `b`。因为 `a` 是 10 而 `b` 是 20，所以结果是 `false`。
- `a != b` 检查 `a` 是否不等于 `b`。因为 `a` 和 `b` 的值不同，所以结果是 `true`。
- `a > b` 检查 `a` 是否大于 `b`。因为 `a` 是 10 而 `b` 是 20，所以结果是 `false`。
- `a < b` 检查 `a` 是否小于 `b`。因为 `a` 是 10 而 `b` 是 20，所以结果是 `true`。
- `a >= b` 检查 `a` 是否大于或等于 `b`。因为 `a` 是 10 而 `b` 是 20，所以结果是 `false`。
- `a <= b` 检查 `a` 是否小于或等于 `b`。因为 `a` 是 10 而 `b` 是 20，所以结果是 `true`。

### 三元运算符

Java中的三元运算符（也称为条件运算符）是一种简洁的语法结构，用于基于一个布尔表达式的结果来选择两个值中的一个。它的语法如下：

```Java
condition ? valueIfTrue : valueIfFalse
```

- `condition` 是一个布尔表达式，它必须返回 `true` 或 `false`。
- 如果 `condition` 为 `true`，则整个表达式的值是 `valueIfTrue`。
- 如果 `condition` 为 `false`，则整个表达式的值是 `valueIfFalse`。

三元运算符的使用示例：

```Java
public class TernaryOperatorDemo {
    public static void main(String[] args) {
        int score = 85;
        String result;

        // 使用三元运算符
        result = (score >= 60) ? "Pass" : "Fail";

        System.out.println("The student's result is: " + result);
    }
}
```

在这个示例中：

- `score` 变量被设置为 85。
- 三元运算符 `(score >= 60) ? "Pass" : "Fail"` 检查 `score` 是否大于或等于 60。
- 如果 `score` 大于或等于 60，则 `result` 被赋值为 `"Pass"`。
- 否则，`result` 被赋值为 `"Fail"`。

程序的输出将是：

> The student's result is: Pass

### 逻辑运算符

学习完关系运算符之后，接下来我们学习一下逻辑运算符。java中的逻辑运算符如下：

- `&&`：逻辑与（与）
- `||`：逻辑或（或）
- `!`：逻辑非（非）

各个逻辑运算符的作用详细说明如下：

1. 逻辑与 (`&&`):

   - 只有两个操作数都为 `true` 时，结果才为 `true`。
   - 如果第一个操作数是 `false`，则不会计算第二个操作数（短路特性）。
2. 逻辑或 (`||`):

   - 只要有一个操作数为 `true`，结果就为 `true`。
   - 如果第一个操作数是 `true`，则不会计算第二个操作数（短路特性）。
3. 逻辑非 (`!`):

   - 用于反转一个布尔值。
   - 如果操作数是 `true`，结果是 `false`；如果操作数是 `false`，结果是 `true`。
4. 异或(^):

   1. 相同为false ,不同为true

下面通过一个示例演示逻辑运算符的用法：

```Java
//需求1：要求手机必须满足尺寸大于等于6.95，且内存必须大于等于8.
//需求2：要求手机要么满足尺寸大于等于6.95，要么内存必须大于等于8.
public class OperatorDemo5 {
    public static void main(String[] args) {
        // 目标：掌握逻辑运算符的使用。
        // 需求：要求手机必须满足尺寸大于等于6.95，且内存必须大于等于8.
        double size = 6.8;
        int storage = 16;
        // 1、& 前后的条件的结果必须都是true ,结果才是true.
        boolean rs = size >= 6.95 & storage >= 8;
        System.out.println(rs);

        // 需求2：要求手机要么满足尺寸大于等于6.95，要么内存必须大于等于8.
        // 2、| 只要多个条件中有一个是true,结果就是true.
        boolean rs2 = size >= 6.95 | storage >= 8;
        System.out.println(rs2);

        // 3、!取反的意思
        System.out.println(!true); // false
        System.out.println(!false); // true
        System.out.println(!(2 > 1)); // false

        // 4、^ 前后条件的结果相同时返回false，不同时返回true.
        System.out.println(true ^ true); // false
        System.out.println(false ^ false); // false
        System.out.println(true ^ false); // true
        System.out.println(false ^ true); // true

        // 5、&& 左边为false，右边不执行。
        int i = 10;
        int j = 20;
        // System.out.println(i > 100 && ++j > 99);
        System.out.println(i > 100 & ++j > 99);
        System.out.println(j);

        // 6、|| 左边是true ，右边就不执行。
        int m = 10;
        int n = 30;
        // System.out.println(m > 3 || ++n > 40);
        System.out.println(m > 3 | ++n > 40);
        System.out.println(n);
    }
}
```

## 综合小案例

### 案例需求

今天我们学习了方法、类型转换、运算符、输入输出，接下来我们运用这些知识点，来实现一个综合案例。案例需求如下：

开发一个简单的健康计算器应用程序，它可以接受用户的输入（如年龄、性别、体重、身高），并计算出用户的BMI（身体质量指数）和基础代谢率（BMR）。这两个指标的计算公式如下：

- BMI指标计算公式

![PSH7bgYxho7tDQxRvRQchC0lnmf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/PSH7bgYxho7tDQxRvRQchC0lnmf.png)

- BMR指标计算公式

![MdplbwaVzoMsuMxYgI2csgvSnZf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/MdplbwaVzoMsuMxYgI2csgvSnZf.png)

### 案例实现

根据以上需求说明，我们最终需要计算2个指标：BMI、BMR，实现思路分析如下：

- 分析指标计算需要的基础数据

BMI：身高、体重

BMR：身高、体重、性别、年龄

- 指标的计算逻辑实现

每个指标的计算公式各不相同，我们可以编写2个方法， 一个用来计算BMI，一个用来计算BMR。根据各指标所需的基础数据分析，我们得知：

计算BMI指标的方法，需要2个参数（身高、体重），要返回一个结果（小数）

计算BMR指标的方法，需要2个参数（身高、体重、性别、年龄），要返回一个结果（小数）

- 提示用户输入自己的数据

可以使用java中的输入输出功能，接收用户数据（身高、体重、性别、年龄）

- 对数据进行计算

获取用户数据后，可以调用上面的2个指标计算方法，进行计算，然后获取结果并输出。

根据以上分析，代码实现如下：

```Java
package com.itheima.demo;

import java.util.Scanner;

public class AllTest {
    public static void main(String[] args) {
        // 目标：完成健康计算器。
        Scanner sc = new Scanner(System.in);
        // 1、先让用户输入自己的个人信息：身高、体重、性别、年龄
        System.out.println("请您输入您的身高：");
        double height = sc.nextDouble();

        System.out.println("请您输入您的体重：");
        double weight = sc.nextDouble();

        System.out.println("请您输入您的性别（男/女）：");
        String sex = sc.next(); // "男"

        System.out.println("请您输入您的年龄：");
        int age = sc.nextInt();

        double bmi = calcBMI(height, weight);
        System.out.println("您的bmi值是：" + bmi);

        // 判断用户的bmi值的情况。。。

        double bmr = calcBMR(height, weight, sex, age);
        System.out.println("您的bmr值是：" + bmr);

        // 判断bmr的情况。。。
    }

    // 2、根据个人信息，计算BMI指数（把数据交给一个独立的方法来计算并返回这个结果）
    public static double calcBMI(double height, double weight) {
        return weight / (height * height);
    }

    // 3、根据个人信息，计算BMR指数（把数据交给一个独立的方法来计算并返回这个结果）
    public static double calcBMR(double height, double weight, String sex, int age) {
        double bmr = 0;
        if ("男".equals(sex)) {
            bmr = 88.362 + (13.397 * weight + 4.799 * height - 5.677 * age) ;
        } else {
            bmr = 447.593 + (9.247 * weight + 3.098 * height - 4.330 * age);
        }
        return bmr;
    }
}

```

## 相关笔记

- [[Memo/development/programming/java/java-basics/02-java-syntax-basics-quiz.md|02 基础语法·问答]]
- [[Memo/development/programming/java/java-syntax-and-control-flow/06-operators-and-common-pitfalls.md|运算符精讲与常见坑]]
- [[Memo/development/programming/java/java-syntax-and-control-flow/07-methods-and-branching.md|方法详解与分支结构]]

---

> 来源：飞书《讲义》知识库 · day02-Java基础语法 · [原文](https://baxsu7o100e.feishu.cn/wiki/TLykwAi5KiwXG3kvmhKc2rwHnqf)
