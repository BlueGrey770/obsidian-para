---
type: memo
date: 2026-11-06
aliases:
  - "01-后台认证授权、小程序登录"
---

# 01-后台认证授权、小程序登录

## 目标

今天的目标为：

- 能够描述清楚RBAC权限模型的作用
- 能够描述清楚RBAC权限模型中涉及到的业务及表关系
- 能够掌握Spring security安全如何完成身份认证和权限验证
- 能够掌握三方接口调用的方式
- 能够独立完成微信登录功能的开发
- 能够独立完成校验token的逻辑代码编写



## RBAC权限模型

### 概述

在企业系统中，通过配置用户的功能权限可以实现让不同的人分管不同业务的需求。基于RBAC模型，RBAC（Role Based Access Control)模型，它的中文是基于角色的访问控制，主要是将功能组合成角色，再将角色分配给用户，也就是说**角色是功能的合集**。

比如：

企业A总共有12个功能，需要创建100个用户。功能包括财务管理、人事管理、销售管理等等，如果不引入基于角色的访问控制（RBAC）模型，我们就需要每创建一个用户就要分配一次功能，这将至少需要进行100次操作（每个用户只能拥有一个功能的前提下）。如果用户数量增加到1000甚至10000，并且一个用户可能会拥有多个功能，操作将会变得非常繁琐。如图：

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/ONsmbvU9Tohyz9xwh6VczyjunFg.png)

经过多次操作后发现，有些人被分配了相同的功能。例如，A、B等10个用户都被分配了客户管理、订单管理和供应商管理这几个模块。我们是否可以将这几个功能模块组合成一个包，然后将整个包分配给需要的用户呢？这个包被称为角色。由于角色和功能之间的对应关系相对稳定，在给用户分配权限时只需分配角色即可，如下图所示：

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/Gqh3bnbftocau8xfZA3cGDDonVg.png)

基于RBAC授权模式后，我们可以达成以下2个目标：

- 解耦用户和功能，降低操作错误率
- 降低功能权限分配的繁琐程度

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/VB4rbyzLyoFYDXxMcwzcjw3Enwc.png)

### **ER图与关系梳理**

在一个核心业务系统中，我们通常通过业务分析，从而抽离出数据库表，表确定之后我们会进一步分析表中应该有的字段，下面先看一份业务ER图：

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/QxZdbxQE0oVOdGxYEIwc8QOxnRf.jpg)

上图中清楚的描述用户、角色、资源、职位、部门之间的关系，同时我们进一步推导出以下结果：

- 用户与职位是N:N关系
- 用户与部门是N:1关系
- 用户与角色是N:N关系
- 角色与资源是N:N关系
- 部门与角色是N:N关系

表关系（简图）

<whiteboard token="GF2kwZjUZhOa3TbSha6cxcOlnpS"></whiteboard>

根据用户名查询菜单信息

```SQL
-- 先根据用户名查询用户id、再根据用户id查询角色id、再根据角色id查询菜单id，最后根据菜单id查询菜单信息
select sm.*
from sys_menu sm
where sm.menu_id in (select srm.menu_id
                     from sys_role_menu srm
                     where srm.role_id in (select sur.role_id
                                           from sys_user_role sur
                                           where sur.user_id = (select su.user_id
                                                                from sys_user su
                                                                where su.user_name = 'laowang')));

-- 优化为关联查询
select sm.*
from sys_menu sm
         left join sys_role_menu srm on sm.menu_id = srm.menu_id
         left join sys_user_role sur on srm.role_id = sur.role_id
         left join sys_user su on sur.user_id = su.user_id
where su.user_name = 'xiaowang';
```

## Spring Security

### 概述

Spring Security是一个基于Spring**框架**的**安全**性解决方案，它为应用程序提供了完整的**安全管理**功能，包括认证、授权、攻击防范和会话管理等，官网地址：https://spring.io/projects/spring-security。

> Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.
> 
> Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirements

在若依系统中，已经集成了Spring Security安全框架。

### 核心作用

我们可以用一张图来说明一下spring security的核心作用

<whiteboard token="TWHGw5x4dhoNBObnRPWcK1imnEb"></whiteboard>

- 用户登录

  - 校验验证码是否正确
  - 通过spring security框架验证用户是否存在，密码是否正确
  - 查询用户所拥有的权限列表
  - 最后把用户信息封装到jwt生成的token中返回
- 其他请求（可以是非登录的任意请求），这些校验工作可以全部交给Spring Security来完成

  - url是否需要检验（**访问接口**是否需要放行，比如登录、注册、验证码这些接口；还有一些**静态资源**也需要放行，比如html、js、css等）
  - token校验，是否为空，是否过期，是否正常解析
  - 校验当前访问的url与当前用户的权限是否匹配，匹配则放行，不匹配则是权限不足

### 权限控制

#### 认证

认证（Authentication）

认证是用户**登录**系统时，通过比较用户提供的凭证（如用户名和密码）与系统中存储的信息是否一致，以验证用户身份真实有效的过程。

当前系统的认证流程：

<whiteboard token="M1Wmw3ySKhTdz2bDqgFcsAMinpc"></whiteboard>



> 代码入口：zzyl-admin模块下找到：com.zzyl.web.controller.system.SysLoginController.login()



#### 授权

授权（Authorization）：

- 授权就是用户登录后(认证后)，控制用户是否有权限访问某些资源。
- 下图是给用户赋予角色，并且可以可以赋予多个角色

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/EIlAbtNrbogCckx6C3KczJuDnnf.png)

下图中，可以给角色赋予多个功能权限

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/MH5dbR3R1owCBzxiKf5cffXVnRd.png)

下图中，我们可以对菜单和按钮进行权限标识的配置

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/LUUybYUtBozUXpx14D3cKEpHnCb.png)

#### 权限验证

一个请求过来之后，需要有诸多的校验来能判断是否能有效访问

<whiteboard token="GTeEwyUqjhHcIEbTKEdcoyEonvf"></whiteboard>

- 请求来了以后，比如是`doc.html`或`login`，是一个静态资源，可直接放行，在SecurityConfig可以配置放行
- 请求来了以后，比如是`/nursing/elder/save`，在SecurityConfig没有配置放行，需要校验当前请求携带的token是否有效，**如果token无效，则是认证失败**，需要重新认证（跳转到登录页）
- 请求来了以后，比如是`/nursing/elder/save`，在SecurityConfig没有配置放行，token也有效，则**判断当前登录人是否拥有该权限(认证成功后，可以获取用户的权限列表)**

  - 如果有权限，则放行
  - 如果没权限，则提醒权限不足



核心代码位置：在zzyl-framework模块中找到类：`com.zzyl.framework.config.SecurityConfig`

```Java
/**
 * spring security配置
 * 
 * @author ruoyi
 */
@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true)
@Configuration
public class SecurityConfig
{
    /**
     * 自定义用户认证逻辑
     */
    @Autowired
    private UserDetailsService userDetailsService;
    
    /**
     * 认证失败处理类
     */
    @Autowired
    private AuthenticationEntryPointImpl unauthorizedHandler;

    /**
     * 退出处理类
     */
    @Autowired
    private LogoutSuccessHandlerImpl logoutSuccessHandler;

    /**
     * token认证过滤器
     */
    @Autowired
    private JwtAuthenticationTokenFilter authenticationTokenFilter;
    
    /**
     * 跨域过滤器
     */
    @Autowired
    private CorsFilter corsFilter;

    /**
     * 允许匿名访问的地址
     */
    @Autowired
    private PermitAllUrlProperties permitAllUrl;

    /**
     * 身份验证实现
     */
    @Bean
    public AuthenticationManager authenticationManager()
    {
        DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();
        daoAuthenticationProvider.setUserDetailsService(userDetailsService);
        daoAuthenticationProvider.setPasswordEncoder(bCryptPasswordEncoder());
        return new ProviderManager(daoAuthenticationProvider);
    }

    /**
     * anyRequest          |   匹配所有请求路径
     * access              |   SpringEl表达式结果为true时可以访问
     * anonymous           |   匿名可以访问
     * denyAll             |   用户不能访问
     * fullyAuthenticated  |   用户完全认证可以访问（非remember-me下自动登录）
     * hasAnyAuthority     |   如果有参数，参数表示权限，则其中任何一个权限可以访问
     * hasAnyRole          |   如果有参数，参数表示角色，则其中任何一个角色可以访问
     * hasAuthority        |   如果有参数，参数表示权限，则其权限可以访问
     * hasIpAddress        |   如果有参数，参数表示IP地址，如果用户IP和参数匹配，则可以访问
     * hasRole             |   如果有参数，参数表示角色，则其角色可以访问
     * permitAll           |   用户可以任意访问
     * rememberMe          |   允许通过remember-me登录的用户访问
     * authenticated       |   用户登录后可访问
     */
    @Bean
    protected SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception
    {
        return httpSecurity
            // CSRF禁用，因为不使用session
            .csrf(csrf -> csrf.disable())
            // 禁用HTTP响应标头
            .headers((headersCustomizer) -> {
                headersCustomizer.cacheControl(cache -> cache.disable()).frameOptions(options -> options.sameOrigin());
            })
            // 认证失败处理类
            .exceptionHandling(exception -> exception.authenticationEntryPoint(unauthorizedHandler))
            // 基于token，所以不需要session
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            // 注解标记允许匿名访问的url
            .authorizeHttpRequests((requests) -> {
                permitAllUrl.getUrls().forEach(url -> requests.antMatchers(url).permitAll());
                // 对于登录login 注册register 验证码captchaImage 允许匿名访问
                requests.antMatchers("/login", "/register", "/captchaImage").permitAll()
                    // 静态资源，可匿名访问
                    .antMatchers(HttpMethod.GET, "/", "/*.html", "/**/*.html", "/**/*.css", "/**/*.js", "/profile/**").permitAll()
                    .antMatchers("/swagger-ui.html", "/swagger-resources/**", "/webjars/**", "/*/api-docs", "/druid/**").permitAll()
                    // 除上面外的所有请求全部需要鉴权认证
                    .anyRequest().authenticated();
            })
            // 添加Logout filter
            .logout(logout -> logout.logoutUrl("/logout").logoutSuccessHandler(logoutSuccessHandler))
            // 添加JWT filter
            .addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class)
            // 添加CORS filter
            .addFilterBefore(corsFilter, JwtAuthenticationTokenFilter.class)
            .addFilterBefore(corsFilter, LogoutFilter.class)
            .build();
    }

    /**
     * 强散列哈希加密实现
     */
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder()
    {
        return new BCryptPasswordEncoder();
    }
}
```



具体的url检验是通过Spring Security的提供的权限注解`@PreAuthorize`实现的，**可以判断具体的url是否可以访问**

- @PreAuthorize 是 Spring Security 框架中提供的一个安全注解，用于实现基于注解的访问控制。它允许开发者在**方法级别**上声明特定的安全约束，以确保只有满足指定条件的用户才能调用该方法
- 当 @PreAuthorize 注解被应用于某个方法时，Spring Security 在**该方法执行前**会先对当前认证的用户进行权限检查。如果检查通过，方法调用得以继续；否则，框架会抛出相应的权限异常（如 AccessDeniedException），阻止方法执行。
- 权限控制代码，若依框架生成的Controller中都可以见到，如下：

  ```Java
  @PreAuthorize("@ss.hasPermi('system:user:list')")
  @GetMapping("/list")
  public TableDataInfo list(SysUser user) {
  
  }
  ```

  - @PreAuthorize  Spring Security框架的权限注解，在执行方法前执行
  
    - 已经开启注解生效，在SecurityConfig类中添加了如下注解：
    - `@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)`
  - @ss.hasPermi('system:user:list')
  
    - 其中的ss是一个被spring管理的bean
    
      - 位置：zzyl-framework模块中的`com.zzyl.framework.web.service.PermissionService`
    - hasPermi  是`PermissionService`类中的一个方法，判断是否拥有该权限
    - system:user:list   为方法的参数
- 权限控制的流程

<whiteboard token="ASEJwUZ5whkiDZbrdnqcgY3inLh"></whiteboard>



## 三方接口

### 接口文档

在项目中对接任何三方接口，最重要的参考永远都是官方给提供的接口文档。所以大家一定要具备阅读接口文档的能力才能更好的对接接口使用。

那接口文档该怎么阅读呢？

其实跟我们定义的接口差不了太多，也必然会包含接口的四要素（请求路径、请求方式、入参、出参）

我们来看一下微信官方给提供的这个接口，这个是用于微信登录的接口，链接如下：

https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/user-login/code2Session.html

这个文档中比较关键的部分就是：

<whiteboard token="S5NKw3iJXh5fMxbN3OjcxZGJnod"></whiteboard>

- 功能描述，查阅是否符合我们业务对接的需求
- 调用方式，三方接口一般都是https居多，更加安全，包含了请求方式和请求地址
- 请求参数，接口的入参，重点关注**必须要传的参数**
- 返回参数，查看返回的数据结构和内容，是否符合自己的返回预期
- 错误码，当接口调用失败的时候，可以根据错误码对应的描述修正代码解决问题

### API客户端

如果我们阅读明白了接口文档，也符合我们的业务需求，那么就可以在我们的后台服务中调用三方接口。

那如何调用呢？

不同的接口调用的方式不太一样，不过，基本的思路都是基于**http请求**来调用，不过，我们之前使用的http请求，都是前端到后端来发起请求，如下图

<whiteboard token="HXmqwCfBxhQBAsbLvf3cobYmnT3"></whiteboard>

- Client客户端一般都是指浏览器发起了请求
- Server是指服务端，Java的后端服务



如果从后端调用三方接口，一样也是用的http请求，只是这次请求是从后端到另外一个后端服务，我们需要借助工具才能发起请求

<whiteboard token="TSqDwQftehEHTkbmQ8acHFSQnic"></whiteboard>

HTTP客户端工具有很多

- HttpClient    体量庞大，jdk11以上支持不好
- OkHttp        学习成本较高
- Spring提供的 RestTemplate    后期课程feign会重点讲解
- **糊涂工具包，使用说明地址：https://doc.hutool.cn/pages/http/**



### 糊涂工具常见方法

我们可以参考糊涂工具的官方文档，来快速感受一下这个工具类：https://doc.hutool.cn/pages/HttpUtil/



- 准备：在zzyl-common模块中添加单元测试依赖

```XML
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```

在zzyl-common模块下创建测试类：com.zzyl.nursing.test.HttpTest

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/FRPCbdre6o3h0qx466YcKxqUnqh.png)

- GET请求

```Java
package com.zzyl.nursing.test;

import cn.hutool.http.HttpResponse;
import cn.hutool.http.HttpUtil;
import cn.hutool.http.Method;
import org.junit.Test;

import java.util.HashMap;
import java.util.Map;

public class HttpTest {

    @Test
    public void testGet() {
        String result = HttpUtil.get("https://www.baidu.com");
        System.out.println(result);
    }

    @Test
    public void testGetByParam() {
        // 构建查询参数
        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("pageNum", 1);
        paramMap.put("pageSize", 5);
        // 分页查询护理项目
        String result = HttpUtil.get("http://localhost:8080/nursing/project/list", paramMap);
        System.out.println(result);
    }

    @Test
    public void testCreateRequest() {
        // 构建查询参数
        Map<String,Object> paramMap = new HashMap<>();
        paramMap.put("pageNum", 1);
        paramMap.put("pageSize", 5);
        // 分页查询护理项目
        HttpResponse response = HttpUtil.createRequest(Method.GET, "http://localhost:8080/nursing/project/list")
                .header("Authorization", "Bearer eyJhbGciOiJIUzUxMiJ9.eyJsb2dpbl91c2VyX2tleSI6ImUyNTgyNzdhLTMxNzAtNGFhNS05MmNhLWRiNjFiNDYyOGYxNCJ9.p8ieeQe4A6lozsxpzTL4Z3JNAS3UgBOVRZckDFnNvJQs8C2I5X5nZVxAQCaUv_0u9yUN-d0FKDCHwtCkMeIdBA")
                .form(paramMap)
                .execute();
        if(response.isOk()){
            System.out.println(response.body());
        }
    }
}
```



- POST请求

```Java
@Test
public void testPost() {
    String url = "http://localhost:8080/nursing/project";
    HashMap<String, Object> paramMap = new HashMap<>();
    paramMap.put("name", "护理项目测试");
    paramMap.put("orderNo", 1);
    paramMap.put("unit", "次");
    paramMap.put("price", 10.00);
    paramMap.put("image", "https://yjy-slwl-oss.oss-cn-hangzhou.aliyuncs.com/ae7cf766-fb7b-49ff-a73c-c86c25f280e1.png");
    paramMap.put("nursingRequirement", "无特殊要求");
    paramMap.put("status", 1);
    String result = HttpUtil.post(url, JSONUtil.toJsonStr(paramMap));
    System.out.println(result);
}

@Test
public void testCreatePost() {
    String url = "http://localhost:8080/nursing/project";
    HashMap<String, Object> paramMap = new HashMap<>();
    paramMap.put("name", "护理项目测试");
    paramMap.put("orderNo", 1);
    paramMap.put("unit", "次");
    paramMap.put("price", 10.00);
    paramMap.put("image", "https://yjy-slwl-oss.oss-cn-hangzhou.aliyuncs.com/ae7cf766-fb7b-49ff-a73c-c86c25f280e1.png");
    paramMap.put("nursingRequirement", "无特殊要求");
    paramMap.put("status", 1);

    // 发送请求
    HttpResponse response = HttpUtil.createPost(url).body(JSONUtil.toJsonStr(paramMap))
            .header("Authorization", "Bearer eyJhbGciOiJIUzUxMiJ9.eyJsb2dpbl91c2VyX2tleSI6ImUyNTgyNzdhLTMxNzAtNGFhNS05MmNhLWRiNjFiNDYyOGYxNCJ9.p8ieeQe4A6lozsxpzTL4Z3JNAS3UgBOVRZckDFnNvJQs8C2I5X5nZVxAQCaUv_0u9yUN-d0FKDCHwtCkMeIdBA")
            .execute();
    if(response.isOk()) {
        System.out.println(response.body());
    }

}
```

`HttpUtil.get`和`HttpUtil.post`这两个方法调用相对简单，如果请求参数中没有额外的条件设置，就可以使用这两个方法，但是，有的时候三方接口调用需要使用到header或者是文件上传，就不能使用这两个方法了，此时我们可以使用`HttpUtil.`*`createRequest`*方法来去构建更多的条件参数



## 微信登录

### 需求分析

我们打开原型文档： 定位在家属端原型图-->登录模块

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/Tx26b1VBMoqXRfxPysycDUQLnXg.png)

在任意需要登录才能显示信息的页面点击时，都会判断当前用户是否登录，如未登录，则进入登录页面

点击微信账号登录，弹出弹窗，需要获取用户手机号进行登录

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/AxnGbZOIaoHPNBxMAszclVamndc.png)

微信登录业务逻辑规则：

<whiteboard token="J48tw8CobhuoC6b7FyfctbxXnxf"></whiteboard>

### 小程序环境搭建

#### 基础环境说明

- 前提：在资料文件夹中找到”微信开发者工具安装包”安装到本地，也可以直接到[官网下载](https://developers.weixin.qq.com/miniprogram/dev/devtools/stable.html)
- 在`中州养老-智能检测系统-华为云\资料\小程序代码`文件夹中找到”mp-weixin.zip”的压缩包，解压到一个没有中文和空格的目录
- 打开微信开发者工具，需要使用微信扫描登录

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/Ee41bTacAoBWv2xFUwzc2uNqnXd.png)

登录成功后，点击“+”号，找到刚才解压的小程序代码的目录，然后打开

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/GL7xbvPMpochbbxnTHHc2THunUe.png)

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/Y6r2b2uxkoZVKHxN7JEceIp9nnd.png)

打开之后，需要信任该项目，加载完毕之后的效果如下：

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/RfUfbtQWDo0YurxKbwccr60Onlf.png)

> 在小程序登录的时候，需要获取微信用户的手机号，目前在开发测试阶段，如果想要获取手机号需要把开发者加入小程序的开发者团队中，一个小程序的开发者团队最多50人。所以不能让每位同学都加入开发者团队中，不过，在微信小程序中可以由个人开通测试号来进行调试，这样就可以获取更多的微信信息了，就包含了手机号。

现在大家需要每人注册一个微信小程序的测试号：<cite doc-id="G0VLwjZdLiq0gwkyrwgcMsamnzh" file-type="wiki" title="小程序AppId申请" type="doc"></cite>



#### 必要配置

1. 修改请求的路径

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/YKasbA0qQoddkYxmRqLckdUUndg.png)

1. 需要忽略https请求

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/Bqggb3Zt8oWv4bxw0iBcLXfXnbd.png)

修改小程序环境的APPID

然后打开 -->详情-->基本信息-->修改**APPID**,改为自己申请的测试号APPID

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/Wv0RbEaA4ooVMwxceTGcla2Snjg.png)

#### 测试前后端

1. 放行小程序请求

由于小程序端单独有一套认证校验的逻辑，不需要使用spring security框架来做认证和授权，需要修改SecurityConfig类，对小程序中的所有请求放行。

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/V99RbGZCKovl0JxMeEFcEIAbnNh.png)

1. 到资料中拷贝准备好的类

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/XBtXbEtpWonLHzxfxHNcqRRenEh.png)

> 这个类是查询所有房型的



拷贝到zzyl-nursing-platform模块中，效果如下：

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/LVQUbPIvooKtXXx7YUXctuE3nmg.png)

重启项目，访问小程序

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/Xzs0bnPTyo4CQuxXd8ccuvkznwf.png)

### 微信登录思路说明

我们可以根据微信小程序开放平台给提供的实现思路来梳理微信登录的流程，链接如下：

微信小程序登录，官方文档：

[https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html)

微信官方推荐登录流程：

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/MFXJbcEN3okmXKxBOsycKqbcnHb.png)

注意点：

- 前端在小程序中集成微信相关依赖，当用户请求登录的同时，调用[wx.login()](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/login/wx.login.html) 获取 **临时登录凭证code** ，并回传到开发者服务器。
- 后端服务器调用 [auth.code2Session](https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/user-login/code2Session.html) 接口，换取 **用户唯一标识 OpenID** 、 用户在微信开放平台账号下的**唯一标识UnionID**（若当前小程序已绑定到微信开放平台账号） 和 **会话密钥 session_key**
- 开发者服务器可以根据用户标识来生成自定义登录态，用于后续业务逻辑中前后端交互时识别用户身份。



### 表结构说明

从小程序端登录的用户主要有两类，第一类是参观预约的用户，第二类是老人的家人（子女），方便查看老人信息、给老人下单、查看账单、查看合同等服务。

如果是老人的家人，需要跟入住的老人进行绑定，方便后期享受更多服务，所以，我们需要将在小程序端登录的用户保存下来，保存到一张表中 family_member（家属表）

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/EoM6b0gDJo2OWKxIwArclmYynIc.png)

建表语句：

```SQL
CREATE TABLE "family_member" (
  "id" bigint NOT NULL AUTO_INCREMENT COMMENT '主键',
  "phone" varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '手机号',
  "name" varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '名称',
  "avatar" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '头像',
  "open_id" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT 'OpenID',
  "gender" int DEFAULT NULL COMMENT '性别(0:男，1:女)',
  "create_time" timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  "update_time" timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  "create_by" bigint DEFAULT NULL COMMENT '创建人',
  "update_by" bigint DEFAULT NULL COMMENT '更新人',
  "remark" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '备注',
  PRIMARY KEY ("id") USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC COMMENT='老人家属';
```

### 接口说明

这个接口跟我们平时开发的接口略有不同，这个需要参考微信开发者平台提供的流程来开发。

目前，小程序的代码已经开发完了，我们需要在后台提供接口就可以了

- 接口路径（已固定）：`/member/user/login`
- 请求方式（已固定）：`POST`
- 请求参数：（已固定）以上三个参数，都是前端开发人员调用`wx.login()`方法返回的数据

  ```JSON
  {
    "code": "0e36jkGa1ercRF0Fu4Ia1V3fPD06jkGW", // 临时登录凭证code
    "nickName": "微信用户",
    "phoneCode": "13fe315872a4fb9ed3deee1e5909d5af60dfce7911013436fddcfe13f55ecad3"
  }
  ```

  - code:临时登录凭证code
  - nickName:微信用户昵称（现在微信统一返回为：微信用户）
  - phoneCode:详细用户信息code，后台根据此code可以获取用户手机号
- 响应示例

  ```JSON
  {
    "code": 200,
    "msg": "操作成功",
    "data": {
      "token": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiLlpb3mn7_lvIDoirE4OTE1IiwiZXhwIjoxNDY1MjI3MTMyOCwidXNlcmlkIjoxfQ.nB6ElZbUywh-yiHDNMJS8WqUpcLWCszVdvAMfySFxIM",
      "nickName": "好柿开花8915"
    },
    "operationTime": null
  }
  ```



### 功能实现

#### 实现思路说明

基于微信官方提供的流程图，结合我们的业务，最终的实现思路，如下图

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/FrcYbwqEJo1XtRxJJvYcBL8ynpc.jpg)

#### 基础代码准备

按照之前的套路，我们使用代码生成的功能，来生成代码，

- 包名:`com.zzyl.nursing`
- 模块名：`nursing`
- 业务名:`member`
- 除了主键之外的Long类型，改为Integer

**只拷贝后端代码到idea中**

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/L4jGbz4dwoiRCrxaOdJcQj6Ennc.png)

#### 控制层

在zzyl-nursing-platform模块中，把FamilyMemberController移动到member包下，结构如下：

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/Fd3Jbx4Jso8oFzxZUpAcsc7Tnje.png)

删除里面所有的方法，新定义一个登录方法，最终代码如下：

```Java
package com.zzyl.nursing.controller.member;

import com.zzyl.common.core.controller.BaseController;
import com.zzyl.common.core.domain.AjaxResult;
import com.zzyl.nursing.dto.UserLoginRequestDto;
import com.zzyl.nursing.service.IFamilyMemberService;
import com.zzyl.nursing.vo.LoginVo;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 老人家属Controller
 * 
 * @author ruoyi
 * @date 2024-09-02
 */
@RestController
@RequestMapping("/member/user")
@Api(tags =  "老人家属相关接口")
public class FamilyMemberController extends BaseController
{
    @Autowired
    private IFamilyMemberService familyMemberService;

    @PostMapping("/login")
    @ApiOperation("小程序登录")
    public AjaxResult login(@RequestBody UserLoginRequestDto userLoginRequestDto) {
        LoginVo loginVo = familyMemberService.login(userLoginRequestDto);
        return success(loginVo);
    }

}
```

基于接口文档创建对应的DTO和VO

UserLoginRequestDto 

```Java
package com.zzyl.nursing.dto;

import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

/**
 * C端用户登录
 */
@Data
public class UserLoginRequestDto {

    @ApiModelProperty("昵称")
    private String nickName;

    @ApiModelProperty("登录临时凭证")
    private String code;

    @ApiModelProperty("手机号临时凭证")
    private String phoneCode;
}
```

LoginVo

```Java
package com.zzyl.nursing.vo;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

/**
 * LoginVO
 * @author itheima
 */
@Data
@ApiModel(value = "登录对象")
public class LoginVo {

    @ApiModelProperty(value = "JWT token")
    private String token;

    @ApiModelProperty(value = "昵称")
    private String nickName;
}
```



#### 业务层

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/G6KfbBL0soFgexxIS8XcLC8VnVe.jpg)

我们从整个流程可以看出来，在后台实现的过程中，我们需要调用两次微信开发者平台的接口来获取数据，一般像这种第三方接口的调用，我们通常都会将所有相关接口的调用都封装到一个单独的业务类中（Service层接口和实现类），使其更通用

我们可以先分析微信开发者平台的接口，接口地址：

- [获取用户openId](https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/user-login/code2Session.html)
- [获取手机号](https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/user-info/phone-number/getPhoneNumber.html)
- [获取token](https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/mp-access-token/getAccessToken.html)

##### 微信接口调用-单独封装

新增WechatService接口

```Java
package com.zzyl.nursing.service;

public interface WechatService {

    /**
     * 获取openid
     * @param code
     * @return
     */
    public String getOpenid(String code);

    /**
     * 获取手机号
     * @param detailCode
     * @return
     */
    public String getPhone(String detailCode);
}
```

新增WechatServiceImpl实现类

```Java
package com.zzyl.nursing.service.impl;

import cn.hutool.core.util.ObjectUtil;
import cn.hutool.http.HttpUtil;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import com.zzyl.nursing.service.WechatService;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Service
public class WechatServiceImpl implements WechatService {


    // 登录
    private static final String REQUEST_URL = "https://api.weixin.qq.com/sns/jscode2session?grant_type=authorization_code";

    // 获取token
    private static final String TOKEN_URL = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential";

    // 获取手机号
    private static final String PHONE_REQUEST_URL = "https://api.weixin.qq.com/wxa/business/getuserphonenumber?access_token=";


    @Value("${wechat.appId}")
    private String appid;

    @Value("${wechat.appSecret}")
    private String secret;


    /**
     * 获取openid
     * @param code
     * @return
     */
    @Override
    public String getOpenid(String code) {

        // 获取公共参数
        Map<String,Object> paramMap = getAppConfig();
        paramMap.put("js_code", code);

        String result = HttpUtil.get(REQUEST_URL, paramMap);
        // 结果是一个map
        JSONObject jsonObject = JSONUtil.parseObj(result);
        // 判断接口响应是否出错
        if(ObjectUtil.isNotEmpty(jsonObject.getInt("errcode"))) {
            throw new RuntimeException(jsonObject.getStr("errmsg"));
        }

        String openid = jsonObject.getStr("openid");

        return openid;
    }

    /**
     * 封装公共参数
     * @return
     */
    private Map<String, Object> getAppConfig() {
        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("appid", appid);
        paramMap.put("secret", secret);
        return paramMap;
    }

    /**
     * 获取手机号
     * @param detailCode
     * @return
     */
    @Override
    public String getPhone(String detailCode) {

        String token = getToken();
        String url = PHONE_REQUEST_URL + token;
        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("code", detailCode);
        // 发起请求
        String result = HttpUtil.post(url, JSONUtil.toJsonStr(paramMap));
        // 是一个map
        JSONObject jsonObject = JSONUtil.parseObj(result);
        // 判断接口响应是否出错
        if(jsonObject.getInt("errcode") != 0) {
            throw new RuntimeException(jsonObject.getStr("errmsg"));
        }

        return jsonObject.getJSONObject("phone_info").getStr("phoneNumber");
    }

    /**
     * 获取token
     * @return
     */
    private String getToken() {

        Map<String, Object> paramMap = getAppConfig();
        // 发起请求
        String result = HttpUtil.get(TOKEN_URL, paramMap);
        // 是一个map
        JSONObject jsonObject = JSONUtil.parseObj(result);
        // 判断接口响应是否出错
        if(ObjectUtil.isNotEmpty(jsonObject.getInt("errcode"))) {
            throw new RuntimeException(jsonObject.getStr("errmsg"));
        }

        String token = jsonObject.getStr("access_token");

        return token;

    }
}
```

由于上述代码需要读取配置来获取微信小程序的appid和appSecret  ，所以需要在application.yml添加对应配置，这里大家注意，这个要跟微信开发者平台设置的相同，就是我们刚才自己申请的测试小程序的appid和appSecret  

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/MISHbjkWpoRHl8xPAyPcuGTKnre.png)

大家注意格式：**wechat.appId**

**注意**，**使用自己申请的appId和secret，不然小程序无法登录**



单元测试：

在zzyl-admin中创建单元测试WechatServiceImplTest 

- 在父工程添加单元测试的起步依赖

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>${spring-boot.version}</version>
</dependency>
```

- 在zzyl-admin中添加起步依赖

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

```Java
package com.zzyl.test;

import com.zzyl.nursing.service.WechatService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class WechatServiceImplTest {

    @Autowired
    private WechatService wechatService;

    @Test
    public void testGetOpenid() {
        String openid = wechatService.getOpenid("0d3fPOkl2VLGbe4IACol2M0x5Q2fPOk5");
        System.out.println(openid);//o3CsK6_C6f4WP9b0AxXNJOkc6q9Q
    }

    @Test
    public void testGetPhone() {
        String phone = wechatService.getPhone("ff1a2f2e61482537ea19c8e57850919051b43ffadbf78cd4fcd3a1707e0c1e14");
        System.out.println(phone);
    }
}
```

参数获取方式：在小程序端获取参数，输入到单元测试中

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/P1vKbIFBzoDKcGxV2tlcL9YEnmg.png)



##### 微信登录业务开发

在IFamilyMemberService中定义login方法

```Java
/**
 * 微信登录
 * @param userLoginRequestDto
 * @return
 */
LoginVo login(UserLoginRequestDto userLoginRequestDto);
```

实现方法：

```Java
@Autowired
private WechatService wechatService;

@Autowired
private TokenService tokenService;

static List<String> DEFAULT_NICKNAME_PREFIX = ListUtil.of(
        "生活更美好",
        "大桔大利",
        "日富一日",
        "好柿开花",
        "柿柿如意",
        "一椰暴富",
        "大柚所为",
        "杨梅吐气",
        "天生荔枝"
);

/**
 * 小程序端登录
 * @param userLoginRequestDto
 * @return
 */
@Override
public LoginVo login(UserLoginRequestDto userLoginRequestDto) {
    // 1.调用微信api,根据code获取openId
    String openId = wechatService.getOpenid(userLoginRequestDto.getCode());

    // 2.根据openId查询用户
    FamilyMember familyMember = getOne(Wrappers.<FamilyMember>lambdaQuery(FamilyMember.class)
            .eq(FamilyMember::getOpenId, openId));

    // 3.如果用户为空，则将openId设置到对象中
    if (ObjectUtil.isEmpty(familyMember)) {
        familyMember = FamilyMember.builder().openId(openId).build();
    }

    // 4.调用微信api获取用户绑定的手机号
    String phone = wechatService.getPhone(userLoginRequestDto.getPhoneCode());

    // 5.保存或修改用户
    saveOrUpdateFamilyMember(familyMember, phone);

    // 6.将用户id存入token,返回
    Map<String, Object> claims = new HashMap<>();
    claims.put("userId", familyMember.getId());
    claims.put("nickName", familyMember.getName());

    String token = tokenService.createToken(claims);
    LoginVo loginVo = new LoginVo();
    loginVo.setToken(token);
    loginVo.setNickName(familyMember.getName());
    return loginVo;
}

/**
 * 保存或修改客户
 * @param member
 * @param phone
 */
private void saveOrUpdateFamilyMember(FamilyMember member, String phone) {

    // 1.判断取到的手机号与数据库中保存的手机号不一样
    if(ObjectUtil.notEqual(phone, member.getPhone())) {
        // 修改手机号
        member.setPhone(phone);
    }
    // 2.判断id存在，则修改
    if (ObjectUtil.isNotEmpty(member.getId())) {
        updateById(member);
        return;
    }
    // 3.id不存在，保存新用户
    // 随机组装昵称，词组+手机号后四位
    String nickName = DEFAULT_NICKNAME_PREFIX.get((int) (Math.random() * DEFAULT_NICKNAME_PREFIX.size()))
            + StringUtils.substring(member.getPhone(), 7);

    member.setName(nickName);
    save(member);
}
```

**其他补充**

1. 在构建Member对象的时候，用到了构建则设计模式，参考文档：<cite doc-id="GeOEwrD5pi6FgnkQAAcccqHvnAb" file-type="wiki" title="构建者设计模式" type="doc"></cite>

在FamilyMember添加三个注解，如下图：

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/V1OKbrCsJoG8ejxlAEXc7HgRnOc.png)

1. 使用TokenService生成token

在zzyl-nursing-platform中导入zzyl-framework包

```XML
<dependency>
    <groupId>com.zzyl</groupId>
    <artifactId>zzyl-framework</artifactId>
</dependency>
```

创建JWT的token的时候，可以使用项目中自带的`TokenService`中的方法`createToken`该方法的访问权限为`private`，需要将其设置为`public`

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/NeejbYviZoH1MexuplbcaAm7nle.png)

1. 修改MP自动填充拦截器

由于小程序端的所有请求不走后台的用户，在新增或修改的时候，不需要自动填充创建人和修改人

```Java
package com.zzyl.framework.interceptor;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import com.zzyl.common.core.domain.model.LoginUser;
import com.zzyl.common.utils.SecurityUtils;
import lombok.SneakyThrows;
import org.apache.commons.lang3.ObjectUtils;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Autowired
    private HttpServletRequest request;

    @SneakyThrows
    public boolean isExclude() {
        String requestURI = request.getRequestURI();
        if(requestURI.startsWith("/member")) {
            return true;
        }
        return false;
    }

    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", Date.class, new Date());
        if(!isExclude()) {
            this.strictInsertFill(metaObject, "createBy", String.class, loadUserId() + "");
        }

    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("updateTime", new Date(), metaObject);
        if(!isExclude()) {
            this.setFieldValByName("updateBy", loadUserId() + "", metaObject);
        }

    }

    /**
     * 获取当前登录人的ID
     *
     * @return
     */
    private static Long loadUserId() {

        // 获取当前登录人的id
        try {
            LoginUser loginUser = SecurityUtils.getLoginUser();
            if (ObjectUtils.isNotEmpty(loginUser)) {
                return loginUser.getUserId();
            }
            return 1L;
        } catch (Exception e) {
            return 1L;
        }
    }
}
```



#### 前后端联调测试

代码都开发完毕之后，我们可以继续打开小程序，找到**我的，**就可以验证是否能登录成功，详细操作如下图

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/INUWbaIK0oKl3exXE0lcDhnmnKe.png)



### 校验token

#### 思路分析

通过我们刚才的流程，我们知道，用户登录成功以后，会给前端返回一个token，这个token就是来验证用户信息的，当用户点击了小程序中的其他操作（需要登录），则会把token携带到请求头（header）中，方便后台去验证并获取用户信息，简易流程如下

<whiteboard token="FERpwziimhs60abmXQEcBoWWnEd"></whiteboard>

如果想要验证用户的token，我们可以使用自定义的拦截器实现，整体的流程如下：

<whiteboard token="O8IMwiwUfh3DfBbcjaCc1dFXnhf"></whiteboard>

#### Threadlocal的使用

详细请查看文档：<cite doc-id="FqSkwQ0HCiU7hckIHlfcwW0pnoe" file-type="wiki" title="ThreadLocal" type="doc"></cite>

把资料中提供的工具类UserThreadLocal拷贝到zzyl-common模块utils包下

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/IUawbO8BkodzdLxnbVVcQG4Wn8b.png)



#### 功能实现

在zzyl-framework模块中定义拦截器MemberInterceptor

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/SteqbVaoCoQls5x92vqc5pW0nSe.png)



代码如下：

```Java
package com.zzyl.framework.interceptor;

import cn.hutool.core.map.MapUtil;
import cn.hutool.core.util.ObjectUtil;
import com.zzyl.common.exception.base.BaseException;
import com.zzyl.common.utils.StringUtils;
import com.zzyl.common.utils.UserThreadLocal;
import com.zzyl.framework.web.service.TokenService;
import io.jsonwebtoken.Claims;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class MemberInterceptor implements HandlerInterceptor {
    private final TokenService tokenService;

    public MemberInterceptor(TokenService tokenService) {
        this.tokenService = tokenService;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        // 如果不是Controller层的请求，直接放行
        if (!(handler instanceof HandlerMethod)) {
            return true;
        }

        // 获取header中的token
        String token = request.getHeader("authorization");

        // 如果token为空，响应401，重新登录
        if (StringUtils.isEmpty(token)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            throw new BaseException("认证失败");
        }

        // 解析token
        Claims claims = tokenService.parseToken(token);
        if (ObjectUtil.isEmpty(claims)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            throw new BaseException("认证失败");
        }

        // 解析token中的数据
        Long userId = MapUtil.get(claims, "userId", Long.class);
        if (ObjectUtil.isEmpty(userId)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            throw new BaseException("认证失败");
        }

        // 将用户id放入ThreadLocal中
        UserThreadLocal.set(userId);

        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        UserThreadLocal.remove();
    }
}
```

注意，在上述代码中我们使用了tokenService来解析token，其中的parseToken方法需要修改为**public**才可以



**使拦截器生效**

找到ResourcesConfig类，这里面定义了拦截器的生效规则，如下图

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/O2kBbUzUloixzixmRPzck1jFnz2.png)

详细的代码如下：

```Java
package com.zzyl.framework.config;

import java.util.concurrent.TimeUnit;

import com.zzyl.framework.interceptor.MemberInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.CacheControl;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import com.zzyl.common.config.RuoYiConfig;
import com.zzyl.common.constant.Constants;
import com.zzyl.framework.interceptor.RepeatSubmitInterceptor;

/**
 * 通用配置
 * 
 * @author ruoyi
 */
@Configuration
public class ResourcesConfig implements WebMvcConfigurer
{
    @Autowired
    private RepeatSubmitInterceptor repeatSubmitInterceptor;

    @Autowired
    private MemberInterceptor membersInterceptor;

    private static final String[] EXCLUDE_PATH_PATTERNS = new String[] {
            "/member/user/login",
            "/member/roomTypes"
    };

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry)
    {
        /** 本地文件上传路径 */
        registry.addResourceHandler(Constants.RESOURCE_PREFIX + "/**")
                .addResourceLocations("file:" + RuoYiConfig.getProfile() + "/");

        /** swagger配置 */
        registry.addResourceHandler("/swagger-ui/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/springfox-swagger-ui/")
                .setCacheControl(CacheControl.maxAge(5, TimeUnit.HOURS).cachePublic());;
    }

    /**
     * 自定义拦截规则
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry)
    {
        registry.addInterceptor(repeatSubmitInterceptor).addPathPatterns("/**");
        registry.addInterceptor(membersInterceptor).excludePathPatterns(EXCLUDE_PATH_PATTERNS).addPathPatterns("/member/**");
    }

    /**
     * 跨域配置
     */
    @Bean
    public CorsFilter corsFilter()
    {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        // 设置访问源地址
        config.addAllowedOriginPattern("*");
        // 设置访问源请求头
        config.addAllowedHeader("*");
        // 设置访问源请求方法
        config.addAllowedMethod("*");
        // 有效期 1800秒
        config.setMaxAge(1800L);
        // 添加映射路径，拦截一切请求
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        // 返回新的CorsFilter
        return new CorsFilter(source);
    }
}
```

想要测试当前拦截器是否生效，可以随便定义一个控制层代码（需符合拦截要求的），测试代码如下：

```Java
package com.zzyl.nursing.controller.member;

import com.zzyl.common.core.controller.BaseController;
import com.zzyl.common.core.domain.R;
import com.zzyl.common.utils.UserThreadLocal;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 预约信息Controller
 * 
 * @author ruoyi
 */
@RestController
@RequestMapping("/member/reservation")
@Api(tags =  "预约信息相关接口")
public class MemberReservationController extends BaseController
{
    @GetMapping("/cancelled-count")
    @ApiOperation("查询取消预约数量")
    public R<Integer> getCancelledReservationCount() {
        Long userId = UserThreadLocal.getUserId();
        System.out.println("------" + userId);
        return R.ok(1);
    }
}
```

以上代码是我们马上要开发的预约管理中的一个需求，这个请求路径是符合拦截需求的，大家可以使用在线接口文档测试或者是直接在小程序中使用，在登录成功以后，可以点击”参观预约”按钮测试，然后查看idea控制台，是否输出了用户的id，如果可以成功输出，则表示成功

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/IX8bbP21SoG6LMxeqczcGE3Onjc.png)



## 作业

- 完成微信登录代码，和小程序端联调测试通过
- 参考<cite doc-id="XkWhwZzx6ieXsmkWYAhcXLr4n8g" file-type="wiki" title="01-IOT产品设备管理" type="doc"></cite>开通华为云物联网平台，完成2.3中的所有步骤

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-san-zhang-ai-fu-zhu-kuai-su-wan-cheng-deng-lu-ren-zheng/Bik7bXy6JoP8gMxPKZ0cdbsxnPe.png)

- 完成<cite doc-id="VTvtwnexJip9y8kYiTUcgpq1nCG" file-type="wiki" title="【作业】护理项目和预约" type="doc"></cite>中的 `1.服务` 中的需求，如果都完成了，可以继续往后做

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第三章-AI辅助快速完成登录认证 / 01-后台认证授权、小程序登录 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/RvOGwZPn9iXPBMkCePVcPX4Tngb)
