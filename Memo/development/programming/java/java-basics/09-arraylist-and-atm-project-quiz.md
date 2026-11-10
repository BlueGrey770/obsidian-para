---
type: memo
date: 2026-11-09
aliases:
  - "09 ArrayList与ATM·问答"
---

# ArrayList 与 ATM 综合实战·问答

## 问答：集合与 ArrayList 基础

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

## 问答：ArrayList 常用功能

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

## 问答：ATM 综合项目

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

## 相关笔记

- [[Memo/development/programming/java/java-basics/09-arraylist-and-atm-project.md|ArrayList 与 ATM 综合实战]]——本章教程全文
- [[Memo/development/programming/java/collections-and-map/index.md|集合框架与 Map]]——后续 Collection 与 Map 体系
