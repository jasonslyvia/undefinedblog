---
title: Backbone 源码阅读笔记
permalink: backbone-source-code
id: 166
updated: '2014-12-24 17:48:33'
date: 2014-12-19 20:09:21
tags:
---

一直在用 Backbone，却不明白具体的实现。都说 Backbone 的源码好读，因此特意花了一个下午通读一遍。以下为零星笔记：

###“多此一举”的 ctx

在 Backbone.Events 中绑定事件时，每一个事件的回调由三个部分组成

```
{callback: callback, context: context, ctx: context || this}
```

其中 callback 是用户调用 on 时传入的回调，context 是用户指定的回调调用时的 this，而 ctx 看起来跟 context 值一致，实则不然。

如果用户在使用 on 绑定事件时没有指定 context，在 trigger 时需要给 callback 指定 context，这个时候的 context 用的是上文中的 ctx 而不是 context，因为 ctx 可以保证有值。但是在 off 时，要与用户绑定时的 context 进行对比，所以使用的是 context 而不是 ctx。

###高效的调用函数

在使用 func.call 或 func.apply 调用函数时，使用数量明确的参数比直接传 arguments 效率要高。

Backbone 源码如下：

```
var triggerEvents = function(events, args) {
    var ev, i = -1, l = events.length, a1 = args[0], a2 = args[1], a3 = args[2];
    switch (args.length) {
      case 0: while (++i < l) (ev = events[i]).callback.call(ev.ctx); return;
      case 1: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1); return;
      case 2: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1, a2); return;
      case 3: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1, a2, a3); return;
      default: while (++i < l) (ev = events[i]).callback.apply(ev.ctx, args); return;
    }
  };
```

可以发现其实 switch 中所有的 case 执行的工作和 default 中的工作是一致的，那为什么要这样写呢？一个简单的[性能测试](http://jsperf.com/limited-arguments-vs-arguments)就能看出来。

在需要使用 arguments 时，直接使用形参的效率要比将 arguments 转换为数组的效率要高太多太多。当然这样带来的问题就是 arguments 的个数必须固定，所以才有了上述 case 0、case 1、case 2 等写法。


###详解 set 方法

在 Backbone 中用到最多的可能就是 Model 中的 set 方法了，那么 set 究竟是如何工作的呢？

```javascript
model.set('a', 1, {silent: true});
model.set({a: 1}, {validate: true});
```

以上是 set 方法暴露出的接口，而 Backbone 内部的处理流程大致如下：

1. 判断传入的第一个参数是不是 object，若是则说明第二个参数是对应的 options；若不是则将第一个参数和第二个参数拼装为 object。将这个 object 命名为 `attrs`，方便下文引用。
2. 调用 Model._validate 进行验证（在 _validate 中若 options.validate 为非真值则直接返回 true 而不进行验证）
3. 设置临时状态位 `changing` 为 this._changing 的值，同时设置状态位 `this._changing` 为 true（即初次 set 时 changing 为 undefined 而 this.chaning 为 true）
4. 如果是初次 set，将当前的 attributes 保存到 this._previousAttributes，同时将 this.changed 置为 {}，用来后续判断哪些属性发生了变化
5. 如果 set 的参数中有 key 为 `id` 的值，则 this.id 发生相应的改变
6. 遍历 attrs 中的每一个属性 attr，将它与 this.attributes 中的对应属性进行深度比较（_.isEqual），判断属性是否发生改变，将改变的属性记录在 changes 数组中。同时再将 attr 与 this._previousAttributes 中的对应属性再进行一次深度对比，将改变的属性记录中 this.changed 中，若没有改变则尝试删除 this.changed 中对应的 key
7. 判断 options 中是否有 `{unset: true}` 的设置，若有，则把 this.attributes 中 attrs 里指定的所有 key 都删除
8. 判断 options 中是否有 `{silent: true` 的设置，若没有，则分别触发 changes 数组中的一个 key 对应的 change 事件，同时 `this._pending = options`（后续步骤使用）
9. 判断 `changing` 状态位是否为 true，若是则返回 this（这一步主要是考虑到步骤 8 中触发的 change 事件会再次导致 set 方法的调用，但是在后续的 set 调用中，由于 `this.changing` 已经设为 true（见步骤3）， `changing = this.changing` 也为 true， 所以后续 set 方法走到这一步就会结束） 
10. 判断 options.silent 是否为 true，若不是则将 this._pending 置为 false，同时触发 change 事件，并不断循环，直到 this._pending 为 false 为止
11. 将 this._changing 和 this._pending 设为 false，返回 this，结束

由此可见，一个小小的 set 方法暗藏了不少玄机啊！

*步骤 10 存疑，详见 [stackoverflow 提问](http://stackoverflow.com/questions/27630751/why-there-is-a-while-loop-in-backbones-set-method)

### 零星片段

 - cid 为 Backbone 自动为 Model 或 View 实例生成的 id（`this.cid = _.uniqueId('c')`），主要用于内部标示；id 对应的是 Model 在数据库中储存的 id，Backbone 会根据一个 Model 是否用于 id 来判断这个 Model 是不是新建的；idAttribute 默认值为 `'id'`，用于表示 id 这个字段的 key，比如你的 Model 是表示一本书，那么你的 id 可能叫做 `bookId`，因此可以将 `idAttribute` 设为 `bookId`。
 - Events 中的 all 事件，其实就是在执行 trigger 时不仅把当前要 trigger 的事件回调找出来调用一遍，同时还把监听 all 这个事件的回调找出来调用一遍。因此如果你 trigger('all')，会发现 all 这个事件的回调被执行了两次。
 - Model 中的 collection 属性一般无需手动设置，当你把 Model 添加进 Collection 时 Backbone 会自动帮你把 model.collection 设为当前 Collection。
 - Collection 其实就是 Backbone 自己维护的一个类似数组的数据结构，核心数据是 collection.models，此外提供了一系列类似数组的方法，如 pop、shift 等。
 - 对 Collection 的操作（add、remove、destory）等并不直接触发 Collection 对应的事件，而是通过触发 Collection 中 model 相应的事件，再在 model 中监听 all 事件，最后在 all 事件的回调中触发 Collection 相应的事件。
 - Backbone.View 真的好轻好轻……