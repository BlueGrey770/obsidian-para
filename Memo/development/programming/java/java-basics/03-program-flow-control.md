---
type: memo
date: 2026-11-09
aliases:
  - "分支与循环"
---

# 程序流程控制

我们前面写的程序，都是按顺序一行行向下执行的，但在实际业务处理中，大多数据场景下，需要根据不同的条件执行不同的代码。比如：在手机银行app中，如果用户登录成功了，则显示个人帐户信息，如果登录失败了，则给予错误提示。要实现这种根据不同情况执行不同代码的效果，就需要对程序的执行流程进行控制，这需要通过专门的语法来实现。本小节我们来学习java中的流程控制语法。

在Java程序中，代码的执行顺序可以通过三种基本控制结构来定义：顺序、分支和循环。这三种结构是程序设计的基础，并且它们决定了程序中语句的执行流程。

![L8CUbkd6VoYDqzx2V5mcYAFln1c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/L8CUbkd6VoYDqzx2V5mcYAFln1c.png)

1. 顺序结构
想象你在按照一个清单做事情。比如你有一个早晨的例行公事清单：先刷牙、然后洗脸、再吃早餐。每完成一步，你就接着做下一步。在编程中也是这样，代码就像你的清单一样，从上到下一条条地被执行。如果没有特别指定，程序就会按部就班地逐行运行下去。
2. 分支结构（条件判断）
假设今天早上起来后，你需要决定是否带伞出门。如果你看窗外发现正在下雨，那么你会带上伞；如果没下雨，你就不会带。在编程里，这叫做“分支”或“条件判断”。根据某个条件（如天气情况），程序会选择执行一部分代码而跳过另一部分。这就像是生活中根据实际情况做出不同选择的过程。
3. 循环结构
循环就像是当你需要重复做一些事情时的情景。比如说，你想给朋友送生日卡片，但你有很多个朋友。对于每个朋友，你都要写一张卡片。在这里，“为每位朋友写卡片”这个动作会重复很多次，直到所有的朋友都收到卡片为止。在编程中，循环允许我们设定一组指令，让它自动重复执行多次，直到满足某个结束条件为止。

通过这三种结构，程序员能够编写出既简单又复杂的逻辑来解决各种问题。

## 分支结构

### if分支结构

接下来我们学习分支结构的第一种形式——if分支。

if它的作用，是用于对条件进行判断，判断的结果只可能有两个值true或者false，然后根据条件判断的结果来决定执行哪段代码。

#### if分支的应用场景

比如，在火车站、地铁站等公共场所，会对过往的旅客测体温。如果体温在37度以内，就属于正常的；如果体温在37读以上，测体温的装置就会报警。

再比如，你在使用微信付钱时，微信内部的程序会先判断你的余额是否足够，如果足够就可以支付成功；如果余额不足，就会提示支付失败。

#### if分支的格式

接下来，我们来看一看if分支在Java语言中长什么样子呢？在Java中if分支有三种格式。

![QqEkbjAdmoHwlRx9Bf0cX3QsnAe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/QqEkbjAdmoHwlRx9Bf0cX3QsnAe.png)

#### if语句第一种格式

if 第一种形式的代码格式，和执行流程如下图所示：

![C1rIb3IGzoxukExiQXWc9wjsnxd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/C1rIb3IGzoxukExiQXWc9wjsnxd.png)

if 第一种形式执行流程如下：

- 首先判断条件表达式的结果
- 如果为true执行语句体
- 为 false 就不执行语句体。

if第一种形式的示例如下：

```Java
// 需求：测量用户体温，发现体温高于37度就报警。
double t = 36.9;
if(t > 37){
    System.out.println("这个人的温度异常，把他赶紧带走~~");
}
```

**运行结果分析：**

- 在这个例子中，`t`被初始化为36.9，这意味着用户的体温是36.9度。
- `if`语句检查`t`是否大于37度。
- 因为36.9并不大于37，所以`if`条件不满足。
- 既然条件不满足，那么`if`块中的`System.out.println()`语句就不会被执行。

因此，运行结果是程序不会输出任何信息。如果将`t`设置为大于37的数值（例如37.1），那么程序就会输出：“这个人的温度异常，把他赶紧带走\~\~”。

#### if语句第二种格式

前面的示例中，只要一个分支，如果程序有两个分支的话就不能应对了。不用着急，if语句还提供了第二种用法。

if 第二种形式的代码格式，和执行流程如下图所示：

![LLB1bEzwWoKpyWxVp7QcLQtnngf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/LLB1bEzwWoKpyWxVp7QcLQtnngf.png)

if 第二种形式执行流程如下：

- 首先判断条件表达式的结果
- 如果为true执行语句体1
- 为 false 就执行语句体2。

if第二种形式的示例如下：

```Java
// 需求2：发红包，你的钱包余额是99元，现在要发出90元
// 如果钱够触发发红包的动作，如果钱不够，则提示：余额不足。
double money = 19;
if(money >= 90){
    System.out.println("发红包成功了~");
}else {
    System.out.println("余额不足~~");
}
```

**运行结果分析：**

- 在这个例子中，`money`被初始化为19，这意味着当前钱包里的余额是19元。
- `if`语句检查`money`是否大于或等于90元。
- 因为19并不大于或等于90，所以`if`条件不满足。
- 既然`if`条件不满足，程序会执行`else`块中的代码。

因此，运行结果将是： 余额不足\~\~

#### if语句第三种格式

前面我们通过if...else语法，可以实现2个分支的代码执行。那么如果碰到多个分支的情况，又应该如何应对呢？接下来我们学习if语句的第三种语法。

if 第三种形式的代码格式，和执行流程如下图所示：

![PZksbUTWeonlEmx1Z9qctVIPntf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/PZksbUTWeonlEmx1Z9qctVIPntf.png)

if 第三种形式执行流程如下：

- 如果 条件表达式1 为true,就执行下面的代码1;
- 如果 条件表达式1 为false，就继续判断条件表达式2;
- 如果 条件表达式2 为true，就执行下面的语句体;
- 如果 条件表达式2 为false，就继续判断条件语句体3;
- 如果 条件表达式3 为true,就执行下面的语句体3
- 如果 条件表达式3 为false,就继续判断后面的表达式;
- ...
- 如果没有任何条件为true，就执行else分支的语句体n+1

if 第三种形式的示例如下：

```Java
// 需求3：某个公司有一个绩效系统，根据员工的打分输出对应的绩效级别。
// [0,60) D  [60,80) C [80,90) B [90,100] A

int score = 298;
if(score >= 0 && score < 60) {
    System.out.println("您的绩效级别是： D");
}else if(score >= 60 && score < 80){
    System.out.println("您的绩效级别是： C");
}else if(score >= 80 && score < 90){
    System.out.println("您的绩效级别是： B");
}else if(score >= 90 && score <= 100){
    System.out.println("您的绩效级别是： A");
}else {
    System.out.println("您录入的分数有毛病~~");
}
```

**运行结果分析：**

- 在这个例子中，`score`被初始化为298。
- 程序首先检查`score`是否在0到59之间，显然298不在这个范围内。
- 接着检查`score`是否在60到79之间，298也不在这个范围内。
- 再次检查`score`是否在80到89之间，298仍然不在这个范围内。
- 最后检查`score`是否在90到100之间，298还是不在这个范围内。
- 由于`score`（298）不在任何一个预设的分数区间内，因此程序会执行`else`块中的代码。

因此，运行结果将是： 您录入的分数有毛病\~\~

#### if语句注意事项

同学们在第一次写if 代码时，经常一不小心会出现一些问题。下面把同学们可能出现的问题给大家看一看，以后大家要避免出现这些问题。

- 第1个问题：if的()后面不能写分号`;` 否则if下面的语句与if无关
- 第2个问题：if后面的大括号，如果只有一行代码，大括号可以省略

### switch分支结构

要实现分支流程控制，除了使用if语句之外，还可以使用switch语句来实现。

在Java中，`switch`语句是一种控制结构，用于基于不同的条件执行多个代码块之一。它提供了一种比使用多个`if-else`语句更清晰、更简洁的方式来处理多分支选择的情况。`switch`语句通常用于当需要根据一个变量的值来决定执行哪一段代码时。

#### switch基本语法

switch语句的基本语法如下：

```Java
switch (expression) {
    case value1:
        // 代码块1
        break;
    case value2:
        // 代码块2
        break;
    ...
    case valueN:
        // 代码块N
        break;
    default:
        // 默认代码块
        break;
}
```

**执行流程：**

1. 计算表达式：首先计算`switch`关键字后面的表达式的值。
2. 匹配case：然后程序会检查每个`case`标签后面跟随的值，看是否有与表达式的值相匹配的值。
3. 执行匹配的case：如果找到匹配的`case`，则从该`case`开始执行下面的代码，直到遇到`break`语句或`switch`块结束为止。
4. 遇到break：`break`语句用来终止`switch`语句的执行，并跳出整个`switch`块。
5. 默认情况：如果没有`case`的值与表达式的值匹配，则执行`default`标签后的代码块（如果有`default`标签的话）。`default`是可选的，可以用来处理所有其他未列出的情况。

下面通过示例，演示switch语句的具体用法：

```Java
/*
需求：做个电子备忘录，在控制台分别输出周一到周五的工作安排
    周一：埋头苦干，解决bug
    周二： 请求大牛程序员帮忙
    周三：今晚啤酒、龙虾、小烧烤
    周四：主动帮助新来的女程序解决bug
    周五：今晚吃鸡
    周六：与王婆介绍的小芳相亲
    周日：郁郁寡欢、准备上班。
*/

String week = "周三";
switch (week){
    case "周一":
        System.out.println("埋头苦干，解决bug");
        break;
    case "周二":
        System.out.println("请求大牛程序员帮忙");
        break;
    case "周三":
        System.out.println("今晚啤酒、龙虾、小烧烤");
        break;
    case "周四":
        System.out.println("主动帮助新来的女程序解决bug");
        break;
    case "周五":
        System.out.println("今晚吃鸡");
        break;
    case "周六":
        System.out.println("与王婆介绍的小芳相亲");
        break;
    case "周日":
        System.out.println("郁郁寡欢、准备上班");
        break;
    default:
        System.out.println("您输入的星期信息不存在~~~");
        break;
}
```

#### switch注意事项

接下来我们学习swtich的注意事项。同学们掌握这些注意事项之后，就可以避免入坑了，也可以应对一些面试笔试题。

- 表达式类型只能是byte、short、int、char，  JDK5开始支持枚举，JDK7开始支持String，  不支持double、float、long

```Java
        int a = 10;
        switch(a){ //只允许: byte、short、int、char、String、枚举
            case 10:
                System.out.println("10");
                //break;
            case 20:
                System.out.println("10");
                break;
            default:
                System.out.println("......");
        }
```

- case给出的值不允许重复，且只能是字面量，不能是变量。
- 正常使用switch的时候，不要忘记写break，否则会出现穿透现象。

所谓穿透现象，是指当switch语句中没有遇到break，就会直接穿透到下一个case语句执行，直到遇到break为止。

这种语法设计也是有它的用处的，当多个case语句想要执行同一段代码时，可以利用case穿透现象，提高代码复用性。

比如：我们下面程序中，想要让周二、周三、周四，都请大牛程序员来写代码。

![MpihbEVOfo4HQUxwBhncEFlZnQh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/MpihbEVOfo4HQUxwBhncEFlZnQh.png)

```Java
package com.heima;

public class Demo {
    public static void main(String[] args) {
        //利用switch的穿透(不写break)
        // 周一：埋头苦干，解决bug
        // 周二：请求大牛程序员帮忙
        // 周三：请求大牛程序员帮忙
        // 周四：请求大牛程序员帮忙
        // 周五：自己整理代码
        // 周六：打游戏
        // 周日：打游戏
        String week = "周日";

        switch(week){ //只允许: byte、short、int、char、String
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

    }
}
```

- jdk12对switch结构的优化

```Java
switch(表达式){
    case 值1,值2,值3 ... ->{
        语句体;
    }
    case 值4,值5,值6 ... ->{
        语句体;
    }
    default ->{
    }
}
如果语句体只有一句{}可以省略
```

例如: 键盘输入月份, 求出对应的季节

```Java
public class Demo {

    public static void main(String[] args) {

        Scanner sc = new Scanner(System.in);
        System.out.println("请输入月份"); // 3~5 春季  6~8 夏季  9~11 秋季  12,1,2 冬季
        int month = sc.nextInt();

        switch (month){
            case  3,4,5 -> System.out.println("春季");
            case  6,7,8 -> System.out.println("夏季");
            case  9,10,11 -> System.out.println("秋季");
            case  12,1,2 -> System.out.println("冬季");
            default -> System.out.println("月份输入有误");
        }

    }
}
```

#### 分支结构的选择

学习完switch 分支之后，有同学可能会想，已经了有if分支，为什么还有switch分支呢？感觉上面的案例用if分支也能做啊？  那我们在具体应用场景下如何选择呢？

如果单从功能上来讲，if 分支 的功能是更加强大的，switch分支能做的事情if 分支都能做。但是具体用哪一种分支形式，也是有一些使用原则的：

- 如果是对一个范围进行判断，建议使用if分支结构。

比如：1-18岁是未成年， 18岁以上是成年。就需要使用If语句，如果用switch语句来实现，则需要将每个年龄都列出来，不可行。

- 如果是与一个一个的值比较的时候，建议使用switch分支结构

比如：判断月份对应的季节，这个情况中有具体的值（12个月份），而且值也不算太多。就可以使用switch将各个不同的值都列出来，进行分支控制。

## 循环结构

接下来我们学习循环结构。循环结构可以控制一段代码重复执行。循环结构有for循环、while循环、do-while循环。

### for循环

`for`循环在Java中主要用于当已知迭代次数或需要根据一个明确的计数器来控制循环时。它允许开发者简洁地定义循环开始前的初始化、循环继续的条件以及每次循环后执行的操作（如递增或递减）。`for`循环非常适合用于进行固定次数的操作。

#### for循环语法

for循环的基本语法及执行流程如下：

![XWOKbevVhoYnSyxY1IvccodcnPh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/XWOKbevVhoYnSyxY1IvccodcnPh.png)

下面我们通过一个示例，了解下for循环的执行流程：

```Java
// 输出3次HelloWorld
for (int i = 0; i < 3; i++) {
    System.out.println("Hello World");
}
```

以上代码的执行流程：

1. 循环一开始，执行int i = 0 一次。
2. 此时  i=0 ，接着计算机执行循环条件语句：0 < 3 返回true , 计算机就进到循环体中执行，输出 ：helloWorld ，然后执行迭代语句i++。
3. 此时  i=1 ，接着计算机执行循环条件语句：1 < 3 返回true , 计算机就进到循环体中执行，输出 ：helloWorld ，然后执行迭代语句i++。
4. 此时  i=2 ，接着计算机执行循环条件语句：2 < 3 返回true , 计算机就进到循环体中执行，输出 ：helloWorld ，然后执行迭代语句i++。
5. 此时  i=3 ，然后判断循环条件：3 < 3 返回false, 循环立即结束！！

#### for循环案例-求和

需求：求1-5之间的数据和，并把求和结果在控制台输出。

```Java
int sum = 0；
for (int i = 1; i <= 5; i++) {
   sum += i；
}
System.out.println("1-5之间的数据累加和是:" + sum);
```

#### for循环案例-求奇数和

需求：求1-10之间的奇数和，并把求和结果在控制台输出。

```Java
int sum = 0；
for (int i = 1; i <= 10 ; i++) {
    if(i % 2 == 1){
        // i = 1 3 5 7 9
        sum += i；
    }
 }
 System.out.println("1-10之间的奇数累加和是:" + sum);
```

#### for循环案例-求奇数和(方案2)

需求：求1-10之间的奇数和，并把求和结果在控制台输出。

思路分析：我们可以声明一个变量i，从1开始，每次加2，这样每次变量i的值刚好是奇数，可以省略奇偶判断。

```Java
int sum = 0；
for (int i = 1; i <= 10 ; i+=2) {
   // i = 1 3 5 7 9
   sum += i；
 }
 System.out.println("1-10之间的奇数累加和是:" + sum);
```

### while循环

接下来我们学习第二种循环结构——while循环，`while`循环在Java中用于当需要重复执行一段代码直到某个条件不再满足时。与`for`循环不同，`while`循环更适用于那些在开始时不知道具体迭代次数的情况，而是根据某个条件来决定是否继续执行循环体。

<callout emoji="💡">
小技巧
知道循环几次：使用for；不知道循环几次建议使用：while
</callout>

#### while循环语法

while循环的基本语法如下：

![CWj3bUas1onMpsxUQ4pcyxWan2g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/CWj3bUas1onMpsxUQ4pcyxWan2g.png)

下面我们通过一个示例，了解下while循环的执行流程：

```Java
int i = 0;
while (i < 3) {
    System.out.println("Hello World");
    i++;
}
```

以上代码的执行流程：

1. 循环一开始，执行int i = 0 一次。
2. 此时  i=0 ，接着计算机执行循环条件语句：0 < 3 返回true , 计算机就进到循环体中执行，输出 ：helloWorld ，然后执行迭代语句i++。
3. 此时  i=1 ，接着计算机执行循环条件语句：1 < 3 返回true , 计算机就进到循环体中执行，输出 ：helloWorld ，然后执行迭代语句i++。
4. 此时  i=2 ，接着计算机执行循环条件语句：2 < 3 返回true , 计算机就进到循环体中执行，输出 ：helloWorld ，然后执行迭代语句i++。
5. 此时  i=3 ，然后判断循环条件：3 < 3 返回false, 循环立即结束！！

#### while循环案例-复利计算

需求：假设你在银行投资了100000元，银行给出的复利是1.7%，问多少年后能实现本金翻倍？

实现思路：

- 初始化投资的本金和目标金额。
- 使用一个变量来跟踪已经经过的年数。
- 在一个while循环中，每年根据复利公式更新投资金额，直到它至少翻倍。
- 打印或返回所需的年数

```Java
double principal = 1000000; // 初始本金
double rate = 0.017; // 年利率
double target = 2 * principal; // 目标金额，本金翻倍
int years = 0; // 已经经过的年数

while (principal < target) {
    principal += principal * rate; // 应用复利公式
    years++; // 增加年数
}

System.out.println("需要的年数：" + years);
```

#### while循环案例-珠峰高度

需求：世界最高山峰珠穆朗玛峰高度是：8848.86米=8848860毫米，假如我有一张足够大的纸，它的厚度是0.1毫米。请问：该纸张折叠多少次，可以折成珠穆朗玛峰的高度？

思路分析：

1. 首先由于不知道折叠多少次，无法使用for循环，我们可以选择用while循环
2. 定义变量存储珠穆朗玛峰的高度、纸张的高度。
3. 使用while循环来控制纸张折叠，循环条件是（纸张厚度<山峰高度），循环每执行一次，就表示纸张折叠一次，并把纸张厚度变为原来两倍。
4. 循环外定义计数变量count，循环每折叠一次纸张，让count变量+1；

```Java
package com.heima;

/**
 * 测试：do...while循环
 *
 * 需求：使用一张0.1毫米的纸张，折叠成珠峰高度：8848860毫米，问需要折叠几次
 */
public class Demo18 {
    public static void main(String[] args) {
        //珠峰高度
        double peakHeight = 8848860; //毫米
        //纸张厚度
        double paperThickness = 0.1; //毫米

        //对折几次
        int count  = 0;

        //循环
        do{
            paperThickness = paperThickness *2;
            count ++; //累加对折次数
            //输出每次对折后,纸的厚度
            System.out.println("对折" + count  +  "次后的厚度是" + paperThickness);
        }while (paperThickness<peakHeight);

        //输出
        System.out.println("总共需要对折的次数:" + count );
    }
}
```

### do-while循环

接下来我们学习循环结构的第三种格式——do-while循环。

`do-while`循环在Java中用于确保循环体至少被执行一次，然后再根据条件判断是否继续执行。这与`while`循环不同，`while`循环可能一次都不执行（如果初始条件就不满足的话）。`do-while`循环适用于那些需要先执行一次操作，然后根据结果决定是否重复的情况。

#### do-while循环语法

do-while循环的基本语法和执行流程如下：

![QCalbwCyQowr9ZxbCu2crafvnZc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/QCalbwCyQowr9ZxbCu2crafvnZc.png)

下面我们通过一个示例，了解下 do-while 循环的执行流程：

```Java
int i = 0;
do {
       System.out.println(“Hello World！");
       i++;
} while( i < 3);
```

以上代码使用了do-while循环来打印"Hello World!"三次，执行流程如下：

1. 初始化：

   - 变量`i`被初始化为0。
2. 第一次迭代：

   - 进入`do`块，执行`System.out.println("Hello World!");`，输出"Hello World!"。
   - 然后执行`i++`，使`i`从0增加到1。
   - 检查`while`条件`i < 3`，此时`i`是1，条件为真（`true`），所以继续下一次迭代。
3. 第二次迭代：

   - 再次进入`do`块，执行`System.out.println("Hello World!");`，输出"Hello World!"。
   - 然后执行`i++`，使`i`从1增加到2。
   - 检查`while`条件`i < 3`，此时`i`是2，条件仍然为真（`true`），所以继续下一次迭代。
4. 第三次迭代：

   - 再次进入`do`块，执行`System.out.println("Hello World!");`，输出"Hello World!"。
   - 然后执行`i++`，使`i`从2增加到3。
   - 检查`while`条件`i < 3`，此时`i`是3，条件为假（`false`），因此退出循环。

#### 三种循环小结

在Java中，`for`、`while`和`do-while`循环都用于重复执行一段代码，但它们在使用场景、语法结构和执行流程上有一些关键的区别。

1. for循环

适用于已知迭代次数的情况，或者需要基于一个计数器来控制循环

先判断后执行

1. while循环

适用于不知道具体迭代次数，但可以根据某个条件来决定是否继续执行的情况

先判断后执行

1. do-while循环

适用于至少需要执行一次循环体的情况，然后再根据条件决定是否继续执行。

先执行后判断

### 死循环

死循环（也称为无限循环）是指一种永远不会自然终止的循环。这种循环会一直执行下去，除非通过某种外部手段强制中断它，比如手动停止程序运行。

死循环的原因：条件永远为真（如果循环的条件表达式始终为true，那么循环将不会停止）。

常见的死循环写法：

```Java
for ( ; ; ) {
    System.out.println("Hello World1");
 }

while (true) {
    System.out.println("Hello World2");
 }

do {
    System.out.println("Hello World3");
 } while (true);
```

说明：

- 上面的for循环没有初始化部分、条件判断部分和更新部分，条件判断部分省略后，相当于条件永远为真(true)，因此它会一直执行下去，永远不会自然终止
- while循环的条件判断部分恒为true，故循环永远不会自然终止
- do-while循环的条件判断部分恒为true，故循环永远不会自然终止

## break、continue

在Java循环中，break和continue是两个控制流语句，它们用于改变循环的正常执行流程。这两个关键字的作用如下：

- break作用：跳出并结束当前所在循环的执行，也可用于结束所在switch分支的执行
- continue作用：结束本次循环，进入下一次循环

下面我们通过示例来理解下这2个关键字的作用：

示例1：演示break的使用，提前终止循环的执行

```Java
// 场景：假如你又有老婆了，你犯错了，你老婆罚你说：5句我爱你
// 说到第三句的时候心软了，让你别再说了。
for (int i = 1; i <= 5; i++) {
    System.out.println("我爱你：" + i);
    if(i == 3){
        // 说明已经说完了第三句了，心软了。
        break; // 跳出并结束当前所在循环的执行。
    }
}
```

以上代码在i为3时，结束循环，运行结果如下：

我爱你：1

我爱你：2

我爱你：3

示例2：演示continue的使用，结束循环中的一次，继续下一次循环

```Java
// 场景: 假如你有老婆，你犯错了，你老婆罚你洗碗5天。
// 第三天的时候，你表现很好，第三天不用洗碗，但是不解恨，第四天还是要继续的。
for (int i = 1; i <= 5; i++) {
    if(i == 3) {
        // 已经到了第三天，第三天不用洗的。
        continue;
    }
    System.out.println("洗碗：" + i);
}
```

以上代码在i为3时，跳过本次循环直接跳到下次循环，运行结果如下：

洗碗：1

洗碗：2

洗碗：4

洗碗：5

## 综合小案例

前面我们学习了java中的分支结构、循环结构的基本语法，接下来我们编写几个综合小案例，加深对知识点的理解和综合运用。

### 简单计算器

需求：设计一个可以执行基本数学运算（加、减、乘、除）的计算器程序。

功能描述：

- 用户输入两个数字、一个运算符（+、-、\*、/）。
- 根据所选运算符执行相应的数学运算，显示运算结果。
- 运行效果如下图所示：

![GiqgbX33aoh9lXx3GVwc3Rmvn0d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/GiqgbX33aoh9lXx3GVwc3Rmvn0d.png)

思路分析：

- 由于要输入数字，所以先创建一个键盘录入器Scanner
- 从键盘接收两个数字
- 从键盘接收一个运算符
- 定义一个方法，用于实现数字的计算
- 调用方法，获取计算结果，并输出

代码实现如下：

```Java
package com.heima.demo;

import java.util.Scanner;

public class Test1 {
    public static void main(String[] args) {
        // 目标：简易版计算器开发。
        // 1、键盘录入两个整数，以及运算符。
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入第一个数字:");
        double num1 = sc.nextDouble();

        System.out.println("请输入第二个数字:");
        double num2 = sc.nextDouble();

        System.out.println("请输入运算符(+, -, *, /):");
        String operator = sc.next(); // "+"

        // 2、把这个数据交给一个独立的计算方法，帮我计算结果并返回给我。
        double result = calc(num1, num2, operator);
        System.out.println("计算结果是：" + result);
    }

    public static double calc(double num1, double num2, String operator) {
        double result = 0; // 定义一个double类型的变量记录本次运算的结果，方便返回！！
        switch (operator) {
            case "+":
                result = num1 + num2;
                break;
            case "-":
                result = num1 - num2;
                break;
            case "*":
                result = num1 * num2;
                break;
            case "/":
                result = num1 / num2;
                break;
            default:
                System.out.println("运算符有误，请重新输入");
        }
        return result;
    }
}
```

### 猜数字游戏

需求：随机生成一个1-100之间的数据，提示用户猜测，猜大提示过大，猜小提示过小，直到猜中结束游戏。

思路分析：

- 先随机生成一个1-100之间的数据。
- 定义一个死循环让用户可以一直猜测。
- 在死循环里，每次都提示用户输入一个猜测的数字，猜大提示过大，猜小提示过小，猜中则结束游戏。

Random工具类的用法：在Java中，Random 是一个工具类，它位于 java.util 包下，用于生成伪随机数。其基本用法如下面代码所示：

```Java
// 生成指定范围内的随机整数 (例如，0 到 9)
int nextIntInRange = random.nextInt(10); // [0, 10)
```

random.nextInt(10)默认生成的是 0-10（不包含10） 之间的随机数字，那如何生成1-10之间的数字呢？我们可以先生成0-9之间的数字，然后再加1：

![WWnEbz7Xcoov7VxcEu0cFPCInWh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/WWnEbz7Xcoov7VxcEu0cFPCInWh.png)

猜数字游戏，整体代码实现：

```Java
public class Practise {

    public static void main(String[] args) {
        demo4();

    }

    /*
       猜数字
       思路:
           (1) 随机生成1-100之间的数字
           (2) 循环不断录入要猜的数字,和随机数字进行对比 , 三种  猜大了, 猜小了, 猜中了
    */
    public static void demo4(){
        //1.随机1-100之间的数字, 采用Random解决
        Random random = new Random();
        //2.随机指定范围的数字
        int randomNumber = random.nextInt(1, 101);
        Scanner sc = new Scanner(System.in);
        System.out.println("本期中奖号码是:"+ randomNumber);
        //定义变量描述开始的以及结束的数字
        int start = 1;
        int end = 100;
        //3.不断的键盘录入要猜的数字
        while (true){
            System.out.println("请输入要猜的数字["+start+","+end+"]");
            int guessNumber = sc.nextInt(); // alt + enter 可以快速修复代码!!!
            //对比猜的数字和随机的数字的关系
            if(guessNumber > randomNumber){
                System.out.println("猜大了");
                // 我们需要修改end为 guessNumber - 1
                end = guessNumber - 1;
            }else if(guessNumber < randomNumber){
                System.out.println("猜小了");
                // 我们需要修改start为 guessNumber + 1
                start = guessNumber + 1;
            }else{
                System.out.println("恭喜你猜中了!!");
                break;// 结束循环
            }
        }
    }
}
```

### 验证码

需求：开发一个程序，可以生成指定位数的验证码，每位可以是**数字、大小写字母**。

![ZRmdbW3VboeEUfxpyLWccWHgnhf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ZRmdbW3VboeEUfxpyLWccWHgnhf.png)

思路分析：

- 使用for循环依次生成每位随机字符
- 并使用一个String类的变量把每个字符连接起来，最后返回该变量即可
- 随机数字：可以使用Random类，生成0-9之间的随机数
- 随机小写字母：小写字母a的ascII值是97，可以生成 97-(97+25) 之间的随机数，然后转成字符
- 随机大写字母：小写字母a的ascII值是65，可以生成 65-(65+25) 之间的随机数，然后转成字符

代码实现如下：

```Java
package com.heima.demo;

import java.util.Random;

/**
 * 案例2:开发验证码
 * 需求：开发一个程序，可以生成指定位数的验证码，每位可以是数字、大写字母、小写字母
 * getCaptcha
 */
public class Test3 {
    public static void main(String[] args) {
        //调用方法,生成验证码
        String code = getCaptcha(4);
        System.out.println(code);

        code = getCaptcha(6);
        System.out.println(code);
    }

    /**
     * 生成指定位数的验证码，每位可以是数字、大写字母、小写字母
     * @param n
     * @return
     */
    public static String getCaptcha(int n){
        //结果
        String code = "";
        //随机数对象
        Random random = new Random();
        //生成随机字符,并连接到结果code上
        for (int i = 0; i < n; i++) {
            //随机生成0-2的整数,分别代表(0-数字, 1-大写字母, 2-小写字母)
            int type = random.nextInt(3);
            //根据类型生成不同的字符
            switch (type){
                case 0:
                    //生成随机的数字0-9
                    int num = random.nextInt(10);
                    code += num;
                    break;
                case 1:
                    //生成随机的大写字母(A-65 ,  65-65+26)
                    char upperLetter = (char) (random.nextInt(26) + 65);
                    code += upperLetter;
                    break;
                case 2:
                    //生成随机的小写字母(a-97 ,  97-97+26)
                    char lowerLetter = (char) (random.nextInt(26) + 97);
                    code += lowerLetter;
            }
        }
        return code;
    }
}
```

## 相关笔记

- [[Memo/development/programming/java/java-basics/03-program-flow-control-quiz.md|03 流程控制·问答]]
- [[Memo/development/programming/java/java-syntax-and-control-flow/08-loops-and-control-flow-practice.md|循环结构与流程控制综合实战]]

---

> 来源：飞书《讲义》知识库 · day03-程序流程控制 · [原文](https://baxsu7o100e.feishu.cn/wiki/D5v4wmG0Fi4X0Kkz0C3ciAsjnCg)
