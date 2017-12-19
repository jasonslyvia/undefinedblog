---
title: Mocha 测试设置 timeout 不生效的问题排查
permalink: set-timeout-for-mocha-with-karma
id: 185
updated: '2017-01-16 19:56:30'
date: 2017-01-16 19:47:25
tags:
---

Mocha 提供了针对不同粒度的测试超时配置项，但是最近在某个使用了 Karma + Mocha 的项目中遇到无论怎么设置 `this.timeout()` Mocha 都顽固的在 2000ms 时报超时错误的问题，经排查疑似为当 Mocha 和 Karma 一起使用时，需要通过 `karma.conf.js` 来配置 Mocha 超时时间。

当没有正确设置 Karma + Mocha 超时时间时，会报出如下错误：

> Error: Timeout of 2000ms exceeded. For async tests and hooks, ensure "done()" is called; if returning a Promise, ensure it resolves.

此时即使在测试代码里显式添加了超时配置也没有用：

```javascript
describe('Suite', function() {
  it('Case', function() {
    this.timeout(60000); // 无用
  });
});
```

这里还需要注意，一些新手经常犯的错误为这里使用了 ES2015 的箭头函数，this 这个 context 发生了改变，导致超时配置不生效。

最后经过一番 Google，发现当 Mocha 和 Karma 一起使用时，要在 `karma.conf.js` 里对 Mocha 进行配置，配置如下：

```javascript
// karma.conf.js
module.exports = function(config) {
  config.set({
    client: {
      mocha: {
        timeout : 6000 
      }
    }
  });
};
```

其它需要传递给 Mocha 的参数也可以在这里配置。此处配置的超时时间为全局级别，因此设置时要考虑最长的 Case 需要的时间。