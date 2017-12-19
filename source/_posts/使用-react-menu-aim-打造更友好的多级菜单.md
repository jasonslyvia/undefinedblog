---
title: 使用 react-menu-aim 打造更友好的多级菜单
permalink: react-menu-aim
id: 173
updated: '2015-07-16 15:02:34'
date: 2015-07-12 02:59:44
tags:
---

写过多级菜单的同学应该都知道当年亚马逊的黑科技：鼠标从一级菜单滑向二级菜单时，如果中间经过了另一个一级菜单，并不会马上切换。这也避免了用户想看二级菜单的时候，必须先精准的横向移动到对应二级菜单的不便。

详细的原理在[这篇博文](http://bjk5.com/post/44698559168/breaking-down-amazons-mega-dropdown) 里都有解释，本文不再赘述。简单地说，就是在一级菜单项 mouseenter 的时候，根据鼠标之前的移动坐标，判断用户当前是否是想移动到二级菜单。若是，则 setTimeout 一段时间，再重新判断；若不是，则直接激活当前一级菜单项。

一图胜千言：

![react-menu-aim](https://cloud.githubusercontent.com/assets/1336484/8591773/198d1d4a-265d-11e5-94b1-97071a591ab1.gif)

[Demo](http://jasonslyvia.github.io/react-menu-aim/demo/index.html)

目前已经有人根据这个原理写了一个 jQuery 插件 [jQuery-menu-aim](https://github.com/kamens/jQuery-menu-aim/)，而我由于在最近的 React 项目中需要使用这个功能，就顺手写了一个 React 版本。

使用的方法也很简单，引入 [react-menu-aim](https://github.com/jasonslyvia/react-menu-aim) 作为一个 mixin，然后在 `componentWillMount` 的时候配置一下，最后在 render 的时候调用 mixin 提供的一些事件响应方法即可。

```
var React = require('react');
var ReactMenuAim = require('react-menu-aim');

var Menu = React.createClass({
  // 使用 ReactMenuAim mixin
  mixins: [ReactMenuAim],

  componentWillMount: function() {
    // 在这里配置 ReactMenuAim
    this.initMenuAim({
      submenuDirection: 'right',
      menuSelector: '.menu'
    });
  },

  // 由于 ReactMenuAim 需要响应组件的 mouseenter 等事件，所以若你也需要
  // 响应这些事件，可以把自己的 event handler 作为参数传给 ReactMenuAim
  // 提供的 event handler。
  //
  // 例如下面就是组件自己的 event handler
  handleSwitchMenuIndex: function(index) {
    // ...
  },


  // this.handleMouseLeaveMenu 和 this.handleMouseEnterRow 是 
  // ReactMenuAim 提供的方法，让它们分别响应鼠标离开菜单和鼠标进入每一个
  // 一级菜单选项的事件。
  //
  // 若 ReactMenuAim 判定应该激活当前 mouseenter 的一级菜单时，会调用
  // 传入的回调。在这个例子里，就是我们自己的 this.handleSwitchMenuIndex 方法
  render: function() {
    return (
      <div className="menu-container">
        <ul className="menu" onMouseLeave={this.handleMouseLeaveMenu}>
          <li className="menu-item" onMouseEnter={this.handleMouseEnterRow.bind(this, 0, this.handleSwitchMenuIndex)}>Menu Item 1</li>
          <li className="menu-item" onMouseEnter={this.handleMouseEnterRow.bind(this, 1, this.handleSwitchMenuIndex)}>Menu Item 1</li>
        </ul>
      </div>
    );
  }
});
```

更多的配置项见 [github repo](https://github.com/jasonslyvia/react-menu-aim)。
