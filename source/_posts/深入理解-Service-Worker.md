---
title: 深入理解 Service Worker
date: 2018-01-14 16:57:33
permalink: dive-into-service-worker
tags:
  - ServiceWorker
  - PWA
---

PWA 作为 2018 年前端领域最火的技术之一已经引起了越来越多开发者的关注，其媲美 native app 的交互体验以及对前端开发者友好的入门难度让人不得不怀疑它将掀起新一轮的「App 技术革命」。而 PWA 最核心的「离线」能力，正是通过对 ServiceWorker 这个概念的灵活使用实现的。

在没有深入理解 Service Worker 之前，我总觉得 PWA 无非是在构建的时候使用类似 webpack OfflinePlugin 之类的插件把所有的静态资源缓存起来做成离线应用即可。但是当深入学习了 ServiceWorker 及 CacheStorage 的设计思想、API 规范甚至是发展历程后，我才真正了解到做一个完整的 PWA 是一件多么复杂的事情。

基于我最近一个月对 ServiceWorker 的学习，结合在几个实际业务中的使用情况，我将通过一个系列文章来为大家深入解读 Service Worker 的概念、原理、设计思想、实战玩法以及如何在 PWA 中发挥关键作用等等。

系列文章将划分为如下三个部分：

### 理论部分

- 什么是 Service Worker
- 怎么使用 Service Worker
- Service Worker 的生命周期
- 什么是 CacheStorage
- 如何在 Service Worker 中使用 CacheStorage
- Service Worker 是否足够成熟（截止 2018 年 1 月）

### 实践部分

- 不同离线策略的差异与适用场景
- 如何避免设计出永远不会更新的 Service Worker
- Service Worker 最佳实践

### 延伸阅读

- Service Worker 的前世今生
- PWA 和 AMP 的异同

那么，让我们一起开始 Service Worker 的探索旅程吧！


> Service Worker 是一个相对比较新的技术，规范和浏览器厂商的落地都在不断的变化和升级。我力争在行文过程中遵循「先实践后结论」的方式，保证所有的内容都经过真实环境测试。
>
> 如果文章存在任何错误，请点击文章标题下方的「查看源码」直接对本文编辑并提交 PR。
