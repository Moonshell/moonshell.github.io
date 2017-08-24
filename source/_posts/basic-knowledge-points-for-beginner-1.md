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

> 相关文章：
>
> [由重构进阶前端开发入门 (一) DOM 操作](http://blog.krimeshu.com/2017/05/04/basic-knowledge-points-for-beginner/)

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
* 操作元素位置和尺寸使用 `offset`, `width`, `height`
* 操作元素属性/特性使用 `prop`, `attr`, `removeProp`, `removeAttr`
* 操作元素样式/`className` 时使用 `css`, `addClass`, `removeClass`, `toggleClass`, `hasClass`
* 父元素插入/追加/移除子元素使用 `prepend`, `append`
* 子元素附近插入/查找邻近元素使用 `prev`, `next`, `prevAll`, `nextAll`
* 子元素移除时使用 `remove`
* 查找父元素/祖先元素使用 `parent`, `parents`

掌握上述最常用也是最基本的 API 的使用方法和对应场景后，就可以实现 90% 以上的日常业务需求了。

## 事件对象 `e`

由于原生 DOM API 写起来太过繁琐，以及兼容性的处理太过复杂，这里推荐使用 jQuery 等现成框架，业余时间再对常见的兼容性进行了解，以便遇到意外时知道问题出在何处。

当我们对页面元素绑定了事件处理器后，常常会看到一个神秘的 `e` 参数：

```javascript
    $('#btn_update').on('click', function (e) {
        // Todo: ...
    });
```

它就是所谓的“事件对象”了，不过上面的例子里似乎并没有用上它啊？它到底有什么作用呢？

个人理解是，“事件”就是用户操作、浏览器状态变化这些正在发生的事情；而“事件对象”就是这个“事件”发生的相关信息。

比如用户点击按键 `#btn_update` 后，触发了点击事件 `click`，事件的监听函数接收到的事件对象 `e` 就会包含这次点击的相关信息，如点击坐标、发起元素、传递路径等等。

此外，事件对象 `e` 还提供了一些 API 可以对这次事件进行一些附加操作，下面进行具体说明。

### 例子：幻灯片切换效果（点击位置判断）

过去对于用户点击图片区域判断，需要通过 img 元素的 usemap 属性实现，使用方式较为复杂，且限制较多，可复用性低。

现在，实现一个简单的幻灯片点击切换效果，只需根据事件对象中相关参数来判断即可。

大致效果是：用户点击左右两侧 20% 区域时，切换展示上/下一章图片；点击中间区域不处理。

```html
<!DOCTYPE html>
<style>
    .pic-list {
        margin: 0;
        padding: 0;
        list-style: none;
        width: 400px;
        height: 300px;
        overflow: hidden;
    }
    .pic-item {
        display: none;
    }
    .pic-item.current {
        display: block;
    }
    .pic-item img {
        display: block;
        width: 100%;
        height: 100%;
    }
</style>
<ul class="pic-list">
    <li class="pic-item current">
        <img src="images/1.jpg"/>
    </li>
    <li class="pic-item">
        <img src="images/2.jpg"/>
    </li>
    <li class="pic-item">
        <img src="images/3.jpg"/>
    </li>
</ul>
```

只需要对列表元素绑定点击事件，然后根据点击位置和列表宽度，就能判断出用户点击的区域，然后做样式切换即可：

```javascript
// 1. 选择 .pic-list 元素，绑定 click 事件监听器
$('.pic-list').on('click', function (e) {
    var $list = $(this),
        $item = $list.find('.pic-item'),
        itemCount = $item.length;
        
    // 2. 获取点击坐标，列表元素坐标和列表宽度
    var clickPos = { left: e.pageX, top: e.pageY },
        listPos = $list.offset(),
        listWidth = $list.width();
        
    // 3. 点击坐标 - 列表坐标，再除以列表宽度，即可得到点击相对列表的横向位置百分比
    var px = (clickPos.left - listPos.left) / listWidth;
    
    // 4. 根据点击是否 0%~20%, 80%~100% 处理上下页切换
    var curIndex = $item.filter('.current').index();    // 当前序号
    if (px < .2) {
        curIndex = (curIndex + itemCount - 1) % itemCount;
    } else if (px >= .8) {
        curIndex = (curIndex + 1) % itemCount;
    }
    
    // 5. 切换当前显示的子元素
    $item.removeClass('current').eq(curIndex).addClass('current');
});
```

上述代码用原生 JS 写会复杂很多，因为需要离开 jQuery 编写兼容 IE8 的代码，需要对事件绑定、事件对象获取、元素查找、点击坐标、元素坐标等操作做大量兼容处理。做兼容处理的代码甚至会比主要逻辑代码还要多得多。

有兴趣的同学课余可以尝试一下，使用原生 JS 兼容 IE8 和现代浏览器后，再对比上述代码，就能明白 jQuery 的强大之处了~

### 附：可用的跨浏览器兼容的 jQuery 标准化事件属性

jQuery 按照 W3C 标准规范，将不同浏览器的事件对象处理成了同一格式，免去了日常业务层反复做浏览器兼容的繁琐工作。

大部分属性只需要参考 W3C 规范即可，各属性具体说明可阅读 jQuery 的 API 文档进行了解：

>官方文档：[jQuery | Event Object](http://api.jquery.com/category/events/event-object/)
>
>中文文档：[jQuery | 事件对象](http://www.jquery123.com/category/events/event-object/)

<!--

## 事件冒泡与事件代理

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