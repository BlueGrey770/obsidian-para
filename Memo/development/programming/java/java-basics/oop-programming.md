---
type: memo
date: 2026-11-09
aliases:
  - "封装与构造器"
---

# 面向对象编程

## 使用对象存储数据

同学们，我们前面学过了：变量、数组，都是用来存储数据的。变量可以记住一个单个的数据，数组可以存储一批同类型的数据。那么如果要存储一个事物的整体数据，比如我们当前有一个需求：在程序中需要存储一些明星的个人资料，那应该怎么存储呢？ 这个问题可以用我们今天学习的面向对象编程来解决。

![Px8MbY3M2oCdfax1jfEcZlKEnJc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Px8MbY3M2oCdfax1jfEcZlKEnJc.png)

![ZNqkbm0LQoeHiyxzIdxcYggCnef](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ZNqkbm0LQoeHiyxzIdxcYggCnef.png)

在java中提供了对象这种机制，可以方便的存储这种场景的数据。那么使用对象是如何存储数据的呢？我们来看下下面的代码：

```Java
public static void main(String[] args) {
    // 创建赵丽颖的Star对象并设置属性
    Star s1 = new Star();
    s1.name = "赵丽颖";
    s1.age = 36;
    s1.height = 165.0;
    s1.weight = 44.6;

    // 创建杨幂的Star对象并设置属性
    Star s2 = new Star();
    s2.name = "杨幂";
    s2.age = 37;
    s2.height = 166.5;
    s2.weight = 45.0;
 }
```

以上代码创建了2个明星对象：s1、s2，每个对象中可以存储该明星的所有数据（姓名、年龄、身高、体重），这些数据合起来就是一个完整的明星的数据资料，这些数据的组合体我们称为对象。

接下来有个问题：上面的代码 Star s1 = new Star(); 中， Star是什么？好像java中没有这个数据类型。其实这个是我们自己编写的数据类型，在这个数据类型中可以自己定义其中包含的数据（姓名、年龄、身高、体重），这个自定义的数据类型，在java中称为**类**。

在java中我们要使用对象，需要遵循以下两个步骤：

1. 先设计对象的模板，也就是对象的设计图：类
2. 通过new关键字，每new一次就得到一个新的对象，对象中可以存储数据

那么应该如何编写自己的数据类型（类）呢？下面通过代码进行演示：

```Java
// 明星类
public class Star {
    String name;
    int age;
    double height;
    double weight;
 }
```

以上代码创建了一个明星类（Star），其中定义了这个数据类型中可以存储的数据（姓名、年龄、身高、体重），这些具体的数据，也称为类的**属性**。

创建了类之后，就可以通过new关键字来创建对象了，示例代码如下：

```Java
public static void main(String[] args) {
    // 创建赵丽颖的Star对象并设置属性
    Star s1 = new Star();
    s1.name = "赵丽颖";
    s1.age = 36;
    s1.height = 165.0;
    s1.weight = 44.6;

    // 创建杨幂的Star对象并设置属性
    Star s2 = new Star();
    s2.name = "杨幂";
    s2.age = 37;
    s2.height = 166.5;
    s2.weight = 45.0;
 }
```

以上代码通过new关键字，创建了两个明星类（Star）的对象：s1、s2。 这两个对象中，都可以存储4个属性（姓名、年龄、身高、体重），如此一来，我们只用两个变量（s1、s2）就可以存储以上全部数据了，与变量、数组的形式对比，代码更简洁，变量更好管理。

## 使用对象处理数据

上面我们使用对象实现了数据的存储，还可以提供功能。假设我们有以下需求：

- 需求描述：请用面向对象编程的方式，存储如下两个学生的信息,并要求打印出每个学生的总成绩和平均成绩

![CKdyb1a9VoCDcqxYG2scsB7Xnoh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/CKdyb1a9VoCDcqxYG2scsB7Xnoh.png)

在这个需求中，我们不但需要存储数据（学生姓名、语文成绩、数学成绩），还需要打印成绩（总成绩、平均成绩）。那么我们的第一印象，是按如下编码实现此功能：

1. 编写学生类：用于存储数据

```Java
package com.itheima.demo1;

// 学生类
public class Student {
    String name;
    double chinese;
    double math;
}
```

1. 使用学生类：创建对象并保存数据

```Java
package com.itheima.demo1;

public class StudentTest {
    public static void main(String[] args) {
        // 目标：创建学生对象存储学生数据
        Student s1 = new Student();
        s1.name = "播妞";
        s1.chinese = 100;
        s1.math = 100;
        System.out.println( s1.name + "的总成绩:" + (s1.chinese + s1.math) );
        System.out.println( s1.name + "的平均成绩:" + (s1.chinese + s1.math)/2 );

        // 目标：创建学生对象存储学生数据
        Student s2 = new Student();
        s2.name = "播仔";
        s2.chinese = 50;
        s2.math = 100;
        System.out.println( s2.name + "的总成绩:" + (s2.chinese + s2.math) );
        System.out.println( s2.name + "的平均成绩:" + (s2.chinese + s2.math)/2 );
    }
}
```

1. 运行程序，结果如下：

![QDM6bKLLboIfRRxUmpBczsnVnFb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/QDM6bKLLboIfRRxUmpBczsnVnFb.png)

以上示例中，程序运行结果虽然正确，但我们的代码不够简洁。主要是：StudentTest 类中的第11、12行与第19、20行计算并输出成绩的代码重复。试想：如果有几十个学生对象，计算并输出成绩的代码岂不是要复制几十遍？代码太不优雅了，而且效率低，容易出错。

为了解决以上问题，java对象中提供了一个机制：允许将数据的操作，也写到类中，最后可以通过对象调用这些操作（也称为功能）。下面我们对学生类的代码进行修改：

```Java
package com.itheima.demo1;

// 学生类
public class Student {
    String name;
    double chinese;
    double math;

    // 计算并输出总成绩
    public void printAllScore(){
        System.out.println(name + "的总成绩是：" +(chinese + math));
    }

    // 计算并输出总成绩
    public void printAverageScore(){
        System.out.println(name + "的平均成绩是：" +(chinese + math) / 2);
    }
}
```

在以上代码中（第10-17行），新增了对数据的操作，其实是在类中编写了2个方法：printAllScore、printAverageScore，分别计算并输出了总成绩、平均成绩。

接下来我们就可以通过对象调用这2个操作（方法）了：

```Java
package com.itheima.demo1;

public class StudentTest {
    public static void main(String[] args) {
        // 目标：创建学生对象存储学生数据
        Student s1 = new Student();
        s1.name = "播妞";
        s1.chinese = 100;
        s1.math = 100;
        s1.printAllScore(); //计算并输出总成绩
        s1.printAverageScore(); //计算并输出平均成绩

        // 目标：创建学生对象存储学生数据
        Student s2 = new Student();
        s2.name = "播仔";
        s2.chinese = 50;
        s2.math = 100;
        s2.printAllScore(); //计算并输出总成绩
        s2.printAverageScore(); //计算并输出平均成绩
    }
}
```

以上代码的输出结果如下：

![WVnBb7lj7od6zKxdBxhcguo0n2d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/WVnBb7lj7od6zKxdBxhcguo0n2d.png)

可见结果与第一个版本代码的结果是相同的，也是正确的，但代码就简洁多了，避免了代码的重复。

总结： 可以把数据（属性）和对数据的处理（方法）放到同一个类中去，方便代码的复用。**对象不但能存储数据，还能处理数据**。

## 对象到底是啥？

### 对象与类

前面我们通过示例，直观的认识了一下对象。接下来我们从理论层面来看一下到底什么是对象？对象在内存中是如何存储的？

对象本质上是一种特殊的数据结构（可以理解成一张表）。在一个空白的表中填上了数据，这个表格就是独一无二的对象。

![Xe8lbqUGYo0n6VxVFfkcgmGHnWg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Xe8lbqUGYo0n6VxVFfkcgmGHnWg.png)

而对象又是通过类来创建的，class也就是类，也称为对象的设计图（或者对象的模板）。类就类似于空白表格，我们得先设计出空白表格，然后才能填写数据创建出一个个包含数据的表格对象。

![HsG8bbP6soOusYxEgLfcjJjWn5f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/HsG8bbP6soOusYxEgLfcjJjWn5f.png)

### 对象在内存中的存储

当我们通过new关键字创建对象时，jvm会将对象存放在内存中。在jvm中内存按存储的数据不同，划分为多种类型的内存区域，我们先来认识下jvm内存中的：栈、堆、方法区。这3块内存区域是jvm中最重要的内存，接下来我们通过一个示例来理解下，java中的各种数据（类、变量、对象）各自存放在哪块内存区域，从而对对象有个更深入的了解。

假设有以下学生类：

```Java
package com.itheima.demo1;

// 学生类
public class Student {
    String name;
    double chinese;
    double math;

    // 计算并输出总成绩
    public void printAllScore(){
        System.out.println(name + "的总成绩是：" +(chinese + math));
    }

    // 计算并输出总成绩
    public void printAverageScore(){
        System.out.println(name + "的平均成绩是：" +(chinese + math) / 2);
    }
}
```

然后我们通过以下代码使用学生类：

```Java
package com.itheima.demo1;

public class Test {
    public static void main(String[] args) {
        // 目标：创建学生对象存储学生数据
        Student s1 = new Student();
        s1.name = "播妞";
        s1.chinese = 100;
        s1.math = 100;
        s1.printAllScore(); //计算并输出总成绩
        s1.printAverageScore(); //计算并输出平均成绩

        // 目标：创建学生对象存储学生数据
        Student s2 = new Student();
        s2.name = "播仔";
        s2.chinese = 50;
        s2.math = 100;
        s2.printAllScore(); //计算并输出总成绩
        s2.printAverageScore(); //计算并输出平均成绩
    }
}
```

在以上代码运行过程中，数据的装载流程如下：

1. 程序从StudentTest类开始执行，此时由于要使用到Test类，此时jvm会将Test类的字节码（包含main中的代码）加载到方法区

![KwHGbqJDiokwJCx4Y1schQffnTe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/KwHGbqJDiokwJCx4Y1schQffnTe.png)

1. 字节码加载完毕后，程序开始执行第5行的main方法，此时jvm会在栈内存中为main方法开辟一块内存，用于存储main运行过程中的部分数据

![Jx5MbnzELovUZyxulalcbLybnUF](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Jx5MbnzELovUZyxulalcbLybnUF.png)

1. 程序执行第7行，创建一个学生对象s1，此时由于需要用到学生类（Student），所以jvm会先将Student类的字节码加载到方法区。接下来，通过new关键字创建了对象，此时jvm会为对象在堆内存中分配一块内存。然后代码将对象的地址赋值给了变量s1，此时s1中存放的是对象在堆内存中的地址，而变量s1本身是存放在栈内存中。注意：此时对象中的属性尚未赋值，所以各个属性都是默认值（name为null，成绩为0.0）

![RnCJbLpZWoqpUUxx31vcd7PpnMd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/RnCJbLpZWoqpUUxx31vcd7PpnMd.png)

1. 接下来执行第8-10行的代码，初始化对象的各个属性，此时内存中的对象如下：

![IA9CbhcZvoFGdJxBRjTc7agPnoc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/IA9CbhcZvoFGdJxBRjTc7agPnoc.png)

1. 执行第11-12行，此时会将成绩输出到控制台上
2. 然后执行第15-18行，逻辑与前面类似，在堆中创建了第2个学生对象，对象在堆中，但对象的地址保存到了s2变量中，s2变量是存放在栈内存中的。

![P9cHb64zZo7CKlx34BBckbZkn1b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/P9cHb64zZo7CKlx34BBckbZkn1b.png)

1. 执行第19-20行，此时会将第2个学生对象的成绩输出到控制台上

通过以上流程分析，我们可得出：

- 类的代码（字节码）是存放在方法区内存中的
- main方法运行时，jvm会给方法分配一个栈内存（一个方法对应一个栈内存）
- 对象是存放在堆内存中的
- 保存对象地址的变量（s1、s2）是存放在main方法的栈内存中的

### 面向对象编程

学习了对象和类的概念和基本语法后，接下来我们来认识一个重要概念：面向对象编程。面向对象编程（Object-Oriented Programming, OOP）在Java中是一种编程范式，它尝试将现实世界中的事物抽象为程序中的“对象”。用通俗的话来说，就像是你在构建一个虚拟的世界，在这个世界里，每样东西都是一个对象，比如人、动物、汽车等等。每个对象都有自己的属性和可以执行的动作。

比如：我们要开发一套电商系统，其中需要保存手机数据，我们就可以设计手机类，然后创建多个手机对象来保存各个不同手机的数据：

![AxObbMlNZovNHIxnTcgcS2gznod](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/AxObbMlNZovNHIxnTcgcS2gznod.png)

如果我们要开发一款游戏，其中需要保存各个人物的数据，我们就可以设计游戏人物类，然后创建多个人物对象来保存各个不同人物的数据：

![Ogg5bfIzEoXcD2xXWDHcJdaTnag](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Ogg5bfIzEoXcD2xXWDHcJdaTnag.png)

以上即为面向对象编程。面向对象编程是一种编写代码的套路，它让程序员能够更好地组织和管理复杂的软件系统，使得代码更容易被理解和维护。

简单来说：在面向对象编程中，万物皆对象，谁的数据谁存储。最终目的是为了更好的组织和管理代码。

## 类的基本语法

前面我们学习了类、对象、面向对象编程的基本知识，接下来我们系统的学习一下类的编写语法。

<callout emoji="🌷">
定义类语法格式:
public class 类名{
属性;
行为;
构造函数;
}
**属性**:  属性本质就是对象的数据, 是需要被存储记录的, 可以采用定义**成员变量**记录对象的数据
**行为**: 行为是类中处理数据的功能, 我们采用**成员方法**来书写
</callout>

**常见问题:**

- **什么是成员变量???**

定义在类中方法的变量我们称为成员变量

- **成员变量和局部变量的区别???**

  - 定义位置: 成员变量需要定义在类中方法外, 局部变量定义在类中方法内
  - 内存的位置: 成员变量在创建对象的时候在堆内存中初始化, 局部变量是在当方法调用的时候栈中为方法开辟的独立的空间内部初始化的
  - 生命周期: 成员变量随着对象的创建而创建, 随着对象的消失而消失, 局部变量大部分都是随着方法调用而存在, 随着方法调用结束而消失
  - 默认值: 成员变量由于在堆中初始化, 堆中的数据都有默认值, 局部变量在栈中初始化没有默认值
- **成员方法和之前定义的方法区别???**比我们之前的方法就少了一个static关键字, 至于static关键字是干什么的, 下面的课程会介绍

### 封装

面向对象编程有三大特征：封装、继承、多态。本小节我们来学习封装。

什么封装呢？我们先来看一个生活中的例子：比如，设计一辆汽车时，汽车的发动机、变速箱等一些零件并不需要让每一个开车的知道，所以就把它们隐藏到了汽车的内部。

把发动机、变速箱等这些零件隐藏起来，这样做其实更加安全，因为并不是所有人都很懂发动机、变速箱，如果暴露在外面很可能会被不懂的人弄坏。

在设计汽车时，除了隐藏部分零件，但是还是得合理的暴露一些东西出来，让司机能够操纵汽车，让汽车跑起来。比如：点火按钮啊、方向盘啊、刹车啊、油门啊、档把啊... 这些就是故意暴露出来让司机操纵汽车的。

![Vn6obVd8PoL3rfx57RucRkaDnhf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Vn6obVd8PoL3rfx57RucRkaDnhf.png)

通过以上示例，我们能总结出关于封装的概念：

- 将危险的、普通人不必直接使用的零件隐藏起来，就是封装。
- 将东西封装起来之后 ，需要对外提供一些普通人易用的组件（方向盘等），以方便进行操作。

在java编程中，也是如此，有一些数据，可能安全性比较高，不允许外部直接进行访问，此时我们就可以进行封装。比如以下代码：

```Java
package com.itheima.demo;
// 学生类
public class Student {
    String name;
    int age;
}

package com.itheima.demo;
public class StudentTest {
    public static void main(String[] args) {
        Student s1 =  new Student();
        s1.name = "张无忌";
        s1.age = 200;
    }
}
```

代码说明：

- 第3行：创建了学生类，第4、5行在类中声明了姓名、年龄两个属性
- 第12行：创建了一个学生对象s1
- 第14行：给学生对象s1的年龄赋值，此时我们将一个非法的年龄赋值给s1的age属性，代码也能运行。此时，第14行代码就是在学生类的外部，可以对内部数据进行任意的访问和修改，所以影响到了数据的安全。

为了提高数据的安全性，我们不应该允许在外部直接访问学生类的name、age两个属性。那如何做到这一点呢？在java中提供了 private 关键字，在属性前面加上private，此时属性就成为了类的私有成员，那么属性就不能在外面直接访问了。我们修改下学生类的代码，修改如下（每个属性前面加上private）：

```Java
package com.itheima.demo;

// 学生类
public class Student {
    private String name;
    private int age;
}
```

一旦在属性前面加上private，那么StudentTest类中的代码就会报错：age has private access，意思是age属性是使用private修饰过的，是类的私有成员，不允许在外面直接访问。

![Zxcib2p1goAIUxxdiVBcRI0JnAe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Zxcib2p1goAIUxxdiVBcRI0JnAe.png)

但我们又需要给对象的属性赋值，那应该怎么做呢？接下来，我们还需要对学生类（Student）进行改造，以便在外部可以给属性赋值，代码改造如下：

```Java
package com.itheima.demo;

// 学生类
public class Student {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String n) {
        name = n;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int a) {
        if(a> 0 && a< 200){
            age = a;
        }else {
            System.out.println("您赋值的年龄数据非法！");
        }
    }
}

package com.itheima.demo;
public class StudentTest {
    public static void main(String[] args) {
        Student s1 =  new Student();
        s1.setName("张无忌");
        s1.setAge(300);
    }
}
```

代码说明：

- 在Student类中，新增了第8-22行的代码
- 第8-10行：在类中新增了getName()访问，而且方法前面加了public修饰符，代表此方法是公有方法，允许在外部访问。在外部调用getName()方法，可以获取到对象的姓名属性值。
- 第12-14行：在类中新增了setName()访问，而且方法前面加了public修饰符，代表此方法是公有方法，允许在外部访问。如果需要在外部给对象设置姓名属性，则调用此方法即可。
- 第20-26行：新增了setAge()方法，且是公有方法，在外部可以访问，而且在setAge方法内，对外部传递过来的年龄进行了校验，如果年龄超了范围则不会将年龄保存到age属性，同时提示年龄非法！
- 第30行：调用了对象的公有方法setName()，给对象设置了姓名；
- 第31行：调用了对象的公有方法setAge()，给对象设置了年龄；

经过以上修改，此时第35行代码由于传递了非法的年龄值，在运行时会显示错误提示：

![IZVNbD0W2oDjigxniVXcWshfnDP](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/IZVNbD0W2oDjigxniVXcWshfnDP.png)

如此一来，我们就保证了数据的安全。总结一下，我们是通过以下实现封装的：

- 属性前面加private，此时属性为私有成员，外部不能直接访问数据
- 但为了让外部能设置属性值，我们提供了公有的方法，外部通过公有方法可以间接的给属性赋值
- 我们在方法中可以进行数据安全检查，防止产生非法数据。

<callout emoji="🌷">
思考:

我们在编写setName(String n)和setAge(int a) 这个时候方法的参数名违背了见名知意的准则,

但是如果我们设置setName(String name) 和 setAge(int age)的话会造成参数的变量名和成员变量名一致的问题

那该怎么办呢?
</callout>

下面是尝试修改成和成员变量一样名字的代码

```Java
package com.itheima.demo;

// 学生类
public class Student {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if(a> 0 && a< 200){
            age = age;
        }else {
            System.out.println("您赋值的年龄数据非法！");
        }
    }
}

package com.itheima.demo;
public class StudentTest {
    public static void main(String[] args) {
        Student s1 =  new Student();
        s1.setName("张无忌");
        s1.setAge(300);
    }
}
```

结果:

![DdynbDlBOo2x0vxqoCscigc5n5b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/DdynbDlBOo2x0vxqoCscigc5n5b.png)

### this

接下来我们再先认识一下this关键字，在java编程中这个关键字也会经常用到。

**（1）this是什么？**

this就是一个变量，用在方法中，可以拿到类的当前对象。

下面通过代码来理解下this：

```Java
package com.itheima.demo;

// 学生类
public class Student {
    public void printThis(){
        System.out.println(this); //此处打印this, this即为当前对象，相当于打印当前对象
    }
}
```

代码说明：

- 上面代码第6行，打印了this，this即为当前对象，相当于打印当前对象。

```Java
package com.itheima.demo;

public class StudentTest {
    public static void main(String[] args) {
        Student s1 =  new Student();
        System.out.println(s1);  //打印对象
        s1.printThis(); //调用方法，通过this打印对象

        Student s2 =  new Student();
        System.out.println(s2);
        s2.printThis();
    }
}
```

代码说明：

- 第6行： 打印s1对象
- 第7行：调用s1对象的printThis()方法，在方法中通过this打印当前对象，所谓当前对象就是通过哪个对象调用的方法，这个对象就是当前对象。此处，s1就是当前对象。
- 以上第6、7两行打印的其实都是s1对象，只是打印的方式不同而已。

上面代码运行结果如下：

![Fu8hbuSrqo9YeTxn6TRcWa9insg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Fu8hbuSrqo9YeTxn6TRcWa9insg.png)

**（2）this有什么用？**

this主要用来解决变量**名称冲突**问题。

就刚才思考的问题就可以采用this来解决

```Java
package com.itheima.demo;

// 学生类
public class Student {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if(a> 0 && a< 200){
            this.age = age;
        }else {
            System.out.println("您赋值的年龄数据非法！");
        }
    }
}

package com.itheima.demo;
public class StudentTest {
    public static void main(String[] args) {
        Student s1 =  new Student();
        s1.setName("张无忌");
        s1.setAge(300);
    }
}
```

### 构造器

好同学们，接下来我们学习一个非常实用的语法知识——叫做构造器。

关于构造器，我们掌握下面几个知识点就可以了：

1. 什么是构造器？
2. 理解构造器的作用 ？
3. 构造器有哪些注意事项？

**（1）什么是构造器？**

- 构造器其实是一种特殊的方法，但是这个方法没有返回值类型，方法名必须和类名相同。
- 如下图所示：下面有一个Student类，构造器名称也必须叫Student；也有空参数构造器，也可以有有参数构造器。
- 构造器也称为**构造方法**

```Java
public class Student {
    /** 构造器 */
    public Student() {
           ...
    }
 }
```

**（2）构造器的作用**

构造函数是用来给属性在堆中初始化的一种语法, 我们创建对象的格式:

类名 对象名 = new 构造函数(对参数赋值);

其中 new 是向堆中申请空间的关键字, 而构造函数会将类中的所有非静态的成员变量在new开辟的空间中申请变量, 而堆中开辟的空间自带默认值, 进行值的初始化, 当然构造函数可以重载, 我们可以通过带参数构造函数对数据进行显示初始化

**（3）构造器的注意事项**

- 造器的名字必须与类名完全相同。
- 构造器没有返回类型，甚至不是void。
- 一个类可以有多个构造器，只要它们的参数列表不同（这叫做构造器重载）。
- 如果你没有为类定义任何构造器，那么Java会提供一个默认的无参构造器
- 一旦定义了有参数构造器，Java就不再提供空参数构造器，此时建议自己加一个无参数构造器。

### Javabean

JavaBean 是一种符合特定规范的 Java 类，它主要用于封装数据。JavaBean 的设计目的是为了方便在不同组件之间传递数据。JavaBean也称为实体类。

编写JavaBean时，需要遵循以下规则：

1. 类必须是public修饰的, 至于为啥 ? 下一次课程会讲解权限修饰符在来理解
2. 私有属性：所有实例变量都应该是私有的（private），以实现封装性。
3. 公共无参构造器：JavaBean 必须有一个公共的无参数构造器，这样可以在没有提供额外信息的情况下创建 Bean 的实例。
4. getter 和 setter 方法：对于每个需要被外部访问或修改的私有属性，必须提供相应的 getter（获取属性值的方法）和 setter（设置属性值的方法）。这些方法通常遵循一定的命名约定：

   - Getter 方法以 `get` 开头，后跟属性名（首字母大写）。
   - Setter 方法以 `set` 开头，后跟属性名（首字母大写）。
   - 对于布尔类型的属性，可以使用 `is` 代替 `get`。

举个简单的例子，这里有一个符合 JavaBean 规范的简单 `Person` 类：

```Java
public class Person implements java.io.Serializable {
    private String name;
    private int age;

    // 无参构造器
    public Person() {}

    // Getter 和 Setter 方法
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

这个 `Person` 类就是典型的 JavaBean，因为它包含了私有属性、公共无参构造器、以及用于访问这些属性的 getter 和 setter 方法。

JavaBean（实体类）的使用场景：

- 实体类的对象只负责数据存取
- 数据的处理交给其他类的对象来完成，以实现数据和数据业务处理相分离。

总之一句话：JavaBean就是用来封装数据的。

## 综合小项目-电影信息系统

### 需求说明

需求：

- 展示系统中的全部电影信息(每部电影展示：名称、价格)。
- 允许用户根据电影编号（id）查询出某个电影的详细信息。

要求：

- 使用所学的面向对象编程实现以上2个需求。

![ZrFmbQ8JAo2XV9xKkuAc3ekJnSE](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ZrFmbQ8JAo2XV9xKkuAc3ekJnSE.png)

### 思路分析

- 要存储电影信息，需要编写电影实体类（JavaBean），其中包含以下属性：id、name、price、actor(导演)
- 编写电影操作类，提供以下功能：展示全部电影、根据id查询某个电影，为了实现这2个功能，操作类中还需要保存电影列表（使用数组）

### 代码实现

代码实现参考：

（1）电影信息类

```Java
package com.itheima.demo2;

public class Movie {
    private int id; // 编号
    private String name;
    private double price;
    private String actor;

    public Movie() {
    }

    // 定义一个有参数构造器
    public Movie(int id, String name, double price, String actor) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.actor = actor;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public String getActor() {
        return actor;
    }

    public void setActor(String actor) {
        this.actor = actor;
    }
}
```

（2）电影操作类

```Java
package com.itheima.demo2;

import java.util.Scanner;

// 电影操作类
public class MovieOperator {
    private Movie[] movies; // 记住一个电影对象的数组。
    public MovieOperator(Movie[] movies) {
        this.movies = movies;
    }

    // 打印全部电影信息
    public void printAllMovies() {
        System.out.println("================全部电影信息如下===============");
        for (int i = 0; i < movies.length; i++) {
            // i = 0 1 2 3 4 5
            Movie m = movies[i];
            System.out.println( m.getId() + "\t" + m.getName() + "\t" + m.getPrice() + "\t" + m.getActor() + "\t" );
        }
    }

    // 根据id查询电影
    public void searchMovieById() {
        System.out.println("请输入要查询的电影id：");
        Scanner sc = new Scanner(System.in);
        int id = sc.nextInt();
        // 遍历每个电影对象
        for (int i = 0; i < movies.length; i++) {
            // 拿到当前遍历到的电影对象
            Movie m = movies[i];
            // 判断当前电影对象的id是否是我们正在找的电影id，是则打印出该电影信息并立即结束方法
            if (m.getId() == id) {
                System.out.println(m.getId() + "\t" + m.getName() + "\t" + m.getPrice() + "\t" + m.getActor() + "\t" );
                return;
            }
        }
        System.out.println("没有找到该电影");
    }
}
```

（3）测试类

```Java
package com.itheima.demo2;

public class Test {
    public static void main(String[] args) {
        // 目标：完成面向对象的综合小案例。
        // 1、设计电影类Movie，以便创建电影对象，封装电影数据。
        // 2、封装系统中的全部电影数据。（自己造一些数据）
        Movie[] movies = new Movie[6];
        // movies = [null, null, null, null, null, null]
        //            0     1     2     3     4      5
        movies[0] = new Movie(1, "星际穿越", 9.6, "安妮海瑟薇");
        movies[1] = new Movie(2, "速度与激情8", 9.2, "瑞秋·费尔南多");
        movies[2] = new Movie(3, "夏洛特烦恼", 9.2, "沈腾");
        movies[3] = new Movie(4, "战狼2", 9.2, "吴京");
        movies[4] = new Movie(5, "让子弹飞", 9.2, "姜文");
        movies[5] = new Movie(6, "暗战", 9.2, "王大陆、渣渣辉");

        // 3、创建电影操作对象出来，专门负责电影数据的业务操作。
        MovieOperator mo = new MovieOperator(movies);
        mo.printAllMovies(); // ALT + ENTER
        mo.searchMovieById();
    }
}
```

## 相关笔记

- [[Memo/development/programming/java/oop/05-oop-programming.md|05 面向对象编程·笔记]]
- [[Memo/development/programming/java/oop/08-class-object-encapsulation-constructors.md|类、对象、封装与构造器精讲]]

---

> 来源：飞书《讲义》知识库 · day05-面向对象编程 · [原文](https://baxsu7o100e.feishu.cn/wiki/F23uw6HTkiKw1hEimAMVk3MiNyc)
