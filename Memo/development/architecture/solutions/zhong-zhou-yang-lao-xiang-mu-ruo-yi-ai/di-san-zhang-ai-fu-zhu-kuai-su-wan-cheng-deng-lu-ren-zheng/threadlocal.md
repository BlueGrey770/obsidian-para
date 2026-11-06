---
type: memo
date: 2026-11-06
aliases:
  - "ThreadLocal"
---

# ThreadLocal

## 概述

ThreadLocal是Java中的一个线程局部变量工具类，它提供了一种在多线程环境下，每个线程都可以独立访问自己的变量副本的机制。ThreadLocal中存储的数据对于每个线程来说都是独立的，互不干扰。

## 使用场景

ThreadLocal适用于以下场景：

- 在多线程环境下，需要保持线程安全性的数据访问。
- 需要在多个方法之间共享数据，但又不希望使用传递参数的方式。

## 基本用法

### 创建ThreadLocal对象

首先，我们需要创建一个ThreadLocal对象来存储线程局部变量。可以使用ThreadLocal的默认构造函数创建一个新的实例。

```Java
ThreadLocal<String> threadLocal = new ThreadLocal<>();
```

### 设置线程局部变量的值

使用`set()`方法可以设置当前线程的局部变量的值。

```Java
threadLocal.set("value");
```

### 获取线程局部变量的值

使用`get()`方法可以获取当前线程的局部变量的值。

```Java
String value = threadLocal.get();
```

### 清除线程局部变量的值

使用`remove()`方法可以清除当前线程的局部变量的值。

```Java
threadLocal.remove();
```

## 示例代码

下面是一个简单的示例代码，演示了如何使用ThreadLocal。

```Java
public class ThreadLocalTest {

    private static final ThreadLocal THREAD_LOCAL = new ThreadLocal();


    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            THREAD_LOCAL.set("itheima");
            getData("t1");
        }, "t1");

        Thread t2 = new Thread(() -> {
            THREAD_LOCAL.set("itcast");
            getData("t2");
        }, "t1");

        t1.start();
        t2.start();

    }

    private static void getData(String threadName){
        Object data = THREAD_LOCAL.get();
        System.out.println(threadName+"-"+data);
    }
}
```

运行以上代码，输出结果为：

```Plain Text
t1-itheima
t2-itcast
```

可以看到，每个线程都可以独立地访问自己的线程局部变量，互不干扰。



## 注意事项

- 需要注意ThreadLocal的内存泄漏问题。由于ThreadLocal的生命周期与线程的生命周期相同，如果没有手动清除线程局部变量的值，可能会导致内存泄漏。
- 避免过多使用ThreadLocal，过多的使用会导致代码的可读性变差，且容易引发线程安全问题。

## 总结

ThreadLocal是Java多线程编程中非常有用的工具类，它提供了一种简单的方式来实现线程局部变量的访问和管理。通过使用ThreadLocal，我们可以在多线程环境下保持数据的独立性，提高程序的并发性能。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第三章-AI辅助快速完成登录认证 / ThreadLocal · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/OZfGww6cWiP0Bkkfkf4cDhGtn0g)
