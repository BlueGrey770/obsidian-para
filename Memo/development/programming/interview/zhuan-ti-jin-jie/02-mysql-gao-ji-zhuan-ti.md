---
type: memo
date: 2026-11-06
aliases:
  - "面试专题：MySQL 高级（索引、事务、MVCC 与锁）"
---

# 面试专题：MySQL 高级（索引、事务、MVCC 与锁）

## 一、概念说明

### Q1：MySQL 查询语句的执行顺序？

编写顺序是 `select → from → join → where → group by → having → order by → limit`，但实际执行顺序是：

```text
from 表名列表
→ join on 表关联
→ where 条件列表
→ group by 分组字段列表
→ having 分组后的条件列表
→ select 字段列表
→ order by 排序字段列表
→ limit 分页参数
```

这解释了为什么 where 里不能用 select 中定义的别名（执行时别名还不存在）。

### Q2：内连接和外连接的区别？

- 内连接：只取连接表中匹配到的数据，匹配不到的不保留；隐式写法 `select ... from 表A, 表B where 条件`，显式写法 `inner join ... on ...`；
- 外连接：匹配不到的也保留，值为 NULL。左外连接以左表为主表，右外连接以右表为主表——主表数据全保留，关联不上的字段显示 NULL。

### Q3：CHAR 和 VARCHAR 的区别？

1. 最大长度：char 最大 255 字符，varchar 最大 65535 字节；
2. char 定长（不足部分用隐藏空格填充），varchar 不定长；
3. char 浪费空间，varchar 节省空间；
4. 查找效率 char 更高（varchar 需要计算内容长度）。

使用建议：枚举值类用 char，描述信息或名字类用 varchar。

### Q4：索引的底层数据结构？

MySQL 索引基于 B+Tree。对比 B-Tree：5 阶 B 树每个节点最多存 4 个 key 对应 5 个指针，key 数量到达上限就裂变、中间元素向上分裂，且 B 树的**非叶子节点和叶子节点都存数据**。

B+Tree 与 B-Tree 的三点区别：

1. 所有数据都出现在叶子节点；
2. 叶子节点形成一个单向链表；
3. 非叶子节点仅起索引作用，不存数据。

MySQL 又对经典 B+Tree 做了优化：在原 B+Tree 基础上增加指向相邻叶子节点的链表指针，形成带顺序指针的 B+Tree，提高区间访问性能、利于排序。

### Q5：聚簇索引与非聚簇索引？

InnoDB 中按索引存储形式分为两种：

- 聚簇索引：叶子节点直接存放整行数据（主键索引就是聚簇索引）；
- 非聚簇索引（二级索引）：叶子节点存的是主键值，查数据需要再回聚簇索引查一次（回表）。

### Q6：事务的 ACID 与并发问题？

- 原子性（Atomicity）：要么全部成功，要么全部失败；
- 一致性（Consistency）：系统总是从一个一致状态转移到另一个一致状态；
- 隔离性（Isolation）：通常来说一个事务提交之前对其他事务不可见（注意"通常来说"，取决于隔离级别）；
- 持久性（Durability）：事务一旦提交就永远生效，系统崩溃也不影响。

并发事务带来的问题：脏读（读到别的事务未提交的修改）、丢失修改（两个事务先后改同一数据，前者的修改被覆盖）、不可重复读（同一事务两次读取结果不同）、幻读（与不可重复读类似，发生在范围查询时别的事务插入了新行）。

隔离级别从低到高：读未提交 → 读已提交（解决脏读）→ 可重复读（MySQL 默认，解决不可重复读）→ 串行化（解决幻读，性能最差）。

### Q7：MySQL 有哪些锁？

按粒度分三类：

- 全局锁：锁定数据库中的所有表；
- 表级锁：每次操作锁住整张表，锁定粒度大、冲突概率高、并发度低（MyISAM、InnoDB、BDB 都支持）；
- 行级锁：每次操作锁住对应行，粒度最小、冲突概率最低、并发度最高（仅 InnoDB）。

### Q8：意向锁的作用？

避免 DML 加的行锁与表锁冲突。没有意向锁时，客户端要给表加表锁，需要逐行检查是否有行锁，效率极低。有了意向锁：执行 DML 时对涉及的行加行锁，同时给表加上意向锁；其他事务加表锁时只需看表上的意向锁，不用逐行判断。

### Q9：共享锁与排他锁？

InnoDB 实现两种行锁：共享锁（S）允许一个事务读一行，阻止其他事务获得相同数据集的排他锁；排他锁（X）允许持有事务更新数据，阻止其他事务获得共享锁和排他锁。

| SQL | 行锁类型 | 说明 |
| --- | --- | --- |
| INSERT / UPDATE / DELETE | 排他锁 | 自动加锁 |
| SELECT（正常） | 不加任何锁 | 快照读 |
| SELECT ... LOCK IN SHARE MODE | 共享锁 | 手动加 |
| SELECT ... FOR UPDATE | 排他锁 | 手动加 |

### Q10：解释一下 MVCC？

MVCC 全称 Multi-Version Concurrency Control（多版本并发控制）：维护一个数据的多个版本，使读写操作没有冲突，快照读就是 MySQL 实现 MVCC 的非阻塞读。具体实现依赖三个东西：

1. **三个隐式字段**：`DB_TRX_ID`（最近修改事务 ID，记录插入或最后一次修改该记录的事务 ID，自增）、`DB_ROLL_PTR`（回滚指针，指向这条记录的上一个版本，配合 undo log）、`DB_ROW_ID`（隐藏主键，表没有指定主键时才生成）；
2. **undo log**：回滚日志，insert/update/delete 时产生。insert 的 undo log 只在回滚时需要，事务提交后可立即删除；update/delete 的 undo log 在快照读时也需要，不会立即删除——旧版本数据正是靠它保留的；
3. **ReadView**：快照读 SQL 执行时 MVCC 提取数据的依据，记录并维护系统当前活跃的事务。

### Q11：MVCC 如何实现可重复读？

在 RR 隔离级别下，事务只在**第一次快照读时生成 ReadView**，后续相同的 select 都复用这个 ReadView；ReadView 相同、版本链匹配规则相同，快照读返回的结果自然一样——这就是可重复读。（对比：RC 读已提交级别下，每次快照读都生成新的 ReadView，所以能读到别的事务已提交的修改。）

总结：MVCC 通过隐藏字段 + UndoLog 版本链 + ReadView 实现；MVCC + 锁实现了事务的隔离性；一致性由 redo log 与 undo log 保证。

## 二、技术点之间的联系

- B+Tree 是索引的物理基础，聚簇/非聚簇是索引的组织方式，回表是二者的交叉点，覆盖索引是避免回表的优化手段——一条 SELECT 快不快，取决于这条链路上走了几步。
- 事务隔离级别的"读问题"（脏读/不可重复读/幻读）与 MVCC 是因果关系：MVCC 用版本链 + ReadView 把"读旧版本"变成可能，不同隔离级别只是 ReadView 生成时机不同。
- 锁与 MVCC 互补：快照读走 MVCC 不加锁，当前读（UPDATE/DELETE/SELECT FOR UPDATE）走锁。"MVCC + 锁 = 隔离性"是标准总结句。
- undo log 一鱼三吃：回滚靠它、MVCC 版本链靠它、原子性靠它；redo log 保持久性。

## 三、方法与实现要点

- 答索引时按"B+Tree 结构 → 聚簇 vs 非聚簇 → 回表 → 覆盖索引 → 最左前缀"的顺序展开，形成完整链路；
- 答 MVCC 时先说一句话定义（多版本并发控制、读写不冲突、非阻塞读），再列三个依赖（隐式字段、undo log、ReadView），最后用 RR 的 ReadView 复用收尾；
- 答锁时按粒度（全局/表/行）→ 意向锁存在的理由 → S/X 锁矩阵 → 常见 SQL 的加锁规则逐层展开；
- 被问"select 要不要加锁"要区分快照读（不加锁，走 MVCC）与当前读（加锁）。

## 四、重点与难点分析

- **重点 1**：B+Tree 与 B-Tree 的三点区别（数据全在叶子、叶子链表、非叶子只索引），以及 MySQL 的双向链式优化对区间查询的意义。
- **重点 2**：RR 靠"ReadView 只生成一次"实现可重复读，RC 靠"每次都生成新 ReadView"实现读已提交——同一个机制，生成时机不同。
- **难点 1**：幻读与不可重复读的区别。不可重复读针对同一行的修改，幻读针对范围查询时插入的新行；RR 级别下 MySQL 通过 MVCC（快照读）+ 临键锁（当前读）基本解决幻读。
- **难点 2**：意向锁是"表级的预告牌"，本身不锁数据，只是让表锁的检查从 O(n) 变 O(1)。
- **易错点**：认为 SELECT 默认加锁（普通 SELECT 是快照读不加锁）；认为 update/delete 的 undo log 提交后即可删（快照读还要用）。

## 五、示例

### 示例 1：版本链的形成过程

```text
原始记录：DB_TRX_ID=1, DB_ROLL_PTR=null
事务2 修改：记录 undo log（旧值）→ 更新记录，TRX_ID=2，ROLL_PTR 指向旧版本
事务3 修改：同样记录 undo log → TRX_ID=3，回滚指针指向事务2的版本
事务4 修改：→ TRX_ID=4，回滚指针指向事务3的版本
结果：当前记录 → v4 → v3 → v2 → v1，形成版本链，快照读按规则沿链找可见版本
```

### 示例 2：RR 与 RC 的 ReadView 差异

```sql
-- RR（可重复读）：第一次快照读生成 ReadView，之后复用
BEGIN;
SELECT * FROM account WHERE id = 1;  -- 生成 ReadView
-- 其他事务修改并提交
SELECT * FROM account WHERE id = 1;  -- 复用旧 ReadView，结果不变

-- RC（读已提交）：每次快照读都生成新 ReadView
BEGIN;
SELECT * FROM account WHERE id = 1;
-- 其他事务修改并提交
SELECT * FROM account WHERE id = 1;  -- 新 ReadView，读到已提交的新值
```

### 示例 3：手动加锁的两种写法

```sql
-- 共享锁：允许别人读，阻止别人写
SELECT * FROM account WHERE id = 1 LOCK IN SHARE MODE;

-- 排他锁：阻止别人读写（当前读）
SELECT * FROM account WHERE id = 1 FOR UPDATE;
```

### 示例 4：事务隔离级别的设置与查看

```sql
-- 查看当前隔离级别
SELECT @@transaction_isolation;
-- 设置会话隔离级别（读已提交）
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```
