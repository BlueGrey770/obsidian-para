---
type: memo
date: 2026-11-06
aliases:
  - "11-deng-lu-hui-hua-token"
  - "11 登录_会话_token"
---
# 11 登录_会话_token

<callout emoji="📚">
登录的本质：就是一个查询

 1. 用户名和密码是否正确：密码当时存的时候是密文存储，我们必须将登录的密码也加密再去    比对才行

       2. 用户的状态：比如是可用的状态才行
</callout>

```Java
User loginUser = userMapper.Login(user.getUsername());
//判断用户是否存在
if (loginUser == null){
    log.info("用户不存在");
    throw new CustomException("用户名或密码错误");
}
//判断密码是否正确
if (!loginUser.getPassword().equals(DigestUtils.md5DigestAsHex(user.getPassword().getBytes()))){
    log.info("密码错误");
    throw new CustomException("用户名或密码错误");
}

//提示账户状态是否可用
if(loginUser.getStatus() ==0){
    log.info("账户Status=0");
    throw new CustomException("账户已停用");
}
```

## 如何将登录的信息保存

### 会话技术

1. **什么是会话技术：解决一次会话多次请求中数据共享的问题**

#### COOKIE

```Java
原理：将数据存储到浏览器中了，浏览器每次请求都会携带cookie，这样服务器就可以得到携带的数据了
     身份认证
优点：http协议自带的技术，使用起来简单
缺点：1. 不安全
     2. 数据大小有限制，一般是4kb左右
     3. 一个站点一般在浏览中只能存在20个左右的cookie
     4. 只能存储键值对的数据而且不支持特殊字符
     5. 不能跨域
     6. 不支持移动端
```

#### SESSION

```Java
原理：在服务器中创建session对象存储，但是如果第一次请求session，会检查是否携带了JSESSIONID
     的cookie这把钥匙，如果没有携带，则判断是第一次请求session，那么会创建新的session对象
     并且会自动将这个seesion的id易cookie的形式，键为JSESSIONID,值为session的id存储到客
     客户端浏览器中，下次再次请求session带有钥匙的话，那么他会找到原来的session对象，下次再次请求
     session带有钥匙的话那么他会找到原来的session对象从而去到之前保存的数据达到共享的目的
优点：服务器自带的技术使用起来也简单
     数据较为安全
缺点：不能跨域
     不支持移动端
```

#### token

```Java
令牌：身份识别的一种机制，而且可以携带数据
组成部分：
       1. header:令牌的类型：JWT
                 加密方式：HS256
       2. payload：载荷：携带的数据部分
       3. signature：数字签名：结合加密方式一种识别
编码原理：底层采用BASE64的编码方式，将内容编码：
        1. 组成部分：a-z,A-Z,0-9，+/，=（补位）
        采用六位二进制0-63从base64取对应的字符完成的，如果最后不够6位需要使用=补位
```

**制作令牌的步骤**

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
  secret: itheima
```

1. 在service层中使用token工具类

<callout emoji="🥛">
根据需要的登录信息（也就是在头部显示的用户信息）来定义VO类
</callout>

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
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / 会话与登录认证 / 11 登录_会话_token · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/LzD7wI23Diw1eukBCjIcR9sinch)
