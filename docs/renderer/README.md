# 渲染器的作用

Vue 实际上都是解析`<template/>`标签内的内容生成对应的`render`函数进行渲染的，渲染器就是这个作用，将生成的权利下放到了用户手中，同时提供`jsx`支持，让用户可以手动决定在`seed`各个部分的渲染方式,当然，seed 也内置了一些常用的渲染器

## Seed 的策略

如果在`seed.render`的字段内传入的是字符串的话，就会自动从内置的渲染器进行查找同样类型的渲染器进行渲染，如果没有，就会尝试使用函数渲染，实际上，内置的渲染器也是同一个方案

## render 函数

`render`函数因为支持返回`jsx`,所以第一个是渲染函数`createElement`, 具体请参考[vue 渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)

`render`的参数按照顺序有如下几个

### h

vue 的 CreateElement

### value

seed 通过内置函数获取的`value`, 当然如果是`form`的话可能返回为`null`

### scope

一些额外的数据，在不同环境下不太一样

- `table` 和 `el-table-column`的渲染函数一致 `{ row, column, $index }`
- `form` 和 `filter` 则是`{ form, handleChange }`，因为内置的`form`的格式问题，请使用`handleChange`来进行更新

### seed

当前的配置，根据不同位置已经将`options`正确合并
