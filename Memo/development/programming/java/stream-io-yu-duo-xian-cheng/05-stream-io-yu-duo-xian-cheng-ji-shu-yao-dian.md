---
type: memo
date: 2026-11-06
aliases:
  - "05 Stream、IO 与多线程技术要点"
---

# 05 Stream、IO 与多线程技术要点

> 三个主题对应 Java 进阶的三块基石：**声明式集合处理（Stream）、数据传输（IO）、并发执行（多线程）**。

## 一、技术概念说明

### 1. Stream 流：集合的声明式处理管道

Stream 不是存储结构，而是「数据源 → 中间操作（0\~N 个）→ 终端操作（1 个）」的处理管道。**只有执行终端操作时，整条管道才真正计算**（惰性求值）。

| 类别 | 方法 | 作用 |
|-|-|-|
| 中间操作 | `filter(条件)` | 过滤 |
|  | `map(转换)` / `mapToInt()` | 元素变形 |
|  | `sorted(比较器)` | 排序 |
|  | `distinct()` | 去重 |
|  | `limit(n)` / `skip(n)` | 取前 n 个 / 跳过前 n 个 |
|  | `flatMap(list -> list.stream())` | 嵌套结构拍平成一维 |
| 终端操作 | `forEach()` / `count()` / `max()` / `min()` | 消费与统计 |
|  | `reduce(初值, 累加器)` | 聚合（如 `reduce(0, Integer::sum)` 求和） |
|  | `collect(Collectors.toList())` | 收集为集合 |
|  | `collect(groupingBy(键, averagingDouble(值)))` | 分组 + 组内聚合 |

### 2. IO 流：数据的输入与输出

- **字节流**（`InputStream/OutputStream` 家族）：处理一切文件（图片、视频、文本皆可），文件复制的通用选择；
- **字符流**（`Reader/Writer` 家族）：专为文本设计，按字符解码；
- 缓冲流（`BufferedXxx`）给基础流加缓冲区，读写效率成倍提升。
- 文件复制四步：开输入流 → 开输出流 → 循环「读一批 → 写一批」→ 关流。

### 3. 多线程：并行执行的任务单元

**两种创建方式**：

| 方式 | 写法 | 特点 |
|-|-|-|
| 继承 `Thread` | `class MyThread extends Thread { public void run(){...} }` | 简单，但占用了唯一的继承位 |
| 实现 `Runnable` | `new Thread(() -> {...}, "线程名").start()` | **推荐**，任务与线程解耦，可共享给多线程 |

主线程等待子线程结束用 `join()`。

**线程六态**：新建（New）→ 就绪（Runnable，调用 start 后）→ 运行中获得 CPU → 阻塞（Blocked，等锁）/ 等待（Waiting，wait/join）/ 超时等待（Timed Waiting，sleep）→ 死亡（Terminated）。

**线程池七大参数**（`ThreadPoolExecutor`）：

| 参数 | 含义 |
|-|-|
| corePoolSize | 核心线程数（常驻） |
| maximumPoolSize | 最大线程数（核心 + 临时） |
| keepAliveTime + unit | 临时线程空闲存活时间 |
| workQueue | 任务等待队列 |
| threadFactory | 线程工厂（定制线程名等） |
| handler | 拒绝策略 |

- **临时线程触发条件**：核心线程全忙 **且队列已满**；
- **拒绝策略触发条件**：核心满 + 队列满 + 临时线程也满（达到 maximumPoolSize）；
- **四大拒绝策略**：`AbortPolicy`（抛异常，默认）、`CallerRunsPolicy`（交还调用者执行）、`DiscardPolicy`（静默丢弃新任务）、`DiscardOldestPolicy`（丢弃队列最老任务再尝试提交）。

### 4. 线程安全与锁

多个线程**同时读写共享数据**就会出现并发问题（售票超卖）。解决方案三件套：同步代码块 `synchronized(锁对象)`、同步方法、`Lock` 锁（`ReentrantLock`），核心思想都是**让共享资源的访问串行化**。

## 二、技术点之间的联系

```
Stream ──本质是──▶ Lambda + 函数式接口 的集大成者
  filter(Predicate) / map(Function) / sorted(Comparator) / forEach(Consumer)
  集合章的「筛选」「合并」= 同一批逻辑的两种写法（循环 vs 流）

IO ──在并发场景下需要──▶ 多线程
  单线程复制 3 个文件 = 串行等待；每文件一个线程 = 并行提速
  进度百分比：已拷字节 / File.length() → NumberFormat.getPercentInstance()

多线程 ──催生──▶ 线程池（复用线程，避免频繁创建销毁）
  共享资源（票池 / 进度变量）──催生──▶ 锁（synchronized / Lock）
```

三条关键联系：

1. **Stream 的每个参数都是函数式接口**：`filter` 收 `Predicate`、`map` 收 `Function`——Stream 是「接口与 Lambda」的直接应用场。
2. **Stream 与集合是「加工线」与「仓库」的关系**：`list.stream()` 上线加工，`collect(Collectors.toList())` 下线入库；流不存数据，也不改原集合。
3. **IO × 多线程 × 锁三位一体**：并发复制文件时每个线程独立操作各自的流（无线程安全问题），但「汇报总进度」若共享计数器就需要同步——判断是否需要锁的标准：**有没有共享的可变数据**。

## 三、方法与实现要点

### 1. Stream 需求的「翻译」方法

把需求动词直接映射到 API：「过滤」→ filter、「转为/提取」→ map、「排序」→ sorted、「去重」→ distinct、「统计个数」→ count、「求和」→ reduce / mapToInt().sum()、「分组」→ groupingBy、「拍平」→ flatMap。先写一条主链，再把多个需求拆成多条链（**一条流只能消费一次**）。

### 2. 空值防御与比较器细节

- 过滤 null 与空串：`s != null && !s.isEmpty()`（`s == ""` 是字符串比较的经典错误）；
- 排序写 `Comparator.comparingInt(String::length)` 比 `(o1, o2) -> o1.length() - o2.length()` 更可读，且避免大数相减溢出的隐患；
- 降序：`.reversed()` 或比较器里 o2 在前。

### 3. 售票系统的锁设计

- 票池是共享变量 → 售票动作放进 `synchronized` 代码块；
- 锁对象必须**唯一**（三个窗口线程共享同一把锁，如同一个 Runnable 实例的成员对象）；
- 判断「还有票」也要在锁内做，否则会出现「判断完票没了」的超卖。

### 4. 线程池的口诀

「**核心 → 队列 → 临时 → 拒绝**」：任务来了先给核心线程，核心满了进队列排队，队列满了开临时线程，临时也满了触发拒绝策略。背下这个顺序，七大参数自然串起来。

## 四、重点与难点分析

### 重点 1：Stream 中间操作与终端操作的分界

只写 `filter/map` 不写终端操作，代码「看起来在执行」实际什么都没做；一条流调用终端操作后即关闭，不能复用（`IllegalStateException: stream has already been operated upon`）。

### 重点 2：线程池参数与拒绝策略

七大参数 + 「核心→队列→临时→拒绝」触发链 + 四大策略的差异，是并发部分的标配知识。临时线程的意义：应对突发流量，闲时按 keepAliveTime 回收。

### 难点 1：分组聚合的嵌套收集器

`groupingBy(Student::getGender, averagingDouble(Student::getScore))` 是两个收集器的组合：外层按键分组，内层对每组做下游聚合。理解「收集器可以嵌套」，就能组合出分组计数、分组求最值等复杂统计。

### 难点 2：线程安全问题的判定与最小同步范围

不是「用了多线程就要加锁」——并发复制文件时各线程操作各自的流，无需同步；售票时共享票池，必须同步。锁的范围要**最小化**：只锁共享数据的读写片段，锁整个方法会牺牲并发度。

### 易错点清单

- 对同一条流连续调用两个终端操作——第二次直接抛异常；
- 字符串判空用 `== ""`——应用 `isEmpty()`，且先判 null 再判空（短路顺序）；
- 继承 Thread 后调用 `run()` 而不是 `start()`——变成普通方法调用，没有新线程；
- 数字格式化忘设 `setMinimumFractionDigits`——百分比小数位不受控。

## 五、示例

### 示例 1：过滤 + 统计 + 求和（三种终端操作）

```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

nums.stream().filter(n -> n % 2 == 0).forEach(System.out::println);          // 输出偶数
System.out.println(nums.stream().filter(n -> n % 2 == 0).count());          // 5
System.out.println(nums.stream().filter(n -> n % 2 == 0)
        .mapToInt(Integer::intValue).sum());                                // 30
System.out.println(nums.stream().filter(n -> n % 2 == 0)
        .reduce(0, Integer::sum));                                          // 30
```

要点：同一条流只能消费一次，所以每个统计各自 `nums.stream()` 开新流。

### 示例 2：字符串清洗 → 转换 → 排序 → 收集（一条链）

```java
List<String> words = Arrays.asList("apple", "banana", "", "orange", "grape", null, "pear");

List<String> result = words.stream()
        .filter(s -> s != null && !s.isEmpty())        // 1. 去 null 与空串
        .map(String::toUpperCase)                      // 2. 转大写
        .sorted(Comparator.comparingInt(String::length)) // 3. 按长度升序
        .collect(Collectors.toList());                 // 4. 收集
```

要点：能把四步合成一条链，说明已理解「中间操作返回流、终端操作结束流」的管道模型。

### 示例 3：分组 + 组内聚合（嵌套收集器）

```java
Map<String, Double> avgByGender = students.stream()
        .collect(Collectors.groupingBy(
                Student::getGender,                          // 外层：按性别分组
                Collectors.averagingDouble(Student::getScore))); // 内层：组内平均分

List<Student> top = students.stream()
        .filter(s -> s.getScore() >= 80)
        .sorted(Comparator.comparingDouble(Student::getScore).reversed())
        .collect(Collectors.toList());
```

### 示例 4：去重 + 最值 + 平方和

```java
int[] arr = {3, 5, 2, 8, 5, 9, 2, 10, 3};

List<Integer> distinct = Arrays.stream(arr).boxed().distinct().collect(Collectors.toList());
int max = Arrays.stream(arr).max().orElse(0);
int min = Arrays.stream(arr).min().orElse(0);

int sum = Arrays.stream(arr)
        .filter(n -> n > 5)
        .map(n -> n * n)
        .distinct()
        .reduce(0, Integer::sum);        // 8²+9²+10² = 245
```

要点：`IntStream` 的 `max()` 返回 `OptionalInt`，必须 `orElse` 兜底；链式顺序会影响结果——先 filter 再 map 再 distinct，才是「大于 5 的元素的平方去重」。

### 示例 5：两集合筛选 + 合并 + 对象转换

```java
List<String> teamA = team1.stream()
        .filter(n -> n.length() == 3 && n.endsWith("风"))
        .limit(6)
        .collect(Collectors.toList());

List<String> teamB = team2.stream()
        .filter(n -> n.startsWith("张"))
        .skip(1)                       // 不要第一个
        .collect(Collectors.toList());

List<Student> squad = Stream.concat(teamA.stream(), teamB.stream())
        .map(Student::new)
        .collect(Collectors.toList());
```

### 示例 6：嵌套集合扁平化 + 词频统计

```java
List<List<Integer>> nested = Arrays.asList(
        Arrays.asList(1, 2, 3), Arrays.asList(4, 5, 6),
        Arrays.asList(7, 8, 9), Arrays.asList(1, 2, 3));

List<Integer> flat = nested.stream().flatMap(List::stream).collect(Collectors.toList());

Map<Integer, Long> freq = flat.stream()
        .collect(Collectors.groupingBy(n -> n, Collectors.counting()));

Integer most = freq.entrySet().stream()
        .max(Map.Entry.comparingByValue())
        .map(Map.Entry::getKey).orElse(-1);
```

要点：`flatMap` 是「一对多再拍平」，`groupingBy(n -> n, counting())` 是词频统计的标准写法。

### 示例 7：字节流复制文件（缓冲区版）

```java
public static void copy(String src, String dest) throws IOException {
    try (InputStream in = new BufferedInputStream(new FileInputStream(src));
         OutputStream out = new BufferedOutputStream(new FileOutputStream(dest))) {
        byte[] buf = new byte[8192];       // 一次读一批，避免一个字节一次系统调用
        int len;
        while ((len = in.read(buf)) != -1) {
            out.write(buf, 0, len);
        }
    }                                    // try-with-resources 自动关流
}
```

### 示例 8：两种线程创建方式 + join

```java
// 方式一：继承 Thread
class PrintOneToTen extends Thread {
    public PrintOneToTen() { super("继承式线程"); }
    @Override public void run() {
        for (int i = 1; i <= 10; i++) System.out.println(getName() + "：" + i);
    }
}

// 方式二：实现 Runnable（推荐）
Thread t2 = new Thread(() -> {
    for (int i = 11; i <= 20; i++) System.out.println(Thread.currentThread().getName() + "：" + i);
}, "实现式线程");

PrintOneToTen t1 = new PrintOneToTen();
t1.start();          // start 才会开新线程；run 只是普通方法调用
t2.start();
t1.join();           // 主线程等待 t1 结束
t2.join();
System.out.println("所有线程执行完成");
```

### 示例 9：售票系统的最小同步范围

```java
public class TicketTask implements Runnable {
    private int tickets = 100;
    private final Object lock = new Object();     // 锁对象唯一且是成员

    @Override public void run() {
        while (true) {
            synchronized (lock) {                 // 判断与扣减必须在同一把锁内
                if (tickets <= 0) break;
                System.out.println(Thread.currentThread().getName() + " 卖出：" + tickets--);
            }
        }
    }
}
// 三个窗口共享同一个 Runnable 实例 → 共享票池与锁
```

要点：`tickets <= 0` 的判断若放到锁外，两个线程可能同时通过判断，造成超卖。

### 示例 10：并发复制文件 + 百分比进度

```java
public static void copyWithProgress(File src, File dest) throws Exception {
    long total = src.length();
    try (InputStream in = new BufferedInputStream(new FileInputStream(src));
         OutputStream out = new BufferedOutputStream(new FileOutputStream(dest))) {
        byte[] buf = new byte[8192];
        long copied = 0;
        int len;
        NumberFormat fmt = NumberFormat.getPercentInstance();
        fmt.setMinimumFractionDigits(2);           // 保留两位小数
        while ((len = in.read(buf)) != -1) {
            out.write(buf, 0, len);
            copied += len;
            double ratio = (double) copied / total;
            // 每个线程打印自己的进度，不涉及共享可变数据
            System.out.println(Thread.currentThread().getName() + " " + fmt.format(ratio));
        }
    }
}
```

要点：进度 = 已拷字节 / 文件总大小；每个线程操作**各自的流和自己的局部变量**，所以无需加锁——这正是判断是否需要同步的典型案例。

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / Stream、IO 与多线程 / 05 Stream、IO 与多线程技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/DzlMw59k5itAM0kYDnXcxQOvnWb)
