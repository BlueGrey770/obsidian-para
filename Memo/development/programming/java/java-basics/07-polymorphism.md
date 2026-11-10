---
type: memo
date: 2026-11-09
---

# 多态

## 认识多态

提到多态， 我们先来看一个生活场景：无论是猫、还是狗，都是动物, 但是却都会叫唤，但各自叫唤的方式又不同，这就是多态。即：叫唤这个事，放在不同的对象上（猫、狗），表现不同。

与此类似，在Java中，多态是指同一操作（方法调用）在不同对象上可以表现出不同的行为。多态是面向对象编程的重要特性之一，通常通过方法重写来实现。

多态的前提：

- 有继承/实现（下次课接口中涉及到）关系；
- 存在父类引用子类对象；
- 存在方法重写

多态示例：

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
        Animal myDog = new Dog();  // 创建 Dog 对象
        Animal myCat = new Cat();  // 创建 Cat 对象
        myDog.makeSound();     // 输出: 汪汪!
        myCat.makeSound();     // 输出: 喵喵!
    }
}
```

代码说明：

- 第27、28行：创建了Dog、Cat对象，但赋给了父类（Animal）的变量
- 第29行：调用myDog的方法，虽然myDog是Animal类型的变量，但是它指向的对象是子类型（Dog）的，所以调用的依然是Dog类的方法
- 第30行：调用的是Cat类中的方法，原理同29行

在上面代码中，myDog、myCat都是Animal类型的，而且调用的都是相同的方法（makeSound），但是输出结果完全不同，这就是多态。

多态的一个注意事项

- 多态是对象和行为的多态，Java中的属性(成员变量)不谈多态。

## 多态的好处

多态性是面向对象编程中的一个核心概念，它提供了许多重要的好处，使得代码更加灵活、可扩展和易于维护。通过多态，你可以编写通用的代码，这些代码可以处理不同类型的对象。这意味着你可以在不修改现有代码的情况下添加新的子类，从而实现功能的扩展。

**示例：**

假设你有一个动物园管理系统，其中包含多种动物。你可以定义一个 `Animal` 接口，并让每种动物实现这个接口：

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

class Zoo {
    public static void main(String[] args) {
        Animal dog = new Dog();
        Animal cat = new Cat();

        // 通用的方法来处理不同类型的动物
        makeAnimalSound(dog); // 输出: Bark bark!
        makeAnimalSound(cat); // 输出: Meow meow!
    }

    public static void makeAnimalSound(Animal animal) {
        animal.makeSound();
    }
}
```

代码说明：

- 如果没有多态机制，我们就需要为Cat、Dog分别写一个 makeAnimalSoud()方法：makeAnimalSound(Dog animal)、makeAnimalSound(Cat animal)。
- 第35行：提供了一个方法，可以指挥各种动物发出叫声，由于参数类型是父类型的，所以可以接收各种动物对象。这就是多态的方便之处。

总之：定义方法时，使用父类类型的形参，可以接收一切子类对象，扩展性更强、更便利。

案例:  请编写一个方法模拟榨汁机的榨汁功能, 完成不同水果榨汁的效果

```Java
public class Fruit {

   public void getJuice(){}
}
```

```Java
public class Apple extends Fruit{
    public void getJuice() {
        System.out.println("获取苹果汁~~~");
    }
}
```

```Java
public class Banana extends Fruit{

    public void getJuice() {
        System.out.println("获取香蕉汁~~~");
    }
}
```

没有采用多态完成榨汁的功能如下:

```Java
public class Test {

    public static void main(String[] args) {

        zha(new Apple());
        zha(new Orange());
        zha(new Banana());
        ...
    }

    public static void zha(Orange orange){
        orange.getJuice();
        System.out.println("加一些科技狠活~~~");
        System.out.println("放1克糖精");
        System.out.println("果葡糖浆");
        System.out.println("将其整合到杯中");

    }
    public static void zha(Apple apple){
        apple.getJuice();
        System.out.println("加一些科技狠活~~~");
        System.out.println("放1克糖精");
        System.out.println("果葡糖浆");
        System.out.println("将其整合到杯中");

    }
    ... 随着不同的水果, 方法根本写不完了,方法通用性太差了!!!!
}
```

采用多态完成榨汁的功能如下:

```Java
public class Test {

    public static void main(String[] args) {

        zha(new Apple());
        zha(new Orange());
        zha(new Banana());
        ...

    }
    // 采用父类类型作为参数提高了方法的扩展性!!!!!
    public static void zha(Fruit fruit) {
        fruit.getJuice();
        System.out.println("加一些科技狠活~~~");
        System.out.println("放1克糖精");
        System.out.println("果葡糖浆");
        System.out.println("将其整合到杯中");
    }

}
```

总结:

<callout emoji="🌷">
多态经常用于在定义方法的时候, 让**父类类型作为参数**, 可以行为多态, 从而接受所有该父类类型对应的子类对象提高方法的**扩展性,通用性**
</callout>

## 多态下的类型转换

刚才我们认识了java中的多态机制，其给我们带来了编码的灵活性。但同时也有一些问题：由于多态是通过父类变量指向子类对象，那么我们**通过父类变量只能访问父类中的方法，而无法访问到子类中特有的方法**。

下面是一个简单的示例来演示这种情况：

```Java
public class Animal {
    public void eat() {
        System.out.println("动物在吃东西...");
    }
}

public class Dog extends Animal {
    @Override
    public void eat() {
        System.out.println("狗在吃东西...");
    }

    public void sound() {
        System.out.println("狗在叫唤...");
    }
}

public class PolymorphismDemo {
    public static void main(String[] args) {
        // 通过 Animal 引用指向 Dog 对象
        Animal animal = new Dog();

        // 通过 Animal 引用调用方法
        animal.eat(); // 输出: 狗在吃东西...

        // 尝试通过 Animal 引用调用子类特有的方法
        // 这行代码会导致编译错误
        animal.sound(); // 编译错误：Animal 类中没有 bark 方法
    }
}
```

上面代码中，第28行代码会报编译错误，因为 Animal 类中并没有 sound 方法。

那么如何解决这个问题呢？ java中提供了类型转换，我们可以将 animal 转换成 子类（Dog）类型，然后再调用子类特有的方法：

```Java
public class PolymorphismDemo {
    public static void main(String[] args) {
        // 通过 Animal 引用指向 Dog 对象
        Animal animal = new Dog();

        // 通过 Animal 引用调用方法
        animal.eat(); // 输出: 狗在吃东西...

        // 尝试通过 Animal 引用调用子类特有的方法
        // 这行代码会导致编译错误
        ( (Dog)animal ).sound(); // 类型转换
    }
}
```

上面代码可以正常运行，第11行将animal强制转换成了Dog类型，这样就可以调用子类Dog类中的方法了。

虽然类型转换可以解决调用子类特有方法的问题，但我们在进行类型转换时，需要保证对象确实是这种类型的。如果对象的真实类型与强转后的类型不同，就会报类型转换异常（ClassCastException）的错误。

比如：下面代码中，要强制将animal对象转成Cat类型，就会报错：

```Java
Animal animal = new Dog();
Cat cat = (Cat)animal
```

那么如何避免这种类型不匹配问题的发生呢？我们可以在进行类型转换之前，先判断一下对象是否是此类型的。java中提供了instanceof关键字，用于检查对象是否是指定的类型。它通常用于确定对象的具体类型，特别是在多态性的情况下，当你有一个父类引用指向子类对象时，可以使用 `instanceof` 来判断该引用实际指向的对象的具体类型。

在进行向下转型（从父类引用转换为子类引用）之前，使用 `instanceof` 可以确保转型的安全性，避免 `ClassCastException` 异常，代码示例如下：

```Java
Animal animal = new Dog();
if( animal instanceof Cat){
  Cat cat = (Cat)animal
}else{
  System.out.println("真实对象不是Cat，不能进行类型转换!");
}

```

## 综合小案例-加油站支付

前面我们学习了继承、多态的有关知识，接下来我们综合运用这些技术点来完成一个综合案例。

**案例需求**：请使用面向对象编程，完成该加油站支付机的办卡以及消费程序

**需求描述**：

1. 某加油站为了吸引更多的车主，推出了如下活动：车主通过充值对应的金额办理金卡和银卡。

(本卡只是在办卡的时候一次性充值, 消费完后卡片回收)

1. 卡片信息包括：车牌号码、车主姓名、电话号码、卡片余额。
2. 办卡流程：金卡办理时入存金额必须>=5000元，银卡办理时预存金额必须>=2000元
3. 支付优惠：金卡支付时享受8折优惠，银卡支付时享受9折优惠
4. 金卡满送：金卡消费满200元可以赠送一张洗车券
5. 需要定义加油站刷卡功能, 支持各种卡的消费

**思路分析：**

- 由于充值卡分金卡、银卡，且基本属性相同（车牌号码、车主姓名、电话号码、卡片余额），所以可以将基本属性编写在父类中，然后由金卡、银卡两个子类继承，减少重复代码。
- 两种卡都有支付功能，但支付时打折额度不相同，可以在父类中提供一个支付方法，由两种卡进行方法重写，各自实现自己的支付逻辑，这里体现了多态机制。
- 定义加油站类, 可以编写刷卡功能, 该功能可以接受各种卡, 可以采用父类接受所有的卡片

    这里体现了多态机制

**代码实现：**

（1）卡类

```Java
package cn.itcast.polymorphic.demo9;

/*
    定义父类编写共享的属性和行为
 */
public class Card {

    //属性:
    private String carId;
    private String name;
    private String phone;
    private double money;

    public Card() {
    }

    public Card(String carId, String name, String phone, double money) {
        this.carId = carId;
        this.name = name;
        this.phone = phone;
        this.money = money;
    }

    public String getCarId() {
        return carId;
    }

    public void setCarId(String carId) {
        this.carId = carId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public double getMoney() {
        return money;
    }

    public void setMoney(double money) {
        this.money = money;
    }
    //行为:
    public void pay(double money){
        //看钱够不够
        if(this.money >= money){
            this.money -= money;
            System.out.println("尊贵的普卡用户, 你本次消费"+money+"元, 卡片余额是:"+ this.money);
        }else{
            System.out.println("卡片余额不足, 请重新购卡");
        }
    }

}
```

（2）金卡类

```Java
package cn.itcast.polymorphic.demo9;

public class GoldCard extends Card {

    public GoldCard() {
    }

    public GoldCard(String carId, String name, String phone, double money) {
        super(carId, name, phone, money);
    }

    @Override
    public void pay(double money) {
        // 打折
        double discountMoney = money * 0.8;
        //在看余额是否足够
        if (this.getMoney() >= discountMoney) {
            setMoney(getMoney() - discountMoney);
            System.out.println("尊贵的金卡用户, 你本次应消费" + money + "元, 享受8折优惠后,实际消费" + discountMoney + "元,目前余额是:" + getMoney() + "元");
            if(discountMoney >= 200){
                System.out.println("由于你实际消费满200元,赠送你的洗车劵一张,祝你用车愉快~~~~");
            }
        }else{
            System.out.println("卡片余额不足, 请重新购卡");
        }
    }
}
```

（3）银卡类

```Java
package cn.itcast.polymorphic.demo9;

public class SliverCard extends Card{

    public SliverCard() {
    }

    public SliverCard(String carId, String name, String phone, double money) {
        super(carId, name, phone, money);
    }

    @Override
    public void pay(double money) {
        double discountMoney = money * 0.9;
        if(getMoney() >= discountMoney){
            setMoney(getMoney() - discountMoney);
            System.out.println("尊贵的银卡用户, 您本次应消费"+money + "元, 享受9折优惠后, 实际消费" + discountMoney + "元, 目前余额是:" + getMoney() + "元");
        }else{
            System.out.println("卡片余额不足, 请重新购卡");
        }
    }
}
```

  (4) 加油站类

```Java
public class Station {
    //定义开卡的功能
    public Card openCard(){
        Scanner sc = new Scanner(System.in);
        System.out.println("1.请输入您的姓名");
        String name = sc.next();
        System.out.println("2.请输入您的手机号");
        String phone = sc.next();
        System.out.println("3.请输入车牌号");
        String carId = sc.next();
        System.out.println("请输入要充值的金额");
        double money = sc.nextDouble();
        if(money >= 5000){
            return new GoldCard(carId, name, phone, money);
        }else if(money >= 2000){
            return new SliverCard(carId, name, phone, money);
        }else {
            return new Card(carId, name, phone, money);
        }
    }

    //定义刷卡的功能
    public void posPay(Card card, int money){
        card.pay(money);
        System.out.println("打印发票");
    }
}
```

（5）测试

```Java
public class Test {

    public static void main(String[] args) {
        //1.创建加油站
        Station station = new Station();
        //2.调用加油站开卡逻辑
        Card card = station.openCard();
        //3.输入消费金额
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入本次消费的金额");
        int money = sc.nextInt();
        //4.调用加油站的刷卡逻辑
        station.posPay(card, money);
    }
}
```

运行测试类，结果如下：

![Kqagbfq03oCVKXxS7t9cvtQEn42](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Kqagbfq03oCVKXxS7t9cvtQEn42.png)

由上可见：

- 金卡消费时打了8折，银卡消费时打了9折。都是消费功能，但金卡、银卡的具体功能不相同，这里体现了多态的特性。

## 相关笔记

- [[Memo/development/programming/java/java-basics/07-polymorphism-quiz.md|07 多态·问答]]
- [[Memo/development/programming/java/oop/10-polymorphism-abstract-final-singleton-enum.md|多态、抽象类、final、单例与枚举精讲]]

---

> 来源：飞书《讲义》知识库 · day07-面相对象高级(多态) · [原文](https://baxsu7o100e.feishu.cn/wiki/F6OkwGgD1iQvWpbwLmpXQkMSVig)
