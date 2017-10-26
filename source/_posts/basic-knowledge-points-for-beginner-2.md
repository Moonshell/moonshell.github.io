---
title: 由重构进阶前端开发入门 (三) 事件冒泡与事件代理
categories:
- 前端
tags:
- 前端
- 初学者
- 事件
- 冒泡
- 代理
- 委托
toc: true
---

上一篇章中，我们掌握了页面事件的基本操作，这次学习事件 API 的进阶和拓展用法了。

<!-- more -->

> 相关文章：
>
> [由重构进阶前端开发入门 (一) DOM 操作](http://blog.krimeshu.com/2017/05/04/basic-knowledge-points-for-beginner/)
> [由重构进阶前端开发入门 (二) 事件与事件对象](http://blog.krimeshu.com/2017/06/29/basic-knowledge-points-for-beginner-1/)

# (三) 事件冒泡与事件代理

## 事件冒泡

假设你需要实现这样的效果：用户登录状态过期了，点击页面内任何按键都给出提示；或者右上角提示通知消息时，点击页面内任何元素后折叠通知消息。

这两种情况下，你会把点击事件的监听器绑定到哪些元素上呢？

把页面所有按键、甚至所有页面元素都绑定一遍？——你肯定是开玩笑的对吧，这么做勉强能达到想要的效果，但未免也太暴力，性能太低、可维护性也太差了。

且不说绑定如此多元素的监听器的效率，一旦页面里的元素有变动、或者状态变更后需要解除绑定，都得做各种额外零散的补救操作。这样的代码可以说没法应对业务的任何变更，几乎能逼死之后的维护人员。

正确姿势其实很简单。

当年浏览器事件模型已经帮我们考虑到了这种情况，提供了“事件冒泡”这一机制。

由于页面内元素是层级嵌套的。当你点击某个按键时，也可以说是点击了它所在的父元素中的某个位置。由此类推，层层递进，就相当于点击了整个 html 文档的某处。

{% img "/images/basic-knowledge-points-for-beginner/popup.png" popup %}

因此点击事件会从事件发生的最底层元素开始，自动一层层往上走，不出意外，会一直触发到页面的最外层。

所以，我们只需要在 **body** 元素上绑定点击事件监听函数，处理完毕后清理掉即可。

```javascript
function showToast(msg) {
  // 1. 展示 toast 内容
  $('.toast-content').text(msg).show();
  // 2. 绑定点击后关闭的事件处理函数
  $('body').on('click', hideToast);
  
  function hideToast() {
    // 3. 收起 toast 内容
    $('.toast-content').hide();
    // 4. 解除绑定的函数
    $('body').off('click', hideToast);
  }
}
```

在第二篇《事件与事件对象》中，我们提到过通常名为 `e` 的事件对象参数。

它除了携带事件相关信息的各种属性之外，还有一个与事件冒泡相关的函数 `stopPropagation`。

在内部元素已经处理完事件，不需要传递到外层，以免引发其它意外情况时，可以用来阻止事件继续向上冒泡。

事件对象的 `target` 属性是触发事件的对象，可用来在外层进行甄别，决定事件的具体处理方式

```javascript
$('.btn-login').on('click', function (e) {
  // 点击登录按键时，就不用提示登录过期了，所以我们阻止这次点击冒泡到 body 层
  e.stopPropagation();
});

// .btn-login 的点击事件在里层被阻止冒泡了，最外层的 body 接收不到，不会再给出过期提示
$('body').on('click', function (e) {
  // 登录过期时，给出登录过期提示
  alert('登录过期，请重新登录后再进行操作。');
});
```



## 事件默认行为

上面举的例子中，提到过登陆过期时的提示信息。

但如果用户点击的是 **a** 标签，跳转的话就很难看见提示信息了，这种情况该怎么办呢？

这个时候可以使用事件对象的另一个函数 `preventDefault` 来阻止浏览器对各种元素的默认处理行为，比如这里的 **a** 标签跳转行为。

```javascript
// .btn-login 的点击事件在里层被阻止冒泡了，最外层的 body 接收不到，不会再给出过期提示
$('body').on('click', function (e) {
  // 登录过期时，给出登录过期提示
  alert('登录过期，请重新登录后再进行操作。');
  e.preventDefault();
});
```


## 事件代理

上面的例子还是比较简单的，实际业务中需要对业务状态、点击的具体元素进行筛选判断才行。

这时候就得用到事件对象里的 `target` 属性了，通过 **jQuery** 对象的 `is`、`closest` 等函数即可做具体的判断：

```javascript
// .btn-login 的点击事件在里层被阻止冒泡了，最外层的 body 接收不到，不会再给出过期提示
$('body').on('click', function (e) {
  // 登录过期时，点击的元素带有 need-login 特性的话，统一给出登录过期提示
  // (如：`<button need-login="true">编辑内容</button>`)
  if (window.LOGIN_STATE === 'expired' && 
     $(e.target).is('[need-login="true"]')) {
    alert('登录过期，请重新登录后再进行操作。');
    e.preventDefault();
  }
});
```

这样将事件监听函数加到父元素上，借助事件冒泡机制来处理数目不定的子元素事件的方式，就被叫做**事件代理**（或**事件委托**）。

除了上述情况，实际业务中可能还会遇到需要处理动态增减的数据，对上百上千的数据条目提供点击处理，都是通过这样绑定父元素做**事件代理**来处理的。

**jQuery** 的 **on** 函数还提供了更快捷的绑定方式，直接在绑定的时候增加一个筛选的选择器即可：

```javascript
$('body').on('click', '[need-login="true"]', function (e) {
  // 登录过期时，点击的元素带有 need-login 特性的话，统一给出登录过期提示
  // (如：`<button need-login="true">编辑内容</button>`)
  if (window.LOGIN_STATE === 'expired') {
    alert('登录过期，请重新登录后再进行操作。');
    e.preventDefault();
  }  
});
```

