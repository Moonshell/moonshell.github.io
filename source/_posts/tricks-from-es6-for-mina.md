---
title: 在你开发微信小程序时能用上的那些ES6特性
date: 2017-02-27 00:00:00
categories: [小程序]
tags: [微信小程序, ES6]
toc: true
---

微信小程序的官方开发工具中，已经集成了 `babel` 插件对 `ES6` 语法进行转换，各种第三方工具自然更少不了了。

所以可以放心的尝试使用 `ES6`，体验新标准带来的各种便利之处，省下时间后学习充电，或者早点下班、锻炼身体、下厨做个菜，调节生活又放松身心，岂不美哉？

那么，在小程序开发的过程中，有哪些 `ES6` 特性是可以给我们带来便利，提高开发效率的呢？这边就结合实例，一一来说一说吧。

<!-- more -->

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
        list: list.filter(helpers.hasTime)  // 筛选掉没有时间戳字段的数据
                  .map(helpers.parseTime)   // 将时间戳字段转化为 JS 的 Date 对象
    });
});
```

## 3. Rest 解构赋值

直到写这篇文章的时候，小程序官方的组件标准也仍然没有出来。

目前的通常处理方案，一般是通过 `template` 配合解构赋值不同对象的数据，实现组件各自状态、事件处理函数互相独立的效果。

如，有两个 `template` 都从 `data` 中绑定数据。

```html
<template name="banner">
    <view class="banner-wrap">
        <view wx:for="{{data}}" class="banner-item">
        <!--...-->
        </view>
    </view>
</template>
<template name="comic-list">
    <view class="comic-list">
        <view wx:for="{{data}}" class="comic-item">
        <!--...-->
        </view>
    </view>
</template>
```

`AppService` 中对于这两个模板创建两个不同对象，即可管理自身状态，不用担心字段名重复的问题。

```javascript
Page({
    onLoad: function () {
        // 加载 Banner 数据并显示
        this.loadData('/bannerState/get', (data) => {
            this.setData({
                bannerState: data
            });
        });
        // 加载 ComicList 数据并显示
        this.loadData('/comicListState/get', (data) => {
            this.setData({
                comicListState: data
            });
        });
    },
    loadData: function (url, callback) {
        var data = [];
        /* 从 url 加载数据的逻辑 */
        setTimeout(() => {
            callback({
                data: data
            });
        }, 100);
    }
});
```

页面内渲染模板时，对 `bannerState` 和 `comicListState` 字段进行解构即可。

```html
<template is="banner" data="{{...bannerState}}"/>
<template is="comicList" data="{{...comicListState}}"/>
```

这是个非常重要且实用的特性，基于这个基础可以封装出有具有通用逻辑的基类，实现模块内部的逻辑闭环，达到组件化开发的效果。

## 4. 增强的对象字面量

### `setData()` 

`setData()` 中的数据字段名与变量名一致时，不需要重复写两遍，上面加载数据的代码就可以这样简写：

```javascript
        this.loadData('/bannerState/get', (bannerState) => {
            this.setData({
                bannerState
            });
        });
```

数据字段较多时，效率会快很多。减少了整理和重构代码需要调整的地方，降低维护成本。

```javascript
// 传统对象字面量
this.setData({
    data1: data1,
    data2: data2,
    data3: data3,
    data4: data4,
    data5: data5
});

// 增强的对象字面量
this.setData({data1, data2, data3, data4, data5});
```

### 成员方法

增强的对象字面量写法，还包括函数的简写：

```javascript
// 传统的对象字面量
var comicState = {
    onTap: function (e) {
        // ...
    },
    onScroll: function (e) {
        // ...
    }
};

// 增强的对象字面量
var comicState = {
    onTap(e) {
        // ...
    },
    onScroll(e) {
        // ...
    }
};
```

这种简洁的成员函数写法，是不是很像 `class` 中的函数声明？

```javascript
class ComicState {
    onTap (e) {
        // ...
    }
    onScroll (e) {
        // ...
    }
}
```

## 5. Class 与继承

使用 `ES5` 的 `prototype` 写法，实现简单的类继承也没太大问题，但涉及到父类函数调用等情况，代码耦合度会变得更高，需要一定经验才能写出方便维护的代码。

通过 `ES6` 语法来实现类继承的话，有了统一的标准，写出的类继承更加直观，更方便调整。

```javascript
class Base {
    constructor (options, otherArg) {
        // Do something.
    }
}

class ChildType extends Base {
    constructor (options) {
        super(options, ChildType);
        // Do something.
    }
}
```

## 6. 块作用域变量

使用 `for` 对数据做迭代遍历时，语句中声明的 `var` 型变量名作用域其实提升到了函数顶部，不同迭代间忘记处理的话，可能会导致数据污染。

改为使用 `ES6` 的 `let`/`const` 可避免这一情况，放心使用块级作用域。

```javascript
for (let k in data1) {
    // ...
}
// ...
let k = 0;
// ...
for (let k in data2) {
    // ...
}
```

## 补充：

微信小程序使用的 `babel` 启用的转码规则可能不是最新的，截止目前版本，测试使用以下 `ES6` 会有问题，需要注意。

* `class` 内部声明的静态字段；

```javascript
// 以下代码在 babel 的 repl 中能正常处理，在小程序开发工具内会报错
class TestClass {
    static MODE = {
        NORMAL: 1,
        DISABLED: -1
    }
}

// 输出：1
console.log(TestClass.MODE.NORMAL);
```

* ~~`for...of` 语法遍历对象（直接使用了 `Symbol.iterator`，移动端可能尚未实现）；~~ 

> 20170329 更新：新版本开发工具似乎已经完善了这个问题，可以使用下面的 `ES6` 写法了：

```javascript
function Test() {
    this.a = 1;
    this.b = 2;
}
Test.prototype = {
    c: 3
};

var o = new Test();

// ES5
for (var k in o) {
    if (k.hasOwnProperty(k)) {
        console.log(o[k]);
    }
}   // 输出：1 2

// ES6
for (let k of Object.keys(o)) {
    console.log(o[k]);
}   // 输出：1 2
```

`for...of` 用于数组遍历时，效果与 `Array.prototype.forEach` 类似，区别是可以在途中 `break` 中断循环，无需每次遍历整个数组。

```javascript
// Array.prototype.forEach
comicList.forEach((c) => {
    // ...
});

// for...of
for (let c of comicList) {
    // ...
}
```