---
type: memo
date: 2026-11-06
aliases:
  - "bing-fa-jin-jie-mian-shi-ti"
  - "多线程并发进阶面试题（JMM、CAS、AQS、volatile）"
---

# 多线程并发进阶面试题（JMM、CAS、AQS、volatile）

## 一、概念说明

本篇收录库内八股文未覆盖的并发进阶题：synchronized 底层原理与锁升级、JMM、CAS、AQS、ReentrantLock、死锁诊断、volatile、ConcurrentHashMap 演进，以及线程控制的几个冷门口袋题。

### 1. synchronized 的底层原理

synchronized 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令（可用 `javap -v xxx.class` 查看字节码验证）：monitorenter 指向同步代码块开始位置，monitorexit 指明结束位置。monitor 对象存在于每个 Java 对象的对象头中，synchronized 锁就是通过这种方式获取的，这也是 Java 中任意对象都可以作为锁的原因。

Monitor 的结构：

- **Owner**：持有锁的线程，同一时间只能有一个；
- **EntryList**：保存处于 Blocked 状态的线程（抢锁失败进入）；
- **WaitSet**：保存处于 Waiting 状态的线程（曾持有锁、调用 wait 后等待条件）。

执行流程：Thread-2 执行 `synchronized(obj)` 将 Owner 置为自己；此时 Thread-3/4/5 再来抢锁就进入 EntryList 阻塞；Thread-2 执行完同步块后，唤醒 EntryList 中的线程来竞争锁——**竞争是非公平的**。

结论：synchronized 底层依赖 JVM 级别的 Monitor 决定线程是否获得锁，属于**悲观锁**，相对性能较低。

### 2. synchronized 锁升级（JDK6 之后）

JDK6 之前 synchronized 都是重量级锁；JDK6 之后做了升级优化，涉及 CAS 自旋、锁消除、锁膨胀、轻量级锁、偏向锁。锁升级与**对象布局**息息相关：对象实例的内存布局分为对象头、对象实例数据、对齐填充三部分，对象头又包括 MarkWord 和类型指针。MarkWord 的最后两位是锁标志位，倒数第 3 位是偏向锁标志：

- 后三位 001（偏向 0 + 锁 01）：无锁模式，MarkWord 记录 hashcode 和 GC 年龄；
- 后三位 101（偏向 1 + 锁 01）：偏向锁模式，记录获取偏向锁的线程 ID、Epoch、GC 年龄；
- 锁标志位 00：轻量级锁模式，锁记录放在加锁线程的虚拟机栈空间中；
- 锁标志位 10：重量级锁模式，MarkWord 保存 monitor 的地址。

锁的具体升级过程（通常情况下）：线程 A 向锁对象加偏向锁，线程 B 来了——若 A 已不存活，B 获得锁时是轻量级；若 A 存活且持锁不放，锁升级为重量级；若 A 存活但在 B 来之前释放了锁（两者没有交替竞争），B 获得锁，升级为轻量级。

偏向锁是只有一个线程使用锁，轻量级锁是两个线程交替使用锁；一旦有竞争就升级为重量级锁。升级过程中会先尝试**自旋**，自旋期间对方释放锁就避免了线程阻塞；自旋多次仍未拿到锁，线程才进入 EntryList 等待。

### 3. JMM（Java 内存模型）

JMM 是 Java 虚拟机规范中定义的一种内存模型，描述了 Java 程序中各种变量（线程共享变量）的访问规则，以及 JVM 将变量存储到内存和从内存读取变量的底层细节。特点：

1. 所有共享变量都存储于**主内存**（这里说的变量是实例变量和类变量，不包含局部变量——局部变量线程私有，不存在竞争）；
2. 每个线程还有自己的**工作内存**，保存被该线程使用变量的工作副本；
3. 线程对变量的所有操作（读、写）都必须在工作内存中完成，不能直接读写主内存；不同线程之间也不能直接访问对方工作内存中的变量，线程间变量值的传递需要通过主内存完成。

### 4. CAS

CAS 全称 Compare And Swap（比较再交换），是现代 CPU 广泛支持的对内存中共享数据进行操作的特殊指令，可以把 read-modify-write 转换为原子操作，由 CPU 直接保证原子性。

CAS 有 3 个操作数：内存值 V、旧的预期值 A、要修改的新值 B。当且仅当 A 和 V 相同时，将 V 修改为 B 并返回 true；否则什么都不做，返回 false。

例：线程 1 与线程 2 都从主内存获取 `a = 100` 到各自工作内存；线程 1 比较 A(100) 与 V(100) 相等，把 B(90) 更新到主内存；线程 2 再比较 A(100) 与 V（已是 90），不相等，更新失败——于是线程 2 自旋：重新读取 V=90，以 A=90、B=80 再次尝试。

### 5. AQS

AQS 全称 AbstractQueuedSynchronizer，是阻塞式锁和相关同步器工具的框架。特点：

- 用 `state` 属性表示资源的状态（分独占模式和共享模式），子类定义如何维护这个状态：`getState` 获取、`setState` 设置、`compareAndSetState` 用 CAS 机制设置；
- 独占模式只有一个线程能访问资源，共享模式允许多个线程访问；
- 提供基于 FIFO 的等待队列（CLH 同步队列，双向队列），类似于 Monitor 的 EntryList；
- 条件变量实现等待/唤醒机制，支持多个条件变量，类似于 Monitor 的 WaitSet。

当前线程获取同步状态失败时，AQS 将当前线程及等待状态构造成 Node 加入 CLH 队列尾部并阻塞；同步状态释放时，把首节点唤醒（公平锁），使其再次尝试获取。

### 6. ReentrantLock 的实现原理

ReentrantLock 主要利用 **CAS + AQS 队列**实现，支持公平锁和非公平锁，构造器接受可选的公平参数（默认非公平）：

```java
public ReentrantLock() {
    sync = new NonfairSync();   // 非公平锁
}
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

NonfairSync 继承自 AQS。非公平体现：某线程释放锁后，从 AQS 队列中激活一个线程（Thread-1）抢锁，此时新来的线程（Thread-4）也可以直接抢——若被 Thread-4 占先，Thread-4 被设为 exclusiveOwnerThread、state=1，Thread-1 重新走 acquireQueued 流程、获取失败再次 park 阻塞。公平锁下 Thread-4 会乖乖进队列等待。公平锁效率往往没有非公平锁高，多线程访问时吞吐量较低。

重入锁的含义：调用 lock 获取锁之后再次调用 lock 不会阻塞，内部直接增加重入次数，支持递归调用场景。

### 7. volatile

共享变量被 volatile 修饰后具备两层语义：

1. **可见性**：一个线程修改了变量的值，新值对其他线程立即可见——volatile 强制将修改的值立即写入主存；
2. **有序性**（禁止指令重排序）。

经典反例"永不停止的循环"：`static boolean stop = false;`，一个线程把 stop 改为 true，另一个线程的死循环 `while(!stop)` 却结束不了——JIT 即时编译器在循环执行次数达到阈值后会优化代码，导致读取不到共享变量的新值。解决方案：一是加 VM 参数 `-Xint` 禁用即时编译器（不推荐，影响全局）；二是用 volatile 修饰该变量。

### 8. ConcurrentHashMap 的演进

- JDK1.7：底层采用**分段的数组 + 链表**（Segment + HashEntry）。数据分一段一段存储，每段配一把锁（Segment 是可重入锁 ReentrantLock）；一个线程占用锁访问其中一段时，其他段仍可被其他线程访问；
- JDK1.8：放弃臃肿的 Segment 设计，采用 **Node + CAS + Synchronized** 保证并发安全，数据结构与 HashMap 1.8 一样（数组 + 链表/红黑树）。synchronized 只锁定当前链表或红黑树的首节点，只要 hash 不冲突就不会产生并发竞争，效率提升。

## 二、技术点之间的联系

- JMM 是总纲：主内存/工作内存的划分解释了"为什么会有可见性问题"；volatile 是解决可见性的轻量手段；synchronized 是互斥手段；CAS 是无锁手段。
- CAS 是地基：AQS 的 `compareAndSetState`、ConcurrentHashMap 1.8 的乐观操作、synchronized 锁升级中的自旋，底层都是 CAS。
- Monitor 与 AQS 是两套对称设计：Owner↔state+exclusiveOwnerThread、EntryList↔CLH 队列、WaitSet↔条件变量。理解这个对应关系，synchronized 和 ReentrantLock 的原理就串起来了。
- 锁升级是"按竞争强度分级付费"：无竞争用偏向锁，交替用轻量级锁，真竞争才用重量级锁——与 ReentrantLock 固定走 AQS 形成对照，也解释了"低竞争下 synchronized 性能不赖"。

## 三、方法与实现要点

### 1. synchronized 与 Lock 的区别

| 维度 | synchronized | Lock |
| --- | --- | --- |
| 语法 | 关键字，JVM 层面（C++ 实现） | 接口，JDK 提供（Java 实现） |
| 释放锁 | 退出同步块自动释放 | 必须手动 unlock，最好在 finally 中 |
| 功能 | 基本互斥、同步、锁重入 | 额外支持获取等待状态、公平锁、可打断、可超时、多条件变量 |
| 实现 | — | ReentrantLock、ReentrantReadWriteLock 等 |
| 性能 | 无竞争时有偏向锁/轻量级锁优化，不赖 | 竞争激烈时通常性能更好 |

二者都属于悲观锁，都具备基本的互斥、同步、锁重入功能。

### 2. 死锁产生的条件与诊断

死锁：一个线程需要同时获取多把锁时容易发生——t1 持有 A 锁想拿 B 锁，t2 持有 B 锁想拿 A 锁，双方互等，程序不结束也不报错。

诊断用 JDK 自带工具：

1. `jps` 查看运行中的 Java 进程，拿到进程号；
2. `jstack -l 进程号` 查看线程运行情况，输出中会明确指出死锁及涉及的锁对象。

### 3. 冷门线程控制题

- **T1、T2、T3 按顺序执行**：用 `join()`——T2 里调用 `t1.join()`、T3 里调用 `t2.join()`，join 会等被加入的线程执行完毕再继续当前线程；
- **notify() 与 notifyAll()**：notify 只随机唤醒一个 wait 线程，notifyAll 唤醒所有 wait 的线程；
- **run() 与 start()**：start 用来启动线程，由线程调用 run 执行其中的逻辑，start 只能调用一次；run 封装了要被线程执行的代码，可以被调用多次（直接调 run 就是普通方法调用，不会开启新线程）；
- **停止线程**：使用退出标志让线程正常退出（run 方法完成后线程终止）；`stop()` 强行终止已作废，不推荐（会造成数据不一致）。

### 4. 为什么不建议用 Executors 创建线程池

开发手册的强制规约：应通过 `new ThreadPoolExecutor(...)` 手动配置七大参数，而不是用 Executors 的工厂方法——因为 `newFixedThreadPool`、`newSingleThreadExecutor` 的等待队列是无界的，任务堆积会导致 OOM；`newCachedThreadPool` 允许创建的线程数近乎无界，同样可能耗尽资源。显式配置队列容量和最大线程数，才能把风险暴露在设计期。

## 四、重点与难点分析

- **重点 1**：synchronized 三句话——Monitor 实现、monitorenter/monitorexit 指令、JDK6 后锁升级（偏向→轻量级→重量级）。
- **重点 2**：CAS 三操作数模型（V、A、B）+ 自旋重试，能口述"线程 2 更新失败后重新读 V 再比较"的全过程。
- **难点 1**：锁升级路径的分支条件（A 是否存活、是否交替竞争）容易记混，抓住"竞争越激烈锁越重"的主线即可。
- **难点 2**：volatile 只保证可见性和有序性，**不保证原子性**——`count++` 这类复合操作即使用 volatile 修饰仍有线程安全问题。
- **易错点**：把 JMM 和 JVM 内存结构（堆/栈/方法区）混为一谈——前者是线程间内存交互规则，后者是运行时数据区划分。

## 五、示例

### 示例 1：死锁代码与诊断

```java
public class Deadlock {
    public static void main(String[] args) {
        Object A = new Object();
        Object B = new Object();
        Thread t1 = new Thread(() -> {
            synchronized (A) {
                System.out.println("lock A");
                try { Thread.sleep(1000); } catch (InterruptedException e) { throw new RuntimeException(e); }
                synchronized (B) {
                    System.out.println("lock B");
                }
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            synchronized (B) {
                System.out.println("lock B");
                try { Thread.sleep(500); } catch (InterruptedException e) { throw new RuntimeException(e); }
                synchronized (A) {
                    System.out.println("lock A");
                }
            }
        }, "t2");
        t1.start();
        t2.start();
    }
}
```

程序不结束即为死锁现象。诊断：`jps` 找到进程号后 `jstack -l 46032`，输出会标注 Found one Java-level deadlock。

### 示例 2：ReentrantLock 的正确用法与重入

```java
static Lock lock = new ReentrantLock();

public static void method1(){
    try {
        lock.lock();
        // 进入 method2 方法（重入：同一线程再次 lock 不会阻塞）
        method2();
        System.out.println("method1...");
    } finally {
        // 尽量在 finally 中释放锁
        lock.unlock();
    }
}

public static void method2(){
    try {
        lock.lock();
        System.out.println("method2...");
    } finally {
        lock.unlock();
    }
}
```

### 示例 3：T1、T2、T3 顺序执行

```java
public class JoinTest {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> System.out.println("t1"));
        Thread t2 = new Thread(() -> {
            try {
                t1.join();   // 加入线程 t1，t1 执行完毕后再执行当前线程
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("t2");
        });
        Thread t3 = new Thread(() -> {
            try {
                t2.join();   // 加入线程 t2
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("t3");
        });
        t1.start();
        t2.start();
        t3.start();
    }
}
```

### 示例 4：volatile 解决"永不停止的循环"

```java
public class ForeverLoop {
    // 加 volatile 后，一个线程修改 stop，其他线程立即可见
    static volatile boolean stop = false;

    public static void main(String[] args) {
        new Thread(() -> {
            try { Thread.sleep(100); } catch (InterruptedException e) { e.printStackTrace(); }
            stop = true;
            System.out.println("modify stop to true...");
        }).start();

        // 若 stop 不加 volatile，JIT 优化后该循环可能永远结束不了
        while (!stop) {
        }
        System.out.println("循环结束");
    }
}
```
