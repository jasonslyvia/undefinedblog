---
title: 怎么使用 Service Worker
date: 2018-01-28 22:24:05
permalink: how-to-use-service-worker
tags:
---

> 本文是 [深入理解 Service Worker](/dive-into-service-worker) 系列文章中的第二篇

本周苹果官方发布了 Safari 11.1 的 TP （技术预览）版，据[更新日志](https://developer.apple.com/library/content/releasenotes/General/WhatsNewInSafari/Articles/Safari_11_1.html)显示，iOS 11.3 及 macOS 10.13.4 中的 Safari 将全面支持 Service Worker，这为 PWA 的推广扫除了最后的兼容性问题。

如果 iOS 能够完美的支持 Service Worker，那么目前 App Store 中至少一半以上纯信息展示型的 App 将无需通过 App Store 就能呈现给用户，无疑将引起新一轮的移动 App 开发革命。当然，根据部分开发者的[反馈](https://medium.com/@firt/pwas-are-coming-to-ios-11-3-cupertino-we-have-a-problem-2ff49fd7d6ea)，Safari 技术预览版中 Service Worker 的实现还存在不少问题。

借着这个振奋人心的消息，本文开始聊聊如何使用 Service Worker。

## 注册一个 Service Worker

Service Worker 虽然是 Web Worker 的一种，但注册和使用方式与 Web Worker 却不尽相同。Web Worker 有一个全局构造函数 `Worker`，你可以通过 `new Worker('worker.js')` 的方式新建一个 Web Worker。

而 Service Worker 则需要需要 `navigator.serviceWorker.register()` 方法进行注册。这里还有个有意思的思考点，为什么同样是 Web Worker，注册方式甚至是挂载的层级都不同（一个是 `window`，一个是 `navigator`）呢？

下面的代码片段中，我们在页面里注册了一个 Service Worker，Service Worker 的具体行为在 `sw.js` 中进行定义。

```html
// index.html
<script>
if (navigator.serviceWorker) {
  navigator.serviceWorker.register('sw.js')
    .then(registration => {
      // 注册成功
    })
    .catch(error => {
      // 注册失败
    });
}
</script>
```

在深入使用 Service Worker 后你会发现，所有相关的 API 都是 Promise-based 的，因此在调用注册方法后，我们需要使用 Promise 的方式来处理注册的结果。

若注册成功，则 Promise resolve 一个 [ServiceWorkerRegistration](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration) 对象，通过这个对象你可以获取到当前注册的 Service Worker 的相关状态，或者订阅新的 Service Worker 注册事件，亦或者是注销一个 Service Worker。

```js
// ...
.then(registration => {
  if (registration.installing) {
    // 当前存在处于 installing 状态的 Service Worker
  }

  if (registration.active) {
    // 当前存在处于 active 状态的 Service Worker，即控制当前页面的那个 Service Worker
  }
});
```

若注册失败，在 catch 中你会捕获到具体的错误。比如说 `sw.js` 中存在语法错误甚至压根不存在，或者没有使用 https 协议等，都会导致 Service Worker 注册失败。

## 注销一个 Service Worker

很多刚刚接触 Service Worker 的同学都会犯的一个错误就是，不知道如何注销一个 Service Worker。由于 Service Worker 有非常强大的网络拦截能力，如果错误的或者过时的 Service Worker 没有被正确注销，将在特定的场景下导致非常严重的问题。

在注册 Service Worker 时我们在代码中调用 `navigator.serviceWorker.register()` 方法，那么是不是只要删除这个注册语句，Service Worker 就会被注销掉呢？答案是否定的。

**你必须显式的调用 `unregister()` 方法才能正确的注销 Service Worker，除此之外无论你是删除了 `register()` 调用还是删除了注册时的那个 `sw.js`，都不会对已经注册成功的 Service Worker 起到任何作用。**

```js
// ...
.then(registration => {
  registration.unregister().then(result => {
    if (result) {
      // 注销成功
    } else {
      // 注销失败
    }
  });
});
```

## 更新一个 Service Worker

上面我们讲到了如何注册或是注销一个 Service Worker，那如果我们的 Service Worker 需要更新呢？

更新时并**没有**一个类似于 `navigator.serviceWorker.update()` 的方法，我们只需要更改 Service Worker 中的内容（修改 `sw.js`），然后再次调用 `register()` 方法即可。

```js
// 第一次注册
navigator.serviceWorker.register('sw.js');

// 修改 sw.js 中的内容

// 第二次注册，实则为更新过程
navigator.serviceWorker.register('sw.js');
```

你无需修改 `sw.js` 的文件名，只要保证其中的内容发生变化，则浏览器会自动启动更新程序。

## 小结

在本文中我们讲到了如何注册、注销和更新 Service Worker，这既是大家非常容易忽略的基础知识，也引出了下一章我们要讲的内容：Service Worker 的生命周期。

因为在你调用 `register()` 后，发生的事情远远没有一行代码那么简单。
