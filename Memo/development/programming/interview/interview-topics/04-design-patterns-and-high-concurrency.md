---
type: memo
date: 2026-11-06
aliases:
  - "04-chang-jing-she-ji-mo-shi-yu-gao-bing-fa"
  - "面试专题：设计模式与高并发场景题"
---

# 面试专题：设计模式与高并发场景题

## 一、概念说明

场景题考察的不是背诵，而是"把组件组装成系统"的能力。本篇分两部分：常用设计模式（含落地案例）与高并发场景（指标、架构、秒杀链路）。

### Q1：策略模式是什么？结构如何？

定义：定义一系列算法，将每个算法封装起来，使它们可以相互替换，且算法的变化不会影响使用算法的客户。它把使用算法的责任和算法的实现分割开来。

三个角色：抽象策略（Strategy，接口或抽象类，给出所有具体策略所需的接口）、具体策略（Concrete Strategy，实现接口提供具体算法）、环境（Context，持有策略引用，供客户端调用）。

促销案例：百货公司针对春节、中秋、圣诞推出不同促销活动，由销售员展示给客户。

```java
// 抽象策略：所有促销活动的共同接口
public interface Strategy {
    void show();
}

// 具体策略：各节日的促销活动
public class StrategyA implements Strategy {
    public void show() {
        System.out.println("买一送一");
    }
}

public class StrategyB implements Strategy {
    public void show() {
        System.out.println("满200元减50元");
    }
}

// 环境角色：销售员，持有策略引用
public class SalesMan {
    private Strategy strategy;

    public SalesMan(Strategy strategy) {
        this.strategy = strategy;
    }

    public void salesManShow() {
        strategy.show();
    }
}
```

### Q2：工厂 + 策略如何优化多方式登录？

需求：登录入口支持用户名密码、短信验证码、微信等方式，前端带 `type` 参数（0 用户名密码、1 短信、2 微信）请求后台。

朴素的业务层写法是一大串 if-else 按 type 分支，新增登录方式就要改业务代码。优化方案：每种登录方式一个策略实现类，工厂按 type 发牌：

```java
// 抽象策略
public interface UserGranter {
    LoginResp login(LoginReq loginReq);
}

// 具体策略：账号登录 / 短信登录 / 微信登录
@Component
public class AccountGranter implements UserGranter {
    @Override
    public LoginResp login(LoginReq loginReq) {
        // 执行业务操作
        return new LoginResp();
    }
}

// 策略工厂：把所有策略装进 Map，按 type 取出
@Component
public class UserLoginFactory {
    @Autowired private AccountGranter accountGranter;
    @Autowired private SmsGranter smsGranter;
    @Autowired private WeChatGranter weChatGranter;

    private static Map<String, UserGranter> granterPool = new ConcurrentHashMap<>();

    @PostConstruct
    public void init() {
        granterPool.put(UserLoginConstants.ACCOUNT_TYPE, accountGranter);
        granterPool.put(UserLoginConstants.PHONE_TYPE, smsGranter);
        granterPool.put(UserLoginConstants.WEB_CHAT_TYPE, weChatGranter);
    }

    // 对外提供获取具体策略
    public UserGranter getGranter(String grantType) {
        return granterPool.get(grantType);
    }
}

// 业务层：没有 if-else，只有发牌
@Service
public class UserService {
    @Autowired
    private UserLoginFactory factory;

    public LoginResp login(LoginReq loginReq) {
        UserGranter granter = factory.getGranter(loginReq.getType());
        if (granter == null) {
            LoginResp loginResp = new LoginResp();
            loginResp.setSuccess(false);
            return loginResp;
        }
        return granter.login(loginReq);
    }
}
```

改造后业务层非常清爽；新增登录方式只需要新增一个策略类并在工厂注册，符合开闭原则。举一反三：凡是"一个入口、多种处理规则"的场景（支付方式、消息推送渠道、审核方式）都能套这个结构。

### Q3：工厂方法、抽象工厂、构建者模式各解决什么问题？

- 简单工厂：一个工厂类根据参数决定创建哪种产品，缺点是新增产品要改工厂代码（违反开闭原则）；
- 工厂方法：把创建延迟到子类，每个产品配一个工厂，新增产品只需新增工厂类；
- 抽象工厂：面向"产品族"——一个工厂能生产一族相关产品（如不同主题风格的整套 UI 组件），适合产品之间存在约束的场景；
- 构建者模式：把复杂对象的构建过程与表示分离，同样的构建过程可以创建不同的表示，适合参数极多、需要分步装配的对象（如链式 setter）。

### Q4：什么是高并发？常用指标有哪些？

高并发是系统在短时间内遇到大量操作请求的情况，主要针对某个或某些接口（如 12306 抢票、双十一秒杀）。

常用指标：

- 响应时间：请求发出到收到响应消耗的时间；
- 吞吐量 = 并发数 / 平均响应时间；
- QPS：每秒处理的请求数；
- 并发数 = QPS × 平均响应时间；
- PV：页面浏览量，每次刷新计一次；UV：独立访客数。

峰值 QPS 估算（二八原则）：`(总PV × 80%) / (每天秒数 × 20%)`；机器数量 = 峰值 QPS / 单机 QPS。

### Q5：如何提升系统的并发能力？

- 垂直扩展：提升单机硬件性能、添加缓存；
- 水平扩展（重要）：增加机器数量、集群。

水平架构的层次：客户端层 → 负载均衡层（反向代理）→ 网关层（鉴权、认证、限流、路由）→ 服务层（集群）→ 数据缓存层 → 数据库层。实际业务中会有较大变化，组件按需取舍。

### Q6：秒杀系统的架构怎么走？

非热点商品链路：请求先到 Nginx，商品详情静态资源走 CDN → Nginx 中用 Lua 脚本做判断（JWT 令牌校验、该用户是否在指定时间内购买过、是否热点商品、是否有库存、是否正在排队）→ 非热点商品由 Nginx 反向代理到网关 → 网关路由到商品微服务扣减库存 → 商品微服务 Feign 调用订单微服务创建订单。注意扣库存 + 创建订单是原子操作，需要分布式事务解决。

热点商品链路：前面判断相同 → 点击热点商品后发消息给 Kafka 异步排队 → 热点下单服务订阅 Kafka 直接创建热点订单 → 扣减库存（热点数据在 Redis 中，扣 Redis 中的库存）→ 下单成功通知订单状态服务 → 订单状态服务通过 WebSocket 通知用户购买结果。注意扣库存要防超卖，需要分布式锁。

热点数据的产生：用户点击商品时 Nginx 记录操作、发日志给 Kafka → Apache Druid 实时存储 → 实时热点分析系统按指标（如页面点击量 1 分钟内超过 1000 次）识别热点 → 隔离锁定热点数据并同步到 Redis。

### Q7：负载均衡的作用与分类？

作用：高并发（均匀分配各节点工作量，提高集群吞吐量）、伸缩性（加减机器由负载均衡分发控制）、高可用（监控候选服务器，不可用自动跳过）、安全防护（黑白名单、防 DDoS）。

分类：

- 硬件负载均衡：定制处理器上运行的独立设备（F5、A10），功能强大、单机可支持百万级并发、安全性高，但成本昂贵、扩展性差；
- 软件负载均衡：应用最广泛，主流产品有 Nginx、HAProxy、LVS。LVS 可做四层负载均衡，性能优于 Nginx；HAProxy 可做 HTTP 和 TCP 负载均衡。

DDoS（分布式拒绝服务攻击）：处于不同位置的多个攻击者同时向一个或数个目标发动攻击，或一个攻击者控制多台机器同时攻击——攻击发出点分布在不同地方，难以封堵。

## 二、技术点之间的联系

- 设计模式解决"代码怎么组织"，高并发解决"系统怎么扛量"，场景题往往两者交织：秒杀链路中 Kafka 排队是削峰、Redis 库存是缓存加速、分布式锁防超卖、分布式事务保原子性——每个组件对应一个具体问题。
- 策略 + 工厂是"消除 if-else"的标准组合：策略负责封装算法，工厂负责按 key 路由；在 Spring 里用 `@Component` + `@PostConstruct` 注册表即可落地。
- 高并发指标之间可以互相换算（并发数 = QPS × 响应时间），估算机器数靠的是二八原则——面试中给出估算公式比报一个绝对数字更专业。
- 秒杀的两条链路（热点走异步排队 + Redis，非热点走同步下单 + 分布式事务）体现"按热度分级处理"的核心思想：把最多的流量挡在最前面（CDN → Nginx Lua → Kafka），数据库只做最终确认。

## 三、方法与实现要点

- 答设计模式题的结构：意图（解决什么问题）→ 角色（Strategy/Context 等）→ 一个贴近业务的案例 → 优缺点与替代方案；
- 答秒杀题时按请求流向逐层说：CDN 扛静态资源 → Nginx 脚本做前置校验 → 热点/非热点分流 → Kafka 削峰 → Redis 扣库存（分布式锁防超卖）→ 下单 → WebSocket 通知；
- 答高并发指标时先给定义再给换算公式，最后落到"峰值 QPS 怎么估算、机器数怎么定"；
- 负载均衡答题框架：作用四点 → 分类两类 → 每层选一个代表（LVS 四层、Nginx 七层）。

## 四、重点与难点分析

- **重点 1**：策略模式的三角色 + 工厂路由，能把登录案例完整讲出来（含"新增方式不改业务代码"的开闭原则论述）。
- **重点 2**：秒杀链路中每个组件的"存在理由"——CDN 为什么放最前、为什么用 Lua 在 Nginx 层判断、热点为什么走 Kafka 而不是直接下单。
- **难点 1**：超卖与原子性是两个不同问题：超卖靠分布式锁（扣 Redis 库存时），扣库存 + 建单的一致性靠分布式事务——别把两者混为一谈。
- **难点 2**：热点识别是"数据驱动"的：日志 → Kafka → Druid → 指标判定 → 同步 Redis，说明热点不是预先设定的，而是实时算出来的。
- **易错点**：并发数 ≠ QPS（差一个响应时间）；把垂直扩展当成主要手段（水平扩展才是根本解）。

## 五、示例

### 示例 1：峰值 QPS 估算

```text
某系统日 PV 3000 万：
峰值 QPS = (30000000 × 80%) / (86400 × 20%) ≈ 1389
若单机可扛 300 QPS，则机器数 ≈ 1389 / 300 ≈ 5 台
```

### 示例 2：秒杀请求的分层过滤

```text
用户请求
  → CDN：静态资源（商品详情页、图片）直接返回，不进后端
  → Nginx（Lua 脚本）：JWT 校验登录、是否重复购买、是否热点、是否有库存、是否在排队
      ├─ 非热点 → 网关 → 商品服务扣库存 + Feign 建订单（分布式事务保证原子性）
      └─ 热点   → Kafka 异步排队 → 热点下单服务消费建单 → 扣 Redis 库存（分布式锁）
                → 订单状态服务 → WebSocket 推送购买结果
```

### 示例 3：策略 + 工厂的注册表写法

```java
private static Map<String, UserGranter> granterPool = new ConcurrentHashMap<>();

@PostConstruct
public void init() {
    granterPool.put("0", accountGranter);   // 用户名密码
    granterPool.put("1", smsGranter);       // 短信验证码
    granterPool.put("2", weChatGranter);    // 微信
}
```

新增"QQ 登录"只需：新建 `QQGranter implements UserGranter` + 在 init 中注册一行，业务层零改动。
