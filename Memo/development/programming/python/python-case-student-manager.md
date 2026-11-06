---
type: memo
date: 2026-11-06
aliases:
  - "综合实战：学生管理系统"
---

# 综合实战：学生管理系统

## 概念说明

学生管理系统是一个控制台交互程序：循环打印功能菜单，用户输入编号选择操作，完成学生信息的增、删、改、查、遍历与退出。它把前面所有知识点——input 录入、if 分支、while 循环、函数封装、列表与字典容器、字符串切片与替换——串成一个完整的最小可用程序，是检验基础语法是否真正内化的试金石。

## 技术点之间的联系

本案例几乎不涉及新语法，价值在于**组合**：while True + break 构成主循环骨架，if/elif 做菜单分发，每个功能封装成函数，学生数据用「列表套字典」组织（列表存所有学生、每个学生是一个字典）。它还演示了两个进阶技巧：字符串切片 + replace 实现手机号脱敏，以及 for...else 处理「遍历完都没找到」的场景。

## 方法与实现要点

### 需求拆解

```
1. 打印提示主界面(1-6的数字)，让用户选择操作
2. 选择1：添加学生(学生编号, 学生姓名, 手机号)
3. 选择2：删除学生(根据编号删除)
4. 选择3：修改学生信息(先输入编号, 只能改姓名、手机号)
5. 选择4：查询单个学生信息(根据编号查)
6. 选择5：查询所有学生信息
7. 选择6：退出系统
```

### 结构设计

- **数据层**：全局变量 `stu_list = []`，每个学生的信息存入一个字典再 append 进列表
- **功能层**：每个菜单项对应一个函数——`show_info()` 打印菜单、`stu_add()` 添加、`del_stu_ip()` 删除、`update_stu_ip()` 修改、`query_one_ip()` 查单个、`query_all()` 查所有
- **工具函数**：`tuoMin(tel)` 负责手机号脱敏，供两个查询函数复用
- **主循环**：`while True` 打印菜单 → input 接收选择 → if/elif 分发到对应函数 → 选 6 时 break 退出

### 功能实现的关键套路

- **增**：录入信息 → 组字典 → 校验（手机号位数、学号是否已存在）→ append
- **删/改/查单个**：统一套路是「先判断列表非空 → 录入目标编号 → for 遍历逐个比对 → 命中则操作并 return/break，遍历完未命中由 for...else 提示」
- **查所有**：判断非空后 for 遍历打印，打印前对手机号脱敏

## 重点与难点分析（踩坑记录）

- **for...else 的语义反直觉**：else 在 for **正常结束（没有被 break/return 打断）** 时才执行。本案例用它表达「整个列表遍历完都没找到这个学号」，比在循环外设标志位更简洁。注意它与 if...else 毫无关系。
- **校验失败要用 return 及时退出函数**：手机号不是 11 位时直接 `return`，避免错误数据流入后续逻辑。这利用了「return 后代码不再执行」的特性，比嵌套 if 更清爽。
- **示例代码中藏着一个真实 bug**：`stu_add` 里检测到学号已存在时只 print 了提示，但没有 return 或 else 拦截，函数末尾的 `stu_list.append(...)` 依然会执行，导致重复学号照样入库。修复方式是在提示后立即 return，或把 append 放进「未找到重复」的分支里。这提醒我们：校验逻辑必须有「拦住」的动作，光提示不够。
- **变量命名要见名知意**：代码里用 `ip` 作学号的键名、用 `iphone` 拼写手机号，属于命名坏习惯的反面教材——学号应是 `stu_no`、手机号应是 `phone`/`tel`。命名混乱在多人协作和后期维护时代价很大。
- **空列表保护**：删、改、查操作前都先判断 `len(stu_list) > 0`，否则对空列表遍历虽不报错，但用户得不到任何反馈；查询场景应明确提示「还没有学生」。
- **input 返回 str 的再利用**：菜单判断 `num == "1"` 直接按字符串比较，省掉了 int 转换——当输入只用于分支判断而不参与运算时，这是更省事且更安全的写法（用户输入字母也不会崩）。

## 示例

整体骨架：菜单 + 主循环分发

```python
# 全局变量，用来存储所有学生
stu_list = []

# 定义打印功能
def show_info():
    print("===========开始==============")
    print("请选择1添加学生(学生编号, 学生姓名, 手机号).")
    print("请选择2删除学生(根据编号删除)")
    print("请选择3修改学生信息(先输入学生编号, 只能改姓名, 手机号)")
    print("请选择4查询单个学生信息(根据编号查)")
    print("请选择5查询所有学生信息")
    print("请选择6退出系统")
    print("===========结束==============")

while True:
    # 1.先打印菜单栏
    show_info()
    # 2.键盘录入你的功能
    num = input("请输入您要操作的功能：")
    if num == "1":
        stu_add()
    elif num == "2":
        del_stu_ip()
    elif num == "3":
        update_stu_ip()
    elif num == "4":
        query_one_ip()
    elif num == "5":
        query_all()
    elif num == "6":
        print("恭喜你的操作完成了！")
        break
    else:
        print("请输入1-6功能，不要随便乱输入！")
```

这段演示什么：菜单型程序的通用骨架——while True 主循环 + input 选择 + if/elif 分发 + break 退出，else 兜底非法输入。

手机号脱敏工具函数：

```python
# 手机号脱敏
def tuoMin(tel):
    iphone = tel[3:7]              # 切片取出中间4位：130 1234 5678
    new_iphone = tel.replace(iphone, "****")
    return new_iphone
```

这段演示什么：切片与 replace 的组合应用；封装成函数后，查询单个和查询所有两处复用，体现「重复逻辑就抽函数」。

添加学生（含校验）：

```python
def stu_add():
    # 1. 键盘录入学生信息
    stu_no = input("请输入你的学号：")
    stu_name = input("请输入你的姓名：")
    stu_iphone = input("请输入你的手机号：")
    if len(stu_iphone) != 11:
        print("请输入11位的手机号")
        return  # 校验失败，直接退出函数，不再执行后续
    # 2.定义字典，用来存储学生信息
    stu_info_dict = {"ip": stu_no, "name": stu_name, "iphone": stu_iphone}
    if len(stu_list) == 0:
        stu_list.append(stu_info_dict)
    else:
        # 3.判断学号是否已经存在
        for stu_dict in stu_list:
            if stu_no == stu_dict['ip']:
                print("你输入的学号已经存在了，请重新输入")
        # 注意：这里缺少return或else，重复学号仍会被append，是需要修复的bug
        stu_list.append(stu_info_dict)
```

这段演示什么：「录入 → 校验 → 组字典 → 查重 → 入库」的添加流程；同时保留了一处真实缺陷——查重后没有拦截动作，可作为调试练习。

按编号删除（for...else 实战）：

```python
def del_stu_ip():
    # 1.判断有没有学生
    if len(stu_list) > 0:
        # 2.键盘录入你要删除的学生编号
        stu_no = input("请输入你要删除的学生编号：")
        # 3.遍历比对
        for stu_dict in stu_list:
            if stu_dict['ip'] == stu_no:
                stu_list.remove(stu_dict)
                # 结束循环，与break功能一样（直接退出整个函数）
                return
        else:
            # 4.循环正常结束（没找到），提示编号不存在
            print("您输入的学生编号不存在，请重新输入！")
```

这段演示什么：for...else 的典型用法——找到就 remove + return 走人，else 只在「翻遍名单都没找到」时执行；空列表保护在最外层。

查询所有学生（脱敏输出）：

```python
def query_all():
    # 先判断列表是否为空，不为空直接遍历，是空的就提示
    if len(stu_list) > 0:
        for stu_dict in stu_list:
            new_iphone = tuoMin(stu_dict['iphone'])
            print(f"学生的学号是：{stu_dict['ip']}，姓名是：{stu_dict['name']}，电话是：{new_iphone}")
    else:
        print("你现在还没有学生，赶紧去添加一个吧")
```

这段演示什么：遍历「列表套字典」结构的标准姿势——for 取出每个字典，再用 `字典[键]` 取字段；展示前过一遍脱敏函数，是数据展示层的常见处理。