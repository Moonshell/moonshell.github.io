---
title: 用 PhantomJS 让邮件报表图文并茂（二）完善篇
date: 2018-8-24 00:00:00
categories: [前端]
tags: [前端, PhantomJS, 邮件]
toc: true
---

根据上一篇文章已经可以实现报表邮件系统的初步 DEMO 了，但其实只是提供了基本的思路。

实际部署过程中，还会遇到各种各样问题，我在这一篇内一起记录下来，希望能对大家真正地有所帮助。

<!-- more -->

## 邮件页面布局兼容性

在你写完邮件，把它发送到邮件客户端内查看的时候，可能会被邮件客户端里的效果给吓一跳。

明明在浏览器里好好的，怎么在邮件客户端里就不能看了？不同客户端里效果还各不相同。

主要原因在于普通网页是由现代浏览器内核的标准来进行加载、解析和渲染的，邮件客户端内的 HTML 渲染内核实现则较为古老，缺少更新。

而且邮件客户端为确保邮件内容的安全性，会对邮件 HTML 进行预处理，移除修改标签、样式表、脚本，甚至阻止外部引用文件的加载。

常见的问题有非内联样式失效、边距失效、图片宽度异常、背景图片失效等等。

基本几个原则如下：

1. 不要写 style 标签，不要用 class 样式，只用元素的样式属性和内联样式；

2. 尽量不用 margin 和 padding。使用 table 布局，通过空行、空列拉开间距效果会更为稳定。所有 table 不要并列排列，尽量嵌套在一个 table 里；

3. 不要使用 float/position:absolute/position:fixed 等方式定位；

4. 图片必须指定宽高和 alt 属性，指定 style="display:block;border:none;margin:0;padding:0;" 等基本样式；

5. 不要简写色值，比如 #FFF，请使用完整的的 #FFFFFF；

想了解得更具体的话，可以阅读参考这几篇文章：

> [Html Email 邮件html页编写指南](https://www.cnblogs.com/lhweb15/p/6404626.html)
>
> [兼容性良好的HTML邮件(EDM)](https://swordair.com/principles-on-good-compatibility-html-mail-edm/)
>
> [HTML 邮件兼容问题与解决方案](https://segmentfault.com/a/1190000008864116)
>
> [HTML Email Templates Getting Started Guide](https://templates.mailchimp.com/)

## 测试工具

重构邮件页面，预览和调试页面效果时，可以使用 [Thunderbird](https://www.thunderbird.net/zh-CN) 向自己的邮箱发送测试邮件。

再使用 [Testi@](https://testi.at) 模拟各类常见邮件客户端和设备，预览邮件效果。

## Windows 下加载页面失败

在 Windows 环境下开发测试的同学可能会碰到的一个问题。

假设本地待处理的网页路径为 **D:\\test-mail\\index.html**，使用的 PhantomJS 版本为 2.1.1。

使用 `page.open(url, callback)` 打开页面时，如果 `url` 为 `fs.absolute(relativePath)` 处理出来的绝对路径，即 **D:\\test-mail\\index.html** 或  **D:/test-mail/index.html**，将会打开失败，回调函数获取的 `status` 为 **fail**。

若添加 **file://** 协议头，如 **file://D:/test-mail/index.html**，将能够打开页面，但无法正常加载页面内容，也无法完成截图。

Linux 和 Mac OS 内，打开 `fs.absolute()` 处理出来的路径是不会有这个问题的。

最后发现，Windows 环境下，改用相对路径处理即可……

所以，保证 PhantomJS 脚本和页面在附近目录，进行开发测试即可。

部署到服务器上时，定时任务执行目录再指定为绝对路径，避免出现意外，找不到文件的情况。

## 字体渲染问题

当你在本地完成上述邮件测试 DEMO，部署到 linux 服务器上后，可能会发现，图表里的文字全都失踪了。

这个问题一般是因为 ECharts 未指定使用的字体时，会根据国情默认采用微软雅黑字体。

而 linux 服务器上没有这个字体文件，就导致 canvas 内文字无法正常渲染了。

解决办法也不复杂，在服务器上添加相应字体就好。

首先获得一套“微软雅黑”字体文件（本地路径为 **C:\Windows\Fonts**，两个文件：msyh.ttf 普通、msyhbd.ttf 加粗）

在服务器的 **/usr/share/fonts** 目录下建立一个子目录，例如 **win**，命令如下：

```shell
mkdir /usr/share/fonts/win
```

将 msyh.ttf 和 msyhbd.ttf 复制到该目录下，例如这两个文件放在 **/root/Desktop** 下，使用命令：

```shell
cd /root/Desktop
cp msyh.ttf msyhbd.ttf /usr/share/fonts/win/
```

建立字体索引信息，更新字体缓存：

```shell
cd /usr/share/fonts/win
mkfontscale
mkfontdir
fc-cache
```

字体安装完毕后，再运行测试 DEMO 就 OK 了。

## 高分屏适配

将图表截图，发送邮件到手机上查看后，可能会发现图表的截图在高分屏上的显示效果很不理想。

{% img "/images/colorful-chart-mail-report-with-phantomjs/font-compare.png" popup %}

这个问题在 Mac 电脑和手机端的屏幕上，看起来会相当明显。

那么如何截取更清晰的图片素材呢？

首先，将 PhantomJS 的 page 对象的 `zoomFactor` 属性设为 2。

这个参数相当于浏览器内，通过 **Ctrl + 鼠标滚轮** 操作将页面放大为 200% 的视图，确保截图能截出两倍的尺寸大小。

同时，为页面内的 `window.devicePixelRatio` 设置为 2，这里是为了让 ECharts 在 Canvas 内绘制两倍像素的图表，否则截取出来的 Canvas 仍是模糊不清的大图而已。

最后，将检测到的 Canvas 坐标和尺寸乘以 2，就能截取出需要的两倍像素高清素材了。

如果需要截取 3 倍的，修改相关参数即可。

## Headless Chrome

就在我们完成这个项目，部署运行的第二天，我们看到了一条新闻：**PhantomJS 开发者宣布“中止”项目开发**。

现有的项目仍然是可以运行的，但若是日后发现其它问题，或者有严重漏洞的情况，可能还是需要使用新的方案来替代它。

通过网上搜索这个话题，发现一个很好的替代方案——谷歌推出的 Puppeteer。

可以在 node.js 环境下很方便的调用 Headless 也就是无 UI 的 Chrome。

然后通过调用它提供的 API，就能实现相同的图表截图需求。不过需要对原有的截图脚本进行相应改动和调整。

对于高分屏的截图方案甚至更简单，直接调用 `page.setViewport` 或 `page.emulate` 模拟高分屏的 **viewport** 即可。

如果页面内有 **alert** 可能会阻塞页面加载，导致无法完成截图操作。

这个情况一般进行监听后，对 **dialog** 做关闭操作即可：

```javascript
page.on('dialog', async dialog => {
    console.log('Dialog:\n', dialog.message());
    await dialog.dismiss();
});
```

其它具体 API 和操作方法，参考官方示例和文档即可：[Puppeteer](https://www.npmjs.com/package/puppeteer)

