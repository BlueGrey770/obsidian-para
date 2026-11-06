---
type: memo
date: 2026-11-06
aliases:
  - "04 集合相关原理技术要点"
---

# 04 集合相关原理技术要点

> 如果说「集合的使用」解决的是「怎么用」，本章解决的是「为什么」——搞懂底层结构，才能解释性能差异、预测异常、写出经得起追问的代码。

## 一、技术概念说明

### 1. 迭代器（Iterator）的遍历原理

迭代器是**通用的集合遍历方式**，与集合的具体实现解耦，使用分两步：

1. `集合.iterator()` 获取该集合的迭代器对象；
2. 用 `hasNext()` 判断 + `next()` 取元素，循环推进内部的「游标」。

增强 for 循环（`for (String s : list)`）底层就是迭代器，所以它的行为约束与迭代器完全一致。

### 2. 并发修改异常（ConcurrentModificationException）的成因

迭代器创建时记录集合的修改计数 `modCount`；每次 `next()` 都校验计数是否变化。**遍历中调用集合自身的增删方法**（如 `list.remove("b")`）会改动 `modCount`，校验失败即抛异常。正确做法：用**迭代器自己的 `remove()`**——它删除元素后会同步更新计数。

### 3. ArrayList：数组结构

- 底层是**数组**：内存中开辟**连续空间**，数组名记录首地址，0 索引旁一定是 1 索引；
- 特点：**查询快（首地址 + 索引 × 元素大小直接定位，O(1)）；增删慢**（中间增删要整体搬移后续元素，O(n)）；
- 容量不足时自动**扩容**（约 1.5 倍），扩容 = 建新数组 + 拷贝，有成本。

### 4. LinkedList：双向链表结构

- 底层是**双向链表**：节点在内存中**散列分布**，每个节点用指针记录前驱和后继，「对象记录对象」；
- 特点：**首尾操作快（O(1) 拿到头尾节点）；查询慢**（必须从一端逐个走到目标，O(n)）。

### 5. HashMap：数组 + 链表（+ 红黑树）的哈希表

存储一个键值对的完整流程：

1. **扰动**：计算键的 `hashCode`，再 **hash >>> 16**（让高 16 位参与运算，减少冲突）；
2. **寻址**：与哈希表长度做**位与（&）运算**得到桶索引；
3. **空桶直存**：该位置为 null，封装成 Entry 节点直接放入；
4. **冲突处理**：该位置已有节点，则遍历此桶所有节点，用 **hashCode + equals 双重比对**——

   - 都相同 → 判定重复，**新值覆盖旧值**，结束；
   - 都不同 → 封装新节点**头插**到链表头部；
5. **扩容**：元素数超过「容量 × 负载因子（默认 0.75）」触发扩容，**默认 2 倍**，所有元素重新计算位置重新分布。

链表过长（树化阈值 8）且表长 ≥64 时，链表转**红黑树**提升查询效率。

### 6. 红黑树的四条规则

1. 根节点是黑色；2. 叶子节点（NIL）是黑色；3. 两个红色节点不能相连；4. 任一节点到其所有叶子节点的简单路径上，**黑色节点数量相同**（黑高一致）。规则保证树的「近似平衡」，查询稳定 O(log n)。

## 二、技术点之间的联系

```
数据结构三分天下
  ├─ 数组（连续空间）── ArrayList 的底子 ── 查询快增删慢
  ├─ 链表（散列空间+指针）── LinkedList 的底子 ── 首尾快查询慢
  │     └─ 删除链表节点 = 亲手操作一次 LinkedList 的内部动作
  └─ 哈希表（数组+链表+红黑树）── HashMap 的底子
        ├─ 寻址：hash>>>16 & (len-1) —— 位运算代替取模，要求长度是 2 的幂
        ├─ 去重：hashCode 分桶 + equals 复核 —— 解释为何要成对重写
        └─ 树化：链表 → 红黑树 —— 用红黑树规则保证极端情况不退化

迭代器 ──站在所有结构之上的统一抽象
  └─ 并发修改异常：modCount 校验机制（与具体结构无关）

应用层综合
  ├─ 分数背包：对象封装 + ArrayList 装载 + Collections.sort(Lambda 比较器)
  └─ StringBuilder：可变的「字符数组缓冲区」—— 与 String 不可变性的对比
```

三条关键联系：

1. **「结构决定性能」贯穿全篇**：ArrayList 快在索引、慢在搬移；LinkedList 快在首尾、慢在遍历；HashMap 快在寻址、怕在冲突——所有结论都能从底层结构推导出来，不用死记。
2. **HashMap 的「头插法 + 扩容」解释了它的无序性**：元素位置由 hash 决定，扩容后位置全变，所以 HashMap 既不保证插入顺序、也不保证顺序稳定——这是 LinkedHashMap/TreeMap 存在的理由。
3. **链表删除是 LinkedList 原理的逆向应用**：理解了「对象记录对象」，就理解为什么删除只要改 `next` 指针、以及为什么虚拟头节点（dummy）能统一「删头」与「删中间」的逻辑。

## 三、方法与实现要点

### 1. 用「结构三问」分析任何集合

遇到陌生集合类，问三个问题：底层什么结构？读写各走什么路径？极端情况（扩容/冲突/树化）怎么应对？——能把 HashMap 存储流程完整口述一遍，集合篇就过了一半。

### 2. 分数背包的贪心框架

「只能取 10 升、每样可取部分」是典型的**分数背包**，贪心策略：按**单位价值**降序排序，能全拿全拿、最后一样补满即止。三个值得学的点：

- 构造时算好 `unitValue = value / weight`（派生属性早算）；
- `Collections.sort(list, (o1, o2) -> Double.compare(o2.getUnitValue(), o1.getUnitValue()))`——Lambda 比较器降序的标准写法（注意是 o2 在前）；
- 循环中「全拿」与「部分拿」用 `if-else + break` 分界。

### 3. 链表删除节点的标准解法（虚拟头节点）

核心技巧：**造一个 dummy 节点指向 head**，把「删除头节点」变成和「删除中间节点」相同的操作，免去特判。循环不变式：`current.next` 是待检查节点——命中就 `current.next = current.next.next` 跳过它，不命中才 `current = current.next` 前进（注意命中后**不能前进**，否则漏查连续重复值）。

### 4. StringBuilder 的效率意识

字符串常量池决定了 String 每次 `+` 都产生新对象，循环拼接是 O(n²) 的对象创建；StringBuilder 是可变缓冲区，append 原地扩展，最后 `toString()` 一次成型。**凡遇循环拼接，必用 StringBuilder**。

## 四、重点与难点分析

### 重点 1：HashMap 存储与去重全流程

扰动（>>>16）→ 位与寻址 → 空桶直存 → 冲突时 hashCode + equals 双检 → 相同覆盖/不同头插 → 超阈值 2 倍扩容重排 → 链表过长转红黑树。这条链路每一步都能被单拎出来追问。

### 重点 2：为什么重写 equals 必须重写 hashCode

只重写 equals 不重写 hashCode：两个 equals 为 true 的对象 hashCode 不同 → 落在不同桶 → 去重失效、get 找不到。约定：**equals 为 true 的对象必须有相同的 hashCode**，且两个方法应使用相同字段。

### 难点 1：位与寻址为什么要求容量是 2 的幂

`hash & (length-1)` 只有当 length 是 2 的幂时才等价于 `hash % length`，且分布均匀；这解释了 HashMap 容量为何总是 16/32/64，扩容为何总是 2 倍。

### 难点 2：红黑树「黑高一致」规则的意义

四条规则合起来保证：**最长路径不超过最短路径的 2 倍**——树不会退化成链表，任何查找都在 O(log n) 内完成。JDK 树化机制正是为了防止哈希冲突攻击导致链表过长。

### 易错点清单

- 误以为增强 for 可以安全 `list.remove()`——底层仍是迭代器，同样触发并发修改异常；
- 链表删除时命中后继续 `current = current.next`——连续两个目标值时会漏删第二个；
- 贪心按「总价值」排序而不是「单位价值」——策略错误，结果不是最优解；
- 忘记 HashMap 扩容后元素位置会重排——误以为迭代顺序稳定。

## 五、示例

### 示例 1：分数背包（对象 + ArrayList + Lambda 比较器）

有若干种液体，各有重量与总价值，只能总共取走 10 升，每种可不拿、全拿或拿一部分，求最大价值。

```java
public class Item {
    private String name;
    private double weight;
    private double value;
    private double unitValue;          // 派生属性：单位价值，构造时算好

    public Item(String name, double weight, double value) {
        this.name = name;
        this.weight = weight;
        this.value = value;
        this.unitValue = value / weight;
    }
    public double getUnitValue() { return unitValue; }
    public String getName() { return name; }
    public double getWeight() { return weight; }
    public double getValue() { return value; }
}

public class Test {
    public static void main(String[] args) {
        List<Item> list = new ArrayList<>();
        list.add(new Item("水", 4, 24));
        list.add(new Item("牛奶", 8, 160));
        list.add(new Item("五粮液", 2, 4000));
        list.add(new Item("可乐", 6, 108));
        list.add(new Item("茅台", 1, 4000));

        // 按单位价值降序：o2 在前才是降序
        Collections.sort(list, (o1, o2) -> Double.compare(o2.getUnitValue(), o1.getUnitValue()));

        double sum = 0, money = 0;
        for (Item item : list) {
            if (sum + item.getWeight() < 10) {          // 能全拿
                sum += item.getWeight();
                money += item.getValue();
                System.out.println(item.getName() + "用了" + item.getWeight() + "升");
            } else {                                     // 最后一样补满即止
                double take = 10 - sum;
                money += item.getUnitValue() * take;
                System.out.println(item.getName() + "用了" + take + "升");
                break;
            }
        }
        System.out.println("一共花了" + money + "元");
    }
}
```

要点：贪心的正确性来自「可分割」这一前提；排序键必须是**单位价值**而不是总价值，否则先拿大件会把容量占死。

### 示例 2：链表删除指定值的所有节点（虚拟头节点）

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}

public static ListNode deleteNodes(ListNode head, int val) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;                 // 虚拟头节点，统一删头与删中间
    ListNode current = dummy;
    while (current.next != null) {
        if (current.next.val == val) {
            current.next = current.next.next;   // 跳过，命中后不前进
        } else {
            current = current.next;
        }
    }
    return dummy.next;
}
```

要点：命中删除后**不能**`current = current.next`，否则连续两个 `6`（如 `6 -> 6`）会漏删第二个。

### 示例 3：StringBuilder 拼接多段落文本

```java
public static String merge(String[] paragraphs) {
    StringBuilder sb = new StringBuilder();
    int index = 1;
    for (String p : paragraphs) {
        if (p == null || p.isEmpty()) continue;      // 空段落跳过，但编号不占位
        sb.append("[").append(index++).append("]. ")
          .append(capitalizeWords(p))
          .append("\n\n");
    }
    return sb.toString();
}

// 每个单词首字母大写（非字母字符后的第一个字母）
private static String capitalizeWords(String s) {
    StringBuilder sb = new StringBuilder();
    boolean upperNext = true;
    for (char c : s.toCharArray()) {
        if (Character.isLetter(c)) {
            sb.append(upperNext ? Character.toUpperCase(c) : c);
            upperNext = false;
        } else {
            sb.append(c);
            upperNext = true;             // 遇到非字母，下一个字母要大写
        }
    }
    return sb.toString();
}
```

输入 `{"hello, world! this is a test.", "java programming is fun.", "", "goodbye, see you later."}` 时输出 `[1]. Hello, World! This Is A Test.` / `[2]. Java Programming Is Fun.` / `[3]. Goodbye, See You Later.`，空段落不参与拼接也不占编号。

### 示例 4：HashMap 存取的代码观察

```java
Map<String, Integer> map = new HashMap<>();
map.put("Java", 90);
map.put("MySQL", 85);
map.put("Java", 95);          // 键重复 → 覆盖旧值，size 仍为 2

System.out.println(map.get("Java"));     // 95
System.out.println(map.containsKey("Redis"));   // false
System.out.println(map.getOrDefault("Redis", 0)); // 0

for (Map.Entry<String, Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + " = " + e.getValue());
}
```

要点：`put` 的覆盖行为就是「hashCode 相同 + equals 相同 → 替换 value」的直观体现；`getOrDefault` 免去 null 判断。

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 集合框架与 Map / 04 集合相关原理技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/GzOswtNbeipBAXk3pDAcRiQ6njb)
