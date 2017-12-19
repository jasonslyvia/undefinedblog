---
title: ReactRouter 使用指南
permalink: react-router
id: 164
updated: '2015-08-19 00:10:23'
date: 2014-09-21 06:47:00
tags:
---

> 本文已过时，推荐阅读最新的 [React Router v0.13.3 版使用指南](http://undefinedblog.com/react-router-0-13-3/)

> 本文已过时，推荐阅读最新的 [React Router v0.13.3 版使用指南](http://undefinedblog.com/react-router-0-13-3/)

> 本文已过时，推荐阅读最新的 [React Router v0.13.3 版使用指南](http://undefinedblog.com/react-router-0-13-3/)

在构建一个复杂的页面时，给不同的操作赋予不同的路由（route）是一个很好的实践。对于习惯Backbone的人来说`Backbone.Router`就是一个很方便的`route-action`映射模块，那么在React中应该怎样增加路由呢？虽然也可以使用Backbone的解决方案，但是ReactRouter或许是一个更符合React思想的路由组件。

ReactRouter（[github地址](https://github.com/rackt/react-router)）是由Ryan Florence开发的应用于ReactJS的路由组件，它通过定义ReactJS组件`<Routes>`及相关子组件来实现页面路由的映射、参数的解析和传递。如果你熟悉`ember.js`，你会发现它处理路由的方式和ember的思想是一致的。

##快速上手


一个最基本的页面，菜单有「图书」和「电影」两个选项，点击「图书」显示图书列表（链接变为/books），点击「电影」显示电影列表（链接变为/movies）。

[DEMO](http://jsfiddle.net/jasonslyvia/t7cuwpma/1/)

```javascript
var ReactRouter = require('react-router');
var Routes = ReactRouter.Routes;
var Route = ReactRouter.Route;

//定义整个页面的路由结构
var routes = (
  <Routes location="hash">
    <Route path="/" handler={App}>
      <Route path="books" name="bookList" handler={Books}/>
      <Route path="movies" name="movieList" handler={Movies}/>
    </Route>
  </Routes>
);
```

这样就定义了包含两条路由规则的路由，一条指向 /books，一条指向 /movies。(严格来说应该是三条路由，还有一个指向 / 的路由)

`<Routes>`组件接受一个名为 location 的 props，这是用来定义用何种前端技术实现路由，可选参数为hash或histroy。hash表示通过改变location.hash更新页面链接，而histroy表示使用history.pushState来更新链接。如果不考虑兼容IE8，可以选择history。

`<Route>`组件是ReactRouter的核心组件，通过它来定义路由对应的路径，处理这条路由的组件（handler）等等。其中path属性是指这条路由对应的路径，比如

```javascript
<Route path="books" name="bookList" handler="Books"/>
```

点击后浏览器中的路径就会变为 `http://你的网址/#/books`。

而handler是指当这条路由被触发时该渲染哪个组件，其中`Books`就是一个React组件。之所以把 name 放在最后讲，是因为 name 是用在使用路由的时候起作用。

上面我们定义了整个页面的路由规则，但总需要点击一个什么链接才能触发这个路由吧，下面我们就来加上这么一个`触发器`，同时看看 name 什么用。

```javascript
//在上述代码的基础上
var Link = ReactRouter.Link;

var App = React.createClass({
  render: function(){
    return (
      <div className="main">
        <nav>
          <Link to="bookList">图书</Link>
          <Link to="moviewList">电影</Link>
        </nav>
        <div className="content">
          <this.props.activeRouteHandler/>
        </div>
      </div>
    );
  }
};

//routes在上述代码中已定义
React.renderComponent(routes, document.body);
```

在上述代码中我们新require了一个React组件`Link`，这就是我们路由的`触发器`，你可以把它理解为高级版的`<a href="#你的hash">链接文字</a>`。注意我们定义的`Link`中有一个 to 属性，这就是对应了之前定义的路由中`<Route>`组件的 name 属性。比如

````javascript
<Link to="bookList">图书</Link>
```

点击后就会触发

```javascript
<Route path="books" name="bookList" handler="Books"/>
```

从而导致`Books`这个React组件被渲染。ReactRouter还做了一个很贴心的功能，当点击Link触发对应的路由后，`Link`本身还会被添加 active 的className，方便你对当前激活的菜单项添加样式。（效果见DEMO，className默认是 active，可以通过传 props 改变）

问题又来了，哪一行代码定义了该渲染哪个组件呢？注意我们的 `App`组件中的`<this.props.activeRouteHandler/>`，这就是ReactRouter为我们自动添加的当前 active 的handler，当点击 `bookList` 时，activeRouteHandler就是Books组件。

另外还需要注意一点，和我们习惯中的React渲染方式不同，最终渲染到DOM上的组件并不是常规的React组件，而是我们定义的`routes`。注意`React.renderComponent(routes, document.body);`
这一行。

[DEMO](http://jsfiddle.net/jasonslyvia/t7cuwpma/1/)

##进阶使用

把玩一下DEMO你会发现，虽然路由起作用了（由于jsfiddle沙盒的限制，路由的改变并没有反映到浏览器的地址栏中），但是在默认状态下除了两个菜单项啥都没有渲染，这和我们的预期不符。

###Redirect

如果你想默认就显示所有图书，即默认渲染`Books`这个React组件，那么你需要引入一个新的ReactRouter组件`Redirect`。

```javascript
var Redirect = ReactRouter.Redirect;

var routes = (
  <Routes location="hash">
    <Route path="/" handler={App}>
      <Route path="books" name="bookList" handler={Books}/>
      <Route path="movies" name="movieList" handler={Movies}/>
      <Redirect to="bookList"/>
    </Route>
  </Routes>
);
```

和`Link`类似，`Redirect`也有 to 属性，这个属性定义了重定向的地址。我们的`routes`更新后，当访问当前页面时（即访问 / 时），会自动跳转到 /books。

[DEMO](http://jsfiddle.net/jasonslyvia/t7cuwpma/2/)

###DefaultRoute

如果你不想默认显示图书或电影，而是另一个组件，比如欢迎信息什么的，可以使用`DefaultRoute`。

```javascript
var DefaultRoute = ReactRouter.DefaultRoute;

var routes = (
  <Routes location="hash">
    <Route path="/" handler={App}>
      <Route path="books" name="bookList" handler={Books}/>
      <Route path="movies" name="movieList" handler={Movies}/>
      <DefaultRoute handler={Welcome}/>
    </Route>
  </Routes>
);
```

这样在访问 / 时，默认会渲染`Welcome`组件，注意这个时候两个菜单项都没有被激活（即没有 active 的className），除非你点击「图书」或「电影」。

[DEMO](http://jsfiddle.net/jasonslyvia/t7cuwpma/3/)

###路由参数及嵌套路由

业务逻辑更复杂了，现在除了点击「图书」显示图书列表，点击「电影」显示电影列表外，点击某一本图书（或某一部电影）还要显示对应的详情。

[DEMO](http://jsfiddle.net/jasonslyvia/t7cuwpma/4/)

首先更新路由对象


```javascript
var routes = (
  <Routes location="hash">
    <Route path="/" handler={App}>
      <Route path="books" name="bookList" handler={Books}>
      	<Route path=":bookId" name="book" handler={Book}/>
      </Route>
      <Route path="movies" name="movieList" handler={Movies}>
      	<Route path=":movieId" name="movie" handler={Movie}/>
      </Route>
      <DefaultRoute handler={Welcome}/>
    </Route>
  </Routes>
);
```

和之前的路由对象类似，需要嵌套的路由直接进行嵌套即可（完全就是字面意思嘛），但是注意到我们的`<Route>`的path变成了一个奇怪的形式，「:bookId」和「:movieId」是什么意思？熟悉`express`的同学应该对这样的路由形式不会陌生，这定义了路由接受的一个参数，简单的说 /moview/:movieId 定义了一个规则，当访问 /moviews/123 的时候，程序会自动把 123 提取出来当做名为 movieId 的参数。

详细的介绍见[路由匹配规则](https://github.com/rackt/react-router/blob/master/docs/guides/path-matching.md)。

既然我们的路由更新了，那么对应的 handler 也应该进行更新。首先给`Books`和`Movies`这两个组件在render时添加`<this.props.activeRouteHandler/>`，然后再新建显示单个图书和单个电影的React组件`Book`及`Movie`。

注意在渲染单个图书或电影的组件中，我们可以通过`this.props.params`来提取通过路由传进来的参数（效果见DEMO）。

另外还要注意一点，正如你在DEMO中看到的，嵌套的路由对应的`Link`对象（即视觉上的菜单项）会全部被添加 active 的className，当你查看某一本书的详情时，书名及图书菜单都处于高亮状态，一条龙高亮。

[DEMO](http://jsfiddle.net/jasonslyvia/t7cuwpma/4/)

###小技巧

这里还有另外一个问题，如果我们想点击某本书的时候不再显示图书列表而是只显示菜单和图书详情该怎么办呢？我们再对路由对象进行小小的改动。

[DEMO](http://jsfiddle.net/jasonslyvia/t7cuwpma/5/)

```javascript
var routes = (
  <Routes location="hash">
    <Route path="/" handler={App}>
      <Route path="books" name="bookList" handler={BookRoute}>
        <Route path=":bookId" name="book" handler={Book}/>
        <DefaultRoute handler={Books}/>
      </Route>
      <Route path="movies" name="movieList" handler={MovieRoute}>
        <Route path=":movieId" name="movie" handler={Movie}/>
        <DefaultRoute handler={Movies}/>
      </Route>
      <DefaultRoute handler={Welcome}/>
    </Route>
  </Routes>
);
```

首先我们修改了原本显示图书列表及电影列表的handler，将它们改为新的React组件`BookRoute`及`MovieRoute`，这两个组件的功能就是渲染 `this.props.activeHandler`。

其次我们将原本的列表当做`<DefaultRoute>`渲染，这样就能实现点击单条信息的时候不再显示列表了。不过这样就需要新增加两个傀儡handler，不知道大家有没有更好的办法实现。

[DEMO](http://jsfiddle.net/jasonslyvia/t7cuwpma/5/)

##ReactRouter小结

以上基本覆盖了SPA中对于路由使用的大多数use case，除了上述功能外，ReactRouter还提供了用于获取当前active路由的mixin `ActiveState`，以及其它实用的工具函数及方法，具体请参考其 [API 文档](https://github.com/rackt/react-router/tree/master/docs/api)。

最后还有个小内幕，React的核心作者之一也参与了ReactRouter的开发，因此看来ReactRouter还是很有潜力的！

> 本文已过时，推荐阅读最新的 [React Router v0.13.3 版使用指南](http://undefinedblog.com/react-router-0-13-3/)

> 本文已过时，推荐阅读最新的 [React Router v0.13.3 版使用指南](http://undefinedblog.com/react-router-0-13-3/)

> 本文已过时，推荐阅读最新的 [React Router v0.13.3 版使用指南](http://undefinedblog.com/react-router-0-13-3/)