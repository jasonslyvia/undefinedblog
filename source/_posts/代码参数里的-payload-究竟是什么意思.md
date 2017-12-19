---
title: 代码参数里的 payload 究竟是什么意思
permalink: something-about-payload
id: 161
updated: '2015-02-02 17:23:56'
date: 2014-08-31 23:59:33
tags:
---

好久没写东西了，趁着今天把 Ghost 升级到 0.5，顺便解决一个一直以来困扰我的问题：代码里的 `payload` 究竟是什么东西。

随便在 Github 搜 payload，就能获得成千上万的代码（[https://github.com/search?p=1&q=payload&type=Code&utf8=✓](https://github.com/search?p=1&q=payload&type=Code&utf8=✓)），其中以C和C++语言居多。

根据词典里的解释，payload指的是

1.有效载重
2. 负载
3. 人事费
4. 弹头内的炸药
5. 火箭所载弹头


Wat?? 完全不能代入到代码里去理解啊，这些参数难道是计算弹头内的炸药含量的么？在一番Google之后，终于在 stackexchange 找到了一份还算靠谱的[答案](http://programmers.stackexchange.com/questions/158603/what-does-the-term-payload-mean-in-programming)。

首先解释一下什么是 payload，payload 在这里却是可以理解为`有效载重`，但是这只是字面意思。对于程序员来说，有效载重究竟是个什么玩意儿，又是一个新的问题（调用栈又多了一层……）。

要解释什么是有效载重，就得说到货运行业。比如有一位客户需要支付一笔费用委托货车司机运送一车石油，石油本身的重量、车子的重量、司机的重量等等，这些都属于`载重(load)`。但是对于该客户来说，他关心的只有石油的重量，所以石油的重量是`有效载重(pay-load，也就是付费的重量)`。

**所以抽象一下，payload 可以理解为一系列信息中最为关键的信息。**

回到代码中，举一个最简单的例子，一个 ajax 请求返回一个 JSON 格式的对象

```json
{
    status: 200,
    hasError: false,
    data: {
        userId: 1,
        name: 'undefined'
    }
}
```

这里的 `data` 就是 payload，也就是关键信息。而 `status`、`hasError`等信息是`load`，虽然也是信息，但相对没有那么重要。

说到这里感觉`load`和`payload`的关系有点像`信息`与`元信息`的关系，感兴趣的同学可以看看知乎上一篇讲`meta`的[文章](http://www.zhihu.com/question/22463701/answer/22004638)，很有启发性。