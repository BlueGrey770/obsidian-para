---
type: memo
date: 2026-11-06
aliases:
  - "策略模式详解"
---

<title>策略模式详解</title>

# 策略模式（Strategy Pattern）：深度解析与实战落地

策略模式是**行为型设计模式**的核心代表，其本质是将“算法/业务逻辑的定义”与“算法/逻辑的使用”解耦，让不同的逻辑可以灵活替换，同时严格遵循面向对象的“抽象（设计思想）+ 封装/继承/多态（实现工具）”层级体系。本文从核心定义、设计思想、实现流程、实战场景、优化方案到框架落地，完整拆解策略模式的所有核心知识点。

## 一、策略模式的核心定义与设计初衷

### 1. 核心定义

策略模式定义了一系列**可替换的算法/业务逻辑（策略）**，将每个算法封装为独立的类，使它们可以互相替换；同时让算法的变化独立于使用算法的客户端，客户端只需关注“选择哪个策略”，无需关心“策略如何实现”。

### 2. 设计初衷（解决的核心问题）

- 替代大量嵌套的 `if-else/switch` 逻辑（如“支付方式判断”“排序算法选择”）；
- 让算法/逻辑可灵活扩展（新增逻辑无需修改原有代码，符合**开闭原则**）；
- 遵循“单一职责原则”：每个策略只负责实现一种逻辑，上下文只负责调用策略。

### 3. 核心角色（3个，缺一不可）

| 角色名称 | 核心职责 | 代码载体示例 |
|-|-|-|
| 抽象策略（Strategy） | 定义所有策略的统一行为契约（只声明“做什么”，不实现“怎么做”） | 接口（如 `PaymentStrategy`）、抽象类 |
| 具体策（ConcreteStrategy） | 实现抽象策略接口，封装具体的算法/业务逻辑（对外隐藏实现细节） | 实现类（如 `WeixinPayment`/`AlipayPayment`） |
| 上下文（Context） | 持有抽象策略的引用，接收客户端传入的具体策略，委托策略执行逻辑（不参与具体实现） | 业务类（如 `ShoppingCart`） |

## 二、策略模式与面向对象特性的层级关系

策略模式是面向对象思想的极致体现，需明确：**抽象是顶层设计思想，封装/继承/多态是底层实现工具**，二者不可混为一谈。

### 1. 抽象（设计思想层）：定义策略的“行为契约”

抽象是策略模式的“起点”，核心是从多个具体逻辑中提取**共性行为**，忽略细节、定义统一标准。

- 例如：从“微信支付、支付宝支付、信用卡支付”中，抽象出“支付”这个核心行为，定义 `PaymentStrategy` 接口的 `pay(BigDecimal amount)` 方法；
- 抽象的价值：让所有具体策略遵循同一套规则，客户端/上下文只需依赖抽象，而非具体实现（符合**依赖倒置原则**）。

### 2. 继承（实现工具层）：绑定抽象与具体

继承（广义上包含接口实现）是连接“抽象契约”与“具体实现”的桥梁：

- 具体策略类必须通过 `implements`（接口）/`extends`（抽象类）绑定抽象策略；
- 强制重写抽象方法，保证所有策略都实现核心行为（如 `WeixinPayment` 必须实现 `pay()` 方法）；
- 示例：`public class WeixinPayment implements PaymentStrategy`。

### 3. 封装（实现工具层）：隔离策略的实现细节

封装是策略模式“高内聚”的核心保障：

- **策略内部封装**：具体策略的特有属性（如微信ID、支付宝账号）私有化，具体逻辑（如SDK调用、参数校验）封装在 `pay()` 方法内，外部不可访问；
- **上下文封装**：通用逻辑（如金额校验、日志记录）封装在上下文（`ShoppingCart`），所有策略共用，避免重复代码；
- 示例：`ShoppingCart` 的 `checkout()` 方法中，先校验金额，再调用策略的 `pay()`，最后记录日志，这些逻辑对所有策略透明。

### 4. 多态（实现工具层）：实现策略的动态切换

多态是策略模式“灵活替换”的核心，也是抽象思想落地的最终效果：

- **父类型引用指向子类型对象**：`PaymentStrategy strategy = new WeixinPayment("wx123")`；
- **运行时动态绑定**：调用 `strategy.pay()` 时，JVM根据对象实际类型执行对应逻辑（微信支付/支付宝支付）；
- **上下文无感知**：`ShoppingCart` 只需调用 `paymentStrategy.pay()`，无需知道当前是哪种支付方式，切换策略只需替换对象。

## 三、策略模式的实现流程（以“支付场景”为例）

### 步骤1：定义抽象策略（接口）

抽象策略是所有具体策略的“行为契约”，只声明方法签名，不包含具体逻辑：

```Java
import java.math.BigDecimal;

// 抽象策略：支付策略接口（体现抽象思想）
public interface PaymentStrategy {
    /**
     * 统一的支付方法
     * @param amount 支付金额
     */
    void pay(BigDecimal amount);
}
```

### 步骤2：实现具体策略（封装不同支付逻辑）

每个具体策略类封装一种支付方式的核心逻辑，隐藏内部细节（如账号、SDK调用）：

```Java
// 具体策略1：微信支付
public class WeixinPayment implements PaymentStrategy {
    // 封装特有属性：微信ID（私有化，外部不可直接访问）
    private String wechatId;

    // 构造方法初始化属性
    public WeixinPayment(String wechatId) {
        this.wechatId = wechatId;
    }

    // 实现抽象策略的支付方法（封装微信支付具体逻辑）
    @Override
    public void pay(BigDecimal amount) {
        System.out.println("【微信支付】用户ID：" + wechatId + "，支付金额：" + amount + " 元");
        // 实际场景：调用微信支付SDK、签名验证、扣减金额等逻辑（均封装在此方法内）
    }
}

// 具体策略2：支付宝支付
public class AlipayPayment implements PaymentStrategy {
    private String alipayAccount;

    public AlipayPayment(String alipayAccount) {
        this.alipayAccount = alipayAccount;
    }

    @Override
    public void pay(BigDecimal amount) {
        System.out.println("【支付宝支付】账号：" + alipayAccount + "，支付金额：" + amount + " 元");
    }
}

// 具体策略3：信用卡支付
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;

    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }

    @Override
    public void pay(BigDecimal amount) {
        System.out.println("【信用卡支付】卡号：" + cardNumber + "，支付金额：" + amount + " 元");
    }
}
```

### 步骤3：设计上下文（解耦策略与客户端）

上下文是策略的“调用者”，核心职责是：

- 持有抽象策略的引用（依赖抽象，而非具体）；
- 对外提供统一的调用入口；
- 封装所有策略的通用逻辑（如金额校验、日志记录）。

```Java
// 上下文：购物车
public class ShoppingCart {
    // 持有抽象策略引用（多态的核心体现）
    private PaymentStrategy paymentStrategy;

    /**
     * 动态设置策略（支持运行时切换）
     * @param paymentStrategy 具体支付策略
     */
    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    /**
     * 统一结账方法（对外暴露的简单接口）
     * @param amount 订单金额
     */
    public void checkout(BigDecimal amount) {
        // 通用逻辑：金额校验（所有策略共用，封装在上下文）
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("支付金额必须大于0");
        }
        // 委托具体策略执行支付（多态：调用抽象方法，执行具体实现）
        paymentStrategy.pay(amount);
        // 通用逻辑：记录支付日志（所有策略共用）
        System.out.println("【通用逻辑】支付完成，订单日志已记录");
    }
}
```

### 步骤4：客户端调用（灵活选择策略）

客户端只需“选择策略”，无需关心策略的实现细节，切换策略只需替换对象：

```Java
// 客户端：模拟用户下单
public class StrategyPatternDemo {
    public static void main(String[] args) {
        // 1. 创建上下文（购物车）
        ShoppingCart cart = new ShoppingCart();
        BigDecimal orderAmount = new BigDecimal("299.99");

        // 2. 场景1：用户选择微信支付
        cart.setPaymentStrategy(new WeixinPayment("wx_888888"));
        cart.checkout(orderAmount);
        // 输出：
        // 【微信支付】用户ID：wx_888888，支付金额：299.99 元
        // 【通用逻辑】支付完成，订单日志已记录

        System.out.println("------------------------");

        // 3. 场景2：用户切换为支付宝支付（无需修改上下文代码）
        cart.setPaymentStrategy(new AlipayPayment("user@aliyun.com"));
        cart.checkout(orderAmount);

        System.out.println("------------------------");

        // 4. 场景3：新增信用卡支付（仅需新增策略类，符合开闭原则）
        cart.setPaymentStrategy(new CreditCardPayment("622848****9999"));
        cart.checkout(orderAmount);
    }
}
```

## 四、策略模式的优化：解决基础版痛点

基础版实现中，客户端需要直接创建具体策略对象（如 `new WeixinPayment(...)`），存在“客户端依赖具体策略、硬编码”的问题，需结合**工厂模式+枚举**优化。

### 1. 优化步骤1：定义策略枚举（统一标识）

避免硬编码字符串，用枚举管理所有策略类型：

```Java
// 支付策略枚举：统一策略标识
public enum PaymentType {
    WECHAT, ALIPAY, CREDIT_CARD
}
```

### 2. 优化步骤2：实现策略工厂（封装策略创建）

工厂类负责策略的创建与管理，客户端只需传入枚举即可获取策略，无需知道具体类：

```Java
import java.util.HashMap;
import java.util.Map;
import java.util.function.Supplier;

// 策略工厂：封装策略创建逻辑
public class PaymentStrategyFactory {
    // 缓存策略创建器（懒加载，避免提前初始化所有策略）
    private static final Map<PaymentType, Supplier<PaymentStrategy>> STRATEGY_MAP = new HashMap<>();

    // 静态初始化：注册策略（新增策略仅需修改此处）
    static {
        STRATEGY_MAP.put(PaymentType.WECHAT, () -> new WeixinPayment("wx_888888"));
        STRATEGY_MAP.put(PaymentType.ALIPAY, () -> new AlipayPayment("user@aliyun.com"));
        STRATEGY_MAP.put(PaymentType.CREDIT_CARD, () -> new CreditCardPayment("622848****9999"));
    }

    // 私有构造：禁止实例化
    private PaymentStrategyFactory() {}

    /**
     * 获取策略对象
     * @param type 策略类型
     * @return 具体策略
     */
    public static PaymentStrategy getStrategy(PaymentType type) {
        Supplier<PaymentStrategy> strategySupplier = STRATEGY_MAP.get(type);
        if (strategySupplier == null) {
            throw new IllegalArgumentException("不支持的支付方式：" + type);
        }
        return strategySupplier.get();
    }
}
```

### 3. 优化步骤3：客户端简化调用

```Java
// 优化后的客户端
public class OptimizedStrategyDemo {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        BigDecimal orderAmount = new BigDecimal("299.99");

        // 选择微信支付（只需传入枚举，无需new具体类）
        cart.setPaymentStrategy(PaymentStrategyFactory.getStrategy(PaymentType.WECHAT));
        cart.checkout(orderAmount);

        // 切换为支付宝支付（仅需修改枚举值）
        cart.setPaymentStrategy(PaymentStrategyFactory.getStrategy(PaymentType.ALIPAY));
        cart.checkout(orderAmount);
    }
}
```

## 五、Spring 环境下的策略模式实战

在Spring项目中，可利用容器特性进一步简化策略管理，无需手动维护工厂映射：

### 1. 步骤1：注册策略为Spring Bean

用 `@Component` 注解标记具体策略，指定Bean名称（与策略类型对应）：

```Java
import org.springframework.stereotype.Component;
import java.math.BigDecimal;

// 微信支付策略：注册为Spring Bean，名称为"WECHAT"
@Component("WECHAT")
public class WeixinPayment implements PaymentStrategy {
    @Override
    public void pay(BigDecimal amount) {
        System.out.println("【Spring-微信支付】支付金额：" + amount + " 元");
    }
}

// 支付宝支付策略：注册为Spring Bean，名称为"ALIPAY"
@Component("ALIPAY")
public class AlipayPayment implements PaymentStrategy {
    @Override
    public void pay(BigDecimal amount) {
        System.out.println("【Spring-支付宝支付】支付金额：" + amount + " 元");
    }
}
```

### 2. 步骤2：Spring策略工厂（自动注入所有策略）

Spring会自动将所有 `PaymentStrategy` 类型的Bean注入到Map中（key=Bean名称，value=Bean实例）：

```Java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import java.util.Map;

// Spring策略工厂
@Component
public class SpringPaymentStrategyFactory {
    // 自动注入所有PaymentStrategy类型的Bean
    private final Map<String, PaymentStrategy> strategyMap;

    // 构造方法注入（Spring自动装配）
    @Autowired
    public SpringPaymentStrategyFactory(Map<String, PaymentStrategy> strategyMap) {
        this.strategyMap = strategyMap;
    }

    // 获取策略
    public PaymentStrategy getStrategy(String type) {
        PaymentStrategy strategy = strategyMap.get(type);
        if (strategy == null) {
            throw new IllegalArgumentException("不支持的支付方式：" + type);
        }
        return strategy;
    }
}
```

### 3. 步骤3：业务层使用策略

```Java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.math.BigDecimal;

@Service
public class OrderService {
    @Autowired
    private SpringPaymentStrategyFactory strategyFactory;

    /**
     * 订单支付
     * @param paymentType 支付类型（WECHAT/ALIPAY）
     * @param amount 支付金额
     */
    public void payOrder(String paymentType, BigDecimal amount) {
        ShoppingCart cart = new ShoppingCart();
        // 从工厂获取策略
        PaymentStrategy strategy = strategyFactory.getStrategy(paymentType);
        cart.setPaymentStrategy(strategy);
        cart.checkout(amount);
    }
}
```

## 六、策略模式的适用场景与核心优势

### 1. 适用场景

- 存在多个逻辑完成同一类任务（如支付方式、排序算法、校验规则），且需要动态切换；
- 避免使用大量 `if-else/switch` 分支逻辑；
- 算法/逻辑需要独立扩展，且不希望影响调用方。

### 2. 核心优势

| 优势 | 具体体现 |
|-|-|
| 符合开闭原则 | 新增策略只需新增类+注册，无需修改上下文/客户端代码 |
| 消除分支逻辑 | 用“策略选择”替代 `if-else`，代码更简洁易维护 |
| 高内聚低耦合 | 策略逻辑封装在独立类中，上下文只依赖抽象，降低模块间耦合 |
| 策略可复用 | 同一策略可在不同上下文/场景中复用 |
| 逻辑可动态切换 | 运行时通过 `setStrategy()` 切换策略，无需重启程序 |

### 3. 注意事项

- 策略类过多时，会增加系统复杂度（可结合“简单工厂+枚举”管理）；
- 策略类尽量设计为**无状态**（避免成员变量共享），保证线程安全；
- 客户端必须知道所有策略类型（可通过配置/枚举优化）。

## 七、策略模式在框架中的典型应用

策略模式是主流框架的核心设计思想之一，典型场景：

1. **Spring 事务管理**：`PlatformTransactionManager` 是抽象策略，`DataSourceTransactionManager`（JDBC事务）、`JtaTransactionManager`（分布式事务）是具体策略，可根据持久化技术动态切换；
2. **Spring 资源加载**：`Resource` 是抽象策略，`ClassPathResource`（类路径资源）、`FileSystemResource`（文件资源）、`UrlResource`（网络资源）是具体策略，`ResourceLoader` 根据路径前缀选择策略；
3. **Spring MVC 视图解析**：`ViewResolver` 是抽象策略，`InternalResourceViewResolver`（JSP视图）、`FreeMarkerViewResolver`（FreeMarker视图）是具体策略。

## 八、总结

策略模式的核心是“分离选择与实现”：

1. **设计层面**：通过“抽象”思想定义统一的行为契约，指导后续实现；
2. **实现层面**：通过“继承”绑定抽象与具体、“封装”隔离策略细节、“多态”实现灵活切换；
3. **实战优化**：结合工厂模式/枚举/Spring容器，解决“客户端依赖具体策略”的问题；
4. **核心价值**：让算法/业务逻辑可灵活扩展、复用，同时符合面向对象的设计原则，是解决“多分支逻辑”的最优方案之一。

掌握策略模式，不仅能写出更优雅的代码，更能理解Spring等框架的设计思想，是从“初级开发者”到“中级开发者”的关键一步。

---
> 来源：飞书技术知识库（脱敏版） · 08 面试知识库 / 项目面试题 / 日常整理 / 策略模式详解 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/EtEEw4iOCiBJyPkgm2Jc80S8nih)
