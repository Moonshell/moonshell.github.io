---
title: 由重构进阶前端开发入门 (四) 面向对象（上）
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

以免写出看似体系庞大而可靠，实则冗余且迟滞的代码，埋下 bug 隐患，影响团队伙伴阅读代码、协作开发的效率，也耽误自己以后优化和迭代项目。

因此，其中重要的原则之一就是 **DRY (Don't Repeat Yourself)** 原则。

当你第一次写下某段代码，之后在另一个地方又写下或粘贴同样的代码，你就应该有需要消除和提取重复代码的冲动了。

等到第三次，再另一个地方又出现同样的代码时，就可以考虑行动起来，提取共用的代码而不是又重复一遍。

## 函数复用、公用库

最基本的方法，就是把重复代码提取成复用的函数。

### 对话框展示函数

例如，在页面某处有一个弹出 Dialog 的逻辑，写下了这样的代码：

```javascript
var $dialog1 = $('<div class="dialog-box">' + 
                 '  <p class="dialog-msg"></p>' +
                 '  <a class="dialog-btn" close-dialog>确定</a>' + 
                 '</div>')
                .text('登陆成功！')
                .on('click','[close-dialog]', function () {
                  $(this).closest('.dialog-box').hide();
                })
                .find('.dialog-msg')
                .text('评论发送失败！');
$('body').append($dialog1);
```

之后又增加了一个类似的 Toast 消息：

```javascript
var $dialog2 = $('<div class="dialog-box">' + 
                 '  <p class="dialog-msg"></p>' +
                 '  <a class="dialog-btn" close-dialog>确定</a>' + 
                 '</div>')
                .on('click', '[close-dialog]', function () {
                  $(this).closest('.dialog-box').hide();
                })
                .find('.dialog-msg')
                .text('评论发送失败！');
$('body').append($dialog2);
```

这段代码与上面那段几乎所是相同的，区别只在于提示语。

明显没必要这样重复，可以提取出一个通用的兑换框展示函数 **showDialog**：

```javascript
function showDialog(msg) {
  var $dialog = $('<div class="dialog-box">' + 
                  '  <p class="dialog-msg"></p>' +
                  '  <a class="dialog-btn" close-dialog>确定</a>' + 
                  '</div>')
                  .on('click', '[close-dialog]', function () {
                    $(this).closest('.dialog-box').hide();
                  })
                  .find('.dialog-msg')
                  .text(msg);
  $('body').append($dialog);
}

showDialog('登陆成功！');

showDialog('评论发送失败！');
```

提取**共用函数**可以说是最基本的编程思想了。

这样之后需要增加新的消息，或是对原有的所有提示消息做调整和修复时，不需要修改散落在四处的代码，只需修改一处，效率大大提升。

有一些代码甚至不止可以用于一个项目，还可以在今后的项目开发中继续复用，这些函数逻辑可以提取成**公用代码库**，节省今后项目开发的时间。

## 抽象成对象/类

上面的思想概括起来，其实就是将处理一类事务的过程，以函数的形式复用。

是一种相对初级的复用思想，随着业务逻辑逐渐复杂，这种办法的效果也越来越弱。

结果就是，这样写出来的 js 文件，到达一定规模之后，其中虽然没什么重复代码，但却有着几十上百个函数。阅读者理清其中的顺序和关系会很耗时，难以保证可读性。

而且函数形式的复用，并不能很好的处理带属性、状态一类的情况。

比如上面的对话框函数，如果要给对话框增加拖动的处理函数，还要在记录坐标、层级、打开状态等等属性时，需要手动从外部传入很多变量来处理。

导致原本是对话框相关的逻辑和数据，却被分散到了文件内的不同地方，需要做属性增减时很难集中调整。

继续增加函数形式的控制逻辑，也容易与其他函数混在一起。项目合作的同事稍不注意，就容易插入其他函数把它们打散。

最后赶出来的项目或许能正常运行，但内部代码却是互相穿插、混乱不堪的**意大利面条代码**，几乎无法维护。

所以计算机软件工程的前人们，探索出了**面向对象**的编程思想。

### 对话框类的定义

让我们从头想想，**对话框**是什么呢？

它应该是具有特定坐标、宽高、背景色等样式，可以设定其内容、坐标、控制按键等属性的绝对定位的特定元素。

那么有没有这样一种办法，使我们可以在需要使用对话框时，做到：

* 简单快速地创建对话框；
* 调用API就可以调整内容、移动、展示、收起对话框；
* 并且使不同对话框操作接口一致，自身数据却互不干扰；
* 有必要时，还可以在原有接口基础上快速增加新的特性呢？

刚才我们提到的这些，可以通过面向对象的**继承、封装**和**多态**来实现。

不过由于 JavaScript 的特殊性，**多态**在鸭子模式下的体现并不明显，暂且不提。先从一些基本概念开始说起。

上一步里，我们抽象出了对话框的基本概念，也就是我们需要的**对话框**大致上是个什么的东西。

运用面向对象的思想，我们可以把它们作为其成员属性、方法，来定义出一个**对话框类**。

为了方便新同学直接在浏览器里测试代码，这里采用 ES5 的类写法举例：

> 关于 JavaScript 的原型链和面相对象的关系，本文暂不深入说明，以免初学者混淆。
> 
> 大家可以先学会运用现有的方式，先知其然后知其所以然，通过实践记忆之后再深入了解原理也会更容易上手。

```javascript
// 对话框类的构造函数
function Dialog(options) {
  var self = this;
  // 创建相应 dom，记录到当前构建的对象内
  this.$dom = $('<div class="dialog-box">' + 
                '  <p class="dialog-msg"></p>' +
                '  <a class="dialog-btn" close-dialog>确定</a>' + 
                '</div>')
                .on('click', '[close-dialog]', function () {
                  self.hide();
                });
  $('body').append(this.$dom);
}

// 对话框的可用方法
Dialog.prototype = {
  // 记录构造函数
  constructor: Dialog,
  // 设置内容
  setContent: function (content) {
    this.$dom.find('.dialog-msg').text(content);
  },
  // 展示对话框
  show: function () {
    this.$dom.show();
  },
  // 收起对话框
  hide: function () {
    this.$dom.hide();
  }
};
```

首先声明对话框类 **Dialog** 的构造函数，之后每个对话框都将通过这个函数构建出具体的实例。

其中通过操作 **this**，可以使所有对话框都有 DOM 对象可供操作，且互相独立不受干扰（比如对话框1和对话框2都具有 **$dom** 的属性，修改对话框1的 **$dom** 时，对话框2的 **$dom** 不会受到任何影响，反之亦然）。

然后，增加了几个 **Dialog** 原型函数 **show, hide, destroy**。这几个函数被称为类的**方法**。

所有对话框都可以调用这些方法，与构造函数一样，其中也可以操作 **this** 来达成不同实例互不干扰的效果。

### 对话框实例

完成了最基本的可复用对话框类的创建，只需要通过 new 就可以实例化后使用了。

```javascript
// 创建对话框1
var dialog1 = new Dialog();
// 设置其内容
dialog1.setContent('登陆成功！');
// 展示对话框1
dialog1.show();

// 创建对话框2
var dialog2 = new Dialog();
// 设置其内容（不影响对话框1）
dialog2.setContent('评论发送失败！');
// 展示对话框2
dialog2.show();
// 3秒后自动收起（同样不影响对话框1）
setTimeout(function () {
  dialog2.hide();
}, 3000);
```

与 **showDialog** 函数相比，这样写有什么优势呢？
