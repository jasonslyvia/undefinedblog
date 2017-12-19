---
title: 浅析 Facebook Flux 架构
permalink: facebook-flux
id: 165
updated: '2015-03-11 14:25:15'
date: 2014-11-05 23:48:59
tags:
---

在公司的项目里使用 React 解决 View 层问题的时候，我们让 Backbone 来扮演 Model 层的角色。其实 Facebook 自己也提出了一套完整的解决方案 —— [Flux](http://facebook.github.io/flux/)。不过由于其思路太过新颖，我们没有在生产环境中使用。

项目开发完毕回头看看 Flux，发现它的**单向数据流(unidirectional data flow)**思想还是很值得借鉴的，不过为了实现单向数据流，Flux 会迫使你把代码拆分的非常别扭。

下面就我自己浅显的认识，来分析一下 Flux 架构的运作方式、特点、与传统 MVC 方式的比较。

> 阅读本文需要一定的 React 基础

##Flux 基础概念

首先，Flux 不是一个框架（Framework）或库（Library），而是一种架构（Architecture）。这一点不要搞混，对于理解下面的内容会有所帮助。

其实看完 Flux 官网的 Overview，我还是没能很好的理解 Flux 中几个关键的概念 —— Dispatcher、Action、Store —— 之间的关系。最后我发现，还是直接看源码来的更加清晰。如果你希望能快速理解 Flux，可以先看完 Overview，然后直接看 [TODO 的例子](https://github.com/facebook/flux/tree/master/examples/flux-todomvc/)。

下面是我看完 Flux 两个官方例子（TODO 和 Chat）后总结的 Flux 运作流程图：

![Flux 架构流程图](/content/images/2015/03/Flux--1-.png)

可以看到其中主要有如下几个概念：

 - Controller-View ：整个 App 的入口，监听 Store 变化以获取新的数据，重新 render 自己及子 Component
 - React Component：从 Controller-View 获取 props，渲染自己，同时响应事件
 - Action： 扩展自 Dispatcher，抽象封装 Component 中可能出现的操作，当响应 DOM 事件时调用 Action 中对应的方法
 - Dispatcher： 一个事件分发控制器，通过 register 和 dispatch 方法来处理事件（一般不直接使用，建议直接阅读源码，简单易懂）
 - Store：维护数据，只有 getter 没有 setter，提供一系列 mutate 数据的方法

至于和 Server 端交互的问题，这里不是重点，就不再赘述。

##Flux 运行机制

明确了上述几个概念点后，让我们从实际应用出发，来一步步分解 Flux 的运作方式。

首先当程序开始 bootstrap，先会使用 `React.renderComponent` 方法渲染 Controller-View 到某个 DOM 节点。其实 Controller-View 可以理解为所有组件的入口，类似一般 React 项目中的 `App` 或者 `Home` Component。不同的是，在 React 中**有且只有** Controller-View 负责监听 Model 层（也就是 Store）的变化，同时根据在变化发生后的新数据重新渲染自己及自己的子组件。

上面说到 Controller-View 需要监听 Store 变化，这个过程是通过 Store 暴露的接口实现的。Store 会扩展 node 中的 EventEmitter，以此来实现事件的 `on` 及 `emit`。那么 Store 怎么知道要更新数据呢，这里就要提到 Action。

Action 在上面说了扩展自 Dispatcher，而 Dispatcher 提供了注册监听事件的方法 `register`。因此 Store 在初始化的时候会调用 `Action.register()` 来注册一个回调函数，这个函数当 Action 接收到事件时会触发。

这里需要详细说说这个回调函数。如果你仔细看上面的示意图会发现，Action 指向 Store 的箭头上写了 `dispatch` 并带了一个 `payload`。 dispatch 是 Dispatcher 提供的方法，用于触发一个事件，而 payload 就是这个事件包含的有效信息（大概可以把 payload 理解成传统 DOM 事件中回调函数里的那个 event 对象？）。

通常 Store 在调用 Action.register 注册回调时的函数长这样：

```javascript
function (payload){
  switch (payload.type){
    case 'create':
    /*...*/
    case 'delete':
    /*...*/
    default:
    /*...*/
  }
}
```

那么 Action 传给 Store 的 `payload` 又是哪来的呢？（故事发展到这里终于要产生闭环了……）没错，就是 React Component 中响应各种事件时调用的。

所以一个精简的结构是这样的：

```
React Components --> DOM 事件(payload形如 {type:'click', id: 42}) --> Action(dispatch 方法) --> Store(注册的回调拿到了 payload，开始判断如何 mutate 数据，mutate 完之后触发 change 事件) --> Controller-View(之前监听了 change 事件，发现数据更新了，那重新 render 一下吧) --> React Components(parent 传来的 props 变了，重新 render 一下吧)
```

这样，Flux 所一直倡导的**单向数据流**终于形成了闭环。从 DOM 事件到 dispatch 到 Store 响应事件到触发 change 到 View 监听更新并重新 render 自己。

##Flux 的特点

其实我相信没有仔细研究过 Flux 的同学即使看到这里也不一定完全明白 Flux 有什么特别之处。下面我简单罗列一下 Flux 的独到之处，并与 Backbone + React 的模式进行对比，以便更好的体现 Flux 的特点。

1. Model 层没有 setter，不暴露任何改变数据的接口给外部
2. 只有在最顶层的 Controller-View 响应 Model 的改变，子组件只能接受父组件传来的 props，不单独响应 Model 改变，也不能改变 Model
3. 所有的事件统一到 Action 处理，不同于 Pub/Sub 模式的是，事件本身不区分类型，回调函数通过 Action.register 注册后所有 Action.dispatch(payload) 的事件都会触发回调函数
4. 回调函数根据 payload 中的信息来确定自己是否需要响应这个事件
5. Store 之间的依赖关系可以通过 Dispatcher.waitFor 方法解决

以上是我认为 Flux 不同与其他架构的 4 点内容，下面将 Flux 与 React + Backbone 方式进行简单的对比（求好用的 Markdown table 解决方案……）：

![](/content/images/2014/Nov/QQ20141105-1-2x-2.png)

其中关于依赖需求的解释如下，model.trigger('someEvent')，有两个组件 A、B 都响应 someEvent，我们希望 A 处理完之后 B 再处理。这个时候我能想到的解决方法就是先触发一个 preSomeEvent 的事件，让 A 监听 pre- 事件，B 监听 someEvent。

这一点可能因为我还没有很好的找到解决 Backbone 依赖的处理方法，所以对比仅供参考。(Backbone.Radio 或 Backbone.Wreqr？)


##Flux 小结

通过上面的对比还是可以发现，Flux 通过严格的控制数据的更新来实现单向数据流，这样的好处是你能清晰的掌握数据的改变方式及相应代码的位置。保证自己的程序不会因为业务的发展变得越来越臃肿（试想每个 Component 都能改变数据，这样在一个项目有几十上百个 Component 的情况下每个 Component 都可能改变了数据，那处理起来可就麻烦大了！）

当然，这种模式带来的问题也显而易见，因为数据都在 Store 更新，所有的事件都要定义在 Action 中，开发过程中处理需求就变得不那么灵活。新增一个 DOM 响应事件要同时修改 Component（View层）、Action 和 Store（回调中多判断一个 payload 的 type）。

同时，Store 向 Action 注册回调时通过 switch ... case ... payload 来区分事件会使得 Store 中的代码变得较为臃肿，这与 Backbone 使用的 Pub/Sub 方法有着巨大的差别。

Flux 还存在另一个问题，通常一个程序中不仅仅只有一种数据结构，因此必然会出现多个 Store。那么一个 Component 可能就需要监听多个 Store 的变化。如果某一个 Store 只在 Controller-View 的孙子的孙子的孙子的孙子节点才可能使用到，Flux 模式这种从顶部一层层传下来的方式也不见得会更加高效。（Flux 并不禁止有多个 Controller-View，但是很明显这样违背了 Flux 单向数据流的设计初衷，关于这一点目前似乎还没有很好的解决方法）

总的来说，Flux 看起来是把开发的模式变复杂了，但是通过严格的数据流向限制保证了程序在走向复杂化的过程中（scale）不会失去控制。如果你的程序可能会十分复杂，那么 Flux 也许值得你一试！

PS. 后续会尝试用 Flux 构建一个 Chrome 插件

##参考引用

1.  [Flux 官网](http://facebook.github.io/flux/)
2.  [React 官网](http://facebook.github.io/react/)
3. [代码里的 payload 是什么意思](http://undefinedblog.com/something-about-payload/)
4. [Flux-example](https://github.com/facebook/flux/tree/master/examples/)