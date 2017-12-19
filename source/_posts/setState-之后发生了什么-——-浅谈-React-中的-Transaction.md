---
title: setState 之后发生了什么 —— 浅谈 React 中的 Transaction
permalink: what-happened-after-set-state
id: 177
updated: '2015-10-26 01:31:51'
date: 2015-10-26 01:28:22
tags:
---

> 本文系对 [深入理解 React 的 batchUpdate 机制](http://undefinedblog.com/understand-react-batch-update/) 的更新，根据 React v0.14 版源码添加并修改了部分内容。同时增加了一张看起来并不容易理解的示意图。

之前在我的博客里有篇文章写了「[为什么 React 这么快](http://undefinedblog.com/why-react-is-so-fast/)」，其中说到一点很重要的特性就是 `batchUpdate`。今天就简单的分析一下 batchUpdate 究竟是怎么实现的？

首先明确一个基础概念，在调用 this.setState 之后，React 会自动重新调用 render 方法。但是如果我们多次调用 setState 方法呢？考虑下面的代码：

```javascript
var Example = React.createClass({
  getInitialState: function() {
    return {
      clicked: 0
    };
  },

  handleClick: function() {
    this.setState({clicked: this.state.clicked + 1});
    this.setState({clicked: this.state.clicked + 1});
  },

  render: function() {
    return <button onClick={this.handleClick}>{this.state.clicked}</button>;
  }
});
```

考虑这两个问题，当点击 button 时：1、 render 方法被调用了几次？2、 handleClick 中第 2 次 setState 时 this.state.clicked 值是多少？

如果你还没有把握，可以先打开这个 [DEMO](http://jsbin.com/zowuve/1/edit?js,console,output) 查看结果。

知道结果后，你能解释为什么 render 方法只被调用了 1 次，且第二次 setState 时 this.state.clicked 依然是 0 么？

这一切，都要归功于 React 的 batchUpdate 设计，但是 batchUpdate 究竟是怎么实现的呢？让我们深入到 React 的源码中一探究竟。

> 以下内容由于本人能力及精力均有限，尚未深究，仅供参考。如有错误，尽请斧正！源码分析基于 React v0.14 版。

一个简单的 setState 方法，其简化的调用栈如下：

```
this.setState                                           // ReactComponent.js
...
this.updater.enqueueSetState                            // ReactUpdateQueue.js
...
获取当前组件的 pendingStateQueue，并将新的 state push 进去 // ReactUpdateQueue.js
...
enqueueUpdate                                           // ReactUpdates.js
...
if(当前不在一次 batchUpdate 的过程中)                      // ReactUpates.js
  执行 batchingStreategy.batchUpdates 方法
else
  将当前 component 存入 dirtyComponents 数组中
...
if (setState 方法存在 callback)                          // ReactComponent.js
  调用 this.updater.enqueueCallback 将 callback 存入队列中
...
```

一次 setState 的过程大概就是这样，那么问题来了，setState 只是把变化存入一个临时队列中，那么新的 state 究竟是如何生效的呢？

让我们从 DOM 事件触发开发对整个链路进行一下梳理：

当一次 DOM 事件触发后，ReactEventListener.dispatchEvent 方法会被调用。而这个方法并不是急着去调用我们在 JSX 中指定的各种回调，而是调用了

```
ReactUpdates.batchedUpdates
```

该方法会执行一个 transaction，而要执行的内容被定义在 handleTopLevelImpl 中。再看 handleTopLevelImpl，其核心内容就是找到事件对应的所有节点并依次对这些节点 trigger 事件。

在进一步介绍之前，我们需要先了解一下 Transaction（事务）。

React 为 Transaction 封装了一个简单的 Mixin，并在源码中生动的介绍了一个 Transaction 是怎么工作的。

```javascript
// 详见 Transaction.js

/**
 *                       wrappers (injected at creation time)
 *                                      +        +
 *                                      |        |
 *                    +-------------------------------------+
 *                    |                 v        |              |
 *                    |      +---------------+   |              |
 *                    |   +--|    wrapper1   -----+         |
 *                    |   |  +---------------+   v    |         |
 *                    |   |          +-------------+  |         |
 *                    |   |     +----|   wrapper2  -------+   |
 *                    |   |     |    +-------------+  |     |   |
 *                    |   |     |                     |     |   |
 *                    |   v     v                     v     v   | wrapper
 *                    | +---+ +---+   +---------+   +---+ +---+ | invariants
 * perform(anyMethod) | |   | |   |   |         |   |   | |   | | maintained
 * +----------------->----->|anyMethod------------->
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | +---+ +---+   +---------+   +---+ +---+ |
 *                    |  initialize                    close    |
 *                    +-----------------------------------------+
 **/
```

实际上，Transacation 就是给需要执行的函数封装了两个 wrapper，每个 wrapper 都有 initialize 和 close 方法。当一个 transaction 需要执行（perform）的时候，会先调用对应的 initialize 方法。同样的，当一个 transaction 执行完成后，会调用对应的 close 方法。

回到刚才的问题，存入临时队列的 state 究竟是怎么被改变成真正的状态的呢？秘密就在 `ReactUpdates.batchedUpdates` 调用中（还记得它吗？DOM 事件的回调中被调用的方法）。这个方法实际上就是执行了一个 transaction，具体实现定义在 ReactDefaultBatchingStrategy.js 中。

这个 transaction 具体的代码就不贴了，简单的说，它定义了一个事务，当这个事务结束时，需要调用 ReactUpdates.flushBatchedUpdates 方法。

好，说到这里可能大家已经被各种名称搞昏了头，下面给大家看一个简单的流程图。

![react transaction 图例](http://ww1.sinaimg.cn/large/831e9385gw1exdvqnke7nj21be0fwtc9.jpg)

上面的流程图中只保留了部分核心的过程，看到这里大家应该明白了，所有的 batchUpdate 功能都是通过执行各种 transaction 实现的。this.setState 调用后，新的 state 并没有马上生效，而是通过 ReactUpdates.batchedUpdate 方法存入临时队列中。当一个 transaction 完成后，才通过 ReactUpdates.flushBatchedUpdates 方法将所有的临时 state merge 并计算出最新的 props 及 state。

纵观 React 源码，使用 Transaction 之处非常之多，React 源码注释中也列举了很多可以使用 Transaction 的地方，比如

 - 在一次 DOM reconciliation（调和，即 state 改变导致 Virtual DOM 改变，计算真实 DOM 该如何改变的过程）的前后，保证 input 中选中的文字范围（range）不发生变化
 - 当 DOM 节点发生重新排列时禁用事件，以确保不会触发多余的 blur/focus 事件。同时可以确保 DOM 重拍完成后事件系统恢复启用状态。
 - 当 worker thread 的 DOM reconciliation 计算完成后，由 main thread 来更新整个 UI
 - 在渲染完新的内容后调用所有 `componentDidUpdate` 的回调
 - 等等

值得一提的是，React 还将 batchUpdate 方法暴露了出来：

```javascript
var batchedUpdates = require('react-dom').unstable_batchedUpdates;
```

当你需要在一些非 DOM 事件回调的函数中多次调用 setState 等方法时，可以将你的逻辑封装后调用 batchedUpdates 执行，以此保证 render 方法不会被多次调用。