---
title: 由重构进阶前端开发入门 (一) DOM 操作
categories:
- 前端
tags:
- 前端
- 初学者
toc: true
---

虽然前端的技术每年都在不断更新，但新人们都还是从基础的 HTML、CSS 样式开始学 Web 前端开发的。

这样切图、使用 HTML + CSS 编写网页的工作过程，我们一般称之为`网页重构`，关于`网页重构`和`前端开发`是否应该分离也一直存在争议。

但就日常工作的情况来看，二者还是很难彻底分开的，前端开发写脚本时必定需要网页重构的基础（比如动画控制、3D变换等），网页重构时也需要提前考虑前端脚本可控制标签的埋点，尽量避免后期再对页面结构和样式调整改动。

总体来说，前端开发是一个主要的发展方向，近几年来也是大趋势。

那么，新同学们掌握网页重构后，又该开始哪些知识的学习？以便向前端开发进阶，拓展自己的技能树、全面发展呢？

个人认为主要需要学习的都是各类 JS 知识点。这里对相关知识点稍作整理归类，可能分几篇文章各自小结一下，希望能抛砖引玉，对新人们的学习有所帮助。

<!-- more -->

# DOM 操作

## 1. 什么是 DOM

做前端开发，每天都在和 DOM 打着交道，那么 DOM 到底是什么呢？

有人可能会说它们就是页面里的 HTML 标签，但这么描述其实不够确切，W3C 对其的标准定义是：

>
> “W3C 文档对象模型 （DOM） 是中立于平台和语言的接口，它允许程序和脚本动态地访问和更新文档的内容、结构和样式。”
>

具体点说，其实是浏览器拿到 Web 文档后，对 HTML 标签进行分析，处理成了对应的可操作对象，这类对象被称为`文档对象模型(Document Object Model, DOM)`。

而除了页面里的标签外，我们也可以自己手动创建 DOM 的，所以 DOM 的来源不只是页面里的 HTML 标签。

获得 DOM 对象之后，我们还可以对它们做一系列操作，以满足日常各种页面开发需求。

## 2. 常用 API

### 2.1 修改内容

来看看最常见，也是最基本的需求之一，修改页面内某个位置的文本，想必大家都做过这样的操作。

比如在页面某处显示当前的时间：

```html
<!DOCTYPE html>
<p>
    当前时间是：<span id="txt_currTime">...</span>
</p>
```

首先需要获取页面元素对应的 DOM 来进行操作，大家一般常用 `getElementById`。其它方式需要自己注意浏览器兼容性问题，以及区分单个 DOM 和 DOM 数组返回值的情况。

```html
<script>
    // 定时修改时间文本
    setInterval(function () {
        var currTime = document.getElementById('txt_currTime');
        currTime.textContent = new Date();
    }, 1000);
</script>
```

根据 `id` 找到占位的 `span` 标签对应 DOM 后，修改 DOM 的 `textContent` 属性，浏览器自动反馈到页面更新，就完成了修改页面内容的常见操作。

这里只做简要介绍，涉及到 `textContent` 的兼容性问题，以及与 `innerText`、`innerHTML` 的区别问题，想要深入学习的可以参考：[《JS魔法堂：被玩坏的innerHTML、innerText、textContent和value属性》](http://www.cnblogs.com/fsjohnhuang/p/4319635.html)。

### 2.2 修改样式

最常见的需求之二，修改页面某处元素的显示效果。

直接修改样式：

```html
<style>
    .title {
        color: red;
    }
</style>
<p class="title" id="txt_mainTitle">
    标题文本
</p>
<script>
    // 修改标题字体样式
    document.getElementById('txt_mainTitle').style.fontWeight = 'bold';
</script>
```

通过切换 `class` 修改显示效果：

```html
<style>
    .title {
        color: red;
    }
    .title.current {
        font-weight: bold;
    }
</style>
<p class="title" id="txt_mainTitle">
    标题文本
</p>
<script>
    // 修改标题样式 class
    document.getElementById('txt_mainTitle').classList.add('current');
    // document.getElementById('txt_mainTitle').className += (' current');
</script>
```

### 2.3 动态创建元素

最常见需求之三，根据获得的数据，动态创建页面元素。

最基本的方法就是修改 `innerHTML`：

```html
<ul id="lst_msgList"></ul>
<script>
    function renderMsgList(data) {
        var list = document.getElementById('lst_msgList');
        var buffer = '';
        for (var i = 0, l = data.length; i < l; i++) {
            buffer += '<li>' + data[i] + '</li>';
        }
        list.innerHTML = buffer;
    }
</script>
```

更标准的方法则是通过 `document.createElement` 创建 DOM 后，加入到指定位置：

```html
<ul id="lst_msgList"></ul>
<script>
    function renderMsgList(data) {
        var list = document.getElementById('lst_msgList');
        for (var i = 0, l = data.length; i < l; i++) {
            var item = document.createElement('li');
            item.textContent = data[i];
            list.appendChild(item);
        }
    }
</script>
```

过去，使用 `innerHTML` 的性能高于 `createElement` 创建元素。但经过浏览器迭代优化后，两者性能已经相差无几了。

而且，在父元素内已有大量子元素时，需要在子元素内删除成员或插入新成员时，直接修改父元素的 `innerHTML` 会导致所有子元素重新渲染，性能开销大。

而且重新渲染创建的子元素与之前的子元素并非同一实例，会丢失之前对子元素绑定的事件监听器，导致各种意外情况，需要注意。

## 3. 属性 (Property) 与特性 (Attribute)

两者有时候都被称为“属性”，容易混淆，简单区分的话，可以这么理解：

`Attribute` 是 `HTML` 形式表示时的页面元素内原有的属性（特性），比如 `id`、`name`、`value`、`title` 以及各种自定义的 `data-xxxx`。

`Property` 则是脚本内获取到的 DOM 对象附带的字段属性，如 `id`、`innerHTML` 等。

|字段     |是否属性  |是否特性  |
|:--------|:--------|:--------|
|id       |√        |√        |
|innerHTML|√        |×        |
|data-xxxx|×        |√        |

操作 `Attribute` 的标准方法是调用 DOM 的 `getAttribute(key)` 和 `setAttribute(key, value)`，其中 `getAttribute` 的返回值和 `setAttribute` 第二个参数 `value` 都必需是字符串类型。

```html
<p class="title" id="txt_mainTitle" data-index="1" style="color: red;"></p>
<script>
    var mainTitle = document.getElementById('txt_mainTitle');

    mainTitle.getAttribute('id');           // 'txt_mainTitle'
    mainTitle.setAttribute('title', 'This is a title.');
</script>
```

`Property` 则是直接使用 JS 对象的字段读写操作就行了，`title.id` 或 `title['id']` 皆可。

常用的 `Attribute`，例如 `id`、`title` 等，会被浏览器自动转为 `Property` 附加到 DOM 对象上，方便日常操作。

不过有些需要注意的情况：

1. 因为 `class` 是 `ECMA` 的关键字，作为 `Property` 使用时字段名叫做 `className`；

2. 为了便于操作，`style` 会被转化成对象形式（键值对），而非其它特性的字符串值。

```javascript
    mainTitle.id === mainTitle.getAttribute('id');              // true
    mainTitle.className === mainTitle.getAttribute('class');    // true

    mainTitle.style;                                            // { color: 'red', ... }
```

除了 HTML 标准内的 `Attribute` 之外，我们可以添加各种自定义 `Attribute` 一般会用于在创建或渲染元素时，附加特定的文本信息。

这些信息对用户不可见，但在页面源代码和浏览器调试工具的 DOM 树内能清晰看到，方便开发者进行元素选取和页面调试等操作。

但这些自定义 `Attribute` 不会自动转为 `Property` 处理，需要通过 `getAttribute`/`setAttribute` 进行操作。

```javascript
    mainTitle.getAttribute('data-index');   // '1'
```

`Property` 操作更加自由，但一般还是不推荐给 DOM 增加自定义 `Property`，因为操作不当时会影响部分浏览器的垃圾回收检测，容易导致内存泄漏。

# 小结

本文是入门内容，只讲述了最常见的几种情况，更多 DOM 操作可以自行根据需求，在 `W3School` 等网站查找更多 API 结合实例进行练习。

由于历史原因，很多 API 涉及到浏览器兼容性问题，建议新手在练习中进行了解，便于日后碰到问题时知道如何应对。

但日常生产环境内，推荐优先使用 jQuery 等框架来处理，而非自己处理兼容性问题。

因为很多兼容性问题需要结合实际情况进行针对处理，产生的代码不方便他人阅读，不利于协同开发。

而且要处理得较为完善，需要对现有 API 进行大量封装，这些工作现有框架已经完成得很出色了，没必要特意重新造轮子。

因此，后续文章将以 jQuery 为例，在介绍原生 DOM 操作 API 之后，给出 jQuery 的处理方案进行对比。希望初学者了解其中原由，不要混淆两者的操作（见过很多新手混淆原生与 jQuery API，有些无奈……）。

<!--

# 事件对象

## 1. 什么是事件对象

## 2. 常用属性与函数

## 3. 事件冒泡

# MVVM

## 1. 摆脱 DOM 操作的轮回

## 2. 数据驱动的优点（购物车、消息数）

## 3. UI 组件化开发

# Markdown

## 1. Talk is cheap, show me the doc.

# Node.js

## 1. 从零开始用 JS 写客户端

## 2. 站在巨人肩膀上做 GUI 客户端

-->