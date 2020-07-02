---
title: 让 WebStorm 自动识别 Webpack 的 alias 配置
date: 2020-5-24 14:20:00
categories: [前端]
tags: [前端, Webpack, WebStorm]
# toc: true
---

我们都经常遇到这个情况，在 Web 项目目录结构划分得细致之后，从某个子组件引用公共模块时，如果使用准确的相对路径，路径可能会变得相当长：

<!-- more -->

```
├── common
│   └── net.js
└── pages
    ├── index
    │   └── top-bar.vue
    └── index.vue
```

此时，如果在 **top-bar.vue** 里引用 **net.js**，路径将会是这样：

```
import * as net from '../../common/net.js'
```

某些情况还需要写好几个 `../`，而且一旦某些逻辑分离成单独组件时，放在了不同层级深度的目录里，这些路径代码就得一一修复引用。

所以我们经常在 Webpack 里做类似这样的配置：

```
// webpack.config.js
module.exports = {
    // ...
    resolve: {
        alias: {
            '@': require('path').join(__dirname, './src'),
        }
    },
    // ...
};
```

然后不管需要用到公共组件的文件在哪个层级，都可以直接写成：

```
import * as net from '@/common/net.js'
```

## 问题

虽然这么写是能通过编译了，但是编码过程中又会发现一些体验上的不便。

在 IDE 中通过准确的路径引用的文件，可以提供便捷的定义跳转、函数提示、自动完成等功能。

而通过别名引用的文件，IDE 似乎就爱莫能助了，按住 ctrl/cmd 看不见跳转链接、写出函数名的前几个字母也不会出现智能提示、对于公用组件的函数 Js Doc 也无法直接看到。

这都 2020 年了，难道没有 IDE 支持常用前端项目结构的 alias 路径解析吗？

答案是有的，WebStorm 里就提供了 Webpack 配置文件的 alias 路径解析。

但是有人可能和我一样，虽然写了 alias，而且确实是官方的语法。但是 WebStorm 并没有对应的提示，那么是哪里出了问题呢？

## 定位

为了定位问题，我先创建一个最基础的 Webpack 项目，然后通过 WebStorm 打开，发现 alias 里的路径全都能正常解析。

并没有什么特殊字符或者目录层级的问题，使用 `@`、`@@`、`{SRC}` 等命名都是可以正常识别和提示的。

但是完全相同的配置，在我的另一个旧项目里就无法识别了。

这个现有项目相比基础的项目，多了构建环境区分、多页面入口检测、各类资源 loader、后置服务器环境配置任务等很多内容，一一排除的话工作量有点大。

这时，突然发现每次修改 **webpack.conf.js** 后 WebStorm 的输出窗口里是有相应提示的。只不过对于解析失败的情况，给出的错误信息非常模糊，只说是一个 `default` 关键字不存在的异常。

看到 `default` 首先想到的是 ES6 模块的默认输出对象，但是项目配置是用 CommonJS 写的，并没有使用 `export default`。倒是根据启动时设定的环境变量，在入口 **webpack.config.js** 内通过 `switch` 引入了不同的任务配置（**development**/**production**），而这个 `switch` 里没有编写 `default` 的处理。

补充了 `default` 的情况后，WebStorm 的输出窗口里不再提示 `default` 异常，但还是提示了另一个错误。不过从错误信息的变化看来，WebStorm 对于 Webpack 配置文件的解析不像是静态解析，更可能是后台执行了一遍 **webpack.confi.js**，然后取了返回结果。

于是在 **webpack.config.js** 内，拼装配置的过程中，添加了一段代码，向当前项目目录内输出了一个临时文件：

```
require('fs').writeFileSync(__dirname + '/detect.log', 'Created:' + new Date());
```

如果 WebStorm 偷偷执行了配置脚本，这边也能通过是否出现 **detect.log** 发现它的踪迹。

果然，保存配置文件刚过了一会儿，并没有启动 Webpack 任务，项目目录中却出现了一个 **detect.log**。

## 解决

既然摸到了 WebStorm 检测的踪迹，接下来就可以开始顺着踪迹逐步定位问题了。

通过在配置文件不同位置打点输出到刚才的临时日志文件，就能定位到项目配置里到底是哪里影响了 WebStorm 的检测了。

这边主要是两个情况：一是项目中的附加参数为空时取不到对应配置；二是某些情况下通过 `realine` 让用户输入相关配置参数，在 WebStorm 检测时是超时无效的。

将 WebStorm 检测时的 `process.env` 打印到文件内，对比正常启动任务和 WebStorm 检测的不同环境变量，针对后台检测时做好跳过处理后，终于项目里也能正常检测到定义的 alias 了，问题解决。

如果大家在使用 WebStorm 的过程中，也遇到类似的问题，可以参考这个方案进行定位和解决问题。
