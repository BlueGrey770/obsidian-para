---
type: memo
date: 2026-11-06
aliases:
  - "06 mybatis"
---
# 06 mybatis

## 抽取jdbc实现mybatis原理

### 动态代理

```XML
作用：不改变源码的情况下对功能进行增强
概念：目标对象-->就是需要对其进行方法增强的对象
     代理对象：就是按照接口创建出来的实现类对象，代理访问任何功能，第三个参数的invoke都会执行，
     在invoke中挑出想要增强的方法，去编写增强逻辑即可
如何创建代理：Proxy.newInstance(ClassLoader cl,Class[] interfaces,involvedHandler（臭接口))
```

### 类加载器的分类

```XML
1. 启动类加载器
    是什么：JVM自带的最顶层类加载器，由C/C++实现（不是Java类）
    作用：负责加载JVM核心类库，保证JVM自身能正常启动。
    负责加载jdk大部分的类
2. 平台类加载器（扩展类加载器）：
    是什么：Java实现的类加载器（属于JVM的扩展层），JDK9以后叫平台类加载器，之前叫扩展类加载器。
    作用：加载JDK的扩展类库,补充启动类加载器未覆盖的JDK功能类。
    负责加载jdk中ext包下的扩展类
3. 系统类加载器：
    是什么：也叫应用类加载器，Java实现的类加载器，使开发者写的代码默认的类加载器
    作用：加载项目的classpath下的类。
    负责加载程序员自定义的类
```

<callout emoji="👍">
双亲委派模型：
是什么：JVM类加载的核心机制（不是类加载器，是类加载的规则）。
目的：

保证类的唯一性：比如java.lang.String只会被启动类加载器加载，避免自定义类冒充核心类；

保证安全：防止恶意代码替换JVM核心类，避免安全风险。

执行逻辑：类加载时，先让父加载器尝试加载，父加载器加载不了，才自己加载（启动类加载器是最顶层的 ”父“）

有加载任务时，第三类会委派给第二类....
</callout>

| 步骤 | 核心动作 | 涉及类加载器 | 关键说明 |
|-|-|-|-|
| 1 | 发起加载请求 | 系统类加载器（应用类加载器） | 开发者写的代码，第三方依赖类，默认先交给系统类加载器处理 |
| 2 | 向上委托负载在其 | 系统类加载器--->平台类加载器 | 系统类加载器不直接加载，先找到自己的父（平台类加载器尝试加载） |
| 3 | 继续向上委托 | 平台类加载器--->启动类加载器 | 平台类加载器也不直接加载，继续找父（启动类加载器） |
| 4 | 顶层加载器尝试加载 | 启动类加载器 | 检查要加载的类是否属于JVM核心类库（如rt.jar）:  <br/>能加载--->直接返回类对象，流程结束；  <br/>加载不了---->向下回传加载请求 |
| 5 | 平台类加载器尝试加载 | 平台类加载器 | 检查要加载的类是否属JDK扩展/平台类库：  <br/>能加在---->返回类对象，流程结束；  <br/>加载不了--->继续向下回传 |
| 6 | 系统类加载器最终加载 | 系统类加载器 | 检查要加载的类是否在项目classpath下：  <br/>能加载--->返回类对象；  <br/>加载不了--->抛出ClassNotFoundException(类找不到异常) |

**给list集合增加去重方法，使用动态代理**

```Java
public class ProxyDemo01 {
    public static void main(String[] args) {
        //创建目标对象
        List<String> list = new ArrayList<>();
        //创建代理对象
       List list1 =  (List)Proxy.newProxyInstance(
               //第一个参数，固定写法
                ClassLoader.getSystemClassLoader(),
                //第二个参数 要代理谁就获取谁的字节码对象
                new Class[]{List.class},
               //第三个参数，对目标对象进行增强处理
               //重新接口的抽象方法invoke
                new InvocationHandler(){
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                            if (method.getName().equals("add")&&args.length ==1){
                                if (!list.contains(args[0])){
                                    return method.invoke(list,args);
                                }
                                return false;
                            }else if(method.getName().equals("add") && args.length ==2){
                                if (!list.contains(args[1])){
                                    return method.invoke(list,args);
                                }
                                return null;

                            }else{
                                return method.invoke(list,args);
                            }
                    }
                }
        );
            list1.add("add");
            list1.add("add");
            list1.add("add");
            list1.add("add");
            list1.add("add");
            list1.add(1,"aaa");
        System.out.println(list1);
    }
}
```

## mybatis

### mybatis的基本使用步骤



```XML
1. 第一步：导入mybatis相关坐标
          创建项目模块时直接选择mybatis framework
          <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>3.0.0</version>
          </dependency>
2. 第二步：配置连接四要素：
            spring:
              datasource:
                # 数据库用户名
                username: root,
                # 数据库密码
                password: 123456,
                # 数据库连接
                url: jdbc:mysql://localhost:3306/itheima142002,
                #数据库驱动
                driver-class-name: com.mysql.cj.jdbc.Driver
3. 定义接口，@Mapper 注解修饰这个接口：
            @Mapper
            public interface StudentMapper {
                }
4. 在接口编写抽象方法，通过注解悬挂sql语句即可：
            @Select("select * from students")
            List<Student> findAll();
            
            @Select("select * from students where name = #{name} and power = #{power}")
            Student findByNameAndPower(@Param("name") String name, @Param("power") Integer power);
            
            @Insert("insert into students values(null,#{name}, #{age},#{sex}, #{power})")
            int AddStudent(Student student);
            
            @Delete("delete from students where id = #{id}")
            int removeStudentById(Integer id);
5. 注入Dao层的对象直接使用：
            @SpringBootTest
            public class StudentMapperTest {
            
                @Autowired
                private StudentMapper studentMapper;
            
                @Test
                public void testfindAll() {
                    List<Student> all = studentMapper.findAll();
                    for (Student student : all) {
                        System.out.println(student);
                    }
                }                      
```

### mybatis的参数处理的解决方案

```XML
1. 单个简单参数：直接使用 #{任意值}
        @Delete("delete from students where id = #{id}")
        int removeStudentById(Integer id);
2. 单个自定义对象类型的参数：在values中使用#{成员变量名1}，#{成员变量名2}中间使用","分隔
        @Insert("insert into students values(null,#{name}, #{age},#{sex}, #{power})")
        int AddStudent(Student student);
3. 单个map集合的参数：这个方法需要在使用时定义一个HashMap() 用键对给每个成员变量赋值 #{键}
        @Insert("insert into students values(null,#{name}, #{age},#{sex}, #{power})")
        int AddStudent(Map<Student,Object> map);
4. 单个list集合或者数组的参数：需要使用xml配置文件进行sql语句的编写 并使用foreach标签
    arry                @Param取别名
    collection list     @Param取别名
        public int removeStudentbyIds(int[] ids);
        <delete id="removeStudentbyIds">
            delete from students where 
            <foreach collection="array" open="id in (" close=")" item="id" separator=",">
                #{id}
            </foreach>
        </delete>
5. 多个参数：需要使用@Param注解给每个参数取别名，要不然会自动识别为（arg0,arg1……）
        @Select("select * from students where name = #{name} and power = #{power}")
        Student findByNameAndPower(@Param("name") String name, @Param("power") Integer power);
```

<callout emoji="❤️">
（1）如果参数只有单个并且是简单类型--->（基本类型，包装类型，String），我们可以通过#{任意名}取参数值设置到sql中
（2）如果参数只有单个并且是自定义对象类型---->我们可以通过#{属性名}来取值，底层采用的setter以及getter方法来做的

属性：类中的setter和getter方法叫做属性!!!!

成员变量：类中方法外定义的变量叫做成员变量
（3）当我们参数是单个并且是Map集合的时候，我们可以通过#{key}取map中对应的值
（4）当参数是多个的时候，我们需要通过#{参数名}获取，但是必须使用@Param（"给参数设置名字"）才行，默认的参数名默认是arg0，arg1...以此编号，建议大家都设置别名，统一风格
</callout>

### mybatis使用xml的使用步骤

```XML
1.第一步：创建一个和Mapper类相同的文件夹（要求层级一模一样，每一层名字一模一样），放在resources下面
2.第二步：添加XML文件必要的文件头
3.第三步：<mapper namespace="com.itheima.mapper.StudentMapper"> ，
            这里面的namespace是核心配置：需要填写你的Mapper接口的全限定类名
4. 第四步：编写sql语句：值得注意的是这里的id就是你的Mapper接口中的方法名（必须一模一样！！）
        <delete id="removeStudentbyIds">
            delete from students where 
            <foreach collection="array" open="id in (" close=")" item="id" separator=",">
                #{id}
            </foreach>
        </delet>
        
 注意：如果是要一次性输入多个参数（数组/集合）的话那么就使用foreach标签去遍历元素，底层会自动给拼接sql
      如果是类似于按条件搜索功能的需求的查询的话那么需要使用<where>+<if>标签。
      
      案例：
      <select id="findByCollectors" resultType="com.itheima.pojo.Student">
        select * from students
        <where>
            <if test="name != null and name   != ''">
                name = #{name}
            </if>
    
            <if test="age != null and age != ''">
                and age = #{age}
            </if>
    
            <if test="power != null and power != ''">
                and power = #{power}
            </if>
        </where>
</select>
```

<callout emoji="🏖️">
注意事项：

1. namespace:必须是Mapper接口的全限定类名
2. 每个sql语句的标签中的id必须是该方法名
3. 如果非要不把xml文件放在规范的位置的情况下需要在yml文件中配置 mapper-locations: classpath:mapper/\*.xml
</callout>

### mybatis的小配置

```YAML
mybatis:
  configuration:
    #开启驼峰命名
    map-underscore-to-camel-case: true
    #开启mybatis日志
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    #类路径
    mapper-locations:classpath:*.xml
```

### boot配置的优先级

```XML
properties > yml > yaml
yml可以使用模板解决，以后每次添加新的配置之后可以修改模板

spring:
  datasource:
    # 数据库用户名
    username: root,
    # 数据库密码
    password: 123456,
    # 数据库连接
    url: jdbc:mysql://localhost:3306/itheima142002,
    #数据库驱动
    driver-class-name: com.mysql.cj.jdbc.Driver
mybatis:
  configuration:
    #开启驼峰命名
    map-underscore-to-camel-case: true
    #开启mybatis日志
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

### maven项目的聚合继承

#### 聚合

```XML
为了聚合多个子功能，方便一件构建

```

#### 继承

```XML
1. 让子工程继承家产：家产定义在XXX
2. 让子工程遵守家规：家规定义在XXX
即:
    (1) 从父工程继承相关依赖,继承"家产"
    在父工程中<dependencies> 中定义的依赖都可以被子工程继承
    (2) 遵循父工程规定"家规"
```

---
> 来源：飞书技术知识库（脱敏版） · 02 数据库与持久层 / MyBatis / 06 mybatis · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/SxXUwtzuNi6oEnkfy4lchDQTnld)
