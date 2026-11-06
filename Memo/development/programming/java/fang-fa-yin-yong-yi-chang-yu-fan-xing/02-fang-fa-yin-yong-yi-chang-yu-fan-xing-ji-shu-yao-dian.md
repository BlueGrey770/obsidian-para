---
type: memo
date: 2026-11-06
aliases:
  - "02 方法引用、异常与泛型技术要点"
---

# 02 方法引用、异常与泛型技术要点

> 三个主题看似独立，实则有共同目标：**让代码更安全（异常）、更通用（泛型）、更简洁（方法引用）**。

## 一、技术概念说明

### 1. 异常（Exception）：程序的非正常事件

程序在**编译或运行期间**出现的不正常操作或现象。注意：异常**不是语法错误**——语法错误编译都过不去，异常是语法正确但逻辑上出了问题（如除零、数组越界、空指针）。Java 用**异常对象**描述问题：哪里出错、什么类型的错、错误原因，都封装在对象里。

**异常的两大家族**：

| 类型 | 代表类 | 特征 | 举例 |
|-|-|-|-|
| 编译时异常（受检异常） | `Exception` 及其非 Runtime 子类 | 编译器**强制处理**，不处理编译不过 | 文件找不到、日期解析失败 |
| 运行时异常（非受检异常） | `RuntimeException` 及其子类 | 编译不强制，运行才暴露，多为常识性错误 | 空指针、数组越界、类型转换错误 |

### 2. 异常处理的两种方式

- **抛（throws）**：方法声明处用 `throws 异常类型` 把异常甩给调用者处理。适合「本方法解决不了」的场景，但一直抛到 JVM 程序就崩溃了。
- **抓（try-catch）：就地处理。try 中一旦出现异常，剩余代码立即不再执行**，从上到下匹配 catch，命中即执行对应处理块并结束整个语句；try 中无异常则 catch 全部跳过。

### 3. 自定义异常

**步骤**：定义类继承 `Exception`（编译时异常）或 `RuntimeException`（运行时异常），重写空参构造和带原因构造。  
**意义**：让异常「简明知意」——`AgeOutOfBoundsException` 比笼统的 `RuntimeException` 更专业地描述业务问题。

### 4. 泛型（Generics）：把类型当参数传

泛型让类/接口/方法在定义时不写死类型，使用时再指定，实现「一套代码，多种类型」。

**确定泛型的时机**（三种形态三种时机）：

| 形态 | 确定时机 |
|-|-|
| 泛型类 | **创建对象时**确定，每 new 一次确定一次，多个对象互不干扰 |
| 泛型接口 | ① 实现类定义时直接写死（退化成普通类）；② 实现类继续声明泛型，把决定权顺延给使用者（变成泛型类） |
| 泛型方法 | **调用方法时**由传入参数自动推断 |

### 5. 泛型限定与通配符

- **泛型限定**：定义泛型的人限制类型范围，只有上限写法——`<T extends Number>` 要求 T 是 Number 或其子类。
- **通配符 `?`**：使用别人定义的泛型类型时做限制：

  - `?` 不限；`? extends 类型` 上限（该类型及其子类）；`? super 类型` 下限（该类型及其父类）。

### 6. 方法引用：Lambda 的再简化

当 Lambda 体只是调用一个**已存在的方法**时，用 `::` 直接引用。四种格式：

| 格式 | 示例 |
|-|-|
| 静态方法引用 `类名::静态方法` | `BinaryOperation op = Calculator::add;` |
| 实例方法引用 `对象::实例方法` | `BinaryOperation op = calculator::multiply;` |
| 特定类型的实例方法 `类名::实例方法` | `String::toUpperCase` |
| 构造器引用 `类名::new` | `ArrayList::new` |

## 二、技术点之间的联系

```
方法引用 ──是──▶ Lambda 的特例 ──依赖──▶ 函数式接口

异常体系 ──保障──▶ 程序健壮性
  │  try-catch：就地消化       throws：责任上移
  │  自定义异常：业务化表达
  │  IllegalArgumentException：方法参数校验的标配
  │
泛型 ──保障──▶ 类型安全 + 代码复用
  │  泛型方法 <T> ──配合──▶ 包装类（Integer/Double 等 Number 子类）
  │  泛型限定 <T extends Number> ──打通──▶ 包装类的 doubleValue() 统一取值
```

三条关键联系：

1. **方法引用依附于函数式接口**：`Calculator::add` 能赋给 `BinaryOperation`，是因为该接口只有一个抽象方法 `operate(int,int)`，且签名与 `add(int,int)` 完全兼容——**签名兼容是方法引用成立的充要条件**。
2. **异常与泛型天然配合**：`swap` 里索引越界抛 `IndexOutOfBoundsException`、`sum` 靠 `Number` 上限拿到 `doubleValue()`——泛型负责「类型对」，异常负责「数据对」。
3. **包装类是泛型的幕后基石**：泛型不能用于基本类型（`List<int>` 非法），必须用包装类；`Double.parseDouble` 等静态方法还承担了字符串 → 数值的解析职责。

## 三、方法与实现要点

### 1. 方法引用的三步骤

固定套路：**先定义函数式接口（签名与目标方法一致）→ 用 `::` 把现成方法赋给接口变量 → 调接口方法**。判断能否引用，只做一件事：比对参数列表和返回值是否匹配。

### 2. 泛型方法定义的两步

- 在返回值前声明 `<T>`（如 `public static <T> void swap(T[] array, int i, int j)`）；
- 方法体内把 T 当普通类型用。  
需要调用具体方法时加上限定（`<T extends Number>` 后就能用 `num.doubleValue()`）。

### 3. 异常处理的选型口诀

- 能就地解决（如给个默认值、重新输入）→ try-catch；
- 解决不了或该由调用方知情 → throws 上抛；
- 参数非法 → 主动 `throw new IllegalArgumentException("原因")`，**尽早失败**比带病运行好。

### 4. try-catch 的两个实操要点

- try 中异常点之后的代码不会执行，所以「出问题后不需要执行的代码」放 try 里；
- 多个 catch 有继承关系时，**子类在上、父类在下**，否则父类先截胡，子类 catch 永远不可达（编译报错）。

### 5. 字符串数值解析的通用模式

`split` 切分 → `包装类.parseXxx` 转换 → 遍历比较。注意 `max` 初值取第一个元素而非 0，避免全负数数组出错。

## 四、重点与难点分析

### 重点 1：编译时异常 vs 运行时异常

- 编译时异常：`Exception` 下非 Runtime 分支，**必须处理**（throws 或 try-catch），否则编译失败；
- 运行时异常：`RuntimeException` 及其子类，编译不强制，属常识性编码错误（越界、空指针），应靠**规范编码**预防而非到处 try。
- 判别方法：看继承链上有没有 `RuntimeException`。

### 重点 2：泛型确定的三个时机（必背）

泛型类 → new 对象时；泛型接口 → 实现类定义时（或顺延）；泛型方法 → 调用时自动推断。混淆这三者是初学者最常见的错误。

### 难点 1：泛型限定为什么只有上限

`<T extends Number>` 限定后，编译器知道 T 至少是 Number，可以放心调用 Number 的方法。若允许下限 `<T super Number>`，T 可能是 Object，编译器无法保证任何可用方法，限定就失去了意义——所以**定义泛型只能用 extends 上限**；`super` 只出现在通配符 `? super` 中，服务于「往里写数据」的场景。

### 难点 2：方法引用的签名兼容判定

`Calculator::divide` 里方法体有 `throw new IllegalArgumentException(...)`，依然能引用成功——**异常不影响签名兼容性**，只看参数和返回值。运行时除零抛出的异常，由调用方决定是否捕获。

### 易错点清单

- 多个 catch 把父类型写在子类型上面——编译报错；
- 泛型数组 `new T[10]`——Java 禁止直接创建泛型数组，swap 操作的是已有数组；
- `Integer` 比较大小用 `==`——超过 -128\~127 缓存区间会踩坑，应使用 `equals` 或 `intValue`；
- 自定义异常只继承不写构造——无法携带错误信息，丢失排查线索。

## 五、示例

### 示例 1：四种形态的方法引用

同一个函数式接口 `BinaryOperation`，分别用静态方法与实例方法引用赋值，直观体现「签名兼容即可引用」。

```java
class Calculator {
    public static int add(int a, int b) { return a + b; }
    public static int subtract(int a, int b) { return a - b; }
    public int multiply(int a, int b) { return a * b; }
    public int divide(int a, int b) {
        if (b != 0) return a / b;
        throw new IllegalArgumentException("除数不能为0");
    }
}

public interface BinaryOperation {
    int operate(int a, int b);
}

public class MethodReferenceBasicExample {
    public static void main(String[] args) {
        BinaryOperation addOperation = Calculator::add;          // 静态方法引用
        System.out.println(addOperation.operate(5, 3));           // 8

        BinaryOperation subtractOperation = Calculator::subtract;
        System.out.println(subtractOperation.operate(10, 4));     // 6

        Calculator calculator = new Calculator();
        BinaryOperation multiplyOperation = calculator::multiply; // 实例方法引用
        System.out.println(multiplyOperation.operate(6, 7));      // 42

        BinaryOperation divideOperation = calculator::divide;     // 方法体内抛异常不影响引用
        System.out.println(divideOperation.operate(15, 3));       // 5
    }
}
```

要点：静态方法用「类名::」，实例方法用「对象::」；`divide` 内部会抛异常，签名仍是 `(int,int) -> int`，引用照样成立。

### 示例 2：try-catch 的执行流程

```java
public static int safeDivide(int a, int b) {
    try {
        return a / b;
    } catch (ArithmeticException e) {          // 子类在上
        System.out.println("除数不能为 0：" + e.getMessage());
        return 0;
    } catch (RuntimeException e) {              // 父类在下
        System.out.println("其他运行时异常");
        return -1;
    }
}
```

要点：`return` 写在 try 里也会先执行 catch（发生异常时），且异常点之后的语句被跳过。

### 示例 3：自定义异常携带业务语义

```java
public class AgeOutOfBoundsException extends RuntimeException {
    public AgeOutOfBoundsException() { }
    public AgeOutOfBoundsException(String message) { super(message); }
}

public static void checkAge(int age) {
    if (age < 0 || age > 150) {
        throw new AgeOutOfBoundsException("年龄不合法：" + age);
    }
}
```

### 示例 4：泛型方法 + 越界校验（数组元素交换）

```java
public class Test {
    public static void main(String[] args) {
        Integer[] arr = {1, 2, 3, 4, 5};
        String[] str = {"hello", "world", "java", "itcast", "NO.1"};
        swap(arr, 2, 3);
        swap(str, 2, 3);
    }

    public static <T> void swap(T[] array, int i, int j) {
        if (i < 0 || i >= array.length || j < 0 || j >= array.length) {
            throw new IndexOutOfBoundsException("索引越界");
        }
        T t = array[i];
        array[i] = array[j];
        array[j] = t;
    }
}
```

要点：`<T>` 写在返回值前；数组长度校验必须在交换前完成，否则 `ArrayIndexOutOfBoundsException` 由 JVM 抛出、信息不可控。

### 示例 5：泛型限定打通数值求和

```java
public class Test {
    public static void main(String[] args) {
        Integer[] arr = {1, 2, 3, 4, 5};
        Double[] arr2 = {1.1, 2.2, 3.3, 4.4, 5.5};
        System.out.println(sum(arr));    // 15.0
        System.out.println(sum(arr2));   // 16.5
    }

    public static <T extends Number> double sum(T[] array) {
        double sum = 0;
        for (T num : array) {
            sum += num.doubleValue();     // 有上限才能调用 Number 的方法
        }
        return sum;
    }
}
```

要点：没有 `<T extends Number>` 就无法调用 `doubleValue()`——泛型限定不是约束装饰，而是**方法体内可调用哪些 API 的前提**。

### 示例 6：包装类解析 + 求最大值

```java
public static void maxString() {
    String s = "12.445,2346.65,23456.34,67.34";
    String[] str = s.split(",");
    double max = Double.parseDouble(str[0]);   // 初值取第一个元素，避免全负数出错
    for (String item : str) {
        double value = Double.parseDouble(item);
        if (value > max) max = value;
    }
    System.out.println("最大值为：" + max);     // 23456.34
}
```

---
> 来源：飞书技术知识库（脱敏版） · 01 Java 语言核心 / 方法引用、异常与泛型 / 02 方法引用、异常与泛型技术要点 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/KbCRw6B7kiTF2JkK9CucnNPTnFe)
