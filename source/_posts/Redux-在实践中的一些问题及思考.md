---
title: Redux 在实践中的一些问题及思考
permalink: some-thoughts-in-redux
id: 178
updated: '2015-12-13 04:35:49'
date: 2015-12-13 03:04:31
tags:
---

React 绝对是 2015 年前端领域的关键词，基于 React 的 Flux 架构也被越来越多的人所熟识。然而 Flux 作为一套架构思想而不是框架让许多开发者在实践中摸不着头脑，因此社区里也诞生了很多基于 Flux 的「轮子」。而今天要说的就是其中最火、逼格最高的轮子 —— Redux。

本文不是一篇介绍 Redux 的入门文章（如果大家有兴趣的话，后面可以写写），因此阅读本文至少需要以下几点基础：

1. [React](http://facebook.github.io/react/)（熟悉）
2. [Flux](http://facebook.github.io/flux/)（了解）
3. [Redux](https://github.com/rackt/redux)（了解）
4. [ES 6](https://github.com/lukehoban/es6features)（了解）

既然不是基础，那么本文着重要讲的是什么呢？其实是我在实际业务中使用 Redux 遇到的一些问题及我自己的思考。促使我写这篇文章的另一个原因是，知乎上有一位朋友私信咨询了我一些 Redux 相关的问题，让我了解到原来我遇到的问题大家也会有疑惑，因此总结成文，方便后人参考。

本文将会用类似 cookbook 的形式，通过一问一答来尝试解释 Redux 在实际应用中的问题。

### 问题一：一个 action 被 reducer1 处理完之后，希望 reducer2 也对这个 action 做出响应，该怎么处理？

这个问题其实要从两个方向考虑：即 reducer1 和 reducer2 对某个 action 的响应是否有先后顺序的限制。

若没有，则在 reducer1 和 reducer2 的 `swtich..case..` 中针对同一个 `ACTION_TYPE` 做出处理即可。至于 `ACTION_TYPE`，则可以在触发这个 action 的 actionCreator.js 里 export 出来。如

```js
// actionCreator.js
export const SOME_TYPE = 'SOME_ACTION_TYPE';
export function doSomeAction() {
  return {
    type: SOME_TYPE,
    payload: 123
  };
}

// reducer1.js
import { SOME_TYPE } from './actionCreator';

export default function reducer(state, action) {
  switch(action.type) {
    case SOME_TYPE: {
      // 这这个 action 做出响应
    }
  ...
}

// reducer2.js 同 reducer1.js
```

若有先后顺序限制，则比较复杂。我们站在整个应用的角度考虑，其实我们的需求是当触发一个 action 时，希望 reducer1 先响应这个 action 更新 state，然后 reducer2 再响应这个 action，也许还需要基于 reducer1 中最新的 store 来更新自己的 state。

这个时候，可以用我自己写的一个 Redux 中间件 —— [redux-combine-action](https://github.com/jasonslyvia/redux-sequence-action)。使用这种中间件后在 actionCreator 中的逻辑大概是这样的：

```js
// actionCreator.js
export function doSomeAction() {
  return [
    (getState, dispatch) => {
      dispatch({
        type: ACTION_TYPE_1,
        payload: 123
      });
    },
    (getState, dispatch) => {
      const state = getState();
      dispatch({
        type: ACTION_TYPE_2,
        payload: state.some.useful.data.from.state
      });
    }
  ];
}
```

可以看到我们在 actionCreator 中返回了一个函数数组，每个函数都获得了 getState 和 dispatch 方法。在上述代码中，首先会 dispatch 一个 `ACTION_TYPE_1` 的 action，假设会被 reducer1 处理，然后会 dispatch 一个 `ACTION_TYPE_2` 类型的 action，则会被 reducer2 处理。

对于 reducer 来说，并不关系你的业务逻辑是先处理哪一个后处理哪一个，它们只是单纯的响应每一个 action 而已。

至于其中的原理，可以自行查看源代码。（写到这的时候，自觉滚去把这个库的测试补上了）需要说明的是，这个中间件的核心思想基本是参考自 [redux-combine-actions](https://github.com/itsmepetrov/redux-combine-actions) 库。

### 问题二：Redux 中如何实现一个公共业务组件？

一个所谓的「公共业务组件」，应该包含了 UI 和数据两部分。比如淘宝上的收货地址选择控件，应该就包含了三个 `select` 选择器和对应的省市区数据。这样一个地址选择控件的逻辑在各个模块中都是一致的，但是怎么设计才能实现复用呢？

> 一个关于 Flux/Redux 的基础概念：任何一个 actionCreator 触发的 action 都会通知到所有的 store/reducer。

考虑一种极端情况，页面上同时有两个模块都使用了这个公共的地址选择组件。当在这个组件里选择了一个省份时，会 dispatch 一个 `SELECT_PROVINCE` 的 action，那么两个模块怎么知道用户到底是选择了哪个模块里公共组件的省份呢？

再考虑另外一个问题，当用户选择了一个省份后，某一个业务模块希望能够再做一些别的数据处理（比如根据当前选择的省份加载对应的热销宝贝）该怎么操作呢？总不能在 reducer 中 dispatch 一个 action 吧？（**注意，这是 anti-pattern，不要在 reducer 中 dispatch action！！**）

其实这两个问题的解决思路是类似的，就是在公共组件中，数据传递的过程中加上更多的限制，我能想到的有这么两种解决方案：

 - 公共组件自己维护状态，业务模块通过给公共组件传入回调来获得公共组件的数据变动，再 dispatch 响应的 action 更新自己的 state。这样设计的问题在于，违背了 Redux 倡导的无状态原则，公共组件内部存在了 state 和 setState 操作。
 - 通过 props 传入指定的 actionType 或 actionCreator（传入 actionCreator 是我同事奇阳的想法）来让公共组件 dispatch 不同的 action，以此达到区分的效果。

一个简单的例子：
```js
// 公共模块自己定义的 actionCreator
import { provinceActionCreator } from './actionCreator';

// 一个公共的地址选择组件
const AddressPicker = React.createClass({
  handleChangeProvince(e) {
    // 优先使用 props 中传入的 actionCreator
    const provinceActionCreator = this.props.provinceActionCreator || provinceActionCreator;
    // dispatch 一个选择省份的 action
    this.props.dispatch(provinceActionCreator(e.target.value));
  },

  render() {
    return (
      <select onChange={this.handleChangeProvince}>
        <option>北京</option>
        <option>浙江</option>
      </select>
    );
  }
});
```

### 问题三：Redux 中跟路由有关的状态应该怎么维护？

现在用 React 做路由的话大家基本都会选择 react-router（这个库我已经吐槽过无数次了，经常改 API），而用 Redux 的话自然也是选择基于 react-router 封装的 redux-router。

> 在讲详细的问题之前，先要阐明一个观点，即「路由也是应用状态的一部分」，应该也的确有一个单独的 store 在维护它。

既然有一个单独的 store，那么其中的状态就能轻松的获取和设置了，具体的操作方法可以参考 redux-router 中的 API。

如果抛开 redux-router 不谈，react-router 中针对路由切换时的数据传递提供了一种全新的思路，即在切换路由时，将「state」存入 session-storage 中，借用这个功能你就可以轻松的实现「回退」操作了。

详细说明见 [history 模块的文档](https://github.com/rackt/history/blob/master/docs/Glossary.md#locationstate)。

### 小结

总的来说，使用 Redux 开发业务功能还是很爽的，尤其是对于数据流动复杂的应用，单一的数据流动、类 Immutable 的数据变动方式以及酷炫的 devtool 这些特性简直堪称神器。

