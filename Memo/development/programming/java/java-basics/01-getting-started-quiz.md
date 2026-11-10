---
type: memo
date: 2026-11-06
aliases:
  - "01-ru-men"
  - "01 入门·笔记"
---

# Java 快速入门·问答
> 系统详解：[[Memo/development/programming/java/java-basics/01-getting-started.md|Java 快速入门]]



## 问答：IDEA 使用与入门案例

**题目: 请简述使用idea编写入门案例时, 首先先创建哪些东西?**

**目标: 掌握idea的基本使用步骤**

> 答：  
> 1、先创建一个空Java项目  
> 2、在空项目中创建空模块  
> 3、在模块中的src下创建包，用于存放代码

> 注意：

> 在创建包时，包名注意格式，公司域名（反转）加技术名,包名注意要全小写

> 格式：cn.itcast.hello-world  
> 4、在包中创建Java类文件，类名要符合编程命名规范

**题目: 请简述什么是变量? 以及其作用?**

**目标: 理解变量的概念和其作用**

> 答：  
> 概念：

> 变量是内存中的一块存储数据的区域  
> 作用：

> 变量的作用是 用来存储数据的容器，方便在程序代码中对不同数据的管理和使用

**题目: 请简述什么是标识符? 以及标识符的规则和规范?**

**目标: 掌握标识符的命名规则和规范**

> 答：  
> 标识符是除系统本身定义的单词之外，自己命名的单词或词组，用于为变量、函数、类、模块等程序元素命名。  
> 规则：

> 只能由字母、数字、下划线和美元符号（\$）组成；

> 不能以数字开头；

> 不能与语言的关键字或保留字冲突；

> 并且标识符中不允许包含空格。

> 尽量避免使用汉字命名（最好禁止使用）

> 注意：标识符是区分大小写的。  
> 命名规范：

> 小驼峰命名法：每一个单词首字母小写，第二个单词及后续单词首字母均大写，如studentName，多用于变量、函数名。

> 大驼峰命名法：每一个单词及后续单词首字母均大写。如StudentName。通常用于类名、类型名。

> 下划线命名法：所有字母小写，每两个单词中间用下划线隔开。如 student_name。

**题目: 请简述java的关键字的概念?以及有什么特点?**

**目标: 掌握java关键字的概念以及其特点**

> 答：  
> 概念：

> Java关键字是Java语言中具有特殊含义、被编译器预先定义的单词，是构成Java语法的基础框架。  
> 特点：

> 关键字数量固定，且单词均为小写

> 绝对不能用作变量名、类名、方法名等自定义标识符

> 用于定义数据类型、流程控制、类与类方法等

**题目: 请简述java的注释类型都有哪些?**

**目标: 掌握注释的写法**

> 答：  
> 注释本身是不会对代码产生影响的，主要是对代码内容或功能的解释，增加代码的可读性  
> 单行注释：

> 格式：

> // 使用双斜杠注释内容  
> 多行注释：

> 格式：

> /\*

> 使用多行注释，以/\*开始 \*/结尾，在该区域范围内的内容不会对代码运行产生影响，也不会被编译。

> \*/  
> 文档注释：

> 略。

**题目: 请描述java的基本数据类型的分类?**

**目标: 掌握基本类型的划分**

> 答：  
> java类型分类：

> 整型：

> byte：长度-128-127 ，长度短，使用场景非常有限

> short：长度较短，容易长度溢出，使用场景相交于int，有一定的局限

> int（默认）：长度较长，足够日常的使用

> long：长度足够长，一般用于超长整数数据，使用时可以在末尾追加L（可以用小写，但不建议使用，和数字1容易混淆），来取消错误提醒

> 浮点型：

> float：单精度，只能保留两位小数，如果超出则会四舍五入到小数点后第二位，日常使用容易造成精度损失。

> double（默认）：双精度，容纳位数较长，通常默认不易损失精度。

> 字符型：

> char：只能存放单个字符，用单引号将字符括住 'x' 。

> 布尔型：

> boolean：只有true和false两个值，主要用于逻辑判断

**题目: 请按照注释要求编写代码：(直接修改即可)**

**目标: 考察变量的定义以及基本使用, 代码的执行流程的理解**

```Java
public class Main {
   public static void main(String[] args) {
      String a = "爱我中国";
      // 请修改下面的3行输出语句代码，以输出爱我中国
      System.out.println(a);
      System.out.println(a);
      System.out.println(a);

      int num = 666;
      // 请编写你的代码，以输出777
      int x = 777;
      System.out.println(x);
      System.out.println(num);

      // 请自定义一个变量，并保证输出语句输出8.8
      double b = 8.8;
      System.out.println(b);
   }
}
```

**题目: Java跨平台操作是通过什么来实现的?**

**目标: 掌握java跨平台的根本原理**

> 答：  
> 通过jvm来实现跨平台编译，jvm将开发者编写的代码，编译成字节码文件，即机器可以直接运行的通用语言。

**题目: 请简单描述jdk,jre,jvm三者的关系和各自的作用?**

**目标: 掌握jdk开发工具包到底包含了哪些部分**

> 答：
> 
> - JDK：软件开发者的工具箱JDK是面向程序员的软件开发工具包。编写、编译和调试Java代码，就必须安装它。它不仅包含了完整的JRE（保证程序可以运行），还提供了一系列强大的开发工具和丰富的类库，这些都是开发过程中不可或缺的。其核心组件包括：

> - 开发工具：如编译器`javac`（将`.java`源码文件编译成`.class`字节码文件）、调试器`jdb`、打包工具`jar`等。
> - Java API：大量预先编写好的类库，为你的程序提供基础功能。
> - JRE：确保开发过程中可以立即运行和测试程序。
> - JRE：程序用户的运行环境JRE是Java运行时环境。如果一个用户只是想运行一个已经开发好的Java程序（比如一个.jar文件），那么他的电脑上只需要安装JRE就够了，而无需安装庞大的JDK。可以把它理解为Java程序的“播放器”。它主要由两部分构成：

> - JVM：负责实际执行字节码的“引擎”。
> - Java核心类库：JVM在执行程序时需要调用的基础库文件（如`rt.jar`）。没有这些类库，JVM无法正常工作。
> - JVM：跨平台能力的核心JVM是Java虚拟机。它是实现Java著名特性“一次编写，多平台运行”的关键。负责将编译生成的与平台无关的字节码解释/编译成所在操作系统能够识别的机器码并执行。它还自动处理内存管理、垃圾回收等底层复杂任务。JVM是平台相关的，Windows、Linux、macOS都有各自版本的JVM，但它们都遵循相同的规范，从而保证了同一份字节码在不同系统上能有一致的表现

**题目: 以下在Java中定义变量的方式正确的有**

**目标: 掌握变量定义的格式**

A、int a = 10;

B、int a ; a = 10 ;

C、int a = 10, b = 20, c = 30;

D、int a = 10, int b = 20,int c = 30;

> 答：  
> A、B、C

**题目: 以下属于常量(字面量)的有**

**目标: 掌握字面量的写法**

A、class

B、666

C、5.5

D、""

E、''

F、'牛'

G、"牛"

H、null

I、true

> 答：  
> A、B、C、D、F、G、H、I

**题目: 以下变量的标识符写法正确的是**

**目标: 掌握标识符的注意事项**

A、#AAA

B、2B

C、class

D、Class

> 答：  
> D

**题目: 以下程序写法正确的是**

**目标: 掌握入口函数的写法**

```Java
A.
public class HelloWorld{
        public static void main(String args){
                System.out.println("HelloWorld");
        }
}

B.
public class HelloWorld{
        public static void main(String[] args){
                System.println("HelloWorld");
        }
}

C.
public class HelloWorld{
        public static void main(String[] args){
                System.out.println("HelloWorld");
        }
}

D.
public class HelloWorld{
        public static void main(String[] args){
                System.out.println("HelloWorld")
        }
}
```

> 答：  
> C

**题目: 请完成常见的基本数据类型的变量的定义案例**

**目标: 掌握变量定义的基本格式**

```Java
//定义一个百位以内的短变量 九九乘法口诀表最大结果
byte count = 81;
//定义一个短整型变量 月份
short month = 12;
//定义一个整数变量 年龄
int x = 18;
//定义一个长整型变量 人口
long chinapeople = 1300000000L;
//定义一个浮点型变量 身高
float y = 175.5F;
double z = 175.5;
//定义一个单字符类型的变量 姓氏
char w = '王';
//定义一个逻辑变量 立flag 打脸愉快
boolean flag = true;
boolean flag2 = false;
```

**请使用记事本完成HelloWorld案例**

```TypeScript
public class HelloWorld {
    //psvm 快捷方式 或 main
    public static void main(String[] args) {
        //sout 快速补全打印代码
        System.out.println("Hello World");

    }
}
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 基础语法与流程控制 / 01 入门·笔记 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/GKXRw60hviqTo2k6vUnccvKJndh)
