---
type: memo
date: 2026-11-06
aliases:
  - "wei-fu-wu-rong-cuo-mian-shi-ti"
  - "微服务容错面试题（限流、熔断、降级与监控）"
---

# 微服务容错面试题（限流、熔断、降级与监控）

## 一、概念说明

库内八股文覆盖了 SpringCloud 五大组件、Eureka、Gateway 断言与 Seata，本篇补齐"服务治理的稳定性手段"：限流算法、断路器状态机、服务降级、链路监控，以及 Nacos/Eureka 对比、Ribbon 负载均衡策略。

### 1. 限流

限流算法主要有两种：漏桶算法、令牌桶算法。

- **漏桶算法**：可以粗略认为就是注水漏水过程。往桶中以任意速率流入水，以一定速率流出水；水超过桶容量则丢弃。因为桶容量不变，保证了整体的流出速率恒定。
- **令牌桶算法**：桶里存放固定容量的令牌，按固定速率 r 往桶里添加令牌，桶满时新令牌被丢弃。请求到达时尝试取令牌：有则继续处理，没有则排队等待或直接丢弃。

两者最明显的区别是**是否允许突发流量**：漏桶强行限制实时处理速率，对突发流量不做额外处理；令牌桶在限制平均速率的同时允许一定程度的突发（流出速率可能瞬间大于 r）。

### 2. 断路器（熔断器）

以 Hystrix 为例，默认关闭，需要在引导类上加 `@EnableCircuitBreaker` 开启。断路器状态机有三个状态：

| 状态 | 行为 |
| --- | --- |
| closed（关闭） | 放行所有请求，并统计异常比例、慢请求比例；超过阈值（默认请求错误率 5%）切换到 open |
| open（打开） | 服务调用被熔断，请求被拒绝、快速失败，直接走降级逻辑；默认 5 秒后进入 half-open |
| half-open（半开） | 放行一次请求，按执行结果决定：成功则回到 closed，失败则回到 open |

### 3. 服务降级

降级逻辑一般是返回默认值。实现非常简单：创建一个类实现 `FallbackFactory` 接口，然后在对应 Feign 客户端接口上通过 `@FeignClient` 指定降级类。

### 4. 微服务监控

链路监控用 SkyWalking：无侵入地采集服务调用链、耗时、异常，配合告警定位慢接口与故障服务。

### 5. 注册中心：Nacos 与 Eureka 的区别

| 维度 | Nacos | Eureka |
| --- | --- | --- |
| CAP | 支持 AP 与 CP 切换 | AP |
| 功能 | 注册中心 + 配置中心二合一 | 仅注册中心 |
| 健康检查 | 心跳 + 主动探测 | 客户端心跳 |
| 生态 | Spring Cloud Alibaba 体系 | Spring Cloud Netflix 体系（已停止新特性维护） |

### 6. 负载均衡：Ribbon 的策略与自定义

常见策略：轮询、随机、权重响应时间、最小并发、重试等。自定义两种方式：

1. **代码方式**：在服务的启动类中定义一个新的 `IRule` 实现（注意作用范围，避免污染全局）；
2. **配置文件方式**：在 `application.yml` 中按"服务名.ribbon.NFLoadBalancerRuleClassName"配置。

## 二、技术点之间的联系

- 限流、熔断、降级是稳定性三板斧，作用点不同：限流在入口挡住超额流量；熔断在下游持续失败时快速失败，避免线程池被拖垮；降级在失败时提供兜底返回值。三者常在网关（限流）+ 服务调用（熔断、降级）两处配合落地。
- 熔断的 half-open 状态本质是"探活"：用小流量试探下游是否恢复，与限流中的令牌桶"试探性放行"思路一致。
- 降级依赖熔断的判定：断路器打开后请求被拒绝，拒绝时执行的正是降级逻辑——两者是判定与处置的关系。
- 负载均衡策略与限流、熔断共同构成流量治理闭环：先按策略分摊流量，超载时限流，失败时熔断降级。

## 三、方法与实现要点

### 1. 限流的落地位置

通常在网关层做全局限流（如 Gateway 的 RequestRateLimiter 过滤器，基于令牌桶），也可在服务内按接口做细粒度限流。按用户 IP 限流时令牌常需要存 Redis，因此要集成 Redis。

### 2. 熔断参数的调优视角

- 错误率阈值过低（如 5%）会导致误熔断，需要结合下游 SLA 调整；
- open 状态的时长决定恢复速度：太长影响可用性，太短会导致雪崩式重试；
- half-open 只放行一次请求，说明它是"试探"而非"恢复"。

### 3. 降级的写法

```java
@FeignClient(value = "item-service", fallbackFactory = ItemClientFallbackFactory.class)
public interface ItemClient {
    @GetMapping("/items")
    List<Item> queryItemsByIds(@RequestParam("ids") Collection<Long> ids);
}

@Component
public class ItemClientFallbackFactory implements FallbackFactory<ItemClient> {
    @Override
    public ItemClient create(Throwable cause) {
        return ids -> {
            // 降级逻辑：返回兜底数据，避免调用方被拖垮
            return Collections.emptyList();
        };
    }
}
```

### 4. 监控指标的选择

关注三类指标：调用量/错误率（判断是否熔断）、P99 耗时（判断是否需要限流）、依赖拓扑（判断故障传播路径）。

## 四、重点与难点分析

- **重点 1**：漏桶与令牌桶的区别要落到"是否允许突发"，并用一句话说清桶容量与速率的含义。
- **重点 2**：断路器三状态及迁移条件（closed→open 靠错误率阈值，open→half-open 靠时间，half-open→closed/open 靠试探结果）。
- **难点 1**：为什么需要 half-open。若 open 到期直接回到 closed，下游尚未恢复时会立刻再次被压垮——half-open 用小流量试探，是保护下游的缓冲。
- **难点 2**：限流与降级的分工常被混淆。限流是"不让请求进来"，降级是"请求进来了但给兜底结果"；熔断是"下游坏了先别调"。
- **易错点**：把熔断阈值说成固定值（实际可配置，Hystrix 默认错误率 5%）；忽略降级要设置超时，否则兜底逻辑本身成为新的阻塞点。

## 五、示例

### 示例 1：令牌桶限流的参数理解

```text
桶容量 b：允许积压的突发请求上限
添加速率 r：每秒补充的令牌数（即平均放行速率）
请求到达：取到令牌 → 放行；取不到 → 排队或丢弃
结论：平均速率受 r 限制，瞬时速率可大于 r（消耗存量令牌）→ 可应对突发流量
```

### 示例 2：断路器状态迁移回答模板

```text
closed：放行全部请求，统计错误率与慢调用比例；错误率超过阈值（默认 5%）→ open
open：拒绝请求，快速失败并走降级；默认 5 秒后 → half-open
half-open：放行一次请求试探；成功 → closed，失败 → open
```

### 示例 3：自定义 Ribbon 负载均衡策略

```java
// 代码方式：为指定服务定义新的 IRule（注意作用域，避免全局污染）
@Bean
public IRule randomRule() {
    return new RandomRule();
}
```

```yaml
# 配置文件方式
item-service:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

### 示例 4：降级 + 监控的完整链路

```text
请求 → 网关（令牌桶限流，超出直接 429）
     → 服务 A 调服务 B（Feign + 断路器统计错误率）
     → 错误率超阈值：断路器 open，请求快速失败 → FallbackFactory 返回兜底数据
     → SkyWalking 记录调用链：定位是 B 慢还是 A 慢
```
