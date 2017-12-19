---
title: Angular2 中那些我看不懂的地方
permalink: angular-the-confusing-part
id: 183
updated: '2016-05-01 20:01:10'
date: 2016-04-17 10:17:56
tags:
---

博客停更了近 3 个月，实在是愧对很多在微博上推荐的同学。因为最近大部分时间都投入在公司里一个比较复杂的项目中，直到本周才算正式发布，稍得解脱。

说这个项目复杂，不仅是因为需求设计复杂，更是因为在这个项目里我们使用了很多新技术 —— Angular2（是的，beta 版），Webpack2（是的，beta 版），RxJS（这两个 [RxJS1](https://github.com/Reactive-Extensions/RxJS)、[RxJS2](https://github.com/ReactiveX/rxjs) 傻傻分不清楚）还有 TypeScript。

关于为什么一个大型线上项目会选择这么冒进的技术选型这里不多做评价，只能说最终团队还是成功将整个产品发布到了线上，我也对 Angular2 这个全新的 Angular 有了更全面的认识。

这篇文章不想对比 React 和 Angular2（不要急，这些东西肯定会出现在项目总结里），而是想站在熟悉 React + Redux 开发模式的前端工程师角度说一说 Angular2，尤其是那些我不太理解的地方。

剧透：这篇文章无意引起骂战，因此如果你看到我哪里写的不对，请直接在评论区里纠正。

## 看不懂的依赖注入

我知道这其实不仅仅针对 Angular2，早在 Angular1 中就存在依赖注入的概念，甚至这还是 Angular 引以为傲的特性。每当谈到依赖注入的时候，我就看到一帮 OOP（以 Java 为首）爱好者眼里闪烁着奇异的光芒。

然而依赖注入的问题一直困扰着我，倒不是说不理解依赖注入的使用方法或是原理，而是不理解**为什么我的代码里需要依赖注入？**

当我把这个问题抛给项目组中强推 Angular 的同学时，他也一时语塞。

在 Angular2 中，一个常见的依赖注入是注入一个 Http 对象，用于发送请求。

```javascript
import {Component, OnInit} from 'angular2/core';
import {Http} from 'angular2/http';

@Component({
  selector: 'test',
  providers: [Http]
})
class Test implements OnInit {
  constructor(private http: Http) {
  }

  ngOnInit() {
    this.http.get('/api/user.json').map(res => res.json()).subscribe(result => {
      // 这里拿到请求的结果
    });
  }
}
```

上述代码是在 Angular 中使用内置 Http 模块发送请求的一段示例代码。具体依赖注入发生在 constructor 函数中。按照 Angular 宣传的那样，你只用简单的在 constructor 中声明这些参数，Angular 将自动为你处理依赖注入的问题。

注意到我们要使用 Http 来发请求，首先需要从 `angular2/http` 中将 Http import 进来，然后需要在组件的配置中添加对应的 providers，最后还需要在 constructor 中声明一个参数。

这个时候我其实是有点懵逼的，因为在一般应用里，发请求大概是这样的。

```javascript
var request = require('superagent');

function getUser() {
  return request.get('/api/user.json').end((err, res) => {
    // 这里是拿到请求的结果
  });
}
```

为了公平起见，我还是假装依赖了一个第三方的库来实现 Ajax 请求。在这份代码里没有什么高大上的依赖注入，更没有什么 constructor，只有简单的引入并使用。

所以我不太明白，Angular 中的依赖注入究竟在多大程度上发挥着作用？

### 语法糖、语法糖和语法糖

很多人在第一眼看到 React 的时候，都在叫嚣 JSX 是「在 JS 里嵌入 HTML」，是邪门歪道，吃枣药丸。我只想说，相比于 Angular2 里的这些语法糖，JSX 简直就是 JavaScript 里的纯情小处男。

以下引用部分 Angular2 文档中关于模板语法的部分：

> Expanding *ngFor
>
> The *ngFor undergoes a similar transformation. We begin with an *ngFor example:
>
> `&lt;hero-detail *ngFor="#hero of heroes; trackBy:trackByHeroes" [hero]="hero"></hero-detail>`
>
Here's the same example after transporting the ngFor to the template directive:
>
>
> `&lt;hero-detail template="ngFor #hero of heroes; trackBy:trackByHeroes" [hero]="hero"></hero-detail>`
>
> And here it is expanded further into a &lt;template> tag wrapping the original &lt;hero-detail> element:
>
>`&lt;template ngFor #hero [ngForOf]="heroes" [ngForTrackBy]="trackByHeroes">
  <hero-detail [hero]="hero"></hero-detail>
&lt;/template>`

读完之后你会发现，`*ngFor` 这个指令，原来是个语法糖。好嘛，其实语法糖在很多框架设计里都有啊。但是当你展开这个语法糖之后，你会发现这个语法糖其实还是另一个语法的语法糖。这也就是说，`*ngFor` 是语法糖的语法糖。

这种例子在 Angular2 中还有不少，任何一个正常的 JavaScript 开发者如果不想把自己逼疯的话，都会喜欢一个原始的 for 循环，或者一个 `[].map` 吧……

看到某篇文章争论说 Angular2 的模板用的是 「Valid HTML」，我只想说管你 Valid 不 Valid，臣妾看不懂啊。

### 模板里面到底发生了什么

其实刚开始写 Angular 应用的时候，心里还是对双向绑定挺期待的。因为在 React + Redux 架构里面，表单处理永远都是一个痛点（这里安利一下自己写的 [redux-form-utils](https://github.com/jasonslyvia/redux-form-utils) 良心工具，线上产品背书）。但是等我真的在写 Component 中的 template 时，就开始慢慢崩溃了。

首先最崩溃的是，我没办法在模板中断点调试（还是我没有找到正确的方法），这导致在一些模板中存在复杂逻辑的场景我只能默默的在心里推倒运行的过程。

然后是在模板里面，你还是可以写大量的逻辑，比如这样

```javascript
@Component({
  template: `
    <ul #listEl [class.rendered]="listEl.rendered">
      <li *ngFor="#item of list; if(last) {listEl.rendered = true;}" (click)="list.push(Math.random()); $event.stopPropagation(); $event.preventDefault(); // Do other fancy stuff">
        {{ item.text }}
      </li>
    </ul>
  `,
})
```

这些在模板中的代码都是完全合法的 Angular2 语法。有人可能觉得，允许你这么写不代表你一定要这么写。事实上，我已开始很多组件都是这么写的，因为很快，因为网上很多 demo 也这么写，因为不知道该怎么更好的组织代码。

这样下去，一个 Angular 组件中，你不仅要阅读组件本身的各种对数据的处理，还要看模板中的各种逻辑。尤其是一些比较复杂的组件，模板和组件本身放在不同的文件中时，这种来回上下文的切换真是酸爽。

### 组件通信操碎了心

在 Angular 中，父组件想要传递一些信息给子组件，还是比较简单的，直接使用属性绑定即可。在子组件中稍微麻烦一点，多声明一个 @Input。

> 这里还想吐槽两句，一个组件想要引用另一个组件，要先 import 进来，然后还要加到当前组件 Component 配置的 directives 属性中。开发过程中好多次莫名其妙的报错都是因为忘了这一步……

```javascript
// Parent.js
@Component({
  selector: 'parent',
  template: `
    <h1>Parent</h1>
    <child [data]="data"></child>
  `
})
class Parent {
  data = 'react';
}

// Child.js
@Component({
  selector: 'child',
  template: `
    <h4>Child</h4>
    <p>Data from parent: {{ data }}
  `
})
class Child {
  @Input() data;
}
```

这个时候问题来了，当子组件发生某些变化父组件想要知道的时候，你有这么几种选择。

1. 在子组件中定一个 @Output 属性，然后父组件用 (event)="handler()" 的语法来监听这个事件
2. 将父组件的一个 event handler 当做属性传给子组件，子组件通过调用这个方法来通知父组件（在 Redux 里，这很常见）
3. 设计一个 Service 然后分别注入到父组件和子组件之中进行通信

在我开发这个项目的时候，Angular2 的官方文档中还没有任何教程说明组件之间沟通该如何进行（现在[有了](https://angular.io/docs/ts/latest/cookbook/component-communication.html#!#parent-to-child)），所以我果断选择了最 Redux 的那种。

结果发现，后来更新的官方文档里明确说明了不提倡使用这样的方式。

我只能说很心累……

### 其它

奇葩的路由设定，用 `/...` 定义一个非最终页面路由，以及没有一个全局的路由结构，让 Angular2 里面的路由变得非常难推导。如果你不一层一层跟踪下去，根本就无法了解整个 Angular2 应用的路由结构。

莫名其妙的报错，而且很多错误都 Google 不到答案，只能 Google 到一群绝望的开发者提出绝望的问题。看看 Github 上 Angular Repo 里那些绝望的 issue，你就知道后悔当初自己为什么要选 Angular2 作为线上项目的架构了。我自己 Watch 了很多遇到的问题，然而收到最多的更新都是「+1」。

复杂的生命周期，官方文档看了一遍又一遍，网上的例子搜了很多，又不能确定是不是最新的 API。至今没有全部弄明白 Angular2 所有的生命周期都是干嘛用的，只用过几个简单的 OnInt、AfterViewInit 和 OnChanges。

### 小结

写着写着快要变成吐槽 Angular2 了，其实 Angular2 有很多设计的非常不错的地方值得其它框架学习。比如内置的 ViewEncapsulation 解决了 CSS 冲突的问题，完善的 NgForm 系列表单指令能够高效的解决表单校验和提交（前提是那些语法你都学会），底层设计与 DOM 无关以便运行在 WebWorker、服务器等不同环境等等。

不管怎么样，从项目开始的 Angular2 Alpha，用到现在的 Angular2 Beta 15，学习到了很多新知识，但学到更多的还是自己原来还有这么多不知道。

后面还是会投入更多的精力在 Redux 生态建设上，毕竟团队内 React + Redux 已经全面开花结果，有必要寻找到 Redux 最优的开发实践。
