---
type: memo
date: 2026-11-06
aliases:
  - "14 AOP"
---
# 14 AOP

## Mybatis-Plus的原理

<callout emoji="🐵">
增强IOC容器中的bean
</callout>

### 步骤

<callout emoji="👍">
1.定义一个类是实现BeanPostProcessor接口
2.重写两个方法
postProcessBeforelenitialization
postProcessAfterInitialization

bean初始化之后进行加强

`BaseMapperEnhance.java`
3.创建代理对象,增强类中的功能
</callout>

示例代码:

```Java
package cn.itcast.day14.mp;

import cn.itcast.day14.aspect.MyAspect1;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.mapping.ResultSetType;
import org.apache.ibatis.mapping.SqlCommandType;
import org.apache.ibatis.mapping.SqlSource;
import org.apache.ibatis.scripting.defaults.RawSqlSource;
import org.apache.ibatis.session.Configuration;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
@Component
public class BaseMapperEnhance implements BeanPostProcessor {
    @Autowired
    private SqlSessionFactory sqlSessionFactory;
    @Autowired
    private SqlSession sqlSession;
    @Autowired
    private MyAspect1 myAspect1;
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof BaseMapper<?>) {
            return createMapper(bean);
        }/*else if(bean instanceof B){
            return createA(bean);
        }*/
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }

    private Object createA(Object bean) {
        ;
        return Proxy.newProxyInstance(
                bean.getClass().getClassLoader(),
                bean.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        // 解析切入点
                        //将原有切入进行封装成PJP
                        ProceedingJoinPointMJJ pjp = new ProceedingJoinPointMJJ();
                        pjp.setArgs(args == null? new Object[0]: args);
                        pjp.setTarget(bean);
                        pjp.setMethod(method);
                        Singenature singenature = new Singenature();
                        singenature.setName(method.getName());
                        singenature.setTypeName(bean.getClass().getName());
                        pjp.setSingenature(singenature);

                       // myAspect1.around(pjp);
                        return null;
                    }
                }
        );
    }

    public Object createMapper(Object bean) {
        return Proxy.newProxyInstance(
                bean.getClass().getClassLoader(),
                bean.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        if (method.getName().equals("insert")) {
                            Object arg = args[0];
                            Class<?> clazz = arg.getClass();
                            TableName annotation = clazz.getAnnotation(TableName.class);
                            String tableName;
                            if (annotation != null) {
                                tableName = annotation.value();
                            } else {
                                tableName = clazz.getSimpleName();
                            }
                            //insert into 表名 (列名1, 列名2,.....) values( #{}.#)
                            StringBuilder sb = new StringBuilder("insert into ");
                            sb.append(tableName).append(" (");
                            //获取类的所有的属性
                            Field[] fields = clazz.getDeclaredFields();
                            for (int i = 0; i < fields.length; i++) {
                                String name = fields[i].getName();

                                if (i == fields.length - 1) {
                                    sb.append(name).append(") values (");
                                } else {
                                    sb.append(name).append(",");
                                }
                            }
                            for (int i = 0; i < fields.length; i++) {
                                String name = fields[i].getName();
                                if (i == fields.length - 1) {
                                    sb.append("#{").append(name).append("})");
                                } else {
                                    sb.append("#{").append(name).append("},");
                                }
                            }
                            String sql = sb.toString();

                            // 获取Configuration
                            Configuration configuration = sqlSessionFactory.getConfiguration();
                            // 创建SQL源
                            SqlSource sqlSource = new RawSqlSource(configuration,
                                    sql,
                                    clazz);
                            // 创建MappedStatement
                            String statementId = clazz.getName() + "." + method.getName();
                            MappedStatement ms = new MappedStatement.Builder(configuration, statementId, sqlSource, SqlCommandType.INSERT)
                                    .resultSetType(ResultSetType.DEFAULT)
                                    .build();

                            // 注册到Configuration（需要处理已存在的情况）
                            configuration.addMappedStatement(ms);
                            return sqlSession.insert(statementId, arg);
                        }
                        return method.invoke(bean, args);
                    }
                }
        );
    }
}

```



## AOP

### 作用

<callout emoji="🦄">
不改变源码的情况下对IOC容器中的功能增强
</callout>

AOP的优势主要体现在以下四个方面：

- 减少重复代码：不需要在业务方法中定义大量的重复性的代码，只需要将重复性的代码抽取到AOP程序中即可。
- 代码无侵入：在基于AOP实现这些业务功能时，对原有的业务代码是没有任何侵入的，不需要修改任何的业务代码。
- 提高开发效率
- 维护方便

<callout emoji="📚">
AOP是一种思想，而在Spring框架中，对这种思想进行了实现
</callout>

### 使用步骤

#### 导入aop的起步依赖

- 在 pom.xml 中引入AOP的依赖

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

#### 在启动类上面开启AOP自动代理

在主程序启动类中开启AOP自动代理`@EnableAspectJAutoProxy`

```TypeScript
@SpringBootApplication
@EnableAspectJAutoProxy//开启AOP的自动代理
public class Day14AopApplication {
    public static void main(String[] args) {
        SpringApplication.run(Day14AopApplication.class, args);
    }
}
```

#### 编写切面类

先在类名上注解`@Aspect` 标注是一个切面类 再添加@Component注解 加入IOC容器中

```Java
@Aspect// 标注是一个切面类
@Component//加入到IOC容器中
public class MyAspect {
}
```

AOP切面类书写步骤:

```XML
将 “增强逻辑” 以指定的 “注入方式" 织入到 “要增强的方法中"
    通知              织入类型            切入点
```

代码示例:

```Java
@Aspect
@Component
public class LoggingAspect {
    
    // 切入点表达式示例
    @Before("execution(public * com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("方法执行前: " + joinPoint.getSignature().getName());
    }
}
```

##### 切入点表达式详解

表达式:

```Java
"execution([修饰符] 返回值类型 包名.类名.方法名(参数类型列表))"
```



##### 通配符说明

```Java
*：匹配任意内容
    返回值类型：*匹配任意类型
    包/类/方法：匹配单个层级
..：多层匹配
    包路径：com..service匹配com包下任意层级的service包
    参数列表：(..)匹配任意个数和类型的参数
```

<callout emoji="🚅">
注意:
尽量包不要使用`..`描述,性能太低 不推荐
</callout>

表达式示例

```JSON
// 精确匹配
"execution(public String com.example.UserService.getName())"

// 通配匹配
"execution(* com.example.service.*.*(..))"          // service包下所有方法
"execution(* com.example..service.*.*(..))"         // 任意层级下的service包
"execution(* *.*(..))"                              // 所有方法（不推荐，性能差）
```

<callout emoji="🦄">
使用场景
名字统一的情况下,可以采用通配符批量增强
名字不统一的时候使用注解方式比较适合
</callout>

注解使用方式

```Java
"@annotation(注解的全类名)"
```

##### 五种通知类型

在通知方法上加上了`@Around`注解，就代表当前通知是一个环绕通知。

<table><colgroup><col/><col/></colgroup><thead><tr><th colspan="2"><b>Spring AOP 通知类型</b></th></tr></thead><tbody><tr><td>@Around</td><td>环绕通知，此注解标注的通知方法在目标方法前、后都被执行 (最常用)</td></tr><tr><td>@Before</td><td>前置通知，此注解标注的通知方法在目标方法前被执行</td></tr><tr><td>@After</td><td>后置通知，此注解标注的通知方法在目标方法后被执行，无论是否有异常都会执行</td></tr><tr><td>@AfterReturning</td><td>返回后通知，此注解标注的通知方法在目标方法后被执行，有异常不会执行</td></tr><tr><td>@AfterThrowing</td><td>异常后通知，此注解标注的通知方法发生异常后执行</td></tr></tbody></table>

###### @Before - 前置通知

```Java
@Before("execution(* com.example.service.*.*(..))")
public void beforeAdvice(JoinPoint joinPoint) {
    System.out.println("方法执行前: " + joinPoint.getSignature().getName());
}
```

###### @After - 后置通知

```Java
@After("execution(* com.example.service.*.*(..))")
public void afterAdvice(JoinPoint joinPoint) {
    System.out.println("方法执行完成: " + joinPoint.getSignature().getName());
}
```

###### @AfterReturning - 返回通知

```TypeScript
@AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", returning = "result")
public void afterReturning(JoinPoint joinPoint, Object result) {
    System.out.println("方法返回: " + result);
}
```

###### @AfterThrowing - 异常通知

```Java
@AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))", throwing = "ex")
public void afterThrowing(JoinPoint joinPoint, Exception ex) {
    System.out.println("方法异常: " + ex.getMessage());
}
```

###### @Around - 环绕通知（功能最全）

```Java
@Around("execution(* com.example.service.*.*(..))")
public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {
    try {
        // 前置处理
        System.out.println("方法执行前");
        
        // 获取和修改参数
        Object[] args = pjp.getArgs();
        if (args != null && args.length > 0) {
            // 参数处理逻辑
        }
        
        // 执行目标方法
        Object result = pjp.proceed(args);
        
        // 返回处理
        System.out.println("方法返回后");
        return result;
        
    } catch (Exception e) {
        // 异常处理
        System.out.println("方法异常");
        throw e;
    } finally {
        // 最终处理
        System.out.println("最终执行");
    }
}
```

环绕通知示例代码:

```TypeScript
/**
 * 自定义切面类 - 演示Spring AOP环绕通知的使用
 * 本切面会对cn.itcast.pojo.TestAspect类中的所有方法进行增强
 * 
 * @Component 注解将该类声明为Spring容器管理的Bean
 * @Aspect 注解声明该类是一个切面类
 */
@Aspect
@Component
public class MyAspect1 {
    
    /**
     * 环绕通知 - 在目标方法执行前后进行拦截和增强
     * 
     * @param pjp 连接点对象，用于控制目标方法的执行
     * @return 目标方法的返回值（可能被增强修改）
     * 
     * @Around 注解声明该方法为环绕通知
     * execution(* cn.itcast.pojo.TestAspect.*(..)) 切入点表达式：
     *   - 匹配cn.itcast.pojo.TestAspect类中的所有方法
     *   - 任何返回类型、任何方法名、任意参数列表
     */
    @Around("execution(* cn.itcast.pojo.TestAspect.*(..))")
    public Object around(ProceedingJoinPoint pjp) {
        try {
            // 前置通知：在目标方法执行前执行
            System.out.println("我是前置通知");            
            // 获取目标方法的参数
            Object[] args = pjp.getArgs();
            // 如果只有一个参数且是String类型，则去除两端空格
            if (args.length == 1 && args[0] instanceof String) {
                args[0] = args[0].toString().trim();
            }            
            // 获取目标方法签名信息
            Signature signature = pjp.getSignature();
            // 获取方法名
            String name = signature.getName();
            System.out.println("方法名：" + name);            
            // 执行目标方法（可传入修改后的参数）
            // proceed()会调用目标方法，并返回其执行结果
            Object proceed = pjp.proceed(args);            
            // 返回通知：在目标方法成功执行后执行
            System.out.println("我返回通知");            
            // 如果返回值是String类型，则追加额外文本
            if (proceed != null && proceed instanceof String) {
                return proceed + "，这是返回通知";
            }            
            // 返回目标方法的原始返回值或修改后的返回值
            return proceed;            
        } catch (Throwable e) {
            // 异常通知：当目标方法抛出异常时执行
            System.out.println("我是异常通知");
            // 将原始异常包装为运行时异常重新抛出
            throw new RuntimeException(e);
        } finally {
            // 后置通知：无论目标方法是否抛出异常都会执行
            System.out.println("我是后置通知");
        }
    }
}
```

### 概念



| 概念 | 说明 | 示例 |
|-|-|-|
| 连接点 | 所有可能被增强的方法 | 程序中的所有方法 |
| 切入点 | 那些需要被增强的方法 | 例如:UserService中的所有方法 |
| 通知  | 增强的逻辑 | 日志记录等 |
| 切面 | 切入点+通知+织入 | 例如:LoggingAspect类 |
| 目标对象 | 需要被增强的对象 | 例如:UserServiceImol实例 |



### 案例

#### 案例1：自动填充时间字段

```Java
package cn.itcast.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

import java.lang.reflect.Field;
import java.time.LocalDateTime;

@Aspect
@Component
public class ParameterAspect {
    // 篡改时间参数
    @Around("execution(* cn.itcast.service.*Service.save*(..)) || execution(* cn.itcast.service.*Service.add*(..))")
    public Object conpletionTime(ProceedingJoinPoint pjp) throws Throwable {
        Object[] args = pjp.getArgs();
        if (args.length == 1) {
            Object arg = args[0];
            Class<?> aClass = arg.getClass();
            Field createTime = aClass.getDeclaredField("createTime");
            if (createTime != null) {
                createTime.setAccessible(true);
                createTime.set(arg, LocalDateTime.now());
            }
            Field updateTime = aClass.getDeclaredField("updateTime");
            if (updateTime != null) {
                updateTime.setAccessible(true);
                updateTime.set(arg, LocalDateTime.now());
            }
        }
        return pjp.proceed(args);
    }
    @Around("execution(* cn.itcast.service.*Service.update*(..)) || execution(* cn.itcast.service.*Service.to*(..)) ||execution(* cn.itcast.service.*Service.set*(..)))")
    public Object conpletionUpdateTime(ProceedingJoinPoint pjp) throws Throwable {
        Object[] args = pjp.getArgs();
        if (args.length == 1) {
            Object arg = args[0];
            Class<?> aClass = arg.getClass();
            Field updateTime = aClass.getDeclaredField("updateTime");
            if (updateTime != null) {
                updateTime.setAccessible( true);
                updateTime.set(arg, LocalDateTime.now());
            }
        }
        return pjp.proceed(args);
    }
}
```

#### 案例2：操作日志记录

```Java
package cn.itcast.aop;

import cn.itcast.common.CommonData;
import cn.itcast.entities.OperateLog;
import cn.itcast.service.OperatorLogService;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.util.Map;

@Aspect
@Component
public class OperatorLogAspect {
    @Autowired
    private OperatorLogService operatorLogService;
    @Autowired
    private CommonData commonData;

    @Pointcut("execution(* cn.itcast.service.*Service.add*(..)) " +
            "|| execution(* cn.itcast.service.*Service.save*(..)) " +
            "|| execution(* cn.itcast.service.*Service.remove*(..)) " +
            "|| execution(* cn.itcast.service.*Service.delete*(..)) " +
            "|| execution(* cn.itcast.service.*Service.update*(..))" +
            "|| execution(* cn.itcast.service.*Service.to*(..))")
    public void pj(){}
    @Around("pj()")
    public Object recordLog(ProceedingJoinPoint pjp){
        long start = System.currentTimeMillis();
        Object proceed = null;
        try {
            proceed = pjp.proceed();
            return proceed;
        } catch (Throwable e) {
            throw new RuntimeException(e);
        } finally {
            long end = System.currentTimeMillis();
            Signature signature = pjp.getSignature();
            Map<String, Object> map = commonData.get();
            OperateLog operateLog = OperateLog.builder()
                    .className(signature.getDeclaringTypeName())
                    .methodName(signature.getName())
                    .operateUserId((Integer) map.get("id"))
                    .operateTime(LocalDateTime.now())
                    .costTime(start - end)
                    .methodParams(pjp.getArgs() == null ? "" : pjp.getArgs().toString())
                    .returnValue(proceed == null ? null : proceed.toString()).build();
            operatorLogService.save(operateLog);
        }
    }
}
```

常见的应用场景如下：

- 记录系统的操作日志
- 权限控制
- 事务管理：我们前面所讲解的Spring事务管理，底层其实也是通过AOP来实现的，只要添加@Transactional注解之后，AOP程序自动会在原始方法运行前先来开启事务，在原始方法运行完毕之后提交或回滚事务



### 配置优先级

#### 优先级顺序（从高到低）

##### **命令行参数**（最高优先级）

```Java
java -jar app.jar --server.port=8081 --spring.datasource.url=jdbc:mysql://localhost:3306/test
```

##### JVM系统参数

```Java
java -Dserver.port=8082 -Dspring.profiles.active=prod -jar app.jar
```

##### Properties文件

```Java
server.port=8083
spring.datasource.url=jdbc:mysql://localhost:3306/prod
```

##### **YAML/YML文件**（最低优先级）

```Java
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/dev
```



### Maven打包部署

#### 打包命令流程

```Java
# 1. 编译项目
mvn compile

# 2. 安装到本地仓库
mvn install

# 3. 打包（跳过测试）
mvn clean package -DskipTests

# 4. 运行jar包 //主程序中启动 java -jar jar包
java -jar target/application.jar
```

#### Spring Boot打包配置

父工程中设置主程序启动类

```XML
<plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <mainClass>cn.itcast.QkManagementApplication</mainClass>
<!--                    需要在安装的时候开启 打包的时候注释掉-->
<!--                    <skip>true</skip>-->
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / AOP、日志与全局异常 / 14 AOP · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/GD01wAls2iu1TvkdDsRcoYqOnxf)
