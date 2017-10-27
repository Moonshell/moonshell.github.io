---
title: 探索PHP与Vue通用直出模板方案
date: 2017-02-13 00:00:00
categories: [前端]
tags: [Vue, php, 模板, 直出, SSR]
toc: true
---

> 测试方案，欢迎提出新的思路，共同探讨！

{% img "/images/vue-php-template-convertor/cover-1.png" cover %}

<!-- more -->

## 什么是“页面直出”

我们通常说的“页面直出”，其实就是服务端渲染（SSR, Server-Side Render）。

最初的 JS SPA 方案有个常见的问题，就是脚本没有加载执行完时，页面中没有内容。不仅影响访问体验，还不利于 SEO。

于是大家要么使用传统的 JSP、PHP、ASP.NET 服务端页面模板，要么采用最新的 React/Vue 服务端渲染方案。

## PHP 与 React/Vue SSR

目前动漫 H5 主站的前端架构没有使用 MVVM，存在大量累赘的 DOM 操作代码。

于是考虑切换到 React/Vue 方案，通过双向绑定和组件开发，简化累赘代码，提高可维护性和测试可能性。

但是为了优化 SEO 效果，H5主站需要做页面直出，而常用的 React/Vue 直出都是基于 node.js 服务端的，我们现有的服务端环境是 PHP，并不能直接使用。

### 方案1：php-v8js

首先找到的是 php-v8j，也就是实现一个 PHP 服务端的 JS V8 运行环境。

但根据 Vue 作者的回复，Vue 依赖于一些第三方模块，以及使用了 node.js 的 stream 等功能，php-v8js 提供的环境并不能实现 Vue 的服务端直出。

> 参考地址，Vue作者对于 php 环境下 SSR 的回复：https://github.com/vuejs/vue/issues/4101

React 的话，有一篇文章，描述了通过 php-v8js 实现的 React SSR (Server-Side Render)。

> 参考地址，使用 react-php-v8js 实现 SSR：http://www.phpied.com/server-side-react-with-php/

目前看来，至少使用 React 的方案是可行的。

可是即便如此，Github 的 react-php-v8js 仍然是“实验性”的项目，php-v8js 的人气也不是很高，issue 中看来可能也会潜在不少问题。两者一起使用，风险较大。

并且 php-v8js 是在 PHP 内运行了个 v8 的沙盒运行环境，执行效率有待商榷。而 React/Vue 使用的虚拟 DOM 虽然在 v8 引擎内渲染速度不错，但相比传统字符串拼接的模板引擎仍然多了不少性能开销，React 很早实现了服务端渲染却没有铺开，便是出于对 node.js 服务端普及率和渲染性能的考虑。

### 方案2：mustache.php

通过搜索 php js template，发现 Mustache 其实已经实现了 [Ruby](https://github.com/mustache/mustache), [JavaScript](https://github.com/janl/mustache.js), [Python](https://github.com/defunkt/pystache),[Erlang](https://github.com/mojombo/mustache.erl), [node.js](https://github.com/raycmorgan/Mu), [PHP](https://github.com/bobthecow/mustache.php), [Perl](https://github.com/pvande/Template-Mustache), [Objective-C](https://github.com/groue/GRMustache), [Java](https://github.com/spullara/mustache.java), [C#/.NET](https://github.com/jdiamond/Nustache), [Android](https://github.com/samskivert/jmustache), C++,[Go](https://github.com/cbroglie/mustache), [Lua](https://github.com/Olivine-Labs/lustache), [ASP](https://github.com/nagaozen/asp-xtreme-evolution/blob/master/lib/axe/classes/Parsers/mustache.asp), [Io](https://github.com/mil/mustache.io), [Dart](https://github.com/valotas/mustache4dart), [Haxe](https://github.com/nadako/hxmustache),[Delphi](https://github.com/synopse/dmustache), [Swift](https://github.com/groue/GRMustache.swift), [Bash](https://github.com/tests-always-included/mo) 等各种平台和语言的支持。

> Mustache 主页: http://mustache.github.io/

其中 PHP 平台可以使用 mustache.php 作为模板引擎，进行服务端页面渲染。

> mustache.php: https://github.com/bobthecow/mustache.php

通过在服务器环境下加入 mustache.php，既可实现前端后台使用统一的 Mustach 语法模板渲染效果。

但是 Mustache 语法与 Vue.js 并不完全兼容（如循环、if 等写法），而 Mustache 本身只是单纯无逻辑的渲染模板，并不能满足我们 MVVM 改造的需求，所以是否使用 mustache.php 仍然有待考虑。

## 为什么需要直出？

回到开始的问题，为什么需要做页面直出呢？SEO 吗？

而为了 SEO 而需要直出的页面有哪些？

这些页面是否都是与用户个人状态无关，可以直接缓存的？

那这些页面使用 PHP 运行 php-v8js 跑出一遍结果后，进行页面缓存，其它页面直接使用前端 React + ajax 渲染数据。是否可行？

php-v8js 出现页面渲染意外的可能性多大？

提供异常时切换到普通方案是否可行？

## 思考解决方案

需要直出的页面一般与用户个人状态无关，可以在服务器端进行页面内容缓存，提高访问效率，利于 SEO。

为了服务端稳定，建议不建议使用重量级的服务端渲染库，尽可能减少现有系统的变动，避免运行中的系统出现异常。而出现异常时切换到普通方案的紧急方案也需要精力去实现和维护，成本略高。

## 换个思路，简化问题

以 Vue 为例，服务端渲染包括很多功能，涉及到 Vue 支持的各种 v-* 命令，需要对渲染后的页面中，各种数据状态、事件状态进行复杂的虚拟 DOM 关联处理，所以需要在 Node.js 环境中借助完整的 SSR 模块来渲染。

但是我们日常真的需要实现这些效果吗？如果切换技术方案的代价这么大，能否折衷一下，找个简单的替代方案？

结合 mustache.php 的思路，是否可以根据业务中直出的需求，使用一种简单的统一模板，让 Vue 和 php 都能支持渲染？

动漫业务中，需要直出的情况通常是输出漫画列表，将漫画信息展示出来，便于 SEO 和缓存。

对于这样的需求，我们可以在切图重构后，微调重构稿代码，将 Vue 挂载到页面内，展示出漫画列表：

``` html
  <ul class="comic-list" id="lst_comicList_1">
      <li v-cloak v-for="(id, item) in list"
          class="comic-item"
          @click="clickComicItem" 
          data-id="c_{{id}}">
          <span class="comic-title">{{item.title}}</span>
        	<span class="comic-desc">{{item.short_desc != null ? item.short_desc : item.brief_intrd}}</span>
      </li>
  </ul> 
	<script type="text/javascript">
  new Vue({
    el: '#lst_comicList_1',
    data: {
      list: {
        '531490': {title: '一人之下', short_desc: '身怀异术该何去何从', brief_intrd: '随着爷爷尸体被盗，神秘少女冯宝宝的造访，少年张楚岚的平静校园生活被彻底颠覆。急于解开爷爷和自身秘密的张楚岚和没有任何记忆“不死少女”冯宝宝开启了“异人”之旅……'},
        '537832': {title: '破晓世纪', brief_intrd: '一个是重度氪金的“非洲”少年林晓，一个是来历不明的神秘少女伊贝林。阴差阳错下两人签订契约，来到一个人类未曾探索过的宇宙。在这个荒诞而有趣的世界里，林晓是否能够摆脱他的“非洲人”属性，并且开辟属于自己的道路呢……'}
      }
    },
    method: {
      'clickComicItem': function (e) {
        var item = e.currentTarget,
            comicId = item.getAttribute('data-id');
        alert('Clicked comic: ' + comicId);
      }
    }
  });
	</script>
```

而目前使用 php 直出，页面模板代码需要稍作调整：

``` html
  <?php $list = array(
          "531490" => array("title" => "一人之下", "short_desc" => "身怀异术该何去何从", "brief_intrd" => "随着爷爷尸体被盗，神秘少女冯宝宝的造访，少年张楚岚的平静校园生活被彻底颠覆。急于解开爷爷和自身秘密的张楚岚和没有任何记忆“不死少女”冯宝宝开启了“异人”之旅……"),
          "537832" => array("title" => "破晓世纪", "brief_intrd" => "一个是重度氪金的“非洲”少年林晓，一个是来历不明的神秘少女伊贝林。阴差阳错下两人签订契约，来到一个人类未曾探索过的宇宙。
  在这个荒诞而有趣的世界里，林晓是否能够摆脱他的“非洲人”属性，并且开辟属于自己的道路呢……")
        ); ?>
  <ul class="comic-list" id="lst_comicList_1">
    <?php foreach ($list as $id => $item) { ?>
    <li class="comic-item"
        @click="clickComicItem"
        data-id="c_<?php echo $id; ?>">
      <span class="comic-title"><?php echo $item['title']; ?></span>
      <span class="comic-desc"><?php echo $item['short_desc'] != NULL ? $item['short_desc'] : $item['brief_intrd']; ?></span>
    </li>
    <?php }?>
  </ul>
	<script type="text/javascript">
  new Vue({
    el: '#lst_comicList_1',
    method: {
      'clickComicItem': function (e) {
        var item = e.currentTarget,
            comicId = item.getAttribute('data-id');
        alert('Clicked comic: ' + comicId);
      }
    }
  });
	</script>
```

经过对比后，发现如果稍作限制，只使用 php 与 javascript 通用的语法的话，可以通过简单地替换就将上面的模板转化为下面的效果。

主要需要处理的地方在于 Vue 模板中的 v-for 和 Mustache 输出标记。

所以，我们做出以下约束：

> 1.    只使用 v-for 对列表数据进行渲染，并且必需指定 (<key>, <value>) 两个字段名（暂不支持 v-if、v-else、v-else-if 的转换）。
> 2.    只允许渲染简单的 DOM 结构（用于 SEO 或缓存），不渲染 Vue 组件。
> 3.    只处理 &#123;&#123;}} 标记的 Mustache 输出语法，将其简单替换为 php 的 echo 函数，各种 v-bind、v-on、v-model 等指令中参数不会被处理（数据状态不同步）。
> 4.    因为是直接替换，php 中不支持的各种 js 运算仍然是不支持的（如：&#123;&#123;item['name'] || item['nick']}} 和 &#123;&#123;item['name'].join()}}）。
> 5.    需要拼接字符串时，请使用 &#123;&#123;id}}_&#123;&#123;item.text}} 的形式，不要使用 &#123;&#123;id + item.text}} 运算（PHP 中不能用 + 运算拼接字符串，会导致转换成整型后做加法）。
> 6.    如果不需要对渲染出的 DOM 数据做绑定或更新，只需要做简单的事件控制。可以直接默认保留服务端渲染的 DOM，使用 vue 对象的事件监听器即可。
> 7.    对于服务端渲染的 DOM，只能绑定监听器，无法在绑定属性内直接传参。如：<li v-on:click="clickItem(item.id)"/>，需要改为：<li v-on:click="clickItem" data-id="&#123;&#123;item.id}}"/>。事件监听器内读取 e.currentTarget 的 data-id 属性，作为点击判断的依据（不过 Vue 不推荐在 HTML 属性内使用 Mustache，如果有更好的方案欢迎提供思路）。
> 8.    如果要通过数据更新 DOM 或者做双向绑定时，需要给服务端渲染的元素增加 clear-before-render 属性。手动输出 json 数据到前端脚本，重新渲染 DOM 替代预渲染的占位 DOM（使用此属性的元素 v-if 会无效化）。

按照以上约束编写的前端模板，即可转换为 php 可用的模板。

于是根据这个思路，在团队日常使用的前端构建工具中，实现了这类脚本的转换构建任务。（日常使用的前端构建工具：[Front Custos GUI](https://github.com/krimeshu/front-custos-gui)）

在构建任务的帮助下，页面只需要编写如下的代码：

``` html
  <?php $list = array(
          "531490" => array("title" => "一人之下", "short_desc" => "身怀异术该何去何从", "brief_intrd" => "随着爷爷尸体被盗，神秘少女冯宝宝的造访，少年张楚岚的平静校园生活被彻底颠覆。急于解开爷爷和自身秘密的张楚岚和没有任何记忆“不死少女”冯宝宝开启了“异人”之旅……"),
          "537832" => array("title" => "破晓世纪", "brief_intrd" => "一个是重度氪金的“非洲”少年林晓，一个是来历不明的神秘少女伊贝林。阴差阳错下两人签订契约，来到一个人类未曾探索过的宇宙。
  在这个荒诞而有趣的世界里，林晓是否能够摆脱他的“非洲人”属性，并且开辟属于自己的道路呢……")
        ); ?>
  <ul class="comic-list" id="lst_comicList_1">
    <vue-php-ssr-template>
      <li v-cloak clear-before-render
          v-for="(id, item) in list"
          class="comic-item"
          @click="clickComicItem" 
          data-id="c_{{id}}">
          <span class="comic-title">{{item.title}}</span>
        	<span class="comic-desc">{{item.short_desc != null ? item.short_desc : item.brief_intrd}}</span>
      </li>
    </vue-php-ssr-template>
  </ul> 
	<script type="text/javascript">
  new Vue({
    el: '#lst_comicList_1',
    data: {
      list: <?php echo json_encode($list); ?>
    },
    method: {
      'clickComicItem': function (e) {
        var item = e.currentTarget,
            comicId = item.getAttribute('data-id');
        alert('Clicked comic: ' + comicId);
      }
    }
  });
	</script>
```

通过前端构建后，生成的代码如下：

``` html
  <?php $list = array(
          "531490" => array("title" => "一人之下", "short_desc" => "身怀异术该何去何从", "brief_intrd" => "随着爷爷尸体被盗，神秘少女冯宝宝的造访，少年张楚岚的平静校园生活被彻底颠覆。急于解开爷爷和自身秘密的张楚岚和没有任何记忆“不死少女”冯宝宝开启了“异人”之旅……"),
          "537832" => array("title" => "破晓世纪", "brief_intrd" => "一个是重度氪金的“非洲”少年林晓，一个是来历不明的神秘少女伊贝林。阴差阳错下两人签订契约，来到一个人类未曾探索过的宇宙。
  在这个荒诞而有趣的世界里，林晓是否能够摆脱他的“非洲人”属性，并且开辟属于自己的道路呢……")
        ); ?>
  <ul class="comic-list" id="lst_comicList_1">
    <!-- vue-php-ssr-template -->
      <?php foreach ($list as $id => $item) { ?>
        <li class="comic-item" 
            @click="clickComicItem" 
            data-id="c_<?php echo $id; ?>"
            v-if="false">
          <span class="comic-title"><?php echo $item['title']; ?></span>
        	<span class="comic-desc"><?php echo $item['short_desc'] != NULL ? $item['short_desc'] : $item['brief_intrd']; ?></span>
        </li>
      <?php } ?>
    
      <li v-cloak clear-before-render
          v-for="(id, item) in list"
          class="comic-item"
          @click="clickComicItem" 
          data-id="c_{{id}}">
          <span class="comic-title">{{item.title}}</span>
          <span class="comic-desc">{{item.short_desc != null ? item.short_desc : item.brief_intrd}}</span>
      </li>
    <!-- vue-php-ssr-template -->
  </ul> 
	<script type="text/javascript">
    new Vue({
      el: '#lst_comicList_1',
      data: {
        list: <?php echo json_encode($list); ?>
      },
      method: {
        'clickComicItem': function (e) {
          var item = e.currentTarget,
              comicId = item.getAttribute('data-id');
          alert('Clicked comic: ' + comicId);
        }
      }
    });
	</script>
```

由于两端差异，并不能真正实现前后端所有语法、状态的一致，只能说最后勉强达到了我们的目的：只需编写一次模板，php 可以根据转化后的模板在服务端渲染出对应 HTML；前端拿到数据后，可以根据原模板重新渲染或者追加数据。

这个方案还有不少问题和限制，但是在满足我们日常的需求的前提下，是目前系统变更成本最低的方案。如果大家有更好的思路，欢迎一起交流讨论~