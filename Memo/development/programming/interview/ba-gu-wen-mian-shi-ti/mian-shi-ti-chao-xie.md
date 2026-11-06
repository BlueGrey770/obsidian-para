---
type: memo
date: 2026-11-06
aliases:
  - "面试题抄写"
---

<title>面试题抄写</title>

面试题：

1、反射机制及主要用到的方法

2、Spring中Bean有几种注入方式

3、悲观锁和乐观锁的区别



# 反射机制及主要用到的方法

```Java
反射机制是Java中一种强大的特性,他允许程序在运行期间动态获取方法区的字节码对象,并且解剖字节码每一部分并进行操作.
简单来说,反射是操作类的另外一种方式,与普通方式直接创建对象\ 访问属性和方法不同.
反射的主要作用:
    在运行期间动态获取字节码对象
    解剖字节码每一部分并进行操作
    实现动态编程和框架开发
反射的主要方法:
    获取字节码对象的三种方式:
        Class.forName("类的全限定类名")
        类名.class
        对象.getClass()
    使用反射创建对象:
        获取字节码对象
        获取无参/有参构造
            字节码对象.getConstructor()
        暴力反射
            有参/无参构造对象.setAccessible(true)
        使用newInstance创建对象
            有参/无参构造对象.newInstance(有参列表)
    使用反射操作成员变量:
        获取字节码对象
        获取成员变量对象
            字节码对象.getDeclareField("成员变量名")
        创建实例对象
            字节码.newInstance()
        设置变量值/获取变量值
            成员变量.set/get
    使用反色操作成员方法:
        获取字节码对象
        拿到成员方法对象
            字节码对象.getDeclaredMethod("方法名",方法的参数列表的数据类型对象...)
        通过成员方法对象调用成员方法
            成员方法对象.invoke("实体类对象名",对方法的参数列表赋值)
            
            
            
```

# Spring中Bean有几种注入方式

```Java
Spring依赖注入(DI)是IOC理念的具体手段,Spring容器在创建对象时自动将对象依赖的其他对象注入到当前对象中.Spring依赖注入有三种核心方式:构造器注入,setter注入和@autowird字段注入
    构造器注入
        通过构造器函数注入依赖的对象,当对象创建时自动注入所需的依赖
    setter注入
        通过setter方法注入依赖对象,Spring在对象创建后调用对应的setter方法注入依赖
    Autowired字段注入
        直接在字段上使用@Autowird注解进行注入,不需要构造器或setter方法
    解决Bean冲突的方法
        当存在多个相同类型的Bean时,可以使用一下方法解决冲突:
            @Autowired+@Qualifier
                可以结合@Qualifier解决,可以指定名字注入
                例如:@Qualifier("bean的名字")
                @Autowired负责找类型,@Qualifier负责精确指定Bean名
            按名称注入
                可以采用@Resource解决,可以通过name属性指定名字注入
                例如:@Resource(name = "bean的名字")
                @Resource:我指定名字,你就给我这个Bean
            指定默认Bean
                可以采用@Primary解决,通过@Primary指定默认的bean,提高优先级
                @Primary是"默认选择"
                如果同时使用@Qualifier,以@Qualifier为准
                
```

# 悲观锁和乐观锁的区别

```Java
两种锁的基本区别:
    实现方式不同:
        悲观锁: 通常是数据库层的实现,如select .. for update语句
        乐观锁: 通常是应用层的实现,如使用版本号或时间戳
    性能表现不同
        悲观锁:在高并发下可能导致性能下降,因为会阻塞其他线程
        乐观锁:在并发下性能更好,因为不会阻塞其他线程
    适用场景不同;
        悲观锁: 适合写操作频繁\冲突率高的场景
        乐观锁: 适合读操作频繁\冲突率低的场景
    冲突处理方式不同:
        悲观锁:在操作前加锁.防止其他线程操作,直接避免冲突
        乐观锁: 先假设没有冲突,操作检查时检查是否冲突,如果发现冲突则重试或放弃
    实现复杂度不同:
        悲观锁:实现相对简单,但需要处理死锁问题
        乐观锁:实现相对复杂,需要处理重试逻辑
    乐观锁通常使用版本号机制实现:在表中增加一个version字段,更新时检查版本号是否一致,一致则更新并增加版本号,否则重试.
        
```

面试题

1.sleep和wait的区别

2.类加载器加载过程

3.HashMap为什么用红黑树不用B树

# sleep和wait的区别

```Java
sleep和wait是Java中两种线程进入等待状态的方法,他主要有一下主要区别:
    状态不同:
        sleep(long time) 会使线程进入 TIME_WAITING(计时等待)状态,到时间后线程会自动转成RUNNABLE状态
        wait()会使线程进入WAITING(无限等待)状态,必须通过锁的notify()或者notifyAll()唤醒才行
    锁的持有情况不同:
        sleep方法不会释放对象锁
        wait方法会释放对象锁
    调用位置不同:
        sleep可以在任何地方调用
        wait只能在同步块或同步方法中调用
    唤醒方式不同
        sleep是自动唤醒
        wait需要其他线程调用notify或notifyAll()来唤醒
```

---
> 来源：飞书技术知识库（脱敏版） · 08 面试知识库 / 八股文面试题 / 面试题抄写 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/KDUqw0KgpiHoOxkuco2crLoYnNc)
