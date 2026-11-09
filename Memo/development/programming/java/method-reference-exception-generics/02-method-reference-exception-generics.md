---
type: memo
date: 2026-11-06
aliases:
  - "02-fang-fa-yin-yong-yi-chang-fan-xing"
  - "02 方法引用,异常,泛型"
---
# 02 方法引用,异常,泛型

## 一. 方法引用

*方法引用是jdk8出现的用来****简化lambda的语法,*** *本质也是简化创建子类对象*

前提：

*当我们对"****函数式接口****"的抽象方法进行重写时, 发现方法体在别的地方已经存在了, 那么我们就不用再写了,* ***直接引用别人的方法体****作为自己的方法即可*

### 1.1 引用成员方法

<callout emoji="🥭">
**引入格式:  对象::成员方法名;**
</callout>

```Java
public interface B {

    void fun();
}

public class A {

    public void method(){
        for (int i = 0; i < 10; i++) {
            System.out.println("我爱java");
        }
    }
}

public class Test {

    public static void main(String[] args) {
        //4.采用方法引用简化
        B b = new A()::method;
        b.fun();
    }
}
```

### 1.2 引入静态方法

<callout emoji="🥭">
格式:  类名::方法名
</callout>

案例：

```Java
public interface B {

    void fun();
}

public class A {

    public static void method(){
        for (int i = 0; i < 10; i++) {
            System.out.println("我爱java");
        }
    }
}

public class Test {

    public static void main(String[] args) {
    //相比于成员方法的引用这里不用再创建（new）对象了直接应用方法就可以了；
        B b = A::method;
        b.fun();
    }
}
```

### 1.3 引用特殊方法

<callout emoji="🥭">
前提:  我们要对函数式接口的抽象方法进行重写的时候, 发现想要的方法体刚好和**第一个参数**里面的某一个方法吻合, 抽象方法的**剩余参数和要引用的方法的参数吻合**, 此时才能满足特殊方法引用
格式:  类名::方法名
</callout>

案例：

```Java
public interface B {

    void fun(SecondCarSeller secondCarSeller, String state, String brand, double price);
}


public class SecondCarSeller {

    public void introduceCar(String state, String brand, double price) {
        System.out.println("这是一台" + brand + "的车");
        System.out.println("新车价格是" + price + "元,目前跳楼价" + price * 0.4 + "折处理");
        System.out.println("该车状况是:"+state);
        System.out.println("保证安全无重大事故, 无泡水, 无火烧,假一赔十~~~~");
    }
}

public class Test {

    public static void main(String[] args) {
            B b = SecondCarSeller::introduceCar;
        b.fun(new SecondCarSeller(), "二年准新车,5万公里无调表, 车主一手车,女司机比较爱车","吉利帝豪gs",120000);
    }
}
```

### 1.4 引用构造函数

前提:

<callout emoji="⚠️">
要求函数式接口的抽象方法的返回值必须是一个拥有构造函数的类型才行
</callout>

格式:

<callout emoji="⚠️">
格式: 类名::new
</callout>

案例：

```TypeScript
public class Student {

    private String name;
    private int age;

    public Student() {
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
   
//setter&getter
//toString    

}

public interface B {
    Student fun(String name, int age);
}

public class Test {

    public static void main(String[] args) {
        B b = Student::new;
        Student fun = b.fun("小明",18);
        System.out.println(fun);

    }
}
```

## 二. 异常：

<callout emoji="🌅">
程序在编译或者运行期间出现不正常的操作或者现象叫做异常, 但是异常不是语法错误
</callout>

### *2.1 jvm处理异常的态度:*

<callout emoji="✍️">
  
    *(1)将异常的类型，原因，位置展示到控制台中*  
    *(2)停止程序*
</callout>

### *2.2 处理异常(作为使用别人功能角色)*

#### *抛异常:(延迟处理异常)*

```Java
遇到异常我们可以将异常通过throws关键字将其抛出去，让调用者处理
格式：
    throws 异常类型，多个异常之间采用逗号隔开即可
```

#### *抓异常：*

```Java
格式：
    try{
        可能出现问题的代码
        以及出现问题后不需要执行的代码
    }catch(异常类型1变量名){
        处理方式1
    }catch(异常类型2变量名){
        处理方式2
    }...
    执行流程：
        首先执行try中的代码,一旦发现异常就会找catch匹配，而且从上往下匹配，一旦匹配成功，
        就会执行对应的处方式，整个try catch语句结束，但是如果try中没有异常则不会执行catch中的逻辑
    注意事项:
        (1)一旦try中出现异常，则剩余代码不会执行，直接找catch
        (2)catch中的异常类型如果有继承关系，大类型要放到下面才行
```

###  *2. 3 造异常描述问题给别人(作为定义功能给别人使用的角色)*

```Java
通过throw关键字 new 异常对象() 即可
格式：
    throw new 常类型(设置异常原因);
```

### ***2.4 自定义异常：***

<callout emoji="🥖">
*(1)定义一个类起一个见名知义的异常类名继承Exception或者RuntimeException,*  
*(2)然后重写带原因的构造即可*
</callout>

## 三. 泛型

**jdk1.5**出现的新特性, 代表是一种**未知类型**, 可以让使用者自己设置该类型

规则:

<callout emoji="🥛">
(1) 遇到泛型类, 泛型接口, 泛型方法, 记得要对这个**未知类型**进行确定, 否则jvm会默认确定成**Object**
(2) 泛型只能被确定成**引用数据类型**, 不能确定成基本数据类型
</callout>

### 3.1 泛型类

格式:

```Java
public class 类名<泛型1, 泛型2, 泛型3....>{
    //属性
    //行为
    //构造
    //代码块
    //内部类
}

注意: 泛型建议使用单个大写字母规定
泛型类是在创建对象的时候确定, 创建一次确定一次, 多个对象之间泛型相互不干扰
```

**总结: 泛型类确定泛型的时机是创建对象的时候确定, 创建一次确定一次**

**注意事项:**

<callout emoji="⚠️">
只有当类中真的有不确定的数据类型的时候在使用泛型描述, 否则不要**滥用泛型**
</callout>

### 3.2 泛型接口

```Java
格式：
    public interface 接口名<泛型1，泛型2...>{
    
    }
确定泛型的时机：
    （1）在定义接口实现类的时候直接确定，相当于将泛型接口转成普通类
    （2）在定义接口的实习类的时候不确定，那么我们需要将泛型继续在实现类中声明出去才行，将泛型接口转成泛型类
```

### 3.3 泛型方法

```Java
格式：
    修饰符<泛型1，泛型2....>返回值类型 方法名(参数列表){
        方法体；
        return返回值；
```

### 3.4 泛型限定

<callout emoji="💡">
泛型方法确定泛型的时机：  
    调用一次方法确定一次，跟Object类型的功能及其类似  
      
泛型的限定：  
    （1）定义泛型的人限制自己的泛型的数据类型范围  
        只能上限不能下限  
            泛型extends类型--->要求该泛型只能是该"类型"以及该类型的子类，上限  
            //泛型Super类型-->要求该泛型只能是该"类型"以及该类型的父类，下限容易模糊，任何类都能看到object  
    （2）通过通配符限制别人的类型中的内部的泛型  
            ？：代表不限制  
            ？extends类型--->上限：只能是该类型以及该类型的子类类型才行  
            ？super类型--->下限：只能是该类型以及该类型的父类类型才行
</callout>

<callout emoji="🎁">
**泛型方法的泛型和Object有啥区别???**  
    泛型方法可以结合**泛型限定**来限制类型的范围, 但是Object限制不了!!!!
</callout>

## 四. 包装类:

基本数据类型（八种）对应的引用数据类型我们称为**包装类**

```Java
int-->Integer
char-->Character
short-->Short
float-->Float
long-->Long
byte-->Byte
double-->Double
boolean-->Boolean
```

```Java
Integer i = 10; // 在jdk1.5之前这个是错误的, 因为类型不匹配, 但是1.5以及之后就是正确的, 因为出现了自动装箱

int i = new Integer(10);//在jdk1.5之前这个是错误的, 因为类型不匹配, 但是1.5以及之后就是正确的, 因为出现了自动拆箱
```

### 4.1 自动装箱

<callout emoji="🎨">
jdk1.5出现的新特性, 基本数据类型和对应引用类型可以相互转换, 而且是自动的
拆箱:  将引用类型转成对应的基本的数据类型
装箱:  将基本数据类型包装成对应的引用数据类型

将包装类看成基本数据类型来用即可
</callout>

### **4.2 包装类还有什么用?**

1.  包装类可以接受null值, 但是基本数据类型不可以

可以将字符串中的各种字符转换成对应的基本数据类型

```TypeScript
int  Integer.parseInt(String number);   将字符串的整数转成int的整数类型
byte Byte.parseByte(String number); 将字符串的整数转成byte的整数类型
short Short.parseShort(String number); 将字符串的整数转成short的整数类型
long Long.parseLong(String number); 将字符串的整数转成long的整数类型
float Float.parseFloat(String number);将字符串的小数转成float的小数类型
double Double.parseDouble(String number);将字符串的小数转成double的小数类型
boolean Boolean.parseBoolean(String b); 将"true"转成true, 其他都转成false
```

1. 可以将字符串类型的东西转成对应的基本数据类型

### 踩坑：数据类型不一致与拆装箱

### 数据类型不一致  拆装箱

包装类型有自动拆装箱的能力

![coxebnuhwolrsaxww6bcs6lxnud](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/coxebnuhwolrsaxww6bcs6lxnud.png)
> 图注：图片展示了一个名为sum的函数代码，该函数接收一个泛型参数list，类型为ArrayList<? extends Number>。代码中，变量sum被初始化为0，接着通过for循环遍历list中的元素，将每个元素转换为Number类型后调用doubleValue()方法，将结果累加到sum中。其中，sum += num.doubleValue()部分被红色框突出显示，表明这是手动拆箱操作，用于解决Number类型不能自动拆箱的问题，与上下文提到的类型报错 - 集合相关。

因为Number不是包装类 不能自动拆装箱！！！！！

```Java
private static double sum(ArrayList<? extends Number> list) {
    double sum = 0;
    for (int i = 0; i < list.size(); i++) {
        Number num = list.get(i);
        //包装类型有自动拆装箱的能力，Number不是包装类，所以需要手动拆装箱
        sum += num.doubleValue();//手动拆装箱
    }
    return sum;
}
```

### 类型报错 - 集合

```SQL
ArrayList<Byte> list1 = new ArrayList<>();
//由于数值类型默认int类型，所以添加到非int类型的集合中需要转换
list1.add((byte)1);
list1.add((byte)2);
list1.add((byte)3);
System.out.println(sum(list1));
ArrayList<Short> list2 = new ArrayList<>();
list2.add((short)1);
list2.add((short)2);
list2.add((short)3);
System.out.println(sum(list2));
ArrayList<Integer> list3 = new ArrayList<>();
//数值类型默认int类型，所以添加到int类型的集合中不需要转换
list3.add(1);
list3.add(2);
list3.add(3);
System.out.println(sum(list3));
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 方法引用、异常与泛型 / 02 方法引用,异常,泛型 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/LBIkwkQSIiGJAfkxXVCcK4gXnJt)
