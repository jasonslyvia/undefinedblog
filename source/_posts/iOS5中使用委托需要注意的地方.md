---
title: iOS5中使用委托需要注意的地方
permalink: ios5zhong-shi-yong-wei-tuo-xu-yao-zhu-yi-de-di-fang
id: 30
updated: '2011-12-07 18:36:32'
date: 2011-12-07 18:36:32
tags:
---

<p>iOS5中对delegate的实现作出了一些调整。假如class A声明遵循了class B的delegate protocol但是没有实现或仅部分实现其delegate method；那么当class B的实例被设为class A实例的delegate时，当调用条件满足时，在iOS5中会引发delegate method调用从而crash，而在旧的系统上则不会引发调用。</p>
