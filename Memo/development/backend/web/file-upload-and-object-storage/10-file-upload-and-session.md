---
type: memo
date: 2026-11-06
aliases:
  - "10-wen-jian-shang-chuan-hui-hua"
  - "10 文件上传_会话"
---
# 10 文件上传_会话

## 文件上传

### 前端必须遵守的规则

```Java
1. 将请求方法改成post
2. 提交参数的方式的为 enctype = "multipart-form-data"
   数据会被一个特殊的分割线隔开
3.必须有上传项<input type = file name="和controller的参数名保持一致">
```

### 后端

```Java
1. MultipartFile：采用这个类型接收上传即可
2. 如何管理上传文件：
        1. 自己管理
        2. 第三方管理：OSS云存储：引入SDK，配置accesskey
        3. 从官网中去拿使用案例
3. 解决问题：
    1. 文件名重名问题：我们采用UUID.randomUUID();
    2. 解决一个文件夹存放大量文件问题：采用hash算法，得到8级目录
```

## 会话技术

### cookie

```Java
定义：客户端（浏览器）存储的小型文本文件，记录用户状态，由服务器下发，浏览器保存，请求时自动携带
优点：Http协议自带的技术,使用简单,存储在客户端，减轻服务端压力，可长期保存（设置有效期）适合记住登录状态等场景。
原理:将数据存储到浏览器中,浏览器每次请求都会携带,cookie，这样服务器就可以得到携带的数据，身份认证
缺点：大小首先（约4KB），一个站点一般在浏览器中只能存在20个左右的cookie,仅存字符串，易被篡改，有跨域限制，隐私性差。
    (1)不安全
    (2)数据大小有限制，一般是4kb大小
    (3)一个站点一般在浏览中只能存在20个左右的cookie
    (4)只能存储键值对的数据，而且不支持特殊字符
    (5）不能跨域
    (6）不支持移动端
```

### SESSION

```Java
定义:服务端存储的用户会话数据，通过Cookie或者URL重写关联客户端，会话结束销毁
优点：服务器自带的技术,使用较为简单,存储在服务端，数据安全，支持复杂数据类型，无存储大小限制
原理:
    在服务器中创建Session对象存储，但是如果第一次请求session，会检查是否携带了JSESSIONID的cookie这把钥
匙，如果没有携带，则判断是第一次请求session，那么会创建新的session对象，并且会自动将这个session的id以
cookie的形式，键为JSESSIONID，值为session的id存储到客户端浏览器中，下次再次请求session带有钥匙的话，那
么他会找到原来的session对象从而去到之前保存的数据达到共享的目的
缺点：占用服务端内存，高并发下性能压力大，分布式场景需要额外处理；会话超时易导致用户状态丢失
    (1) 不能跨域
    (2)不支持移动端
```

### JWT（令牌）

```Java
定义：基于JSON的轻量级令牌，用于在客户端与服务端间安全传递身份/权限等声明信息，由头部，载荷，签名
    三部分组成，通过Base64编码+加密签名生成，可自包含验证
组成部分:
    header:
        令牌类型:JWT
        加密方式:HS256
    payload:
        载荷:携带数据部门
    signature:
        数字签名:结合加密方式的一种识别
原理:
    底层采用BASE64的编码方式,将内容编码
        组成部分:
            a-z    A-Z    0-9    +/    
            = :补位
    采用六位二进制0~63从base64取对应的字符完成的如果最后不够6位需要使用=补位
    
后台:
    制作令牌:
            String token = Jwts.builder()
                .signWith(SignatureAlgorithm.HS256,secret.getBytes())  // 使用HS256算法和密钥签名
                .setClaims(claims)
                .setExpiration(expiration)  // 设置过期时间
                .compact();// 生成最终的JWT字符串
        return token;
        
    下发令牌:将令牌以相应的方式给到前端
    校验(解析)令牌:
    Claims claims = Jwts.parser()
        .setSigningKey(secret.getBytes())  // 设置签名密钥
        .parseClaimsJws(token)  // 解析令牌
        .getBody();
    return claims;
    VO:
        View Object: 专门对接前端要求的数据格式定义的实体类,我们采用VO结尾

前端:
    保存令牌:
        不能cookie保存,采用localStong的方式保存,这种方式是移动端和PC端都支持的
        前端自己写代码才行
    发送令牌:每次请求都要携带令牌,需要写逻辑
(令牌种类太多,不需要专门记忆,只需学会能查到会调试 调用即可)
优点：无状态，支持跨域/分布式系统，传输体积小，解决快，签名防篡改，可附加自定难以业务信息，如用户ID，权限
缺点：令牌一旦签发无法主动作废，易解码，不可存敏感信息，令牌过长可能增加传输开销。
典型应用场景：前后端分离，微服务鉴权，跨系统单点登录，替代传统SESSION实现无状态认证。
```

<callout emoji="🏕️">
注意区别:
加密是不可逆的,编码是可逆的
</callout>

#### **制作令牌的步骤**

1. 在pom.xml文件中引入JWT的坐标

```XML
<!-- JWT依赖-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>${jjwt.version}</version>
</dependency>
```

1. 在公共使用模块中定义获取token的工具类

```TypeScript
@Component
public class TokenUtils {
    @Value("${jwt.secret}")
    private  String secret;
    
    public String makeToken(Map<String, Object> claims, Date expiration) {

        String token = Jwts.builder()
                .signWith(SignatureAlgorithm.HS256, secret.getBytes())  // 使用HS256算法和密钥签名
                .setClaims(claims)
                .setExpiration(expiration)  // 设置过期时间
                .compact();// 生成最终的JWT字符串
        return  token;
    }

    public Map<String, Object> parseToken(String token) {

        Claims claims = Jwts.parser()
                .setSigningKey(secret.getBytes())  // 设置签名密钥
                .parseClaimsJws(token)  // 解析令牌
                .getBody();
        return claims;
    }
}
```

1. 在yml配置文件中设置令牌密钥

```Java
jwt:
  secret: bluegrey777
```

1. 在service层中使用token工具类

<callout emoji="🥛">
根据需要的登录信息（也就是在头部显示的用户信息）来定义VO类
</callout>

Token使用代码示例:

```JavaScript
//给loginVO赋值
LoginVO loginVO = new LoginVO();
loginVO.setId(loginUser.getId());
loginVO.setUserName(loginUser.getUsername());
loginVO.setName(loginUser.getName());
loginVO.setImage(loginUser.getImage());
loginVO.setRoleLabel(loginUser.getRoleLabel());

//开始造JWT
HashMap<String, Object> clims = new HashMap<>();
clims.put("id",loginUser.getId());
clims.put("username",loginUser.getUsername());
clims.put("name",loginUser.getName());
clims.put("image",loginUser.getImage());
clims.put("roleLabel",loginUser.getRoleLabel());
//设置token的过期时间
String token = tokenUtils.makeToken(clims,new Date(System.currentTimeMillis() + 1000 * 60*60*24*7));
//将VO放进token中
loginVO.setToken(token);
```

<callout emoji="🦄">
在上一步是在后端操作的使用JWT制作了令牌之后还有后续的步骤
前端还要保存令牌和发送令牌
</callout>

```Java
保存令牌：不能cookie保存，采用localStorge的方式保存，这种方式是移动端和pc端都支持的
         前端自己需要写代码才行
发送令牌：每次请求都要携带令牌，需要写逻辑
```

## 登录验证

<callout emoji="❤️">
就是如果没有登录的话就拦截其对URI资源的请求，使其不能访问，只有登录之后才可以访问内部页面
</callout>

### 过滤器

<callout emoji="🌰">
介绍：web的三大组件之一，是服务器内置的技术，他是在请求达到servlet之前做的一些统一操作用的
</callout>

#### 使用步骤

> 1. 定义一个类去实现Filter接口，将其放入到ioc容器中即可
> 2. 重写doFilter的方法
> 3. 在doFilter中编写过滤逻辑适当的放行：比如登录页面和注册页面就不需要拦截，所有人都可以无条件访问

```Java
@Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest) servletRequest;
    HttpServletResponse response = (HttpServletResponse) servletResponse;
    String uri = request.getRequestURI();
    if (uri.equals("/login")|| uri.equals("/register")){
        filterChain.doFilter(servletRequest,servletResponse);
        return;
    }

    String token = request.getHeader("token");
    if (token == null){
        response.setStatus(401);
        return;
    }

    try {
        tokenUtils.parseToken(token);
    } catch (Exception e) {
        response.setStatus(401);
        return;
    }

    filterChain.doFilter(request,response);


}
```

1. 细节

```Java
多个过滤器的执行顺序是由过滤器的名字的字典顺序有关，如果不想跟字典顺序有关系，可以通过@Order(2...)建议从2开始
    因为系统的过滤器用的是1
    不仅请求过滤器，响应也会经过过滤器
```

### 拦截器Intercepter

<callout emoji="💡">
介绍：是SpringMVC技术，他是DishpatherServlet之后执行的Controller之前执行的
使用步骤：1.定义一个类取实现HandlerInterceptor接口
          2. 选择从写三个方法preHandle,postHandle,afterConpletion三个方法
                3. 配置拦截器
</callout>

#### 使用步骤

1. 首先定义一个拦截器类

```Java
 @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("拦截器生效");
        //获取token
        String token = request.getHeader("token");
        //检验token
        if (token == null) {
            response.setStatus(401);
            return false;
        }
        //解析token 看是否有效
        try {
            //解析并检验token
            tokenUtils.parseToken(token);
        } catch (Exception e) {
            response.setStatus(401);
            return false;
        }

        return HandlerInterceptor.super.preHandle(request, response, handler);
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
```

1. 定义拦截器的配置类  
用来指定拦截和放行的资源

```Java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册拦截器
        registry.addInterceptor(myInteceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/login","/register");
        WebMvcConfigurer.super.addInterceptors(registry);
    }
}
```

<callout emoji="❤️">
postHandle和afterCompletion的区别：
1. postHandle但凡有任何一个拦截器return false，则所有的postHandle都不会走
2. afterCompletion：当前的拦截器的afterCompletion不会走，已经走过的拦截器的所有afterCompletion都会走
</callout>

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / 文件上传与对象存储 / 10 文件上传_会话 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/BkxJwIMgjiOkJOkuHWac5hYgn1e)
