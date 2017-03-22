---
title: 在你开发微信小程序时能用上的那些ES6特性
categories:
- 小程序
tags:
- 微信小程序
- ES6
---

微信小程序的官方开发工具中，已经集成了 `babel` 插件对 `ES6` 语法进行转换，各种第三方工具自然更少不了了。

所以可以放心的尝试使用 `ES6`，体验新标准带来的各种便利之处，省下时间后学习充电，或者早点下班、锻炼身体、下厨做个菜，调节生活又放松身心，岂不美哉？

那么，在小程序开发的过程中，有哪些 `ES6` 特性是可以给我们带来便利，提高开发效率的呢？这边就结合实例，一一来说一说吧。

## 1. 箭头表达式

做前端开发的，开始阶段基本会遇到 `this` 与 *闭包* 带来的坑————一些异步操作中，回调函数中丢失了当前函数的上下文对象，导致异步操作完成后，更新原有上下文失败。

为了避免这个问题，以前大家都是自己用变量保存一个闭包外部上下文的引用，取的名字可能千奇百怪：
`that`/`_this`/`$this`/`self`...在异步操作完成后的回调中，通过调取这个闭包外层的变量，达到更新回调前函数上下文对象的目的。

`ES6` 中增加了 *箭头表达式*，效果和匿名函数相似，但箭头表达式更为简练，且内部执行时的 `this` 与外侧一致，不再需要每次都额外增加变量引用了。

微信小程序里，对每个页面编写的代码逻辑，都作为生命周期钩子函数（如：onLoad, onShow, onUnload）和自定义函数（如：各类组件回调函数）写在 `AppService` 内。

这两种函数内，`this` 都指向当前 `Page` 对象，在这些函数里做的各种异步操作，回调内的 `this` 基本都应该仍然保持为当前 `Page` 对象。在这个情况下，使用箭头表达式可以减少重复的工作、也减少遗漏 `this` 时出错的几率。

```javascript
// ES5
var net = require('../public/net');
Page({
    data: {
        list: []
    },
    onShow: function () {
        var self = this;
        net.get('/Index/getList', function (res) {
            res = res || {};
            var status = res.status,
                data = res.data,
                list = data.list;
            if(status == 2) {
                self.setData({list: list});
            }
        });
    }
});

// ES6
import * as net from '../public/net';
Page({
    data: {
        list: []
    },
    onShow: function () {
        net.get('/Index/getList', (res = {}) => {
            let {status, data: {list}} = res;
            if (status == 2) {
                // 此处 this 的指向与 onShow 内一致，无需增加 self 变量
                this.setData({list});
            }
        });
    }
});
```

## 2. 数组方法

虽然都说微信小程序 `wxml` 的 `Mustache` 语法与 `Vue.js` 很相似。但据说是为了分离 `UI` 线程和 `AppService` 线程，微信小程序暂时并不支持 `&#123;&#123;value | filter}}` 的写法。

这时候可以借助于 `ES5` 中为数组对象增加的方法，之前因为浏览器兼容性问题，不一定全部能用。如今在移动端了，就尽情用起来吧：

输出数据前，对后台传来的列表数据做一些预处理后再显示时，通常使用 `Array.prototype.forEach` 和 `Array.prototype.map` 进行相应处理；
筛选掉无效数据，可以使用 `Array.prototype.filter`。

```javascript
// js
var helpers = {
    // 判断是否有时间参数
    hasTime: (i) => {
        return !isNaN(parseInt(i.stamp));
    },
    // 时间转换处理
    parseTime: (i) => {
        i.time = new Date(i.stamp + '000');
        return i;
    }
};

net.get('/Index/getList', (res = {}) => {
    let {status, data: {list}} = res;
    this.set({
        list: list.filter(helpers.hasTime)  // 去掉没有时间戳字段的数据
                  .map(helpers.parseTime)   // 将时间戳字段转化为 JS 的 Date 对象
    });
});
```

## 3. Rest 解构赋值

截止我写这篇文章的时候，小程序官方的组件标准仍然没有出来。

通常方案是，通过 `template` 配合解构赋值不同对象的数据，实现组件各自状态独立的效果。

## 4. 增强的对象字面量

* `setData()` 中的字段名

* 为成员添加方法

## 5. Class 与继承

```javascript
class Base {
    constructor (options, otherArg) {
        // Do something.
    }
}

class ChildType {
    constructor (options) {
        super(options, ChildType);
        // Do something.
    }
}
```

## 6. 其它

字符串模板、`let` 与 `const`。

## 补充：

微信小程序使用的 `babel` 启用的转码规则可能不是最新的，截止目前版本，测试使用以下 `ES6` 会有问题，需要注意。

* Class 内部的字段；
* for...in 语法遍历对象；
* 对象字面量中，函数间逗号问题；