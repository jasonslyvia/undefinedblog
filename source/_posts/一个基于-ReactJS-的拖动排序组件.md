---
title: 一个基于 ReactJS 的拖动排序组件
permalink: build-react-anything-sortable
id: 163
updated: '2015-02-02 17:23:35'
date: 2014-09-15 19:37:34
tags:
---

最近由于工作上的需要接触了Facebook的前端框架[React.js](http://facebook.github.io/react/)，简单的了解并上手使用后感觉对于非SPA但前端逻辑依然复杂的应用来说，React绝对是不二的选择。

下面就一个稍微复杂的一点的实际应用例子来演示ReactJS的使用方法。

[react-anything-sortable项目完整源代码](https://github.com/jasonslyvia/react-anything-sortable)

##需求
1. 实现一个可以拖动排序的React组件，被拖动排序的元素作为 `this.props.children`传入，其内部属性及操作不影响拖动排序的进行
2. 兼容IE 8，即无法使用 HTML5 `draggable` 属性
3. 拖动结束后返回排序正确的数据

效果图

![](https://raw.githubusercontent.com/jasonslyvia/react-anything-sortable/master/demo.gif)

##思路分析

首先，由于要兼容 IE 8，方便的 HTML5 `draggable`第一个就被 ban 掉了，那么实现的方法只能是`mousedown`、`mousemove`和`mouseup`事件模拟。

其次，由于不能在拖动排序组件内部渲染被拖动的元素，而是这些元素作为 children 传入，因此要在 children 的 render 过程中加以干涉，否则无法改变 children 的样式。

又因为`this.props.children`是可读的，父元素无法改变其顺序，因此在拖动排序组件中我们需要手动维护一个子元素的顺序数组。同时考虑到子元素的尺寸不一，我们需要记录每一个子元素的相对坐标（相对于最直接的定位父元素）和尺寸（长宽），用于在拖动时计算目标位置。

##初始化操作

分析完成后，我确定需要至少两个 React Component 才能完成这个需求，一个是拖动组件 `Sortable`，一个是用于 mixin 到被拖动的元素中的 `SortableItemMixin`。

`Sortable` 中主要用于维护每一个 children 的各种信息，同时监听 `mousemove`及`mouseup`事件来处理拖动的过程和松开鼠标完成拖动的过程。为什么不监听`mousedown`？这个事件交给被拖动的子元素来完成，`Sortable`中仅负责处理`mousedown`的响应。

`SortableItemMixin`中主要定义了一些生命周期函数，在 mount 完成后将自己的坐标及尺寸“反馈”给父元素，同时在 render 时根据父元素指定的一些 props 渲染自身。

##核心技术

在拖动的过程中，其实 DOM 中是多了一个“被拖动”的元素，这个元素并不在 `this.props.children`中，而是我们利用 `React.addons.cloneWithProps`虚构出来的一个子元素。在拖动开始时，我们根据当前被拖动的元素的索引值利用 `cloneWithProps`新建一个“虚拟元素”并交由 render 进行渲染。

同时，被点击的原始元素加上了`ui-sortable-placeholder`的class成为一个placeholder。在鼠标移动的过程中不断更新这个虚拟元素的 top、left值，同时实时计算当前鼠标的位置是否已经达到与相邻元素交换位置的程度，如果是则交换placeholder与目标元素的位置。

当鼠标松开时，不再渲染这个虚拟元素，同时取消placeholder的class，使其成为一个正常的元素。这样，就完成了一次排序。

至于最后返回排序后的数据，首先要求被拖动排序的元素都要指定一个 `sortData` 的 props，当 `mouseup` 触发时，`Sortable` 会根据自己维护的一个新的、排序后的顺序数组依次取出子元素中的 `sortData`，并通过回调返还给使用者。

##待优化
1. 使用了 jQuery 进行 document 上的事件绑定，主要是因为如果将 `mousemove`事件绑定在被拖动的元素本身，一旦鼠标移动过快就会丢失焦点，鼠标会移出被拖动的元素，最终导致`mousemove`事件不再响应。使用 jQuery 绑定事件不符合 React 的设计哲学，然而这里没有更好的办法解决。
2. 对于被拖动的子元素自身需要被销毁，需要依赖使用`Sortable`的父组件重新用新的 props 渲染 `Sortable` 解决。

如果你对这个项目感兴趣，可以阅读
[react-anything-sortable项目完整源代码](https://github.com/jasonslyvia/react-anything-sortable)，或者直接通过 npm 使用它。

````
npm install react-anything-sortable --save-dev
```