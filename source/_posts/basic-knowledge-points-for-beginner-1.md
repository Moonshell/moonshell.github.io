---
title: 由重构进阶前端开发入门 (二) 事件与事件对象
categories:
- 前端
tags:
- 前端
- 初学者
- 事件
toc: true
---

# 事件与事件对象

## 事件与 DOM

### 1. 从最初的“点击”开始

> “点击这个按键时，XXX 变成 YYY，然后……”

新手最初学会的，基本是这样使用 `onclick` 事件属性进行处理：

```html
<button onclick="alert('Hello world!')">Button</button>
```

或者对复杂些的处理，绑定全局函数：

```html
<span id="txt_info"></span>
<button onclick="updateInfo">Button</button>
<script>
    function updateInfo() {
        document.getElementById('txt_info').textContent = new Date().toString();
    }
</script>
```

### 2. 新的处理方式

网上古老的 JS 入门教程可能都是这么写的。
不过，二十年来业务不断进化，这样的处理方式虽然还被浏览器所支持，但暴露出越来越多问题，已经不再建议使用了。

一是这种方式只能绑定一个处理函数，且不能取消绑定不够灵活；二是全局函数容易混淆，项目到达一定规模后容易失控，导致意外。

W3C 标准推荐使用 DOM 对象的 `addEventListener` 和 `removeEventListener` 方法来绑定和取消绑定处理函数。

```html
<span id="txt_info"></span>
<button id="btn_update">Button</button>
<script>
    document.getElementById('btn_update').addEventListener('click', function () {
        document.getElementById('txt_info').textContent = new Date().toString();
    }, false);
</script>
```

低版本 IE 浏览器使用了自家的另一套事件相关方法，已经随着浏览器标准迭代被废弃，这里不再赘述。

不过日常需要对 IE8 这一类浏览器进行支持时，一般使用 jQuery 等现成做好了兼容性处理的框架，使用方便快捷，API 也是一目了然，非常容易理解。

上述代码在使用 jQuery 的时候可以写作：

```html
<span id="txt_info"></span>
<button id="btn_update">Button</button>
<script>
    $('#btn_update').on('click', function () {
        $('#txt_info').text(new Date().toString());
    });
</script>
```

### 3. DOM 对象与 jQuery 对象

上面 jQuery 的代码和之前的原生 JS 代码等效，但有一点需要注意，也是新手经常混淆的。

`$('<css选择器>')` 的效果是根据 CSS 选择器找到页面上对应的元素，返回的对象可以对其做绑定事件处理器等操作，如上面的 `$.fn.on` (即 `$(...).on`)。

但是仔细观察这个返回对象，你会发现它并不是原生的 DOM 对象，对它做原生 DOM 相关的操作时会发现它并不支持原生 DOM 相关的 API。

```javascript
    $('#txt_info').textContent = new Date();    // × - 无效
    $('#txt_info').text(new Date());            // √ - 有效
```

它其实是一个特殊数组，里面存放着传入 CSS 选择器所对应的所有 DOM 对象，通过 `.length` 可以查看其中选中的元素数量，通过数组下标可以取出其中的原生 DOM 对象：

```javascript
    $('#text_info').length;                     // => 1
    $('#text_info')[0] === 
        document.getElementById('text_info');   // => true
```

当 jQuery 选择了多个元素时，各类 API 操作可以对立面所有成员进行处理，非常方便：

```javascript
    $('blockquote').prepend('<span>“</span>');  // 所有 blockquote 元素内容前插入 “ 符号
```

### 4. 常用 jQuery API

* 选择元素和创建元素使用 `$()`, `find`, `filter`
* 处理事件使用 `on`, `off`, `trigger`
* 操作元素内容使用 `text`, `html`
* 操作元素属性/特性使用 `prop`, `attr`, `removeProp`, `removeAttr`
* 操作元素样式/`className` 时使用 `css`, `addClass`, `removeClass`, `toggleClass`, `hasClass`
* 父元素插入/追加/移除子元素使用 `prepend`, `append`
* 子元素附近插入/查找邻近元素使用 `prev`, `next`, `prevAll`, `nextAll`
* 子元素移除时使用 `remove`
* 查找父元素/祖先元素使用 `parent`, `parents`

掌握上述常用 API 的使用方法和对应场景后，基本就可以实现 90% 以上的日常业务需求了。

## 事件对象 `e`

由于原生 DOM API 写起来太过繁琐，以及兼容性的处理太过复杂，这里推荐使用 jQuery 等现成框架，业余时间再对常见的兼容性进行了解，以便遇到意外时知道问题出在何处。

当我们对页面元素绑定了事件处理器后，常常会看到一个神秘的 `e` 参数：

```javascript
    $('#btn_update').on('click', function (e) {
        // Todo: ...
    });
```

它就是所谓的“事件对象”了，不过上面的例子里似乎并没有用上它啊？它到底有什么作用呢？

<!--

# 事件与事件对象

## 1. 什么是事件

## 2. 处理事件

### 事件属性

### 事件监听器

## 3. 事件冒泡与捕获

## 4. 事件对象

## 5. 事件代理

# MVVM

## 1. 摆脱 DOM 操作的轮回

## 2. 数据驱动的优点（购物车、消息数）

## 3. UI 组件化开发

## 4. 面向对象（类、实例、方法、字段）

# Markdown

## 1. Talk is cheap, show me the doc.

# Node.js

## 1. 从零开始用 JS 写客户端

## 2. 站在巨人肩膀上做 GUI 客户端

-->