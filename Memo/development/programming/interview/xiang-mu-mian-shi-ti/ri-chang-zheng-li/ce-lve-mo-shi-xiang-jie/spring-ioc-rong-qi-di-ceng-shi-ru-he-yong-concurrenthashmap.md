---
type: memo
date: 2026-11-06
aliases:
  - "Spring IoC容器底层是如何用ConcurrentHashMap存储Bean"
---

# Spring IoC容器底层是如何用ConcurrentHashMap存储Bean

**Spring IoC容器底层是如何用ConcurrentHashMap存储Bean**，以及和策略模式中使用ConcurrentHashMap有哪些关联？

### 一、Spring IoC容器底层存储Bean的核心原理

Spring IoC容器（核心是`DefaultListableBeanFactory`）的核心职责是管理Bean的定义、实例化和缓存，其底层正是通过`ConcurrentHashMap`来存储Bean相关数据，这也是保证容器在多线程环境下线程安全的关键。

#### 1. 核心存储结构（关键ConcurrentHashMap）

Spring源码中，`DefaultListableBeanFactory`定义了多个`ConcurrentHashMap`来存储不同维度的Bean数据：

```Java
// 核心1：存储Bean定义（BeanDefinition），key=beanName，value=BeanDefinition
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);

// 核心2：存储单例Bean实例（缓存），key=beanName，value=Bean实例
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

// 核心3：存储Bean名称与对应的类型映射，key=beanType，value=beanName列表
private final Map<Class<?>, String[]> allBeanNamesByType = new ConcurrentHashMap<>(64);

// 核心4：存储类型到Bean实例的映射（简化获取）
private final Map<Class<?>, Map<String, Object>> singletonObjectsByType = new ConcurrentHashMap<>(64);
```

#### 2. 为什么选择ConcurrentHashMap？

- **线程安全**：Spring容器本身是多线程环境（如Web应用中多个请求线程同时获取Bean），`ConcurrentHashMap`保证了Bean的存/取操作的原子性、可见性；
- **性能优势**：`ConcurrentHashMap`采用分段锁（JDK1.7）/CAS+Synchronized（JDK1.8），读操作无锁（volatile保证可见性），写操作只锁当前分段/节点，远优于`Hashtable`的全局锁；
- **初始化特性**：容器启动时（BeanDefinition加载、实例化）是写操作，运行时（获取Bean）是读操作，`ConcurrentHashMap`能完美适配这种“少写多读”的场景。

#### 3. 核心流程（以获取单例Bean为例）

```Plain Text
graph TD
    A[调用getBean(beanName)] --> B{检查singletonObjects（ConcurrentHashMap）};
    B -->|存在| C[直接返回Bean实例];
    B -->|不存在| D[加锁创建Bean实例];
    D --> E[实例化Bean并填充属性];
    E --> F[存入singletonObjects];
    F --> C;
```

- 第一步：优先从`singletonObjects`（ConcurrentHashMap）中读取，无锁；
- 第二步：若不存在则加锁创建（避免重复实例化），创建完成后存入`singletonObjects`；
- 整个过程既保证了多线程下的线程安全，又通过`ConcurrentHashMap`的特性保证了读性能。

### 二、Spring容器存储Bean vs 策略模式存储策略Bean

我们之前写的`OrderCancelStrategyManager`中用`ConcurrentHashMap`存储策略Bean，本质是**模仿Spring容器的设计思路**，两者的核心逻辑高度一致：

| 维度 | Spring IoC容器底层 | 策略模式中的StrategyManager |
|-|-|-|
| 存储结构 | ConcurrentHashMap<String, Object>（Bean实例） | ConcurrentHashMap<String, OrderCancelStrategy>（策略Bean） |
| 线程安全设计 | 少写多读：启动时写入，运行时只读 | 少写多读：@PostConstruct时写入，运行时只读 |
| 核心目的 | 保证多线程获取Bean的线程安全 | 保证多线程获取策略Bean的线程安全 |
| 性能优化 | 读操作无锁，写操作分段锁 | 读操作无锁，写操作分段锁 |
| 初始化时机 | 容器启动时（refresh()） | Bean初始化时（@PostConstruct） |

### 三、关键细节：Spring容器的ConcurrentHashMap扩展设计

Spring不仅用ConcurrentHashMap存储Bean，还基于它做了额外的线程安全保障，这也是我们在业务代码中可以借鉴的：

#### 1. 双重检查锁（DCL）+ ConcurrentHashMap

Spring获取单例Bean时的核心代码（简化版）：

```Java
// DefaultSingletonBeanRegistry.java
public Object getSingleton(String beanName) {
    // 第一步：无锁读取ConcurrentHashMap
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null) {
        // 第二步：加锁（避免重复创建）
        synchronized (this.singletonObjects) {
            // 第三步：再次检查（双重检查锁）
            singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                // 创建Bean实例
                singletonObject = createBean(beanName);
                // 第四步：存入ConcurrentHashMap
                this.singletonObjects.put(beanName, singletonObject);
            }
        }
    }
    return singletonObject;
}
```

- 借鉴点：我们的策略模式中，若需要动态添加策略（而非仅@PostConstruct初始化），可以用这种“DCL + ConcurrentHashMap”的方式，既保证线程安全，又提升性能。

#### 2. 不可变设计（Bean初始化后不修改）

Spring容器中，BeanDefinition和单例Bean实例一旦存入ConcurrentHashMap，运行时几乎不会修改（除非手动调用`refresh()`/`destroy()`），这和我们的策略模式中“初始化后只读取策略Bean”的场景完全一致——**不可变的数据结构 + ConcurrentHashMap = 极致的线程安全和性能**。

### 四、和我们业务代码的关联（策略模式+ConcurrentHashMap）

我们之前写的`OrderCancelStrategyManager`本质是一个“迷你版的Spring Bean管理器”：

1. **存储载体**：用ConcurrentHashMap存储策略Bean，和Spring底层保持一致；
2. **初始化逻辑**：@PostConstruct时加载所有策略Bean并存入Map，对应Spring容器启动时加载BeanDefinition并实例化；
3. **访问逻辑**：运行时通过key获取策略Bean，对应Spring的`getBean()`方法；
4. **线程安全**：依赖ConcurrentHashMap的特性，保证多线程下获取策略Bean的安全。

### 总结

1. **核心原理**：Spring IoC容器底层通过`ConcurrentHashMap`存储Bean定义、单例实例等核心数据，利用其“线程安全 + 高并发性能”的特性，适配容器多线程访问的场景；
2. **设计借鉴**：我们在业务中（如策略模式）使用ConcurrentHashMap存储Bean，本质是复用Spring的设计思路——**单例环境下，少写多读的哈希存储优先用ConcurrentHashMap**；
3. **关键特性**：ConcurrentHashMap的读操作无锁、写操作低粒度锁，完美适配“初始化时写入，运行时只读”的Bean管理场景，这也是Spring选择它的核心原因。

简单来说，Spring IoC容器是“大而全”的Bean管理器，我们的策略模式中的StrategyManager是“小而专”的Bean管理器，两者底层都选择ConcurrentHashMap，核心都是为了在多线程环境下保证Bean存储/访问的线程安全和性能。

---
> 来源：飞书技术知识库（脱敏版） · 08 面试知识库 / 项目面试题 / 日常整理 / 策略模式详解 / Spring IoC容器底层是如何用ConcurrentHashMap存储Bean · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/ESxwwZSBniEMu6kAgBJc0h22nPh)
