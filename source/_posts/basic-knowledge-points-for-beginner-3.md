---
title: 由重构进阶前端开发入门 (四) 面向对象
date: 2018-1-4 00:00:00
categories: [前端]
tags: [前端, 初学者, 面向对象]
toc: true
---

了解了在浏览器环境下，使用 JS 编程的基础概念之后，开始思考如何组织优化自己的代码，从编程技巧上提升开发和维护工作的效率吧。

<!-- more -->

> 相关文章：
>
> {% post_link basic-knowledge-points-for-beginner %}
> {% post_link basic-knowledge-points-for-beginner-1 %}
> {% post_link basic-knowledge-points-for-beginner-2 %}

# (四) 面向对象

## DRY (Don't Repeat Yourself) 原则

JavaScript 是一门编程语言，和其它计算机语言一样，在你编码的过程中需要有避免重复代码和逻辑的意识，注意不断优化自己的代码。

以免写出看似体系庞大而可靠，实则冗余且迟滞的代码，埋下 bug 隐患，影响团队伙伴阅读代码、协作开发的效率，也耽误自己以后优化和迭代项目。

因此，其中重要的原则之一就是 **DRY (Don't Repeat Yourself)** 原则。

当你第一次写下某段代码，之后在另一个地方又写下或粘贴同样的代码，你就应该有需要消除和提取重复代码的冲动了。等到第三次，再另一个地方又出现同样的代码时，就可以考虑行动起来，提取共用的代码而不是又重复一遍。

## 函数复用、公用库

例如，在页面某处有一个弹出 Toast 的逻辑，写下了这样的代码：

```javascript
var $toast1 = $('<p class="toast-msg"></p>')
                .text('登陆成功！')
                .on('click', function () {
                  $(this).remove();
                });
$('body').append($toast1);
```

之后又增加了一个类似的 Toast 消息：

```javascript
var $toastX = $('<p class="toast-msg"></p>')
                .text('评论发送失败！')
                .on('click', function () {
                  $(this).remove();
                });
$('body').append($toastX);
```

可以看出几乎所有逻辑都是相同的，区别只在于提示语。这个时候已经没必要再重复，可以提取出一个共用的函数：

```javascript
function showToast(msg) {
  var $toast = $('<p class="toast-msg"></p>')
                  .text(msg)
                  .on('click', function () {
                    $(this).remove();
                  });
  $('body').append($toast);
}

showToast('登陆成功！');

showToast('评论发送失败！');
```

提取**共用函数**可以说是最基本的编程思想了。

这样之后需要增加新的消息，或是对原有的所有提示消息做调整和修复时，不需要修改散落在四处的代码，只需修改一处，效率大大提升。

有一些代码甚至不止可以用于一个项目，还可以在今后的项目开发中继续复用，这些函数逻辑可以提取成**公用代码库**，节省今后项目开发的时间。

## 抽象成对象/类

上面的思想概括起来，其实就是对处理一类事情的过程以函数的形式复用。

是一种相对初级的复用思想，随着业务逻辑逐渐复杂，这种办法的效果也越来越弱。

具体表现就是，打开一个 js 文件，虽然其中没什么重复代码，但却有着几十上百个函数，理不清其中的顺序和关系，可读性很难把控。