---
type: memo
date: 2026-11-06
aliases:
  - "03 Collection 与 Map 的使用技术要点"
---

# 03 Collection 与 Map 的使用技术要点

> 本章围绕「集合框架选型 → 迭代器安全 → 集合承载业务对象 → Map 双向映射」层层递进，是 Java 集合从「会用 API」走向「会做设计」的关键一章。

## 一、技术概念说明

### 1. Collection：单列集合的根接口

所有单列集合的共性功能都定义在这里，这组方法是高频操作的基础：

| 方法 | 作用 |
|-|-|
| `add(E e)` | 添加元素 |
| `addAll(Collection c)` | 把另一个集合的元素并入本集合 |
| `remove(Object o)` | 按内容删除元素 |
| `contains(Object o)` | 判断是否包含某元素 |
| `clear()` | 清空集合 |
| `size()` / `isEmpty()` | 长度 / 判空 |

### 2. List 与 Set：两大分支的特性对比

| 分支 | 实现类 | 特点 |
|-|-|-|
| **List** | ArrayList / LinkedList | **有序（存取顺序一致）、有索引**、**可重复** |
| **Set** | HashSet | 无序、无索引、**不可重复** |
|  | LinkedHashSet | 有序、无索引、不可重复 |
|  | TreeSet | **自动排序**、无索引、不可重复 |

选型一句话：要按下标操作、允许重复 → List；要去重 → HashSet；要去重且保序 → LinkedHashSet；要去重且排序 → TreeSet。

### 3. ArrayList vs LinkedList

- **ArrayList**：底层数组，查询快（按下标 O(1)），增删慢（要搬移元素）；
- **LinkedList**：底层双向链表，首尾增删快，查询慢。  
日常开发 **90% 场景用 ArrayList**，只有频繁首尾增删才考虑 LinkedList。

### 4. Iterator：集合的统一遍历器

- `hasNext()` 问「还有没有」；`next()` 取下一个；`remove()` 删除**刚取出的那个**。
- 三条铁律：**一次 next 只能配一次 remove**；**必须先 next 才能 remove**；**遍历中禁用集合自身的增删方法**。

### 5. Map：双列键值对集合

`HashMap<K, V>` 以键找值。Map 的高阶用法是**双向映射**：`Club → List<Student>` 与 `Student → List<Club>` 两张表互为主键索引，换任意方向查询都是 O(1) 定位 + O(n) 遍历列表。

### 6. equals 与 hashCode：Set/Map 去重的钥匙

HashSet 去重、HashMap 键定位，依赖的判定规则是：**先比 hashCode，再用 equals 复核**。所以自定义类（Student、Club）作为 Set 元素或 Map 键时，**必须同时重写 equals 和 hashCode**，否则「内容相同的两个对象」会被当成两个不同元素。

## 二、技术点之间的联系

```
Collection（根接口，定共性功能）
  ├─ List（有序有索引可重复）── ArrayList / LinkedList
  │     │  遍历三件套：fori / 增强for / Iterator
  │     │  多条件筛选 = 遍历 + if 条件组合
  │     │  承载业务对象：ArrayList<Product> → 购物车
  │     │
  └─ Set（不可重复）── HashSet / LinkedHashSet / TreeSet
        │  去重能力依赖：equals + hashCode
        │
Map（键值对）
  └─ HashMap：键的去重同样依赖 equals + hashCode
        高阶结构：Map<K, List<V>> → 一对多关系（社团-成员）
        双 Map 互相映射 → 双向高效查询

迭代器安全 ──贯穿所有遍历场景
  并发修改异常 ←→ 正确姿势：it.remove() / 普通 fori
```

三条关键联系：

1. **「集合选型」决定「能用什么操作」**：要用索引反序装填 → 必须 List；要按 ID 快速删除商品 → `HashMap<Integer, Product>`（ID 作键，O(1) 定位）比遍历 List 更专业。
2. **多条件筛选是 Stream 的预热**：本质都是「遍历 + 条件 + 收集」，与 Stream 的 `filter/map/collect` 一一对应，现在用循环写熟逻辑，后面换 Stream 只是换写法。
3. **equals/hashCode 是 Set 和 Map 共同的底层约定**：Set 元素、Map 的键，去重与定位都走「hashCode 分桶 → equals 确认」两步，这也是为什么两个方法必须成对重写、且字段保持一致。

## 三、方法与实现要点

### 1. 迭代器异常的对号入座

- `NoSuchElementException` → 循环里 **next 了两次**（`hasNext` 只保证一个）；
- `ConcurrentModificationException` → 遍历中用了 **集合自身的 remove**（应改 `it.remove()`）；
- `IllegalStateException` → 没 next 就 remove，或一次 next 配了多次 remove。

### 2. 遍历中安全删除的三种姿势

1. **迭代器删**：`it.next()` 后 `it.remove()`；
2. **普通 fori 倒序删**：从后往前遍历，删了不影响前面索引；
3. **新建集合收集**：不删旧的，把符合条件的放进新集合（最不容易出错，适合新手）。

### 3. 多条件筛选的拆解方法

把自然语言翻译成条件表达式是核心能力：

| 题意 | 翻译要点 |
|-|-|
| 「3 次及以上」 | `failCount >= 3`（注意「及以上」含边界） |
| 「南宫婉 **和** 陆雪琪」 | 是「班主任 ∈ {南宫婉, 陆雪琪}」的**或**关系，不是一个变量同时等两个值 |
| 「**不是** 安妮**和** 张远」 | `!equals("安妮") && !equals("张远")`，德摩根律：非(A或B) = 非A且非B |
| 「及格次数超过 2 次」 | 总考 5 次 → `5 - failCount > 2`，即 `failCount < 3` |
| 「名字提取」 | 遍历中只取 `getName()`，是 map 操作的雏形 |

### 4. 业务对象集合的架构意识

- **购物车**：`ArrayList<Product>` 承载业务对象，「添加已存在则累加数量」体现**先查后写**的防御逻辑；`updateQuantity` 数量 ≤ 0 转删除，是业务规则内聚到方法里的好示范。
- **社团管理**：双向 HashMap 是「以空间换查询效率」的经典结构，关键是**注册/移除时两张表必须同步更新**，否则数据不一致——这就是数据库外键维护、缓存一致性的思想源头。

## 四、重点与难点分析

### 重点 1：List/Set 特性与选型

「有序、有索引、可重复」九个字概括 List；「不可重复」概括 Set 家族，再按「无序/保序/排序」区分 HashSet、LinkedHashSet、TreeSet。给场景选集合是开发的基本功。

### 重点 2：迭代器三铁律

一次 next 一次 remove、先 next 后 remove、遍历中不动集合本身。违反任何一条都抛**运行时异常**——编译不报错，必须靠记忆规避。

### 难点 1：并发修改异常的成因

迭代器创建时记录了集合的修改计数 `modCount`，每次 next 都会校验；集合自身 `remove` 会改动 `modCount` 导致校验失败、直接抛异常。`it.remove()` 之所以安全，是因为它会**同步更新迭代器内部的计数**。理解了这一点，就明白为什么增强 for 循环（底层就是迭代器）里同样不能 `list.remove()`。

### 难点 2：自定义对象作为 HashSet/Map 键的正确姿势

只重写 equals 不重写 hashCode 的后果：两个内容相同的对象 hashCode 不同 → 落在不同桶 → 永远走不到 equals 比较 → 去重失效。重写时两个方法必须**使用相同的字段**（如 Student 用 studentId + name + grade 同时参与两个方法）。

### 易错点清单

- 字符串比较用 `==` 而不是 `equals`（`teacher == "南宫婉"` 会漏判）；
- 增强 for 循环中调用 `list.remove()`——必抛并发修改异常；
- `next()` 取出后没有保存就再次 `next()`，数据被跳过；
- 用 ArrayList 频繁做中间插入删除——选型失误，应考虑 LinkedList。

## 五、示例

### 示例 1：List / Set 基本特性对照

```java
List<String> list = new ArrayList<>();
list.add("Java"); list.add("MySQL"); list.add("Java");
System.out.println(list);            // [Java, MySQL, Java]，有序、可重复

Set<String> set = new HashSet<>(list);
System.out.println(set);             // [Java, MySQL]，去重、无序

Set<String> linked = new LinkedHashSet<>(list);
System.out.println(linked);          // [Java, MySQL]，去重、保持插入顺序

Set<String> tree = new TreeSet<>(list);
System.out.println(tree);            // [Java, MySQL]，去重、自然排序
```

### 示例 2：迭代器规范遍历

```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();            // 一次 next 只取一个，先保存再用
    if (s.length() > 4) {
        System.out.println(s);
    }
}
```

### 示例 3：遍历中删除非法数据（QQ 号清洗）

```java
List<String> qqs = new ArrayList<>(Arrays.asList(
        "123", "1234", "12345", "654123", "456789123", "1112222333444", "555555"));

Iterator<String> it = qqs.iterator();
while (it.hasNext()) {
    String qq = it.next();
    if (qq.length() < 5 || qq.length() > 10) {
        it.remove();                 // 用迭代器自己的 remove，不会触发并发修改异常
    }
}
System.out.println(qqs);             // [12345, 654123, 456789123, 555555]
```

要点：长度校验用 `length() < 5 || length() > 10` 一次表达区间外，比两个独立 if 更清晰。

### 示例 4：并发修改异常的正确修正

```java
// 错误写法：遍历中调用集合自身 remove
for (String s : list) {
    if ("b".equals(s)) list.remove(s);      // ConcurrentModificationException
}

// 修正一：迭代器删除
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if ("b".equals(it.next())) it.remove();
}

// 修正二：倒序 fori 删除
for (int i = list.size() - 1; i >= 0; i--) {
    if ("b".equals(list.get(i))) list.remove(i);
}

// 修正三：过滤到新集合
List<String> result = new ArrayList<>();
for (String s : list) {
    if (!"b".equals(s)) result.add(s);
}
```

### 示例 5：多条件筛选学生

```java
// 不及格 3 次及以上
List<Student> r1 = new ArrayList<>();
for (Student s : students) {
    if (s.getFailCount() >= 3) r1.add(s);
}

// 班主任是南宫婉或陆雪琪，且及格次数超过 2 次（总考 5 次 → failCount < 3）
int count = 0;
for (Student s : students) {
    boolean teacherMatch = "南宫婉".equals(s.getTeacher()) || "陆雪琪".equals(s.getTeacher());
    if (teacherMatch && (5 - s.getFailCount()) > 2) count++;
}

// 不是安妮和张远，且不及格超过 2 次
List<Student> r3 = new ArrayList<>();
for (Student s : students) {
    if (!"安妮".equals(s.getTeacher()) && !"张远".equals(s.getTeacher())
            && s.getFailCount() > 2) {
        r3.add(s);
    }
}
```

### 示例 6：ArrayList 承载业务对象（购物车）

```java
public class ShoppingCart {
    private final List<Product> products = new ArrayList<>();

    public void addProduct(Product product) {
        for (Product p : products) {
            if (p.getId() == product.getId()) {            // 先查后写
                p.setQuantity(p.getQuantity() + product.getQuantity());
                return;
            }
        }
        products.add(product);
    }

    public void removeProduct(int id) {
        for (int i = 0; i < products.size(); i++) {
            if (products.get(i).getId() == id) {
                products.remove(i);                        // fori 删除后立即 return
                return;
            }
        }
    }

    public void updateQuantity(int id, int newQuantity) {
        for (int i = 0; i < products.size(); i++) {
            Product p = products.get(i);
            if (p.getId() == id) {
                if (newQuantity <= 0) products.remove(i);   // 业务规则内聚在方法里
                else p.setQuantity(newQuantity);
                return;
            }
        }
    }

    public double calculateTotalPrice() {
        double total = 0;
        for (Product p : products) total += p.getPrice() * p.getQuantity();
        return total;
    }

    public void clearCart() { products.clear(); }
}
```

### 示例 7：双向 HashMap 实现社团双向查询

```java
public class ClubManager {
    private final Map<Club, List<Student>> clubStudents = new HashMap<>();
    private final Map<Student, List<Club>> studentClubs = new HashMap<>();

    public void registerStudent(Club club, Student student) {
        clubStudents.computeIfAbsent(club, k -> new ArrayList<>()).add(student);
        studentClubs.computeIfAbsent(student, k -> new ArrayList<>()).add(club);
        // 两张表必须同步更新，否则双向查询结果不一致
    }

    public List<Student> getStudentsByClub(Club club) {
        return clubStudents.getOrDefault(club, Collections.emptyList());
    }

    public List<Club> getClubsByStudent(Student student) {
        return studentClubs.getOrDefault(student, Collections.emptyList());
    }

    public Map<Club, Integer> countStudentsPerClub() {
        Map<Club, Integer> result = new HashMap<>();
        for (Map.Entry<Club, List<Student>> e : clubStudents.entrySet()) {
            result.put(e.getKey(), e.getValue().size());
        }
        return result;
    }

    public void removeStudentFromClub(Club club, Student student) {
        List<Student> members = clubStudents.get(club);
        if (members != null) members.remove(student);
        List<Club> clubs = studentClubs.get(student);
        if (clubs != null) clubs.remove(club);
    }

    public List<Club> getClubsByType(String type) {
        List<Club> result = new ArrayList<>();
        for (Club club : clubStudents.keySet()) {
            if (type.equals(club.getType())) result.add(club);
        }
        return result;
    }
}
```

要点：`Club` 与 `Student` 作为 Map 的键，**必须重写 equals 与 hashCode**（用相同的字段），否则同一个社团对象换个实例就查不到成员。`computeIfAbsent` 省掉「判空 → 创建 → put」三行样板代码。

### 示例 8：过滤 + 转换 + 合并的集合流水线

```java
// 1. 队伍一：名字 3 个字且以「风」结尾，取前 6 个
List<String> team1 = new ArrayList<>();
for (String name : list1) {
    if (name.length() == 3 && name.endsWith("风")) team1.add(name);
}
if (team1.size() > 6) team1 = team1.subList(0, 6);

// 2. 队伍二：姓张，去掉第一个
List<String> team2 = new ArrayList<>();
for (String name : list2) {
    if (name.startsWith("张")) team2.add(name);
}
if (!team2.isEmpty()) team2.remove(0);

// 3. 合并
List<String> merged = new ArrayList<>(team1);
merged.addAll(team2);

// 4. 字符串 → Student 对象
List<Student> students = new ArrayList<>();
for (String name : merged) students.add(new Student(name));
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 集合框架与 Map / 03 Collection 与 Map 的使用技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/FDiHwGUb3iyMhLk1GjkcukhjnHh)
