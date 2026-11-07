---
type: memo
date: 2026-11-06
aliases:
  - "08-qk-gong-neng-bu-chong-ri-zhi-kuang-jia-quan-ju-yi-chang-chu"
  - "08 qk功能补充_日志框架_全局异常处理"
---
# 08 qk功能补充_日志框架_全局异常处理

## 日志框架-Logback

<callout emoji="🎹">
Logback基于Log4j升级而来，提供了更多的功能和配置选项，性能优于Log4j。
</callout>

### 依赖部署

引入logback的依赖（springboot中无需引入，在springboot中已经传递了此依赖）

```XML
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.11</version>
</dependency>
```

### 使用步骤

```Java
第一步：加入@Slf4j注解
第二步：导入logbac日志配置文件，已经在idea中配置好模板了直接生成即可
第三步：使用log对象，进行日志输出
        1.debug:给框架底层调试用的，开发中一般用不到
        2.info：建议使用这个等级
        3.warn
        4.error
```

### 使用原理

```Java
就是通过动态代理的方式创建一个slf4j的Logger接口的实现类，只要执行方法，都会比当前方法的级别
和用户配置的级别的关系，决定是否输出，框架采用其他线程打印，不影响主线的逻辑
```

代码示例:

```Java
package com.qk.controller;

import com.qk.common.PageResult;
import com.qk.common.Result;
import com.qk.entity.Dept;
import com.qk.service.DeptService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

/**
 * 部门管理控制器
 */
@Slf4j
@RestController
@RequestMapping("/depts")
public class DeptController {

    @Autowired
    private DeptService deptService;

    /**
     * 新增部门
     */
    @PostMapping
    public Result addDept(@RequestBody Dept dept) {
        log.info("新增部门: {}" , dept);
        deptService.addDept(dept);
        return Result.success();
    }
    /**
     * 修改部门
     *
     * @param dept 部门信息
     * @return 统一响应结果
     */
    @PutMapping
    public Result updateDept(@RequestBody Dept dept) {
        log.info("修改部门: {}" , dept);
        deptService.updateDept(dept);
        return Result.success();
    }

    /**
     * 根据ID查询部门
     *
     * @param id 部门ID
     * @return 查询结果
     */
    @GetMapping("/{id}")
    public Result getDeptById(@PathVariable("id") Integer id) {
        log.info("根据ID查询部门: {}" , id);
        Dept dept = deptService.getDeptById(id);
        return Result.success(dept);
    }

    /**
     * 条件分页查询部门
     *
     * @param name     部门名称
     * @param status   状态
     * @param page     页码
     * @param pageSize 每页记录数
     * @return 分页查询结果
     */
    @GetMapping
    public Result listDepts(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) Integer status,
            @RequestParam(defaultValue = "1") Integer page,
            @RequestParam(defaultValue = "10") Integer pageSize) {
        log.info("条件分页查询部门: name={}, status={}, page={}, pageSize={}", name, status, page, pageSize);
        PageResult<Dept> pageResult = deptService.listDepts(name, status, page, pageSize);
        return Result.success(pageResult);
    }

    /**
     * 删除部门
     *
     * @param id 部门ID
     * @return 统一响应结果
     */
    @DeleteMapping("/{id}")
    public Result deleteDept(@PathVariable("id") Integer id) {
        log.info("删除部门: {}" , id);
        deptService.deleteDept(id);
        return Result.success();
    }
}
```

<callout emoji="✅">
lombok中提供的@Slf4j注解，可以简化定义日志记录器这步操作。添加了该注解，就相当于在类中定义了日志记录器，就下面这句代码：
`private static Logger log = LoggerFactory. getLogger(Xxx. class);`
</callout>

### 好处

日志用于记录程序运行信息，Logback 基于 Log4j 升级，功能多、性能好，可配置输出格式、位置和日志开关。

<callout emoji="📌">
1.不修改源码，可以开关式控制日志输出  
2.持久化存储日志：可以在日志配置文件中设置日志打印保存的文件地址  
3.开启线程打印日志，不影响主线逻辑  
4.拼接参数的特殊处理，采用`{}`占位即可，底层采用`StringBuilder`的方式拼接，相较于手动 `+` 效率高
</callout>

## 全局异常处理

在项目中未做异常处理时，异常会逐层上抛，最终由框架返回默认错误页面/默认 JSON，**不符合我们统一响应结果规范**（例如 `Result.error(...)`）。  
 所以通常会使用 **全局异常处理器**，集中捕获异常并统一返回规范的响应。

### 部署依赖

**Spring Boot Web 项目默认已具备全局异常处理所需能力**（基于 SpringMVC），无需额外引入“异常处理”专门依赖。

只要确保项目中有 Web 依赖即可，例如（非必须写到 pom）：

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

```

> 说明：`@RestControllerAdvice`、`@ExceptionHandler` 都属于 Spring Web/MVC 的能力。



### 使用步骤

**第一步：定义统一返回对象 Result（已有的话可忽略）**  
 全局异常最终要返回统一格式：例如 `Result.success()` / `Result.error(msg)`。

**第二步：定义自定义异常（可选，但推荐）**  
 常见做法：把“业务异常”和“系统异常”区分开，便于返回不同提示。

示例（按你代码里的命名来）：

```Java
// 业务异常：例如参数不合法、数据不存在、状态不允许操作等
public class CustomException extends RuntimeException {
    public CustomException(String message) {
        super(message);
    }
}

// 系统异常：例如未知/底层异常包装后抛出（也可用 RuntimeException 代替）
public class SystemException extends RuntimeException {
    public SystemException(String message) {
        super(message);
    }
}

```

**第三步：编写全局异常处理器（核心）**

- 使用 `@RestControllerAdvice`：对所有 `@RestController` 生效，返回 JSON
- 使用 `@ExceptionHandler(异常类型.class)`：指定要处理的异常类型

处理方式,示例代码:

```Java
package cn.itcast.exception;

import cn.itcast.Result;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {
    //编写逻辑处理异常

    //处理自定义异常
    @ExceptionHandler(CustomException.class)
    public Result handleException(CustomException e){
        e.printStackTrace();
        return Result.error(e.getMessage());
    }

    //处理系统异常，比如空指针异常，数组越界异常等
    //处理系统异常
    @ExceptionHandler(SystemException.class)
    public Result handleException(SystemException e){
        e.printStackTrace();
        return Result.error("亲服务器正忙~~~~");
    }

    //处理其他异常
    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e){
        e.printStackTrace();
        return Result.error("未知错误请联系系统管理员!!!!");
    }

}
```

**第四步：在业务代码中抛异常即可**  
 例如校验失败直接：

```Java
if (dept == null) {
    throw new CustomException("部门不存在");
}

```

Logback.xml配置模版:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!--
        CONSOLE ：表示当前的日志信息是可以输出到控制台的。
    -->

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss:SSS} [%thread] %highlight(%-5level) %cyan(%logger{36}) - %magenta(%msg) \n
            </pattern>
        </encoder>
    </appender>

    <!-- File是输出的方向通向文件的 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg \n</pattern>
            <charset>utf-8</charset>
        </encoder>
        <!--日志输出路径-->
        <file>D:/log/itheima-data.log</file>
        <!--指定日志文件拆分和压缩规则-->
        <rollingPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!--通过指定压缩文件名称，来确定分割文件方式-->
            <fileNamePattern>D:/log/itheima-data-%i-%d{yyyy-MM-dd}-.log.gz</fileNamePattern>
            <!--文件拆分大小-->
            <maxFileSize>1MB</maxFileSize>
        </rollingPolicy>
    </appender>

    <!--
        1、控制日志的输出情况：如，开启日志，取消日志
        off >  error > warn > info > debug > trace > all
        日志技术, 会输出比配置的级别要高的日志
    -->
    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

### 使用原理

全局异常处理本质是 SpringMVC 的异常解析机制：

1. 请求进入 Controller 执行业务方法
2. 发生异常后，异常会沿调顺序向上抛出
3. SpringMVC 的 `DispatcherServlet` 在处理请求时，会调用一套 **异常解析器链**
4. 当项目中存在 `@ControllerAdvice / @RestControllerAdvice` 标注的类时，Spring 会把它注册为全局增强组件
5. 解析器会匹配 `@ExceptionHandler` 方法：

   <callout emoji="🎼">
   **优先匹配最具体的异常类型**（比如 `CustomException` 优先于 `Exception`）
   匹配到后执行对应方法，得到一个返回对象
   </callout>
6. 因为是 `@RestControllerAdvice`，返回对象会被当成 JSON 写回响应体（不会跳转页面）

### 好处

```Java
1.统一返回格式：所有异常都能返回 Result.error(...)，前端好处理、接口规范统一
2.降低重复代码：不用在每个 Controller/Service 里 try-catch
3.异常分类更清晰：业务异常给明确提示；系统异常隐藏细节，避免把内部信息暴露给前端
4.便于日志与排查：集中打印堆栈、记录请求信息、定位问题更快
5.可扩展性强：后期可增加对参数校验异常、数据库异常、权限异常等的专项处理
```

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / AOP、日志与全局异常 / 08 qk功能补充_日志框架_全局异常处理 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/WvMgw1uRCi1wcSkALB8ciZgKnib)
