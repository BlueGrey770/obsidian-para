---
type: memo
date: 2026-11-09
aliases:
  - "ArrayList入门"
  - "09 ArrayList与ATM·问答"
---

# ArrayList 与 ATM 综合实战

## 认识集合

什么是集合?

集合是一种容器，用来装数据的，类似于数组。

集合和数组的区别是什么?

1. 集合长度可变, 数组长度不可改变
2. 集合只能存储引用数据类型的数据, 而数组可以存储基本数据类型以及引用类型

## ArrayList集合

JDK提供了非常多种类的集合供我们使用, 但是我们基础班, 只介绍ArrayList集合, 其余的集合我们在就业班前期

会讲解

### ArrayList的创建以及添加功能

```Java
public ArrayList​()  构造

boolean add(E e); 添加功能
```

例如:

```Java
public static void demo1() {
    //创建集合对象
    ArrayList al = new ArrayList();
    //访问功能
    al.add("梅军建");
    al.add(new Object());
    al.add(new Student());
    System.out.println(al);
}
```

演示完后, 我们发现集合果然是长度可以改变的,  但是默认集合可以存储任何的引用数据类型, 但是如果容器中存储的数据类型不统一的话, 将来以后遍历批量操作的时候无法做到统一,那该怎么办呢???

### 泛型的简单介绍

基于刚才的问题这个大家无须担心, jdk考虑已经考虑过这个问题, 在JDK5的时候提供了泛型机制, 我们翻阅API文档, 发现ArrayList<E>的类名后面写一个<E> 格式语法, 这种语法叫做泛型类, 我们在进阶会专业讲解泛型类, 泛型接口, 泛型方法等,  试学班, 我们只需要记住**泛型的作用**即可 , 他是用来**限定集合存储的数据类型**用的,我们可以在创建集合对象的时候, 确定一下集合存储的数据类型, 那么在次使用添加方法的时候, 不能往集合中添加其他数据类型了, 例如:

```Java
public static void demo1() {
    //创建集合对象
    ArrayList<String> al = new ArrayList();
    //ArrayList<int> al = new ArrayList();// 这个是典型的错误, 泛型不能确定为基本数据类型
    //访问功能
    al.add("梅军建");
    //al.add(new Object());//报错,该集合中只能存储String类型的数据
    //al.add(new Student());//报错,该集合中只能存储String类型的数据
    System.out.println(al);
}
```

<callout emoji="🌷">
注意:
1. 泛型只能规定为引用数据类型
2. 遇到泛型类我们必须确定泛型的类型, 否则会被jvm默认确定为Object
</callout>

集合如何存储基本数据类型的数据?

### 基本数据类型对应的包装类

#### 认识包装类

jdk其实给所有的基本数据类型编写了其对应的引用类型, 我们称基本类型对应的引用类型叫做包装类, 分别如下:

| 基本数据类型 |  包装类型 |
|-|-|
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
| float | Float |
| double | Double |
| char | Character |
| boolean | Boolean |

#### 拆箱和装箱

JDK1.5的时候为了让大家使用的更方便, 出现了自动拆装箱的机制

什么是拆箱???

将包装类型转成对应的基本数据类型, 叫做拆箱

什么是装箱???

将基本数据类型转成对应的包装类型,叫做装箱

自动拆装箱???

JDK1.5出现的自动拆装箱, 基本数据类型和包装类的转换是自发的, 不需要我们手动拆箱和装箱就可以完成操作

那么我们可以直接将包装类看成基本数据类型使用即可

例如:

```Java
private static void demo5() {
    ArrayList<Integer> list = new ArrayList<>();
    // ArrayList<Double> list1 = new ArrayList<>();
    list.add(1);// 这里其实发生了自动装箱
    list.add(2);// 这里其实发生了自动装箱
    list.add(3);// 这里其实发生了自动装箱
    //求出集合元素和
    int sum = 0;
    //遍历集合得到元素累加到sum中
    for (int i = 0; i < list.size(); i++) {
        sum+=  list.get(i); // 引用类型其不能直接运算的,这里其实发生了自动拆箱
    }
    System.out.println(sum);
}
```

#### 基本类型和包装类的区别

1. 基本类型不能存储null值, 但是包装类可以接受null值
2. 基本数据类型没有功能, 而包装类有成员数据和方法可以使用

注意: 包装类的其他属性和功能我们在进阶阶段使用到的时候还是介绍

### ArrayList常见的功能

```Java
boolean add(E e);
        向集合末尾添加元素,返回是否添加成功, ArrayList恒成功
void add(int index, E e);
        根据索引添加元素, 原索引处以及之后的元素会往后顺延
E get(int index);
        根据索引获取元素
E remove(int index);
        根据索引删除元素, 返回被删除的元素, 该索引之后的元素都会往前顺延
boolean remove(Object obj);
        根据元素删除, 底层拿着当前参数的元素去和集合中的所有元素采用equals方法比较, 如果相同了,则删除满足条件第一个元素,删除完后, 后面的元素往前顺延
        要求我们元素必须重写equals方法才行
E set(int index , E  newElement);
        根据索引修改元素, 返回被修改的旧元素
int size();
        获取集合中的元素个数
boolean contains(Object obj);
        判断集合中是否包含某一个元素
        底层拿着元素和集合中的元素进行equals对比但是有相同的则包含!!!
int indexOf(Object obj);
        获取元素在集合中的索引位置, 默认也是采用equals方法在比较, -1代表不存在
```

演示方法的代码:

```Java
private static void demo2() {
    //1.创建集合
    ArrayList<String> list = new ArrayList<>();
    //2.访问功能
    //添加 --->  add(E e); 添加元素到集合的末尾
    list.add("梅军建");
    list.add("林青霞");
    list.add("黎姿");
    //根据索引添加, 插入元素 add(int index, E element); 在index位置存入元素, 原来索引以及之后的元素往后顺延!!!
    //list.add(2,"梅军建");
    // 根据索引获取元素  E  get(int index);
    String s = list.get(1);
    System.out.println(s);
    //获取集合的长度:  int size();
    int size = list.size();
    System.out.println(size);
    /*
        根据元素删除元素boolean remove(Object obj); 删除和参数内容一致的元素,底层使用equals在比较, 默认删除第一个!!!
        要求元素必须重写equals方法才行, 否则可能删除的跟你想的不一样
     */
    //list.remove("梅军建");
    // E  remove(int index); 根据索引删除元素, 返回被删除的元素, 被删除索引之后的元素会往前顺延!!!
    // String remove = list.remove(1);
    // System.out.println(remove);
    String s1 = list.set(2, "豆腐妹");
    System.out.println(s1);
    System.out.println(list);

}
```

```Java
private static void demo4() {
    ArrayList<String> list = new ArrayList<>();
    list.add("梅军建");
    list.add("林青霞");
    list.add("黎姿");

    boolean result = list.contains("豆腐妹");
    System.out.println(result);

    int i1 = list.indexOf("豆腐妹");
    System.out.println(i1);

}
```

### 集合的遍历

遍历: 访问集合的每一个元素过程叫做遍历集合

我们可以集合int size()和 int get(int index)方法来完成集合的遍历

例如:

```Java
private static void demo5() {
    ArrayList<String> list = new ArrayList<>();
    list.add("梅军建");
    list.add("林青霞");
    list.add("黎姿");
    for (int i = 0; i < list.size(); i++) {
        String s = list.get(i);
        System.out.println(s);
    }
    System.out.println(list);
}
```

## ATM综合案例

恭喜大家, 已经成功入门java, 接下来我们准备完成一个综合案例, 来对我们基础班级的课程进行综合联系, 这个案例叫做ATM, 我们应该不是很陌生对这个综合案例来说, 比如下图:

**登录ATM系统**

![HMh3bkLreoGyJnxNdrUcKG0wn3d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/HMh3bkLreoGyJnxNdrUcKG0wn3d.png)

**登录ATM系统后的菜单**

![HiYKb9fEIoJr9Axyyl3c5RwVnjd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/HiYKb9fEIoJr9Axyyl3c5RwVnjd.png)

**菜单中一些功能:**

![YhMJblskho5G1bx68eMctvaGnxb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/YhMJblskho5G1bx68eMctvaGnxb.png)

### 技术选型

1. 面向对象编程:

每个账户都是一个账户对象：所以需要设计账户类Account，用于创建账户对象封装账户信息。ATM同样是一个对象，需要设计ATM类，代表ATM管理系统，负责对账户的处理

1. 使用集合容器:

ATM管理类中需要提供一个容器用于存储系统全部账户对象的信息，我们选ArrayList集合。

1. 程序流程控制:

需要结合分支、循环、跳转关键字等程序流程控制的知识，用来控制系统的业务流程。

1. 使用常见API:

登录信息的内容比较，数据的分析，处理等都需要常用API来解决，如使用String。

### 预期效果

1. 收获一: 面向对象编程能力
2. 收获二: 清晰、缜密的业务分析能力、数据处理能力。
3. 收获三: 熟练使用程序流程语句来控制计算机完成自己的想法
4. 收获四: 良好的编码习惯，一定的编码经验，让基础知识形成体系，为后续Java高级技术的学习做有力的支撑

### 功能

#### 系统架构搭建、欢迎页设计

充分采用面向对象的思想完成: 我们的主方法所在的类, 就是为了启动程序,  而展示菜单属于ATM的功能, 我们准备放入到ATM中编写功能完成, 而且这个主菜单应该是无限循环, 直接用户选择退出才行

```Java
public class App {

    public static void main(String[] args) throws IOException {
        //启动类,就是为了启动程序用的, 主菜单属于ATM的功能,将主菜单收
        ATM atm  = new ATM();
        atm.showMainMenu();
    }
}
```

```Java
public class ATM {
    /**
     * 作用: 展示一级菜单
     */
    public void showMainMenu() throws IOException {
        // alt + ctrl + T
        while (true) {
                System.out.println("~~~~~~~欢迎来到黑马ATM~~~~~~~~");
                System.out.println("1.登录");
                System.out.println("2.注册");
                System.out.println("3.退出");
                System.out.println("请选择编号:");
                Scanner sc = new Scanner(System.in);
                int choose = sc.nextInt();

                switch (choose) {
                    case 1 -> {
                         System.out.println("处理登录逻辑");
                    }
                    case 2 -> {
                        System.out.println("处理注解逻辑");
                    }
                    case 3 -> {
                        System.out.println("退卡操作");
                        return; //结束方法
                    }
                }
            }
        }
    }
}
```

效果:

![CQYTbMILuoDxrEx4dtHcM4PAnEe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/CQYTbMILuoDxrEx4dtHcM4PAnEe.png)

#### 开户功能实现

我们需要定义账户类, 来封装账户有关的数据, 还需要在成员变量区域定义集合来存储所有的账户, 最后在ATM中编写开户的功能,

```Java
public class Account {

    //属性:
    private String cardId;
    private String password;
    private String name;
    private String gender;
    private double money;
    private double limit = 3000;
    private String phone;

    public Account() {
    }

    public Account(String cardId, String password, String name, String gender, double money, double limit, String phone) {
        this.cardId = cardId;
        this.password = password;
        this.name = name;
        this.gender = gender;
        this.money = money;
        this.limit = limit;
        this.phone = phone;
    }
    // 额外添加的功能方便后期根据性别获取尊称
    public String getNick(){
        return gender.equals("男") ? "先生":"女士";
    }

    public String getCardId() {
        return cardId;
    }

    public void setCardId(String cardId) {
        this.cardId = cardId;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public double getMoney() {
        return money;
    }

    public void setMoney(double money) {
        this.money = money;
    }

    public double getLimit() {
        return limit;
    }

    public void setLimit(double limit) {
        this.limit = limit;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    @Override
    public String toString() {
        return "Account{" +
                "cardId='" + cardId + '\'' +
                ", password='" + password + '\'' +
                ", name='" + name + '\'' +
                ", gender='" + gender + '\'' +
                ", money=" + money +
                ", limit=" + limit +
                ", phone='" + phone + '\'' +
                '}';
    }
}
```

```Java
public class ATM {
    //定义集合用于存储所有的账户信息
    private ArrayList<Account> accounts = new ArrayList<>();
    /**
     * 作用: 展示一级菜单
     */
    public void showMainMenu() throws IOException {
        // alt + ctrl + T
        while (true) {
                System.out.println("~~~~~~~欢迎来到黑马ATM~~~~~~~~");
                System.out.println("1.登录");
                System.out.println("2.注册");
                System.out.println("3.退出");
                System.out.println("请选择编号:");
                Scanner sc = new Scanner(System.in);
                int choose = sc.nextInt();

                switch (choose) {
                    case 1 -> {
                         System.out.println("处理登录逻辑");
                    }
                    case 2 -> {
                        register();
                    }
                    case 3 -> {
                        System.out.println("退卡操作");
                        return; //结束方法
                    }
                }
            }
        }
    }
    /**
     * 作用: 开户功能
     */
    public void register() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入姓名");
        String name = sc.next();
        System.out.println("请输入性别");
        String gender = sc.next();
        String password;
        while (true) {
            System.out.println("请设置密码");
            password = sc.next();
            System.out.println("请确认密码");
            String confirmPassword = sc.next();
            if (password.equals(confirmPassword)) {
                break;
            } else {
                System.out.println("两次密码不一致, 请重新设置");
            }
        }
        System.out.println("请输入限额");
        double limit = sc.nextDouble();
        System.out.println("请输入手机号");
        String phone = sc.next();
        // 生成卡号 --> 随机长度10的字符串,作为卡号
        String cardId = generateCardId();
        //将数据封装成账户对象
        Account account = new Account(cardId, password, name, gender, 0, limit, phone);
        //将账户保存到atm的集合(保存到数据库中)
        accounts.add(account);
        System.out.println("恭喜您注入成功,请收好您的卡片,您的卡号是:" + cardId);
    }
    /**
     * 作用: 生成卡号
     *
     * @return 返回生成的卡号
     */
    public String generateCardId() {
        // String str = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        String str = "";
        for (int i = 0; i < 10; i++) {
            str += i;
        }
        for (char c = 'A'; c <= 'Z'; c++) {
            str += c;
        }
        for (char c = 'a'; c <= 'z'; c++) {
            str += c;
        }
        String cardId = "";
        //循环10次, 生成一个卡号
        Random r = new Random();
        for (int i = 0; i < 10; i++) {
            int index = r.nextInt(str.length());
            //取出索引对应的字符
            char c = str.charAt(index);
            cardId += c;
        }
        return cardId;
    }
}
```

标注黄色就是刚刚添加的新功能的难点以及关键点

#### 登录功能实现

注册成功后, 我们为了不让每次启动都要先注册才能登录, 我们可以给集合设置一些默认数据, 这样在做后期功能的时候可以简单测试的步骤, 所谓的登录就是输入卡号和密码, 然后从集合中所有账户中查询的一个动作, 但是值得注意的是, 登录后的结果应该是根据卡号密码查出来的完整的账户对象才对, 而且这个对象还需要放入成员区域, 因为一会登录进入的二级菜单中都需要用到这个数据

```Java
public class ATM {
    //定义集合用于存储所有的账户信息
    private ArrayList<Account> accounts = new ArrayList<>();
    //用于记录登录成功的完整账户对象
    private Account currentAccount;
    // 在构建ATM的时候, 可以设置默认的账户, 提交后面功能测试效率
    public ATM() throws IOException {
        accounts.add(new Account("1234567890", "123456", "梅军建", "男", 10000, 5000, "13888888888"));
        accounts.add(new Account("1234567891", "123456", "林青霞", "女", 1000000, 500000, "13888888889"));

    }
    /**
     * 作用: 展示一级菜单
     */
    public void showMainMenu() throws IOException {
        // alt + ctrl + T
        while (true) {
                System.out.println("~~~~~~~欢迎来到黑马ATM~~~~~~~~");
                System.out.println("1.登录");
                System.out.println("2.注册");
                System.out.println("3.退出");
                System.out.println("请选择编号:");
                Scanner sc = new Scanner(System.in);
                int choose = sc.nextInt();

                switch (choose) {
                    case 1 -> {
                        login();
                    }
                    case 2 -> {
                        register();
                    }
                    case 3 -> {
                        System.out.println("退卡操作");
                        return; //结束方法
                    }
                }
            }
        }
    }
    /**
     * 作用: 根据卡号和密码登录
     */
    public void login() throws IOException {
        //接受用户输入的卡号和密码
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入卡号");
        String cardId = sc.next();
        System.out.println("请输入密码");
        String password = sc.next();
        //在accounts集合中查询 和输入的卡号和密码一致的账户
        Account login = null;
        for (int i = 0; i < accounts.size(); i++) {
            Account account = accounts.get(i);
            if (account.getCardId().equals(cardId) && account.getPassword().equals(password)) {
                //登录成功
                login = account;
                break;
            }
        }
        //判断login是否为null;
        if (login == null) {
            System.out.println("用户名或密码错误");
        } else {
            System.out.println("登录成功");
            //将查询出来的账户放到成员区域, 方便其他功能找到数据
            this.currentAccount = login;
            //展示二级菜单
            showSecondMenu();
        }
    }

    /**
     * 作用: 展示二级菜单
     */
    public void showSecondMenu() throws IOException {
        while (true) {
            System.out.println("==================欢迎" + currentAccount.getName() + currentAccount.getNick() + "进入ATM系统==================");
            System.out.println("1.查询账户信息");
            System.out.println("2.取款");
            System.out.println("3.存款");
            System.out.println("4.转账");
            System.out.println("5.修改密码");
            System.out.println("6.注销");
            System.out.println("7.退出");
            System.out.println("请输入你的操作");
            Scanner scanner = new Scanner(System.in);
            int choose = scanner.nextInt();
            switch (choose) {
                case 1 -> {
                    System.out.println("查询账户信息");
                }
                case 2 -> {
                    System.out.println("取款");
                }
                case 3 -> {
                    System.out.println("存款");
                }
                case 4 -> {
                    System.out.println("转账");
                }
                case 5 -> {
                    System.out.println("修改密码");
                case 6 -> {
                }
                case 7 -> {
                    System.out.println("请收好您的卡片");
                    return;
                }
            }
        }
    }
    /**
     * 作用: 开户功能
     */
    public void register() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入姓名");
        String name = sc.next();
        System.out.println("请输入性别");
        String gender = sc.next();
        String password;
        while (true) {
            System.out.println("请设置密码");
            password = sc.next();
            System.out.println("请确认密码");
            String confirmPassword = sc.next();
            if (password.equals(confirmPassword)) {
                break;
            } else {
                System.out.println("两次密码不一致, 请重新设置");
            }
        }
        System.out.println("请输入限额");
        double limit = sc.nextDouble();
        System.out.println("请输入手机号");
        String phone = sc.next();
        // 生成卡号 --> 随机长度10的字符串,作为卡号
        String cardId = generateCardId();
        //将数据封装成账户对象
        Account account = new Account(cardId, password, name, gender, 0, limit, phone);
        //将账户保存到atm的集合(保存到数据库中)
        accounts.add(account);
        System.out.println("恭喜您注入成功,请收好您的卡片,您的卡号是:" + cardId);
    }
    /**
     * 作用: 生成卡号
     *
     * @return 返回生成的卡号
     */
    public String generateCardId() {
        // String str = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        String str = "";
        for (int i = 0; i < 10; i++) {
            str += i;
        }
        for (char c = 'A'; c <= 'Z'; c++) {
            str += c;
        }
        for (char c = 'a'; c <= 'z'; c++) {
            str += c;
        }
        String cardId = "";
        //循环10次, 生成一个卡号
        Random r = new Random();
        for (int i = 0; i < 10; i++) {
            int index = r.nextInt(str.length());
            //取出索引对应的字符
            char c = str.charAt(index);
            cardId += c;
        }
        return cardId;
    }
}
```

#### 查询账户

```Java
public class ATM {
    //定义集合用于存储所有的账户信息
    private ArrayList<Account> accounts = new ArrayList<>();
    //用于记录登录成功的完整账户对象
    private Account currentAccount;
    // 在构建ATM的时候, 可以设置默认的账户, 提交后面功能测试效率
    public ATM() throws IOException {
        accounts.add(new Account("1234567890", "123456", "梅军建", "男", 10000, 5000, "13888888888"));
        accounts.add(new Account("1234567891", "123456", "林青霞", "女", 1000000, 500000, "13888888889"));

    }
    /**
     * 作用: 展示一级菜单
     */
    public void showMainMenu() throws IOException {
        // alt + ctrl + T
        while (true) {
                System.out.println("~~~~~~~欢迎来到黑马ATM~~~~~~~~");
                System.out.println("1.登录");
                System.out.println("2.注册");
                System.out.println("3.退出");
                System.out.println("请选择编号:");
                Scanner sc = new Scanner(System.in);
                int choose = sc.nextInt();

                switch (choose) {
                    case 1 -> {
                        login();
                    }
                    case 2 -> {
                        register();
                    }
                    case 3 -> {
                        System.out.println("退卡操作");
                        return; //结束方法
                    }
                }
            }
        }
    }
    /**
     * 作用: 根据卡号和密码登录
     */
    public void login() throws IOException {
        //接受用户输入的卡号和密码
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入卡号");
        String cardId = sc.next();
        System.out.println("请输入密码");
        String password = sc.next();
        //在accounts集合中查询 和输入的卡号和密码一致的账户
        Account login = null;
        for (int i = 0; i < accounts.size(); i++) {
            Account account = accounts.get(i);
            if (account.getCardId().equals(cardId) && account.getPassword().equals(password)) {
                //登录成功
                login = account;
                break;
            }
        }
        //判断login是否为null;
        if (login == null) {
            System.out.println("用户名或密码错误");
        } else {
            System.out.println("登录成功");
            //将查询出来的账户放到成员区域, 方便其他功能找到数据
            this.currentAccount = login;
            //展示二级菜单
            showSecondMenu();
        }
    }

    /**
     * 作用: 展示二级菜单
     */
    public void showSecondMenu() throws IOException {
        while (true) {
            System.out.println("==================欢迎" + currentAccount.getName() + currentAccount.getNick() + "进入ATM系统==================");
            System.out.println("1.查询账户信息");
            System.out.println("2.取款");
            System.out.println("3.存款");
            System.out.println("4.转账");
            System.out.println("5.修改密码");
            System.out.println("6.注销");
            System.out.println("7.退出");
            System.out.println("请输入你的操作");
            Scanner scanner = new Scanner(System.in);
            int choose = scanner.nextInt();
            switch (choose) {
                case 1 -> {
                    showAccountInfo();
                }
                case 2 -> {
                    System.out.println("取款");
                }
                case 3 -> {
                    System.out.println("存款");
                }
                case 4 -> {
                    System.out.println("转账");
                }
                case 5 -> {
                    System.out.println("修改密码");
                case 6 -> {
                }
                case 7 -> {
                    System.out.println("请收好您的卡片");
                    return;
                }
            }
        }
    }
    /**
     * 作用: 展示当前账户的信息
     */
    public void showAccountInfo() {
        System.out.println("卡号是::" + currentAccount.getCardId());
        System.out.println("姓名是::" + currentAccount.getName());
        System.out.println("性别是::" + currentAccount.getGender());
        System.out.println("余额是::" + currentAccount.getMoney());
        System.out.println("手机号是::" + currentAccount.getPhone());
        System.out.println("昵称是::" + currentAccount.getNick());
        System.out.println("额度是::" + currentAccount.getLimit());

    }
    /**
     * 作用: 开户功能
     */
    public void register() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入姓名");
        String name = sc.next();
        System.out.println("请输入性别");
        String gender = sc.next();
        String password;
        while (true) {
            System.out.println("请设置密码");
            password = sc.next();
            System.out.println("请确认密码");
            String confirmPassword = sc.next();
            if (password.equals(confirmPassword)) {
                break;
            } else {
                System.out.println("两次密码不一致, 请重新设置");
            }
        }
        System.out.println("请输入限额");
        double limit = sc.nextDouble();
        System.out.println("请输入手机号");
        String phone = sc.next();
        // 生成卡号 --> 随机长度10的字符串,作为卡号
        String cardId = generateCardId();
        //将数据封装成账户对象
        Account account = new Account(cardId, password, name, gender, 0, limit, phone);
        //将账户保存到atm的集合(保存到数据库中)
        accounts.add(account);
        System.out.println("恭喜您注入成功,请收好您的卡片,您的卡号是:" + cardId);
    }
    /**
     * 作用: 生成卡号
     *
     * @return 返回生成的卡号
     */
    public String generateCardId() {
        // String str = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        String str = "";
        for (int i = 0; i < 10; i++) {
            str += i;
        }
        for (char c = 'A'; c <= 'Z'; c++) {
            str += c;
        }
        for (char c = 'a'; c <= 'z'; c++) {
            str += c;
        }
        String cardId = "";
        //循环10次, 生成一个卡号
        Random r = new Random();
        for (int i = 0; i < 10; i++) {
            int index = r.nextInt(str.length());
            //取出索引对应的字符
            char c = str.charAt(index);
            cardId += c;
        }
        return cardId;
    }
}
```

#### 存款、取款功能实现

```Java
public class ATM {
    //定义集合用于存储所有的账户信息
    private ArrayList<Account> accounts = new ArrayList<>();
    //用于记录登录成功的完整账户对象
    private Account currentAccount;
    // 在构建ATM的时候, 可以设置默认的账户, 提交后面功能测试效率
    public ATM() throws IOException {
        accounts.add(new Account("1234567890", "123456", "梅军建", "男", 10000, 5000, "13888888888"));
        accounts.add(new Account("1234567891", "123456", "林青霞", "女", 1000000, 500000, "13888888889"));

    }
    /**
     * 作用: 展示一级菜单
     */
    public void showMainMenu() throws IOException {
        // alt + ctrl + T
        while (true) {
                System.out.println("~~~~~~~欢迎来到黑马ATM~~~~~~~~");
                System.out.println("1.登录");
                System.out.println("2.注册");
                System.out.println("3.退出");
                System.out.println("请选择编号:");
                Scanner sc = new Scanner(System.in);
                int choose = sc.nextInt();

                switch (choose) {
                    case 1 -> {
                        login();
                    }
                    case 2 -> {
                        register();
                    }
                    case 3 -> {
                        System.out.println("退卡操作");
                        return; //结束方法
                    }
                }
            }
        }
    }
    /**
     * 作用: 根据卡号和密码登录
     */
    public void login() throws IOException {
        //接受用户输入的卡号和密码
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入卡号");
        String cardId = sc.next();
        System.out.println("请输入密码");
        String password = sc.next();
        //在accounts集合中查询 和输入的卡号和密码一致的账户
        Account login = null;
        for (int i = 0; i < accounts.size(); i++) {
            Account account = accounts.get(i);
            if (account.getCardId().equals(cardId) && account.getPassword().equals(password)) {
                //登录成功
                login = account;
                break;
            }
        }
        //判断login是否为null;
        if (login == null) {
            System.out.println("用户名或密码错误");
        } else {
            System.out.println("登录成功");
            //将查询出来的账户放到成员区域, 方便其他功能找到数据
            this.currentAccount = login;
            //展示二级菜单
            showSecondMenu();
        }
    }

    /**
     * 作用: 展示二级菜单
     */
    public void showSecondMenu() throws IOException {
        while (true) {
            System.out.println("==================欢迎" + currentAccount.getName() + currentAccount.getNick() + "进入ATM系统==================");
            System.out.println("1.查询账户信息");
            System.out.println("2.取款");
            System.out.println("3.存款");
            System.out.println("4.转账");
            System.out.println("5.修改密码");
            System.out.println("6.注销");
            System.out.println("7.退出");
            System.out.println("请输入你的操作");
            Scanner scanner = new Scanner(System.in);
            int choose = scanner.nextInt();
            switch (choose) {
                case 1 -> {
                    showAccountInfo();
                }
                case 2 -> {
                    getMoney();
                }
                case 3 -> {
                    saveMoney();
                }
                case 4 -> {
                    System.out.println("转账");
                }
                case 5 -> {
                    System.out.println("修改密码");
                case 6 -> {
                }
                case 7 -> {
                    System.out.println("请收好您的卡片");
                    return;
                }
            }
        }
    }
    // 取款功能
    public void getMoney() throws IOException {
        System.out.println("请输入取款金额：");
        Scanner sc = new Scanner(System.in);
        int money = sc.nextInt();
        if (money % 100 != 0) {
            System.out.println("atm不能取零钱");
            return;
        }
        //超过你的总钱数
        if (currentAccount.getMoney() < money) {
            System.out.println("余额不足,你自己几斤几两不清楚吗???");
            return;
        }
        //超过限额不能取
        if (money > currentAccount.getLimit()) {
            System.out.println("超过了限额");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() - money);
        System.out.println("您本次取款" + money + ", 目前余额是:" + currentAccount.getMoney() + "元");

    }
    // 存款功能
    public void saveMoney() throws IOException {
        Scanner sc = new Scanner(System.in);
        int money = sc.nextInt();
        //判断money是否是整百
        if (money % 100 != 0) {
            System.out.println("atm不存零钱!!!");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() + money);
        System.out.println("存钱成功~~~");

    }
    /**
     * 作用: 展示当前账户的信息
     */
    public void showAccountInfo() {
        System.out.println("卡号是::" + currentAccount.getCardId());
        System.out.println("姓名是::" + currentAccount.getName());
        System.out.println("性别是::" + currentAccount.getGender());
        System.out.println("余额是::" + currentAccount.getMoney());
        System.out.println("手机号是::" + currentAccount.getPhone());
        System.out.println("昵称是::" + currentAccount.getNick());
        System.out.println("额度是::" + currentAccount.getLimit());

    }
    /**
     * 作用: 开户功能
     */
    public void register() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入姓名");
        String name = sc.next();
        System.out.println("请输入性别");
        String gender = sc.next();
        String password;
        while (true) {
            System.out.println("请设置密码");
            password = sc.next();
            System.out.println("请确认密码");
            String confirmPassword = sc.next();
            if (password.equals(confirmPassword)) {
                break;
            } else {
                System.out.println("两次密码不一致, 请重新设置");
            }
        }
        System.out.println("请输入限额");
        double limit = sc.nextDouble();
        System.out.println("请输入手机号");
        String phone = sc.next();
        // 生成卡号 --> 随机长度10的字符串,作为卡号
        String cardId = generateCardId();
        //将数据封装成账户对象
        Account account = new Account(cardId, password, name, gender, 0, limit, phone);
        //将账户保存到atm的集合(保存到数据库中)
        accounts.add(account);
        System.out.println("恭喜您注入成功,请收好您的卡片,您的卡号是:" + cardId);
    }
    /**
     * 作用: 生成卡号
     *
     * @return 返回生成的卡号
     */
    public String generateCardId() {
        // String str = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        String str = "";
        for (int i = 0; i < 10; i++) {
            str += i;
        }
        for (char c = 'A'; c <= 'Z'; c++) {
            str += c;
        }
        for (char c = 'a'; c <= 'z'; c++) {
            str += c;
        }
        String cardId = "";
        //循环10次, 生成一个卡号
        Random r = new Random();
        for (int i = 0; i < 10; i++) {
            int index = r.nextInt(str.length());
            //取出索引对应的字符
            char c = str.charAt(index);
            cardId += c;
        }
        return cardId;
    }
}
```

#### 转账功能实现

这里加入名字校验功能, 必须输入正确名字才能进行转账

```Java
public class ATM {
    //定义集合用于存储所有的账户信息
    private ArrayList<Account> accounts = new ArrayList<>();
    //用于记录登录成功的完整账户对象
    private Account currentAccount;
    // 在构建ATM的时候, 可以设置默认的账户, 提交后面功能测试效率
    public ATM() throws IOException {
        accounts.add(new Account("1234567890", "123456", "梅军建", "男", 10000, 5000, "13888888888"));
        accounts.add(new Account("1234567891", "123456", "林青霞", "女", 1000000, 500000, "13888888889"));

    }
    /**
     * 作用: 展示一级菜单
     */
    public void showMainMenu() throws IOException {
        // alt + ctrl + T
        while (true) {
                System.out.println("~~~~~~~欢迎来到黑马ATM~~~~~~~~");
                System.out.println("1.登录");
                System.out.println("2.注册");
                System.out.println("3.退出");
                System.out.println("请选择编号:");
                Scanner sc = new Scanner(System.in);
                int choose = sc.nextInt();

                switch (choose) {
                    case 1 -> {
                        login();
                    }
                    case 2 -> {
                        register();
                    }
                    case 3 -> {
                        System.out.println("退卡操作");
                        return; //结束方法
                    }
                }
            }
        }
    }
    /**
     * 作用: 根据卡号和密码登录
     */
    public void login() throws IOException {
        //接受用户输入的卡号和密码
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入卡号");
        String cardId = sc.next();
        System.out.println("请输入密码");
        String password = sc.next();
        //在accounts集合中查询 和输入的卡号和密码一致的账户
        Account login = null;
        for (int i = 0; i < accounts.size(); i++) {
            Account account = accounts.get(i);
            if (account.getCardId().equals(cardId) && account.getPassword().equals(password)) {
                //登录成功
                login = account;
                break;
            }
        }
        //判断login是否为null;
        if (login == null) {
            System.out.println("用户名或密码错误");
        } else {
            System.out.println("登录成功");
            //将查询出来的账户放到成员区域, 方便其他功能找到数据
            this.currentAccount = login;
            //展示二级菜单
            showSecondMenu();
        }
    }

    /**
     * 作用: 展示二级菜单
     */
    public void showSecondMenu() throws IOException {
        while (true) {
            System.out.println("==================欢迎" + currentAccount.getName() + currentAccount.getNick() + "进入ATM系统==================");
            System.out.println("1.查询账户信息");
            System.out.println("2.取款");
            System.out.println("3.存款");
            System.out.println("4.转账");
            System.out.println("5.修改密码");
            System.out.println("6.注销");
            System.out.println("7.退出");
            System.out.println("请输入你的操作");
            Scanner scanner = new Scanner(System.in);
            int choose = scanner.nextInt();
            switch (choose) {
                case 1 -> {
                    showAccountInfo();
                }
                case 2 -> {
                    getMoney();
                }
                case 3 -> {
                    saveMoney();
                }
                case 4 -> {
                    transferMoney();
                }
                case 5 -> {
                    System.out.println("修改密码");
                case 6 -> {
                }
                case 7 -> {
                    System.out.println("请收好您的卡片");
                    return;
                }
            }
        }
    }
    public void transferMoney() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入要转账的金额");
        double money = sc.nextDouble();
        if(money > currentAccount.getMoney()){
            System.out.println("余额不足");
            return;
        }
        if(money > currentAccount.getLimit()){
            System.out.println("超出限制金额");
            return;
        }
        System.out.println("请输入要转入的卡号");
        String cardId = sc.next();
        //通过卡号查询该账户
        Account target = findAccountByCardId(cardId);
        if(target == null){
            System.out.println("您要转账账户不存在");
            return;
        }
        String name = target.getName();
        Random r = new Random();
        int index = r.nextInt(name.length());
        String newName = name.substring(0,index) + "*" + name.substring(index+1);

        System.out.println("请输入"+newName+"的*号对应的字符");
        String inputName = sc.next();

        if (!(name.charAt(index)+"").equals(inputName) ){
            System.out.println("输入的名字不对,为了您的安全, 转账失败~~~");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() - money);
        target.setMoney(target.getMoney() + money);
        System.out.println("转账成功,您目前的余额是:"+ currentAccount.getMoney());
        saveData();

    }
    public Account findAccountByCardId(String cardId){
        Account account = null;
        for (int i = 0; i < accounts.size(); i++) {
            Account account1 = accounts.get(i);
            if (account1.getCardId().equals(cardId)){
                account = account1;
                break;
            }
        }
        return account;

    }

    // 取款功能
    public void getMoney() throws IOException {
        System.out.println("请输入取款金额：");
        Scanner sc = new Scanner(System.in);
        int money = sc.nextInt();
        if (money % 100 != 0) {
            System.out.println("atm不能取零钱");
            return;
        }
        //超过你的总钱数
        if (currentAccount.getMoney() < money) {
            System.out.println("余额不足,你自己几斤几两不清楚吗???");
            return;
        }
        //超过限额不能取
        if (money > currentAccount.getLimit()) {
            System.out.println("超过了限额");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() - money);
        System.out.println("您本次取款" + money + ", 目前余额是:" + currentAccount.getMoney() + "元");

    }
    // 存款功能
    public void saveMoney() throws IOException {
        Scanner sc = new Scanner(System.in);
        int money = sc.nextInt();
        //判断money是否是整百
        if (money % 100 != 0) {
            System.out.println("atm不存零钱!!!");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() + money);
        System.out.println("存钱成功~~~");

    }
    /**
     * 作用: 展示当前账户的信息
     */
    public void showAccountInfo() {
        System.out.println("卡号是::" + currentAccount.getCardId());
        System.out.println("姓名是::" + currentAccount.getName());
        System.out.println("性别是::" + currentAccount.getGender());
        System.out.println("余额是::" + currentAccount.getMoney());
        System.out.println("手机号是::" + currentAccount.getPhone());
        System.out.println("昵称是::" + currentAccount.getNick());
        System.out.println("额度是::" + currentAccount.getLimit());

    }
    /**
     * 作用: 开户功能
     */
    public void register() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入姓名");
        String name = sc.next();
        System.out.println("请输入性别");
        String gender = sc.next();
        String password;
        while (true) {
            System.out.println("请设置密码");
            password = sc.next();
            System.out.println("请确认密码");
            String confirmPassword = sc.next();
            if (password.equals(confirmPassword)) {
                break;
            } else {
                System.out.println("两次密码不一致, 请重新设置");
            }
        }
        System.out.println("请输入限额");
        double limit = sc.nextDouble();
        System.out.println("请输入手机号");
        String phone = sc.next();
        // 生成卡号 --> 随机长度10的字符串,作为卡号
        String cardId = generateCardId();
        //将数据封装成账户对象
        Account account = new Account(cardId, password, name, gender, 0, limit, phone);
        //将账户保存到atm的集合(保存到数据库中)
        accounts.add(account);
        System.out.println("恭喜您注入成功,请收好您的卡片,您的卡号是:" + cardId);
    }
    /**
     * 作用: 生成卡号
     *
     * @return 返回生成的卡号
     */
    public String generateCardId() {
        // String str = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        String str = "";
        for (int i = 0; i < 10; i++) {
            str += i;
        }
        for (char c = 'A'; c <= 'Z'; c++) {
            str += c;
        }
        for (char c = 'a'; c <= 'z'; c++) {
            str += c;
        }
        String cardId = "";
        //循环10次, 生成一个卡号
        Random r = new Random();
        for (int i = 0; i < 10; i++) {
            int index = r.nextInt(str.length());
            //取出索引对应的字符
            char c = str.charAt(index);
            cardId += c;
        }
        return cardId;
    }
}
```

#### 修改密码

```Java
public class ATM {
    //定义集合用于存储所有的账户信息
    private ArrayList<Account> accounts = new ArrayList<>();
    //用于记录登录成功的完整账户对象
    private Account currentAccount;
    // 在构建ATM的时候, 可以设置默认的账户, 提交后面功能测试效率
    public ATM() throws IOException {
        accounts.add(new Account("1234567890", "123456", "梅军建", "男", 10000, 5000, "13888888888"));
        accounts.add(new Account("1234567891", "123456", "林青霞", "女", 1000000, 500000, "13888888889"));

    }
    /**
     * 作用: 展示一级菜单
     */
    public void showMainMenu() throws IOException {
        // alt + ctrl + T
        while (true) {
                System.out.println("~~~~~~~欢迎来到黑马ATM~~~~~~~~");
                System.out.println("1.登录");
                System.out.println("2.注册");
                System.out.println("3.退出");
                System.out.println("请选择编号:");
                Scanner sc = new Scanner(System.in);
                int choose = sc.nextInt();

                switch (choose) {
                    case 1 -> {
                        login();
                    }
                    case 2 -> {
                        register();
                    }
                    case 3 -> {
                        System.out.println("退卡操作");
                        return; //结束方法
                    }
                }
            }
        }
    }
    /**
     * 作用: 根据卡号和密码登录
     */
    public void login() throws IOException {
        //接受用户输入的卡号和密码
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入卡号");
        String cardId = sc.next();
        System.out.println("请输入密码");
        String password = sc.next();
        //在accounts集合中查询 和输入的卡号和密码一致的账户
        Account login = null;
        for (int i = 0; i < accounts.size(); i++) {
            Account account = accounts.get(i);
            if (account.getCardId().equals(cardId) && account.getPassword().equals(password)) {
                //登录成功
                login = account;
                break;
            }
        }
        //判断login是否为null;
        if (login == null) {
            System.out.println("用户名或密码错误");
        } else {
            System.out.println("登录成功");
            //将查询出来的账户放到成员区域, 方便其他功能找到数据
            this.currentAccount = login;
            //展示二级菜单
            showSecondMenu();
        }
    }

    /**
     * 作用: 展示二级菜单
     */
    public void showSecondMenu() throws IOException {
        while (true) {
            System.out.println("==================欢迎" + currentAccount.getName() + currentAccount.getNick() + "进入ATM系统==================");
            System.out.println("1.查询账户信息");
            System.out.println("2.取款");
            System.out.println("3.存款");
            System.out.println("4.转账");
            System.out.println("5.修改密码");
            System.out.println("6.注销");
            System.out.println("7.退出");
            System.out.println("请输入你的操作");
            Scanner scanner = new Scanner(System.in);
            int choose = scanner.nextInt();
            switch (choose) {
                case 1 -> {
                    showAccountInfo();
                }
                case 2 -> {
                    getMoney();
                }
                case 3 -> {
                    saveMoney();
                }
                case 4 -> {
                    transferMoney();
                }
                case 5 -> {
                   //难点: 这类需要根据修改的成功与否 决定是否退出二级菜单
                   if(updatePassword()) return;//成功(退出二级菜单) return: 失败(保留二级菜单)
                case 6 -> {
                }
                case 7 -> {
                    System.out.println("请收好您的卡片");
                    return;
                }
            }
        }
    }
    public  boolean updatePassword() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入原密码");
        String password = sc.next();
        if(!currentAccount.getPassword().equals(password)){
            System.out.println("原密码输入错误");
            return false;
        }
        System.out.println("请输入新密码");
        String newPassword = sc.next();
        System.out.println("请确认密码");
        String confirmPassword = sc.next();
        if(!newPassword.equals(confirmPassword)){
            System.out.println("两次密码输入不一致,修改失败");
            return false;
        }
        //走到这里应该
        currentAccount.setPassword(newPassword);
        //退出登录,重新登录
        System.out.println("修改密码成功, 请重新登录~~~");
        saveData();
        return true;
    }

    public void transferMoney() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入要转账的金额");
        double money = sc.nextDouble();
        if(money > currentAccount.getMoney()){
            System.out.println("余额不足");
            return;
        }
        if(money > currentAccount.getLimit()){
            System.out.println("超出限制金额");
            return;
        }
        System.out.println("请输入要转入的卡号");
        String cardId = sc.next();
        //通过卡号查询该账户
        Account target = findAccountByCardId(cardId);
        if(target == null){
            System.out.println("您要转账账户不存在");
            return;
        }
        String name = target.getName();
        Random r = new Random();
        int index = r.nextInt(name.length());
        String newName = name.substring(0,index) + "*" + name.substring(index+1);

        System.out.println("请输入"+newName+"的*号对应的字符");
        String inputName = sc.next();

        if (!(name.charAt(index)+"").equals(inputName) ){
            System.out.println("输入的名字不对,为了您的安全, 转账失败~~~");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() - money);
        target.setMoney(target.getMoney() + money);
        System.out.println("转账成功,您目前的余额是:"+ currentAccount.getMoney());
        saveData();

    }
    public Account findAccountByCardId(String cardId){
        Account account = null;
        for (int i = 0; i < accounts.size(); i++) {
            Account account1 = accounts.get(i);
            if (account1.getCardId().equals(cardId)){
                account = account1;
                break;
            }
        }
        return account;

    }

    // 取款功能
    public void getMoney() throws IOException {
        System.out.println("请输入取款金额：");
        Scanner sc = new Scanner(System.in);
        int money = sc.nextInt();
        if (money % 100 != 0) {
            System.out.println("atm不能取零钱");
            return;
        }
        //超过你的总钱数
        if (currentAccount.getMoney() < money) {
            System.out.println("余额不足,你自己几斤几两不清楚吗???");
            return;
        }
        //超过限额不能取
        if (money > currentAccount.getLimit()) {
            System.out.println("超过了限额");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() - money);
        System.out.println("您本次取款" + money + ", 目前余额是:" + currentAccount.getMoney() + "元");

    }
    // 存款功能
    public void saveMoney() throws IOException {
        Scanner sc = new Scanner(System.in);
        int money = sc.nextInt();
        //判断money是否是整百
        if (money % 100 != 0) {
            System.out.println("atm不存零钱!!!");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() + money);
        System.out.println("存钱成功~~~");

    }
    /**
     * 作用: 展示当前账户的信息
     */
    public void showAccountInfo() {
        System.out.println("卡号是::" + currentAccount.getCardId());
        System.out.println("姓名是::" + currentAccount.getName());
        System.out.println("性别是::" + currentAccount.getGender());
        System.out.println("余额是::" + currentAccount.getMoney());
        System.out.println("手机号是::" + currentAccount.getPhone());
        System.out.println("昵称是::" + currentAccount.getNick());
        System.out.println("额度是::" + currentAccount.getLimit());

    }
    /**
     * 作用: 开户功能
     */
    public void register() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入姓名");
        String name = sc.next();
        System.out.println("请输入性别");
        String gender = sc.next();
        String password;
        while (true) {
            System.out.println("请设置密码");
            password = sc.next();
            System.out.println("请确认密码");
            String confirmPassword = sc.next();
            if (password.equals(confirmPassword)) {
                break;
            } else {
                System.out.println("两次密码不一致, 请重新设置");
            }
        }
        System.out.println("请输入限额");
        double limit = sc.nextDouble();
        System.out.println("请输入手机号");
        String phone = sc.next();
        // 生成卡号 --> 随机长度10的字符串,作为卡号
        String cardId = generateCardId();
        //将数据封装成账户对象
        Account account = new Account(cardId, password, name, gender, 0, limit, phone);
        //将账户保存到atm的集合(保存到数据库中)
        accounts.add(account);
        System.out.println("恭喜您注入成功,请收好您的卡片,您的卡号是:" + cardId);
    }
    /**
     * 作用: 生成卡号
     *
     * @return 返回生成的卡号
     */
    public String generateCardId() {
        // String str = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        String str = "";
        for (int i = 0; i < 10; i++) {
            str += i;
        }
        for (char c = 'A'; c <= 'Z'; c++) {
            str += c;
        }
        for (char c = 'a'; c <= 'z'; c++) {
            str += c;
        }
        String cardId = "";
        //循环10次, 生成一个卡号
        Random r = new Random();
        for (int i = 0; i < 10; i++) {
            int index = r.nextInt(str.length());
            //取出索引对应的字符
            char c = str.charAt(index);
            cardId += c;
        }
        return cardId;
    }
}
```

#### 2.3.8. 销户功能实现

```Java
public class ATM {
    //定义集合用于存储所有的账户信息
    private ArrayList<Account> accounts = new ArrayList<>();
    //用于记录登录成功的完整账户对象
    private Account currentAccount;
    // 在构建ATM的时候, 可以设置默认的账户, 提交后面功能测试效率
    public ATM() throws IOException {
        accounts.add(new Account("1234567890", "123456", "梅军建", "男", 10000, 5000, "13888888888"));
        accounts.add(new Account("1234567891", "123456", "林青霞", "女", 1000000, 500000, "13888888889"));

    }
    /**
     * 作用: 展示一级菜单
     */
    public void showMainMenu() throws IOException {
        // alt + ctrl + T
        while (true) {
                System.out.println("~~~~~~~欢迎来到黑马ATM~~~~~~~~");
                System.out.println("1.登录");
                System.out.println("2.注册");
                System.out.println("3.退出");
                System.out.println("请选择编号:");
                Scanner sc = new Scanner(System.in);
                int choose = sc.nextInt();

                switch (choose) {
                    case 1 -> {
                        login();
                    }
                    case 2 -> {
                        register();
                    }
                    case 3 -> {
                        System.out.println("退卡操作");
                        return; //结束方法
                    }
                }
            }
        }
    }
    /**
     * 作用: 根据卡号和密码登录
     */
    public void login() throws IOException {
        //接受用户输入的卡号和密码
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入卡号");
        String cardId = sc.next();
        System.out.println("请输入密码");
        String password = sc.next();
        //在accounts集合中查询 和输入的卡号和密码一致的账户
        Account login = null;
        for (int i = 0; i < accounts.size(); i++) {
            Account account = accounts.get(i);
            if (account.getCardId().equals(cardId) && account.getPassword().equals(password)) {
                //登录成功
                login = account;
                break;
            }
        }
        //判断login是否为null;
        if (login == null) {
            System.out.println("用户名或密码错误");
        } else {
            System.out.println("登录成功");
            //将查询出来的账户放到成员区域, 方便其他功能找到数据
            this.currentAccount = login;
            //展示二级菜单
            showSecondMenu();
        }
    }

    /**
     * 作用: 展示二级菜单
     */
    public void showSecondMenu() throws IOException {
        while (true) {
            System.out.println("==================欢迎" + currentAccount.getName() + currentAccount.getNick() + "进入ATM系统==================");
            System.out.println("1.查询账户信息");
            System.out.println("2.取款");
            System.out.println("3.存款");
            System.out.println("4.转账");
            System.out.println("5.修改密码");
            System.out.println("6.注销");
            System.out.println("7.退出");
            System.out.println("请输入你的操作");
            Scanner scanner = new Scanner(System.in);
            int choose = scanner.nextInt();
            switch (choose) {
                case 1 -> {
                    showAccountInfo();
                }
                case 2 -> {
                    getMoney();
                }
                case 3 -> {
                    saveMoney();
                }
                case 4 -> {
                    transferMoney();
                }
                case 5 -> {
                    //难点: 这类需要根据修改的成功与否 决定是否退出二级菜单
                    if(updatePassword()) return;//成功(退出二级菜单) return: 失败(保留二级菜单)
                case 6 -> {
                    if (logout()) return;//成功(退出二级菜单) return: 失败(保留二级菜单)
                }
                case 7 -> {
                    System.out.println("请收好您的卡片");
                    return;
                }
            }
        }
    }

    public boolean logout() {
        if (currentAccount.getMoney() > 0) {
            System.out.println("您还有未取款金额,请取完款后再退出");
            return false;
        }
        accounts.remove(currentAccount);
        System.out.println("注销成功,请取走您的银行卡");
        return true;
    }

    public  boolean updatePassword() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入原密码");
        String password = sc.next();
        if(!currentAccount.getPassword().equals(password)){
            System.out.println("原密码输入错误");
            return false;
        }
        System.out.println("请输入新密码");
        String newPassword = sc.next();
        System.out.println("请确认密码");
        String confirmPassword = sc.next();
        if(!newPassword.equals(confirmPassword)){
            System.out.println("两次密码输入不一致,修改失败");
            return false;
        }
        //走到这里应该
        currentAccount.setPassword(newPassword);
        //退出登录,重新登录
        System.out.println("修改密码成功, 请重新登录~~~");
        saveData();
        return true;
    }

    public void transferMoney() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入要转账的金额");
        double money = sc.nextDouble();
        if(money > currentAccount.getMoney()){
            System.out.println("余额不足");
            return;
        }
        if(money > currentAccount.getLimit()){
            System.out.println("超出限制金额");
            return;
        }
        System.out.println("请输入要转入的卡号");
        String cardId = sc.next();
        //通过卡号查询该账户
        Account target = findAccountByCardId(cardId);
        if(target == null){
            System.out.println("您要转账账户不存在");
            return;
        }
        String name = target.getName();
        Random r = new Random();
        int index = r.nextInt(name.length());
        String newName = name.substring(0,index) + "*" + name.substring(index+1);

        System.out.println("请输入"+newName+"的*号对应的字符");
        String inputName = sc.next();

        if (!(name.charAt(index)+"").equals(inputName) ){
            System.out.println("输入的名字不对,为了您的安全, 转账失败~~~");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() - money);
        target.setMoney(target.getMoney() + money);
        System.out.println("转账成功,您目前的余额是:"+ currentAccount.getMoney());
        saveData();

    }
    public Account findAccountByCardId(String cardId){
        Account account = null;
        for (int i = 0; i < accounts.size(); i++) {
            Account account1 = accounts.get(i);
            if (account1.getCardId().equals(cardId)){
                account = account1;
                break;
            }
        }
        return account;

    }

    // 取款功能
    public void getMoney() throws IOException {
        System.out.println("请输入取款金额：");
        Scanner sc = new Scanner(System.in);
        int money = sc.nextInt();
        if (money % 100 != 0) {
            System.out.println("atm不能取零钱");
            return;
        }
        //超过你的总钱数
        if (currentAccount.getMoney() < money) {
            System.out.println("余额不足,你自己几斤几两不清楚吗???");
            return;
        }
        //超过限额不能取
        if (money > currentAccount.getLimit()) {
            System.out.println("超过了限额");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() - money);
        System.out.println("您本次取款" + money + ", 目前余额是:" + currentAccount.getMoney() + "元");

    }
    // 存款功能
    public void saveMoney() throws IOException {
        Scanner sc = new Scanner(System.in);
        int money = sc.nextInt();
        //判断money是否是整百
        if (money % 100 != 0) {
            System.out.println("atm不存零钱!!!");
            return;
        }
        currentAccount.setMoney(currentAccount.getMoney() + money);
        System.out.println("存钱成功~~~");

    }
    /**
     * 作用: 展示当前账户的信息
     */
    public void showAccountInfo() {
        System.out.println("卡号是::" + currentAccount.getCardId());
        System.out.println("姓名是::" + currentAccount.getName());
        System.out.println("性别是::" + currentAccount.getGender());
        System.out.println("余额是::" + currentAccount.getMoney());
        System.out.println("手机号是::" + currentAccount.getPhone());
        System.out.println("昵称是::" + currentAccount.getNick());
        System.out.println("额度是::" + currentAccount.getLimit());

    }
    /**
     * 作用: 开户功能
     */
    public void register() throws IOException {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入姓名");
        String name = sc.next();
        System.out.println("请输入性别");
        String gender = sc.next();
        String password;
        while (true) {
            System.out.println("请设置密码");
            password = sc.next();
            System.out.println("请确认密码");
            String confirmPassword = sc.next();
            if (password.equals(confirmPassword)) {
                break;
            } else {
                System.out.println("两次密码不一致, 请重新设置");
            }
        }
        System.out.println("请输入限额");
        double limit = sc.nextDouble();
        System.out.println("请输入手机号");
        String phone = sc.next();
        // 生成卡号 --> 随机长度10的字符串,作为卡号
        String cardId = generateCardId();
        //将数据封装成账户对象
        Account account = new Account(cardId, password, name, gender, 0, limit, phone);
        //将账户保存到atm的集合(保存到数据库中)
        accounts.add(account);
        System.out.println("恭喜您注入成功,请收好您的卡片,您的卡号是:" + cardId);
    }
    /**
     * 作用: 生成卡号
     *
     * @return 返回生成的卡号
     */
    public String generateCardId() {
        // String str = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        String str = "";
        for (int i = 0; i < 10; i++) {
            str += i;
        }
        for (char c = 'A'; c <= 'Z'; c++) {
            str += c;
        }
        for (char c = 'a'; c <= 'z'; c++) {
            str += c;
        }
        String cardId = "";
        //循环10次, 生成一个卡号
        Random r = new Random();
        for (int i = 0; i < 10; i++) {
            int index = r.nextInt(str.length());
            //取出索引对应的字符
            char c = str.charAt(index);
            cardId += c;
        }
        return cardId;
    }
}
```

---

## 自测问答

> 覆盖本章考点与常见易错点；答不上来，回到上文对应小节重读。

### 问答：集合与 ArrayList 基础

**题目: 请简述集合与数组的区别。**

**目标: 掌握集合与数组的本质差异及各自适用场景**

```Java
1. 长度：集合长度可变，可随意增删元素；数组长度在创建时固定，不可改变。
2. 存储类型：集合只能存储引用数据类型（基本类型要靠包装类）；数组既可以存基本类型，也可以存引用类型。
3. 功能：集合是类，自带 add/get/remove/size 等大量操作方法；数组只有 length，其余要靠自己写循环。

结论：元素个数不确定、需要频繁增删 -> 用集合；个数固定、追求性能 -> 用数组。
```

**题目: 请书写创建 ArrayList 集合并添加元素的代码，并说明泛型的作用。**

**目标: 掌握 ArrayList 的构造、add 方法以及泛型限定作用**

```Java
ArrayList<String> list = new ArrayList<>();
list.add("张三");
list.add("李四");

泛型 <E> 的作用：限定集合存储的数据类型。创建时确定类型后，再添加其他类型会直接编译报错，
从而保证集合内数据类型统一，方便后续遍历和批量处理。
注意：
1. 泛型只能写引用数据类型，ArrayList<int> 是错误写法，应写 ArrayList<Integer>；
2. 使用泛型类时必须确定类型，否则 JVM 默认按 Object 处理，集合内可混装任意引用类型。
```

**题目: 集合不能直接存基本数据类型，那如何往 ArrayList 中存 int？请写出代码并说明发生了什么。**

**目标: 掌握包装类、装箱与拆箱机制**

```Java
ArrayList<Integer> list = new ArrayList<>();
list.add(1);              // 自动装箱：int -> Integer
int sum = 0;
for (int i = 0; i < list.size(); i++) {
    sum += list.get(i);   // 自动拆箱：Integer -> int 后才能参与运算
}
System.out.println(sum);

包装类对应关系：byte-Byte、short-Short、int-Integer、long-Long、
float-Float、double-Double、char-Character、boolean-Boolean。
装箱：基本类型 -> 包装类型；拆箱：包装类型 -> 基本类型；
JDK5 起自动完成，可把包装类当基本类型使用。
```

**题目: 基本数据类型和包装类有什么区别？**

**目标: 掌握包装类相对基本类型的两个特性**

```Java
1. 基本类型不能存储 null，包装类可以接收 null 值；
2. 基本类型没有任何功能，包装类有成员变量和方法（如 Integer.MAX_VALUE、Integer.parseInt）。
```

### 问答：ArrayList 常用功能

**题目: 请说明 add(E e) 和 add(int index, E e) 的区别。**

**目标: 掌握两种添加方式的语义与索引顺延规律**

```Java
boolean add(E e)：把元素添加到集合末尾，ArrayList 恒返回 true。
void add(int index, E e)：在指定索引处插入元素，原索引及之后的元素统一往后顺延一位。
注意插入位置必须满足 0 <= index <= size()，否则抛 IndexOutOfBoundsException。
```

**题目: remove(int index) 和 remove(Object obj) 有什么区别？删除元素时有什么坑？**

**目标: 掌握两种删除方式，理解 equals 比较与重载歧义**

```Java
E remove(int index)：按索引删除，返回被删除的元素，其后元素往前顺延。
boolean remove(Object obj)：按内容删除，底层用 equals 逐个比较，删除第一个匹配的元素；
    元素必须重写 equals，否则可能删不掉或删错。
坑：集合元素为 Integer 时，remove(1) 走的是按索引删除（删下标 1），
    想按值删除 1 必须写成 remove(Integer.valueOf(1))。
```

**题目: 请说明 get、set、size、contains、indexOf 的作用与返回值。**

**目标: 掌握 ArrayList 查询类 API 的语义**

```Java
E get(int index)             根据索引取元素
E set(int index, E newEle)   修改指定索引的元素，返回被修改的旧元素
int size()                   获取元素个数（不是 length）
boolean contains(Object obj) 判断是否包含某元素，底层用 equals 比较
int indexOf(Object obj)      返回元素首次出现的索引，不存在返回 -1，底层同样是 equals
```

**题目: 请用 ArrayList 存储 3 个姓名，遍历打印，并删除其中指定的一个姓名后再次打印。**

**目标: 掌握 size() + get() 配合遍历集合的标准写法**

```Java
ArrayList<String> list = new ArrayList<>();
list.add("张三");
list.add("李四");
list.add("王五");

for (int i = 0; i < list.size(); i++) {   // 条件用 size()，不能用 length
    System.out.println(list.get(i));
}

list.remove("李四");                        // 按内容删除
System.out.println(list);                  // [张三, 王五]
```

### 问答：ATM 综合项目

**题目: ATM 项目为什么要设计 Account 类和 ATM 类？各自职责是什么？**

**目标: 掌握面向对象拆分职责的设计思路**

```Java
Account（账户类）：一个账户就是一个对象，用成员变量封装卡号、密码、姓名、性别、余额、
    限额、手机号等信息，并提供 getNick() 这类根据性别返回尊称的行为。
ATM（管理类）：代表 ATM 系统本身，内部用 ArrayList<Account> 保存全部账户对象，
    负责开户、登录、存取款、转账、改密、销户、退出等业务流程。
App（启动类）：只保留 main 方法，创建 ATM 对象并调用 showMainMenu() 启动程序。
这样分工后，账户只关心自己的数据，系统只关心业务流程，符合单一职责。
```

**题目: ATM 系统的账户容器为什么选 ArrayList，而不是数组？**

**目标: 理解集合在业务系统中的选型依据**

```Java
开户数量事先不确定，数组长度固定、无法动态扩容，而 ArrayList 长度可变，
直接 add 新账户即可；同时 ArrayList 提供 get/size/remove 等现成方法，
配合遍历即可完成"按卡号查找账户""删除销户账户"等操作，代码量远小于数组。
```

**题目: 请描述 ATM 登录功能的业务流程，并指出其中用到了哪些已学知识。**

**目标: 综合运用集合遍历、String 比较、分支循环完成业务编码**

```Java
流程：
1. 键盘录入卡号、密码；
2. 遍历 accounts 集合，逐个取出 Account 对象；
3. 先用 getCardId() 与录入卡号做 equals 比较，找不到则提示"卡号不存在"；
4. 卡号匹配后再比较密码，不匹配提示"密码错误"；
5. 两者都匹配则登录成功，进入二级菜单（查询/存款/取款/转账/改密/销户/退出）。

用到的知识：
- ArrayList 遍历（size + get）与元素查找；
- String 的 equals 比较（不能用 ==）；
- if/switch 分支与 while 循环控制菜单流转；
- 面向对象：通过 getter 取账户数据、通过 setter 修改余额。
```

### 相关笔记

- [[Memo/development/programming/java/java-basics/09-arraylist-and-atm-project.md|ArrayList 与 ATM 综合实战]]——本章教程全文
- [[Memo/development/programming/java/collections-and-map/index.md|集合框架与 Map]]——后续 Collection 与 Map 体系

## 相关笔记

- [[Memo/development/programming/java/collections-and-map/03-collection.md|Collection 与 ArrayList]]
- [[Memo/development/programming/java/comprehensive-review.md|综合复习]]

---

> 来源：飞书《讲义》知识库 · 第 09 章 · 综合项目实战-ATM · [原文](https://baxsu7o100e.feishu.cn/wiki/VdUVw21h3iuqv9iWxi2pMRP8Tnq)
