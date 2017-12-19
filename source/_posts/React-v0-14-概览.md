---
title: React v0.14 概览
permalink: react-v0-14
id: 176
updated: '2015-10-08 03:27:25'
date: 2015-10-08 02:21:24
tags:
---

> 基本上本文就是对 [React 官方 v0.14 博文](http://facebook.github.io/react/blog/2015/10/07/react-v0.14.html) 的翻译加上一小部分个人理解。

正文开始之前先讲个笑话，随着 React 的风靡许多基于 React 的衍生库也火得一塌糊涂，比如 [React Router](https://github.com/rackt/react-router)。如果没记错的话 React Router 的作者说过版本号永远和 React 保持一致，从 v0.10 开始确实如此。然后 v0.11，v0.12，v0.13，眼看着时间已经过去了一年多……怎么着也该出 v1.0 了吧，结果 React 发布了 v0.14，而且还规划了 v0.15，让 React Router 的若干个 v1.0-beta 版本哭瞎在厕所。

![react router github repo](http://ww3.sinaimg.cn/bmiddle/831e9385gw1ewt4ti4nevj20c50kqq4s.jpg)

好了，回到 React v0.14 上。

### React 「一分为二」

原本的 `react` package 被拆分为 `react` 及 `react-dom` 两个 package。其中 `react` package 中包含 `React.createElement`、 `.createClass`、 `.Component`， `.PropTypes`， `.Children` 这些 API，而 `react-dom` package 中包含 `ReactDOM.render`、 `.unmountComponentAtNode`、 `.findDOMNode`。

原本在服务端渲染用的两个 API `.renderToString` 和 `.renderToStaticMarkup` 被放在了 `react-dom/server` 中。

改变之后的结构，一个基本的 React 组件变成了这样：

```javascript
var React = require('react');
var ReactDOM = require('react-dom');

var MyComponent = React.createClass({
  render: function() {
    return <div>Hello World</div>;
  }
});

ReactDOM.render(<MyComponent />, node);
```

此外，原本 `React.addons` 下面的工具全部变成了独立的 package：

- react-addons-clone-with-props
- react-addons-create-fragment
- react-addons-css-transition-group
- react-addons-linked-state-mixin
- react-addons-perf
- react-addons-pure-render-mixin
- react-addons-shallow-compare
- react-addons-test-utils
- react-addons-transition-group
- react-addons-update
- ReactDOM.unstable_batchedUpdates （在 `react-dom` 中）

当然，原本的 API 在 v0.14 版中仍然可以使用，只不过会有 warning，最终会在 v0.15 版的时候完全移除。

### refs 变成了真正的 DOM 节点

当我们需要获取 React 组件上某个 DOM 节点时，React 提供了 refs 方法方便我们快速引用。为了方便我们使用，React 还「贴心」地对 refs 做了一层封装，使用 `this.refs.xxx.getDOMNode()` 或 `React.findDOMNode(this.refs.xxx)` 可以获取到真正的 DOM 节点。

结果发现大家真正需要的就是 DOM 节点本身，封装了半天完全是浪费感情。

于是在 v0.14 版中 refs 指向的就是 DOM 节点，同时也会保留 `.getDOMNode()` 方法（带 warning），最终在 v0.15 版中去除该方法。

```javascript
var Zoo = React.createClass({
  render: function() {
    return <div>Giraffe name: <input ref="giraffe" /></div>;
  },
  showName: function() {
    // 之前：
    // var input = this.refs.giraffe.getDOMNode();
    //
    // v0.14 版：
    var input = this.refs.giraffe;
    alert(input.value);
  }
});
```

需要注意的是，如果你给自定义的 React 组件（除了 DOM 自带的标签，如 `div`、`p` 等）添加 refs，表现和行为与之前一致。

### 无状态的函数式组件

其实在实际业务系统中使用 React 时，我们会写很多只有 `render` 方法的 React 组件。为了减少冗余的代码量，React v0.14 中引入了 `无状态的函数式组件（Stateless functional components）` 的概念。先看看长啥样：

```javascript
// 一个 ES6 箭头函数定义的无状态函数式组件
var Aquarium = (props) => {
  var fish = getFish(props.species);
  return <Tank>{fish}</Tank>;
};

// 或者更加简化的版本
var Aquarium = ({species}) => (
  <Tank>
    {getFish(species)}
  </Tank>
);

// 最终使用方式: <Aquarium species="rainbowfish" />
```

可以看到，没有 `React.createClass`，也没有显式的 `render`，写起来更加轻松了。

当然，新语法也有需要注意的地方：

 1. 没有任何生命周期方法，如 `componentDidMount` 等
 2. 不能添加 refs
 3. 可以通过给函数添加属性定义 `propTypes` 和 `defaultProps`

### react-tools 及 JSXTransformer.js 已弃用

拥抱 Babel 吧同学们！

### 编译器优化

在 Babel 5.8.23 及更新的版本中，新增了两项专门针对 React 的优化配置，**仅推荐在生产环境中开启**，因为优化后会导致代码的报错更加扑朔迷离（本来报错就已经很难定位了……）。

 * `optimisation.react.inlineElements` 将 JSX 元素转换为对象而非使用 `React.createElement`
 * `optimisation.react.constantElements` 针对拥有完全静态子树的组件，将其创建过程提升到顶层（Top level），从而减少对 `React.createElement` 方法的调用

### 其它变化

 * `React.initializeTouchEvents` 已弃用
 * 由于 refs 的相关变化（见上文），`TestUtils.findAllInRenderedTree` 及相关的方法不再接受 DOM 组件作为参数，只能传入自定义的 React 组件
 * `props` 一旦创建永远不可修改，因此 `.setProps` 及 `.replaceProps` 已废弃
 * `children` 不可以传对象类型，推荐传入数组，或使用 `React.createFragment` 方法（其实就是转换为了数组）
 * `React.addons.classSet` 已经移除，使用 `classnames` package 替代

### 将要发生的改变

在 v0.15 版中，下列内容将会发生改变：

 * `this.getDOMNode()` 方法将会废弃，推荐使用 `React.findDOMNode()`
 * `setProps` 及 `replaceProps` 将会废弃
 * `React.addons.cloneWithProps` 已废弃，推荐使用 `React.cloneElements`，新方法不会自动 merge `className` 及 `style`
 * `React.addons.CSSTransitionGroup` 将不再监听 transition 事件，因此使用者需要显式指定动画的 timeout，如：`transitionEnterTimeout={500}`。
 * ES6 组件类必须 extends React.Component（如果使用 React.createClass 语法则不受影响）
 * 在多次 render 中重用并改变 style 对象已经被弃用（这一点不是太明白，中心思想貌似是不要 mutate object？）



