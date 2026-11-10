---
type: memo
date: 2026-11-06
aliases:
  - "05-mian-xiang-dui-xiang-bian-cheng"
  - "05 面向对象编程·笔记"
---

# 05 面向对象编程·笔记
> 系统详解：[[Memo/development/programming/java/java-basics/oop-programming.md|面向对象编程]]

## 问答：类、对象与封装

**题目: 请简述什么是类?什么是对象?以及二者的关系?**

**目标:分清楚类和对象的关系**

```Java
类是一个大范围内多个属性和类型相同的集合。
对象是类中特定且具体的一个。
关系：
    类是由多个相同类型的对象构成的，类包含对象。
类定义了“是什么”，对象代表了“具体的一个”
```

**题目: 请写出定义一个类的基本语法格式,并且写出类中三大组成部分**

**目标: 掌握定义类的语法格式以及类中包含的内容**

```Java
类
定义格式：
public class 类名 {
    属性的类型 属性名1;//成员变量
    属性的类型 属性名2;
    ...
    //行为
    //构造函数
    }




 类的构造函数：
    有参格式1：
        public 类名(属性的类型 属性名1,属性的类型 属性名2;){
            this.属性名1 = 属性名1;
            this.属性名2 = 属性名2;
            ...
        }
    无参格式2：
        public 类名(){}

```

**题目: 简述构造方法的格式,作用及注意事项;**

**目标: 掌握构造器的格式以及注意事**项

```Java
成员方法：
    格式：
        public void 方法名() { } //void 可以更改为返回值类型

方法名与类名相同：
    public 方法名(参数列表) { this.成员变量 = 方法参数; } //方法名和类名相同 表明这是构造方法

修饰符 类名(参数列表){
    方法体;
}
(1)类名和构造函数大小写都要一致
(2)没有返回值类型，连void都不能有，因此不能出现return返回值动作
(3)如果我们不给任何构造系统会赠送无参构造
(4)我们一旦给了任何构造，系统将不再赠送
(5)构造函数可以重载
```

**题目: 下列信息中包含的内容不是类与对象关系的是（ ）**

**目标: 分清楚对象和类的关系**

A、学生和小明同学

B、电脑和这台联想Y430电脑

C、猫和狗

D、手机和这部华为P20手机

```Java
B
```

**题目: 下列描述正确的是**

**目标: 分清楚对象和类的关系**

A、类与对象之间是包含关系，类包含对象。

B、类与对象之间是等价关系，类和对象其实是同一个概念。

C、对象是类的实例，类是对象的模板。

D、通过一个类，只能创建一个对象。

```Java
A、C
```

**题目: 下列描述错误的是**

**目标: 分清楚局部变量和成员变量的特点**

A、在方法中的变量是局部变量

B、局部变量随着方法的调用而存在，随着方法的调用结束而消失

C、成员变量在类中、方法外

D、局部变量有默认的初始化值

```Java
D
```

**题目: 阅读下面的程序,选择正确的结果**

**目标: 熟知类的组成部分以及每一个部分的特点**

```Java
public class Student {
        private String name;
        private int age;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        name = name; //未声明成员变量 返回默认值 
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}

public class Demo{
        public static void main(String [] args){
        Student s = new Student();
        s.setName("小丽");
        s.setAge(16);
        System.out.println(s.getName() + "---" + s.getAge());
    }
}
```

A、语法错误，编译报错。

B、小丽 --- 16

C、null --- 16

D、小丽 --- 0

```Java
C
```

**题目: 下列说法正确的是**

**目标: 局部变量和成员变量的特点和this关键字的作用**

A、当成员变量和局部变量重名时，可以使用this来区分。

B、this可以调用本类的成员变量和成员方法

C、this表示本类对象的引用。

D、在方法中访问本类成员变量(非重名)，可以省略this不写。

```Java
A、B、C、D
```

**题目: 下列描述正确的是**

**目标: 局部变量和成员变量的特点和this关键字的作用**

A、类的{}中定义的是类的成员，可以有成员变量和成员方法。

B、定义成员变量时，可以不给变量赋值。

C、必须先定义成员变量，再定义成员方法。

D、成员变量的名字可以和局部变量的名字相同。

```Java
A、B、D
```

**题目: 下列描述正确的是**

**目标: 掌握面向对象的核心思想**

A、面向对象思想是基于面向过程思想而来的,强调的是对象

B、面向对象思想关注的是功能的每一个步骤

C、面向对象思想完成一个功能,关注的是功能的结果,而不是每一个步骤,强调的是对象

D、面向对象思想,关注的是对象,让对象帮我们完成一个功能,强调的是对象

```Java
A、C、D
```

**题目: 下列横线处填写Person类的构造方法的正确语法格式是**

**目标: 掌握构造函数的定义格式**

```TypeScript
public class Person{ 
       private String name;
       __________________ 
} 
```

A、public void Person(String name){ name = name; }

B、public Person(){ }

C、public Person(String name){ this.name = name; }

D、public Person(String name){ return name; }

```Java
C
```

**题目：**

**1、实现从控制台接收一个学员信息，并存储到一个对象中**

**2、打印这个对象的所有属性值。**

**要求:**

**设计，并定义一个学员类：Student，要求有以下属性：**

**学员编号（String）**

**姓名（String）**

**性别（String）**

**身高（double）**

**年龄（int）**

**使用封装的原则，并提供无  参和全参的构造方法。**

**1 定义MainApp类，并包含main()方法。**

**2 程序启动后，应分别提示用户输入学员编号、姓名等信息。 数据接收到程序中，并定义局部变量存储；创建Student对象，通过构造方法将所有数据存储到Student对象中；在main方法中获取并打印对象中的每个属性值。案例执行效果:**

![biqubwdgfosuphxgs0yc7kumnwb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/biqubwdgfosuphxgs0yc7kumnwb.png)
> 图注：图片展示的是一个面向对象编程案例执行效果示例。画面背景为浅绿色，上方有提示“请输入学生编号：”，下方依次显示编号“001”、姓名“张三”、性别“男”、身高“185”、年龄“18”。最后输出“学号为:001的学生姓名是:张三，性别是:男，身高是:185.0，年龄是:18”。该图片与文档中“分析以下需求，并用代码实现”部分内容对应，是按照需求用代码实现后的执行效果展示。

**目标: 掌握面向对象基础的语法**

```TypeScript
public class Student {
    String id;
    String name;
    private String gander;
    private double height;
    private int age;

    public Student() {
    }

    public Student(String id, String name, String gander, double height, int age) {
        this.id = id;
        this.name = name;
        this.gander = gander;
        this.height = height;
        this.age = age;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGander() {
        return gander;
    }

    public void setGander(String gander) {
        if (gander.equals("男") || gander.equals("女")){
            this.gander = gander;
        }else
            System.out.println("请输入正确的性别！");
    }

    public double getHeight() {
        return height;
    }

    public void setHeight(double height) {
        if (height < 0 && height > 3){
            System.out.println("请输入正常的身高");
        }else
            this.height = height;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age < 0 && age >168){
            System.out.println("请输入正常的年龄范围！");
        }else
            this.age = age;
    }
}
```

```Java
public static void main(String[] args) {
   studentInfoTypeIn();
}
private static void studentInfoTypeIn() {
    Scanner sc = new Scanner(System.in);
    Student stu = new Student();
    System.out.println("请输入学生编号：");
    String  id= sc.next();
    stu.setId(id);

    System.out.println("请输入姓名：");
    String name = sc.next();
    stu.setName(name);

    System.out.println("请输入性别：");
    String gander = sc.next();
    stu.setGander(gander);

    System.out.println("请输入身高：");
    double height = sc.nextDouble();
    stu.setHeight(height);

    System.out.println("请输入年龄：");
    int age = sc.nextInt();
    stu.setAge(age);

    System.out.println(stu.getName() + "同学的学生编号为：" + stu.getId() +" ");
    System.out.print("该同学的姓名为：" + stu.getName() + "");
    System.out.print(stu.getName() + "同学的年龄为：" + stu.getAge() + "");
    System.out.print(stu.getName() + "同学的性别为：" + stu.getGander() + "");
    System.out.print(stu.getName() + "同学的身高为：" + stu.getHeight());
}
```

**题目:**

**分析以下需求，并用代码实现**

**1.项目经理类Manager 属性：姓名name 工号id 工资salary 奖金bonus 行为： 工作work()**

**2.程序员类Coder 属性： 姓名name 工号id 工资salary 行为： 工作work()**

**要求:**

**1.按照以上要求定义Manager类和Coder类,属性要私有,生成空参、有参构造，setter和getter方法**

**2.定义测试类,在main方法中创建该类的对象并给属性赋值(演示两种方法:setter方法和构造方法)**

**3.调用成员方法work之后,在work方法内打印数据如下:**

**工号为123基本工资为15000奖金为6000的项目经理一龙正在努力的做着管理工作,分配任务,检查员工提交上来的代码.....**

**工号为135基本工资为10000的程序员方便正在努力的写着代码......**

**目标: 掌握面向对象的基础语法**

员工类

```TypeScript
public class Manager {
    private String name;
    private String id;
    private double salary;
    private double bonus;

    public Manager() {
    }

    public Manager(String name, String id, double salary, double bonus) {
        this.name = name;
        this.id = id;
        this.salary = salary;
        this.bonus = bonus;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }

    public double getBonus() {
        return bonus;
    }

    public void setBonus(double bonus) {
        this.bonus = bonus;
    }
    /*
        工号为123基本工资为15000奖金为6000的项目经理一龙正在努力的做着管理工作,分配任务,检查员工提交上来的代码.....
     */
    public void work(String id,double salary, double bonus,String name){
        System.out.println("工号为" + id + "本工资为" + salary + "奖金为" + bonus + "的项目经理" + name + "正在努力的做着管理工作,分配任务,检查员工提交上来的代码");

    }
```

程序员类

```TypeScript
public class Coder {
    private String name;
    private String id;
    private double salary;

    public Coder() {
    }

    public Coder(String name, String id, double salary) {
        this.name = name;
        this.id = id;
        this.salary = salary;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }
    /*
        工号为135基本工资为10000的程序员方便正在努力的写着代码
     */

    public void work(String id,double salary,String name){
        System.out.println("工号为" + id + "基本工资为" + salary + "的程序员" + name + "正在努力的写着代码");

    }
```

测试类

```TypeScript
public static void main(String[] args) {
    coderWork();
    managerWork();
}

private static void managerWork() {
    Manager ma = new Manager();
    ma.setId("123");
    ma.setName("一龙");
    ma.setSalary(15000);
    ma.setBonus(6000);
    ma.work(ma.getId(),ma.getSalary(),ma.getBonus(),ma.getName());
}

private static void coderWork() {
    Coder co = new Coder();
    co.setId("135");
    co.setName("方便");
    co.setSalary(10000);
    co.work(co.getId(),co.getSalary(),co.getName());
}
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 面向对象 / 05 面向对象编程·笔记 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/MdL3wCW4eikjVgkPxV1cy22Pnbf)
