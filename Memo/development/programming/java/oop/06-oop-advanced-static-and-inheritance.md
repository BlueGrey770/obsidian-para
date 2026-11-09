---
type: memo
date: 2026-11-06
aliases:
  - "06-mian-xiang-dui-xiang-gao-ji-static-ji-cheng"
  - "06 面向对象高级(static , 继承)·笔记"
---

# 06 面向对象高级(static , 继承)·笔记
> 讲义全文：[[Memo/development/programming/java/basics-course/day06-static-and-inheritance.md|Day06 static与继承讲义]]

## 课堂问答：static 与继承

**题目: 为什么类中的实例变量(普通的成员变量)需要创建对象才能使用?**

**目标: 理解并掌握普通成员变量的初始化过程**

```Java
实例变量是对象特有的属性，
创建对象时，会在堆中为实例变量分配存储空间；不创建对象，实例变量就没有实际的内存地址，无法存储数据。
```

**题目: 为什么静态的成员变量不需要创建对象就能使用?**

**目标: 理解并掌握静态成员变量的初始化过程**

```Java
静态成员变量在类加载器 加载时直接在堆中开辟空间初始化数据一次，
所以不需要对象在堆中开辟空间初始化数据也能够使用。
```

**题目: 为什么静态的方法中不能使用普通的成员变量和成员方法?**

目标: 掌握static修饰方法的

掌握静态和非静态之间的访问关系, 理解他们加载的先后顺序问题

```Java
因为普通成员变量需要创建对象后在得到具体的数据或分配堆地址后才能使用，
而静态方法是在类加载器 加载类的时候就已经分配堆地址，所以如果静态方法中存在成员变量和方法的话会因为没有具体数据，无法运行
即：
    静态方法在类加载时就可用，而实例成员需要对象创建后才存在
    静态方法没有隐含的this参数，无法确定要操作哪个对象的实例成员
    无法预先确保普通的成员变量和成员方法是否已经在堆中创建。
```

**题目: 请将继承的语法格式写出来**

**目标: 掌握继承的语法**

```Java
格式：
   public class 子类 extends 父类{
   }
```

**题目: 请描述继承有什么注意事项? 提示:一个人能有两个亲爹么?**

**目标: 掌握继承的特点**

```Java
一个子类只能有一个父类，而一个父类可以有多个子类
父类可以多代叠加，即 一级父类（被子类最先引用的），二级父类是一级父类的 父类
    如果出现上述情况，子类引用父类的方法或属性出现调用冲突，最近定义的变量会被优先考虑
    私有化的对象 无法通过继承调用 但可以通过 间接调用，即 getter 和 setter
```

**题目: 请描述继承有什么好处?**

**目标: 理解继承的好处**

```Java
可以提高代码的重用性，减少一些重复代码书写的重复
```

**题目: 权限修饰符有哪些? 重点的两个各自有什么特点?**

**目标: 认识常见的权限修饰符**

```Java
四种权限修饰符的使用范围：
    private 只能本类调用
    缺省 本类、同一个包中的类
    protected 本类，同一个包中的类、子孙类中
    public 任意位置
private 和 public 是最常用的两个也是最重要的：
    public 是在任何地方都能调用的，方便用于变量和方法的扩展和使用
    private 
        是私有化修饰符，主要用于保证变量和对象的安全，不允许随意修改和访问，
        同时也可以私有化方法，仅在在本类中调取和使用

```

**题目: 请说明什么是方法重写, 以及什么时候需要方法重写?**

**目标: 掌握方法重写使用场景**

```Java
什么是重写：
    重写是在子类中对父类中相同方法调用时对功能或行为的修改，
    如果需要增加功能可以用super.方法名();来简略父类中的代码，

使用场景：
    一般在对方法或功能更新迭代时会用到重写，以满足更多新需求。
```

**题目: 请描述子类和父类出现相同的属性和方法时, 如果创建子类对象, 默认使用谁的? 为啥?**

**目标: 理解子类和父类的成员冲突访问特点**

```Java
当子类和父类出现相同的属性和方法时，创建子类对象后，默认使用子类的属性和方法。
原因在于：
    Java的“就近原则”，即首先在当前类（子类）中查找，如果找到则直接使用。
    如果子类重写了父类的方法，则调用子类重写的方法；

```

**题目:**

**如果子类和父类中出现相同的属性和方法时, 在子类的方法中非要使用父类的数据和功能我们该怎么办?**

**目标: 掌握继承格式下如何显示访问父类的内容**

```Java
如果子类中声明了与父类同名的属性，想要不依靠就近原则 需要子类中明确指定要访问父类的成员。
如果需要在子类中访问父类的成员，可以使用super关键字，如super.属性名或super.方法名()。
在方法重写时，也可以保留父类功能的同时添加新功能
```

**题目:**

**分析以下需求并实现**

**1.项目经理类Manager 属性： 姓名name 工号id 工资salary 奖金bonus 行为： 工作work()**

**2.程序员类Coder 属性： 姓名name 工号id 工资salary 行为： 工作work()**

**要求:**

**1.按照以上要求定义Manager类和Coder类,属性要私有,生成空参、有参构造，setter和getter方法,采用继承简化书写**

**2.定义测试类,在main方法中创建该类的对象并给属性赋值**

**3.调用成员方法,打印格式如下: 工号为123基本工资为15000奖金为6000的项目经理正在努力的做着管理工作,分配任务,检查员工提交上来的代码..... 工号为135基本工资为10000的程序员正在努力的写着代码......**

(注意:本案例和昨天作业不一致, 需要通过继承解决重复代码的问题)

**目标: 掌握继承的实际应用**

```TypeScript
public class Pepole {
    private String id;
    private String name;
    private double salary;

    public Pepole() {
    }

    public Pepole(String id, String name, double salary) {
        this.id = id;
        this.name = name;
        this.salary = salary;
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

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }
}
```

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
    public void work(){
        System.out.println("工号为" + id + "本工资为" + salary + "奖金为" + bonus + "的项目经理" + name + "正在努力的做着管理工作,分配任务,检查员工提交上来的代码");

    }
}
```

```Java
public class Coder extends Pepole {
    public void work(){
        System.out.println("工号为" + this.getId() + "基本工资为" + this.getSalary() + "的程序员" + this.getName() + "正在努力的写着代码");

    }
}
```

```Java
public class Test {
    public static void main(String[] args) {
        Manager ma = new Manager();
        ma.setId("123");
        ma.setName("一龙");
        ma.setSalary(15000);
        ma.setBonus(6000);
        ma.work();

        Coder co = new Coder();
        co.setId("135");
        co.setName("方便");
        co.setSalary(10000);
        co.work();
    }
}
```

**题目:**

**分析以下需求并实现**

**1.猫类Cat 属性: 毛的颜色color 品种breed 行为: 吃饭eat() 抓老鼠catchMouse()**

**2.狗类Dog 属性: 毛的颜色color 品种breed 行为: 吃饭() 看家lookHome()**

**要求:**

**1.按照以上要求定义Cat类和Dog类,属性要私有,生成空参、有参构造，setter和getter方法,采用继承简化书写**

**2.定义测试类,在main方法中创建该类的对象并给属性赋值**

**3.调用成员方法,打印格式如下:**

**花色的波斯猫正在吃鱼.....**

**花色的波斯猫正在逮老鼠....**

**黑色的藏獒正在啃骨头.....**

**黑色的藏獒正在看家.....**

**目标: 掌握继承的实际应用**

```TypeScript
public class Animal {
    private String color;
    private String breed;

    public Animal() {
    }

    public Animal(String color, String breed) {
        this.color = color;
        this.breed = breed;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public String getBreed() {
        return breed;
    }

    public void setBreed(String breed) {
        this.breed = breed;
    }
    public void eat(){
        System.out.print(this.getColor() + "的" + this.getBreed() + "正在");

    }
}
```

```TypeScript
public class Cat extends Animal{
    public Cat() {
    }

    public Cat(String color, String breed) {
        super(color, breed);
    }

    public void catchMouse(){
        super.eat();
        System.out.print("逮老鼠...");
        System.out.println();
    }
    public void eat(){
        super.eat();
        System.out.print("吃鱼..");
        System.out.println();
    }
}
```

```TypeScript
public class Dog extends Animal{
    public void lookHome(){
        super.eat();
        System.out.print("看家...");
        System.out.println();

    }
    public void eat(){
        super.eat();
        System.out.print("啃骨头...");
        System.out.println();
    }
}
```

```Java
public class Test {
    public static void main(String[] args) {
        Cat cat = new Cat();
        cat.setColor("花色");
        cat.setBreed("波斯猫");
        cat.eat();
        cat.catchMouse();

        Dog dog = new Dog();
        dog.setColor("黑色");
        dog.setBreed("藏獒");
        dog.eat();
        dog.lookHome();
    }
}
```

**题目: 下面程序编译是否会报错并说明原因？**

![ai4pbcmhzomzjbxsnmzc5x3oncf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ai4pbcmhzomzjbxsnmzc5x3oncf.png)
> 图注：图片展示了一段Java代码，定义了两个类。`Fu`类有一个带参数的构造方法，用于输出“Fu有参数的构造方法”；`Zi`类继承自`Fu`类，其构造方法无参数，用于输出“Zi无参数的构造方法”。该代码用于说明继承之后子类和父类构造函数的特点，是课堂问答中关于程序编译是否会报错并说明原因的示例代码。

**目标: 掌握继承之后子类和父类构造函数的特点**

```Java
父类没有定义无参构造，定义了有参构造，无参构造系统不会生成，同时子类也没有定义父类的有参构造方法，继承失败
所以会报错
需要在子类的无参构造中添加super()来声明父类有参构造方法

（1）子类构造函数第一行如果没有显示走任何构造，则默认走父类的无参构造super（）；
（2）父类如果没有空参构造，则子类必须手动访问父类的有参构造
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 面向对象 / 06 面向对象高级(static , 继承)·笔记 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/H6y1wHsZJiPd08kAdHQcGUqdnsb)
