---
type: memo
date: 2026-11-06
aliases:
  - "键盘录入与运算符"
---

# 键盘录入与运算符

## 概念说明

程序要处理的数据不总是写死在代码里——input 函数让程序在运行时从键盘接收数据，实现人机交互。运算符则是对数据进行加工的工具：算术运算符做计算，赋值与复合赋值运算符做存储，比较与逻辑运算符产生 True/False 的判断原料。由于 input 拿到的永远是字符串，类型转换是连接「录入」与「运算」的必经桥梁。

## 技术点之间的联系

上一篇的 print 负责输出，本篇的 input 负责输入；input 返回 str，要做数学运算就必须先经 int()/float() 转换；比较和逻辑运算符的结果（bool 值）则直接喂给下一篇的 if 判断语句——可以说本篇一半的篇幅是在为分支结构准备条件表达式。

## 方法与实现要点

### input 键盘录入

格式：`变量名 = input("提示语")`

三个要点：

1. 参数是给用户的提示信息，在获取数据之前展示
2. input 有返回值（用户输入的内容），用等号左边的变量接收
3. **返回值一律是字符串类型**，无论用户输入的是数字还是字母

### 数据类型转换

以下函数都有返回值，需用变量接收：

| 函数 | 说明 |
|---|---|
| int(x) | 将 x 转换为整数 |
| float(x) | 将 x 转换为浮点数 |
| str(x) | 将 x 转换为字符串 |
| bool(x) | 将 x 转换为布尔值 |

转换规则与限制：

- 非数值内容的字符串（如 "张三"）不能转 int/float，会报错
- 字符串形式的小数（如 "90.6"）不能直接 int()，需先 float 再 int
- float 转 int 有精度损失（直接舍弃小数部分），慎用
- bool 转 int：True 是 1，False 是 0
- bool(x) 的判定：`0`、`-0`、`0.0`、`-0.0`、`""`、`''`、`None` 为 False；其余（包括 `" "` 空格字符串）均为 True

### 算术运算符

| 运算符 | 描述 | 示例（a=10, b=20） |
|---|---|---|
| + | 加（也可用于字符串重复拼接） | a + b → 30 |
| - | 减 | a - b → -10 |
| * | 乘（也可返回重复若干次的字符串） | a * b → 200 |
| / | 除，结果一般是 float 类型 | b / a → 2.0 |
| // | 取整除（商的整数部分） | 9 // 2 → 4 |
| % | 取余（取模），常用于奇偶校验 | 9 % 2 → 1 |
| ** | 指数（幂） | 2 ** 3 → 8 |

### 赋值与复合赋值运算符

- `=`：永远是等号右侧的结果赋给左侧的变量
- 复合赋值是为了简化开发：`c += a` 等效于 `c = c + a`，同理有 `-=`、`*=`、`/=`、`%=`、`**=`、`//=`

### 比较运算符

返回值必然是 bool 类型，一般用于判断和选择：

| 运算符 | 描述 |
|---|---|
| == | 判断两端是否相等 |
| != | 判断两端是否不相等 |
| > / < / >= / <= | 大小比较 |

### 逻辑运算符

| 运算符 | 逻辑表达式 | 描述 |
|---|---|---|
| and | x and y | 布尔「且」：x 和 y 同时为 True 才返回 True，任一为 False 则返回 False |
| or | x or y | 布尔「或」：任一为 True 则返回 True |
| not | not x | 布尔「非」：取反，not True → False |

## 重点与难点分析

- **input 的字符串陷阱**：`num1 = input(...)` 得到的是 `"1"` 而不是 `1`，`"1" == 1` 的结果是 False。要么录入后立即 int()/float() 转换，要么就按字符串做比较，两者不可混淆。
- **/ 与 // 的区别**：`/` 即使整除也返回 float（`15 / 3` 得 5.0），这会让 `num1 /= num2` 之后的变量悄悄变成 float 类型；要整数商用 `//`。
- **% 的经典用途是奇偶判断**：`num % 2 == 0` 为偶数，`num % 2 == 1` 为奇数；也用于倍数判断（`i % 7 == 0` 即 7 的倍数）。
- **" " 与 "" 不同**：空字符串转 bool 是 False，但含一个空格的字符串是非空字符串，转 bool 是 True。
- **比较运算符可以串进逻辑运算符**：`age >= 1 and age <= 3` 是判断区间的标准写法；and 优先级高于 or，复杂条件建议加括号明确意图。
- **input 常与 int 合体写**：`age = int(input("请输入年龄："))` 等价于先录入再转换两行代码。

## 示例

input 录入与格式化输出：

```python
# 1.键盘录入水果的单价
price = input("请输入您要买的水果的单价：")
# 2.键盘录入要买的水果重量
weight = input("请输入您要买的重量:")
# 3.由于input返回值是字符串，这里按字符串直接拼接输出
money = 45
print("榴莲的单价是：%s 元/斤，购买了 %s 斤，需要支付 %d 元。" % (price, weight, money))
```

这段演示什么：input 的参数是提示语、返回值需变量接收；未做类型转换时 price、weight 都是 str，只能按 %s 输出。

录入后转换类型再运算（f-string 版）：

```python
# 1.键盘录入水果的价格
price = input("请输入您要买的水果单价：")
# 2.键盘录入水果的重量
weight = input("请输入您要买的重量：")
# 3.类型转换：单价带小数点转 float，标准重量转 int
price = float(price)
weight = int(weight)
# 4.输出，大括号内可直接写运算表达式
print(f"榴莲单价 {price} 元／斤，购买了 {weight} 斤，需要支付 {price * weight} 元")
```

这段演示什么：input → 类型转换 → 运算 → f-string 输出，是交互式计算程序的固定四步套路。

算术运算符一览：

```python
num1 = 5
num2 = 3
print(num1 + num2)   # 8
print(num1 - num2)   # 2
print(num1 * num2)   # 15
print(num1 / num2)   # 1.66666...7，注意是 float
print(num1 // num2)  # 1
print(num1 % num2)   # 2
print(num1 ** num2)  # 125
```

这段演示什么：七种算术运算符的实际输出，重点观察 `/` 返回 float、`//` 取商、`%` 取余的差异。

类型转换与 bool 判定：

```python
# float 转 int 有精度损失，慎用
height = 1.80
height = int(height)
print(type(height), height)   # <class 'int'> 1

# bool 转 int：True 是 1，False 是 0
a = int(True)
b = int(False)
print(a, b)   # 1 0

# 哪些是 False：0、0.0、""、None；其余为 True（含 " " 空格字符串）
name = ""
print(bool(name))   # False
name = " "
print(bool(name))   # True
a = None
print(bool(a))      # False
```

这段演示什么：类型转换函数都有返回值需接收；bool() 的判定清单是后面写 if 条件时的重要直觉。

比较与逻辑运算符组合：

```python
num1 = 10
num2 = 20
print(num1 == num2)  # False
print(num1 != num2)  # True
print(num1 < num2)   # True

# 验证and：判断年龄大于18岁，并且身高高于185.0
age = 19
height = 186.0
result = (age > 18 and height > 185.0)
print(result)        # True，两个条件同时成立才为 True

# 验证or：任一条件成立即为 True
print(age > 18 or height > 185.0)  # True

# 验证not：取反
print(not True)      # False
```

这段演示什么：比较运算符产出 bool 值，and/or/not 把多个条件组合成复合条件——这正是下一篇 if 语句的输入。