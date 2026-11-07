---
type: memo
date: 2026-11-06
aliases:
  - "03 程序流程控制·笔记"
---

# 03 程序流程控制·笔记



## 课堂问答：键盘录入与流程控制

**题目: 根据你的成绩给予奖励或是惩罚。(标注: 成绩请输入键盘录入接受)**

![](../../../../../Attachments/development/programming/java/ji-chu-yu-fa-yu-liu-cheng-kong-zhi/Fib2bLeicoqg3mxTBghcrCn5n28.png)

**目标: 掌握键盘录入和if选择结构的灵活使用**

```Java
import java.util.Scanner;
public static void main(String[] args) {
scoreSelect();
}

private static void scoreSelect() {
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入您的成绩：");
    int score = sc.nextInt();
    if (score < 0 && score > 100){
        System.out.println("请输入0~100范围内的正确分数！");
    }else if (score >= 95){
        System.out.println("恭喜您，奖励一辆三八大杠");
    }else if (score >= 90){
        System.out.println("表现不错，奖励去游乐园一次");
    }else if (score >= 80){
        System.out.println("请继续努力，奖励变型金钢一个");
    }else if (score < 80){
        System.out.println("奖励抗击打能力测试一套！");
    }
}
```

**题目: 请书写for循环的基本语法格式, 并使用文字描述循环的执行流程**

**目标: 掌握for循环的语法格式,以及执行流程**

```Java
格式：
    for(初始化语句;循环条件;迭代语句){
        循环体重复执行语句；
    }
执行流程：
    根据定义的初始化语句，进行入循环条件，进行逻辑判断，如果判断结果为true则执行循环语句，执行完成后执行迭代语句，对初始化语句中的变量赋值，
    再次进入循环条件中进行逻辑判断，如果结果还为true则继续执行循环体执行语句，直至初始化语句中的值不满足循环条件（即逻辑判断结果为false），
    才能结束循环，并执行for循环后续语句。
```

**题目: 请书写switch分支的格式:**

**目标: 掌握switch语句的格式**

```Java
格式：
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

**题目: 请书写if分支有三种形式语法格式：**

**目标: 掌握if语法的写法**

```Java
if格式一：
    if(条件){
        执行语句；
    }

if格式二：
    if(条件1){
        执行语句；
    }else if(条件2){
        执行语句；
    }else if(条件3){
        执行条件；
    }

if格式三：
        if(条件1){
        执行语句；
    }else if(条件2){
        执行语句；
    }else if(条件3){
        执行语句；
    }
    ...
    else{
        执行语句；
    }
```

**题目: 请书写while循环的语法格式, 并使用while循环计算1-100之间的偶数和**

**目标: 掌握while循环的语法格式**

```Java
while格式：
    while（循环执行条件）{
        循环执行语句；
        迭代语句；
    }

    public class NumCount {
    public static void main(String[] args) {
        numCount1();
    }
    private static void numCount1() {
        int count = 0;
        int num = 1;
        while (num < 100){
            if (num % 2 == 0)
                count += num;
            num++;
        }
        System.out.println(count);
    }
```

**题目: 求[1, 10]之间的数据和，并把求和结果在控制台输出**

**目标: 掌握for循环和变量的综合应用**

```TypeScript
public static void main(String[] args) {
    numCount2();
}

private static void numCount2() {
    int count = 0;
    for (int i = 0; i < 10; i++) {
        count += i;
    }
    System.out.println("和为" + count);
}
```

**题目: 观察代码请选择正确结果**

```Java
public class Demo {
   public static void main(String[] args) {
    for (int i = 1; i <= 10; i++) {
      if (i % 3 == 0) {
        System.out.print(i + "  ");
      }
    }
  }
}
```

**目标: 掌握以及理解for循环基本执行流程**

A、2 4 6 8 10

B、1 3 6 9

C、3 6 9

D、1 2 3 6 9

```Java
C
```

**题目: 观察以下代码 , 关于for循环语句,输出结果正确的是**

```Java
public class Demo {
  public static void main(String[] args) {
    int sum = 0;
    for (int i = 1; i <= 5; i++) {
      sum += i;
      System.out.print(sum + " ");
    }
  }
}
```

**目标: 掌握以及理解for循环基本执行流程**

A、15

B、1 3 6 10 15

C、编译报错

D、5

```Java
B
```

**题目:**

**中国有闰年的说法。闰年的规则是：四年一闰，百年不闰，四百年再闰。**

**（年份能够被4整除但不能被100整除算是闰年，年份能被400整除也是闰年）。请打印出1988年到2024年的所有闰年年份。**

**目标: 掌握for循环,选择结构, 变量, 运算符等综合应用**

```Plain
private static void leapYear() {
    for (int i = 1988; i <= 2024; i++) {
        if (i % 4 == 0 && i % 100 != 0 || i % 400 == 0){
            System.out.println("闰年：" + i);
        }
    }
}
```

**题目:**

**小明手中有一张10元纸币。需要到商店兑换零钱。商店只有1元纸币和5角硬币。那么请列举出所有的兑换方式。**

**打印效果如下：**

换1元纸币0张，5角硬币20个

换1元纸币1张，5角硬币18个

换1元纸币2张，5角硬币16个

换1元纸币3张，5角硬币14个

... ...

换1元纸币10张，5角硬币0个

**目标: 掌握for循环,选择结构, 变量, 运算符等综合应用**

```Java
public class Test {
    public static void main(String[] args) {

        for (int i = 0; i <= 10 ; i++) {
            int count = (10 -i) * 2;
            System.out.println("换1元纸币"+i+"张，5角硬币"+count+"个");
        }
    }
}

private static void moneyType() {
    int money = 10;
    for (int i = 0;i <= 10;i++ ){
        int ruslt = (money - i * 1) * 2;
        System.out.println("换1元纸币" + i + "张，5角硬币" + ruslt + "个");
    }
}
```

**题目:**

**在中国历法中有十二生肖年份,2024年是龙年,请在控制台输出从1949年(包含)到2024年(包含)中所有是龙年的年份。**

**目标: 掌握for循环,选择结构, 变量, 运算符等综合应用**

```Plain
private static void animalsYear() {
    for (int i = 2024; i >= 1949; i -= 12) {
        System.out.println(i + "是龙年");
    }
}

private static void animalsYear2() {
    int year = 2024;
    for (int i = 2024; i >= 1949; i--) {
        if ((year- i) % 12 == 0)
            System.out.println(i + "是龙年");
    }
}
```

**题目:**

**某商场推出幸运抽奖活动，抽奖规则如下：**

**(1)键盘录入四位数字(1000-9999的数字),作为顾客的会员卡号**

**(2)该会员卡号(键盘录入的四位数字)各位数字之和大于20，则为幸运客户, 请编程验证用户输入的是否是幸运客户**

**目标: 选择结构, 变量, 运算符等综合应用**

```Java
public class Test {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入你的会员卡号");
        int cardId = sc.nextInt();
        //求出每一位int ge = cardId % 10;
        int shi = cardId / 10  % 10;
        int bai = cardId / 100 % 10;
        int qian = cardId / 1000 % 10;
        if(ge + shi + bai + qian >20 ){
            System.out.println("您是幸运客户");
        }else{
            System.out.println("您是普通客户");
        }
    }
}

public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入您的会员卡号：");
    int card = sc.nextInt();
    luckyVip(card);
}
/*
      某商场推出幸运抽奖活动，抽奖规则如下：
          (1)键盘录入四位数字(1000-9999的数字),作为顾客的会员卡号
          (2)该会员卡号(键盘录入的四位数字)各位数字之和大于20，则为幸运客户, 请编程验证用户输入的是否是幸运客户
 */

private static void luckyVip(int card) {
    int ge = card % 10;
    int shi = card / 10 % 10;
    int bai = card / 100 % 10;
    int qian = card /1000 % 10;
    if (card < 1000 && card > 9999){
        System.out.println("卡号为1000-9999的数字，请输入正确的会员卡号！");
    }else if (ge + shi + bai + qian == 20){
        System.out.println("会员卡号为：" + card + "的用户，成为幸运客户");
    }
}
```

**题目: 生成 10个 [40,45] 之间的随机数输出到控制台**

**目标: 掌握随机数随机指定范围的数字**

```Java
private static void luckyNum() {
    Random rn = new Random();
    for (int i = 1; i <= 10; i++) {
        int num = rn.nextInt(40,46);
        System.out.println(num);
    }
}
```

**题目: 随机生成一个[1,100]之间的数据，提示用户猜测，猜大提示过大，猜小提示过小，直到猜中结束游戏。**

**目标: 掌握随机数, 键盘录入, 运算符, 选择结构, 变量, 循环等综合练习**

```Java
private static void luckyNum2() {
    Random rn = new Random();
    int num = rn.nextInt(1,101);
    Scanner sc = new Scanner(System.in);
    while (true){
        System.out.println("请输入您要猜的数字：");
        int guessnum = sc.nextInt();
        if (guessnum > num){
            System.out.println("大了，请重新猜！");
        }else if (guessnum < num){
            System.out.println("小了，请重新猜！");
        }else{
            System.out.println("恭喜你，猜对了，正确数字是：" + num);
            break;
        }
    }
}
```

**题目:**

**系统密码是520，请用户不断的输入密码验证，**

- **验证失败输出：密码错误**
- **验证成功输出：欢迎进入系统，并停止程序**

**目标: 掌握键盘录入, 选择结构, 死循环的使用场景**

```Java
private static void luckyNum3() {
    Scanner sc = new Scanner(System.in);
    while (true){
        System.out.println("请输入密码：");
        int password = sc.nextInt();
        if (password == 520 ){
            System.out.println("欢迎进入系统，并停止程序");
            break;
        }else {
            System.out.println("密码错误");
        }
    }
}
```

**题目:**

**世界最高山峰珠穆朗玛峰高度是：8848.86米=8848860毫米，假如我有一张足够大的纸，它的厚度是0.1毫米。请问：该纸张折叠多少次，可以折成珠穆朗玛峰的高度？**

**目标: 掌握for循环,选择结构, 变量, 运算符等综合应用**

```Java
private static void mountCount1() {
    int count = 0;
    int mount = 8848860;
    for (double paper = 0.1; paper < mount; paper *= 2) {
        System.out.println("纸的厚度为"+paper);
        count +=1 ;
    }
    System.out.println("需要折：" + count + "次数");
}
```

**题目:**

**在控制台输出所有的“水仙花数”，水仙花数必须满足如下2个要求：**

1. **水仙花数是一个三位数**
2. **水仙花数的个位、十位、百位的数字立方和等于原数例如:**

   ```
    **153是一个水仙花数，因为153=1^3＋5^3＋3^3**
   ```

**目标: 掌握for循环,选择结构, 变量, 运算符等综合应用**

```Java
public static void flowerNum(){
    for(int i = 100;i <= 1000;i++){
       int ge = i % 10;
       int shi = i / 10 % 10;
       int bai = i / 100 % 10;
       int flower = ge * ge * ge + shi * shi * shi + bai * bai * bai;
       if(flower == i) {
           System.out.println(i);
       }
    }
}
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 基础语法与流程控制 / 03 程序流程控制·笔记 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Mhwywm1WqiumF1ktq6NcJaSanMb)
