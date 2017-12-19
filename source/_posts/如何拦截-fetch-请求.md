---
title: 如何拦截 fetch 请求
permalink: how-to-intercept-fetch
id: 187
updated: '2017-04-23 21:08:17'
date: 2017-04-23 20:47:58
tags:
---

在前端工程实践中，经常会有拦截 Ajax 请求的需求，比如统一添加 CSRF token，或者统一实现缓存处理等。在前 fetch 时代，如果使用了 jQuery，可以直接通过配置 `jQuery.ajaxPrefilter` 实现；如果用的是原生 API，也可以通过 hack XMLHttpRequest 完成同样的功能。

然而在 fetch API 出现后，事情就没那么简单了。在我上一篇文章[《fetch 没有你想象的那么美》](https://undefinedblog.com/window-fetch-is-not-as-good-as-you-imagined/)中，我提到了对 fetch 进行 Monkey Patch 时遇到的奇怪报错：

> Uncaught TypeError: Already read

同时解释了这是 fetch 使用 Stream API 导致的限制，那这是不是意味着我们无法 hack 原生的 fetch 从而实现对 fetch 结果的统一拦截过滤了呢？答案是否定的。

在仔细阅读完 fetch 相关的文档后，我在 `Response` 对象的[文档页面](https://developer.mozilla.org/en-US/docs/Web/API/Response)找到了一个有意思的属性和一个更有意思的方法。

### Reponse.bodyUsed

这个属性用于标示这个 Response 对象是否已经被读取过，比如下面的代码：

```javascript
fetch('/api/user.json')
.then(response => {
  console.log(response.bodyUsed); // false
  return response.json().then(json => {
    console.log(response.bodyUsed); // true

    // 如果这里再次尝试对 response 的 Stream 数据进行读取，则会报错
    // response.text();   // Uncaught TypeError: Already read
    return json;
  });
})
.then(json => {
  // 拿到服务端响应的 JSON 对象
});
```

就很好的解释了 bodyUsed 属性的作用和使用方法。但是从实用角度来说，这个布尔值并没有太大的作用，真正帮我们解决拦截并统一响应 fetch 请求的功能在下面。

### Reponse.prototype.clone()

这个 API 顾名思义就是对 response 对象的一个复制，根据 [文档](https://developer.mozilla.org/en-US/docs/Web/API/Response/clone) 描述，复制出来的 response 对象和原来的对象数据和行为完全一致，只是保存在不同的变量中而已。

那么 clone 方法如何解决 Already Read 报错的问题呢，让我们稍微修改一下上面的代码：

```javascript
fetch('/api/user.json')
.then(response => {
  console.log(response.bodyUsed); // false
  return response.clone().json().then(json => {
    console.log(response.bodyUsed); // false

    // 如果这里再次尝试对 response 的 Stream 数据进行读取，没有问题
    // response.text().then(text => console.log(text));
    return json;
  });
});
```

注意第 4 行我们在 `response.json()` 调用前添加了 `response.clone().json()`，这样就完成了对 response 对象的复制。后续的 Stream API 读取发生在一个全新的 response 对象上，所以原来的 response 对象 bodyUsed 属性为 false，也可以对其调用不同的读取方法，如 .text() 或 .blob()。

唯一需要注意的是，clone 方法的调用一定要发生在对 response 调用任何读取之前，比如下面的代码依然会报错：

```javascript
fetch('/api/user.json')
.then(response => {
  console.log(response.bodyUsed); // false
  return response.json().then(json => {
    // 如果这里先尝试 clone，再对 response 的 Stream 数据进行读取，依然会报错
    // response.clone().text().then(text => console.log(text));
    return json;
  });
});
```