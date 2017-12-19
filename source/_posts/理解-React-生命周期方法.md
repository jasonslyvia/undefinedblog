---
title: 理解 React 生命周期方法
permalink: understabd-react-lifecycle
id: 169
updated: '2015-02-02 17:22:55'
date: 2015-02-02 17:22:55
tags:
---

刚接触 React 的时候就发现了 API 里有一系列的 `componentXXX` 方法，按照[官方文档](http://facebook.github.io/react/docs/component-specs.html)的说法，这些方法会在「应用程序生命周期的特定时间点」被执行。但是在实际写代码的过程中，这样模糊的说法还是不能解决所有的疑惑。且看下面的内容：

## mixin 中的方法先执行

考虑以下代码片段：

```
var DummyMixin = {
  componentDidMount: function(){
    console.log('mixin');
  }
};

var DummyClass = React.createClass({
  mixins: [DummyMixin],
  
  componentDidMount: function(){
    console.log('component');
  },
  
  render: function(){
    return null;
  }
});
```

将 `DummyClass` 渲染到 DOM 节点在 console 中输出的结果是：

```
mixin
component
```

[DEMO](http://jsbin.com/zeqiqo/1/edit?html,js,console)

因此可以确认如果在 mixin 中和组件中同时定义了某个生命周期方法，mixin 中的方法会先执行。

## 重复 mount 一个组件

考虑以下代码：

```
var DummyClass = React.createClass({
  render: function(){
    return null;
  }
});

React.render(<DummyClass />, document.body);
React.render(<DummyClass />, document.body);
```

都有哪些生命周期方法被调用，它们的顺序又是怎样的呢？看答案之前先自己思考一下。

```
/* 第一次 render */
getDefaultProps
getInitialState
componentWillMount
render
componentDidMount
/* 第二次 render */
componentWillReceiveProps
shouldComponentUpdate
componentWillUpdate
render
componentDidUpdate
```

## 从 mount 到 unmount

考虑以下代码：

```
var DummyClass = React.createClass({
  render: function(){
    return null;
  }
});

React.render(<DummyClass />, document.body);
React.unmountComponentAtNode(document.body);
```

调用结果是：

```
/* mount */
getDefaultProps
getInitialState
componentWillMount
render
componentDidMount
/* unmount */
componentWillUnmount
```

这里再提一个小问题，先 mount 一个组件再 unmount 再 mount，生命周期方法的调用情况是怎样的？

## 第三方代码控制生命周期

如果是你使用了 react-router 等第三方库来控制你组件的生命周期，你可能不能直观的掌握组件什么时候会被 mount，什么时候会被 unmount。

这个时候你可以使用 [react-lifecycle](https://github.com/jasonslyvia/react-lifecycle) mixin，将这个 mixin 添加到你需要观察的组件中，当任何生命周期方法被调用时，你都能在 console 中看到对应的日志。

当然，这个 mixin 不会输出 render 方法的调用。