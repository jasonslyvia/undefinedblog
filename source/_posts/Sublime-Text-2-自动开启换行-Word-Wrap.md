---
title: Sublime Text 2 自动开启换行 Word Wrap
tags: 

  - sublime text 2
permalink: sublime-text-2-auto-word-wrap
id: 131
updated: '2013-04-25 20:35:58'
date: 2013-01-23 03:57:58
---

首先当然要夸一下神器 Sublime Text 2，自从第一次用我就彻底把神马 Notepad++ 和 TextMate 打入冷宫，用来开发 WEB 项目从此 IDE 都不需要了！

下面讲讲如何自动开启换行功能，当一行代码太长时，Sublime Text 2 默认并不会自动开启换行，你必须从 View-&gt;Word Wrap 开启，虽然只有一步但是能从简就从简。

解决方法是：
<blockquote>打开 Preferences -&gt; Setting - User</blockquote>
添加如下内容即可：
<blockquote>{

"word_wrap" : true

}</blockquote>
下一步计划认真学习 Markdown，重新写一个自己博客，用 Git 发布~