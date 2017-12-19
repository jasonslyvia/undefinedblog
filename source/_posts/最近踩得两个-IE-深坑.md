---
title: 最近踩得两个 IE 深坑
permalink: recent-ie-quirks
id: 174
updated: '2015-08-12 23:05:55'
date: 2015-08-12 22:49:17
tags:
---

如果这个世界上没有 IE，前端程序员的寿命至少延长十年。作为一个有节操的程序员，对于 IE 下的各种 quirks 就算不说了如指掌也应该略有了解。什么 IE 6 下 margin 双边距啦、IE 的 Object.defineProperty 只支持 DOM 节点啦，都是小意思！最近踩了两个不太常见的 IE 坑，记录下来跟大家分享，也希望后人能避免再次踩坑。

## IE 9 的样式表限制

IE 对 CSS 选择器的支持非常有限，按理说 IE 9 应该基本算是一个可以用的浏览器了，然而，它对于样式表也有变态的限制。具体限制规则如下：

 - 一个样式表（.css 文件）最多只能包含 4,095 个选择器
 - 一个样式表最多只能 `@import` 31 个样式表
 - `@import` 指令最多只能嵌套 4 层

其中我就结结实实的踩中了第一个坑，因为我们的应用非常复杂，所以单独抽出了一个基础库，包含了多个页面公用的 js 和 css，谁曾想公共的 css 文件选择器数量不知不觉已经达到了 4,300 多个。这样直接导致的后果就是部分选择器及样式被 IE 9 无情的抛弃了。

解决方案：不要触犯以上规则，拆分样式表，去除不必要的选择器

解决工具：[gulp-legacy-ie-css-lint](https://github.com/jasonslyvia/gulp-legacy-ie-css-lint)

来源参考：[so](http://stackoverflow.com/questions/9906794/internet-explorers-css-rules-limits)

## IE 8 下 resize 事件的频繁触发

最近因为业务需要写了一个 [react-lazyload]() 组件（非常好用喔，已经在阿里的对外业务上全面铺开使用了），但是在 IE 8 下性能巨差。经过艰难的断点调试发现，lazyload 的回调被调用的太过频繁了（即使已经加了 300ms 的 debounce）。经过一番 Google 后发现，IE 8 处理 resize 事件的方式简直是变态。

根据 MDN 的描述：

>The resize event is fired when the document view has been resized.

resize 事件应该是当文档可视区域发生变化时触发的（即当你放大或缩小浏览器窗口时），而 IE 8 下却不是。IE 8 下触发 resize 事件的逻辑是：

当**一个元素的尺寸**发生改变的时候，就触发 resize 事件

这样当你更新 DOM 节点的时候，resize 事件的触发频率及次数可想而知。

解决方案：不需要使用 resize 事件的时候尽量不要绑定这个事件；实在需要的话在 DOM 可能发生改变前解除 resize 事件的绑定，在 DOM 更新完成后重新监听 resize 事件

来源参考：[so](http://stackoverflow.com/questions/1852751/window-resize-event-firing-in-internet-explorer)

这里多说一句，很多应用在 IE 8 下觉得卡顿，也可以检查一下是否因为绑定了 resize 事件的缘故。

