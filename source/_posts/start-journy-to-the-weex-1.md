---
title: Weex 开发新手上路 - (2) 前端避坑篇
date: 2019-05-22 00:00:00
categories: [node.js]
tags: [node.js, weex]
toc: true
---

接入 WeexSDK 后，前端开发者编写 Weex 页面时会发现，Weex 页面和以前写的 Web 页面还是有一定区别的，一不注意就踩坑了。这里把常见的一些坑列出来，希望能让大家少走弯路：

<!-- more -->

## 页面模板

页面模板方面，只要注意官方文档内提到哪些组件，以及自己安装的第三方组件，记住只使用这些标签来编写模板即可。

其它经常踩的坑只有一个：文本必须放在 **text** 组件内，**a** 标签内也是如此。且只能使用 Mustache 语法作为节点内容输出，暂不支持 `v-text` 指令输出。

```html
<!-- Wrong -->
<a :href="url1">{{text1}}</a>
<a :href="url1">
    <text v-text="text1"></text>
</a>
```

```html
<!-- Right -->
<a :href="url1">
    <text>{{text1}}</text>
</a>
```

## 页面样式

样式方面，需要注意的就比较多了。直接当作 CSS 来写，最可能发生的结果就是样式失效白屏。

除了官方文档提到的属性外，还有不少需要留意的地方。

### 1. 仅支持单 class 选择器，不支持样式继承

Weex 样式内不支持标签选择器、属性选择器、ID选择器、后代和子层级选择器、相邻选择器，以及 CSS3 中增加的各种计数选择器等。

```css
/* Wrong */
text {
    color: #333333;;
}

input[checked] {
    background-color: red;
}

#link1 {
    margin: 20px;
}

.header a, .header > p, .icon + p {
    background-color: rgba(0,0,0, 0.05);
}
```

说了这么多，其实只需要记住一种：Weex 内目前仅支持 `.className` 式的类型选择器。

```css
/* Right */
.class1 {
    margin: 20px;
    background-color: rgba(0,0,0,0.05);
}

.text1 {
    font-size: 24px;
    color: #333333;
}
```

伪类方面，Weex 支持四种伪类：`active`, `focus`, `disabled`, `enabled`。

所有组件都支持 `active`, 但只有 `input` 组件和 `textarea` 组件支持 `focus`, `enabled`, `disabled`。

### 2. 只支持 px 单位，不支持百分比宽高

根据官方文档的描述，我们知道 Weex 内只能使用像素值单位 `px`。

> - Weex 不支持类似 `em`、`rem`、`pt`,`%` 这样的 CSS 标准中的其他长度单位；
>
> - 单位 `px` 不可省略，否则在 Web 环境无法正确渲染；

且这个单位并非真实屏幕像素，而是把所有屏幕视为 `750px` 宽度，`1px` 则是**屏幕宽度的七百五十分之一**，而非一个真实像素的宽度，以此来实现不同设备屏幕宽度的适配。

### 3. 默认纵向布局，子元素拉伸填充侧轴

Weex 中支持且仅支持 flexbox 布局方式。但你会发现，在不指定 `flex-direction` 属性的时候内部元素是纵向布局的。

因为 Weex 中默认 `flex-direction` 为 `column` 而不是 `row`。

而且不设置子元素的宽度，父元素的 `align-items` 为默认的 `stretch` 时，子元素将自动拉伸填充侧轴宽度。

{% img "/images/start-journy-to-the-weex/flex-direction-and-stretch.png" cover %}

由于 Weex 不支持 `%` 单位，某个组件需要适配父组件宽度的时候，可以通过这个方式实现宽度 `100%` 的效果。

### 4. 属性名尽量用全称

在完成本文章时，使用 Weex 版本 `v1.3.11` 测试以下样式写法的情况如下：

```css
.t1 {   /* 有效 */
  margin: 20px;
}

.t2 {   /* 无效 */
  margin: 20px 40px;
}

.t3 {   /* 有效 */
  margin-top: 20px;
  margin-bottom: 20px;
  margin-left: 40px;
  margin-right: 40px;
}

.t4 {   /* 无效 */
    border: 1px solid blue;
}

.t5 {   /* 有效 */
    border-width: 1px;
    border-style: solid;
    border-color: blue;
    
    border-top-width: 2px;
    border-top-style: dashed;
    border-top-color: green;
}

.t6 {   /* 无效 */
    background: red;
}

.t7 {   /* 有效 */
    background-color: red;
}

```

可以看出，使用简写形式书写的样式，Weex 不一定支持，所以在 Weex 内尽量使用全称的样式，以免失效。

### 5. background-image 不能使用图片资源

尝试在 Weex 内使用 `background-image` 样式引入图片，然而并没有效果，官方文档内此属性也只用来实现背景色渐变。

看来 Weex 内应该是只能 `image` 组件展示图片资源了。

### 6. 多行等分布局问题

之前说过，默认子元素侧轴拉伸对齐的情况下，不设置子元素宽度即可实现宽度 `100%` 的适配。

如果子元素需要等分父元素宽度，只要使他们的 `flex-grow` 权重一致即可，但这个适配方案只能处理单行的情况。

对于 Web 页面 flex 多行布局的情况，我们给父元素设置 `flex-wrap: wrap;` 属性后，通常根据每行子元素数量设定子元素宽度的百分比。如每行两个子元素时，就给它们设置 `width: 50%;`。

但之前说过，Weex 内不支持百分比单位，而 `flex-grow` 达不到这样的效果。

所以只能假定，父元素一定是占满屏幕宽度，子元素根据 `750px/n` 来设置宽度。或者从父组件，通过属性传递告知子组件宽度，再对子组件内的子元素进行宽度计算适配，流程较复杂，且样式部分耦合到了脚本逻辑内。

目前暂时没有想到更好的方案，只能假定父元素一定是 `750px` 宽度处理，日常业务一般也是这样的情况，只能保持警惕，遇到这样的情况时注意适配了。