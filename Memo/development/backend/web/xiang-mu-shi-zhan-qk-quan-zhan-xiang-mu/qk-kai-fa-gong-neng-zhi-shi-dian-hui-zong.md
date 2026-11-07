---
type: memo
date: 2026-11-06
aliases:
  - "qk开发功能知识点汇总"
---
# qk开发功能知识点汇总

<whiteboard token="C1U4wg0fMhn8axbGagTcIkuGnoe"></whiteboard>

##  **项目初始化**

**核心：创建父工程（聚合工程）**

**目的**：

```Java
聚合管理：将多个相关的子模块（如web、service、dao）组织在一起，便于统一构建和依赖管理。
统一依赖：在父工程中集中定义项目所需的第三方库（依赖）及其版本，避免各子模块版本冲突。
锁定版本：通过 dependencyManagement锁定依赖版本，子模块引入依赖时无需指定版本号，确保整个项目版本一致。
```

**关键配置（父工程 `pom.xml`）**：

```XML
<!-- 1. 打包方式必须为 pom -->
<packaging>pom</packaging>

<!-- 2. 定义子模块列表 -->
<modules>
    <module>module-a</module>
    <module>module-b</module>
</modules>

<!-- 3. 统一管理依赖版本（关键） -->
<properties>
    <spring-boot.version>2.7.18</spring-boot.version>
    <mysql.version>8.0.33</mysql.version>
</properties>

<!-- 4. 在 dependencyManagement 中声明依赖及版本 -->
<dependencyManagement>
    <dependencies>
        <!-- Spring Boot 依赖管理 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- 其他通用依赖，如MySQL驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**注意**：父工程中的 `dependencyManagement`仅**声明**依赖和版本，子模块需要显式引入才能生效。

---

##  **开发功能分层总结**

### **Controller 层**

**职责**：接收请求参数，调用Service处理业务，返回响应。

**关键技术点**：

**参数接收**：

```Java
简单参数：使用 @RequestParam 接收URL或表单参数。
JSON对象：使用 @RequestBody 接收请求体中的JSON数据，并自动绑定到Java对象。
路径变量：使用 @PathVariable 接收RESTful风格的URL路径参数。
```

**参数校验**：

```Java
使用JSR-303规范注解（如 @NotNull，@Size ，@Email）在接收参数的实体类字段上进行标注。
在Controller方法参数前添加 @Valid 或 @Validated 注解以触发校验。
```

**全局异常处理**：

```Java
使用 @RestControllerAdvice 注解定义一个全局异常处理类。
在类内的方法上使用 @ExceptionHandler 注解处理特定异常，并返回统一的错误信息格式。
```

**响应封装**：统一使用 `Result`或 `CommonResponse`等自定义类包装返回数据，包含`code`、`msg`、`data`字段，便于前端处理。

### **Service 层**

**职责**：封装核心业务逻辑，校验补全数据,协调多个Dao操作,。

**关键技术点**：

**业务逻辑**：实现具体的业务规则和流程。

**事务管理**：

```Java
在类或方法上添加 @Transactional注解，声明事务边界。
传播行为：例如 @Transactional(propagation = Propagation.REQUIRED)，定义方法如何参与或创建事务。
异常回滚：默认在遇到 RuntimeException时回滚，可通过 rollbackFor属性自定义。
```

**分页逻辑**：

```Java
接收Controller传递的页码（page）和每页条数（pageSize）。
调用Dao层方法前，优先使用PageHelper等分页插件（在方法调用前执行 PageHelper.startPage(page, pageSize)），插件会自动改写后续的第一个SQL，添加LIMIT语句。
将Mapper(Dao)查询到的 List结果，包装成 PageInfo对象返回给Controller，其中包含总记录数、总页数等分页信息。
```

### **Dao / Mapper 层**

**职责**：负责与数据库进行交互，执行SQL。

**关键技术点**：

#### **数据库交互**：

```Java
使用MyBatis框架，通过 Mapper接口+XML映射文件 或 注解 的方式定义SQL。
XML方式（推荐复杂SQL）：在XML中编写SQL，namespace对应接口全限定名，id对应方法名。
注解方式（适合简单SQL）：直接在接口方法上使用 @Select， @Insert等注解。
```

#### **动态SQL**：

```Java
在MyBatis的XML映射文件中，使用 <if>， <where>， <foreach>等标签，根据参数动态拼接SQL，避免手动拼接字符串的复杂和SQL注入风险。
例如：<if test="name != null and name != ''"> AND name = #{name} </if>
```

#### **条件查询**：

```Java
方法参数通常使用一个“查询条件封装对象”（如XxxQuery），而非多个散参数。
在XML中使用动态SQL标签，根据该对象中字段是否为 null 或空，来动态添加 WHERE 条件。
```

#### **关联查询**：

```Java
处理多表关联时，在XML中编写关联SQL（JOIN）。
使用 <resultMap>定义复杂的映射规则，将查询结果正确映射到实体类及其关联对象的属性上。
```

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / 项目实战：qk 全栈项目 / qk开发功能知识点汇总 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/EgiQw0tMkibJOmkCoN3crQMxnuc)
