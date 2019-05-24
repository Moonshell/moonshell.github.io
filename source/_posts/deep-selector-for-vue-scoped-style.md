---
title: 为什么要在 Vue 组件的 scoped 样式中使用 /deep/ 选择器
date: 2018-12-01 00:00:00
categories: [前端]
tags: [vue, style, less, sass]
toc: true
---

最近使用一个 UI 框架时，遇到一些情况，需要修改框架提供组件的默认样式。

```html
<style lang="scss" scoped>
    .iv-menu .title {
        font-size: 12px;
    }
</style>

<template>
    <iv-menu></iv-menu>
</template>
```