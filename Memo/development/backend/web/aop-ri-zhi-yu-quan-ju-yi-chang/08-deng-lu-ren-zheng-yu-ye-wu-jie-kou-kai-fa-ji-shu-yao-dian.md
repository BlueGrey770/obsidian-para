---
type: memo
date: 2026-11-06
aliases:
  - "08 登录认证与业务接口开发技术要点"
---

# 08 登录认证与业务接口开发技术要点

> 本章聚焦 **Token 认证机制、Filter 与 Interceptor 的差异、RESTful 路径参数接口**，以及「按接口文档交付 + 全模块回归」的工程素养。

## 一、技术概念说明

### 1. 登录认证的整体机制（Token 方案）

```
登录请求(账号密码) ──▶ 校验通过 ──▶ 服务端生成 Token（如 JWT）──▶ 返回给前端
                                                            │
后续每个请求 ──▶ 前端在请求头携带 Token ──▶ 服务端校验 ──▶ 通过则放行，否则 401
```

为什么需要它：HTTP 协议**无状态**，服务器不记得「谁登录过」。Token 就像景区的门票——进门发一张，之后每个景点（接口）验票放行。

### 2. 令牌校验的两个实现位置：Filter 与 Interceptor

| 维度 | Filter（过滤器） | Interceptor（拦截器） |
|-|-|-|
| 出身 | Servlet 规范（Tomcat 层） | Spring MVC 框架 |
| 拦截范围 | 几乎所有请求（含静态资源） | 进入 Controller 的请求 |
| 执行时机 | 进入 Spring 之前 | Controller 方法前后 |
| 依赖注入 | 不方便（不在 Spring 容器） | 方便（可注入 Service/配置） |

同一条校验链在两个位置的写法差异，可以归纳为四步：**获取请求头 → 判空 → 解析 Token → 通过则放行，失败则写回 401**。

### 3. RESTful 路径参数（Path Variable）

形如 `GET /courses/subject/{subject}` 的接口，参数直接嵌在 URL 路径里（如 `/courses/subject/1`）。接收方式：

```java
@GetMapping("/subject/{subject}")
public Result listBySubject(@PathVariable Integer subject) { ... }
```

适用场景：参数是**资源的标识**（学科 ID、类型 ID）；查询字符串（`?name=x`）适合可选的过滤条件——「按类型查询」用路径参数，因为学科/类型就是资源本身的分类维度。

### 4. 回归测试（Regression Check）

新功能开发完后**回头验证旧功能没有被改坏**。认证过滤器加错路径、统一响应结构调整，都可能让老接口集体 401 或格式错乱。手段：把各模块接口按文档全跑一遍。

## 二、技术点之间的联系

```
登录认证 ──守门──▶ 所有业务接口
  │  TokenFilter / TokenInterceptor 二选一（或同练）
  │  白名单：登录接口本身必须放行（否则永远无法登录）
  │
三层架构（上一章）──承载──▶ 课程/活动接口开发
  │  Controller 新增 @PathVariable 收参方式
  │  Mapper 新增 "where subject = #{subject}" 等值查询
  │
接口文档 ──约束──▶ 每个接口的路径/方法/参数/响应
  │
全模块回归 ──收口──▶ 认证 + 五个业务模块的兼容性验证
```

三条关键联系：

1. **认证是所有接口的前置横切逻辑**：课程、活动、部门……每个接口都要验 Token，但校验代码只写一份——放在 Filter/Interceptor 里统一拦截，这正是「横切关注点」思想，与 AOP 同源。
2. **新接口开发 = 老套路 + 新参数形式**：按学科查课程，Mapper 就是一个 `where subject = ?` 的等值查询，Controller 唯一的新知识点是 `@PathVariable`——功能增量小，熟练度增量大。
3. **回归测试验证的是「横切改动」的波及面**：加认证、改统一响应这类全局性改动，影响的是所有模块，必须全量回归——只测新接口是测不出来的。

## 三、方法与实现要点

### 1. 令牌校验的自查问题清单

能回答以下问题才算「捋清楚」：请求头里 Token 叫什么名？没带 Token 返回什么状态码和报文？Token 解析失败（过期/伪造）怎么走？登录接口的白名单怎么排除？校验通过后用户信息放哪（ThreadLocal）？

### 2. Filter 与 Interceptor 的选择口径

需要拦静态资源、或要在 Spring 容器之前处理 → Filter；需要注入 Spring Bean（如查库验证用户）、只拦 Controller 请求 → Interceptor。二者可以共存，Filter 永远先执行。

### 3. 按文档开发的最小验收动作

对照接口文档四要素逐项核对：路径（`/courses/subject/{subject}` 注意路径段顺序）、方法（GET 别写成 POST）、参数（路径参数 vs 查询参数）、响应结构（code/msg/data）。在 Apifox 里建用例保存，回归时一键全跑。

### 4. 回归检查的实用清单

- **认证类**：无 Token 访问是否 401、带错 Token 是否 401、登录后各模块是否正常；
- **业务类**：每个模块的增删改查各抽一条用例；
- **边界类**：分页第一页/最后一页、条件查询空条件、路径参数非法值。

## 四、重点难点分析

### 重点 1：Token 认证的完整链路

登录签发 → 前端存储（localStorage）→ 请求头携带 → 服务端解析校验 → 放行/拒绝。能讲清每一步「数据在哪、谁生成、谁验证、过期怎么办」，就掌握了无状态认证的骨架。

### 重点 2：Filter 与 Interceptor 的区别

出身（Servlet vs SpringMVC）、范围、执行顺序（Filter 先于 Interceptor）、注入能力四点能对比着说清，并给出选型理由。

### 难点 1：白名单与拦截顺序

登录接口 `/login` 必须被排除在校验之外（白名单），否则系统永远无法进入。Filter 里通常按路径前缀匹配排除；拦截器在注册时 `excludePathPatterns("/login")`。顺序错了或漏配，表现为「全站 401」。

### 难点 2：路径参数与请求体/查询参数的混用规则

同一个 Controller 里可能同时出现：`@PathVariable`（路径里的学科 ID）、`@RequestParam`（查询字符串的分页参数）、`@RequestBody`（POST 的 JSON 体）。三者来源不同、注解不同，混用或漏注解都会拿到 null——报错不明显，排查先看注解。

### 易错点清单

- 校验逻辑里「解析失败」和「没携带」混为一谈——应分别返回明确提示，便于前端区分「未登录」与「登录过期」；
- `@PathVariable` 漏写或名字与路径占位符不一致——参数绑定为 null；
- 认证 Filter 误拦了登录接口或静态资源——白名单配置要覆盖；
- 回归只点新功能——认证类改动必须全模块过一遍。

## 五、示例

### 示例 1：Token 校验过滤器

```java
@Component
public class TokenFilter implements Filter {
    private static final List<String> WHITE_LIST = Arrays.asList("/login", "/captcha");

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;
        String uri = request.getRequestURI();

        // 1. 白名单放行：登录接口不能拦，否则永远进不来
        if (WHITE_LIST.stream().anyMatch(uri::startsWith)) {
            chain.doFilter(req, resp);
            return;
        }

        // 2. 取请求头中的 token
        String token = request.getHeader("token");
        if (token == null || token.isEmpty()) {
            writeError(response, "未登录，请先登录");
            return;
        }

        // 3. 解析校验（过期/伪造都会抛异常）
        try {
            Claims claims = JwtUtils.parse(token);
            CurrentHolder.set(claims.get("userId", Integer.class));   // ThreadLocal 传递用户
        } catch (Exception e) {
            writeError(response, "登录已过期，请重新登录");
            return;
        }

        chain.doFilter(req, resp);          // 4. 放行
    }

    private void writeError(HttpServletResponse response, String msg) throws IOException {
        response.setStatus(401);
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().write("{\"code\":0,\"msg\":\"" + msg + "\",\"data\":null}");
    }
}
```

要点：「没携带」与「解析失败」分别给不同提示——前端才能区分跳登录页还是提示续期。

### 示例 2：Token 校验拦截器（可注入 Spring Bean）

```java
@Component
public class TokenInterceptor implements HandlerInterceptor {

    @Autowired
    private JwtUtils jwtUtils;            // 拦截器在 Spring 容器内，可注入

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        if ("OPTIONS".equalsIgnoreCase(request.getMethod())) return true;   // 放行预检

        String token = request.getHeader("token");
        if (token == null || token.isEmpty()) {
            response.setStatus(401);
            return false;                  // 返回 false 即中断，不再进 Controller
        }
        try {
            CurrentHolder.set(jwtUtils.parseUserId(token));
            return true;
        } catch (Exception e) {
            response.setStatus(401);
            return false;
        }
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                                Object handler, Exception ex) {
        CurrentHolder.remove();            // 线程复用，必须清理 ThreadLocal
    }
}

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    private TokenInterceptor tokenInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(tokenInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/login", "/error");    // 白名单
    }
}
```

要点：`afterCompletion` 里必须 `remove()` ThreadLocal，否则线程池复用时会串用户数据。

### 示例 3：路径参数接口（按学科查课程）

```java
@RestController
@RequestMapping("/courses")
public class CourseController {

    @Autowired
    private CourseService courseService;

    // GET /courses/subject/1
    @GetMapping("/subject/{subject}")
    public Result listBySubject(@PathVariable Integer subject) {
        return Result.ok(courseService.listBySubject(subject));
    }

    // GET /courses?name=x&page=1&pageSize=10   → 查询参数
    @GetMapping
    public Result page(@RequestParam(required = false) String name,
                       @RequestParam(defaultValue = "1") Integer page,
                       @RequestParam(defaultValue = "10") Integer pageSize) {
        return Result.ok(courseService.page(name, page, pageSize));
    }

    // POST /courses  {"name": "...", ...}      → 请求体
    @PostMapping
    public Result add(@RequestBody Course course) {
        return courseService.add(course) > 0 ? Result.ok("新增成功") : Result.error("新增失败");
    }
}
```

要点：三种参数来源对应三个注解——路径里用 `@PathVariable`、URL 问号后用 `@RequestParam`、JSON 体用 `@RequestBody`；漏注解参数就是 null。

### 示例 4：Mapper 的等值查询

```xml
<select id="listBySubject" resultType="cn.itcast.entities.Course">
    select * from course where subject = #{subject} order by update_time desc
</select>
```

要点：路径参数是确定的资源标识，不需要 `<if>` 动态判断——比条件查询简单一层。

### 示例 5：回归测试用例表

| 类别 | 用例 | 期望 |
|-|-|-|
| 认证 | 不带 token 访问 `/courses` | 401，提示「未登录」 |
| 认证 | 带过期 token 访问 `/courses` | 401，提示「登录已过期」 |
| 认证 | 正常登录后访问各模块 | 200，code=1 |
| 业务 | 部门/角色/用户/课程/活动 各一条增删改查 | 200，数据正确 |
| 边界 | `page=1`、`page=最后一页`、`page` 超出范围 | 首页/末页正常，超出返回空列表 |
| 边界 | `subject` 传非数字 | 400 参数类型错误 |

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / AOP、日志与全局异常 / 08 登录认证与业务接口开发技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/S505wxu4DiWNIhkFR5TccgZonNg)
