---
title: 那些你不知道的 node.js 桌面应用开发框架
categories:
- node.js
tags:
- node.js
- 桌面应用
- 感想
toc: true
---

> 说到 `node.js` 的 GUI 开发方案，首先想到的就是 `electron` 和 `nw.js`。但除了它们之外，是否存在其它更轻量级的技术方案可供选择呢？

<!-- more -->

这两天，翻出了几年前在校时用 `winform` 写的小工具，发现虽然能使用，部分功能却是已经需要改进了。

工具的源码已经丢失，现在用的电脑里也没有再安装 `winform` 相关的开发环境。于是决定使用 `node.js` 重写工具。

估算一下，重写的核心代码大概也就一、两KB，毕竟只是每次打开后只用几分钟的小工具，用来爬一爬网站内容之类的需求，业务逻辑挺简单的。

在这个基础上，再加上个方便操作的 GUI 就好了，于是首先想到的方案自然就是 `electron` 或者 `nw.js`。

![electron](https://camo.githubusercontent.com/11e7cfd04eceb1ea7464e99edda0e7000487f343/68747470733a2f2f656c656374726f6e2e61746f6d2e696f2f696d616765732f656c656374726f6e2d6c6f676f2e737667)

但打包了 `Chromium` 内核和 `node.js` 环境后，如今的 `electron` 和 `nw.js` 动不动就上百MB的大小，用来开发这样的小工具实在是不划算。

而且在自己另一台破电脑上，`electron` 启动时间动不动就是十几秒，操作响应也不是很灵敏，达不到小而快的目标效果。

除了它们之外，是否还有其它的 `node.js` GUI 开发方案呢？

于是找了找其它 `node.js` 的 GUI 框架，大家有兴趣的话可以关注一下：

## `positron` 

`positron` 兼容于 `electron`，只是 Web 内核方面，把 `Chromium` 换成了 `Gecko`。

目前此项目已停止开发。

项目地址：[https://github.com/mozilla/positron](https://github.com/mozilla/positron)

## `libui-node`

![libui-node](https://github.com/parro-it/libui-node/raw/master/docs/media/Window-macOS.png)

`libui-node` 提供了简单便携的基于 C 的 GUI 库，目标是成为相对于 `electron` 之外跨平台 GUI 开发的轻量级选择。

跑了一下 DEMO，组件反馈虽然略有迟缓，但可用组件看起来还挺丰富的。

不过目前整个项目大小有上百MB，暂时似乎没有完善的部署发布方案，希望之后会日渐完善吧。

项目地址：[https://github.com/parro-it/libui-node](https://github.com/parro-it/libui-node)

## `react-x11`

![react-x11](https://cloud.githubusercontent.com/assets/173025/24536323/6af97598-1625-11e7-88d4-74f429b7f470.gif)

`react-x11` 刚刚起步，目前可用组件只有 `window`，且需要运行在 X Window 环境下（通常是 linux 桌面环境，或者 osx + `XQuattz`）。

项目地址：[https://github.com/sidorares/react-x11](https://github.com/sidorares/react-x11)

## `node-qt`

![node-qt](https://github.com/arturadib/node-qt/raw/master/examples/helloworld.png)

`node-qt` 以 `node.js` 附件的形式提供了 Qt 库的原生绑定。

但在我的 Windows 环境下似乎水土不服，安装失败（貌似是需要 `MSVC++` 而不是 `VS2013` 的缘故）。

项目地址：[https://github.com/yue/node-gui](https://github.com/yue/node-gui)

## `reactXP`

`reactXP` 是由微软 skype 团队近期推出的跨平台开发框架（`XP` = `Cross Platform`）。支持安卓、iOS、桌面等多平台。

看起来具体实现基本就是把 `react-native`、`electron` 等方案整个打包，再增加了对 Win10 的 UWP 支持，是个大而全而非小而轻的方案？

项目地址：[https://github.com/Microsoft/reactxp](https://github.com/Microsoft/reactxp)

---

还有一些不成熟或是有问题的方案，在这就没有列举出来了。

其他语言方面，尝试了 `python` 的 `Tkinter`。发现挺方便的，安装完 `Python` 就能使用，但能实现的效果似乎很有限；`PyQt` 更强大、美观，但需要花时间去学习 QT，没法现在就立刻动手做。

而且两者都不如类 Web 的 GUI 方案来的灵活便捷（`React` 也算此类）。

这么说来，近几年桌面开发似乎越来越不温不火，大家的关注中心似乎都转移到了移动端上。

而移动端的话，原生开发方面，从传统原生开发方式与 `react-native` 的出现、苹果推出 `Swift`，到最近谷歌钦定 `Kotlin` 作为安卓开发的一级语言；Web 前端开发方面，`node.js` 带来的 `grunt`、`gulp`、`webpack` 工作流，以及各种优秀 MVVM 框架竞相出现，新技术不断出现，让前端开发成为了众所周知最爱造轮子的一拨人。

移动端涌现的各种新工具、框架、语言，对行业入门者来说有一定学习成本，但也确实带来了种种便利。它们一般都解决了各种以往开发方式中的痛点

对于日渐变化的移动端开发需求，确实提高了生产效率，改善开发体验。但新出现的方案毕竟需要在实践中逐步完善，所以它们每天都在不断迭代更新，甚至又出现更新的其它方案。

或许可以这么说，我们现在正经历着桌面端开发向移动端转移中心的过渡期。桌面端 GUI 开发的需求存在感日渐稀薄，现有传统开发方案已能应对日常需求，所以虽然也有一些技术痛点，却并没有更新开发方式的必要。

## 后记

这次的小工具的开发，最后采用的形式是编写好关键的 JS 脚本，加入 `Greasemonkey` 后直接在浏览器内执行，抓取需要的内容。

除此之外，做成 `Chrome` 插件，或者直接写成命令行工具，也是可选的轻量级方案。

