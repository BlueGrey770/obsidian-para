---
type: memo
date: 2026-11-06
aliases:
  - "微服务拆分技术要点"
---

# 微服务拆分技术要点

> 本章围绕把单体工程按业务域拆成独立微服务，覆盖 **工程骨架、Nacos 服务注册、数据库按服务隔离、Feign 远程调用** 四个核心能力。

## 一、技术概念说明

### 1. 微服务拆分的本质

把单体应用中**边界清晰的业务域**（用户、交易、商品、购物车……）各自抽成独立工程：独立启动、独立端口、独立数据库、独立部署。拆分后服务之间不再共享代码和数据库，只能通过**网络接口**协作。

### 2. 微服务工程的标准四件套

每个新服务都要搭好这四样：

1. **pom.xml**：继承 `hmall-parent`（统一版本管理），依赖 `hm-common`（公共工具）、`hm-api`（服务间契约）、web、mysql、mybatis-plus、`spring-cloud-starter-alibaba-nacos-discovery`（注册发现）；
2. **启动类**：`@SpringBootApplication` + `@MapperScan("com.hmall.user.mapper")`（每个服务只扫自己的 mapper 包）；
3. **application.yaml**：`server.port` 各服务不同、`spring.application.name` 服务名（注册到 Nacos 的身份证）、`cloud.nacos.server-addr` 注册中心地址；
4. **独立数据库**：`hm-user`、`hm-trade`——**一个服务一个库**，表不允许跨库 join。

### 3. Nacos 服务注册与发现

- **注册**：服务启动时，把自己的「服务名 + IP + 端口」上报到 Nacos；
- **发现**：调用方凭服务名向 Nacos 要实例列表，负载均衡挑一个发起调用；
- 配置只需两步：引依赖 + 配 `server-addr`，`spring.application.name` 就是注册名。

### 4. Feign：声明式远程调用

单体里的「调本地 Service」在微服务里变成「调远程接口」。Feign 让你**像调本地接口一样发 HTTP 请求**：

- 启动类加 `@EnableFeignClients(basePackages = "com.hmall.api")`（扫描共享的 api 模块中的客户端接口）；
- 远程契约（接口 + DTO）统一放在 `hm-api` 模块，服务双方共同依赖——**契约共享，实现私有**。

### 5. 配置的复用与差异化

从单体复制三个 yaml（主 + dev + local），**环境特有配置**（数据库地址、Nacos 地址）放 `application-dev.yaml` 保持不变，**服务特有配置**（端口、服务名、数据源库名）在 `application.yaml` 中改。`spring.profiles.active` 决定加载哪个环境。

## 二、技术点之间的联系

```
单体 hmall（所有业务一个工程一个库）
  │  按业务域垂直拆分
  ▼
user-service ──┐                    ┌── trade-service
  独立工程      │                    │   独立工程
  独立端口8084  │                    │   独立端口8085
  独立库hm-user │                    │   独立库hm-trade
  ─────────────┼──── Nacos ────────┼──────────────
               │  注册:我是 user-service
               │  发现:给我 trade-service 的地址
               └── Feign 远程调用 ──────────┘
                   （契约在 hm-api 共享模块）

公共层：hmall-parent（版本仲裁）→ hm-common（工具）→ hm-api（契约）
认证层：JWT 秘钥文件 hmall.jks 随服务走（每个服务独立验签）
```

三条关键联系：

1. **拆分 = 物理隔离 + 网络协作**：代码隔离（各自 module）、数据隔离（各自 database）、运行隔离（各自进程）是三面墙；Nacos + Feign 是在墙上开的「标准门」。
2. **三层共享模块是拆分的润滑剂**：parent 管版本、common 管工具、api 管契约——没有这三层，每个服务各维护一份重复代码，拆分反而制造混乱。
3. **认证从「单体内拦截」变成「各服务自验」**：JWT 秘钥文件（hmall.jks）要复制到每个服务，因为每个服务都要独立验证 Token——微服务认证的基本形态。

## 三、方法与实现要点

### 1. 拆一个服务的标准工序

**建 module → 配 pom（五类依赖别漏 nacos-discovery）→ 写启动类（MapperScan 指向本服务包）→ 拷 yaml 改三处（端口/服务名/库名）→ 拷业务代码（只拷本域：user/address/jwt）→ 导独立数据库 → 启动 → knife4j 接口测试**。

### 2. 代码拷贝的边界判断

「复制与 user、address、jwt 有关的代码」——拆分的关键动作是**识别业务域边界**：domain 包按 dto/po/vo 分层拷全，config/controller/mapper/service/util 只留本域，跨域引用（如用户里引用了购物车）必须切断或改为 Feign 调用。

### 3. 验证链条

启动无报错 → Nacos 控制台能看到服务实例 → knife4j（doc.html）调通登录接口拿到 token——三层验证逐级排障：起不来查依赖与配置，注册不上查 nacos 地址，接口不通查数据与代码。

### 4. 端口与库名的分配纪律

每个服务的端口（8081\~8086…）和库名（hm-user/hm-trade/hm-pay…）要在团队内登记，避免冲突；这些属于「服务身份证」信息，写进项目的部署文档。

## 四、重点难点分析

### 重点 1：微服务拆分的两个「独立」

独立工程部署 + 独立数据库。特别是后者——**数据库私有**是微服务铁律，跨库 join 被禁止，跨服务取数必须走接口（Feign）。违反这条，拆分就失去了意义。

### 重点 2：Nacos 注册发现的工作流程

启动注册（服务名 → 实例列表）→ 调用方按名发现 → 负载均衡选实例 → 发起调用。配错 `server-addr` 或漏引 discovery 依赖，服务就是「黑户」，别人找不到。

### 难点 1：单体代码拆分时的依赖解耦

最难的不是新建工程，而是**切断原单体中的交叉引用**：user 模块的代码若调用了 cart 模块的类，编译都过不了。处理原则：能删的删（用不到的功能）、能搬的搬（公共代码进 hm-common）、必须跨域的改 Feign。

### 难点 2：共享 api 模块的设计

`hm-api` 里放 Feign 客户端接口和 DTO——它是「服务间唯一允许共享代码」的地方。设计要点：只放契约（接口、参数、返回值），不放实现；契约变更要双向兼容，否则调用方会崩。

### 易错点清单

- `@MapperScan` 忘记改包路径——扫不到自己 mapper 或扫到别人的；
- yaml 只改了端口没改 `spring.application.name`——两个服务同名注册，互相覆盖；
- 忘记给新服务导数据库脚本（或用了单体老库）——数据混居，拆分失效；
- `@EnableFeignClients` 的 basePackages 没指到 hm-api——Feign 客户端不生效，调用报找不到 Bean。

## 五、示例

### 示例 1：新服务的 pom 依赖骨架

```xml
<project>
    <parent>
        <artifactId>hmall-parent</artifactId>
        <groupId>com.hmall</groupId>
        <version>1.0.0</version>
    </parent>
    <artifactId>user-service</artifactId>

    <dependencies>
        <dependency>                                  <!-- 公共工具 -->
            <groupId>com.hmall</groupId>
            <artifactId>hm-common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>                                  <!-- 服务间契约 -->
            <groupId>com.hmall</groupId>
            <artifactId>hm-api</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <dependency>                                  <!-- 注册发现，最易漏 -->
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 示例 2：启动类（MapperScan 只扫本服务包）

```java
package com.hmall.user;

@MapperScan("com.hmall.user.mapper")     // 只扫自己域的 mapper
@SpringBootApplication
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}
```

### 示例 3：application.yaml 的三处差异化

```yaml
server:
  port: 8084                                   # ① 端口各服务不同
spring:
  application:
    name: user-service                         # ② 服务名 = Nacos 注册名
  profiles:
    active: dev
  datasource:
    url: jdbc:mysql://${hm.db.host}:3306/hm-user?serverTimezone=Asia/Shanghai
    username: root                             # ③ 库名独立：hm-user
    password: ${hm.db.pw}
  cloud:
    nacos:
      server-addr: 192.168.101.68              # 注册中心地址
```

要点：从单体复制 yaml 后，只改这三处；`hm.db.host` / `hm.db.pw` 这类环境变量留在 `application-dev.yaml` 里复用。

### 示例 4：Feign 契约放在 hm-api

```java
// hm-api 模块：com.hmall.api.user.UserClient
@FeignClient("user-service")                   // 值 = 目标服务在 Nacos 的注册名
public interface UserClient {
    @PutMapping("/users/money/deduct")
    void deductMoney(@RequestParam("pw") String pw,
                     @RequestParam("amount") Integer amount);
}
```

```java
// 调用方启动类
@EnableFeignClients(basePackages = "com.hmall.api")
@MapperScan("com.hmall.trade.mapper")
@SpringBootApplication
public class TradeApplication { ... }
```

要点：`@FeignClient` 的 value 是服务名而非 IP——地址由 Nacos 提供，服务扩容/漂移都不用改代码。

### 示例 5：跨服务调用的业务写法

```java
@Service
@RequiredArgsConstructor
public class PayOrderServiceImpl implements IPayOrderService {

    private final UserClient userClient;         // 注入的是 Feign 代理
    private final TradeClient tradeClient;

    @Override
    @Transactional
    public void payOrder(PayOrderFormDTO dto) {
        Long userId = UserContext.getUser();

        // 1. 扣减余额：调用 user-service（远程）
        userClient.deductMoney(dto.getPw(), dto.getAmount());

        // 2. 标记订单支付成功：调用 trade-service（远程）
        tradeClient.markOrderPaySuccess(dto.getOrderId());
    }
}
```

要点：跨服务调用与本地调用写法一致（这就是 Feign 的价值），但**事务不再跨服务生效**——`@Transactional` 只管本地库，跨服务一致性要靠后续的分布式事务方案。

### 示例 6：拆分后的服务清单

| 服务 | 端口 | 独立库 | 职责 |
|-|-|-|-|
| user-service | 8084 | hm-user | 用户、地址、余额 |
| trade-service | 8085 | hm-trade | 订单、订单明细 |
| pay-service | 8086 | hm-pay | 支付单 |
| item-service | 8081 | hm-item | 商品 |
| cart-service | 8082 | hm-cart | 购物车 |

要点：端口与库名是「服务身份证」，团队内必须登记统一，避免撞车。

---
> 来源：飞书技术知识库（脱敏版） · 04 微服务与中间件 / 服务注册与发现 / 01 微服务服务注册与发现 / 微服务拆分技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/CPq4wc0vEiEwMVk8s0ic0fUXnbe)
