---
title: 为什么 React 这么快？
permalink: why-react-is-so-fast
id: 167
updated: '2014-12-31 17:25:54'
date: 2014-12-31 17:25:54
tags:
---

第一次使用 React 的同学都会对 React.js 的文件大小感到隐隐的不安。的确，未压缩的代码有 450KB+，GZIP 后还有近 100KB，这么庞大的库会不会在性能上存在瓶颈呢？

其实在实际开发过程中，并没有感觉到我们基于 React 构建的 WebApp 有任何的性能瓶颈。按照 React 官方的说法，即使在 iOS 的 UIWebView 中（无法使用 JavaScript JIT 引擎），React 也能保证 60FPS 的更新率。

那么 React 是如何做到如此高效的呢，其核心开发者之一 Pete Hunt 在 [Quora](http://www.quora.com/Pete-Hunt/Posts/React-Convincing-the-Boss) 上针对这一问题做了阐述。

下面就我个人的理解结合 Pete Hunt 的回答，整理归纳了 5 点 React 如此高效的原因：


**1. React 的核心代码在开发过程中就考虑了编译器优化问题**

具体的优化措施包括使用对象池来避免频繁的垃圾回收（垃圾回收过程可能导致浏览器出现卡顿现象）、兼容 Google 的 [Closure Compiler](https://developers.google.com/closure/compiler/)以及保证代码被 JIT 正确的优化。

**2. 每一个 React 组件都有完整的生命周期**

组件的生命周期如 `getInitialState`、`componentWillMount` 等，这些生命周期方法保证所有对 DOM 的修改都是批量更新的（batch update）。这样你就可以避免读一次 DOM、写一次 DOM 再读一次 DOM 这样频繁触发 layout 的糟糕代码了。

**3. 强大的 Virtual DOM**

实际上，JavaScript 之所以让人感觉慢就是因为 DOM 操作慢。试想随便新建一个 DOM 元素就有无数个属性、方法、事件、回调，这样的性能损耗是不能接受的。当 state 发生改变时，React 提供的 `render` 方法并不会直接把你定义的 HTML 结构重新写进 DOM 中，而是在内部的 Virtual DOM 中进行 diff，再计算出需要更新的 DOM，最后再把这部分需要更新的 DOM 写入真正的 DOM 中。

**4. 高效的单向数据绑定**

写过 Angular 的同学都知道 Angular 提供的双向数据绑定用着很爽，但是当需要绑定的数据越来越多时，Angular 的脏值检测方法就显得力不从心了。而 React 提供的仅是单向数据绑定，这样的绑定并不会让你觉得不便，反而配合 React 自己的事件系统，用起来得心应手。

**5. 用于极致优化 shouldComponentUpdate()**

即使 React 提供了这么多性能优化的方法，你还是可以进一步优化你的组件的性能，这就是 `shouldComponentUpdate()` 方法。在这个方法中你可以根据新的 props 和当前 props 的差异对比确认当前这个组件是否需要更新。


有了以上 5 点优化，开发者无需担心如何优化 React 的性能，而可以更多的把精力集中在业务代码的实现上，从而真正实现高效的前端开发。