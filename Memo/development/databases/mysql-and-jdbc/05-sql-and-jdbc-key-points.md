---
type: memo
date: 2026-11-06
aliases:
  - "05-sql-yu-jdbc-ji-shu-yao-dian"
  - "05 SQL 与 JDBC 技术要点"
---

# 05 SQL 与 JDBC 技术要点

> 本章三部分环环相扣：**SQL 手写能力（DML + DQL）→ 从页面原型反推表结构 → 原生 JDBC 六步 CRUD**，对应「会用 SQL、会设计表、会用 Java 连库」三项基本功。

## 一、技术概念说明

### 1. SQL 的两大操作族

| 族 | 动词 | 典型用法 |
|-|-|-|
| **DML**（数据操纵） | `INSERT` / `UPDATE` / `DELETE` | 按条件改（薪资增减、NULL 值修复）、批量插入 |
| **DQL**（数据查询） | `SELECT` | 条件过滤、排序、聚合、分组、模糊、分页 |

DML 返回「受影响行数」；DQL 返回结果集。写 UPDATE/DELETE 的铁律：**先写 WHERE 再写操作**，漏了 WHERE 就是全表灾难。

### 2. 查询的六要素及其书写顺序

```sql
SELECT 字段/聚合函数        -- 5️⃣ 取什么
FROM 表                     -- 1️⃣ 从哪取
WHERE 行过滤条件             -- 2️⃣ 筛行
GROUP BY 分组字段            -- 3️⃣ 分组
-- HAVING 组过滤条件        -- 4️⃣ 筛组
ORDER BY 字段 [DESC]        -- 6️⃣ 排序
LIMIT 起始索引, 条数;        -- 7️⃣ 截断
```

**书写顺序 ≠ 执行顺序**：实际执行是 FROM → WHERE → GROUP BY → SELECT → ORDER BY → LIMIT，这解释了为什么 WHERE 里不能用 SELECT 的别名。

### 3. 高频语法要点

- **模糊查询**：`name like '李%'`（开头匹配）、`name like '__'`（两个下划线 = 恰好两个字）；
- **范围**：`between 5000 and 8000`（含两端）、`entry_date > '2015-01-01'`（日期可直接比较）；
- **NULL 判断**：`job is null` / `is not null`——**不能用 `= null`**；
- **去重计数**：`count(distinct job)`；
- **分组聚合**：`group by gender` 配 `avg(salary)`，select 里只能出现**分组字段 + 聚合函数**；
- **多字段排序**：`order by salary desc, entry_date asc`（前一个相同才看后一个）；
- **LIMIT 分页**：`limit 1,3` = 跳过 1 条取 3 条（第 2\~4 名）。

### 4. 表结构设计：业务字段 + 辅助字段

看页面原型设计表的套路：**列表页每个可见列 → 一个业务字段（课程名、价格、学科…）；搜索框 → 需要建索引的查询字段**；**隐藏的管理需求 → 辅助字段**（`id` 主键自增、`create_time`/`update_time` 审计字段，几乎每表必备）。枚举型状态（学科、适用人群）用 `tinyint` + 注释标明取值含义。

### 5. JDBC：Java 连数据库的原始六步

```
1. 加载驱动   Class.forName("com.mysql.cj.jdbc.Driver")
2. 获取连接   DriverManager.getConnection(url, user, pwd)
3. 创建执行者 connection.createStatement()
4. 执行SQL    executeQuery(DQL) 返回 ResultSet / executeUpdate(DML) 返回行数
5. 解析结果   while(rs.next()) { rs.getInt("id"); rs.getString("name"); }
6. 释放资源   先关结果集 → 再关执行者 → 最后关连接（后开先关）
```

## 二、技术点之间的联系

```
页面原型
  │  列表列 + 搜索框 ──反推──▶ 表结构设计（业务字段 + 辅助字段）
  ▼
数据库表 ──被操作──▶ SQL（DML 改数据 / DQL 查数据）
  ▲                        │
  │                        ▼
  └── JDBC 六步 ──Java 程序执行 SQL、解析结果集
                           │
                           ▼
                  MyBatis（后续章节）= 把 JDBC 六步中
                  2/3/5/6 步的模板代码全部自动化
```

三条关键联系：

1. **表设计是 SQL 的上游**：字段类型选错（如价格用 varchar）、漏了辅助字段，后面所有 SQL 都会别扭——先有好表结构，才有顺手的 SQL。
2. **JDBC 是 MyBatis 的里子**：MyBatis 的 `SqlSession` 封装的就是 Connection + Statement；理解 `executeQuery/executeUpdate` 的分工，就理解 MyBatis 里 `@Select` 与 `@Insert/@Update/@Delete` 的区分。
3. **手写 SQL 是动态 SQL 的词汇表**：WHERE 条件、排序、聚合这些「零件」，在 MyBatis 动态 SQL 中被 `<if>` 按需组装——先能手写，再谈自动生成。

## 三、方法与实现要点

### 1. 写复杂查询的「剥洋葱」法

先写骨架 `select name, entry_date from emp` → 逐个加条件（`entry_date between ...`、`name like '__'`）→ 最后加 `order by`。**每加一层跑一次**，比一次写完整条再调试快得多。

### 2. 排序的方向标注

多字段排序要**逐字段标注方向**：`order by entry_date asc, salary desc`。只写 `order by salary` 默认升序，与「从高到低」的意图相反；写完拿两条样本数据心算验证一遍。

### 3. UPDATE 的百分比计算

「薪资增加 10%」写 `salary = salary * 1.1`（不是 `salary + 10%`）；「降低 10%」是 `salary * 0.9`。基于旧值的新值计算，SET 右侧可以直接引用原列。

### 4. JDBC 资源的关闭顺序

后开的先关（ResultSet → Statement → Connection），真实项目用 try-with-resources 自动关闭；忘记关连接会造成**连接泄漏**，数据库连接数被打满后整个系统挂掉。

## 四、重点难点分析

### 重点 1：GROUP BY 的「选择性约束」

分组查询中 SELECT 后面只能跟**分组字段**和**聚合函数**——`select job, max(salary) from emp group by job` 合法，但 `select name, max(salary) ... group by job` 中 name 是未分组字段（一个组里有多个人，无法确定显示谁），严格模式下直接报错。

### 重点 2：NULL 的三不认

`= null` 查不到、`null + 1 = null`、`count(job)` 不统计 job 为 null 的行（`count(*)` 才统计所有行）。判断空值只能 `is null` / `is not null`。

### 难点 1：LIMIT 起始索引与页码的换算

「薪资排名前三」写 `limit 0,3`（或 `limit 3`）；写 `limit 1,3` 实际取的是第 2\~4 名。公式：`limit (页码-1)*每页条数, 每页条数`——与 MyBatis 分页的换算完全一致，一处学透两处受用。

### 难点 2：LIKE 的通配符语义

`%` 匹配任意长度（含 0）任意字符，`_` 匹配**恰好一个**字符。「姓李」是 `'李%'`，「姓名两个字」是 `'__'`——混淆两者是模糊查询失准的常见原因；中文姓名按字符算，`_` 可以匹配一个汉字。

### 易错点清单

- UPDATE/DELETE 忘写 WHERE——全表更新/删除，生产事故级错误；
- `between A and B` 忘了**含两端**——边界值核对；
- 字符串/日期字面量忘加单引号；
- JDBC URL 中时区参数缺失（`serverTimezone`）导致连接失败或时间字段差 8 小时。

## 五、示例

### 示例 1：员工表建表与字段设计

```sql
create table emp(
  id          int unsigned primary key auto_increment comment '主键',
  name        varchar(10)  not null comment '姓名',
  image       varchar(255)          comment '头像',
  gender      tinyint unsigned not null comment '性别 1:男 2:女',
  job         tinyint unsigned      comment '职位 1:班主任 2:讲师 3:学工主管 4:教研主管 5:咨询师',
  salary      int unsigned not null comment '薪资',
  entry_date  date                  comment '入职日期',
  create_time datetime              comment '创建时间',
  update_time datetime              comment '更新时间'
) comment '员工信息表';
```

要点：`id` 主键自增 + 两个时间审计字段是**几乎每张表都要有的辅助字段**；枚举值写在 `comment` 里，相当于把字典内置到表定义。

### 示例 2：DML 三类操作

```sql
-- 插入：单条
insert into emp(name, gender, salary, job, entry_date)
values ('张明明', 1, 12000, 2, null);

-- 插入：批量
insert into emp(name, gender, salary, job, entry_date) values
  ('王明', 1, 15600, 3, '2020-10-01'),
  ('李桃红', 2, 13600, 1, '2023-09-01');

-- 更新：基于旧值计算（涨薪 10%、降薪 10%）
update emp set salary = salary * 1.1 where entry_date < '2015-01-01';
update emp set salary = salary * 0.9 where job = 5;

-- 更新：NULL 值修复
update emp set job = 1 where job is null;

-- 更新：指定值
update emp set salary = salary + 2000 where id = 5;

-- 删除：务必带条件
delete from emp where salary < 4600;
```

### 示例 3：条件过滤与排序

```sql
-- 女性员工的姓名与职位
select name, job from emp where gender = 2;

-- 薪资超过 15000
select name, job, salary from emp where salary > 15000;

-- 班主任按入职日期倒序
select * from emp where job = 1 order by entry_date desc;

-- 薪资 5000~8000 的咨询师，按薪资降序
select name, salary from emp where job = 5 and salary between 5000 and 8000
order by salary desc;

-- 多字段排序：薪资降序，相同再按入职日期升序
select name, salary from emp order by salary desc, entry_date asc;
```

### 示例 4：模糊查询与组合条件

```sql
-- 姓李
select * from emp where name like '李%';

-- 姓名恰好两个字（两个下划线）
select name, salary, job from emp
where entry_date > '2010-01-01' and gender = 2 and name like '__';
```

### 示例 5：聚合、去重与分组

```sql
-- 有多少种不同职位
select count(distinct job) from emp;

-- 讲师人数（count(具体列) 不统计 NULL）
select count(job) from emp where job = 2;

-- 男女平均薪资
select gender, avg(salary) from emp group by gender;
```

### 示例 6：分页取前 N 名

```sql
-- 薪资前三：从第 0 条开始取 3 条
select name, salary from emp order by salary desc limit 0, 3;

-- 第 2 页、每页 10 条
select * from emp limit 10, 10;
```

### 示例 7：从页面原型反推表结构

页面有课程列表（课程名、学科、价格、适用人群、介绍）与活动列表（活动名、渠道来源、活动明细、起止时间），据此建表：

```sql
create table course(
  id           int unique primary key auto_increment comment '主键id',
  subject      tinyint unsigned not null comment '学科 1.Java 2.Python 3.C++',
  curriculum   tinyint unsigned not null comment '课程内容 1.JavaSE基础 2.JavaSE进阶 3.Java基础班',
  price        int unsigned not null comment '价格',
  target_users tinyint unsigned comment '适用人群 1.零基础 2.中级程序员',
  introduce    varchar(50) comment '课程介绍',
  create_time  date comment '创建时间',
  update_time  datetime comment '更新时间'
);

create table event(
  id         int unique primary key auto_increment comment '主键id',
  name       varchar(10) not null comment '活动名称',
  source     tinyint unsigned not null comment '渠道 1.推广介绍 2.线上活动',
  detail     tinyint unsigned not null comment '明细 1.9.9折 2.8.8折 3.100元券 4.500元券',
  start_time date comment '开始时间',
  over_time  datetime comment '结束时间'
);
```

要点：列表页每个可见列对应一个业务字段；枚举用 `tinyint` 并注释取值；时间字段区分 `date`（只到日）与 `datetime`（到秒）。

### 示例 8：JDBC 六步（查询 / 更新）

```java
@Test
public void query() throws Exception {
    // 1. 加载驱动
    Class.forName("com.mysql.cj.jdbc.Driver");
    // 2. 获取连接
    Connection connection = DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/db142002?serverTimezone=Asia/Shanghai", "root", "123456");
    // 3. 创建执行者
    Statement statement = connection.createStatement();
    // 4. 执行 DQL
    ResultSet rs = statement.executeQuery("select * from employee");
    // 5. 解析结果集：next() 指向下一行，返回 false 表示结束
    while (rs.next()) {
        System.out.println(rs.getInt("id") + "\t" + rs.getString("name") + "\t" + rs.getInt("job"));
    }
    // 6. 释放资源：后开先关
    rs.close();
    statement.close();
    connection.close();
}

@Test
public void update() throws Exception {
    Class.forName("com.mysql.cj.jdbc.Driver");
    try (Connection connection = DriverManager.getConnection(
                 "jdbc:mysql://localhost:3306/db142002?serverTimezone=Asia/Shanghai", "root", "123456");
         Statement statement = connection.createStatement()) {
        int row = statement.executeUpdate("update employee set job = 1 where id = 10");
        System.out.println("影响行数：" + row);       // DML 返回行数，DQL 返回 ResultSet
    }
}
```

要点：`executeQuery` 只用于 DQL，`executeUpdate` 用于 DML；`try-with-resources` 的关闭顺序由 JVM 逆序保证，正好符合「后开先关」。

---
> 来源：飞书技术知识库（脱敏版） · 02 数据库与持久层 / MySQL 与 JDBC / 05 SQL 与 JDBC 技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/F5Thw8dVZiONn3kEooCchv0hnGe)
