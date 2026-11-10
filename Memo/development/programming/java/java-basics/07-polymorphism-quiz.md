---
type: memo
date: 2026-11-06
aliases:
  - "07-mian-xiang-dui-xiang-gao-ji-duo-tai"
  - "07 面向对象高级(多态)·笔记"
---

# 多态·问答
> 系统详解：[[Memo/development/programming/java/java-basics/07-polymorphism.md|多态]]

## 问答：多态

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
