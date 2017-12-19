---
title: 再谈 React Router 使用方法
permalink: react-router-0-13-3
id: 175
updated: '2015-08-19 00:26:48'
date: 2015-08-18 21:17:10
tags:
---

去年 9 月份写了一篇 [ReactRouter 使用指南](http://undefinedblog.com/react-router/)，不小心在百度搜索「react-router」关键词排到了第一名。最近收到很多同学反馈说这篇文章里的例子挂了让我补一下。

其实例子里的代码已经很老了，React Router 的 API 也发生了很多变化。因此今天抽出一晚上的时间，再以最新的 React Router 稳定版（截止 2015年08月18日21:23:40 为 v0.13.3 版，与 React 版本号一致）为基础讲讲如何使用 React Router。

> 阅读本文需要你有一定的 ReactJS 基础，如果你正在寻找 ReactJS 中文入门教程，推荐我参与翻译的 [React - 引领未来的用户界面开发框架](http://book.douban.com/subject/26378583/) 一书。

## 快速上手

一个最基本的页面，菜单有「图书」和「电影」两个菜单项，点击「图书」显示图书列表（链接变为/books），点击「电影」显示电影列表（链接变为/movies）。

[Demo](http://jsbin.com/duduta/14/edit?js,output)

说实话，这个例子并不简单。下面逐步分析一下用到的代码和它们分别是干什么的。

```javascript
var Router = ReactRouter; // 由于是html直接引用的库，所以 ReactRouter 是以全局变量的形式挂在 window 上
var Route = ReactRouter.Route; 
var RouteHandler = ReactRouter.RouteHandler;
var Link = ReactRouter.Link;
var StateMixin = ReactRouter.State;
```

由于 Demo 需要直接从网页上引用 React 和 React Router，因此它们都被挂在了 window 对象上（现在但凡有点追求的前端都上 webpack 了，但是例子的话大家就将就着看吧）。这几行就是获取 ReactRouter 提供的几个组件和 mixin。

接下来声明了 4 个组件，都是最基本的只有 render 方法的 React 组件，分别是：`Movies 电影列表`、`Movie 电影详情`、`Books 图书列表`、`Book 图书详情`。

关于组件唯一需要说明的是用到了 ReactRouter 提供的 `State` 这个 mixin，主要功能就是让组件能够通过 this.getParams() 或 this.getQuery() 等方法获取到当前路由的各种值或参数。

然后是应用的入口，也是我们渲染菜单的地方：

```javascript
// 应用入口
var App = React.createClass({
  render: function() {
    return (
      <div className="app">
        <nav>
          <Link to="movies">电影</Link>
          <Link to="books">图书</Link>
        </nav>
        <section>
          <RouteHandler />
        </section>
      </div>
    );
  }
});
```

这里用到了两个 ReactRouter 提供的组件：`Link` 和 `RouteHandler`。

Link 组件可以认为是 ReactRouter 提供的对 `<a>` 标签进行封装的组件，你可以查看 Link 组件渲染到 DOM 上其实就是 a 标签。它接受的 props 有 to、params 和 query。to 可以是一个路由的名称（下文会讲到），也可以是一个完整的 http 地址（类似 href 属性）；params 和 query 都是这个链接带的参数，下文细讲。

此外，Link 组件还有一个小特点，就是如果这个 Link 组件指向的路由被激活的话，组件会自动添加一个值为 `active` 的 className，方便你对当前激活的菜单项设置不同的样式（注意 demo 中红色的菜单项）。

而 RouteHandler 组件是 ReactRouter 的核心组件之一，它代表着当前路由匹配到的 React 组件。假设当前的路由为 `/books`，那么 App 这个组件里的 RouteHandler 其实就是 Books 组件。

那么这个关系是怎么得来的呢？就要看下面定义的页面路由了。

```javascript
// 定义页面上的路由
var routes = (
  <Route handler={App}>
    <Route name="movies" handler={Movies} />
    <Route name="movie" path="/movie/:id" handler={Movie} />
    <Route name="books" handler={Books} />
    <Route name="book" path="/book/:id" handler={Book} />
  </Route>
);
```

这里又出现了另外一个 ReactRouter 提供的组件 `Route`，这个组件就是定义整个页面路由的基础，可以嵌套。

Route 接受的 props 包括 name、path、handler 等等。其中 name 就是上文提到的路由名称，可以通过 `<Link to="路由的名称">` 来生成一个跳转到该路由的链接。path 指明的是当前路由对应的 url，如果不指定，那么默认就是 `name` 对应的值；如果 `name` 也不指定，那默认是 `/`，即根目录。另外 path 还支持指定 params（上文有提到），就是上面的例子中 `:` 及后面跟着的名称。

>params 和 query 的区别在于，params 定义的是「路由」中的参数，比如 /movies/:id ，params 为 id；而 query 是 「URL」中的参数，是跟在 URL 中「?」后面的。定义路由时一般不考虑也不能限制 query 会是什么。

比如 `<Route name="movies" handler={Movie} />` 就指明了一条指向 `/movies` 的路由，当该路由激活时，调用 `Movies` 这个组件进行渲染。

接下来就是最后一步，根据上面定义的路由判断出当前该渲染哪个组件，并将其渲染到 DOM 中。

```javascript
// 将匹配的路由渲染到 DOM 中
Router.run(routes, Router.HashLocation, function(Root){
  React.render(<Root />, document.body);
});
```

Router 即 ReactRouter，run 方法接受 2 - 3个参数，其中第一个参数必填，即我们指定的路由规则。第二个参数选填，即路由的实现方式，`Router.HashLocation` 指明了当前页面使用 hash 变化来实现路由，反映在浏览器的地址栏中就是类似 `xxx.com/#/movies` 这样的地址。使用这种 Location 的好处是兼容 IE 8，如果你的应用不需要兼容 IE 8，可以使用更高级的 `Router.HistoryLocation`。

最后一个参数是一个回调函数，函数的第一个参数是 ReactRouter 判断出的当前路由中需要渲染的根节点组件，在这里就是 `<App />` 这个组件（虽然名字叫做 Root，但在本例中 Root 指的就是 App）。

最后的最后，就是熟悉的 React API，将 Root 渲染到 DOM 中，看到的结果如下：

![react router demo](http://ww2.sinaimg.cn/bmiddle/831e9385gw1ev76w5qp08j205b01iwea.jpg)

尝试点击一下各种链接看看效果吧！

[Demo](http://jsbin.com/duduta/14/edit?js,output)

## 进阶使用

上面贴了这么多代码分析了这么一大段，只实现了一个非常基础的 ReactRouter 路由例子。你在把玩这个例子的时候应该会发现，页面默认打开的时候（即路由为 `/` 的时候），除了菜单什么也没有显示，显得比较单调。

这个时候你有两种选择：使用 `DefaultRoute` 或 `Redirect`。

如果我们希望页面默认进来的时候除了菜单之外再显示一个类似首页的组件，那么可以用 ReactRouter 提供的 `DefaultRoute`。

[Demo](http://jsbin.com/duduta/15/edit?js,output)

```javascript
// 定义页面上的路由
var routes = (
  <Route handler={App}>
    <Route name="movies" handler={Movies} />
    <Route name="movie" path="/movie/:id" handler={Movie} />
    <Route name="books" handler={Books} />
    <Route name="book" path="/book/:id" handler={Book} />
    <DefaultRoute handler={Index} />
  </Route>
);
```
注意 routes 中定义的路由，多了一个 DefaultRoute，它的 handler 是 Index 组件，即我们希望用户在默认打开页面时看到的组件内容。

如果你不想这么麻烦，想用户进来默认就看到电影列表，则可以使用 Redirect 组件。

[Demo](http://jsbin.com/duduta/18/edit?js)

```javascript
// 定义页面上的路由
var routes = (
  <Route handler={App}>
    <Route name="movies" handler={Movies} />
    <Route name="movie" path="/movie/:id" handler={Movie} />
    <Route name="books" handler={Books} />
    <Route name="book" path="/book/:id" handler={Book} />
    <Redirect to="movies" />
  </Route>
);
```

可以看到，当我们直接访问这个 URL 的时候，自动被重定向到了 `/movies`。Redirect 同时接受 from 这个 props，可以指定当什么规则下才进行重定向。

## 再优化

配合 DefaultRoute 和 Redirect，我们的路由已经基本成型了，但是目前还遇到一个问题：在显示电影详情或图书详情时，对应的菜单项没有高亮。

![react-router-demo](http://ww4.sinaimg.cn/bmiddle/831e9385gw1ev76wlk6anj205r049wei.jpg)


![react-router-demo](http://ww2.sinaimg.cn/bmiddle/831e9385gw1ev76wtyoqoj208c03cmx3.jpg)

这是为什么呢？上文说到了 Link 组件会在指向的路由被激活时自动添加值为 「active」 的 className。而我们的两个 Link 分别指向的是 `/movies` 和 `/books`。而详情页的 URL 是 `/movie/1` 或 `/book/1` 这种形式，显然不满足 Link 被激活的条件。

这个时候又有两个方案可以选择了……「嵌套路由」或是「动态判断」。

### 嵌套路由

让我们对路由进行一些改造：

```javascript
// 定义页面上的路由
var routes = (
  <Route handler={App}>
    <Route name="movies" handler={Movies}>
      <Route name="movie" path=":id" handler={Movie} />
    </Route>
    <Route name="books" handler={Books}>
      <Route name="book" path=":id" handler={Book} />
    </Route>
  </Route>
);
```

同时改造一下图书列表和电影列表组件：

```javascript

/**
 * 图书列表组件
 */
var Books = React.createClass({
  render: function() {
    return (
      <div>
        <ul>
          <li key={1}><Link to="book" params={{id: 1}}>活着</Link></li>
          <li key={2}><Link to="book" params={{id: 2}}>挪威的森林</Link></li>
          <li key={3}><Link to="book" params={{id: 3}}>从你的全世界走过</Link></li>
        </ul>
        <RouteHandler />
      </div>
    );
  }
});
```

注意到首先我们的路由变成了嵌套模式，原本电影列表和电影详情的路由是平级的，一个指向 `/movies`，一个指向 `/movie/:id`。改造之后，路由形成了嵌套关系，电影列表的路由依然是 `/movies`，而电影详情变成了 `/movies/:id`。

[Demo](http://jsbin.com/duduta/20/edit?js)

经过改造之后显示详情的时候菜单可以正确的高亮了，然而列表内容也被显示了出来，很多时候我们并不需要列表和详情同时出现，这个时候就需要另一种路由处理方式：动态判断。

>其实我们的路由从最开始就存在嵌套模式，记得最外层 handler 是 App 的 Route 吗？里面所有的路由都是被嵌套的。

### 动态判断

让我们再次对路由进行改造：

```javascript
// 定义页面上的路由
var routes = (
  <Route handler={App}>
    <Route name="movies" path="/movies/:id?" handler={Movies} />
    <Route name="books" path="/books/:id?" handler={Books} />
  </Route>
);
```

取消了嵌套在列表路由下的详情页路由，同时改造了列表页路由的 path，从原来的 `/movies` 改为 `/movies/:id?`。这样的 path 代表着这条路由匹配所有 `/movies/` 开头的 URL，同时可能存在一个 id 参数，也可能不存在。

看起来路由变简单了不少，不过对应的代价是组件中的代码需要大改。

主要的变动包括：

列表组件中渲染增加逻辑动态判断

```
/**
 * 图书列表组件
 */
var Books = React.createClass({
  mixins: [StateMixin],
  
  render: function() {
    
    var id = this.getParams().id;
    return id ? <Book id={id} /> : (
      <div>
        <ul>
          <li key={1}><Link to="books" params={{id: 1}}>活着</Link></li>
          <li key={2}><Link to="books" params={{id: 2}}>挪威的森林</Link></li>
          <li key={3}><Link to="books" params={{id: 3}}>从你的全世界走过</Link></li>
        </ul>
        <RouteHandler />
      </div>
    );
  }
});
```

详情页组件不再从路由中获取数据，而是根据父组件提供的 props 进行渲染：

```javascript
/**
 * 单本图书组件
 */
var Book = React.createClass({
  render: function() {
    return (
      <article>
       <h1>这里是图书 id 为 {this.props.id} 的详情介绍</h1>
      </article>
    );
  }
});
```

[Demo](http://jsbin.com/duduta/24/edit?js)

## 静态方法

ReactRouter 还提供了 `willTransitionTo` 和 `willTransitionFrom` 两个静态方法，用于某个组件将要被激活和某个组件将要被取消激活时进行拦截或进行相关操作。

由于这两个 API 很可能在 1.0 正式版中被砍掉，这里就不多做介绍了。

## ReactRouter 小结

花了一个晚上的时间把自己一年前挖下的坑松了松土，看到这里你以为自己已经学会 ReactRouter 怎么用了吗？呵呵，偷偷告诉你，ReactRouter 1.0 的 API 变化挺大的（目前已有 1.0 beta3 版）。这个库非常激进，维护的非常频繁，带来的后果就是 API 变动也很频繁。

不管怎么样，拥抱变化吧！

[React Router 官方文档](http://rackt.github.io/react-router/)