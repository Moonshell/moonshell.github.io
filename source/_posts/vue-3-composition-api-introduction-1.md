---
title: 初探 Vue 3.0 的组装式 API（一）
date: 2020-08-30 17:30:00
categories: [前端]
tags: [Vue, Vue3, CompositionAPI]
toc: true
---

# （一）响应式数据

## 1. 简单例子

从最简单的数据绑定开始，在 Vue 2.0 中，我们这样将一个数据绑定到模板的指定位置：

在组件创建参数的 `data` 构造函数中返回一个用来绑定的数据对象，其中有个 `now` 字段，会被渲染到模板内的 `.app > p` 内。

```html
<template>
    <div class="app">
        <h1>Hello world!</h1>
        <p>Now is: {{now.toString()}}</p>
    </div>
</template>

<script>
// Vue 2.0
export default {
    data() {
        return {
            now: new Date(),
        };
    },
};
</script>
```

用 Vue3 的组装 API 实现的话，则是这样：

```javascript
// Vue 3.0
export default {
    setup() {
        return {
            now: new Date(),
        };
    },
};
```

## 2. 更新数据

奇怪，看起来好像没啥区别，只是把 `data` 改成了 `setup` 吗？

并不是，假如我们现在对这个 DEMO 做个小改动，让它每秒钟刷新一次时间，用 Vue2 大概是这样实现：

```javascript
// Vue 2.0
export default {
    data() {
        return {
            now: new Date(),
        };
    },
    mounted() {
        setInterval(() => this.now = new Date(), 1000);
    },
};
```

而 Vue3 的等效实现则为：

```javascript
// Vue 3.0
import { ref, onMounted } from 'vue';

export default {
    setup() {
        const now = ref(new Date());
        onMounted(() => {
            setInterval(() => now.value = new Date(), 1000);
        });
        return {
            now,
        };
    },
};
```

## 3. 对比分析

写了太多 Vue 的我们可能已经忘了，Vue2 的代码从标准 JS 模块的角度来看有多奇怪：

* `mounted` 中修改的 `this.now` 数据是在哪创建的？我们在模块 `default` 对象的成员里并没有找到对应字段，倒是在 `data` 内返回的另一个对象中有这个字段；
* 而 `data` 中返回的 `now` 也不是真正的 `this.now`，而是 `this.now` 的初始值，在 `data` 中 `setInterval` 修改 `now` 并不能更新渲染出来的时间；
* 如果想复用这个数据和它的更新逻辑，你必须将这样的结构单独写一份，然后通过特殊的 `mixin` 函数混入到当前组件的构造参数内。

这一切，是因为**整个模块 `default` 对象其实是 `vm` 对象的构造参数。其背后隐藏了对象的创建逻辑，在构造对象时构造参数中的一些不同层级的字段被绑定到了 `vm` 对象上**。

不少新手可能都犯过一个错误，在 `data` 中返回的数据字段和 `props`、`methods` 或者 `computed` 中的字段命名撞车（尤其是使用名为 `data` 的字段），在编码阶段并不能被 IDE 直接发现。就是因为上面的原因，这些字段创建时隶属于不同的位置，在之后构造时才被绑在了同一个对象上，导致了运行时才能发现的冲突。

Vue3 中，改成提供 `ref`、`reactive`、`toRef`、`onMounted` 等函数的形式实现，例子中：

* 在 `setup` 中看到的 `now` 即是用于绑定的 `this.now`；
* 修改 `now.value` 即可看到页面状态的更新；
* 如果要封装这份数据处理，只需要将 `now` 和 `onMounted` 处理提取到同一个函数内，再将 `now` 返回即可，不再需要黑盒的 `mixin` 处理。

可以说 Vue3 是直接将响应数据的创建决定权、生命周期的通知回调，都通过 API 的形式交给了开发者，更直观明了和可控。

## 4. API 说明

下面详细说说常用的几个响应式数据相关 API：`ref`, `reactive` 和 `toRefs`。

**(1) ref**

上面例子中使用到的 `ref`，可以将一个数据包装成响应式数据代理对象。

```javascript
const count = ref(0);
console.log(count.value); // => 0

count.value++;
console.log(count.value); // => 1
```

当你修改代理对象的 `count.value` 属性时，模板中使用到 `count` 的位置将响应数据的变化，更新视图中的数据状态。

**(2) reactive**

对于对象的响应式封装，使用 `ref` 稍显麻烦：

```javascript
const state = ref({
    count: 0,
});
console.log(state.value.count); // => 0

state.value.count++;
console.log(state.value.count); // => 1
```

这时可以改为使用 `reactive`，像操作普通对象的字段一样修改 `count` 即可更新视图：

```javascript
const state = reactive({
    count: 0,
});
console.log(state.count); // => 0

state.count++;
console.log(state.count); // => 1
```

对代理对象 `state` 添加新的字段也可触发视图更新。

**(3) toRefs**

有时候，对象的名字过长，我们想直接在模板内使用对象内部字段，直接使用解构是不行的：

```javascript
import { reactive } from 'vue';

export default {
    setup() {
        const position = reactive({
            x: 0,
            y: 0,
        });
        return {
            // 错误，解构出来的 x, y 并没有响应式代理。绑定到模板上后，数据变化无法触发视图更新
            ...position,
        };
    },
};
```

这个情况下，使用 `toRefs` 处理后再解构赋值即可：

```javascript
import { reactive, toRefs } from 'vue';

export default {
    setup() {
        const position = reactive({
            x: 0,
            y: 0,
        });
        return {
            ...toRefs(position),
        };
    },
};
```

但需要注意，`toRefs` 只处理调用时 `position` 的现有字段，如果在之后对 `position` 增加新字段，将无法触发视图更新。

<!-- 
## 2. 事件处理

## 3. 监听与计算属性

## 4. 子级数据传递
-->