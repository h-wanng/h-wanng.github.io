+++
author = "H. Wang"
title = "Vue Issues"
date = "2022-09-05"
description = "Vue的一些问题记录"
tags = [
    "Vue"
]
categories = [
    "Vue"
]

image = ""
+++

## Ruoyi 字典取值并回显到页面
```html
<el-table-column label="职级" align="center" key="job_grade" prop="job_grade" :show-overflow-tooltip="true" >
    <template slot-scope="scope">
        <dict-tag :options="dict.type.job_grade_type" :value="scope.row.job_grade" />
    </template>
</el-table-column>
```
通过 `slot-scope` `scope.row.job_grade` 获取当前column的prop，在字典中获取对应数据（label）

## Ruoyi vue下拉框取字典表值回显为数字不是文字
业务场景为点击`el-table-column`中的`修改`操作弹出dialog并自动填充当前行的实体信息。Dialog中表单其中一项为下拉框，数据绑定（v-model）的值为value，需要根据value从字典中取对应的label并回显。

具体实现中出现回显的是数字而非label文本。

原因为Ruoyi中字典数据表的键和值均为 `varchar` 类型，而自己的数据表相关字段为 `int`
```sql
`dict_label` varchar(100) DEFAULT '' COMMENT '字典标签',
`dict_value` varchar(100) DEFAULT '' COMMENT '字典键值',
```
解决方法：
1. 将数据表（实体）中的字段改为varchar。
2. 将value的值用Number包起来：
```html
<el-select v-model="form.job_grade" clearable placeholder="请选择职级">
    <el-option
        v-for="dict in dict.type.job_grade_type"
        :key="dict.value"
        :label="dict.label"
        :value="Number(dict.value)"
        ></el-option>
</el-select>
```
## JS中null和undefined
遇见一个有意思的小问题。
业务场景为Dialog中表单的数据提交，id没有值，后端通过雪花算法生成自增id，前端中JS异步传值（整个表单的值）。
当表单参数中有初始化，且id默认设为null时，如：
```js
form: {
    id: null,
    ...
}
```
后端接收到的数据为`"id": nil`。将id设置为'',也是一样的情况，最终导致，插入（insert）数据报错，错误信息大致为 字段被定义两次。

解决办法就是不初始化参数，或将其初始化为undefined。

[undefined和null的区别](https://blog.csdn.net/m0_47135993/article/details/119800231):
6、undefined和null的用途
null表示没有对象，即不应该有值，经常用作函数的参数，或作为原型链的重点。

undefined表示缺少值，即应该有值，但是还没有赋予（变量提升时默认会赋值为undefined，函数参数未提供默认为undefined，函数的返回值默认为undefined）
但不建议显式赋值为undefined：
```
建议：无论在什么情况下都没有必要将一个变量显示的赋值为undefined。如果需要定义某个变量来保存将来要使用的对象，应该将其初始化为null.
```
