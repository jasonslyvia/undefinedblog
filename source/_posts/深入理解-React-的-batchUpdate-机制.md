---
title: 深入理解 React 的 batchUpdate 机制
permalink: understand-react-batch-update
id: 172
updated: '2015-06-22 23:47:52'
date: 2015-06-22 14:43:02
tags:
---

之前有篇文章写了「[为什么 React 这么快](http://undefinedblog.com/why-react-is-so-fast/)」，其中说到一点很重要的特性就是 `batchUpdate`。我一直以为 batchUpdate 是 Virtual DOM 的什么黑科技，直到上周跑去支付宝跟承玉等大牛交流后才直到自己理解的有偏差。

废话少说，直接上代码：

```
var Parent = React.createClass({
  getInitialState: function() {
     return {
       text: 'default'
     };
  },
  
  handleChildClick: function(){
    this.setState({
      text: Math.random() * 1000
    });
  },
  
  render: function(){
    console.log('parent render');
    
    return (
      <div className="parent">
       this is parent!
       <Child text={this.state.text} onClick={this.handleChildClick} />
      </div>
    );
  }
});


var Child = React.createClass({
  getInitialState: function() {
    return {
     text: this.props.text + '~'
    };
  },
  
  componentWillReceiveProps: function(nextProps) {
    this.setState({
      text: nextProps.text + '~'
    });
  },
  
  handleClick: function(){
    this.setState({
      text: 'clicked'
    });
    
    this.props.onClick();
  },
  
  render: function() {
    console.log('child render');
    
    return (
     <div className="child">
       I'm child
       <p>something from parent:</p>
       <p>{this.state.text}</p>
       <button onClick={this.handleClick}>click me</button>
     </div>
    );
  }
});

React.render(<Parent/>, document.body);
```

[DEMO](http://jsbin.com/yaqimo/2/edit?js,console,output)

当组件首次渲染时，console 输出内容如下：

```
parent render
child render
```

下面考虑点击 `Child` 中的 button，问 console 中输出的内容是怎样的？

让我们看看 Child 的 `handleClick`，首先调用了一次 `setState`

```
this.setState({
  text: 'clciked'
});
```

然后调用了 Parent 的 `onClick` 回调。而 Parent 的 handleChildClick 方法更新了自己的 state，从而触发了新一轮的 render。

那么，console 中输出的内容莫非是：

```
child render   // Child setState
parent render  // Parent setState
child render   // Parent state 更新，Child 跟着 re-render
```

但实际情况是：

```
parent render
child render
```

我们预期的 Child 调用 setState 导致的 render 似乎并没有发生。

下面再来一个简单点的例子：

```
//...
  handleClick: function() {
    this.setState({
      text: 'clicked'
    });
    
    var newText = this.state.text + ' agian';
    this.setState({
      text: newText
    });
  }
//...
```

如果我们把 Child 的 handleClick 改写成上面这样，render 的触发情况又是怎样的呢？Child 的 state 中 `text` 的值又是什么呢？

答案见 [DEMO](http://jsbin.com/yaqimo/3/edit?js,console,output)，本文不再赘述。

到这里，我们已经领会了 batchUpdate 的强大之处，但是 batchUpdate 究竟是怎么实现的呢？让我们深入到 React 的源码中一探究竟。

> 以下内容由于本人能力及精力均有限，尚未深究，仅供参考。如有错误，尽请斧正！

首先，我们在 Child 的 handleClick 中调用了 setState，其简化的调用栈如下：

```
this.setState
...
this.replaceState
...
将更新后的 state 保存为 this._pendingState
...
if (不是在 componentWillMount 的生命周期中)
ReactUpdates.enqueneUpdate
...
将当前 component 存入 dirtyComponents 数组中
...
if (存在 callback)
将 callback 保存到 component._pendingCallbacks 数组中
...
```

一次 setState 调用就这么结束了，而下一行调用 `this.props.onClick()` 触发的也是 Parent 的 setState，整体调用栈类似。

那么问题来了，两次 setState 都是把变化存入一个 pending 数组中，那么变化最后究竟是怎么起作用的呢？

实际上我们看看一次事件触发的完整调用栈就能大概明白了。

当一次 DOM 事件触发后，ReactEventListener.dispatchEvent 方法会被调用。而这个方法并不是急着去调用我们在 JSX 中指定的各种回调，而是调用了 

```
ReactUpdates.batchedUpdates
```

这个方法就是 React 整个 batchUpdate 思想的核心。该方法会执行一个 transaction，而要执行的内容被定义在 handleTopLevelImpl，其实就是找到事件对应的所有节点并依次对这些节点触发事件。

在进一步介绍之前，我们需要先了解一下 Transaction。

React 为 Transaction 封装了一个简单的 Mixin，并在源码中生动的介绍了一个 Transaction 是怎么工作的。

```
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
 
回到刚才的问题，pending 的状态究竟是怎么被改变成真正的状态的呢？其实在刚开始的时候，一个事件触发后 ReactEventListener.dispatchEvent 被调用，这个函数中调用了 batchingStrategy 的 batchUpdate 方法。而 batchingStrategy 中使用了一个 ReactDefaultBatchingStrategyTransaction 的示例 transaction， 那么这个名字超长的类究竟干了什么？


````
function ReactDefaultBatchingStrategyTransaction() {
  this.reinitializeTransaction();
}

assign(
  ReactDefaultBatchingStrategyTransaction.prototype,
  Transaction.Mixin,
  {
    getTransactionWrappers: function() {
      return TRANSACTION_WRAPPERS;
    }
  }
);
```

注意它被 assgin 的 getTransactionWrappers 方法，返回了一个常量 TRANSACTION_WRAPPERS。

看到这里你应该就明白了，这个常量里定义了最后把 pendingState 改写为真正的 state 的方法。

```
var NESTED_UPDATES = {
  initialize: function() {
    this.dirtyComponentsLength = dirtyComponents.length;
  },
  close: function() {
    if (this.dirtyComponentsLength !== dirtyComponents.length) {
      // Additional updates were enqueued by componentDidUpdate handlers or
      // similar; before our own UPDATE_QUEUEING wrapper closes, we want to run
      // these new updates so that if A's componentDidUpdate calls setState on
      // B, B will update before the callback A's updater provided when calling
      // setState.
      dirtyComponents.splice(0, this.dirtyComponentsLength);
      flushBatchedUpdates();
    } else {
      dirtyComponents.length = 0;
    }
  }
};

var UPDATE_QUEUEING = {
  initialize: function() {
    this.callbackQueue.reset();
  },
  close: function() {
    this.callbackQueue.notifyAll();
  }
};

var TRANSACTION_WRAPPERS = [NESTED_UPDATES, UPDATE_QUEUEING];
```

我们需要关注的就是 NESTED_WRAPPER 中的 close 方法，也就是这个方法指明了当所有的事件触发响应结束后，flushBatchUpdates()。至于究竟是怎么 flush 的，本文暂不深究。

纵观 React 源码，使用 Transaction 之处非常之多，React 源码注释中也列举了很多可以使用 Transaction 的地方，比如

 - 在一次 DOM reconciliation（调和，即 state 改变导致 Virtual DOM 改变，计算真实 DOM 该如何改变的过程）的前后，保证 input 中选中的文字范围（range）不发生变化
 - 当 DOM 节点发生重新排列时禁用事件，以确保不会触发多余的 blur/focus 事件。同时可以确保 DOM 重拍完成后事件系统恢复启用状态。
 - 当 worker thread 的 DOM reconciliation 计算完成后，由 main thread 来更新整个 UI
 - 在渲染完新的内容后调用所有 `componentDidUpdate` 的回调
 - 等等
 