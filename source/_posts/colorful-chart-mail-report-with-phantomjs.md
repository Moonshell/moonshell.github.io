---
title: 用 PhantomJS 让邮件报表图文并茂
date: 2018-3-28 00:00:00
categories: [前端]
tags: [前端, PhantomJS, 邮件]
toc: true
---

在部门日常业务中，每天都会产生各种各样的数据。为了让抽象的数据，更加调理方便人阅读，就需要将数据整理成表格、图表等形式，以更生动的面貌展示在人们眼前。

通常 Web 端可以采用 ECharts 等方案来实现丰富的图表效果，但报表邮件由于各种邮件客户端环境的关系，虽然是使用 HTML 编写邮件内容，可用的样式、布局都有会诸多限制，甚至不允许执行 JavaScript 脚本。

传统报表邮件中，只能以简单的 table 表格来展示数据，一但数据维度增加、业务日渐复杂，报表邮件将变得越来越冗杂、难以理解。

那么有没有什么办法，让邮件也能实现图文并茂的图表呢？

## 将图表转换为图片

虽然邮件不支持脚本生成的 canvas 图表，但却是支持图片展示的。

那么只要能将图表截取为图片添加回邮件内，就能在邮件客户端里看到了，这就是我们要做的第一步。canvas 已经提供了 toDataURL 的方法，可以将画布内容转换成 img 能显示的 DataURL。

所以将网页内的 canvas 内容都提取出来，放到相同大小的 img 标签内，替换掉原本文档流中的 canvas，这样在邮件客户端内就能看到图表内容了吧？

```javascript
document.querySelectorAll('canvas')
    .forEach(function () {
        var img = document.createElement('img');
        img.className = canvas.className;
        // img.style.cssText = canvas.style.cssText;
        var cssText = canvas.getAttribute('style');
        img.setAttribute('style', cssText);
        img.width = canvas.width;
        img.height = canvas.height;

        img.src = canvas.toDataURL();

        canvas.parentElement.insertBefore(img, canvas);
        canvas.parentElement.removeChild(canvas);
    });
```

打开图表页面，在控制台执行这段脚本后，页面里的所有 canvas 都被替换成了静态截图。

需要注意，很多图表库可能会有“启动/展开动画”，在这段动画完成前截图，得到的并不是我们想要的效果。

所以还需要给这段截图脚本加个延时处理，在所有图表完全展示后在进行截图。我们一般简单设定个2-3秒即可。

对于一些具有交互效果的图表（如鼠标 hover 时展示数值），由于变成了静态图，这些交互都会消失。

所以一些关键数据，需要改为默认显示，不需要通过交互触发，以便脚本截图时能截取到。

## 自动化任务实现

基本思路出来了，那么如何把它运用在我们生成报表邮件的服务器上呢？

