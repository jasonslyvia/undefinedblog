---
title: 记一次使用 git bisect 快速定位 bug 的过程
permalink: git-bisect
id: 182
updated: '2016-01-20 00:01:28'
date: 2016-01-19 23:43:57
tags:
---

前一阵子跟三个同事一起合作开发了基于 Redux 的单页应用，我负责的部分完成的比较早，所有功能测试通过之后代码就没有改动过。

结果项目上线后不久接到反馈说我开发的某个功能突然用不了了，我自己一试果然不行。但是自己明明已经做过功能测试，甚至用户也试用过，怎么会突然用不了呢？

因为是一个单页应用，我开始怀疑是别人把我的代码搞坏了。

于是我尝试 checkout 到一个比较早的 commit，发现一切功能正常，所以肯定是这个 commit 一直到 HEAD 之间的某个（或某几个）commit 把功能搞坏了。

这个时候我想起了强大的 git bisect 功能。

不知道为什么，git 的官方文档总是写的让人感觉晦涩难懂，包括 [bisect 的文档](https://git-scm.com/docs/git-bisect)。于是经过一番 Google，我摸索到了核心的使用流程：

首先，尽可能找到第一个出现问题的 commit，记录其 hash 值；然后，尽可能找到最后一个正常的 commit，记录其 hash 值。

然后我们就开始了探案过程：

```bash
# 开始 bisect
$ git bisect start

# 录入正确的 commit
$ git bisect good xxxxxx

# 录入出错的 commit
$ git bisect bad xxxxxx

# 然后 git 开始在出错的 commit 与正确的 commit 之间开始二分查找，这个过程中你需要不断的验证你的应用是否正常
$ git bisect bad
$ git bisect good
$ git bisect good
...

# 直到定位到出错的 commit，退出 bisect
$ git bisect reset
```

简直是查 bug 小能手！