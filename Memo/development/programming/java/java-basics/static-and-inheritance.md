---
type: memo
date: 2026-11-09
---

# static 与继承

static也称为静态，可以修饰成员变量、成员方法。下面分别讲解static的各种使用方式。

## static修饰成员变量

在java中，类的属性也称为成员变量，在属性前面可以加修饰符static。根据成员变量前面是否有static，可以将成员变量分为两类：

1. 静态变量（类变量）

有static修饰，属于类，在计算机里只有一份，会被类的全部对象共享。

静态变量属于类，与类一起加载一次，在内存中只有一份，可以被类和类的所有对象共享

静态变量可以通过类访问（类名.成员名），也可以通过对象访问（对象.成员名）

1. 实例变量（对象的变量）

无static修饰，属于每个对象的。

属于对象，每个对象中都有一份，只能用对象访问（对象.实例变量）

![HKuebd5jnoVTdfxehbxcgaTDn5c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/HKuebd5jnoVTdfxehbxcgaTDn5c.png)

如上图所示：Student类中有一个静态变量name，一个实例变量age。现在创建两个个对象：s1、s2，则s1、s2各自有一个自己的age实例变量，但s1、s2共享一个静态变量name。

以下代码演示了static的用法：

```Java
package com.itheima.staticfield;

public class Student {
    // 静态变量：有static修饰，属于类，只加载一份，可以被类和类的全部对象共享访问
    static String name;
    // 实例变量：没有static修饰，属于对象，每个对象都有一份
    int age;
}

package com.itheima.staticfield;
public class Test {
    public static void main(String[] args) {
        // 目标：认识static修饰成员变量，特点、访问机制，搞清楚作用。
        // 1、类名.静态变量（推荐）
        Student.name = "袁华";
        System.out.println(Student.name);

        // 2、对象名.静态变量（不推荐）
        Student s1 = new Student();
        s1.name = "马冬梅";

        Student s2 = new Student();
        s2.name = "秋雅";

        System.out.println(s1.name); // 秋雅（通过对象访问静态变量）
        System.out.println(Student.name); // 秋雅（通过类访问静态变量）

        // 3、对象名.实例变量
        s1.age = 23;
        s2.age = 18;
        System.out.println(s1.age); // 23
        // System.out.println(Student.age); // 报错！（实例变量必须通过对象访问）
    }
}
```

静态变量的应用场景：

- 如果某个数据只需要一份，且希望能够被共享(访问、修改），则该数据可以定义成静态变量来记住。

## static修饰方法

static关键字除了可以放在成员变量前面，还可以放在方法前面。根据方法前面是否有static，可以将方法分为两类：

1. 静态方法

有static修饰的成员方法，属于类。

 静态方法可以通过类名访问（类名.静态方法），也可以通过对象名访问（对象名.静态方法）

```Java
public static void printHelloWorld(){
   System.out.println("Hello World!");
   System.out.println("Hello World!");
 }
```

1. 实例方法无static修饰的成员方法，属于对象。实例方法，必须通过对象访问（对象.实例方法）

```Java
public void printPass(){
   ...
}
```

那么什么方法应该被设置成静态方法呢？

- 如果一个方法只是为了做一个功能且不需要直接访问对象的数据，这个方法直接定义成静态方法。
- *如果这个方法是对象的行为，需要访问对象的数据，这个方法必须定义成实例方法。*

静态方法示例如下：

```Java
package com.itheima.staticmethod;

public class Student {
    private double score;
    // 静态方法：有static修饰，属于类持有。
    public static void printHelloWorld(){
        System.out.println("Hello World");
        System.out.println("Hello World");
        System.out.println("Hello World");
    }

    // 实例方法：没有static修饰，属于对象持有。
    public void printPass(){
        System.out.println(score >= 60 ? "通过了" : "您挂科了！");
    }

    public void setScore(double score) {
        this.score = score;
    }
}

package com.itheima.staticmethod;

public class Test {
    public static void main(String[] args) {
        // 目标：认识static修饰和不修饰方法的区别
        // 1、类名.静态方法（推荐）
        Student.printHelloWorld();

        // 2、对象.静态方法（不推荐）
        Student s1 = new Student();
        s1.printHelloWorld();

        // 3、对象名.实例方法
        // Student.printPass(); // 报错
        s1.setScore(59.5);
        s1.printPass();
    }
}
```

## 静态方法的应用

在java开发中，好多场景中都可以使用静态方法，其中使用最广泛的一个场景是：工具类。工具类中的方法都是一些静态方法，每个方法用来完成一个功能，以便供给开发人员直接使用。一个典型的工具类如下所示：

```Java
public class XxxxUtil {
    public static void xxx(){
        ...
    }
    public static boolean xxxx(String email){
        ...
    }
    public static String xxxxx(int n){
        ...
    }
    ...
 }
```

那么使用静态方法来设计工具类有什么优势呢？

- 主要提高了代码复用、调用方便（不必生成对象），提高了开发效率。
- 实例方法需要创建对象来调用，此时对象只是为了调用方法，对象占内存，这样会浪费内存。
- 静态方法，直接用类名调用即可，调用方便，也能节省内存。

<callout emoji="💡">
多学一招
工具类没有创建对象的需求，建议将工具类的构造器进行私有。
</callout>

工具类示例（验证码工具类）：

```Java
package com.itheima.staticmethod;
// 静态方法设计工具类。
public class VerifyCodeUtil {

    // 工具类没有创建对象的必要性，所以建议私有化构造器
    private VerifyCodeUtil() {}

    public static String getCode(int n) {
        // 2、定义一个字符串变量用于记录生产的验证码。
        String code = "";
        // 3、循环n次，每次生成一个验证码。
        for (int i = 0; i < n; i++) {
            // i = 0 1 2 3
            // 4、为当前位置随机生成一个数字或者大写字母或者小写字母。 数字0 / 大写1 / 小写2
            // 随机一个 0 或者 1 或者 2 表示当前位置随机的字符类型。
            int type = (int) (Math.random() * 3);  // [0,1) *3 => 0 1 2
            // 5、使用switch判断当前位置随机的字符类型
            switch (type) {
                case 0:
                    // 6、如果当前位置是数字，则随机生成一个数字0-9，然后拼接。
                    int num = (int) (Math.random() * 10);
                    code += num;
                    break;
                case 1:
                    // 7、如果当前位置是大写字母，则随机生成一个字母A-Z，然后拼接。     'A' 65    'Z' 65+25
                    int num1 = (int) (Math.random() * 26); // [0,25]
                    char ch = (char) (65 + num1); // 得到大写字母的随机编号，转成大写字母
                    code += ch;
                    break;
                case 2:
                    // 8、如果当前位置是小写字母，则随机生成一个字母a-z，然后拼接。    'a' 97    'z' 97+25
                    int num2 = (int) (Math.random() * 26); // [0,25]
                    char ch1 = (char) (97 + num2);
                    code += ch1;
                    break;
            }
        }
        return code;
    }
}
```

## static的注意事项

在使用static的过程中，有一些规则需要注意：

- 静态方法中可以直接访问静态成员，不可以直接访问实例成员。
- 实例方法中既可以直接访问静态成员，也可以直接访问实例成员。
- 实例方法中可以出现this关键字，静态方法中不可以出现this关键字的。

以下代码演示了上述规则：

```Java
public class StaticDemo {

    // 静态成员变量
    public static int staticVar = 10;

    // 实例成员变量
    public int instanceVar = 20;

    // 静态方法
    public static void staticMethod() {
        // 可以直接访问静态成员变量
        System.out.println("staticMethod: staticVar = " + staticVar);

        // 不可以直接访问实例成员变量（编译错误）
        // System.out.println("staticMethod: instanceVar = " + instanceVar); // 编译错误

        // 不可以使用 this 关键字（编译错误）
        // System.out.println("staticMethod: this.staticVar = " + this.staticVar); // 编译错误
    }

    // 实例方法
    public void instanceMethod() {
        // 可以直接访问静态成员变量
        System.out.println("instanceMethod: staticVar = " + staticVar);

        // 可以直接访问实例成员变量
        System.out.println("instanceMethod: instanceVar = " + instanceVar);

        // 可以使用 this 关键字
        System.out.println("instanceMethod: this.instanceVar = " + this.instanceVar);
        System.out.println("instanceMethod: this.staticVar = " + this.staticVar);
    }

    public static void main(String[] args) {
        // 调用静态方法
        StaticDemo.staticMethod();

        // 创建对象
        StaticDemo demo = new StaticDemo();

        // 调用实例方法
        demo.instanceMethod();
    }
}
```

## Math工具类介绍

介绍:  Math就是JDK专门为数学相关的计算写的一款工具类, 里面的提供很多静态的功能以及属性供我们使用

Math提供的静态属性:

```Java
public static final double PI = 3.14159265358979323846;
```

Math提供的静态的方法:

```Java
double Math.abs(double number); //求一个数的绝对值, 整数的绝对值是其本身, 负数绝对值是其对应的相反数
double Math.sqrt(double number); //求一个数的平方根,开方
double Math.pow(double number, double exponent); //求number的exponent次幂
double Math.ceil(double number);//向上取整
double Math.floor(double number);//向下取整
double Math.round(double number); //四舍五入
double Math.max(double a, double b);
double Math.min(double a, double b);

```

方法演示:

```Java
public static void demo() {
    //使用Math的静态属性
    System.out.println(Math.PI);
    //求4的开方
    double sqrt = Math.sqrt(4);
    System.out.println(sqrt);
    //求3的3次方
    double pow = Math.pow(3, 3);
    System.out.println(pow);
    // 3.5向上取整,  (1)可以采用.var快速生成返回值类型的变量装方法调用的结果!!!! (2) alt + enter 这个快速修复的快捷键
    // double ceil = Math.ceil(3.5);
    double ceil = Math.ceil(3.5);
    System.out.println(ceil);
    //3.6向下取整
    double floor = Math.floor(3.6);
    System.out.println(floor);
    //3.499 四舍五入
    long round = Math.round(3.5);
    System.out.println(round);
}

```

案例:

请键盘输入圆的半径求出圆的周长和面积???

```Java
public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入圆的半径");
    double r = sc.nextDouble();
    //求出其周长 2πr
    System.out.println("周长是:" + 2 * Math.PI * r);
    //求出面积 πr²
    System.out.println("面积是:" + Math.PI * Math.pow(r, 2));
}
```

请键盘输入矩形的长和宽求出矩形周长和面积???

```Java
public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    System.out.println("请输矩形的长");
    double length = sc.nextDouble();
    System.out.println("请输矩形的宽");
    double length = sc.nextDouble();
    //求出其周长 2πr
    System.out.println("周长是:" + 2 * Math.PI * r);
    //求出面积 πr²
    System.out.println("面积是:" + Math.PI * Math.pow(r, 2));
 }
```

思考: 请键盘输入三角形三边,请求三角形的周长和面积???

<callout emoji="🌷">
提示:  要求任意两条边的和必须大于第三边, 三角形面积可以通过海伦公式完成, 公式如下:
</callout>

![DDdBbjoFpoKZQLxa3rrc6swZnMe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/DDdBbjoFpoKZQLxa3rrc6swZnMe.png)

学生自己实现:

```Java

```

## 继承

### 认识继承

继承是面向对象编程中最重要的特点，那什么是继承呢？我们先来看一个需求：

需求：现正在开发一套黑马员工管理系统，其中需要处理讲师、咨询师的数据

- 讲师的数据有：姓名、具备的技能；
- 咨询的数据有：姓名、解答问题的总人数。

为了存储讲师、咨询师的数据，我们需要开发两个类：讲师类（Teacher）、咨询师类（Consultant），代码如下：

```Java
public class Teacher {
    private String name;
    private String skill;  // 技能

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getSkill() {
        return skill;
    }
    public void setSkill(String skill) {
        this.skill = skill;
    }
 }

 public class Consultant {
    private String name;
    private int number;  // 解答问题的人数

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getNumber() {
        return number;
    }
    public void setNumber(int number) {
        this.number = number;
    }
}

```

我们观察以上代码后，发现两个类中有些代码是重复的：

![M48GbHB6OoYUeGxMmB2cchhSn6b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/M48GbHB6OoYUeGxMmB2cchhSn6b.png)

目前这两个类中只是有一个name属性及其getter、setter方法重复，在实际开发中，可能会有更多的属性是重复的（如：部门、年龄、工龄、工资等），如果按以上方式编码，会导致太多的重复代码。为了解决这个问题，java中提供了继承的机制，**通过继承，可以提高代码的重用性，减少一些重复代码的书写**。

那么继承如何使用呢？接下来我们使用继承来优化一下上述代码，代码如下：

```Java
public class People {
    private String name;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}

public class Teacher extends People{
    private String skill;  // 技能

    public String getSkill() {
        return skill;
    }
    public void setSkill(String skill) {
        this.skill = skill;
    }
}

public class Consultant extends People{
    private int number;  // 解答问题的总人数

    public int getNumber() {
        return number;
    }
    public void setNumber(int number) {
        this.number = number;
    }
}

public class Test{
    public static void main(String[] args){
        Teacher t1 = new Teacher();
        t1.setName("张老师");
        t1.setSkill("java");

        Consultant c1 = new Consultant();
        c1.setName("李老师");
        c1.setNumber(10);
    }
}
```

代码说明：

- 在上述代码中，将讲师类（Teacher）、咨询师（Consultant）重复的数据，专门提出来，放到一个专门的类中（People），这个类称为**父类**
- 第12行：讲师类（Teacher）通过 extends People语法，继承了People类，此时 Teacher 类称为People的子类，子类能继承父类的非私有成员（成员变量、成员方法），即：父类的setName()、getName()方法被继承到了子类
- 第24行：与讲师类相似，也继承了People类
- 第38-44行：创建了讲师对象和咨询师对象，第39、43行依然可以调用setName()方法，因为此方法从父类继承过来了，子类对象中既包含自己的属性，也包含父类的属性，即：子类的对象是由子类、父类共同完成创建的。

通过以上优化后，讲师类、咨询师类中不再有name属性及其setter、getter方法了，但通过讲师类、咨询师类依然能够调用到getName()、setName()方法，完美的减少了代码的重复。

<callout emoji="🌷">
综上，我们可以得到继承的特点：
- Java中提供了一个关键字extends，用这个关键字，可以让一个类和另一个类建立起父子关系
- 被继承的类称为父类（或基类、超类）， 子类也称为派生类
</callout>

![H8OrbgKxBom1dRx7ZeJc1R0pnSf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/H8OrbgKxBom1dRx7ZeJc1R0pnSf.png)

<callout emoji="🌷">
- 子类能继承父类的非私有成员（成员变量、成员方法）
- 子类的对象是由子类、父类共同完成创建的，所以子类对象是完整的。所以通过子类对象能调用父类的方法。
</callout>

### 权限修饰符

在Java中，权限修饰符用来控制类、变量、方法和构造方法的可见性。它们定义了代码的哪些部分可以从其他部分访问。Java提供了四个主要的权限修饰符：public、private、protected 和 默认（没有明确指定任何修饰符时）。

![IBW9b2Hi6oP22ixvgFucMsg3n2d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/IBW9b2Hi6oP22ixvgFucMsg3n2d.png)

四个权限修饰符的具体用法如下：

- public作用：被声明为 public 的类、方法、构造方法和接口能够被任何其他类访问。基本用法：当成员需要对外提供服务或被广泛访问时使用。

  ```Java
  public class MyClass {
      public void myMethod() { // 可以从任何地方调用
          // 方法体
      }
  }
  ```
- Private作用：限制仅在声明它们的类内部访问。这意味着外部类无法直接访问这些元素。基本用法：通常用于封装数据，确保数据的安全性和完整性。

  ```Java
  public class MyClass {
      private int secretData; // 只能在这个类里访问

      private void internalUseOnly() { // 只能在MyClass内调用
          // 方法体
      }
  }
  ```
- protected作用：允许同一个包中的所有类访问，同时也允许不同包中的子类访问。基本用法：适用于希望基类的某些功能只被继承它的子类访问的情况。

  ```Java
  public class MyBaseClass {
      protected void aProtectedMethod() { // 同一个包下的类可以访问，子类也可以访问
          // 方法体
      }
  }

  package anotherPackage;
  public class MySubClass extends MyBaseClass {
      @Override
      protected void aProtectedMethod() {
          super.aProtectedMethod(); // 可以访问父类的protected方法
          // 子类的方法体
      }
  }
  ```
- 默认（不加任何修饰符）作用：也被称为包私有（package-private），只有同一包内的类才能访问。基本用法：适用于不需要公开给整个项目，但可以在同一模块内共享的功能。

  ```Java
  class MyClassInSamePackage {
      void defaultAccessMethod() { // 只有相同包里的类可以访问
          // 方法体
      }
  }
  ```

综上，各个修饰符的权限如下图所示：

![WYb0bGCSFocfMWxKPhrcd95unMf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/WYb0bGCSFocfMWxKPhrcd95unMf.png)

### 继承的特点

前面我们学习了继承的概念及语法，本部分我们学习一下java中继承的特点，java中的继承最重点的有4个特点：单继承、多层继承、根类、就近原则。下面我们逐个分析这些特点：

#### 单继承

在Java中，单继承是指一个类只能直接继承自一个父类。这是Java语言的一个核心特性，它意味着当创建一个新的类时，该类可以选择扩展（即继承）另一个已存在的类，但只能选择一个这样的父类。

```Java
class C extends A {
    // 子类的方法和成员
}
```

上面代码中， extends后面只能有一个父类，如果写多个会报错：

```Java
class C extends A,B {
    // 子类的方法和成员
}
```

Java 语言设计时决定不支持多继承（同时继承多个父类），主要是出于简化语言和避免一些复杂问题的考虑。比如考虑以下情况：

```Java
public class A {
    public void method(){
        System.out.println("复习数学~");
    }
 }

public class  B {
    public void method(){
        System.out.println("复习语文~");
    }
 }

public class C extends A, B{
}

public classTest{
    public static void main(String[] args) {
             C c = new C ();
             c.method();
    }
}
```

以上代码中：

- 假设java支持多继承，第13行代码的C类，就可以同时继承A、B两个类，此时C类会分别从A、B类中继承一个method方法，导致C类中会有两个 method() 方法
- 第19行：调用C类对象的method()方法时，由于C类中有2个method方法，java编译器就无法确定应该使用哪个版本的方法。如果要解决这个问题，则java中就必须再引入新语法（比如：C++就支持多继承，因而 c++中就有更多复杂的语法需要程序员去掌握）。

为了减少多继承带来的复杂问题，所以java不支持多继承。但在java中提供了接口（下次课会讲到）作为多继承的一种替代机制，这样就可以拥有类似于多继承的功能，但没有了多重继承所带来的一些复杂性。

#### 多层继承

Java中虽然不支持多继承，但是支持多层继承。在Java中，多层继承是指一个类可以从另一个类继承，而这个父类本身也可以从另一个类继承。这种层次结构可以一直延续下去，形成一个继承链。通过这种方式，子类不仅可以继承直接父类的属性和方法，还可以间接地继承更高层级祖先类的非私有成员。

![JuAWbDxsgooQcpxwOeRcfRb1nkJ](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/JuAWbDxsgooQcpxwOeRcfRb1nkJ.png)

我们通过以下代码来理解下这个机制：

```Java
class GrandParent {
    // 祖先类的方法和字段
}

class Parent extends GrandParent {
    // 父类的方法和字段
}

class Child extends Parent {
    // 子类的方法和字段
}
```

上面示例中，`Child` 类继承了 `Parent` 类，而 `Parent` 类又继承了 `GrandParent` 类。因此，`Child` 类将拥有 `Parent` 和 `GrandParent` 中所有非私有的方法和字段。

### 子类和父类成员冲突的问题

#### 当子父类的成员变量一致

在Java中，访问变量时遵循一种称为“就近原则”的规则。如果当父类和子类中出现相同的成员变量的时候, 如果我们创建的是子类对象, 那么会优先使用子类中成员变量, 子类没有才会访问父类的成员变量, 如果非要访问父类的成员变量java为我们提供的super关键字, 这个super关键字和this关键字的使用格式及其类型, 专门用于访问父类相关的属性使用.  例如:

```Java
class SuperClass {
    protected int number = 10;
}

class SubClass extends SuperClass {
    int number = 20; // 子类变量覆盖了父类变量

    public void printNumbers() {
        System.out.println(number); // 输出 20
        System.out.println(super.number); // 输出 10 (通过super关键字访问父类同名变量)
    }
}
```

总之，在Java中，当你在同一作用域内遇到同名变量时，最近定义的变量会被优先考虑。

#### 当子父类的成员方法一致

<callout emoji="🌷">
方法重写是Java中的一个重要特性，允许子类重新定义从父类继承的方法。当子类觉得父类中的某个方法不好用，或者无法满足自己的需求时，子类可以重写一个方法名称、参数列表一样的方法，去覆盖父类的这个方法，这就是方法重写。
</callout>

方法重写的注意事项：

- 子类重写父类方法时，访问权限必须大于或者等于父类该方法的权限（ public > protected > 缺省 ）。
- 重写的方法返回值类型，必须与被重写方法的返回值类型一样，或者范围更小。
- 私有方法、静态方法不能被重写，如果重写会报错的。
- 我们可以采用jdk提供的@Override注解帮助我们校验方法是否正确重写

方法重写示例：`Animal` 类有一个 `makeSound()` 方法，而 `Dog` 和 `Cat` 分别继承了 `Animal` 并重写了 `makeSound()` 方法以发出各自的声音：

```Java
// 父类
class Animal {
    public void makeSound() {
        System.out.println("Some generic animal sound");
    }
}

// 子类
class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("汪汪!");
    }
}

// 子类
class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("喵喵!");
    }
}

// 测试代码
public class TestOverride {
    public static void main(String[] args) {
        Dog myDog = new Dog();  // 创建 Dog 对象
        myDog.makeSound();     // 输出: 汪汪!

        Cat myCat = new Cat();  // 创建 Cat 对象
        myCat.makeSound();     // 输出: 喵喵!
    }
}
```

代码说明：

- 第3行：Animal类中创建了一个makeSound()方法
- 第11行：Dog类继承了Animal类，并且提供了一个与父类同名的方法makeSound()，我们称Dog类中的方法重写了父类的方法。
- 第19行：Cat类继承了Animal类，并且提供了一个与父类同名的方法makeSound()，我们称Cat类中的方法重写了父类的方法。
- 第27-28行：创建了Dog的对象myDog，然后调用了makeSound()方法，此时调用的是子类中的方法。
- 第30-31行：创建了Cat的对象myCat，然后调用了makeSound()方法，此时调用的是子类中的方法。

### 子类构造器的特点

在Java中，父类和子类的构造器执行有一些特定的特点和规则。理解这些规则对于正确实例化对象非常重要。

**构造器执行特点：**

1. **父类构造器优先执行**：

   - 当创建子类的对象时，首先会调用父类的构造器。这是因为子类需要先初始化继承自父类的部分。
   - Java会自动调用父类的无参构造器，如果父类没有无参构造器，则需要在子类构造器中使用 `super()` 显式调用父类的构造器，并传递必要的参数。
2. **顺序**：

   - 构造器执行的顺序为：父类构造器 -> 子类构造器。
3. **`super`关键字**：

   - 在子类构造器中，可以使用 `super()` 来调用父类的构造器。如果这样做了，必须是构造器的第一行代码。
   - 可以通过 `super(参数)` 调用父类的带参构造器。

下面通过代码理解一下上面的规则：

```Java
package com.itheima.demo1;

public class People{
    private String name;
    private int age;
    public People() {
    }

    public People(String name, int age) {
        this.name = name;
        this.age = age;
    }

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

代码说明：

- 第6行：提供了无参的构造器
- 第9行：提供了带参数的构造器

```Java
package com.itheima.demo1;

public class Teacher extends People{
    private String skill;
    public Teacher(){
    }

    public Teacher(String name, int age, String skill) {
        super(name, age);
        this.skill = skill;
    }

    public String getSkill() {
        return skill;
    }

    public void setSkill(String skill) {
        this.skill = skill;
    }
}
```

代码说明：

- 第9行：通过super(name,age)调用了父类的带参数的构造器，将数据保存到了父类的属性中(name、age)

```Java
package com.itheima.demo1;

public class Test {
    public static void main(String[] args) {
        // 目标：子类调用父类构造器的常见应用场景。
        Teacher t = new Teacher("李四", 36, "Java");
        System.out.println(t.getName());
        System.out.println(t.getAge());
        System.out.println(t.getSkill());
    }
}
```

代码说明：

- 第6行：创建Teacher对象时，会先调用父类的构造器，然后再调用子类的构造器

<callout emoji="💡">
多学一招
this(…)调用兄弟构造器
</callout>

任意类的构造器中，是可以通过this(…) 去调用该类的其他构造器的，示例如下：

![XH1ZbJ3hmoxGsKxtNeMcnMYcn6b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/XH1ZbJ3hmoxGsKxtNeMcnMYcn6b.png)

this(...)和super(…)使用时的注意事项：

- this(…) 、super(…) 都只能放在构造器的第一行，因此，有了this(…)就不能写super(…)了，反之亦然。

### Object类介绍

在Java中，`java.lang.Object` 类是所有类的根类。这意味着每个类都直接或间接地继承自 `Object` 类。即使你在定义一个类时没有明确指定其父类，编译器也会自动让它继承 `Object`。`Object` 类提供了基本的功能支持，使得所有的Java对象都能够使用这些通用的操作。

比如以下代码：

```Java
package com.itheima.demo1;
public class A {
    public void method(){
        System.out.println("复习数学~");
    }
}

package com.itheima.demo1;
public class Test {
    public static void main(String[] args) {
        A a1 = new  A();
        a1.method();
        a1.toString();
    }
}
```

上面的代码中，A类从语法上看虽然没有继承任何类，但在java编译代码时，会让A类自动继承Object类，所以A类中不仅有 method方法，还有很多其他方法（从Object中继承过来的）。我们在idea中写代码时，就可以看到a1对象中可以使用的所有方法：

![Gjvub6Ky8oFtF7xiXDkcB3XUnSe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Gjvub6Ky8oFtF7xiXDkcB3XUnSe.png)

<callout emoji="🌷">
总之，在java中，Object类是所有类的祖宗类，任何类都直接或间接的继承了Object类，所以任何对象都可以使用Object类中的方法。
</callout>

Object 提供的常用方法介绍

```Java
boolean equals(Object obj) // 用来比较内容用的, 但是需要重写才行!!!
   注意: ==符号, 如果基本数据类型的话比较的是数值, 如果是引用数据类型比较的是地址值
            Object中的equals方法默认也是采用==比较的, 如果想要比较内容我们可以对该方法进行重写
String toString() // 返回对象的字符串表示形式。
   注意:  默认子类不重写的话我们看到的是带@符号一个"地址值"!!!
        如果重写了我们可以看到对象的内容
如何重写???
     右键 ---> generate ---> toString;
     右键 ---> generate ---> equals & hashcode;
```

案例: 希望打印学生对象看到学生的内容, 比较学生比较的是学生内容

```Java
public class Student  extends Object{

    private String name;
    private int age;

    public Student() {
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

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

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        //this =001  o = 002
        if (this == o) return true;
        if (o == null ||this.getClass() != o.getClass()) return false;
        //走到 o的类型和this的类型相同
        Student student = (Student) o;

        if (age != student.age) return false;
        return Objects.equals(name, student.name);
    }

}
```

```Java
public class Test {

    public static void main(String[] args) {
        Student student1 = new Student("林青霞", 18);//001
        Student student2 = new Student("林青霞", 18);//002
        System.out.println(student1);
        System.out.println(student2);
        boolean equals = student1.equals(student2);
        System.out.println(equals);
        //System.out.println(student1 == student2);
    }
}
```

控制台输出结果:

![RXcMb6ycvoo1xdx6e8cctFyQnvc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/RXcMb6ycvoo1xdx6e8cctFyQnvc.png)

## 相关笔记

- [[Memo/development/programming/java/oop/06-oop-advanced-static-and-inheritance.md|06 static与继承·笔记]]
- [[Memo/development/programming/java/oop/09-static-and-inheritance.md|static 关键字与继承机制精讲]]

---

> 来源：飞书《讲义》知识库 · day06-面向对象高级(static , 继承) · [原文](https://baxsu7o100e.feishu.cn/wiki/TARkwAzDEiYoRSM9MRKQzh1Qn7e)
