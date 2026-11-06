---
type: memo
date: 2026-11-06
aliases:
  - "07 面向对象高级(多态)·笔记"
---

# 07 面向对象高级(多态)·笔记

## 课堂问答：多态

题目: 多态的前提有哪些? 格式是什么?

目标: 记住多态的前提,掌握多态的格式

```Java
多态的前提是继承和重写
格式：
    父类名 对象名 = new 子类()
```

**题目: 多态的通常使用场景是什么?**

**目标: 理解多态的使用场景**

```Java
多个类拥有重复的功能，常用于同方法的迭代更新，提高可扩展和易于维护
```

**题目: 多态的弊端是什么? 为啥?**

**目标: 理解多态的弊端以及原因**

```Java
弊端是：
    在使用多态时无法访问子类特有属性和方法，如果调用则会报错
    如果需要调用，要用向下转型，来实现对子特有类属性和方法的调用
    或向上转型
```

**题目: 多态的有几种转型?  分别是什么? 以及有哪些注意事项?**

**目标: 理解多态转型问题**

```Java
两种转型
向上转型和向下转型
注意：
    通过父类变量只能访问父类中的方法，而无法访问到子类中特有的方法。
```

**题目: 多态为啥必须有子类重写父类方法?**

**目标: 理解动态的前提**

```Java
因为多态的前提是继承和重写，来实现对父类属性的调用及对子类重写父类方法的复用，
如果没有子类重写父类方法，则多态的使用将失去意义。
```

**题目 ：动物叫声模拟器（基础继承 + 多态）**

**题目要求：**

1. **定义父类 `Animal`，包含：**

   - **属性 `name`（姓名，`String` 类型）**
   - **构造方法（初始化姓名）**
   - **普通方法 `makeSound()`，输出 “动物发出叫声”**
2. **定义两个子类 `Dog` 和 `Cat`，继承自 `Animal`：**

   - **`Dog` 重写 `makeSound()` 方法，输出 “小狗 [姓名] 汪汪叫”**
   - **`Cat` 重写 `makeSound()` 方法，输出 “小猫 [姓名] 喵喵叫”**
3. **在主类中：**

   - **创建一个 `Animal` 数组，存储 2 只狗和 2 只猫（自行指定姓名）**
   - **遍历数组，调用每个对象的 `makeSound()` 方法，观察多态效果**

```Java
public class Animal {
    private String name;

    public Animal(String name) {
        this.name = name;
    }

    public Animal() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
    public void makeSound(){
        System.out.println("动物发出叫声");
    }
}

public class Cat extends Animal{
    public Cat(String name) {
        super(name);
    }

    public Cat() {
    }

    @Override
    public void makeSound() {
        System.out.println("小猫" + getName() + "喵喵叫");
    }
}

public class Dog extends Animal{
    public Dog(String name) {
        super(name);
    }

    public Dog() {
    }

    @Override
    public void makeSound() {
        System.out.println("小狗" + getName() + "汪汪叫");
    }
}

public class Test {
    public static void main(String[] args) {
    Animal[] cat1;
    cat1 = new Animal[2];
    cat1[0] = new Cat("小花");
    cat1[1] = new Cat("小草");
    for (int i = 0; i < cat1.length; i++) {
        Animal cat = cat1[i];
        cat.makeSound();
    }
    Animal[] dog1 = new Animal[2];
    dog1[0] = new Dog("小白");
    dog1[1] = new Dog("小黑");
    for (int i = 0; i < dog1.length; i++) {
        Animal dog = dog1[i];
        dog.makeSound();
        }
    }
    }
}
```

**题目：商品价格计算器（继承 + 多态 + 数组遍历）**

**题目要求：**

1. **定义父类 `Product`，包含：**

   - **属性 `name`（商品名称，`String` 类型）、`price`（单价，`double` 类型）**
   - **构造方法（初始化名称和单价）**
   - **普通方法 `calculateTotal(int quantity)`，返回总价（单价 × 数量）**
   - **普通方法 `showInfo(int quantity)`，输出商品名称、单价、数量和总价**
2. **定义两个子类 `DiscountProduct` 和 `TaxedProduct`，继承自 `Product`：**

   - **`DiscountProduct`（折扣商品）：新增属性 `discountRate`（折扣率，`double` 类型，如 0.8 表示 8 折），重写 `calculateTotal(int quantity)`，返回折扣后总价（单价 × 数量 × 折扣率）**
   - **`TaxedProduct`（含税商品）：新增属性 `taxRate`（税率，`double` 类型，如 0.1 表示 10% 税），重写 `calculateTotal(int quantity)`，返回含税总价（单价 × 数量 × (1 + 税率)）**
3. **在主类中：**

   - **创建一个 `Product` 数组，存储 1 个普通商品、1 个折扣商品、1 个含税商品（自行指定属性值）**
   - **遍历数组，调用每个对象的 `showInfo(int quantity)` 方法（数量统一设为 5），输出商品信息和总价**

```Java
public class DiscountProduct extends Product{
    private double discountRate;

    public DiscountProduct(String name, double price, double discountRate) {
        super(name, price);
        this.discountRate = discountRate;
    }

    public DiscountProduct(double discountRate) {
        this.discountRate = discountRate;
    }

    public DiscountProduct() {
    }

    public double getDiscountRate() {
        return discountRate;
    }

    public void setDiscountRate(double discountRate) {
        this.discountRate = discountRate;
    }

    @Override
    public double calculateTotal(int quantity) {
        double total = quantity * getPrice() * discountRate;
        return total;
    }
}

public class Product{
    private String name;
    private double price;

    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }

    public Product() {
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
    public double calculateTotal(int quantity){
        double total = price * quantity;
        return total;

    }
    public void showInfo(int quantity){
        double total = calculateTotal(quantity);
        System.out.println("商品名：" + name + "总价为：" + total + "元");
    }
}

public class TaxedProduct extends Product{
    private double taxRate;

    public TaxedProduct(String name, double price, double taxRate) {
        super(name, price);
        this.taxRate = taxRate;
    }

    public TaxedProduct(double taxRate) {
        this.taxRate = taxRate;
    }

    public TaxedProduct(String name, double price) {
        super(name, price);
    }

    public TaxedProduct() {
    }

    @Override
    public double calculateTotal(int quantity) {
        double total = getPrice() * quantity * (1 + taxRate);
        return total;
    }
}

public class Test {
    public static void main(String[] args) {
        Product[] products = new Product[3];
        products[0] = new DiscountProduct("pino", 100, 0.8);
        products[1] = new TaxedProduct("mike", 5, 0.1);
        products[2] = new Product("normal", 50); // 普通商品
        for (int i = 0; i < products.length; i++) {
            Product product1 = products[i];
            product1.showInfo(5);
        }
    }
}

```

**题目 ：交通工具行驶模拟（继承 + 多态 + 循环）**

**题目要求：**

1. **定义父类 `Vehicle`，包含：**

   - **属性 `brand`（品牌，`String` 类型）、`speed`（速度，`int` 类型，单位：km/h）**
   - **构造方法（初始化品牌和速度）**
   - **普通方法 `drive(int hours)`，输出 “[品牌] 以 [速度] km/h 行驶了 [小时] 小时，总里程：[速度 × 小时] km”**
2. **定义两个子类 `Car` 和 `Bicycle`，继承自 `Vehicle`：**

   - **`Car`（汽车）：重写 `drive(int hours)` 方法，输出 “汽车 [品牌] 以 [速度] km/h 行驶了 [小时] 小时，总里程：[速度 × 小时] km，消耗汽油：[速度 × 小时 ×0.1] 升”（假设每公里消耗 0.1 升汽油）**
   - **`Bicycle`（自行车）：重写 `drive(int hours)` 方法，输出 “自行车 [品牌] 以 [速度] km/h 行驶了 [小时] 小时，总里程：[速度 × 小时] km，消耗卡路里：[速度 × 小时 ×50] 卡”（假设每公里消耗 50 卡路里）**
3. **在主类中：**

   - **创建一个 `Vehicle` 数组，存储 1 辆汽车、1 辆自行车（自行指定品牌和速度）**
   - **遍历数组，调用每个对象的 `drive(int hours)` 方法（小时数分别设为 2 和 3），模拟不同时长的行驶**

```Java
public class Vehicle {
    private String brand;
    private int speed;

    public Vehicle(String brand, int speed) {
        this.brand = brand;
        this.speed = speed;
    }

    public Vehicle() {
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public int getSpeed() {
        return speed;
    }

    public void setSpeed(int speed) {
        this.speed = speed;
    }
    public void drive(int hours){
        double total = speed * hours;
        System.out.println(brand + "以" + speed + "Km/h行驶了" + hours + "小时，总里程" + total + "km");
    }
}

public class Car extends Vehicle{
    public Car(String brand, int speed) {
        super(brand, speed);
    }

    public Car() {
    }

    @Override
    public void drive(int hours) {
        double total = getSpeed() * hours;
        double kll =  total * 0.1;
        System.out.println("汽车 "+ getBrand() + " 以 " + getSpeed() + " km/h 行驶了 "+ hours + " 小时，总里程：" + total + " km，消耗汽油：" + kll + " 升");
    }
}

public class Bicycle extends Vehicle{
    public Bicycle(String brand, int speed) {
        super(brand, speed);
    }

    public Bicycle() {
    }

    @Override
    public void drive(int hours) {
        double total = getSpeed() * hours;
        double kll =  total * 50;
        System.out.println("自行车 "+ getBrand() + " 以 " + getSpeed() + " km/h 行驶了 "+ hours + " 小时，总里程：" + total + " km，消耗卡路里：" + kll + " 卡");

    }
}

public class Test {
    public static void main(String[] args) {
        Vehicle[] vehicle = new Vehicle[2];
        vehicle[0] = new Car("宝马",180);
        vehicle[1] = new Bicycle("凤凰",30);
        for (int i = 0; i < vehicle.length; i++) {
            Vehicle vehicle1 = vehicle[i];
            if (vehicle1.getClass().equals(Car.class)) {
                vehicle1.drive(2);
            }else
                vehicle1.drive(3);
        }
    }
}

```

**题目: 请在main方法中通过多态创建对象，随后使用对象，使得程序符合最终运行结果**

```Java
public class Demo1 {
    /*
        请在main方法中通过多态创建对象，随后使用对象，使得程序符合最终运行结果
     */public static void main(String[] args) {
     Fu zi = new Zi();
     System.out.println(num);
     zi.method();
     Zi zi1 = (zi) Zi;
     zi1.show();


    }
}

class Fu {
    int num = 10;

    public void method(){
        System.out.println("父类的method方法");
    }
}

class Zi extends Fu{
    int num = 20;

    public void method(){
        System.out.println("子类的method方法");
    }

    public void show(){
        System.out.println("子类的show方法");
    }
}
```

运行结果：

```Java
10
子类的method方法
子类的show方法
```

**题目:**

**定义人(Person)类,人类中有吃饭方法（eat）和工作方法（work）。**

**定义超人（SuperMan）类，超人类中有救人方法（save）并且超人也属于人类。**

**要求：使用多态的方式创建超人对象，调用吃饭和工作的方法。再将此对象转为超人对象，调用救人的方法。**

**目标:  掌握多态的语法格式以及转型问题**

```Java
public class Person {
    public void eat(){
        System.out.println("干饭！");
    }
    public void work(){
        System.out.println("努力工作");
    }
}

public class SuperMan extends Person{
    public void save(){
        System.out.println("救人！");
    }

    @Override
    public void eat() {
        super.eat();
    }

    @Override
    public void work() {
        super.work();
    }
}

public class Test {
    public static void main(String[] args) {
        Person superMan = new SuperMan();
        superMan.eat();
        superMan.work();
        SuperMan superMan1 = (SuperMan) superMan;
        superMan1.save();
    }
}

```

## 课堂问答：抽象类与 String

**题目: 请描述抽象方法的特点**

**目标: 掌握抽象方法的特点**

```Java
抽象方法的特点是，在本类，即本抽象类中没有功能体，只能通过子类对该抽象方法重写实现功能
只有一个方法名，但无功能
```

**题目: 请描述抽象类的特点?**

**目标: 掌握抽象类的特点**

```Java
抽象类不能创建对象，仅作为一种特殊的父类，让子类继承并实现。
不能直接实例化
必须定义抽象类的子类，重写所有的抽象方法，然后使用抽象类的子类
```

**题目: 下面程序编译是否会报错并说明原因？并给出正确的代码！**

![这张图片展示了一段Java代码，是名为Animal的公共类定义。该类包含一个公共抽象方法eat，注释标注这是吃饭的抽象方法，类的定义以分号结尾，符合抽象方法的语法要求。同时还包含一个名为sleep的非抽象方法，方法内通过println语句输出“sleep方法”，注释说明了这是一个非抽象方法。该代码内容和文档中“掌握抽象类和抽象方法的特点”的学习目标相关，属于抽象类相关的代码示例。](../../../../../Attachments/development/programming/java/mian-xiang-dui-xiang/WZBBb5OEjopoYpxojdvcxZvGn4g.png)

**目标: 掌握抽象类和抽象方法的特点**

```Java
会报错，因为抽象方法必须在抽象类中才能实现，但抽象类中也不一定全是抽象方法，
所以要在定义类时也加上关键字 abstract
public abstract class Animal
```

**请分析如下程序是否会报错并说明原因？**

![这张图片展示了一段Java代码，属于面向对象多态相关的课堂内容。代码定义了抽象类Animal，再创建继承自Animal的Dog类，Dog类中实现了非抽象的eat方法，还包含main方法，main方法里通过多态格式创建了Animal类型的对象animal，实例化为Dog类的对象，接着调用animal的eat方法，右侧还标注了两行注释，分别说明该行是通过多态的方式创建Animal对象，以及调用eat方法。这段代码是用于分析该程序是否会报错的示例程序，对应文档中关于抽象类与多态的课堂问答任务。](../../../../../Attachments/development/programming/java/mian-xiang-dui-xiang/Kl4sbOroCogkBQxNnwVcf4LWnWf.png)

**目标: 掌握多态格式下成员的访问特点以及抽象类的注意事项**

```Java
会报错 ，子类调用父类类型，但是不能使用子类特有方法，所以会报错，
需要在父类中定义与子类方法同名的抽象类，子类才能使用父类类型使用，要么父类向下转型
父类修改为
public abstract class Animal{
    public abstract eat();
}
```

9、

**题目: 请分析如下程序在控制台的输出结果是什么并说明原因？以及如何避免该问题？**

![图片展示了一个面向对象编程中的多态示例代码。首先定义了一个抽象类Animal，其中包含一个抽象方法eat。接着分别定义了Cat和Dog类，它们都继承自Animal并重写了eat方法。最后在main方法中，通过多态方式创建Animal对象，将其转换为Cat类型，再调用eat方法。该图片与文档中课堂问答部分相关，用于说明抽象类与多态转型的注意事项。](../../../../../Attachments/development/programming/java/mian-xiang-dui-xiang/DqQvb9vwwoG4mJxiJw1c4PBlnzc.png)

**目标: 掌握多态转型以及和抽象类的注意事项**

```Java
会报错，原因是因为在通过多态创建对象时，是创建的子类中的Dog，
在对dog向下转型时，要转型的类型是同级子类cat，类型冲突，所以报错
既然需要使用的是cat的方法，那么应该将多态创建的子类构建函数改为Cat()，即：
Animal animal = new Cat();
```

**题目: 请将字符串遍历格式书写一遍**

**目标: 掌握遍历字符串的方式**

```Java
for (int i = 0; i < 字符串.lenght();i++){
    System.out.println(字符串.charAt(i));
}
```

**题目: 字符串中将内容变成大写和小写的方法是?**

**目标:掌握字符串常见的方法**

```Java
转换为大写：
    字符串.toUpperCase();
转换为小写：
    字符串.toLowerCase();

返回值为字符串类型
```

**题目: 字符串判是否包含指定内容的方法是?**

**目标: 掌握字符串常见的方法**

```Java
字符串.contains("指定内容")
返回值为boolean类型
```

**题目: 字符串中切割字符串的方法是? 注意事项有哪些?**

**目标: 掌握字符串常见的功能**

```Java
方法：
    字符串.split("切割标识")
注意：
    字符串切割完后，返回值类型为数组，且切割后的数组中不包含切割标识，切割标识作为分割点，全部丢弃了
    在使用切割标识中，尽量避免使用.作为切割标识，如果必须要用，要加上转义符\\.一起使用，才能生效
    字符串首尾出现连续字符，也尽量避免将连续字符作为切割标识，不然会出现连续的空字符串
```

**题目: 字符串中判断以某段字符串内容开头以及结尾的方法是?**

**目标: 掌握字符串常用功能**

```Java
开头：
    字符串startsWith("指定字段")
结尾：
    字符串.endsWith("指定字段")
```

**题目: 字符串截取子串的方法是?**

**目标: 掌握字符串的常用功能**

```Java
字符串.substring(截取字符串开始索引,截取字符串结束索引)
```

**题目: 请简述String的特点**

**目标: 考察字符串的特点**

```Java
字符串是Java最常用的类之一，用于表示字符串文本, String是一个不可改变的字符序列, 一旦创建无法修改
```

**题目: 请说明字符串为何不能使用==进行比较**

**目标: 考察字符串的注意事项**

```Java
==常用于数值类型的比较，比较的是其数值的地址，在用于比较字符串类型时，往往会输出错误对比，
这是因为字符串需要对比的是内容，往往相同的字符串地址未必相同。
如果要对比字符串最好使用 字符串.equals("比较内容") 来进行内容比较
```

**题目:**

**现在整理一堆目录下文件名的信息,**

**"d:/a.png","c:/aaa/a.txt", "d:/aaa/bbb/e.txt", "c:/c/aa/b.txt", "c:/aaa/g.txt", "c:/aaa/c.png", "d:/aaa/d.jpg", "d:/ccc/ddd/eee/g.mp3", "d:/nnn/mmm/kkkk/m.txt"**

**请将这些内容存到数组中, 然后通过遍历集合, 筛选出后缀名是.txt所有信息, 并且打印出文件名信息(注意不要前面文件夹的名字)**

**例如: "c:/aaa/a.txt"这个路径满足条件则应该输出 a.txt**

**目标: 掌握字符串的综合应用能力**

```TypeScript
    public static void main(String[] args) {
        fileName();
    }
    private static void fileName() {
        String[] filename = {"d:/a.png","c:/aaa/a.txt", "d:/aaa/bbb/e.txt", "c:/c/aa/b.txt", "c:/aaa/g.txt", "c:/aaa/c.png", "d:/aaa/d.jpg", "d:/ccc/ddd/eee/g.mp3", "d:/nnn/mmm/kkkk/m.txt"};
        for (int i = 0; i < filename.length; i++) {
            if (filename[i].contains(".txt")) {
//                System.out.println(filename[i]);
                String[] split = filename[i].split("/");
                for (int j = 0; j < split.length; j++) {
                if (split[j].endsWith(".txt"))
                    System.out.println(split[j]);
                }
            }
        }
    }
}
```

**题目:**

**现在很多网站都需要实名制, 所以不在让用户输入生日和年龄, 都是直接通过身份证号中获取,**

**请键盘录入自己的身份证号, 使用String的技术将生日的信息获取出来打印到控制台上**

**目标: 掌握字符串的综合应用能力**

```TypeScript
public static void main(String[] args) {
    idBirthday();
}

private static void idBirthday() {
    String id = "412722200012280058";
    String birthday = id.substring(6, 14);
    String year = birthday.substring(0,4);
    String month = birthday.substring(4,6);
    String day = birthday.substring(6,8);
    System.out.println("生日是：" + year + " 年" + month + "月" + day + "日");
}
```

**题目:**

**1.键盘录入一个大字符串,再录入一个小字符串**

**2.统计小字符串在大字符串中出现的次数**

**3.代码运行打印格式如下:**

**请输入大字符串:woaiheima,heimabutongyubaima,wulunheimahaishibaima,zhaodaogongzuojiushihaoma**

**请输入小字符串:heima**

**控制台输出:共出现3次**

**目标: 掌握字符串的综合应用能力**

```TypeScript
public static void main(String[] args) {
    bigSamllWord();
}

private static void bigSamllWord() {
    int count = 0;
    String[] word = {"woaiheima","heimabutongyubaima","wulunheimahaishibaima","zhaodaogongzuojiushihaoma"};
    for (int i = 0; i < word.length; i++) {
        if (word[i].contains("heima"))
            count++;
    }
    System.out.println("出现了：" + count + "次");
}
```

**题目:**

**请定义一个方法用于判断一个字符串是否是对称的字符串， 并在主方法中测试方法。**

**例如："abcba"、"上海自来水来自海上"均为对称字符串。**

**目标: 掌握字符串的综合应用能力**

```Java
public class SymmetyString {
    public static void main(String[] args) {
        symmetyString();
    }

    private static void symmetyString() {
        int count = 0;
        String word = "abcba";
        for (int i = 0; i < word.length()/2; i++) {
            for (int j = word.length()-1 -i; j >= word.length()-1-i; j--) {
                if (word.charAt(i) != word.charAt(j)) {
//                    System.out.println(word.charAt(i));
//                    System.out.println(word.charAt(j));
                    count++;
                }
            }
        }
        if (count == 0)
            System.out.println("字符串：" + word +"是对称字符串");
    }
}
```

**题目:**

**请编写程序，由键盘录入一个字符串，统计字符串中英文字母和数字分别有多少个。**

**比如：Hello12345World中字母：10个，数字：5个。**

**目标: 掌握字符串的综合应用能力**

```Java
public class CnAndEnCount {
    public static void main(String[] args) {
        cnEnCount();
    }

    private static void cnEnCount() {
        String word = "Hello12345World";
        int cn = 0;
        String num = "1234567890";
        for (int i = 0; i < word.length(); i++) {
            for (int j = 0; j < num.length(); j++) {
                if (word.charAt(i) == num.charAt(j)) {
                    cn++;
                }
            }
        }
        int en = word.length() - cn;
        System.out.println("数字有：" + cn + "个");
        System.out.println("字母有：" + en + "个");
    }
}
```

**题目:**

**在登录注册页面中，除了用户名和密码外，通常也会包含验证码。验证码是用来区分用户是计算机还是人， 防止恶意破解密码、刷票、灌水等行为。**

**请查看Random相关API， 定义方法，获取一个包含4个字符的验证码，每一位字符是随机选择的字母和数字，可包含a-z,A-Z,0-9。例如 7Kt5**

**目标: 掌握字符串的综合应用能力**

```Java
public class Login {
    public static void main(String[] args) {
        login();
    }
    private static void login() {
        // 定义一个字符串，包含所有字符
        String word = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        String code = "";
        Random random = new Random();
        for (int i = 0; i < 4; i++) {
            int index = random.nextInt(word.length());
            code += word.charAt(index);
        }
        System.out.println(code);
    }
}
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 面向对象 / 07 面向对象高级(多态)·笔记 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/K7P4wKhI1iqczLklEVLc88Rfnch)
