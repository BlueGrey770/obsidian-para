---
type: memo
date: 2026-11-06
aliases:
  - "03-前后端快速开发"
---

# 03-前后端快速开发

## 目标

今天的目标为：

- 能够使用Element Plus组件改造护理项目的列表展示
- 能够使用若依框架的字典功能配合Element Plus组件展示状态数据
- 能够使用Element Plus组件改造护理项目的新增和修改的弹窗
- 能够独立完成OSS集成到项目中
- 能够独立完成护理计划的接口开发

## 护理项目页面改造

### 列表查询

#### 问题分析

护理项目中的效果图：

![pnpyb3faaotwdixzl8ccw7hundg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/pnpyb3faaotwdixzl8ccw7hundg.png)
> 图注：这张图片展示的是改造后的护理项目列表页面效果图，页面顶部有名称输入框、状态下拉选择框，以及搜索、重置按钮和新增按钮。表格区域包含序号、护理图片、护理项目名称、价格、单位、排序号、状态、创建时间和操作列。状态列清晰显示“启用”“停用”文字，而非数字；创建时间展示了完整的年月日时分秒；操作列包含修改、删除、禁用/启用按钮，列表中显示了2条护理项目数据，页面底部还有分页控制栏。

目前使用若依快速代码生成的效果图：

![n4jtbefblommxcxyr43c8x7tnsc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/n4jtbefblommxcxyr43c8x7tnsc.png)
> 图注：这张图是使用若依快速代码生成工具制作的护理项目列表查询页面效果图，页面顶部设有名称搜索框，搭配“搜索”和“重置”按钮，操作栏包含新增、修改、删除、导出功能；表格列表显示测试护理项目的相关信息，其中状态字段显示为数字“1”，创建时间仅展示到年月日，未显示时分秒，同时缺少状态筛选条件和禁用按钮，还存在非必要的导出按钮，序号列是数据库的主键id，并非规范的连续序号，底部分页显示共1条数据，每页10条。

经过我们的对比，有以下几点不同

- 搜索栏缺少**状态**条件
- 状态展示为数字，应该展示**禁用**或**启用**
- 缺少**禁用**按钮和功能，多了几个不需要的按钮
- 没有序号，多了一列编号，显示的是数据库表中的id
- 创建时间没有展示**时分秒**



#### 前端代码阅读

找到src/views/nursing/project/index.vue文件，护理项目的前端代码已经添加了详细的注释，如下：

```JavaScript
<template>
  <div class="app-container">

    <!-- 搜索表单  start -->
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="名称" prop="name">
        <el-input
          v-model="queryParams.name"
          placeholder="请输入名称"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>
    <!-- 搜索表单 end -->

    <!-- 按钮区域 start -->
    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['nursing:project:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['nursing:project:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['nursing:project:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['nursing:project:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>
    <!-- 按钮区域 end -->

    <!-- 数据展示区域 start -->
    <el-table v-loading="loading" :data="projectList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="编号" align="center" prop="id" />
      <el-table-column label="名称" align="center" prop="name" />
      <el-table-column label="排序号" align="center" prop="orderNo" />
      <el-table-column label="单位" align="center" prop="unit" />
      <el-table-column label="价格" align="center" prop="price" />
      <el-table-column label="图片" align="center" prop="image" width="100">
        <template #default="scope">
          <image-preview :src="scope.row.image" :width="50" :height="50"/>
        </template>
      </el-table-column>
      <el-table-column label="护理要求" align="center" prop="nursingRequirement" />
      <el-table-column label="状态" align="center" prop="status" />
      <el-table-column label="创建时间" align="center" prop="createTime" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['nursing:project:edit']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['nursing:project:remove']">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 数据展示区域 end -->
    

    <!-- 分页组件 start -->
    <pagination
      v-show="total>0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />
    <!-- 分页组件 end -->

    <!-- 新增或修改弹窗  start -->
    <!-- 添加或修改护理项目对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="projectRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="名称" prop="name">
          <el-input v-model="form.name" placeholder="请输入名称" />
        </el-form-item>
        <el-form-item label="排序号" prop="orderNo">
          <el-input v-model="form.orderNo" placeholder="请输入排序号" />
        </el-form-item>
        <el-form-item label="单位" prop="unit">
          <el-input v-model="form.unit" placeholder="请输入单位" />
        </el-form-item>
        <el-form-item label="价格" prop="price">
          <el-input v-model="form.price" placeholder="请输入价格" />
        </el-form-item>
        <el-form-item label="图片" prop="image">
          <image-upload v-model="form.image"/>
        </el-form-item>
        <el-form-item label="护理要求" prop="nursingRequirement">
          <el-input v-model="form.nursingRequirement" placeholder="请输入护理要求" />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
    <!-- 新增或修改弹窗  end -->
  </div>
</template>

<script setup name="Project">
// 引入后端接口
import { listProject, getProject, delProject, addProject, updateProject } from "@/api/nursing/project";
// 获取当前实例代理对象，用于访问组件的数据、方法等属性。
const { proxy } = getCurrentInstance();

// 列表数据
const projectList = ref([]);
// 是否显示弹窗
const open = ref(false);
// 加载状态
const loading = ref(true);
// 是否显示搜索栏
const showSearch = ref(true);
//记录被选中的id集合
const ids = ref([]);
//是否是单选，用于是否高亮修改按钮
const single = ref(true);
//是否是多选，用于是否高亮删除按钮
const multiple = ref(true);
//总条数
const total = ref(0);
// 标题
const title = ref("");

const data = reactive({
  // 新增或修改表单数据
  form: {},
  // 查询参数
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    name: null,
    status: null,
  },
  // 表单校验
  rules: {
    status: [
      { required: true, message: "状态不能为空", trigger: "change" }
    ],
  }
});
//toRefs将传入的数据对象的特定属性转换为响应式对象
const { queryParams, form, rules } = toRefs(data);

/** 查询护理项目列表 */
function getList() {
  loading.value = true;
  listProject(queryParams.value).then(response => {
    projectList.value = response.rows;
    total.value = response.total;
    loading.value = false;
  });
}

// 取消按钮
function cancel() {
  open.value = false;
  reset();
}

// 表单重置
function reset() {
  form.value = {
    id: null,
    name: null,
    orderNo: null,
    unit: null,
    price: null,
    image: null,
    nursingRequirement: null,
    status: null,
    createBy: null,
    updateBy: null,
    remark: null,
    createTime: null,
    updateTime: null
  };
  proxy.resetForm("projectRef");
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef");
  handleQuery();
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = "添加护理项目";
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value
  getProject(_id).then(response => {
    form.value = response.data;
    open.value = true;
    title.value = "修改护理项目";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["projectRef"].validate(valid => {
    if (valid) {
      if (form.value.id != null) {
        updateProject(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addProject(form.value).then(response => {
          proxy.$modal.msgSuccess("新增成功");
          open.value = false;
          getList();
        });
      }
    }
  });
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value;
  proxy.$modal.confirm('是否确认删除护理项目编号为"' + _ids + '"的数据项？').then(function() {
    return delProject(_ids);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("删除成功");
  }).catch(() => {});
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('nursing/project/export', {
    ...queryParams.value
  }, `project_${new Date().getTime()}.xlsx`)
}

getList();
</script>

```

#### 字段处理-序号和时间

##### 序号处理

目前的代码中，如下代码

```JavaScript

<!-- 数据展示区域 start -->
    <el-table v-loading="loading" :data="projectList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />  <!-- 多了一个复选框 -->
      <el-table-column label="编号" align="center" prop="id" />  <!-- 是序号不是编号 -->
      <!-- 其他代码略 -->
    </el-table>
<!-- 数据展示区域 end -->
```

目前在页面中展示的编号，接口中对应的字段是id（表中的主键值，并不能保证是连续的）

解决：

在Table表格组件中，提供了在表格中展示序号的方案：

参考链接：https://element-plus.org/zh-CN/component/table.html

修改之后的代码：

```JavaScript
<!-- 数据展示区域 start -->
    <el-table v-loading="loading" :data="projectList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="序号" type="index" align="center" />
      <!-- 其他代码略 -->
    </el-table>
<!-- 数据展示区域 end -->
```

最终效果：

![gsz9bbyebomtlgxejsmc1pe8nze](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gsz9bbyebomtlgxejsmc1pe8nze.png)
> 图注：图片展示了护理项目列表查询的最终效果。表格中包含序号、图片、名称三列，序号从1到6依次排列，图片为护理相关场景图，名称分别为翻身拍背、复健运动、公共空间清洁、心理咨询、洗脚、洗脸。序号列与上下文提到的“在Table表格组件中，提供了在表格中展示序号的方案”相呼应，展示了修改代码后的序号展示效果。



##### 时间展示处理

目前的代码：

```JavaScript
<!-- 数据展示区域 start -->
    <el-table v-loading="loading" :data="projectList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="创建时间" align="center" prop="createTime" width="180">
         <template #default="scope">
           <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d}') }}</span>
         </template>
      </el-table-column>
      <!-- 其他代码略 -->
    </el-table>
<!-- 数据展示区域 end -->
```

很显然，目前在页面中显示就是年月日，我们需要在后边添加参数就可以了

> parseTime方法的作用就是来格式化日期字段的，方法中有两个参数
> 
> - 第一个参数，需要格式化的日期字段
> - 第二个参数，格式化之后字符串展示效果
> 
> 如果想要展示 年月日 时分秒 则可以修改第二个参数：`{y}-{m}-{d} {h}:{i}:{s}`

修改之后的代码

```JavaScript
<!-- 数据展示区域 start -->
    <el-table v-loading="loading" :data="projectList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="创建时间" align="center" prop="createTime" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
        </template>
      </el-table-column>
      <!-- 其他代码略 -->
    </el-table>
<!-- 数据展示区域 end -->
```

最终效果：

![hienb1p5wobeqsx3e4octjqwnpb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hienb1p5wobeqsx3e4octjqwnpb.png)
> 图注：图片展示了护理项目列表中“创建时间”的最终效果。表格中显示了三个时间记录，分别为2023 - 09 - 26 15:59:08、2023 - 09 - 26 15:46:09、2023 - 09 - 26 15:21:10。这与文档中“时间展示处理”部分的内容相关，说明在修改代码后，创建时间在页面中能以年月日时分秒的形式展示，解决了创建时间仅显示年月日的问题。

#### 状态样式调整

目前的状态显示中，都是数值表示，0或者1，我们想要的效果是**启用**或**禁用**

在数据展示部分，我们可以对某一个字段使用<template>标签来重构某个字段的显示

完成后的代码如下：

```JavaScript
<!-- 数据展示区域 start -->
<el-table v-loading="loading" :data="projectList" @selection-change="handleSelectionChange">
  <el-table-column type="selection" width="55" align="center" />
  <el-table-column label="状态" align="center" prop="status" >
    <template #default="scope">
      <el-tag :type="scope.row.status === 1 ? 'success' : 'danger'">{{ scope.row.status === 1 ? '启用' : '禁用' }}</el-tag>
    </template>
  </el-table-column>
  <!-- 其他代码略... -->
</el-table>
<!-- 数据展示区域 end -->
```

> - <template #default="scope"> 这是一个Vue模板插槽的语法。`#default`表示默认插槽，`scope`是作用域参数，通过`scope`可以访问到父组件传递给插槽的任何数据
> - scope.row 代表的是这一行数据
> - :type=success | danger 表示显示不同的样式   success:成功   danger:危险

#### 新增查询条件

在目前的后端代码中已经对状态的条件进行了检索，我们只需要修改前端的代码，在搜索的表单中添加一个新的条件即可。完成后的效果如下：

![rohmbb93zo8ieyxrrgvcdrv2nwh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rohmbb93zo8ieyxrrgvcdrv2nwh.png)
> 图注：这张图片展示的是护理项目页面的列表查询区域，其中状态栏使用了el-select下拉选择组件，下拉选项包含“启用”和“停用”两个状态选项，该下拉选择组件是用于调整状态样式的实现内容，其选择后的值将绑定到查询参数变量queryParams中。页面的其他区域还设有名称输入框、新增、修改、删除按钮，以及搜索、重置功能按钮，整体为护理项目页面改造后状态样式调整完成后的效果。

这里需要使用的是下拉选择的组件`<el-select>`

参考链接：https://element-plus.org/zh-CN/component/select.html

完成后的代码：

```JavaScript
<!-- 搜索表单  start -->
<el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
  <el-form-item label="名称" prop="name">
    <el-input v-model="queryParams.name" placeholder="请输入名称" clearable @keyup.enter="handleQuery" />
  </el-form-item>
  <el-form-item label="状态" prop="status">
    <el-select v-model="queryParams.status" placeholder="请选择" clearable>
      <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
    </el-select>
  </el-form-item>
  <el-form-item>
    <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
    <el-button icon="Refresh" @click="resetQuery">重置</el-button>
  </el-form-item>
</el-form>
<!-- 搜索表单 end -->

<script setup name="Project">

//下拉选择框的选择项
const options = ref([
  {
    value: 1,
    label: "启用"
  },
  {
    value: 0,
    label: "停用"
  }
])
</script>
```

> - 后端需要的状态数据，可以参考已经提供的接口文档
> - 选择下拉选择组件之后，选择后的值需要绑定到查询的参数变量**queryParams**中

#### 禁用功能开发

在目前的操作栏目中，只有两个按钮，分别是**修改**和**删除，**我们需要参考这两个按钮，新增一个**禁用**按钮，并且需要给这个按钮绑定事件，方便提交数据到后端进行修改

> 注意：我们无需新增后端的禁用接口，可直接使用**修改**接口

- 新增按钮

```JavaScript
<!-- 数据展示区域 start -->
<el-table v-loading="loading" :data="projectList" @selection-change="handleSelectionChange">
  <el-table-column type="selection" width="55" align="center" />
  <!-- 其他代码略... -->
  <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
    <template #default="scope">
      <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)"
        v-hasPermi="['nursing:project:edit']">修改</el-button>
      <el-button link type="danger" icon="Delete" @click="handleDelete(scope.row)"
        v-hasPermi="['nursing:project:remove']">删除</el-button>
      <el-button link type="primary" :icon="scope.row.status == 0 ? 'Lock' : 'Unlock'" @click="handleEnable(scope.row)" >{{ scope.row.status == 0 ? '启用' : '禁用' }}</el-button>
    </template>
  </el-table-column>
</el-table>
<!-- 数据展示区域 end -->
```

图标组件链接：https://element-plus.org/zh-CN/component/icon.html



- 按钮中绑定方法handleEnable(scope.row)，调用后端接口

```JavaScript
/* 启用禁用按钮操作 */
function handleEnable(row) {
  // 获取当前状态
  const status = row.status;
  // 提示信息
  const msg = status === 1 ? '禁用' : '启用';

  // 构建参数
  const params = {
    id: row.id,
    status: status === 1 ? 0 : 1
  };
  
  proxy.$modal.confirm(`是否确认${msg}该护理项目的数据项？`).then(function() {
    return updateProject(params);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess(`${msg}成功`);
  }).catch(() => {});
}
```

最终效果：

![we3pbavv0ok1tvxevg1cbkxknxf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/we3pbavv0ok1tvxevg1cbkxknxf.png)
> 图注：图片展示的是护理项目页面改造中列表查询完成后的效果。页面上方有“名称”“状态”筛选框及“搜索”“重置”按钮。下方列表呈现护理项目信息，包括序号、图片、名称、价格、单位、排序号、创建人、状态、创建时间及操作栏的“修改”“删除”“禁用”按钮。弹出的“系统提示”框询问是否确认启用护理项目的数据项，有“取消”“确定”两个按钮。该图与上下文介绍的列表查询及状态样式调整等内容对应，直观呈现了改造后的页面样式。

#### 其他调整

如下图：

![k5brbtqkrobnwbxor99co10enqg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/k5brbtqkrobnwbxor99co10enqg.png)
> 图注：这张图片是护理项目列表页面的改造示例，展示了列表查询相关的调整效果。页面上方的操作区有+新增按钮，被红色箭头和红色框标注出要删除的“修改”“删除”“导出”按钮，对应文档中删除没用按钮的要求；页面下方的护理项目列表“操作”列，按钮实现了一排展示，被红色框标注，体现了调整列宽让操作按钮横向排列、固定操作列的效果，符合文档中列表查询相关调整的内容要求。

1. 删除一些没用的按钮
2. 调整列宽，让修改、删除、启用/禁用按钮在一行展示
3. 固定操作列



##### 删除无关按钮

如下图，把对应的按钮代码注释掉或删除即可

![hodpbas5qoaacxxwoijcffy8nuf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hodpbas5qoaacxxwoijcffy8nuf.png)
> 图注：这张图片是一段代码截图，对应护理项目页面改造中“删除无关按钮”的操作，展示了列表页操作按钮的相关代码内容。截图里的代码区域为红色方框标出部分，包含修改、删除、导出等按钮的相关代码，每个按钮均标注了对应的权限标识，该内容是删除这些无关按钮前的代码展示，后续只需对应注释或删除这些按钮的代码即可完成操作调整。

- 其中的导出的操作，也可以找到对应的handleExport方法，进行注释或删除



##### 操作列固定展示

如果想要按钮一排展示，我们还需要参考Element Plus官网文档的Table组件的说明

https://element-plus.org/zh-CN/component/table.html

![gjxjbvdnwoef1bxiyofcsqjqnrg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gjxjbvdnwoef1bxiyofcsqjqnrg.png)
> 图注：这是一张Element Plus Table组件相关的说明图片，用于说明固定列的使用规则与效果。图片上方标注“固定列”，并说明横向内容过多时可选择固定列，固定列需使用fixed属性，该属性接受布尔值（为true时列将被左侧固定）或字符串（left或right，分别表示固定在左侧或右侧）。图片下方展示了包含Date、Name、State、City、Address、Operations列的表格，其中最右侧的Operations操作列处于固定在页面右侧的效果，符合固定列的实现示例，对应上下文提到的固定操作列的改造需求。

代码修改如下：

```HTML
<el-table v-loading="loading" :data="projectList" @selection-change="handleSelectionChange">
      
      <!-- 其他字段省略.... -->
      
      <el-table-column label="操作" fixed="right" width="200" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)"
            v-hasPermi="['nursing:project:edit']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)"
            v-hasPermi="['nursing:project:remove']">删除</el-button>
          <el-button link type="primary" :icon="scope.row.status == 0 ? 'Lock' : 'Unlock'" @click="handleEnable(scope.row)"
            v-hasPermi="['nursing:project:edit']">{{ scope.row.status == 0 ? '启用':'禁用' }}</el-button>
        </template>
      </el-table-column>
    </el-table>
```

- fixed  列是否固定在左侧或者右侧。  left | right
- width  对应列的宽度

### 数据字典

#### 什么是数据字典

字典管理用来维护字典类型的数据，例如：状态的启用、禁用；性别的男、女；订单的已下单、支付中、已完成；通常用在如下拉框、单选按钮、复选框、树选择中的数据中，方便系统管理员维护。

主要功能包括：**字典分类管理**、**字典数据管理**

> 简单的理解：数据字典就是企业中对一些**常量**的统一管理



![obiybhpvooxyjyx8hw0cuabyn87](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/obiybhpvooxyjyx8hw0cuabyn87.png)
> 图注：这张图片是中州养老系统的字典管理页面，左侧导航栏里被红框标出了“字典管理”选项，对应页面展示了字典列表。页面的字典类型列中被红框重点标注，展示了用户性别、菜单开关、任务状态等不同字典对应的专属类型标识，整体呈现了系统中数据字典的分类管理内容，和文档里提到的数据字典管理相关内容相呼应，直观呈现了数据字典的字典类型设置情况。

每个字典类型，可能会有多个值，比如，下面是用户**性别**的字典数据

![cuqmbsn0io46spxbhmjc6t3mnif](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/cuqmbsn0io46spxbhmjc6t3mnif.png)
> 图注：这是用户性别数据字典的管理页面，上方设置了字典名称、字典标签、状态的筛选搜索栏，配有搜索、重置按钮，还提供新增、修改、删除、导出、关闭功能选项。页面主体以表格形式展示字典数据，共3条有效数据，对应男性、女性、未知三种用户性别分类，每条数据包含字典编码、字典标签、字典键值、字典排序、状态、备注、创建时间等信息，操作栏可对单条数据进行修改或删除，底部支持分页操作。

- 表关系

![klogblk7hosfixxctntcivi7n8f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/klogblk7hosfixxctntcivi7n8f.png)
> 图注：这张图片展示了两张数据字典相关的数据库表结构，左侧为sys_dict_type字典类型表，右侧为sys_dict_data字典数据表，用来说明二者的关联关系。左侧的字典类型表包含字典主键、字典名称、字典类型等字段；右侧的字典数据表包含字典编码、字典排序、字典标签等字段，其中有一个红色箭头明确指向右侧表的“dict_type:字典类型”字段，体现出右侧表的该字段和左侧表存在关联，该表关系的展示对应了文档中关于数据字典表关系的说明内容，帮助理解数据字典的表间对应逻辑。

#### 使用方式

在刚刚开发的护理项目中，状态下拉列表就可以使用数据字典来完成数据的展示，如下图

![f2f7bl4qjo48vcx8q62cdcnlnqg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/f2f7bl4qjo48vcx8q62cdcnlnqg.png)
> 图注：图片展示了护理项目页面中状态下拉列表的展示效果。列表中有“启用”和“禁用”两个选项，其中“启用”选项被红色框线突出显示。该图片与上下文紧密相关，上下文提到在护理项目中，状态下拉列表可使用数据字典来完成数据展示，此图直观呈现了状态下拉列表的样式及选项，辅助说明了使用数据字典在展示查询条件状态时的具体应用。

在展示查询条件状态的时候，有两个选项值，目前代码中是写死的内容，我们定义了一个options，如果使用字典可以先定义护理项目的状态字典，然后在组件中引用字典数据即可。下面是详细的操作步骤。



1. 新增数据字典

![rfjtbgb8qoika9xtvxzcygypn9g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rfjtbgb8qoika9xtvxzcygypn9g.png)
> 图注：图片展示的是中州养老系统中护理项目页面的字典管理界面。左侧导航栏选中“字典管理”，右侧显示“护理项目”字典管理页面，有“新增”按钮。弹出的“修改字典类型”窗口中，字典名称为“护理项目状态”，字典类型为“nursing_project_status”，状态为“启用”，备注栏为空，有“确定”和“取消”按钮。该图片与文档中“新增数据字典”内容相关，直观呈现了新增字典类型的操作界面。

1. 新增字典数据

选中字典类型，可以进入字典数据页面，对字典数据进行新增

![rclbbjh9eokgcjxxyiqc4qvln2c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rclbbjh9eokgcjxxyiqc4qvln2c.png)
> 图注：图片展示的是护理项目数据字典管理界面。界面上方有“首页”“字典管理”等导航栏。中间区域有“字典名称”“字典类型”“状态”等搜索栏，下方列表显示字典编号、字典名称、字典类型、状态、备注、创建时间等信息，其中“护理项目状态”字典类型为“nursing_project_status”。底部有分页显示共11条记录，每页10条。该图片与上下文介绍的护理项目数据字典相关，直观呈现了字典管理页面及部分字典数据。

新增字典数据，添加两条数据

- 启用 1
- 禁用 0

![gua0bkqmaoumeaxspsucvmfhntf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gua0bkqmaoumeaxspsucvmfhntf.png)
> 图注：图片展示的是护理项目中字典数据的添加界面。左侧为字典管理页面，有“+新增”按钮。右侧弹出“添加字典数据”窗口，可输入字典类型、数据标签、数据键值等信息，如字典类型为“nursing_project_status”，数据标签为“应用”，数据键值为1等。窗口底部有“确定”和“取消”按钮。该图片与文档中“新增字典数据，添加两条数据”上下文对应，直观呈现了新增字典数据的操作界面。

在护理项目的组件中引用字典并使用

```JavaScript
<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="名称" prop="name">
        <el-input v-model="queryParams.name" placeholder="请输入名称" clearable @keyup.enter="handleQuery" />
      </el-form-item>
      <el-form-item label="状态" prop="name">
        <el-select v-model="queryParams.status" placeholder="请选择" clearable>
          <el-option 
          v-for="item in nursing_project_status" 
          :key="item.value" 
          :label="item.label" 
          :value="item.value" />
        </el-select>
      </el-form-item>
     
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>
    <!-- 其他代码略 -->
  </div>
</template>

<script setup name="Project">
import { listProject, getProject, delProject, addProject, updateProject } from "@/api/nursing/project";

//获取当前组件的实例
const { proxy } = getCurrentInstance();

//引用数据字典
const { nursing_project_status } = proxy.useDict("nursing_project_status");

/* 
const options = ref([
  {
    value: 1,
    label: "启用"
  },
  {
    value: 0,
    label: "停用"
  }
]) */
</script>
```

> - 注释掉或删除options 
> - 使用proxy调用全局方法**useDict**,传入**字典类型，**返回值就是我们想要的数据
> - 在el-option标签中，循环获取字典数据
> - 使用数据字典之后，如果页面上没有生效，需要重启前端项目

### 护理项目-新增弹窗改造

目前若依框架给提供的新增或修改的弹窗效果如下：

这个很显然不太符合我们的需求

目前生成好的效果
![v4yjbboiiorpfoxkenyculvanbe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/v4yjbboiiorpfoxkenyculvanbe.png)
> 图注：这是基于若依框架的护理项目新增弹窗的原始界面，界面顶部标注有“添加护理项目”，包含名称、排序号、单位、价格、图片、护理要求等输入项，各输入项均配有提示文字，其中排序号输入框为普通文本输入框，价格输入框也为普通文本输入框，界面底部有“确定”与“取消”两个操作按钮，图片上传区域明确提示可上传大小不超过5MB的png、jpg、jpeg格式文件，该界面是文档中提到的与需求不符的原始弹窗效果，后续需要按要求将排序号和价格的文本输入框修改为Element Plus的数字输入框，同时新增状态单选字段来优化该弹窗。

期望达到的效果
![ffambqtjwonzs3xm2ymczutongc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ffambqtjwonzs3xm2ymczutongc.png)
> 图注：这是护理项目新增弹窗的实际效果，是对原有若依框架提供的弹窗效果改造后的成果，符合项目期望效果。弹窗的表单包含多个字段，其中排序号、价格字段旁带有加减按钮，为数字输入控件，状态字段设置了“启用”“禁用”两个单选框，此外还有名称、单位、图片、护理要求等输入项，底部设有确定和取消按钮。


经过对比，需要修复的内容如下：

- **排序号**和**价格**字段修改为数字输入框

  - 数字输入框  https://element-plus.org/zh-CN/component/input-number.html
- 新增一个状态字段，这个字段是一个单选框

  - 单选框  https://element-plus.org/zh-CN/component/radio.html

最终的代码，标颜色的部分是修改后的代码：

```HTML
<!-- 添加或修改护理项目对话框 -->
<el-dialog :title="title" v-model="open" width="500px" append-to-body>
  <el-form ref="projectRef" :model="form" :rules="rules" label-width="80px">
    <el-form-item label="名称" prop="name">
      <el-input v-model="form.name" placeholder="请输入名称" />
    </el-form-item>

    <el-form-item label="排序号" prop="orderNo">
      <el-input-number v-model="form.orderNo" 
      placeholder="请输入" 
      :min="1" 
      :max="20" />
    </el-form-item>
    <el-form-item label="单位" prop="unit">
      <el-input v-model="form.unit" placeholder="请输入单位" />
    </el-form-item>
    <el-form-item label="价格" prop="price">
      <el-input-number v-model="form.price" 
      placeholder="请输入" 
      :min="1" 
      :max="100" 
      :step="1" />
    </el-form-item>

    <el-form-item label="状态" prop="status">
      <el-radio-group v-model="form.status">
        <el-radio v-for="dict in nursing_project_status" 
        :key="dict.value" 
        :label="dict.value"
        >{{ dict.label}}</el-radio>
      </el-radio-group>
    </el-form-item>

    <el-form-item label="图片" prop="image">
      <image-upload v-model="form.image" />
    </el-form-item>
    <el-form-item label="护理要求" prop="nursingRequirement">
      <el-input v-model="form.nursingRequirement" placeholder="请输入护理要求" type="textarea"/>
    </el-form-item>
  </el-form>
  <template #footer>
    <div class="dialog-footer">
      <el-button type="primary" @click="submitForm">确 定</el-button>
      <el-button @click="cancel">取 消</el-button>
    </div>
  </template>
</el-dialog>
```

### 护理项目修改数据回显不全（状态）

修改护理项目的index组件，把状态改为字符串类型

```JavaScript
/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value
  getProject(_id).then(response => {
    form.value = response.data;
    form.value.status = String(form.value.status)
    open.value = true;
    title.value = "修改护理项目";
  });
}
```



## OSS集成

### 问题分析

在若依框架目前的实现中，是把图片存储到了服务器本地的目录，通过服务进行访问，这样做存储的是比较省事，但是缺点也有很多：

- **硬件与网络要求**：服务器通常需要高性能的硬件和稳定的网络环境，以保证文件传输的效率和稳定性。这可能会增加硬件和网络资源的成本和维护难度。
- **管理难度**：服务器目录需要管理员进行配置和管理，包括权限设置、备份策略等。如果管理不善或配置不当，可能会引发一些安全问题和性能问题。
- **性能瓶颈**：如果服务器处理能力不足或网络带宽不够，可能会导致性能瓶颈，影响文件上传、下载和访问的速度。
- **单点故障风险**：服务器故障可能导致所有存储在其上的文件无法访问，尽管可以通过备份和冗余措施来降低这种风险，但单点故障的风险仍然存在。

基于以上原因，企业中很多的文件都会存储到OSS中，OSS可以解决以上所有的问题，并且成本也不高，下面我们就把阿里的OSS集成到若依项目中。



目前实现的方案，对应的代码如下：

位置：zzyl-admin模块下，com.zzyl.web.controller.common.CommonController.uploadFile

```Java
/**
 * 通用上传请求（单个）
 */
@PostMapping("/upload")
public AjaxResult uploadFile(MultipartFile file) throws Exception
{
    try
    {
        // 上传文件路径
        String filePath = RuoYiConfig.getUploadPath();
        // 上传并返回新文件名称，保存到本地的目录中
        String fileName = FileUploadUtils.upload(filePath, file);
        String url = nursingrConfig.getUrl() + fileName;
        AjaxResult ajax = AjaxResult.success();
        ajax.put("url", url);
        ajax.put("fileName", fileName);
        ajax.put("newFileName", FileUtils.getName(fileName));
        ajax.put("originalFilename", file.getOriginalFilename());
        return ajax;
    }
    catch (Exception e)
    {
        return AjaxResult.error(e.getMessage());
    }
}
```



### 创建OSS模块

#### 方案说明

- 第一：把它集成到zzyl-framework模块中
- 第二：单独创建一个模块，专门来管理OSS（推荐使用）

  - 好处：可以实现模块化、解耦、复用和依赖管理，提高开发效率和代码质量

#### 集成步骤

1. 在父工程的pom文件中引入oss的依赖

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.zzyl</groupId>
    <artifactId>zzyl</artifactId>
    <version>3.8.8</version>

    <name>zzyl</name>
    <url>http://www.ruoyi.vip</url>
    <description>中州养老后台管理系统</description>
    
    <properties>
        <aliyun.sdk.oss>3.17.4</aliyun.sdk.oss>
    </properties>

    <!-- 依赖声明 -->
    <dependencyManagement>
        <dependencies>
            <!-- 其他依赖省略... -->
            <dependency>
                <groupId>com.aliyun.oss</groupId>
                <artifactId>aliyun-sdk-oss</artifactId>
                <version>${aliyun.sdk.oss}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```



1. 在父工程中创建子模块为：zzyl-oss，如下图所示

![sqiibuoq0o5ryix3jaickocgnwe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/sqiibuoq0o5ryix3jaickocgnwe.png)
> 图注：这张图片展示了创建新模块的界面，是前后端快速开发中OSS集成步骤里，在父工程下创建zzyl-oss子模块的操作窗口。界面中模块名称设为zzyl-oss，位置对应本地代码路径，开发语言选择Java，构建系统为Maven，JDK使用Project SDK 11，父模块关联为zzyl，分组ID为com.zzyl，构件ID为zzyl-oss，且未勾选创建示例代码，点击“Create”按钮即可完成该OSS管理子模块的创建，匹配了文中在父工程下创建专门管理OSS的子模块的推荐方案。

在zzyl-oss模块中引入新的依赖

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.zzyl</groupId>
        <artifactId>zzyl</artifactId>
        <version>3.8.8</version>
    </parent>

    <artifactId>zzyl-oss</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.aliyun.oss</groupId>
            <artifactId>aliyun-sdk-oss</artifactId>
        </dependency>

        <dependency>
            <groupId>com.zzyl</groupId>
            <artifactId>zzyl-common</artifactId>
        </dependency>
    </dependencies>

</project>
```



1. 创建oss配置类及客户端

![baxvbvdsqo3ccpx61dtcz5q8nxd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/baxvbvdsqo3ccpx61dtcz5q8nxd.png)
> 图注：这张图片展示了zzyl-oss子模块的src/main/java/com.zzyl.oss路径下的文件结构，其中该路径下红框标注了两个核心类，分别是AliyunOSSOperator和AliyunOSSProperties，对应文档中提及的OSS模块开发相关的配置类和交互类。该模块是按照集成步骤要求创建的独立子模块，后续可通过这两个类实现与阿里云OSS的交互，同时依托该模块实现解耦与复用。

上面的两个类，可以从课程资料中直接拷贝，或者从web课程资料中获取（目前资料中已提供）

- AliyunOSSProperties：配置类，读取aliyun.oss为前缀的属性值（桶名称，域名站点）
- AliyunOSSOperator：通过OSSClient对oss进行交互，提供了**上传**和**删除**两个方法



1. zzyl-admin模块引入并配置

先把zzyl-oss模块注册到父工程中，在zzyl父工程的pom文件添加以下内容：

```XML
<!-- 依赖声明 -->
<dependencyManagement>
    <dependencies>

        <!-- 其他依赖配置省略...-->
       
        <!-- OSS模块 -->
        <dependency>
            <groupId>com.zzyl</groupId>
            <artifactId>zzyl-oss</artifactId>
            <version>${zzyl.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

在zzyl-admin模块中引入zzyl-oss

```XML
<dependency>
    <groupId>com.zzyl</groupId>
    <artifactId>zzyl-oss</artifactId>
</dependency>
```

在application.yml文件中添加对oss的配置

```YAML
# 阿里云OSS相关配置
aliyun:
  oss:
    endpoint: https://oss-cn-beijing.aliyuncs.com
    bucketName: java110-ai
```

注意：需要将bucketName改成自己的哦



1. 修改上传的接口

找到zzyl-admin模块中的com.zzyl.web.controller.common.CommonController类，修改单个文件上传的方法

```Java
@Autowired
private AliyunOSSOperator aliyunOSSOperator;

/**
 * 通用上传请求（单个）
 */
@PostMapping("/upload")
public AjaxResult uploadFile(MultipartFile file) throws Exception
{
    try
    {
        // 上传文件路径
        String filePath = RuoYiConfig.getUploadPath();
        // 上传并返回新文件名称
//            String fileName = FileUploadUtils.upload(filePath, file);
        String url = aliyunOSSOperator.upload(file.getBytes(), file.getOriginalFilename());

//            String url = serverConfig.getUrl() + fileName;
        AjaxResult ajax = AjaxResult.success();
        ajax.put("url", url);
        ajax.put("fileName", url);
        ajax.put("newFileName", FileUtils.getName(url));
        ajax.put("originalFilename", file.getOriginalFilename());
        return ajax;
    }
    catch (Exception e)
    {
        return AjaxResult.error(e.getMessage());
    }
}
```

### 前端改造使用

由于之前的图片访问是本地的路径和服务，需要向后端发起请求才能获取图片，现在我们使用了OSS，图片可以直接通过互联网访问，无需再次访问后端服务，所以需要修改前端的图片访问逻辑

- 修改文件位置：src/components/imageUpload/index.vue
- 如果图片路径是以http开头的，则不走后台服务访问，直接访问OSS，之前的不变，如下图

```JavaScript
watch(() => props.modelValue, val => {
  if (val) {
    // 首先将值转为数组
    const list = Array.isArray(val) ? val : props.modelValue.split(",");
    // 然后将数组转为对象数组
    fileList.value = list.map(item => {
      if (typeof item === "string") {
        /* if (item.indexOf(baseUrl) === -1) {
          item = { name: baseUrl + item, url: baseUrl + item };
        } else {
          item = { name: item, url: item };
        } */
        if (item.indexOf("http") === -1) {
          item = { name: baseUrl + item, url: baseUrl + item };
        } else {
          item = { name: item, url: item };
        }
      }
      return item;
    });
  } else {
    fileList.value = [];
    return [];
  }
},{ deep: true, immediate: true });
```

测试是否能上传图片并展示

![avg4b5uw1og4lsxqwkjcsehhnbd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/avg4b5uw1og4lsxqwkjcsehhnbd.png)
> 图注：这张图片展示了与前端OSS配置相关的开发测试界面，左侧是浏览器的网络请求调试面板，请求名称为upload，响应内容中包含以https开头的图片URL，对应使用阿里云OSS的网络访问资源，验证了图片直接从OSS访问的逻辑；右侧是“添加护理项目”的表单界面，在“图片”区域的示例缩略图旁标注了红色箭头，该区域用于上传展示图片，和上下文修改前端图片访问逻辑的内容相呼应，可用于测试图片上传与展示的效果。

## 护理计划功能开发

到目前为止，我们已经完成了一个表的增删改查，并且也做了相应的优化，接下来，我们开始做一个跟护理项目相关的模块：护理计划

### 需求说明

我们打开原型来看：

- 列表查询

![txdsbjgogopo4yx2m9acxauqnih](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/txdsbjgogopo4yx2m9acxauqnih.png)
> 图注：这张图是护理计划功能的列表查询页面，页面设有名称输入框、状态选择下拉框，搭配绿色的“搜索”按钮与“重置”按钮，还有“新增”按钮用于添加新护理计划。列表展示了护理计划的序号、名称、状态、创建时间，两条示例计划分别为“3号护理计划（禁用）”和“2号护理计划（启用）”，每条计划的操作栏对应有修改、删除、查看、启用/禁用功能按钮，页面底部显示总条数、分页设置及页码切换控件，该页面是护理计划功能开发里原型展示的列表查询部分，用于实现护理计划的查询及基础操作。

在这个页面中，同时也有一些按钮：新增、修改、删除、查看、禁用

- 新增或修改的弹窗

![kgeab5omqo6hhxx49yxcbd0qndd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kgeab5omqo6hhxx49yxcbd0qndd.png)
> 图注：这是护理计划功能开发中新增或修改护理计划的弹窗界面。界面带有必填项标识，已填充“护理计划名称”为“3号护理计划”，“状态”选中“禁用”，“排序”为1；下方护理项目列表展示了“助餐”“整理床铺”两个已选项目，可编辑各项目的期望服务时间、执行周期与执行频次，界面右下角设有“确定”与“取消”按钮，用于确认或放弃操作，对应开发中新增/修改护理计划的需求内容。

- 查看详情

![he60bbdftodhtrx1g7gcnxdhnge](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/he60bbdftodhtrx1g7gcnxdhnge.png)
> 图注：这是护理计划功能开发环节中，新增护理计划的弹窗界面，属于新增或修改护理计划的操作界面。界面顶部明确标注“新增护理计划”，带有必填项标识的输入区域，包括已填入内容的“护理计划名称”输入框、“排序”输入框，以及用于选择“启用”或“禁用”状态的选项，下方还有护理项目的配置区域，可添加如助餐、整理床铺等护理项目，每个项目可设置期望服务时间、执行周期和执行频次，是为老人选配护理服务的相关配置界面。



### 表关系说明

护理计划与护理项目的关系为**多对多，**需要有一张中间表nursing_project_plan

- 一个护理项目可以被多个护理计划引用
- 一个护理计划中也可以引用多个护理项目
- 具体的表关系，如下图：

![mdgub4xz3ogxeexvraycwv3inaf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mdgub4xz3ogxeexvraycwv3inaf.png)
> 图注：图片展示了护理计划相关表的表关系。左侧为nursing_plan表，包含编号、排序号、名称、状态、创建时间等字段；中间是nursing_project_plan表，记录计划和项目的关系，有计划ID、项目ID、执行时间等字段；右侧为nursing_project表，有编号、名称、排序号、单位、价格等字段。箭头连接各表，表明它们之间的多对多关系，如一个护理项目可被多个护理计划引用，一个护理计划中也可引用多个护理项目。此图与文档中表关系说明部分对应，直观呈现了护理计划表间关系。

<callout emoji="✍️">
需求
服务管理：护理项目、护理计划、护理等级
后期给老人选配使用的，老人办理入住，要根据老人的健康情况、年龄推荐配置（护理等级）
不同的护理等级所关联的护理计划是不一样的，不同的护理计划，包含的护理项目是不同的，因此价格也是不同的
</callout>

### 接口说明

详细的接口文档，请查看：<cite doc-id="Dn9wwiXBvik60GkY3eFcSRQOnuc" file-type="wiki" title="护理模块-接口文档" type="doc"></cite>



### 功能实现

#### 准备工作

在前面的课程中，我们使用若依框架完成了护理计划和护理等级相关的代码生成，但是生成的代码并不能完全符合我们的业务需求，所以我们需要在生成代码的基础上进行**改造**

<callout emoji="📌">
后期的开发套路也都差不多，都是对生成之后的代码进行**改造**
</callout>

改造分为两部分，分别是前端代码和后端代码。在护理计划这个模块，**大家重点开发改造后端的代码**

##### 前端代码准备

前端代码可以使用当天资料中提供好的代码：

在这个文件夹中，分别包含了四个文件，大家需要拷贝到前端的对应目录下：

![nhf0b3ooookfhzxpacqcg5crnbr](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nhf0b3ooookfhzxpacqcg5crnbr.png)
> 图注：图片展示了护理计划功能开发中前端代码准备时需拷贝的四个文件。分别是index.scss、index.vue、plan.js和variables.module.scss，其中index.scss和index.vue以紫色图标显示，plan.js以绿色图标显示，variables.module.scss以粉色图标显示。该图片与上下文紧密相关，上下文提到前端代码可使用当天资料中提供好的代码，需将这四个文件拷贝到前端对应目录下，替换原有文件，且特别指出在plan.js文件中要保证后端接口路径一致。

1. 拷贝index.scss和index.vue到之前护理计划所对应的目录下，替换掉之前的

![flpabln5aobasvx35avchop0ndb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/flpabln5aobasvx35avchop0ndb.png)
> 图注：图片展示了护理计划模块下的文件结构。其中，index.scss和index.vue两个文件被红色框线突出显示，分别位于views和plan文件夹下。这与文档中“前端代码准备”部分内容相关，文档提到需将index.scss和index.vue拷贝到之前护理计划所对应的目录下，替换掉之前的文件，此图直观呈现了这两个文件在文件结构中的位置，帮助开发者明确操作对象。

1. 拷贝plan.js到api/nursing目录下并替换掉之前的（可根据自己的实际情况选择目录进行拷贝）

![kmlhbxan3o4mawxxuhsciucenkc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kmlhbxan3o4mawxxuhsciucenkc.png)
> 图注：图片展示了前端代码文件夹结构中的api目录下的文件情况。其中，plan.js文件被红色框线突出显示。根据上下文，前端代码可使用当天资料中提供的代码，需将index.scss和index.vue等文件拷贝到对应目录下，plan.js文件需拷贝到api/nursing目录下并替换原有文件，且特别注意plan.js文件中定义的后端接口路径需与后端一致。

**特别注意：在plan.js文件中，定义了后端接口的路径，要保证与后端一致（如与自己的后端不一致，需要进行修正）**

1. 拷贝variables.module.scss到src/assets/styles目录下，替换原有的文件

![d06qbyzciouv6fxteoccgktvnnc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/d06qbyzciouv6fxteoccgktvnnc.png)
> 图注：图片展示了前端代码文件夹结构中的assets和styles目录内容。assets目录下有401_images、404_images、icons、images、logo等子目录；styles目录下有btn.scss、element-ui.scss、index.scss、mixin.scss、ruoyi.scss、sidebar.scss、transition.scss等文件，其中variables.module.scss文件被高亮显示。该图片与文档中前端代码准备部分内容相关，用于说明在拷贝index.scss和index.vue等文件时，需将variables.module.scss文件替换原有文件。

1. 在@/api/nursing/project.js中新增一个接口，查询所有的护理项目（稍后会开发这个接口）

```JavaScript
//查询所有护理项目
export function getProjectAll() {
  return request({
    url: '/nursing/project/all',
    method: 'get'
  })
}
```

1. 修复导入的路径和api请求路径

![vvlvbqfmwo0zadxbkuhcah8mnmf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vvlvbqfmwo0zadxbkuhcah8mnmf.png)
> 图注：图片展示了护理计划功能开发中前端代码准备阶段的代码界面。代码中红框突出显示了“import { getNursingPlan } from '@/api/nursing/plan'”和“import { getProjectCall } from '@/api/nursing/project'”两行代码，分别用于获取护理计划和护理项目数据。该图片与上下文紧密相关，上下文提到需在前端代码中拷贝相关文件并修复导入路径和api请求路径，此图直观呈现了代码中需修改的导入路径部分，辅助理解前端代码准备的具体操作。

![d2uvbjyteopv9bxyobnciwwfnxf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/d2uvbjyteopv9bxyobnciwwfnxf.png)
> 图注：图片展示了护理计划功能开发中前端代码准备时，对plan.js文件的修改情况。文件中定义了查询护理计划列表和查询护理计划详情的接口，其中请求路径中的“serve”被修改为“nursing”。文件还包含对请求参数的处理，如传递query参数等。该图片与上下文紧密相关，上下文提到在plan.js文件中定义了后端接口路径，需保证与后端一致，此图直观呈现了修改后的代码内容。

集成之后，报错属于正常现象，因为还没有提供对应的后端接口

![ajfmbfay6o1ykcxkfbocobcpnif](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ajfmbfay6o1ykcxkfbocobcpnif.png)
> 图注：这是中州养老系统的护理计划页面，顶部有一条红色错误提示，标注“请求参数类型不匹配，参数要求类型为：'java.lang.Long'，但输入值为：'test'”，并有红色箭头指向该提示。页面设有“护理计划”标签，下方有名称和状态的筛选栏，包含输入框、状态选择下拉框，以及“搜索”“重置”按钮，还设有“+新增”按钮；列表区域展示了一条测试护理计划的信息，包含序号、名称、状态、创建时间及修改、删除、查看、启用的操作选项，底部还有分页控制区域，整体是完成护理计划功能开发前的前端页面状态，错误提示源于前端接口调用时的参数类型不匹配问题，与文档中提到的需保证前端接口路径和参数与后端一致的要求相呼应。



##### 后端代码准备

在这个模块需要操作两张表，分别是：

- **护理计划表（nursing_plan）**  该表的代码已生成（前一天已完成，修改若依代码生成模板后重新生成并替换掉原始的后端代码，资料中也提供了对应的代码）
- **护理项目计划关系表(nursing_project_plan)**  该表的代码需要借助若依生成**部分文件（实体类、mapper.java、mapper.xml）**





1. 打开后台管理系统，找到**代码生成**的菜单，导入**nursing_project_plan**表，然后直接预览，如下图

![qo6obdnccojjycxw95ecmiiinrr](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qo6obdnccojjycxw95ecmiiinrr.png)
> 图注：这张图片展示了系统代码生成模块的表信息列表界面，表格内有序号、表名称、表描述、实体等栏目，分别列出了护理项目表、护理等级表、护理计划表以及护理计划和项目关联表，其中红色箭头指向第4行“护理计划和项目关联表”对应的操作栏中的预览图标。该图片对应上下文里操作步骤的第一步展示内容，用于指导使用者打开后台管理系统的代码生成菜单找到目标关联表。

1. 拷贝内容如下图所示（实体类和mapper相关）

![fv9fbaklgonjyux0afbcadtsnrb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/fv9fbaklgonjyux0afbcadtsnrb.png)
> 图注：图片展示的是护理项目计划关系表（nursing_project_plan）代码生成的代码预览界面。界面上方有多个选项卡，其中“domain.java”“mapper.java”“mapper.xml”被红色框线突出显示。下方代码内容为NursingProjectPlan类，包含id、planId、projectId等属性及get和set方法。该图片与文档中“后端代码准备”部分内容相关，用于指导开发者在打开后台管理系统找到“代码生成”菜单导入该表后，直接预览并获取代码内容，再将其拷贝到项目中进行替换操作。

拷贝到项目中，如下：

![jl3qbc7pronnjcxgjxccrfa8ndg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jl3qbc7pronnjcxgjxccrfa8ndg.png)
> 图注：这张图片展示了项目代码的目录结构，呈现zzyl-nursing-platform项目的src/main/java目录下的代码包内容，重点标识出与护理项目计划关系表相关的文件，包括domain包中的NursingProjectPlan实体类、mapper包中的NursingProjectPlanMapper接口，以及resources/mapper/nursing目录下的NursingProjectPlanMapper.xml映射文件。这些被红色框标注的内容，正是前后端快速开发护理计划功能时，需要借助若依代码生成工具生成并拷贝到项目中的核心文件，对应文档中准备工作里的第二步操作内容。

替换完前后端代码后，一定要重启后端测试护理计划页面是否能正常工作，如果有问题，先修改正确后再继续后面的内容哦。

#### 创建护理计划数据字典  

在护理计划列表中，有一个状态选项，是从数据字典（nursing_plan_status）获取数据的，效果如下：

![geavb7lltocoyoxqdrnc8yuin5f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/geavb7lltocoyoxqdrnc8yuin5f.png)
> 图注：图片展示了护理计划列表界面。在“状态”下拉菜单中，有“禁用”和“启用”两个选项，其中“启用”选项被红色框突出显示。列表中显示了两条护理计划记录，序号分别为1和2，名称分别是“1号护理计划”和“2号护理计划”，状态栏分别显示“启用”和“禁用”。该图片与上文提到的护理计划列表中状态选项是从数据字典（nursing_plan_status）获取数据的内容相关，直观呈现了状态选项的展示效果。

在**系统管理/字典管理**菜单中定义新的数据字典，详细定义如下：

![ahtmbaiyuojc40xyqtpcmje4nrf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ahtmbaiyuojc40xyqtpcmje4nrf.png)
> 图注：这张图片是系统字典管理页面的界面截图，用于展示护理计划功能开发中数据字典的相关配置。页面上方区域展示了当前系统中已有的字典类型，其中红线下标注的“nursing_plan_status”，就是文档中提到的护理计划状态对应的数据字典。页面下方区域是该数据字典的具体内容列表，包含“启用”“禁用”等字典值及对应的排序、状态等信息。整个页面支持新增、修改、删除字典及字典项的操作，还具备搜索、重置功能，可实现数据字典的管理操作，契合文档中创建护理计划数据字典的功能实现场景。

#### 查询所有护理项目

在新增计划对话框中，需要选择关联护理项目的列表，如下图

![phflbmauwoci9axtzw7c5vronmf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/phflbmauwoci9axtzw7c5vronmf.png)
> 图注：这张图片是新增护理计划的界面截图，对应文档中查询所有护理项目的相关内容。界面上方为护理计划名称、状态、排序等基础信息项，其中“状态”选中“启用”选项。中间护理项目选择区域，点击“请选择”后展开下拉列表，列表中用红色方框标注，包含“喂饭”“洗衣服”“测试护理项目”三个护理项目选项，用于在新增护理计划时关联选择所需的护理项目。

这个需要在护理项目的模块中定义一个查询的接口来查询所有的护理项目进行展示（需要结合<cite doc-id="Dn9wwiXBvik60GkY3eFcSRQOnuc" file-type="wiki" title="护理模块-接口文档" type="doc"></cite>中的定义）



1. 控制层

在NursingProjectController中定义新的方法，如下：

```Java
/**
 * 查询护理项目列表
 */
@GetMapping("/all")
@ApiOperation(value = "查询所有护理项目")
public AjaxResult listAll()
{
    List<NursingProjectVo> list = nursingProjectService.selectAll();
    return success(list);
}
```

NursingProject**Vo**

<callout emoji="📌">
**vo**即“Value Object”的缩写，通常用于封装数据，便于数据的传输和访问。
使用“**vo**”作为方法返回值的命名后缀，主要目的是明确该方法返回一个值对象，而不是一个实体（Entity）或者数据访问对象（DAO）等。这样做的好处包括：
1. 提高代码的可读性：其他开发者在阅读代码时，可以快速理解该方法返回一个值对象，而不是其他类型的对象。
2. 明确对象的用途：值对象通常用于封装数据，以便在应用程序的不同部分之间传输。通过命名约定，可以清晰地表明这些对象的用途。
3. 促进代码的一致性：在团队开发中，遵循共同的命名约定有助于保持代码的一致性，使得代码更易于维护和扩展。
4. 一般用作前端返回结果的封装
</callout>



**依据接口文档中定义的返回值**，该Vo具体的内容如下：

```Java
package com.zzyl.nursing.vo;

import lombok.Data;

@Data
public class NursingProjectVo {
    /**
     * 项目名称
     */
    private String label;

    /**
     * 项目ID
     */
    private String value;
}
```



1. 业务层

在INursingProjectService定义新的方法，如下：

```Java
/**
 * 查询所有护理项目
 * @return
 */
List<NursingProjectVo> selectAll();
```

对应的实现方法：

```Java
/**
 * 查询所有护理项目
 *
 * @return
 */
@Override
public List<NursingProjectVo> selectAll() {
    return nursingProjectMapper.selectAll();
}
```



1. 持久层

在NursingProjectMapper中定义新的方法，注意sql的返回值，要与接口文档中保持一致，如下：

```Java
@Select("select id value,name label from nursing_project where status = 1 ")
List<NursingProjectVo> selectAll();
```



1. 测试

重启服务，点击新增护理计划，检查是否能查看到所有的护理项目列表



#### 新增护理计划

1. 实现思路

接收到前端的参数包含了两部分内容，分别是：

- 护理计划相关的数据
- 护理计划绑定的护理项目列表（需要保存到nursing_project_plan中间表中）
- 由于涉及到了多张表操作，需要让多次数据库操作处于同一个事务下（保证数据一致性）
- 整体的处理逻辑如下：

<whiteboard token="AdTtwRIBXhSPv1byfzGcEgeFn6d"></whiteboard>



1. 控制层

在NursingPlanController中修改add方法，按照接口定义的参数 来创建对应的Dto参数，如下：

```Java
/**
 * 新增护理计划
 */
@PreAuthorize("@ss.hasPermi('nursing:nursingPlan:add')")
@Log(title = "护理计划", businessType = BusinessType.INSERT)
@PostMapping
public AjaxResult add(@RequestBody NursingPlanDto dto)
{
    return toAjax(nursingPlanService.insertNursingPlan(dto));
}
```

NursingPlan**Dto**

<callout emoji="📌">
DTO（Data Transfer Object）用于封装数据，以便在不同的软件应用程序之间传输数据，是一种开发规范
一般是接收前端参数
</callout>

根据接口文档中参数的定义，类的内容如下：

```Java
package com.zzyl.nursing.dto;

import com.zzyl.nursing.domain.NursingProjectPlan;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.util.List;

@Data
public class NursingPlanDto {


    private Long id;

    /**
     * 排序号
     */
    @ApiModelProperty(value = "排序号")
    private Integer sortNo;

    /**
     * 计划名称
     */
    @ApiModelProperty(value = "计划名称")
    private String planName;

    /**
     * 状态（0：禁用，1：启用）
     */
    @ApiModelProperty(value = "状态（0：禁用，1：启用）")
    private Integer status;

    /**
     * 护理计划关联项目列表
     */
    List<NursingProjectPlan> projectPlans;
}
```



1. 业务层

修改`INursingPlanService`中的`insertNursingPlan`方法，修改其参数，接收`NursingPlanDto`，详细如下：

```Java
/**
 * 新增护理计划
 * 
 * @param dto 护理计划
 * @return 结果
 */
public int insertNursingPlan(NursingPlanDto dto);
```

修改对应的实现类：

```Java
@Autowired
private NursingProjectPlanMapper nursingProjectPlanMapper;

/**
 * 新增护理计划
 * @param dto 护理计划
 * @return 结果
 */
@Transactional
@Override
public int insertNursingPlan(NursingPlanDto dto)
{
    // 保存护理计划
    // 属性拷贝
    NursingPlan nursingPlan = new NursingPlan();
    BeanUtils.copyProperties(dto, nursingPlan);
    nursingPlan.setCreateTime(DateUtils.getNowDate());
    nursingPlanMapper.insertNursingPlan(nursingPlan);

    // 批量保存护理项目计划关系
    int count = nursingProjectPlanMapper.batchInsert(dto.getProjectPlans(), nursingPlan.getId());
    return count == 0 ? 0 : 1;
}
```

<callout emoji="📌">
**代码说明**
1. `BeanUtils.`*`copyProperties`*`(dto, nursingPlan);`
含义是使用`BeanUtils`工具类将`dto`对象中的属性值拷贝到nursingPlan对象中，要求就是属性名和类型必须相同
1. 在当前方法上添加了一个`@Transactional`，保存方法内的多个表操作具备事务的特性，这个注解也可以定义在类上，表示当前类中的所有方法，都具备事务特性
</callout>



1. 持久层

我们需要在NursingProjectPlanMapper中提供一个批量插入的方法，代码如下：

```Java
int batchInsert(@Param("list") List<NursingProjectPlan> projectPlans, @Param("planId") Long planId);
```

- `@Param` 主要用于在方法参数中使用**多个**参数时，**明确参数名称**，以便在[SQL映射文件](https://www.baidu.com/s?wd=SQL%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6&rsv_idx=2&tn=62095104_26_oem_dg&usm=2&ie=utf-8&rsv_pq=a3fcde4d008c032c&oq=mybatis%E7%9A%84%40param%E6%B3%A8%E8%A7%A3&rsv_t=83a2iiI4IaABo8Bz%2FtjC6GEnN%2B3GnuNCCGEzNo3EYB86s6AunLizwt7BOZkCJXUcKk%2Bsf%2Ba6NDvk&sa=re_dqa_generate)中通过`#{参数名}`的方式来引用这些参数。

对应的xml映射文件

```XML
<insert id="batchInsert">
    insert into nursing_project_plan (plan_id, project_id, execute_time, execute_cycle, execute_frequency,
                                      create_time)
    values
    <foreach collection="list" item="item" separator=",">
        (#{planId}, #{item.projectId}, #{item.executeTime}, #{item.executeCycle}, #{item.executeFrequency},
        sysdate())
    </foreach>
</insert>
```



#### 根据id查询护理计划

1. 思路说明

在修改护理计划或者是查看护理计划的时候，都需要查询护理计划的详细数据，并且这里面需要包含两部分内容：护理计划和护理计划关联的护理项目

<whiteboard token="WGVewn7NQhPg0YbFPSecfZGAnQg"></whiteboard>



1. 定义返回结果类：

NursingPlanVo 

```Java
package com.zzyl.nursing.vo;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.util.List;

@Data
@ApiModel(description = "护理计划")
public class NursingPlanVo {

    /**
     * 护理计划id
     */
    private Long id;

    /**
     * 排序号
     */
    @ApiModelProperty(value = "护理计划排序号")
    private Integer sortNo;

    @ApiModelProperty(value = "护理计划名称")
    private String planName;

    @ApiModelProperty(value = "状态（0：禁用，1：启用）")
    private Integer status;

    @ApiModelProperty(value = "护理计划项目计划列表")
    List<NursingProjectPlanVo> projectPlans;

}
```

NursingProjectPlanVo 这个类就是把projectId改为了字符串类型，方便前端回显

```Java
package com.zzyl.nursing.vo;

import com.zzyl.common.annotation.Excel;
import lombok.Data;

@Data
public class NursingProjectPlanVo {

    /** 项目id */
    private String projectId;

    /** $column.columnComment */
    private Long id;

    /** 计划id */
    @Excel(name = "计划id")
    private Long planId;

    /** 计划执行时间 */
    @Excel(name = "计划执行时间")
    private String executeTime;

    /** 执行周期 0 天 1 周 2月 */
    @Excel(name = "执行周期 0 天 1 周 2月")
    private String executeCycle;

    /** 执行频次 */
    @Excel(name = "执行频次")
    private Long executeFrequency;
}
```

1. 控制层

将返回值类型改为NursingPlanVo：

```SCSS
/**
 * 获取护理计划详细信息
 */
@PreAuthorize("@ss.hasPermi('nursing:plan:query')")
@GetMapping(value = "/{id}")
@ApiOperation("获取护理计划详细信息")
public R<NursingPlanVo> getInfo(@ApiParam(value = "护理计划ID", required = true)
        @PathVariable("id") Long id)
{
    return R.ok(nursingPlanService.selectNursingPlanById(id));
}
```

1. 业务层

修改INursingPlanService中的selectNursingPlanById，定义符合接口规范的返回值

```Java
/**
 * 查询护理计划
 * 
 * @param id 护理计划主键
 * @return 护理计划
 */
public NursingPlanVo selectNursingPlanById(Long id);
```



修改其实现方法返回值，以及详细的实现逻辑如下：

```Java
/**
 * 查询护理计划
 * 
 * @param id 护理计划主键
 * @return 护理计划
 */
@Override
public NursingPlanVo selectNursingPlanById(Long id)
{
    // 1.根据id查询护理计划
    NursingPlan nursingPlan = nursingPlanMapper.selectNursingPlanById(id);
    NursingPlanVo nursingPlanVo = new NursingPlanVo();
    BeanUtils.copyProperties(nursingPlan, nursingPlanVo);

    // 2.根据护理计划id查询关联的所有护理项目
    List<NursingProjectPlanVo> list = nursingProjectPlanMapper.selectByPlanId(id);

    nursingPlanVo.setProjectPlans(list);
    return nursingPlanVo;
}
```



1. 持久层

在NursingProjectPlanMapper定义按照护理计划ID查询列表的方法，如下

```Java
/**
 * 根据计划id查询项目计划
 * @param id
 * @return
 */
List<NursingProjectPlanVo> selectByPlanId(Long id);
```

对应的映射文件：

```XML
<select id="selectByPlanId" resultType="com.zzyl.nursing.vo.NursingProjectPlanVo">
    select id, plan_id, project_id, execute_time, execute_cycle, execute_frequency
        from nursing_project_plan
        where plan_id = #{id}
</select>
```



#### 修改护理计划

**实现思路**

修改护理计划的时候，也是操作两张表，基本的思路如下图：

<whiteboard token="L8ymwWUYthPlWhb19XfcQfm2nBg"></whiteboard>

1. 控制层

修改控制层的edit方法的参数，接收NursingPlanDto类

```Java
/**
 * 修改护理计划
 */
@PreAuthorize("@ss.hasPermi('nursing:plan:edit')")
@Log(title = "护理计划", businessType = BusinessType.UPDATE)
@PutMapping
@ApiOperation("修改护理计划")
public AjaxResult edit(@ApiParam(value = "护理计划实体", required = true) @RequestBody NursingPlanDto dto)
{
    return toAjax(nursingPlanService.updateNursingPlan(dto));
}
```

1. 业务层

基于提供的思路流程图，修改INursingPlanService接口中的修改的方法的参数，代码如下：

```Java
/**
 * 修改护理计划
 * 
 * @param dto 护理计划
 * @return 结果
 */
public int updateNursingPlan(NursingPlanDto dto);
```

实现类中的方法：

```Java
/**
 * 修改护理计划
 * 
 * @param dto 护理计划
 * @return 结果
 */
@Override
public int updateNursingPlan(NursingPlanDto dto)
{
    try {
        // 属性拷贝
        NursingPlan nursingPlan = new NursingPlan();
        BeanUtils.copyProperties(dto, nursingPlan);

        // 判断dto中的项目列表为空，如果不为空，则先删除护理计划与护理项目的关系，然后重新批量添加
        if (dto.getProjectPlans() != null || !dto.getProjectPlans().isEmpty()) {
            // 删除护理计划对应的护理项目列表
            nursingProjectPlanMapper.deleteByPlanId(nursingPlan.getId());
            // 补全护理计划id
            List<NursingProjectPlan> projectPlans = dto.getProjectPlans();
            projectPlans.forEach(nursingProjectPlan -> nursingProjectPlan.setPlanId(nursingPlanDto.getId()));
            // 批量保存护理计划对应的护理项目列表
            nursingProjectPlanMapper.batchInsert(projectPlans, nursingPlan.getId());
        }

        // 不管项目列表是否为空，都要修改护理计划
        return nursingPlanMapper.updateNursingPlan(nursingPlan);
    } catch (BeansException e) {
        throw new RuntimeException(e);
    }
}
```

1. 持久层

在NursingProjectPlanMapper中定义根据护理计划ID删除的方法和SQL

```Java
@Delete("delete from nursing_project_plan where plan_id = #{planId}")
void deleteByPlanId(Long planId);
```

1. 其他补充，由于新增、修改、删除都是操作两张表，都需要事务的控制，可以在NursingPlanServiceImpl上添加事务的注解`@Transactional`

```Java
@Transactional
@Service
public class NursingPlanServiceImpl implements INursingPlanService 
{
//其他代码省略...
}
```

#### 删除护理计划

**实现思路**

删除护理计划的时候，也要操作两张表，基本的思路是如下图：

<whiteboard token="ME13wo0XDhkOMNbjV0Ecmi9Sn5g"></whiteboard>

1. 控制层

在NursingPlanController修改删除的方法，如下：

```Java
/**
 * 删除护理计划
 */
@PreAuthorize("@ss.hasPermi('nursing:plan:remove')")
@Log(title = "护理计划", businessType = BusinessType.DELETE)
@DeleteMapping("/{id}")
@ApiOperation("删除护理计划")
public AjaxResult remove(@ApiParam(value = "要删除的数据id") @PathVariable Long id)
{
    return toAjax(nursingPlanService.deleteNursingPlanById(id));
}
```

1. 业务层，只需要修改deleteNursingPlanById即可，需要在两张表中都删除

```Java
/**
 * 删除护理计划信息
 * 
 * @param id 护理计划主键
 * @return 结果
 */
@Override
public int deleteNursingPlanById(Long id)
{
    // 删除护理计划与护理项目的关系
    nursingProjectPlanMapper.deleteByPlanId(id);
    // 删除护理计划
    return removeById(id) ? 1 : 0;
}
```

## 微实战-护理等级

### 需求说明

下图是实现后的护理等级查询列表

![dph1btxzvorht0xpz5pcguvfnmd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dph1btxzvorht0xpz5pcguvfnmd.png)
> 图注：这是护理等级查询列表的实现界面，上方设有等级名称输入框、状态下拉选择框，以及搜索、重置按钮，还有新增按钮；列表包含序号、等级名称、护理计划、护理费用、状态、创建时间和操作列，共2条数据，分别为1号护理计划、测试护理等级，状态均为启用，操作列对应修改、删除、禁用按钮；列表底部标注共2条数据，支持设置10条/页，当前为第1页，可执行翻页操作，界面符合前后端开发中护理等级模块的查询功能展示需求。

在这个列表中有一些按钮：新增、修改、删除、启用|禁用



下图是新增和修改的弹窗，其中的护理计划是一个下拉列表

![ruqybr98hojfttxxtegcbufgnde](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ruqybr98hojfttxxtegcbufgnde.png)
> 图注：图片展示的是护理等级模块中“添加护理等级”弹窗界面。弹窗标题为“添加护理等级”，包含“等级名称”“护理计划”“护理费用”“状态”“等级说明”等输入框或选择项，其中“护理计划”下拉列表中显示“3号护理计划”“2号护理计划”选项，当前选中“3号护理计划”。“状态”处有“启用”“禁用”两个单选按钮，选中“启用”。“等级说明”输入框内无内容。底部有“确定”“取消”两个按钮。该图片与文档中要求利用开发经验开发护理等级模块的内容相关，展示了新增护理等级时的输入界面。

下拉框组件：https://element-plus.org/zh-CN/component/select.html

### 要求

请利用自己开发护理项目和护理计划的经验来开发护理等级模块。

重点提示：

- 首要任务是读懂需求
- 根本是熟悉表中每一个字段含义
- 一定要结合接口文档



### 提示

服务管理共有4张表

<whiteboard token="SDzKw8xw4hnUSubfZ8gc1m4knde"></whiteboard>



## 写完作业后，准备明天的初始代码

参考

<cite doc-id="EUXKwZci8iyMAFkMkYWcYPf9nEf" file-type="wiki" title="初始代码-环境准备" type="doc"></cite>

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第一章-若依+AI快速开发新项目 / 03-前后端快速开发 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/TzGMwl98ricc4Zk2AtccdvtBnrf)
