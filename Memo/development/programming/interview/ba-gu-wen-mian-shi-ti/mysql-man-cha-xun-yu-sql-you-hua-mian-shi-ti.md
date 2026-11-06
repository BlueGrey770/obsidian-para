---
type: memo
date: 2026-11-06
aliases:
  - "MySQL 慢查询与 SQL 调优面试题"
---

# MySQL 慢查询与 SQL 调优面试题

## 一、概念说明

库内八股文已覆盖索引原理、回表、最左匹配、事务与隔离级别。本篇补齐"调优实操"这一段：慢查询日志、EXPLAIN 执行计划、索引失效的十种场景、超大分页、主从同步延时、分库分表策略。

### 1. 慢查询日志

慢查询日志记录所有执行时间超过指定参数（`long_query_time`，单位秒，默认 10 秒）的 SQL 语句。开启方式是在 MySQL 配置文件（`/etc/my.cnf`）中配置，重启服务后可在 `/var/lib/mysql/localhost-slow.log` 查看：

```ini
# 开启 MySQL 慢日志查询开关
slow_query_log=1
# SQL 执行时间超过 2 秒即视为慢查询，记录日志
long_query_time=2
```

### 2. EXPLAIN 执行计划

用 `EXPLAIN` 或 `DESC` 命令获取 MySQL 如何执行 SELECT 语句的信息，包括表如何连接、连接的顺序：

```sql
EXPLAIN SELECT 字段列表 FROM 表名 WHERE 条件;
```

关键字段：

| 字段 | 含义 |
| --- | --- |
| id | 查询序列号，表示执行 select 子句或操作表的顺序（id 相同从上到下执行；id 不同则值越大越先执行） |
| select_type | SELECT 类型：SIMPLE（简单表）、PRIMARY（主查询）、UNION、SUBQUERY 等 |
| type | 连接类型，性能由好到差：NULL > system > const > eq_ref > ref > range > index > all |
| possible_key | 可能用到的索引，一个或多个 |
| key | 实际使用的索引，为 NULL 表示没有使用索引 |
| key_len | 索引中使用的字节数（字段最大可能长度，非实际长度），不损失精确性时越短越好 |
| rows | MySQL 认为必须执行查询的行数（InnoDB 中是估计值） |
| filtered | 返回结果行数占需读取行数的百分比，越大越好 |
| Extra | 额外建议，如 Using index、Using where、Using filesort 等 |

判断 SQL 是否需要优化，主要看 type（连接类型）、possible_key（是否可能命中索引）、key（实际命中索引）、key_len（索引占用大小）、Extra（额外优化建议）。

### 3. 索引失效的典型场景

1. **最左前缀法则**：多列索引要从最左前列开始且不跳过索引中的列；跳过某一列时只有最左列索引生效；
2. **范围查询右边的列不能使用索引**：`name`、`status`、`address` 三列索引，按 name、status 查询走索引，但 status 是范围条件时其后的 address 用不到索引；
3. **索引列上做运算**，索引失效；
4. **字符串不加单引号**：查询优化器会自动做类型转换，造成索引失效；
5. **尽量用覆盖索引，避免 `select *`**：查询列超出索引列会降低性能；
6. **or 连接的列中有一个没索引**，涉及的索引都不会被用到；
7. **以 % 开头的 like 模糊查询**索引失效（仅尾部模糊匹配不会失效），可用覆盖索引缓解；
8. **MySQL 评估用索引比全表更慢**时不使用索引；
9. `is NULL`、`is NOT NULL` 有时索引失效；
10. `in` 走索引，`not in` 索引失效。

Extra 的四个常见取值含义：`using index` 表示使用覆盖索引；`using where` 表示需回表查询数据；`using index condition` 表示使用了索引但仍需回表；`using index; using where` 表示所需数据都在索引列中，无需回表。

### 4. 超大分页

数据量大时 `limit` 分页越往后越慢。原因：`limit 2000000,10` 需要 MySQL 排序前 2000010 条记录，仅返回 2000000~2000010 这 10 条，其余全部丢弃，排序代价非常大。优化思路：用**覆盖索引 + 子查询**，先在索引上取出主键，再回表取数据。

### 5. 主从同步与读写分离延时

主从复制的核心是二进制日志（BINLOG），它记录所有 DDL 和 DML 语句，不包括 SELECT、SHOW 等查询。同步三步：

1. Master 主库在事务提交时把数据变更记录到 Binlog；
2. 从库读取主库的 Binlog，写入从库的中继日志 Relay Log；
3. Slave 重做中继日志中的事件，将变更反映到自己的数据。

延迟原因：**随机重放**——主库 binlog 是顺序写所以很快，而 Slave 的 SQL Thread 重放 DML/DDL 是随机 IO，成本高得多，跟不上主库写 binlog 的速度；**锁等待**——SQL Thread 单线程，主库并发高或从库有大查询产生锁等待时延时加剧。

解决方案：

- 强制读主库：支付类强实时业务直接读主库（从库退化为备份）；
- 延迟读：写入后等一段时间再读（问题是不知道同步完成需要多久）；
- 降低并发：控制主库写入速度，减少延迟概率；
- **并行复制（推荐）**：MySQL 5.6 之后把 SQL 线程转为多个 work 线程并行重放。

### 6. 分库分表

单库瓶颈来自 IO（热点数据多、缓存不足、磁盘 IO 高、网络带宽不够）与 CPU（排序、分组、连接查询、聚合统计消耗大）。中心思想是**把数据分散存储**，让单一库/表的数据量变小。

- **垂直拆分**：按业务/字段拆分（垂直分库按业务拆库，垂直分表把冷热字段拆开）；
- **水平拆分**：以字段为依据按策略拆分。水平分库：每个库表结构一样、数据不一样、并集是全量；水平分表同理。

实现技术：ShardingJDBC（基于 AOP 在应用层拦截 SQL，解析、改写、路由，需自行编码，仅支持 Java，性能较高）；MyCat（数据库中间件，不改代码，支持多语言，性能不及前者）。

分片后主键：数据库中间件可设置主键自增；UUID 本地生成不依赖数据库，但太长占空间、作为主键性能差且无序（会导致 B+ 树频繁分裂）。

## 二、技术点之间的联系

- 慢查询日志负责"发现"，EXPLAIN 负责"定位"，索引优化与 SQL 改写负责"治疗"——这是 SQL 调优的标准闭环。
- 索引失效的十条场景中，最左前缀、覆盖索引、回表三者互为因果：最左前缀决定能否用上联合索引，覆盖索引决定是否需要回表，回表次数决定实际耗时。
- 超大分页的优化本质是"减少回表"：先用覆盖索引拿到主键，再按主键回表取少量数据，把 `limit` 的排序代价从全表降到索引扫描。
- 主从复制、读写分离、分库分表是数据量增长的三个台阶：先加从库扛读，再垂直拆业务，最后水平分片扛写。

## 三、方法与实现要点

### 1. 定位慢 SQL 的步骤

1. 开启慢查询日志，设置合理的 `long_query_time`；
2. 从日志中捞出耗时最长的 SQL；
3. 对目标 SQL 执行 `EXPLAIN`，看 type 是否退化到 index/all、key 是否为空、rows 是否过大、Extra 是否有 filesort；
4. 建索引或改写 SQL（避免函数运算、避免前导 %、避免隐式类型转换）；
5. 复测并回归。

### 2. 超大分页的改写

```sql
-- 慢：排序 200 万行只取 10 行
SELECT * FROM tb_sku LIMIT 2000000, 10;

-- 快：先用覆盖索引在索引树上定位主键，再回表
SELECT s.* FROM tb_sku s
JOIN (SELECT id FROM tb_sku ORDER BY id LIMIT 2000000, 10) t
ON s.id = t.id;
```

### 3. 表设计与索引设计要点

- 字段类型按实际范围选最小够用的（能用 tinyint 不用 int）；
- 字符串枚举值可用 char，描述性文本用 varchar（char 定长查找更快但浪费空间，varchar 节省空间但需计算长度）；
- 联合索引把区分度高、查询频率高的列放前面；
- 控制索引数量：索引越多写入越慢。

### 4. 分库分表的取舍

优先做垂直拆分（按业务解耦），水平拆分放在最后。水平分片要考虑：分片键选择（避免跨片查询）、分布式 ID、跨片分页与聚合、分布式事务。

## 四、重点与难点分析

- **重点 1**：EXPLAIN 的 type 等级顺序（const > eq_ref > ref > range > index > all），看到 all 就要警惕全表扫描。
- **重点 2**：索引失效的判定要能对应到具体 SQL 形态，尤其是"索引列参与运算""字符串不加引号""or 两侧有一侧无索引"这三条高频项。
- **难点 1**：范围查询之后的列用不上索引。本质是 B+ 树在范围扫描后无法继续利用后续列的排序性，所以联合索引的列顺序要把等值条件放前、范围条件放后。
- **难点 2**：主从延时的根因是"binlog 顺序写 vs 从库随机重放"的不对称，能讲清这一点，解决方案（并行复制）就顺理成章。
- **易错点**：认为"加了索引就一定走索引"（MySQL 会基于成本估算放弃索引）；认为分表后查询一定更快（跨片查询与聚合反而更慢）。

## 五、示例

### 示例 1：慢查询日志配置与验证

```ini
# /etc/my.cnf
slow_query_log=1
long_query_time=2
slow_query_log_file=/var/lib/mysql/localhost-slow.log
```

配置完成后重启 MySQL，执行一条耗时超过 2 秒的 SQL，即可在日志文件中看到记录。

### 示例 2：EXPLAIN 结果判读

```sql
EXPLAIN SELECT * FROM tb_seller WHERE name = 'abc' AND status = 1;
```

```text
重点关注：
type = ref          → 命中非唯一索引，尚可
key  = idx_name     → 实际用了 idx_name
rows = 120          → 预估扫描 120 行
Extra= Using where  → 需要回表取数据
若 type = ALL 且 key = NULL → 全表扫描，必须优化
```

### 示例 3：最左前缀与范围条件

```sql
-- 联合索引 idx_name_status_address(name, status, address)
SELECT * FROM tb_seller WHERE name = 'a' AND status = 1 AND address = 'x';  -- 三列都走索引
SELECT * FROM tb_seller WHERE status = 1 AND address = 'x';                 -- 违反最左前缀，索引失效
SELECT * FROM tb_seller WHERE name = 'a' AND status > 1 AND address = 'x';  -- status 是范围，address 用不到索引
```

### 示例 4：or 导致索引失效

```sql
-- name 有索引，createtime 无索引 → 整体不走索引
EXPLAIN SELECT * FROM tb_seller
WHERE name = 'abc' OR createtime = '2088-01-01 12:00:00';
```

### 示例 5：主从同步三步的回答模板

```text
1. 主库事务提交，把变更写入 Binlog；
2. 从库 IO 线程读取主库 Binlog，写入本地 Relay Log；
3. 从库 SQL 线程重放 Relay Log，数据与主库保持一致。
延时根因：Binlog 顺序写很快，从库重放是随机 IO，单线程跟不上 →
解决：并行复制（5.6+ 多 work 线程重放）为主，辅以强制读主库/延迟读/降低并发。
```
