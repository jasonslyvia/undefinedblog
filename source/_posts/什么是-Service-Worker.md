---
title: 什么是 Service Worker
date: 2018-01-14 17:14:26
permalink: what-is-service-worker
tags:
---

> 本文是 [深入理解 Service Worker](/dive-into-service-worker) 系列文章中的第一篇

## Service Worker 的定义

Service Worker 听起来很像 Web Worker。

根据 [Service Worker 的规范定义](https://w3c.github.io/ServiceWorker/#service-worker-concept)，它正是 Web Worker 中的一种（严格的说，Service Worker 应该是 [Shared Worker](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker) 的一种），但更专注于对网络请求的拦截、后台数据更新等，这些能力正是为构建一个 PWA 打好了坚实的基础。

![service worker 概念](service worker.png)

<p class="caption">一个简化的 Service Worker 工作示意图</p>

如果你还不清楚什么是 Web Worker，可以把它简单的理解为一个独立于主线程执行的 JavaScript 模块（无论是一个独立的 `.js` 文件或者 `type="module"`类型的 `<script>`标签）。之所以需要独立于主线程运行，是为了解决一些计算密集型的操作阻塞界面渲染和用户交互的问题。由于工作在不同的线程中，Web Worker 和主线程之间通过 postMessage 等方式进行通信。

## Service Worker 到底是什么

只看定义可能很难对 Service Worker 有比较感性的认知，那我们直接上代码：

```js
// service-worker.js
self.addEventListener('install', e => {
  console.log('installing service worker');
});

self.addEventListener('activate', e => {
  console.log('activating service worker');
});

self.addEventListener('fetch', e => {
  console.log(`fetching ${e.request.url}`);
});
```

上面的代码片段定义了一个很简单的 Service Worker，通过在页面上调用 `navigator.serviceWorker.register('/service-worker.js')` 即可将这个 Service Worker 注册到当前域下。

关于 Service Worker 的注册及使用，会在第二篇《怎么使用 Service Worker》中进行详细介绍。现在，你只需要对 Service Worker 有一个初步的认识即可。

## Service Worker 的性质

1\. **与 Web Worker 一样，因为运行在独立的线程中，Service Worker 无法直接进行任何 DOM 操作**

```js
//service-worker.js
self.addEventListener('install', e => {
  alert('注册成功');  // TypeError, alert is not a function
});
```

2\. **Service Worker 运行在独立的全局上下文环境（[ServiceWorkerGlobalScope](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope)）中**

你可以通过 `self` 关键字在 Service Worker 代码的任意位置来指向这个全局环境，有点类似 node.js 中的 `global` 关键字。

当然，这不意味着你只能在 Service Worker 中使用 `self`，`this` 依然可以被使用，只是考虑到各种作用域的变更（如箭头函数、bind 等），`self` 能够在任何位置、任何环境下永远指向这个全局作用域。

3\. **Service Worker 仅能运行在 HTTPS 环境下**

主要是考虑到网络请求拦截等能力的危险程度，但为了调试方便可以在 HTTP 环境的 localhost 中进行调试。

4\. **Service Worker 的注册必选遵守「同源策略（Same Origin Policy）」**

这也是很多人疑惑为什么注册一个 CDN 上的 Service Worker 会报错的问题所在，比如页面是 `https://www.exmaple.com`，注册一个 `https://cdn.example.com/service-worker.js`，就会导致跨域错误。

5\. **Service Worker 有固定的作用范围，可以在注册时通过 `scope` 参数指定，若留空则自动根据 Service Worker 的 location 来判断**

举个例子，若没有特别设置 `scope`， `https://www.example.com/sw.js` 的作用域就是 `/`，因此这个 Service Worker 可以拦截该域下页面的任意请求；但是 `https://www.example.com/folder/sw.js` 的作用域只能是 `/folder/`，若访问的页面是 `https://www.example.com/another-folder/index.html`，这个 Service Worker 不会拦截到任何的请求。

因此一般来说，Service Worker 应该注册在根目录下，根据应用的需要指定 `scope` 即可。

6\. **基于第 5 点性质，一个域下可以注册若干个 Service Worker，只要他们的 `scope` 不同即可**

7\. **Service Worker 是事件驱动的**

当事件触发时会执行 Service Worker 里注册的事件回调函数，当事件处理完成后会这个 Service Worker 对应的线程可能会被销毁。因此你无法在 Service Worker 中定义全局变量，如果需要持久化一些内容在不同的事件触发时进行传递，可以考虑使用 [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)。

8\. **Service Worker 一旦注册成功，便不依托于某个具体的页面存在**

举个例子，你在 `https://www.example.com/index.html` 使用 `register('service-worker.js')` 成功注册了一个 Service Worker。关闭浏览器后重新访问 `https://www.example.com/index.html`，移除掉 `register('service-worker.js')` 的调用，之前注册的 Service Worker 依然会正常运行。

9\. **Service Worker 在同源页面下是共享的**

这一点从 「Service Worker 是 [Shared Worker](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker) 的一种」可以推导出，不再单独解释。

## 小结

在这篇文章中，我们介绍了什么是 Service Worker、Service Worker 的性质，以及通过一个简短的例子让大家了解到了一个 Service Worker 长什么样子。

由于 Service Worker 是从 Web Worker 和 Shared Worker 演化而来的 API，很多 Web Worker 的性质、用法等还需要读者再仔细研究。
