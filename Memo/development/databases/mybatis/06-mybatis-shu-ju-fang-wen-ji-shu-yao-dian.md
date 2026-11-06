---
type: memo
date: 2026-11-06
aliases:
  - "06 MyBatis 数据访问技术要点"
---

# 06 MyBatis 数据访问技术要点

> 本章围绕课程（course）与活动（activity）两条业务线，梳理 **Mapper 层开发套路：注解 SQL、XML 动态 SQL、参数传递规范、单元测试驱动验证**。

## 一、技术概念说明

### 1. MyBatis：Java 对象与 SQL 之间的映射器

MyBatis 把「写 SQL、传参数、把结果集封装成 Java 对象」三件事自动化。开发一个 Mapper 的四件套：

1. **实体类**（`Course`/`Activity`）：属性与表字段一一对应，驼峰命名自动映射下划线列名（`createTime` ↔ `create_time`）；
2. **Mapper 接口**：只声明方法，不写实现，`@Mapper` 注解交给 Spring 扫描生成代理对象；
3. **SQL 配置**：注解方式或 XML 方式；
4. **单元测试**：`@SpringBootTest` + `@Autowired` 注入 Mapper 直接调，验证 SQL 正确性。

### 2. 两种 SQL 配置方式的分工

| 方式 | 写法 | 适用场景 |
|-|-|-|
| **注解配置** | `@Insert/@Delete/@Update/@Select("SQL")` 写在方法上 | SQL 简短、条件固定的增删改查 |
| **XML 配置** | `XxxMapper.xml` 中写 `<select>` 等标签，`namespace` 绑定接口全类名 | 条件可变的**动态 SQL**、复杂 SQL |

分工原则：四个单条操作（增/删/改/按 ID 查）用注解，条件组合查询用 XML。

### 3. 参数占位与传参规范

- **`#{属性名}`**：预编译占位符（JDBC `?`），防 SQL 注入，自动处理类型；
- **`@Param("course")`**：方法有多个参数时必须用它给每个参数起名字，XML/注解里才能以 `#{course.name}`、`#{id}` 的方式引用；
- 单个简单参数可直接 `#{任意名}`，但**养成与形参一致的习惯**。

### 4. 动态 SQL：`<where>` + `<if>`

```xml
<select id="selectByCondition" resultType="com.itheima.entity.Course">
    select * from course
    <where>
        <if test="name != null and name != ''">
            and name LIKE CONCAT('%', #{name}, '%')
        </if>
        <if test="subject != null">
            and subject = #{subject}
        </if>
    </where>
</select>
```

- `<where>` 自动补 `WHERE` 关键字，并**智能去掉多余的 and/or**；
- `<if test="...">` 按条件决定是否拼接该片段——前端没传的条件就不出现在 SQL 里；
- 模糊查询标准写法：`LIKE CONCAT('%', #{name}, '%')`，**不能**直接 `LIKE '%#{name}%'`（`#{}` 在字符串字面量内不解析）。

## 二、技术点之间的联系

```
数据库表（course / activity）
  │  DDL 定义字段与注释 ── 实体类镜像映射
  ▼
实体类 Course/Activity ◀── 结果集自动装箱 ── Mapper 接口
  ▲                                        │
  │        ┌─ 注解 SQL：固定四操作（增删改查单条）
  └────────┤
           └─ XML SQL：动态条件查询（<where><if>）
                       │
Spring 容器：@Mapper 扫描生成代理 → @Autowired 注入 → @SpringBootTest 单测验证
```

三条关键联系：

1. **实体类是表与代码之间的契约**：表加字段 → 实体类加属性 → SQL 的 `#{}` 同步改，三者必须保持一致；驼峰映射规则让 `update_time` 与 `updateTime` 自动对上。
2. **XML 与接口靠「全类名 + 方法名」精确绑定**：`namespace="com.itheima.mapper.CourseMapper"` + `<select id="selectByCondition">`，写错一个字母就是 `BindingException`。
3. **动态 SQL 是「查询条件可选」的技术答案**：搜索表单里名称/学科/人群可填可不填，一条动态 SQL 覆盖全部 2³ 种组合——否则得写 8 个方法。

## 三、方法与实现要点

### 1. Mapper 开发的标准工序

**建表插数据 → 写实体类 → 写 Mapper 接口（先注解后 XML）→ 写 XML（如需动态 SQL）→ 每个方法配一个 `@Test` 跑通**。每写一个 SQL 立刻验证，而不是攒到最后联调。

### 2. 写动态 SQL 的检查清单

- 字符串条件：判 `!= null and != ''`；数字条件：只判 `!= null`（0 可能是合法值）；
- 所有条件都包在 `<where>` 里，不要在 `select * from t` 后手写 `WHERE 1=1` 再拼 and；
- 模糊查询用 `CONCAT('%', #{x}, '%')`。

### 3. 更新操作的返回值意识

增删改方法返回 `int`（受影响行数），测试里打印出来——返回 0 说明 SQL 执行了但没命中数据（如 ID 不存在），这是区分「代码错」与「数据错」的第一线索。

### 4. 代码规范

注解 SQL 字符串过长时用 `+` 分行拼接；方法名见名知义（`selectByCondition` 比 `selectCourse1` 更专业）；**更新必须用 `@Update`**，误写成 `@Insert` 虽然可能跑通，但语义错、后续维护必踩坑。

## 四、重点难点分析

### 重点 1：`#{}` 与 `${}` 的区别

`#{}` 是预编译参数占位（安全、防注入、自动加引号）；`${}` 是字符串原样拼接（有注入风险，仅用于表名/列名/排序字段等无法用 `#{}` 的场景）。**默认一律 `#{}`**。

### 重点 2：动态 SQL 的标签族

`<where>`、`<if>` 之外还有 `<choose>/<when>/<otherwise>`（多选一）、`<set>`（更新时智能去逗号）、`<foreach>`（批量 in 查询/批量插入）。掌握 `where + if` 后其余同理。

### 难点 1：多参数传递的 `@Param` 机制

没有 `@Param` 时，MyBatis 只能按 `arg0/param1` 这类默认名取参，可读性差且容易踩坑；加了 `@Param("course")` 后才能在 SQL 里写 `#{course.subject}` 这种「对象.属性」路径。**凡是两个以上参数，一律 `@Param`**。

### 难点 2：null 值字段的插入策略

活动表中 `discount` 与 `voucher` 互斥（折扣活动没有代金券），插入时传 `null`——MyBatis 会把 null 写入数据库；如果业务要求「不传就不更新该字段」，需要 `<if test="discount != null">` 动态拼接 set 子句（`<set>` 标签场景）。

### 易错点清单

- XML 的 `namespace` 写成接口的简单类名或包路径拼错——启动即 BindingException；
- `LIKE '%#{name}%'` 不生效——`#{}` 必须在引号外，用 CONCAT 拼；
- `parameterType` 与多 `@Param` 参数混用概念——多参数时 parameterType 可省略；
- 测试类忘加 `@SpringBootTest`——`@Autowired` 注入失败报 NullPointerException。

## 五、示例

### 示例 1：课程表结构

```sql
create table course (
    id          int unsigned primary key auto_increment comment '课程id, 主键',
    subject     tinyint unsigned not null comment '学科 1:Java 2:Python 3:鸿蒙 4:大数据 5:嵌入式 6:测试 7:运维',
    name        varchar(20) not null comment '课程名称',
    price       int unsigned not null comment '课程价格（元）',
    target      tinyint unsigned not null comment '适用人群 1:小白 2:初级程序员 3:中级程序员',
    description varchar(100) comment '课程介绍',
    create_time datetime not null comment '创建时间',
    update_time datetime not null comment '修改时间'
) comment '课程表';
```

### 示例 2：注解 SQL 完成单条增删改查

```java
@Mapper
public interface CourseMapper {

    @Insert("insert into course(subject, name, price, target, description, create_time, update_time) " +
            "values(#{subject}, #{name}, #{price}, #{target}, #{description}, #{createTime}, #{updateTime})")
    int addCourse(Course course);

    @Delete("delete from course where id = #{id}")
    int deleteCourse(int id);

    @Update("update course set subject = #{course.subject}, name = #{course.name}, price = #{course.price}, " +
            "target = #{course.target}, description = #{course.description}, " +
            "update_time = #{course.updateTime} where id = #{id}")
    int updateCourse(@Param("course") Course course, @Param("id") int id);

    @Select("select * from course where id = #{id}")
    Course selectCourseById(int id);

    // 条件组合查询 → 交给 XML
    List<Course> selectByCondition(@Param("name") String name,
                                   @Param("subject") Integer subject,
                                   @Param("target") Integer target);
}
```

要点：`updateCourse` 有两个参数，必须两个都加 `@Param`；SQL 里以 `#{course.xxx}` / `#{id}` 引用。

### 示例 3：XML 动态条件查询

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.mapper.CourseMapper">

    <select id="selectByCondition" resultType="com.itheima.entity.Course">
        select * from course
        <where>
            <if test="name != null and name != ''">
                and name LIKE CONCAT('%', #{name}, '%')
            </if>
            <if test="subject != null">
                and subject = #{subject}
            </if>
            <if test="target != null">
                and target = #{target}
            </if>
        </where>
    </select>
</mapper>
```

三个条件全传 → 三个 and 都拼上；只传一个 → 只拼一个，`<where>` 还会自动去掉开头多余的 `and`。

### 示例 4：活动表与互斥字段

```sql
create table activity (
  id          int unsigned primary key auto_increment comment 'id, 主键',
  channel     tinyint unsigned not null comment '渠道 1:线上活动 2:推广介绍',
  name        varchar(20) not null comment '活动名称',
  start_time  datetime not null comment '开始时间',
  end_time    datetime not null comment '结束时间',
  description varchar(100) not null comment '活动简介',
  type        tinyint unsigned not null comment '类型 1:课程折扣 2:代金券',
  discount    double(2,1) comment '课程折扣',
  voucher     int unsigned comment '代金券金额（元）',
  create_time datetime not null comment '创建时间',
  update_time datetime not null comment '修改时间'
) comment '活动表';
```

要点：`type` 决定 `discount` / `voucher` 谁有值——折扣活动 voucher 为 NULL，代金券活动 discount 为 NULL，这是「同构数据 + 类型字段区分」的典型建模。

### 示例 5：单元测试驱动验证每个方法

```java
@SpringBootTest
public class CourseMapperTest {
    @Autowired
    private CourseMapper courseMapper;

    @Test
    void addCourseTest() {
        Course course = new Course(null, 1, "SpringBoot", 100, 1, "SpringBoot", null, null);
        System.out.println("影响条数：" + courseMapper.addCourse(course));
    }

    @Test
    void deleteCourseTest() {
        System.out.println("影响条数：" + courseMapper.deleteCourse(15));
    }

    @Test
    void updateCourseTest() {
        Course course = new Course(null, 1, "SpringBoot", 120, 1, "SpringBoot 进阶", null, null);
        System.out.println("影响条数：" + courseMapper.updateCourse(course, 8));
    }

    @Test
    void selectByConditionTest() {
        courseMapper.selectByCondition("Java", 1, 1).forEach(System.out::println);
    }
}
```

要点：`@SpringBootTest` 起完整容器才能注入 Mapper；每个方法独立可跑，返回 0 就是「SQL 没问题但数据不匹配」的信号。

---
> 来源：飞书技术知识库（脱敏版） · 02 数据库与持久层 / MyBatis / 06 MyBatis 数据访问技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Sbs7wGGmMi6CMgkz37kcDdDGnad)
