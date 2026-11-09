---
type: memo
date: 2026-11-06
aliases:
  - "02-java-ji-chu-yu-fa"
  - "02 java基础语法·笔记"
---

# 02 java基础语法·笔记
> 讲义全文：[[Memo/development/programming/java/basics-course/day02-java-syntax-basics.md|Day02 基础语法讲义]]



## 课堂问答：类型转换与基础语法

**题目: 请书写强制类型的语法格式**

**目标: 掌握强制类型转换的格式**

```Java
在java中是不允许大类型和小类型一起运算，如果非要使用小类型容器来容纳结果，必须使用强制类型转换的语法格式才行。
格式：
    目标类型 变量名= (目标类型)结果;
例：
byte a = 8;
int b = 10;
c = (byte)(a+b)

```

**题目: 请书写出java的八种基本类型的数据类型**

**目标: 能够掌握八种基本数据类型的书写格式**

```Java
整型：
    byte：
        格式：byte 变量名 = 整数值 长度范围：-128~127 
    short：
        格式：short 变量名 = 整数值
    int：
        格式：int 变量名 = 整数值
        整数型值，一般默认使用int型
    long：
        格式：long 变量名 = 整数值
浮点型：
    float：
        格式：float 变量名 = 小数值
    double：
        格式：double 变量名 = 小数值
字符型：
    char：
        格式：char 变量名 = '单个字符值'
    String：
        格式:String 变量名 = "0~多个字符"
布尔型：
    boolean：布尔型只有两个值：true和false ，一般用于逻辑运算和流程控制
```

**题目:**

**某小伙想定一份外卖，商家的优惠方式如下：**

**鱼香肉丝单点24元，油炸花生米单点8元，米饭单点3元。**

**订单满30元8折优惠。鱼香肉丝优惠价16元，**

**但是优惠价和折扣不能同时使用。**

**那么这个小伙要点这三样东西，最少要花多少钱？**

**目标: 掌握常见的运算符**

```Java
public static void goShopping(){
    // 1.定义 鱼香肉丝 、 油炸花生、米饭 三种菜品的价格
    int fish = 24;
    int chicken = 8;
    int rice = 3;
    // 2.定义sum变量 存放三个菜品全购买的价格
    int sum = fish + chicken + rice;
    // 3.判断 优惠价和折扣不能同时使用时 使用那种方式的价格更低 并赋值给新变量
    double moeny = (sum * 0.8) < (sum - 16) ? (sum * 0.8) : (sum - 16);
    // 4.打印输出最终结果
    System.out.println("这个小伙要花："+moeny);
}
```

**题目:**

**红茶妹妹有21元钱，她攒了几天钱之后自己的钱比原来的两倍还多三块。绿茶妹妹有24元钱，她攒了几天钱之后自己的钱正好是原来的两倍。那么红茶和绿茶现在的钱一样多，请问对么？**

**目标: 掌握常见的运算符**

```Java
public static void priceEstimate(){
    //1.先定义两个变量，存放两个妹妹攒钱的余额
    int redteamm = 21;
    int greenteamm = 24;
    //2.判断两个妹妹现在的余额是否一样多
    String teamm_money = (redteamm * 2 + 3) == (greenteamm * 2) ? "对的" : "错，不";
    //3.打印结果
    System.out.println(teamm_money+"一样多");
}
```

**题目:**

**身高是具有遗传性的，子女的身高和父母的身高有一定的关系。假定，父母和子女的身高遗传关系如下：**

**儿子身高（厘米）＝(父亲身高＋母亲身高) ×1.08÷2**

**女儿身高（厘米）＝(父亲身高×0.923＋母亲身高) ÷2**

**现有父亲身高177CM,母亲身高165CM。求子女身高分别预计为多少？**

**目标: 掌握常见的运算符**

```Java
public static void main(String[] args) {
    //1.先定义父亲身高和母亲身高
    int f = 177;
    int m = 165;
    //5.根据方法参数列表 输入调用参数
    heightCount(f, m);
}
//2.定义子女身高计算方法
public static void heightCount(int f, int m){
    //3.定义变量 设立子女身高计算公式 
    double son = (f + m) * 1.08 / 2;
    double daughter = (f * 0.923 + m) / 2;
    //4.打印结果
    System.out.println("儿子身高为:"+son);
    System.out.println("女儿身高为:"+daughter);

}
```

**题目: 键盘录入三个整数, 请求三个数中的最小值**

**目标: 掌握键盘录入, 比较运算符, 三元运算符, 变量**

```Java
public static void main(String[] args) {
    //1.键盘录入需要判断的三个整数
    System.out.println("请输入三个整数：");
    Scanner sc = new Scanner(System.in);
    //2.调用打印方法
    int a = sc.nextInt();
    int b = sc.nextInt();
    int c = sc.nextInt();
    //6.调用方法 定义变量 接收 方法返回的最小值
    int min_num = minNum(a,b,c);
    //7.打印结果
    System.out.println("最小值是："+min_num);
}
//3.定义比较方法
public static int minNum(int a, int b, int c ){
    //4.定义三元判断 三个数中的最小值
    int min_num = (a < b ? a : b) < c ? (a < b ? a : b) : c;
    //5.返回最小值
    return min_num;
}
```

**题目: 键盘录入一个三位数,请进行数值拆分，并将个位、十位、百位打印在控制台**

**目标: 掌握键盘录入, 算数运算符, 变量等知识**

```Java
public static void main(String[] args) {
    //1.键盘录入一个三位数
    System.out.println("请输入一个三位数：");
    Scanner sc = new Scanner(System.in);
    int a = sc.nextInt();
    //4.调用方法进行数值打印
    numPrint(a);
}
public static void numPrint(int a){
    //2.进行数值拆分
    int ge = a % 10;
    int shi = a / 10 % 10;
    int bai = a / 100 % 10;
    //3.打印结果
    System.out.println("个位是:"+ ge);
    System.out.println("十位是:"+ shi);
    System.out.println("百位是:"+ bai);
}
```

**键盘录入用户密码, 如果密码为 111111, 程序输出密码正确，否则输出密码有误 (标注: 这里密码使用int类型即可)**

**目标: 掌握常用的比较运算符**

```Java
public static void main(String[] args) {
    //1.键盘录入提示用户输入密码
    System.out.println("请输入密码：");
    Scanner sc = new Scanner(System.in);
    //2.将获取的用户密码存储到变量a中
    int a = sc.nextInt();
    //6.调用判断方法 获取结果
    estimatePassword(a);
}
//3.定义方法 判断密码是否正确
public static void estimatePassword(int a){
    //4.三元 判断密码是否正确
    String estimate = a == 111111 ? "密码正确" : "密码有误" ;
    //5.打印判断结果
    System.out.println(estimate);
}
```

**题目: 现有代码如下, 请回答打印的结果, 并描述中间的运算过程**

**目标: 掌握隐藏的强制类型转换**

```Java
int a = 10;
double b = 12.3;
a += b;
System.out.println(a);
```

```Java
a += b;
等同于：
    a =(int)(a + b);
因为要赋值给变量a，所以最终结果也a的类型  
```

**题目: 现有代码如下, 想要运算结果是2.5， 代码应该怎么改 ? (直接题目上改即可)**

**目标: 掌握/号的特点**

```Java
public class Test1 {

    public static void main(String[] args) {
        int i = 5 / 2;
        System.out.println(i);
    }
}
改：
public class Test1 {

    public static void main(String[] args) {
        double i = 5 / 2;
        System.out.println(i);
    }
}
```

**题目: 描述下算数运算符中 , / 和 % 的特点**

**目标: 能够区分/和%运算符区别**

```Java
/的特点是取商 且取值为整数 例：10 / 3 = 3
%的特点是取余，也称取模，是除数除以被除数的剩余值，如果能被整除则余0。例：10 % 3 = 1
```

**题目: 观察代码,说出rs1, rs2, rs3 , rs4的结果**

**目标: 掌握++,--放前放后的区别**

```Java
int a = 20;

int rs1 = a++; //rs1为20
    //此时a为21
int rs2 = a--;//rs2为21
    //此时a为20
int rs3 = ++a;//rs3为21
    //此时a为21
int rs4 = --a;//rs4为20
    //此时a为20
++和--的前后区别：
            在前的，根据从左到右的计算原则，应该先计算后赋值给变量。
            在后的，根据从左到右的计算原则，应该先赋值给变量，再进行计算，
                且计算后的值会赋值给自己，下次参与运算将以此值继续运算。
```

**题目: 观察代码, 请说明控制台打印的结果:**

```Java
System.out.println("hello " + 100 + 'b');//打印结果为：hello 100b
System.out.println(100 + 'b' + " world");//198world
//根据从左即右的原则，先使用什么类型的数值就以什么类型的数值进行运算，如果遇到字符串才需要拼接
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 基础语法与流程控制 / 02 java基础语法·笔记 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/GwJCwdhk1iNK3dk1oTdcFA35n1f)
