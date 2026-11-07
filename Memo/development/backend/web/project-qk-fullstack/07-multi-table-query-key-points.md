---
type: memo
date: 2026-11-06
aliases:
  - "07-duo-biao-cha-xun-ji-shu-yao-dian"
  - "07 多表查询技术要点"
---

# 07 多表查询技术要点

> 多表查询是 SQL 从「单表增删改查」走向「真实业务」的关键一跃。本章覆盖连接查询、四种子查询、非等值连接与分组统计的组合。

## 一、技术概念说明

### 1. 连接查询（JOIN）：让表与表对话

员工表只有 `dept_id`，部门表才有部门名——查「员工姓名 + 部门名称」必须把两张表**按关联字段拼起来**。

| 连接类型 | 写法 | 结果集 |
|-|-|-|
| 内连接 | `from A [inner] join B on A.x = B.x`（或隐式 `from A, B where A.x = B.x`） | 只保留**两边都匹配**的行 |
| 左外连接 | `from A left join B on ...` | A 表**全保留**，B 匹配不上的补 NULL |
| 右外连接 | `from A right join B on ...` | B 表**全保留**，A 匹配不上的补 NULL |

判断口诀：**以谁为主表，谁的数据就一条不能少**。「查询所有员工（含没分部门的）」必须用左连接；换成内连接，没分部门的员工就凭空消失了。

### 2. 子查询：查询套查询

按子查询的返回形态分四类：

| 类型 | 返回 | 典型用法 |
|-|-|-|
| **标量子查询** | 单行单列一个值 | `where salary > (select salary from ... where name = '俞莲舟')` |
| **列子查询** | 一列多个值 | 配 `in`：`where dept_id in (select id from ...)` |
| **行子查询** | 一行多个字段 | `where (entrydate, job) = (select entrydate, job from ...)` |
| **表子查询** | 多行多列（临时表） | 放在 from 后当表用，或配 in |

### 3. 非等值连接：区间匹配（薪资等级）

`tb_salgrade` 用 `losal~hisal` 区间定义等级，连接条件不是等值而是**范围**：

```sql
select e.name, e.salary, s.grade
from tb_emp e
join tb_salgrade s on e.salary between s.losal and s.hisal;
```

JOIN 的 `on` 后面可以写任何条件——打破了「连接只能等值」的直觉。

### 4. 分组统计 + 连接的组合

「每个部门的员工人数」：`from tb_dept left join tb_emp ... group by 部门`——**先连接铺开明细，再分组聚合**。注意 count 要用 `count(emp.id)` 而非 `count(*)`，否则没有员工的部门会被错误计为 1（left join 补的那行 NULL 也会被 `count(*)` 算进去）。

## 二、技术点之间的联系

```
单表 SQL（SQL 基础章）
  │  加一张关联表 ──▶ 连接查询（内/左/右）
  │  条件里嵌查询 ──▶ 子查询（标量/列/行/表）
  │  两者叠加 ──▶ 分组统计 + 连接（每部门人数）
  │  条件变区间 ──▶ 非等值连接（薪资等级）
  ▼
三层架构模块开发
  = 单表 CRUD 套路的再次复练（部门 → 角色 → 课程/活动）
  = 套路已固化，速度即目标
```

三条关键联系：

1. **连接与子查询常常殊途同归**：「查教研部员工」既能 `join + where d.name = '教研部'`，也能 `where dept_id = (子查询)`——多数场景两者等价，但连接一次能带出两张表的字段，子查询只能过滤。选型看「结果要不要另一张表的列」。
2. **多表查询是递进的**：连接基础 → 子查询四形态 → 连接 + 聚合 → 区间连接 + 相关子查询，这条路径本身就是学习顺序。
3. **同一套三层架构换表复用**：模块开发的价值在于形成「看到新模块就知道怎么搭」的肌肉记忆。

## 三、方法与实现要点

### 1. 写连接查询的三步

① 确定要哪些列、它们分别来自哪张表；② 找表之间的关联字段（外键 `dept_id` → `id`）；③ 根据「主表数据能不能少」选 join 类型。写完先用 `select *` 跑通连接，再裁剪列。

### 2. 子查询的「由内向外」读写法

先单独把子查询写出来跑通（如 `select id from tb_dept where name = '教研部'` 确认返回 2），再嵌进外层——这是最省时的调试姿势。

### 3. 表别名的规范

一旦连接，**所有字段都带上别名前缀**（`e.name`、`d.name`）：两表同名字段（都有 name、id）不加前缀直接报歧义错误。别名用表名的语义缩写（emp→e、dept→d），别用 a/b/c。

### 4. 「最值对应的人」类需求的两步法

需求形如「低于本部门平均工资」「每部门最高薪的人」：

- **低于本部门平均**：相关子查询 `where salary < (select avg(salary) from tb_emp e2 where e2.dept_id = e1.dept_id)`，子查询引用外层表字段；
- **每部门最高薪的人**：先按部门分组求最高薪，再连接回员工表按「部门 + 薪资」双条件匹配出人名。

## 四、重点难点分析

### 重点 1：内连接 vs 外连接的选择

判断标准一句话：**结果里要不要保留「没有匹配项」的行**？要保留员工就用员工表 left join；要保留部门就用部门表 left join（或员工表 right join，同一件事换个主语）。

### 重点 2：四种子查询的形态判断

看子查询返回几行几列：一个值 → 标量（配 >、<、=）；一列 → 列子查询（配 in）；一行多列 → 行子查询（配 `(a,b) = (…)`）；多行多列 → 表子查询（放 from 后加别名当临时表）。

### 难点 1：隐式与显式内连接的等价性

`from A, B where A.x = B.x` 与 `from A join B on A.x = B.x` 结果相同，但显式写法把「连接条件」与「过滤条件」分开（on vs where），可读性更好、也是企业规范。

### 难点 2：分组统计时 NULL 组的处理

left join + group by 统计部门人数时：没有员工的部门，emp.id 全为 NULL，`count(emp.id)` = 0 是期望结果；而 `count(*)` = 1 是错误的。聚合函数对 NULL 的敏感度：`count(字段)` 跳过 NULL，`count(*)` 数行不数空。

### 易错点清单

- 连接忘写 on 条件——笛卡尔积（17 员工 × 5 部门 = 85 行）；
- 多表同名字段不加别名——Column 'name' in field list is ambiguous；
- 子查询返回多行却用 `=`——Subquery returns more than 1 row，应改 `in`；
- `group by` 的 select 里混入未分组字段——ONLY_FULL_GROUP_BY 模式直接报错；
- 区间连接忘写 `between` 的边界包含关系——薪资 3000 整落在哪级取决于区间设计（losal 从 3001 起则 3000 属上一级）。

## 五、示例

### 示例 1：内连接的两个写法

```sql
-- 隐式内连接：连接条件写在 where
select e.name, d.name
from tb_emp e, tb_dept d
where e.dept_id = d.id;

-- 显式内连接：连接条件写在 on（推荐）
select e.name, d.name
from tb_emp e
join tb_dept d on e.dept_id = d.id;
```

### 示例 2：左/右外连接的差异

```sql
-- 所有员工都保留（含未分配部门的陈友谅），部门名为 NULL
select e.name, d.name
from tb_emp e
left join tb_dept d on e.dept_id = d.id;

-- 所有部门都保留（含没有员工的人事部），员工名为 NULL
select d.name, e.name
from tb_emp e
right join tb_dept d on e.dept_id = d.id;
```

要点：两张 SQL 的表顺序与 join 方向不同，主表也就不同——「谁一条不能少，谁当主表」。

### 示例 3：标量子查询

```sql
-- 教研部的所有员工（子查询只返回一个部门 id）
select * from tb_emp
where dept_id = (select id from tb_dept where name = '教研部');

-- 比方东白入职更晚的员工
select * from tb_emp
where entrydate > (select entrydate from tb_emp where name = '方东白');
```

### 示例 4：列子查询

```sql
-- 教研部和咨询部的所有员工（子查询返回一列多个值，配 in）
select * from tb_emp
where dept_id in (select id from tb_dept where name in ('教研部', '咨询部'));
```

### 示例 5：行子查询

```sql
-- 与韦一笑入职日期和职位都相同的人
select * from tb_emp
where (entrydate, job) = (select entrydate, job from tb_emp where name = '韦一笑')
  and name != '韦一笑';
```

要点：行子查询必须两个字段**同时匹配**，用 `(a, b) = (子查询)` 元组写法；拆成两个 `=` 且各写一个子查询也等价，但写法冗长。

### 示例 6：表子查询

```sql
-- 2006 年之后入职的员工，及其部门信息
select e.*, d.name as dept_name
from (select * from tb_emp where entrydate > '2006-01-01') e
left join tb_dept d on e.dept_id = d.id;
```

要点：子查询放在 from 后必须起别名，当作一张临时表使用。

### 示例 7：连接 + 分组统计

```sql
-- 所有部门及员工人数（没有员工的部门计 0）
select d.id, d.name, count(e.id) as emp_count
from tb_dept d
left join tb_emp e on e.dept_id = d.id
group by d.id, d.name;

-- 只统计有员工的部门
select d.id, d.name, count(e.id) as emp_count
from tb_dept d
join tb_emp e on e.dept_id = d.id
group by d.id, d.name;
```

要点：`count(e.id)` 才是人数，`count(*)` 会把 left join 补出的 NULL 行也算成 1。

### 示例 8：非等值连接（薪资等级）

```sql
-- 所有员工的姓名、工资及等级
select e.name, e.salary, s.grade
from tb_emp e
join tb_salgrade s on e.salary between s.losal and s.hisal;

-- 每个等级有多少员工
select s.grade, count(e.id)
from tb_salgrade s
left join tb_emp e on e.salary between s.losal and s.hisal
group by s.grade;
```

要点：`on` 的条件可以是范围判断；用 `left join` 从等级表出发，才能保留「没有人落在该区间」的等级（计 0）。

### 示例 9：相关子查询（低于本部门平均工资）

```sql
select e1.name, e1.salary, e1.dept_id
from tb_emp e1
where e1.salary < (
    select avg(e2.salary)
    from tb_emp e2
    where e2.dept_id = e1.dept_id       -- 引用外层表的字段 → 相关子查询
);
```

要点：相关子查询对外层每一行都执行一次，逻辑直观但数据量大时要留意性能。

### 示例 10：每部门最高薪水的人（两步法）

```sql
-- 第一步：每个部门的最高薪资
select dept_id, max(salary) as max_salary
from tb_emp
where dept_id is not null
group by dept_id;

-- 第二步：连接回员工表，按「部门 + 薪资」双条件匹配出人名
select e.name, e.dept_id, e.salary
from tb_emp e
join (
    select dept_id, max(salary) as max_salary
    from tb_emp
    where dept_id is not null
    group by dept_id
) m on e.dept_id = m.dept_id and e.salary = m.max_salary;
```

要点：不能写成 `select name, max(salary) ... group by dept_id`——name 不是分组字段，语义上不确定「取谁」。

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / 项目实战：qk 全栈项目 / 07 多表查询技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Cu1twvYtmiNAXikaVORcOD4CnDd)
