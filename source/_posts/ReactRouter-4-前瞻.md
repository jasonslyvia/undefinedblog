---
title: ReactRouter 4 前瞻
permalink: reactrouter-4-foresee
id: 184
updated: '2016-09-18 06:28:09'
date: 2016-09-18 04:44:57
tags:
---

要问用 React 技术栈的前端同学对哪个库的感情最复杂，恐怕非 ReactRouter 莫属了。早在 React 0.x 时代，ReactRouter 就凭借与 React 核心思想一致的声明式 API 获得了大量开发者的喜爱。后续更是并入 reactjs group 并有 React 核心开发成员参与，俨然是 React 官方路由套件一样的存在。

然而从 1.x 版本起，ReactRouter 的洪荒之力就开始慢慢爆发，目前最稳定的版本已经是  `2.8.1`，而下一个正式版本将是 —— 没错，正如你在标题里看到的那样 —— `4.0.0`。先收拾一下你崩溃的心情，让我们来看看这一切到底是怎么回事。

## ReactRouter 3 去哪儿了

从 `2.x` 直接跨入 `4.x`，ReactRouter 的 maintainer 数学还不至于那么差，那这一切都是为什么呢？事实上 `3.x` 版本相比于 `2.x` 并没有引入任何新的特性，只是将 `2.x` 版本中部分废弃 API 的 warning 移除掉而已。按照规划，没有历史包袱的新项目想要使用稳定版的 ReactRouter 时，应该使用 ReactRouter 3.x。

目前 `3.x` 版本也还处于 beta 阶段，不过会先于 `4.x` 版本正式发布。**如果你已经在使用 `2.x` 的版本，那么升级 `3.x` 将不会有任何额外的代码变动。**

## 为什么又要大改 API

ReactRouter 的 API 是出了名的善变，这一点我已经在[之前的博文](https://undefinedblog.com/react-v0-14/)中吐槽过一次了。没想到连 React 都稳定的在 `15.x` 版本中维护了这么久，ReactRouter 还是想要搞点大新闻。

根据 ReactRouter 核心开发者 [ryanflorence](https://github.com/ryanflorence) 的说法，ReactRouter 4.x 之所以要大刀阔斧的修改 API，主要是为了『声明式的可组合性（Declarative Composability）』。怎么理解声明式的可组合性呢？实际上当你在写 React 组件时，就不知不觉的利用了这一特性。

```javascript
function ComponentA() {
  return <div>A</div>;
}

function ComponentB() {
  return <div>B</div>;
}

function Page() {
  return <div><ComponentA /><ComponentB /></div>;
}
```

正如上面的代码片段，在 Page 组件中你可以方便的将任意组件在 render 方法中渲染出来。但是这种特性遇到 ReactRouter 时，你只能把匹配当前路由的组件用`this.props.children`（最早的时候是 `this.props.routerHandler`，有多少人还有印象）渲染 。

此外，ReactRouter 虽然宣称是声明式的路由库，但仍然提供了不少非声明式的 API，其中路由组件的生命周期方法就是最典型的例子。其实我们不难想到，明明用来渲染的 React 组件本身已经有了一套完备的生命周期方法（componentWillMount、componentWillUnmount 等），为什么还需要 ReactRouter 提供的什么 `onEnter` 和 `onLeave` 呢？

当然，这一切都是 ryanflorence 个人的说法，不知道又有多少人真的会为了追求纯正的 declarative 花时间去重构代码兼容 ReactRouter 4 呢？

## ReactRouter 4 有什么变化

虽然目前还是 alpha 版本，但是 ReactRouter 4 的 API 已经有了雏形，其[官方文档](https://react-router-website-xvufzcovng.now.sh/)也有不少带 demo 的例子，下面简单总结一下 ReactRouter 4 的变化。

### 更少的 API

先来看看目前 ReactRouter 2 的 API 列表：

![ReactRouter 2.x API 列表](http://ww3.sinaimg.cn/mw690/831e9385gw1f7xaeadze7j209d0j5t9v.jpg)

再看看 4.x 计划中的 API：

![ReactRouter 4.x API 列表](http://ww3.sinaimg.cn/mw690/831e9385gw1f7xaf8ju4lj204406f3yj.jpg)

直观看起来 API 确实少了不少，但这也意味着使用了被废弃的 API 将会有更大的重构工作量。不，实际上即使用了没有被废弃的 API，也会有很大的重构工作量。但是对于新手来说，学习成本相对会比较低。

### &lt;Router>变身为容器

在目前的 API 中，&lt;Router> 组件的 children 只能是 ReactRouter 提供的各种组件，如 `<Route>、<IndexRoute>、<Redirect>`等。而在 ReactRouter 4 中，你可以将各种组件及标签放进 `<Router>` 组件中，他的角色也更像是 Redux 中的 `<Provider>`。

不同的是 `<Provider>` 是用来保持与 store 的更新，而 `<Router>` 是用来保持与 location 的同步。

以下为新版 ReactRouter 可能的用法：

```javascript
<Router>
  <div>
    <h2>Accounts</h2>
    <ul>
      <li><Link to="/netflix">Netflix</Link></li>
      <li><Link to="/zillow-group">Zillow Group</Link></li>
      <li><Link to="/yahoo">Yahoo</Link></li>
      <li><Link to="/modus-create">Modus Create</Link></li>
    </ul>

    <Match pattern="/:id" component={Child} />
  </div>
</Router>
```

**不过这样的变动将会导致一个很严重的问题：由于路由定义里不再是纯粹的路由相关的组件，你无法在一个文件中（通常是用来定义路由的 `routes.js`）看到整个 App 的路由设计及分布。**

目前 ReactRouter 提供了一个[工具库](https://github.com/ReactTraining/react-router-addons-routes)来解决将完整的路由定义转换为 `<Match>` 组件的问题，但是粗略的看了一下用法感觉还是比较别扭，期待正式发布时能有更好的解决方案。

### 再见 &lt;Route>，你好&lt;Match>

ReactRouter 中被使用最多的 `<Route>` 组件这次没有幸免，取而代之的是 `<Match>` 组件。那么 `<Match>` 究竟有什么不同呢？

1\. `pattern` 取代 `path`

很大程度上这只是为了配合 Match 这个名称的转换而已，实际功能与 path 无异。


```javascript
<Match pattern="/users/:id" component={User}/>
```

2\. `component` 还是 `component`，`components` 没了

判断路由命中时该渲染哪个组件的 props 还是叫 `component`，但是为一个路由同时提供多个命名组件的 props `components` 没有了。下文会详细介绍在 ReactRouter 中如何解决一个路由下渲染多个组件的问题。

3\. 使用 `render` 渲染行内路由组件

能够更自由的控制当前命中组件的渲染，以及可以方便的添加进出的动画。

```javascript
// convenient inline rendering
<Match pattern="/home" render={() => <div>Home</div>}/>

// wrapping/composing
const MatchWithFade = ({ component:Component, ...rest }) => (
  <Match {...rest} render={(matchProps) => (
    <FadeIn>
      <Component {...matchProps}/>
    </FadeIn>
  )}/>
)

<MatchWithFade pattern="/cool" component={Something}/>
```

### &lt;Miss> 取代曾经的 &lt;NotFoundRoute>

早在 0.x 时代，ReactRouter 提供了 `<NotFountRoute>` 来解决渲染 404 页面的问题。不过这一组件在 `1.x` 时就被移除了，你不得不多写若干行代码来解决这个问题。

```javascript
// 如果想要展示一个 404 页面（当前 URL 不变）
<Route path='*' component={My404Component} />

// 如果想要展示一个 404 页面（当前 URL 重定向到 /404 ）
<Route path='/404' component={My404Component} />
<Redirect from='*' to='/404' />
```

在 ReactRouter 4 中，你可以省点儿事直接用 `<Miss>` 组件。

```javascript
const App = () => (
  <Router>
    <Match pattern="/foo"/>
    <Match pattern="/bar"/>
    <Miss component={NoMatch}/>
  </Router>
)

const NoMatch = ({ location }) => (
  <div>Nothing matched {location.pathname}.</div>
)
```

## ReactRouter 4 的大杀器：一个路由，多次匹配

在上文中我们提到过，目前 ReactRouter 匹配到某个路由时，将直接渲染通过 `component` 定义的组件，并把命中的子路由对应的组件作为 `this.props.children` 传入。

在前端 App 日益复杂的今天，一条路由的改变不仅仅简单的是页面的切换，甚至可能精细到页面上某些组件的切换。

让我们直接看下面的代码：

```javascript
// 每条路由有两个组件，一个用于渲染 sidebar，一个用于渲染主页面
const routes = [
  { pattern: '/',
    exactly: true,
    sidebar: () => <div>Home!</div>,
    main: () => <h2>Main</h2>
  },
  { pattern: '/foo',
    sidebar: () => <div>foo!</div>,
    main: () => <h2>Foo</h2>
  },
  { pattern: '/bar',
    sidebar: () => <div>Bar!</div>,
    main: () => <h2>Bar</h2>
  }
]

<Router history={history}>
  <div className="page">
    <div className="sidebar">
      <ul>
        <li><Link to="/">Home</Link></li>
        <li><Link to="/foo">Foo</Link></li>
        <li><Link to="/bar">Bar</Link></li>
      </ul>

      /* 对当前路由进行匹配，并渲染侧边栏 */
      {routes.map((route, index) => (
        <Match
          key={index}
          pattern={route.pattern}
          component={route.sidebar}
          exactly={route.exactly}
        />
      ))}
    </div>

    <div className="main">
      /* 对当前路由进行匹配，并渲染主界面 */
      {routes.map((route, index) => (
        <Match
          key={index}
          pattern={route.pattern}
          component={route.main}
          exactly={route.exactly}
        />
      ))}
    </div>
  </div>
</Router>
```

通过上面的代码片段我们可以看出，`<Match>` 的设计理念完全不同于 `<Route>`。使用 `<Match>` 可以让我们在当前路由中尽情的匹配并渲染需要的组件，无论是需要渲染 Sidebar、BreadCrumb 还是主界面。

若使用现有的 ReactRouter API，只能通过当前路由匹配主界面对应的组件，至于 Sidebar 和 BreadCrumb，只能传入当前的 pathname 手动进行匹配。由此可见 `<Match>` 将会为此类需求带来极大的便利。

## 其他疑惑

当然，还有几个大家普遍关心的问题官方文档中也有所提及：

1\. ReactRouter 的 API 是否还会有大的变化？

答：只要 React 的 API 不变，这一版的 API 也不会变。

2\. 是否有对 Redux 的支持？

答：将提供一个受控的 `<ControlledRouter>`，支持传入一个 location。（作者注：这下真的不需要 react-router-redux 了，撒花！）

3\. 新版是否对滚动位置管理添加支持？

答：将会对 `window` 和独立组件的滚动位置提供管理功能。（作者注：目前在不添加额外配置的情况下，路由切换时并不会恢复滚动位置）


## ReactRouter 4 小结

了解了这么多 ReactRouter 4 的内容，你是否喜欢新版的 API 设计呢？

其实从 ReactRouter 4 的这些变动不难看出，ReactRouter 正在努力的朝纯声明式的 API 方向迈进，一方面减少不必要的命令式 API，一方面也提供更语义化的路由匹配方案。

此外新的 ReactRouter 也提供了非常强大的动态路由能力，甚至对嵌套路由也有所支持，这也是一个复杂的前端应用所必须的功能。

## 参考资料

1. [ReactRouter 4 文档](https://react-router-website-xvufzcovng.now.sh/)
2. [ReactRouter 4 Github repo](https://github.com/ReactTraining/react-router/tree/v4)
3. [ReactRouter](https://github.com/ReactTraining/react-router)