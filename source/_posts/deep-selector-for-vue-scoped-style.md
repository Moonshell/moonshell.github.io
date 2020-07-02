---
title: Vue 样式中的深度选择器 /deep/ 和 >>>
date: 2020-07-02 00:00:00
categories: [前端]
tags: [vue, style, less, sass]
toc: true
---

当你使用第三方 UI 框架时，可能会遇到内置样式不满足业务需求，需要亲手修改框架提供组件的默认样式的情况。

<!-- more -->

比如 UI 框架提供了一个菜单组件 `<iv-menu>`，但是其中标题文本的效果不符合我们的预期。我们在 Chrome Inspector 中找到对应 DOM，发现 **className** 为 `.title`，于是就添加了这样的样式：

```html
<!-- page.vue -->

<style lang="scss" scoped>
    .page {
        margin: 0;
    }

    .iv-menu {
        background-color: #ffffff;
    }

    .iv-menu .title {
        font-size: 12px;
    }
</style>

<template>
    <div class="page">
        <iv-menu title="标题"></iv-menu>
    </div>
</template>
```

结果——添加的样式并没有生效。

## 原因

因为 **page.vue** 这里我们使用了 `scoped` 样式作用域，Vue 会为当前模板内所有元素会被增加一个特殊属性（如：`[data-v-5ef48958]`），并且为所有样式选择器最后一级添加这个属性的选择器。

生成的样式和 DOM 大致是这样的：

```html
<style>
    .page[data-v-5ef48958] {
        margin: 0;
    }
</style>

<div class="page" data-v-5ef48958></div>
```

可以看到 `.page` 选择器自动变成了 `.page[data-v-5ef48958]`，从而达到这个组件的 `.page` 样式不污染其它同名样式的效果。

而这个处理，也就是导致我们无法修改子组件内样式的原因。毕竟，不污染子组件样式其实就是样式作用域本身预期的效果。

## 分析

上面例子中修改 `<iv-menu>` 组件内标题的例子，生成代码大致如下：

```html
<div class="page" data-v-5ef48958>
    <div class="iv-menu" data-v-5ef48958>
        <p class="title">标题</p>
    </div>
</div>
```

对应样式为：

```css
.page[data-v-5ef48958] {
    margin: 0;
}

.iv-menu[data-v-5ef48958] {
    background-color: #ffffff;
}

.iv-menu .title[data-v-5ef48958] {
    font-size: 12px;
}
```

其中 `.page[data-v-5ef48958]` 和 `.iv-menu[data-v-5ef48958]` 的样式对应的 DOM 选择器都是正确的。

但是对于 `.iv-menu` 内部的 `.title`，Vue 的样式作用域处理逻辑认为它属于当前组件，所以生成的选择器是 `.iv-menu .title[data-v-5ef48958]`。

而实际需要的选择器其实是 `.iv-menu[data-v-5ef48958] .title`。

也就是说，只需要告诉 Vue 的样式作用域处理逻辑：“*我们这个组件只管到 `.iv-menu`，后面的 `.title` 是属于更深的子组件样式，不要加作用域处理*”，就行了。

## 解决

而 Vue 已经提供了这样的告知方法，就是深度选择器 `/deep/`。只需要在组件样式内加入它就行了：

```html
<!-- page.vue -->

<style lang="scss" scoped>
    .page {
        margin: 0;
    }

    .iv-menu {
        background-color: #ffffff;
    }

    .iv-menu /deep/ .title {
        font-size: 12px;
    }
</style>

<template>
    <div class="page">
        <iv-menu title="标题"></iv-menu>
    </div>
</template>
```

如果遇到 SASS 处理器不识别 `/deep/` 而报错，可以改成 `>>>` 或者 `::v-deep` 操作符，三者是一样的。

> 官方说明：https://vue-loader.vuejs.org/zh/guide/scoped-css.html#%E6%B7%B1%E5%BA%A6%E4%BD%9C%E7%94%A8%E9%80%89%E6%8B%A9%E5%99%A8
