---
type: memo
date: 2026-11-06
aliases:
  - "jvm-jin-jie-mian-shi-ti"
  - "JVM 进阶面试题（引用类型与垃圾回收器）"
---

# JVM 进阶面试题（引用类型与垃圾回收器）

## 一、概念说明

本篇收录库内八股文未覆盖的 JVM 进阶题：四种引用类型、垃圾判定算法、可达性分析与 finalize、主流垃圾回收器（Serial / Parallel / CMS / G1）。

### 1. 强引用、软引用、弱引用、虚引用

- **强引用**：最普通的引用方式，表示对象处于"有用且必须"的状态。只要对象具有强引用，GC 就不会回收它；即便堆内存不足，宁可抛出 OOM 也不回收。
- **软引用**：表示对象处于"有用但非必须"的状态。内存足够时 GC 不回收；内存空间不足时，会在 OOM 出现之前对其进行回收。注意 GC 线程优先级较低，软引用并不会立即被回收。

```java
User user = new User();
SoftReference softReference = new SoftReference(user);
```

- **弱引用**：表示对象"可能有用但非必须"。GC 线程扫描内存区域时，一旦发现只具有弱引用的对象，**无关内存是否足够都会回收**（同样因 GC 线程优先级低，不是立刻回收）。

```java
User user = new User();
WeakReference weakReference = new WeakReference(user);
```

延伸考点：**ThreadLocal 内存泄漏**。ThreadLocal 的 Entry 用的就是弱引用——key 是当前 ThreadLocal（弱引用，GC 发现即回收），value 是强引用不会被回收。key 被回收后 value 失去了访问入口却占着内存，即内存泄漏。使用建议：使用完毕后注意调用清理方法（remove）。

- **虚引用**：必须配合引用队列一起使用。当虚引用所引用的对象被回收时，由 Reference Handler 线程将虚引用对象入队，这样就可以知道哪些对象被回收了，从而对它们关联的资源做进一步处理。

```java
PhantomReference a = new PhantomReference(new A(), referenceQueue);
```

### 2. 对象什么时候可以被垃圾回收

简单一句话：如果一个或多个对象没有任何引用指向它，它就是垃圾，可能会被回收。定位垃圾有两种方式：引用计数法、可达性分析算法。

**引用计数法**：一个对象被引用一次，对象头上的引用计数就递增一次；引用次数为 0 代表可回收。

- 优点：实时性较高，无需等内存不够才开始回收；垃圾回收过程中应用无需挂起；更新计数器只影响该对象，不扫描全部对象。
- 缺点：每次引用都要更新计数器，有时间开销、浪费 CPU 资源；**无法解决循环引用问题，会引发内存泄漏**（最大缺点）——a 和 b 相互引用且外部引用都置 null 后，引用计数不为 0，永远不会被回收。

**可达性分析算法**：现在的虚拟机都采用可达性分析来确定垃圾。从根节点（GC Roots）出发，引出它指向的下一个节点，再依次往下遍历，直到所有节点遍历完毕。GC Roots 是那些肯定不能当垃圾回收的对象：局部变量、静态方法、静态变量、类信息。核心判断：某对象是否与根对象有直接或间接的引用，没有则可回收。

注意：被标记为可回收的对象**并不会马上被回收**。对象中存在 `finalize` 方法——发生 GC 时先判断该对象的 finalize 是否执行过，未执行则先执行它，执行过程中如果对象重新与 GC Roots 产生关联，可以"复活"一次。

### 3. 垃圾回收器

JVM 实现了多种垃圾收集器：串行、并行、CMS（并发）、G1。

| 收集器 | 作用区域 | 算法 | 特点 |
| --- | --- | --- | --- |
| Serial | 新生代 | 复制 | 单线程回收，回收时所有应用线程暂停（STW，Stop-The-World）；交互性强的应用不能接受，JavaWeb 应用一般不采用 |
| ParallelNew | 新生代 | 复制 | 在 Serial 基础上改为多线程回收，缩短回收时间；JDK8 默认；仍会 STW，但暂停更短 |
| Parallel Scavenge | 新生代 | 复制 | 目标是**可控的吞吐量**（吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间）），适合后台运算而不需要太多交互的任务 |
| Serial Old | 老年代 | 标记-整理 | 单线程，主要给 Client 模式的虚拟机使用 |
| Parallel Old | 老年代 | 标记-整理 | 注重吞吐量及 CPU 资源敏感的场合，优先 Parallel Scavenge + Parallel Old 组合 |

**CMS 垃圾收集器**（Concurrent Mark Sweep）：并发的、使用**标记-清除**算法的老年代回收器，目标是获取最短回收停顿时间；最大特点是垃圾回收时应用仍然能正常运行。执行过程：

1. 初始标记（Initial Mark）：仅标记 GC Roots 能直接关联到的对象，速度快，但需要 STW；
2. 并发标记（Concurrent Mark）：追踪引用链，与用户线程并发执行；
3. 重新标记（Remark）：修正并发标记期间因用户线程继续运行而变化的标记记录，需要 STW，但远短于并发标记；
4. 并发清除（Concurrent Sweep）：清除可回收对象，与用户线程并发执行。

**G1 垃圾收集器**：同时应用于新生代和老年代，采用标记-整理算法，软实时、低延迟、可设定最大 STW 停顿时间目标，用于代替 CMS，适用于较大的堆（>4~6G），JDK9 之后默认使用。

- 内存布局：G1 最大的区别在于**取消了年轻代、老年代的物理划分**，把堆划分为若干个区域（Region），逻辑上仍分年轻代、老年代。新增 Humongous 区（属于老年代）：超过分区容量一半的巨大对象进入该区，一个 H 区装不下就寻找连续 H 区存储，有时甚至不得不启动 Full GC；
- G1 会估计每个 Region 中的垃圾比例，**优先回收垃圾较多的区域**；
- 三种回收模式：Young GC（eden region 耗尽时触发，存活对象拷贝到 survivor 或晋升 old region）、Mixed GC（老年代占比达到阈值触发，`-XX:InitiatingHeapOccupancyPercent` 默认 45%，回收整个 young region + 部分 old region，过程与 CMS 类似：初始标记 STW → 并发标记 → 最终标记 STW → 筛选回收）、Full GC；
- 由于回收时通过对象在 Region 间复制完成清理，G1 顺带完成了堆的压缩，**不存在 CMS 的内存碎片问题**。

## 二、技术点之间的联系

- 引用类型是"判定门槛"，可达性分析是"判定方法"，垃圾回收算法（标记-清除/复制/标记-整理/分代收集）是"回收动作"，垃圾回收器是"算法的产品化组合"——四者构成 GC 的完整链路。
- 四种引用强度依次递减，给 GC 提供了"回收优先级"：强引用永不回收、软引用内存不足才回收、弱引用见到就回收、虚引用只用于回收通知。
- ThreadLocal 内存泄漏是弱引用最典型的实战考点，把"弱引用见到就回收"与"value 是强引用"两个规则拼在一起就是答案。
- CMS 与 G1 的对比本质是"低停顿的两种路线"：CMS 靠并发标记/清除压低停顿但留下内存碎片；G1 靠 Region 化 + 可预期停顿模型兼顾碎片问题，因此 JDK9 起 G1 取代 CMS 成为默认。

## 三、方法与实现要点

1. 回答四种引用时，按"强度递减 + 回收时机"两轴组织语言，每种引用补一个应用场景（软引用可做缓存、弱引用对应 ThreadLocal、虚引用对应堆外内存回收通知）。
2. 被问"对象何时被回收"时，先答引用计数法及其循环引用缺陷，再答可达性分析 + GC Roots 的组成，最后补 finalize 的"复活一次"细节——层次递进。
3. 回答垃圾回收器时按代际组织：新生代（Serial / ParallelNew / Parallel Scavenge，复制算法）→ 老年代（SerialOld / ParallelOld / CMS，标记-整理或标记-清除）→ 整堆（G1，Region 化）。
4. 吞吐量与停顿时间是一对矛盾：吞吐量大适合后台计算型任务，停顿时间短适合交互型应用——选收集器的论述要从这个权衡出发。

## 四、重点与难点分析

- **重点 1**：弱引用"无关内存是否充足，见到就回收"，这是它与软引用的本质区别，也是 ThreadLocal 泄漏题的前置知识。
- **重点 2**：引用计数法的循环引用缺陷必须能画出 a、b 互相引用、外部置 null 的场景。
- **难点 1**：CMS 四个阶段中只有两个阶段 STW（初始标记、重新标记），耗时最长的并发标记和并发清除是与用户线程并发的——这是"CMS 停顿短"的根本原因。
- **难点 2**：G1 的 Mixed GC 只回收**一部分**老年代（可以选择回收哪些 old region），不是全部老年代，借此控制回收耗时；触发阈值默认是堆占用 45%。
- **易错点**：finalize 复活只有一次机会（finalize 只会被系统调用一次）；虚引用 get() 永远返回 null，存在感完全依赖引用队列。

## 五、示例

### 示例 1：虚引用配合引用队列做资源清理

```java
public class TestPhantomReference {
    public static void main(String[] args) throws InterruptedException {
        ReferenceQueue<String> queue = new ReferenceQueue<>();
        List<MyResourse> list = new ArrayList<>();
        list.add(new MyResourse(new String("a"), queue));
        list.add(new MyResourse("b", queue));
        list.add(new MyResourse(new String("c"), queue));

        System.gc();
        Thread.sleep(100);

        // 对象被回收后，虚引用入队，轮询队列即可感知并清理关联资源
        Object ref;
        while ((ref = queue.poll()) != null) {
            if (ref instanceof MyResourse) {
                MyResourse.clean();
            }
        }
    }

    static class MyResourse extends PhantomReference<String> {
        public MyResourse(String referent, ReferenceQueue<? super String> q) {
            super(referent, q);
        }

        // 释放外部资源的方法
        public static void clean() {
            System.out.println("clean");
        }
    }
}
```

### 示例 2：ThreadLocal 的弱引用结构

```java
// ThreadLocalMap 中 Entry 的源码结构
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
        super(k);      // key 是弱引用，GC 发现即回收
        value = v;     // value 是强引用，不会被回收 → 内存泄漏风险
    }
}
```

### 示例 3：引用计数法的循环引用失效

```java
A a = new A();
B b = new B();
a.instance = b;
b.instance = a;
a = null;
b = null;
// 虽然 a 和 b 的外部引用都为 null，但两者互相引用，
// 引用计数都不为 0，引用计数法下永远不会被回收 → 内存泄漏
```

### 示例 4：CMS 与 G1 的回答模板

```text
CMS：并发标记清除，针对老年代。四阶段：初始标记(STW，快) → 并发标记 → 重新标记(STW，较短)
→ 并发清除。停顿最短，但基于标记-清除算法，会产生内存碎片。

G1：Region 化整堆收集，可设定最大停顿目标，JDK9 默认。优先回收垃圾比例高的 Region；
Mixed GC 只回收部分老年代，阈值默认堆占用 45%。回收即复制，天然压缩，无碎片问题。
```
