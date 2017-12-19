---
title: fetch 没有你想象的那么美
permalink: window-fetch-is-not-as-good-as-you-imagined
id: 186
updated: '2017-03-18 20:25:46'
date: 2017-03-18 17:37:11
tags:
---

前端工程中发送 HTTP 请求从来都不是一件容易的事，前有骇人的 `ActiveXObject`，后有 API 设计十分别扭的 `XMLHttpRequest`，甚至这些原生 API 的用法至今仍是很多大公司前端校招的考点之一。

也正是如此，fetch 的出现在前端圈子里一石激起了千层浪，大家欢呼雀跃弹冠相庆恨不得马上把项目中的 `$.ajax` 全部干掉。然而，在新鲜感过后， fetch 真的有你想象的那么美好吗？

> 如果你还不了解 fetch，可以参考我的同事 @camsong 在 2015 年写的文章 [《传统 Ajax 已死，Fetch 永生》](https://github.com/camsong/blog/issues/2)

在开始「批斗」fetch之前，大家需要明确 fetch 的定位：**fetch 是一个 low-level 的 API，它注定不会像你习惯的 `$.ajax` 或是 `axios` 等库帮你封装各种各样的功能或实现。**也正是因为这个定位，在学习或使用 fetch API 时，你会遇到不少的挫折。

（对于没有耐心看完全文的同学，请先记住本文的主旨不在于批评 fetch，事实上 fetch 的出现绝对是前端领域的进步体现。在了解主旨的前提下，关注**加黑**部分即可。）

## 发请求，比你想象的要复杂

很多人看到 fetch 的第一眼肯定会被它简洁的 API 吸引：

```javascript
fetch('http://abc.com/tiger.png');
```

原来需要 `new XMLHttpRequest` 等小十行代码才能实现的功能如今一行代码就能搞定，能不让人动心吗！

但是当你真正在项目中使用时，少不了需要向服务端发送数据的过程，那么使用 fetch 发送一个对象到服务端需要几行代码呢？（出于兼容性考虑，大部分的项目在发送 POST 请求时都会使用 `application/x-www-form-urlencoded` 这种 `Content-Type`）

先来看看使用 jQuery 如何实现：

```javascript
$.post('/api/add', {name: 'test'});
```

然后再看看 fetch 如何处理：

```javascript
fetch('/api/add', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8'
  },
  body: Object.keys({name: 'test'}).map((key) => {
    return encodeURIComponent(key) + '=' + encodeURIComponent(params[key]);
  }).join('&')
});
```

等等，`body` 字段那一长串代码在干什么？**因为 fetch 是一个 low-level 的 API，所以你需要自己 encode HTTP 请求的 payload，还要自己指定 HTTP Header 中的 `Content-Type` 字段。**

这样就结束了吗？如果你在自己的项目中这样发送 POST 请求，很可能会得到一个 `401 Unauthorized` 的结果（视你的服务端如何处理无权限的情况而定）。如果你在仔细看一遍文档，会发现**原来  fetch 在发送请求时默认不会带上 Cookie！**

好，我们让 fetch 带上 Cookie：

```javascript
fetch('/api/add', {
  method: 'POST',
  credentials: 'include',
  ...
});
```

这样，一个最基础的 POST 请求才算能够发出去。

同理，如果你需要 POST 一个 JSON 到服务端，你需要这样做：

```javascript
fetch('/api/add', {
  method: 'POST',
  credentials: 'include',
  headers: {
    'Content-Type': 'application/json;charset=UTF-8'
  },
  body: JSON.stringify({name: 'test'})
});
```

相比于 `$.ajax` 的封装，是不是复杂的不是一点半点呢？

## 错误处理，比你想象的复杂

按理说，fetch 是基于 `Promise` 的 API，每个 fetch 请求会返回一个 Promise 对象，而 Promise 的异常处理且不论是否方便，起码大家是比较熟悉的了。然而 fetch 的异常处理，还是有不少门道。

假如我们用 fetch 请求一个不存在的资源：

```javascript
fetch('xx.png')
.then(() => {
  console.log('ok');
})
.catch(() => {
  console.log('error');
});
```

按照我们的惯例 console 应该要打印出 「error」才对，可事实又如何呢？有图有真相：

![fetch response ok fail](https://img.alicdn.com/tfs/TB1qBeJQXXXXXcoXpXXXXXXXXXX-1168-158.png)

为什么会打印出 「ok」呢？

**按照 MDN 的[说法](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch#Checking_that_the_fetch_was_successful)，fetch 只有在遇到网络错误的时候才会 reject 这个 promise，比如用户断网或请求地址的域名无法解析等。只要服务器能够返回 HTTP 响应（甚至只是 CORS preflight 的 OPTIONS 响应），promise 一定是 resolved 的状态。**

所以要怎么判断一个 fetch 请求是不是成功呢？你得用 `response.ok` 这个字段：

```javascript
fetch('xx.png')
.then((response) => {
  if (response.ok) {
    console.log('ok');
  } else {
    console.log('error');
  }
})
.catch(() => {
  console.log('error');
});
```

再执行一次，终于看到了正确的日志：

![fetch response ok success](https://img.alicdn.com/tfs/TB1DmSmQXXXXXbcaXXXXXXXXXXX-1214-426.png)

## Stream API，比你想象的复杂

**当你的服务端返回的数据是 JSON 格式时，你肯定希望 fetch 返回给你的是一个普通 JavaScript 对象，然而你拿到的是一个 `Response` 对象，而真正的请求结果 —— 即 `response.body` —— 则是一个 `ReadableStream`。**

```javascript
fetch('/api/user.json?id=2')   // 服务端返回 {"name": "test", "age": 1} 字符串
.then((response) => {
  // 这里拿到的 response 并不是一个 {name: 'test', age: 1} 对象
  return response.json();  // 将 response.body 通过 JSON.parse 转换为 JS 对象
})
.then(data => {
  console.log(data); // {name: 'test', age: 1}
});
```

你可能觉得，这些写在规范里的技术细节使用 fetch 的人无需关心，然而在实际使用过程中你会遇到各种各样的问题迫使你不得不了解这些细节。

首先需要承认，fetch 将 `response.body` 设计成 ReadableStream 其实是非常有前瞻性的，这种设计让你在请求大体积文件时变得非常有用。然而，在我们的日常使用中，还是短小的 JSON 片段更加常见。而为了兼容不常见的设计，我们不得不多一次 `response.json()` 的调用。

不仅是调用变得麻烦，如果你的服务端采用了严格的 REST 风格，**对于某些特殊情况并没有返回 JSON 字符串，而是用了 HTTP 状态码（如：`204 No Content`），那么在调用 `response.json()` 时则会抛出异常。**

此外，**`Response` 还限制了响应内容的重复读取和转换**，例如如下代码：

```javascript
var prevFetch = window.fetch;
window.fetch = function() {
  prevFetch.apply(this, arguments)
  .then(response => {
    return new Promise((resolve, reject) => {
      response.json().then(data => {
        if (data.hasError === true) {
          tracker.log('API Error');
        }
        resolve(response);
      });
    });
  });
}

fetch('/api/user.json?id=1')
.then(response => {
  return response.json();  // 先将结果转换为 JSON 对象
})
.then(data => {
  console.log(data);
});
```

是对 fetch 做了一个简单的 AOP，试图拦截所有的请求结果，并当返回的 JSON 对象中 `hasError` 字段如果为 `true` 的话，打点记录出错的接口。

然而这样的代码会导致如下错误：

> Uncaught TypeError: Already read

调试一番后，你会发现是因为我们在切面中已经调用了 `response.json()`，这个时候重复调用该方法时就会报错。（实际上，再次调用其它任何转换方法，如 `.text()` 也会报错）

因此，想要在 fetch 上实现 AOP 仍需另辟蹊径。

## 其它问题

1\. fetch 不支持同步请求

大家都知道同步请求阻塞页面交互，但事实上仍有不少项目在使用同步请求，可能是历史架构等等原因。如果你切换了 fetch 则无法实现这一点。

2\. fetch 不支持取消一个请求

使用 XMLHttpRequest 你可以用 `xhr.abort()` 方法取消一个请求（虽然这个方法也不是那么靠谱，同时是否真的「取消」还依赖于服务端的实现），但是使用 fetch 就无能为力了，至少目前是这样的。

3\. fetch 无法查看请求的进度

使用 XMLHttpRequest 你可以通过 `xhr.onprogress` 回调来动态更新请求的进度，而这一点目前 fetch 还没有原生支持。

## 小结

还是要再次明确，fetch API 的出现绝对是推动了前端在请求发送功能方面的进步。

然而，也需要意识到，**fetch 是一个相当底层的 API，在实际项目使用中，需要做各种各样的封装和异常处理，而并非开箱即用**，更做不到直接替换 `$.ajax` 或其他请求库。

## 参考资料

1. fetch spec https://fetch.spec.whatwg.org/#body
2. fetch 实现 https://github.com/github/fetch
3. 什么是 Already Read 报错 http://stackoverflow.com/questions/34786358/what-does-this-error-mean-uncaught-typeerror-already-read
4. 使用 fetch 处理 HTTP 请求失败 https://www.tjvantoll.com/2015/09/13/fetch-and-errors/
5. https://jakearchibald.com/2015/thats-so-fetch/