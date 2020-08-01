---
title: Vue 3.0 组装API初体验
date: 2020-08-01 18:00:00
categories: [前端]
tags: [Vue, Vue3, CompositionAPI]
toc: true
---

## 1. 数据响应与事件绑定

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

每秒钟更新一次时间：

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

```javascript
// Vue 3.0
import { ref } from 'vue';

export default {
    setup() {
        const now = ref(new Date());
        setInterval(() => now.value = new Date(), 1000);
        return {
            now,
        };
    },
};
```
