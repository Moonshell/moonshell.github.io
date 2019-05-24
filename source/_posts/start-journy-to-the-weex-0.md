---
title: Weex 开发新手上路 - (1) 启程踩坑篇
date: 2019-03-22 00:00:00
categories: [node.js]
tags: [node.js, weex]
toc: true
---

最近，面对产品经理们越来越任性的需求、测试同学越来越火眼睛睛的眼力，不由得感觉到：WebView 是有极限的——该试试更贴近原生的框架了！ 

什么是 Weex ？官方文档中的定义是 **“Weex 是使用流行的 Web 开发体验来开发高性能原生应用的框架”**。

关于如何使用 Weex，官方文档中已经有步骤说明。但你按照文档开始尝试时，还是可能会遇到一些没提及的问题。

这里对之前遇到的问题和解决方法做个记录，大家如果遇到类似情况可以参考处理。

<!-- more -->

基本流程请阅读 [Weex 官方文档](https://weex.apache.org/zh/guide/introduction.html)，这里不再赘述。

# SDK 安装失败

开始的第一个问题：全局安装 weex-toolkit 失败。

搜索参考 [Global installs (sudo npm i -g) fail on Mac after 6.5 upgrade. Works fine after 6.4.1 downgrade.](https://npm.community/t/global-installs-sudo-npm-i-g-fail-on-mac-after-6-5-upgrade-works-fine-after-6-4-1-downgrade/4082)

通过修复以前安装的模块和 cache 权限解决，执行以下命令（请确认在自己的个人电脑环境下执行）：

```shell
sudo chown -R $(whoami) ~/.npm
sudo chown -R $(whoami) /usr/local/lib
sudo chown -R $(whoami) /usr/local/bin
```

将配置修复后，终于顺利安装了 weex-toolkit。

# 启动 iPhone 调试失败
 
一路继续创建项目、运行 Web 版 demo、加入 iOS 平台支持、执行 `weex run ios` 启动 iOS 调试——

新问题出现了。

如果没安装 Xcode 的话，这一步会提示 ios-deploy 安装失败：

> npm ERR! ios-deploy@1.9.4 preinstall: `./src/scripts/check_reqs.js && xcodebuild`

这个情况的话，只要从 AppStore 安装 Xcode 即可。

首次启动点击确认同意协议、确定安装相关开发组件，再次尝试执行 `weex run ios`，就不会出现刚才的问题了。

# CocoaPods 的安装

这次 ios-deploy 安装成功了，但之后马上发现还是无法启动 iOS 调试：

> Command failed: pod update
> 
> /bin/sh: pod: command not found

搜索错误关键字 `pos update`，得知需要安装 cocoapods，似乎是一个 iOS 的第三方开源组件库管理器。

执行 `sudo gem install cocoapods`，等半天后提示从 https://ruby.taobao.org 下载失败。

然后发现，这个站点居然停止维护了: [taobao Gems 源已停止维护，现由 ruby-china 提供镜像服务](https://www.oschina.net/news/71749/taobao-gems-ruby-china)

于是更换源，网上搜了几个，换了半天都没用。

怀疑是公司的内网代理问题，但 proxifier 里并没有反应，终端里输出 `$http_proxy` 也是正确的代理地址。

最后通过手动添加 `--http-proxy` 参数，终于安装成功了……

各种常用工具的配置方法: [设置 git/npm/bower/pip/gem镜像或代理](https://cloud.tencent.com/developer/article/1114138)

# 安装完毕

经过半天的折腾，环境终于安装完毕，可以在 iPhone 模拟器内启动项目了。

