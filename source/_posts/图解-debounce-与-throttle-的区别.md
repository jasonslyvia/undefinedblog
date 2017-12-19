---
title: 图解 debounce 与 throttle 的区别
permalink: debounce-and-throttle
id: 168
updated: '2015-01-25 21:46:33'
date: 2015-01-25 21:41:43
tags:
---

在实现一些需要被频繁调用的函数时，我们通常都会使用 `debounce` 或 `throttle` 方法。在我的印象中，它们的作用就是减少函数被调用的次数，但具体有什么区别，却真的不能说清楚。

直到最近看了一篇精彩的[博文](http://drupalmotion.com/article/debounce-and-throttle-visual-explanation)，用可视化的方法展示了两者的区别，很有启发性，值得参考。

![debounce 与 throttle](/content/images/2015/Jan/QQ20150125-1-2x.png)

注意到上图，第一行 `Mousemove Events` 展示了 `mousemove` 事件触发的频率。第二行和第三行是分别使用 underscore 与 jQuery 的 debounce 方法(leading 参数为true，保证一开始先调用一次目标函数)后事件的触发频率。第四、五行则是 trailing 参数为 true（保证 delay 结束调用一次）的触发频率。与之对比的是最后三行，使用的是 throttle 方法。

首先我们的直观感觉是使用 debounce 方法相比于 throttle 方法事件触发的频率更低，但实际上不能这么理解。要解释上图的结果，首先需要了解 debounce 和 throttle 的原理。

当我们阅读 lodash 的代码时会发现，throttle 方法不过是 debounce 方法的一个修饰。也就是说，_.throttle 和 _.debounce 最终都会都会调用 debounce 方法。那么 debounce 究竟是如何工作的呢？

首先看 _.debounce 的 API：

```
_.debounce(func, delay, options);
```

func 是需要被调用的目标函数，delay 是时间限制，options 是一系列的配置，为了简化问题，这里不多提。

当调用 _.debounce 后，会返回一个函数，这个函数在被调用时会生成一个 setTimeout(delayed, delay)。其中 delayed 又是一个内部方法，在 delayed 被调用时进行如下检测：当前时间 - 上次func被调用事件 是否 小于 0 或 大于 delay ？如果是则执行一次 func，记录并返回执行结果，同时更新上次被调用时间；如果不是则调用 setTimeout 进行下一次的判断。

因此，当你像下面这样绑定事件，

```
$(window).on('mousemove', _.debounce(moveHandler, 500));
```
并频繁移动鼠标时，你会发现 moveHandler 压根没有被调用过！直到你停下来后，moveHandler 才会被调用一次。

至于 _.throttle 方法，只不过是多给 debounce 传了 maxWait、leading=true、trailing=true 选项，这些选项的意思是至少保证在每 maxWait 时间让 func 被调用一次。

说到这儿，你应该就明白了为什么会出现上图那样的调用情况。如果你频繁的移动鼠标，throttle 会保证在每 maxWait 时间调用 func 一次，而 debounce 如果没有明确设置 maxWait，是一直不会调用 func 直到你停止移动鼠标后才会调用一次。

> 因此，可以笼统的说 _.throttle 就是设置了 leading=true, trailing=true 及 maxWait 的 _.debounce。

**那什么时候该用 debounce 什么时候该用 throttle 呢？**

下面我列举了一些场景：

 - input 中输入文字自动发送 ajax 请求进行自动补全： debounce
 - resize window 重新计算样式或布局：debounce
 - mouseleave 时隐藏二级菜单：debounce，并合理使用 cancel 方法
 - scroll 时更新样式，如随动效果：throttle
 
最重要的还是理解两者对调用时间及次数上的处理，根据业务逻辑选择最合适的优化方案！