---
type: memo
date: 2026-11-06
aliases:
  - "03-san-ceng-jia-gou-ye-wu-mo-kuai-kai-fa-ji-shu-yao-dian"
  - "03 三层架构业务模块开发技术要点"
---

# 03 三层架构业务模块开发技术要点

> 本章以部门 / 角色管理为载体，梳理 **Controller-Service-Mapper 三层架构的开发套路、统一响应封装、分页查询闭环、按接口文档交付**。

## 一、技术概念说明

### 1. 三层架构：请求的标准旅程

| 层 | 职责 | 关键注解 |
|-|-|-|
| **Controller**（表现层） | 接收请求、封装参数、调用 Service、返回结果 | `@RestController`、`@RequestMapping("/roles")` |
| **Service**（业务层） | 参数校验与补全、业务规则、组织数据、调用 Mapper | `@Service` |
| **Mapper**（持久层） | 与数据库交互，只做 SQL | `@Mapper` |

口诀：**Controller 管收发、Service 管逻辑、Mapper 管数据**。层与层之间只通过接口调用，上层不越级碰数据库。

### 2. 统一响应封装：Result 与 PageData

`Result` 是所有接口的统一返回壳（code / msg / data），`PageData<T>` 是分页数据的标准形态（total / rows）。前端永远面对同一种 JSON 结构，无论成功失败——这是团队协作的契约。

### 3. DTO：数据传输对象

`RoleDTO` 把查询条件（name、label）与分页参数（page、pageSize）打包成一个对象，Controller 直接整体接收前端参数。实体类 `Role` 映射表结构，DTO 映射请求参数——**两者分离**，互不污染。

### 4. 分页查询的实现要点

- 前端传「第几页 page、每页几条 pageSize」；
- SQL 的 `LIMIT` 要的是**起始索引**：`(page - 1) * pageSize`——换算放在 Service 层完成（业务逻辑归 Service 的典型例子）；
- 分页接口要跑**两条 SQL**：`select * ... limit ?,?` 取当前页数据 + `select count(*)` 取总条数，两条的**查询条件必须完全一致**。

### 5. 接口文档驱动开发

请求路径（`/roles/list`）、请求方式（GET）、参数格式、响应字段（code/msg/data）逐项对齐，开发完用 Apifox 自测——这就是企业里前后端联调的标准流程。

## 二、技术点之间的联系

```
前端请求（GET /roles?name=x&page=1&pageSize=10）
  │
  ▼
Controller：RoleDTO 整体收参 ──调用──▶ RoleService（接口）
  ▲                                     │
  │                                     ▼
Result.ok(pd) ◀──返回── ServiceImpl：补默认分页、换算起始索引
（统一壳）                              │
                                        ▼
                              RoleMapper：两条动态SQL
                              （条件列表 + 条件总数）
                                        │
                                        ▼
                              MySQL role 表 ←── 实体类 Role 映射
```

三条关键联系：

1. **「换一个表换一种业务，套路完全不变」：部门管理与角色管理的代码结构逐行对应——这正是分层架构的价值：业务可变，骨架不变**。
2. **DTO 串起三层**：Controller 接收 DTO → Service 读取并补全 DTO → Mapper 把 DTO 字段填入动态 SQL。DTO 是三层的公共语言。
3. **三层架构是 MyBatis 单表 CRUD 的自然延伸**：动态 SQL + Mapper 开发在这里被装进三层骨架，成为完整可交付的接口。

## 三、方法与实现要点

### 1. 开发一个模块的标准顺序（自底向上最稳）

**建表 → 实体类 → Mapper 接口 + XML（先跑通 SQL）→ Service 接口 + 实现 → Controller → Apifox 按文档验收**。自底向上写，每层都能独立测试；自顶向下写容易做到最后才发现 SQL 不通。

### 2. 分页参数的两个防御

- page/pageSize 为 null 时补默认值（1 和 10），防止前端漏传导致 SQL 报错；
- 页码换算 `(page-1)*pageSize` 在 Service 里完成且**只换算一次**，建议用单独变量承载，避免覆盖 DTO 里「前端传的页码」造成排查困难。

### 3. 条件查询与总数的「双胞胎」原则

`findByPageAndCondition` 与 `findByTotalAndCondition` 的 `<where>` 必须逐字一致——不一致就会出现「总数 100 条但只查到 10 条」的对不上账。技巧：先写列表 SQL，再复制改成 count。

### 4. 接口文档验收清单

路径、方法、参数名、响应结构四项逐项核对；特别留意「查询所有角色 `/roles/list`」这类无参接口与分页接口的路径不要冲突。

## 四、重点难点分析

### 重点 1：三层职责边界

Controller 不写 if 业务判断、Service 不碰 HttpServletRequest、Mapper 不写业务规则。「补默认分页、换算起始索引」放在 Service 而不是 Controller，正是职责边界的体现——**与 HTTP 无关的逻辑都该下沉到 Service**。

### 重点 2：分页的完整闭环

参数（page/pageSize）→ 换算起始索引 → 两条 SQL（数据 + 总数）→ PageData 封装 → Result 统一返回。缺任何一环，前端分页组件就渲染不出来。

### 难点 1：动态 SQL 与 LIMIT 的拼接顺序

`<where>` 动态条件拼完后，`limit #{page},#{pageSize}` 写在 `<where>` 标签**之外、语句最末**——顺序错了 SQL 直接语法报错。LIMIT 本身不参与条件判断，永远生效。

### 难点 2：MyBatis 返回 `int` 与 `Integer` 的总数查询

`select count(*)` 返回单列单值，Mapper 用 `Integer findByTotalAndCondition(...)` 接收；count 永远不会是 null（没数据也是 0），不用判空，但 `resultType="int"` 与包装类的映射要写对。

### 易错点清单

- `@RestController` 与 `@ResponseBody` 重复标注——`@RestController` 已包含后者，取其一即可；
- Service 接口拼写错误（如 `RoleSerive`）——编译能过但给协作埋雷，命名要规范；
- 分页换算后把 `page` 字段直接覆盖回 DTO——日志排查时分不清「前端传的页码」与「换算后的索引」；
- 路径冲突：`@RequestMapping("/roles")` 的类级路径与方法级路径叠加规则记错，导致 404。

## 五、示例

### 示例 1：角色表结构

```sql
create table role (
    id          int unsigned not null primary key auto_increment comment '角色id, 主键',
    name        varchar(10) not null unique comment '角色名称',
    label       varchar(20) not null unique comment '角色标识',
    remark      varchar(50) comment '备注',
    create_time datetime not null comment '创建时间',
    update_time datetime not null comment '修改时间'
) comment '角色表';
```

要点：`name` 与 `label` 都加 `unique`，把「角色名不可重复」的业务规则固化到数据库层。

### 示例 2：统一响应壳与分页容器

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Result {
    private Integer code;      // 1 成功 / 0 失败
    private String msg;
    private Object data;

    public static Result ok(Object data) { return new Result(1, "success", data); }
    public static Result ok(String msg, Object data) { return new Result(1, msg, data); }
    public static Result error(String msg) { return new Result(0, msg, null); }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public class PageData<T> {
    private Integer total;     // 总条数
    private List<T> rows;      // 当前页数据
}
```

### 示例 3：DTO 接收查询条件与分页参数

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class RoleDTO {
    private String name;       // 角色名称（模糊）
    private String label;      // 角色标识（精确）
    private Integer page;      // 当前页码
    private Integer pageSize;  // 每页条数
}
```

要点：DTO 与实体类分开——前端可能传 `page`，而数据库表里没有这个字段。

### 示例 4：Controller 层（只做收发）

```java
@RestController
@RequestMapping("/roles")
public class RoleController {

    @Autowired
    private RoleService roleService;

    @GetMapping
    public Result findByPageAndCondition(RoleDTO roleDTO) {
        PageData<Role> pd = roleService.findByPageAndCondition(roleDTO);
        return pd == null ? Result.error("查询失败") : Result.ok(pd);
    }

    @GetMapping("/list")
    public Result findAll() {
        return Result.ok(roleService.findAll());
    }
}
```

### 示例 5：Service 层（补默认值 + 换算 + 组装）

```java
@Service
public class RoleServiceImpl implements RoleService {
    @Autowired
    private RoleMapper roleMapper;

    @Override
    public PageData<Role> findByPageAndCondition(RoleDTO dto) {
        int page = dto.getPage() == null ? 1 : dto.getPage();
        int pageSize = dto.getPageSize() == null ? 10 : dto.getPageSize();

        RoleDTO query = new RoleDTO();
        query.setName(dto.getName());
        query.setLabel(dto.getLabel());
        query.setPage((page - 1) * pageSize);   // 起始索引，只换算一次
        query.setPageSize(pageSize);

        List<Role> rows = roleMapper.findByPageAndCondition(query);
        Integer total = roleMapper.findByTotalAndCondition(query);
        return new PageData<>(total, rows);
    }
}
```

### 示例 6：Mapper 与 XML（两条条件一致的 SQL）

```java
@Mapper
public interface RoleMapper {
    List<Role> findByPageAndCondition(RoleDTO dto);
    Integer findByTotalAndCondition(RoleDTO dto);
    List<Role> findAll();
}
```

```xml
<mapper namespace="cn.itcast.mapper.RoleMapper">

    <select id="findByPageAndCondition" resultType="cn.itcast.entities.Role">
        select * from role
        <where>
            <if test="name != null and name != ''">
                and name like concat('%', #{name}, '%')
            </if>
            <if test="label != null and label != ''">
                and label = #{label}
            </if>
        </where>
        limit #{page}, #{pageSize}
    </select>

    <select id="findByTotalAndCondition" resultType="int">
        select count(*) from role
        <where>
            <if test="name != null and name != ''">
                and name like concat('%', #{name}, '%')
            </if>
            <if test="label != null and label != ''">
                and label = #{label}
            </if>
        </where>
    </select>

    <select id="findAll" resultType="cn.itcast.entities.Role">
        select * from role order by update_time desc
    </select>
</mapper>
```

要点：`limit` 在 `<where>` 之外、语句末尾；两条 SQL 的 `<where>` 块必须逐字一致，否则总数与列表对不上。

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / Spring 与 SpringBoot / 03 三层架构业务模块开发技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/YUOUwUS9Ji7v46kwVyBcOrewn5g)
