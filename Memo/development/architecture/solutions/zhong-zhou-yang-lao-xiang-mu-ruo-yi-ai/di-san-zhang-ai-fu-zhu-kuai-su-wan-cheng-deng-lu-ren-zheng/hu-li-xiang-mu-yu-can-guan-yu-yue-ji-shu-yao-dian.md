---
type: memo
date: 2026-11-06
aliases:
  - "护理项目与参观预约技术要点"
---

# 护理项目与参观预约技术要点

> 本章覆盖两个业务功能：**护理项目展示下单**（小程序端服务选购）与**参观/探访预约**（表结构设计、状态机、定时任务自动过期）。核心是从需求到表结构的设计方法、业务状态机建模、时间驱动的业务规则落地。

## 一、技术概念说明

### 1. 护理项目：典型的「列表 + 详情」展示型业务

小程序端展示后端维护的护理项目（修剪指甲、助浴、洗头……），结构是最经典的电商式两段：**列表页**（名称 + 价格）→ **详情页**（介绍图文 + 服务家人 + 期望时间 + 立即购买）。技术上没有新框架，是「按接口文档开发接口 + 小程序页面渲染」的组合，重点在**数据模型与展示需求的对应**。

### 2. 预约表设计：从需求到字段的推导过程

教科书式的表设计四步：

**第 1 步 · 从需求里抠业务字段**：预约人姓名、手机号、家人姓名（探访人）、预约时间。

**第 2 步 · 补冗余/审计字段**（经验值约 6 个）：主键、创建时间、修改时间、创建人、修改人、备注。

**第 3 步 · 按业务规则加维度字段**：

- 一张表存两种预约 → 加 `type`（0 参观 / 1 探访）——**同构数据合并存储，用类型字段区分**；
- 预约有生命周期 → 加 `status`（0 待报道 / 1 已完成 / 2 取消 / 3 过期）。

**第 4 步 · 用约束兜底业务规则**：「同一手机号同一时间段只能预约一次」→ 唯一索引 `UNIQUE KEY (mobile, time)`——**把业务规则固化到数据库层**，比只靠代码校验可靠。

### 3. 状态机：预约的生命周期

```
            ┌──────────────┐
   创建预约  │              ▼
──────────▶ 待报道(0) ──到院──▶ 已完成(1)
              │
              ├─用户主动取消──▶ 取消(2)
              │
              └─超过预约时间未到──▶ 过期(3)  ← 定时任务驱动
```

四种状态、三条流转边。建模要点：**状态值用数字 + 注释**，状态流转只在明确的业务动作中发生，禁止随意 set。

### 4. 定时任务：处理「时间驱动」的状态流转

「过期」这个状态没有任何用户操作触发，只能靠系统自己巡检——这就是定时任务的典型场景：每小时第 1、31 分钟扫描 `status = 0 且 time < 当前时间` 的记录，批量改为已过期。技术方案：`@Scheduled`（cron 表达式）或 xxl-job。

## 二、技术点之间的联系

```
需求文档/原型图
  │  抠字段 ──▶ 表结构设计（业务字段 + 审计字段 + 维度字段 + 约束）
  ▼
reservation 表 ──承载──▶ 预约接口开发（增删改查 + 状态流转）
  │                        │
  │  status 字段 ◀──────────┤ 用户动作驱动：报道/取消
  │      ▲                  │
  │      └──── 定时任务驱动：过期（系统动作）
  │
  ├─ type 字段 ──复用──▶ 一张表服务「参观/探访」两种业务
  │
  └─ 唯一索引(mobile,time) ──兜底──▶ 「一人一时段一预约」规则

前置依赖：小程序登录 + 拦截器取用户信息（预约单要知道「谁」在预约）
护理项目：独立的展示型支线，与预约共享「小程序端接口开发」套路
```

三条关键联系：

1. **type + status 是业务表设计的两大高频维度**：type 解决「一类表存多种业务」，status 解决「业务有生命周期」——养老项目里的预约、订单、合同全是这个套路。
2. **登录认证是预约接口的前置**：预约单上的「预约人」不是手填的，而是从 Token 里解析的，认证与业务在这里合流。
3. **定时任务与状态机是固定搭档**：凡是状态机里有「超时就自动变」的边，实现上一定有定时任务在后面跑——看到状态机就想到问「哪些边是系统驱动的」。

## 三、方法与实现要点

### 1. 表设计的自查四问

拿到需求先问：业务字段抠全了吗？审计字段补了吗（主键/创建修改时间/创建修改人/备注）？需要 type/status 维度吗？哪些业务规则可以下沉成约束（唯一索引、非空、默认值）？——四问答完，建表语句自然成型。

### 2. 状态值用数字而不用字符串

`status int comment '0待报道,1已完成,2取消,3过期'`：存储省、比较稳、国际化友好；展示层再映射文案。**注释必须写全取值含义**，后人维护不靠猜。

### 3. 定时任务的频率设计

「每小时 1 分和 31 分钟」而不是整点——错开整点高峰是定时任务的常见优化；检查逻辑要**幂等**（重复执行结果一致），过期判定条件写全（`status = 0 AND time < now`），避免误伤已完成/已取消的单子。

### 4. 接口开发先读文档再动手

两个功能的接口都以接口文档为准：路径、参数、响应结构逐项对齐，再进入三层架构套路开发——流程与前序模块完全一致，变化只在业务建模。

## 四、重点难点分析

### 重点 1：type/status 维度字段的建模价值

合并存储（一张表 + type）vs 拆分存储（参观表、探访表两张）的取舍：字段高度重合、业务规则相同 → 合并。状态机让「预约后来怎么样了」可追踪、可统计（待报道多少、转化率多少），是运营数据的基础。

### 重点 2：唯一约束兜底并发

「同手机号同时间段只能约一次」只靠代码先查后插，并发下会双双通过（两个请求同时查都没查到）；数据库唯一索引是**最后一道也是唯一可靠的防线**，代码层捕获 `DuplicateKeyException` 转成友好提示即可。

### 难点 1：定时任务与业务并发

用户恰好在定时任务扫描的瞬间到院报到——任务要把单子改过期，报到要把单子改已完成。处理原则：状态流转 SQL 都带**前置状态条件**（`update ... set status = 3 where id = ? and status = 0`），后执行者因条件不满足自然失败，避免互相覆盖。

### 难点 2：时间的边界语义

「超过预约时间未到」的判定是 `time < now` 还是 `time + 宽限期 < now`，属于业务口径，必须在需求里明确并写进代码注释——过期判定的边界差一分钟，线上就是一批客诉。

### 易错点清单

- 状态直接 `set status = 3` 不带前置条件——并发下覆盖用户刚完成的报到；
- 唯一索引字段包含可空列——NULL 不参与唯一性校验，规则形同虚设（本表 mobile/time 均非空，是正确的）；
- 定时任务没配幂等 / 没控制扫描范围（全表扫）——数据量上来后拖垮数据库；
- type/status 注释不完整——三个月后没人知道 2 是取消还是拒绝。

## 五、示例

### 示例 1：预约表建表语句

```sql
CREATE TABLE `reservation` (
  `id`          bigint NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `name`        varchar(50) NOT NULL COMMENT '预约人姓名',
  `mobile`      varchar(20) NOT NULL COMMENT '预约人手机号',
  `time`        datetime    NOT NULL COMMENT '预约时间',
  `visitor`     varchar(50) NOT NULL COMMENT '探访人',
  `type`        int NOT NULL COMMENT '预约类型，0：参观预约，1：探访预约',
  `status`      int NOT NULL COMMENT '预约状态，0：待报道，1：已完成，2：取消，3：过期',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `create_by`   bigint   DEFAULT NULL COMMENT '创建人id',
  `update_by`   bigint   DEFAULT NULL COMMENT '更新人id',
  `remark`      varchar(255) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_mobile_time` (`mobile`, `time`)
) COMMENT '预约信息表';
```

要点：业务字段 4 个 + 审计字段 6 个 + 维度字段 2 个（type / status）+ 1 条唯一约束，字段清单是推导出来的，不是拍出来的。

### 示例 2：状态流转的四个动作（SQL 都带前置状态）

```sql
-- 创建：初始状态待报道
insert into reservation(name, mobile, time, visitor, type, status, create_time, create_by)
values (#{name}, #{mobile}, #{time}, #{visitor}, #{type}, 0, now(), #{userId});

-- 到院报到：只能从「待报道」流转到「已完成」
update reservation set status = 1, update_time = now()
where id = #{id} and status = 0;

-- 用户取消：待报道才允许取消
update reservation set status = 2, update_time = now()
where id = #{id} and status = 0 and create_by = #{userId};

-- 定时任务置过期：批量、幂等、只动待报道
update reservation set status = 3, update_time = now()
where status = 0 and time < now();
```

要点：每条 UPDATE 都带 `status = 0` 前置条件——并发下后到的那条自然影响 0 行，天然避免状态互相覆盖。

### 示例 3：定时任务（每小时 1 分、31 分）

```java
@Component
public class ReservationExpireTask {

    @Autowired
    private ReservationMapper reservationMapper;

    // 每小时的第 1 分钟和第 31 分钟执行，错开整点高峰
    @Scheduled(cron = "0 1,31 * * * ?")
    public void expire() {
        int rows = reservationMapper.expireOverdue();
        if (rows > 0) {
            log.info("本次置为已过期的预约数：{}", rows);
        }
    }
}

// 启动类开启定时任务支持
@EnableScheduling
@SpringBootApplication
public class ReservationApplication { ... }
```

要点：`@EnableScheduling` 不能漏；任务本身幂等（重复跑只是影响 0 行），不需要分布式锁即可满足单机场景。

### 示例 4：预约人从 Token 中取（不从前端传）

```java
@PostMapping("/reservations")
public Result create(@RequestBody ReservationDTO dto) {
    Long userId = UserContext.getUser();       // 拦截器解析 Token 后写入
    if (userId == null) return Result.error("未登录");
    reservationService.create(dto, userId);     // 预约人 = 当前登录用户
    return Result.ok("预约成功");
}
```

要点：预约人由服务端确定，前端传的「预约人」不可信——这是认证与业务合流的典型位置。

### 示例 5：唯一约束的冲突处理

```java
try {
    reservationMapper.insert(entity);
} catch (DuplicateKeyException e) {
    // uk_mobile_time 冲突：同一手机号同一时间段已有预约
    throw new BizException("您在该时间段已有预约，请选择其他时间");
}
```

要点：先查后插挡不住并发，最终要靠唯一索引报错兜底；把数据库异常翻译成业务提示，前端才能展示可读文案。

### 示例 6：护理项目列表与详情的接口形态

```java
@GetMapping("/nursing/items")
public Result list(@RequestParam(required = false) String keyword) {
    return Result.ok(nursingService.listByName(keyword));
}

@GetMapping("/nursing/items/{id}")
public Result detail(@PathVariable Long id) {
    NursingItemVO vo = nursingService.getDetail(id);   // 含图文介绍、价格、服务家人
    return vo == null ? Result.error("服务不存在") : Result.ok(vo);
}
```

要点：列表只返回轻量字段（名称、价格、图片），详情才带富文本与购买所需信息——这是「列表 + 详情」业务的通用接口设计。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第三章-AI辅助快速完成登录认证 / 护理项目与参观预约技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/HAh7wRxeBiWW3ek2RT1c7f1snNb)
