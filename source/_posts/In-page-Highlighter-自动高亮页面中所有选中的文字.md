---
title: In-page Highlighter 自动高亮页面中所有选中的文字
tags: 

  - chrome
permalink: chrome-plugin-in-page-highlighter
id: 144
updated: '2013-09-13 21:28:49'
date: 2013-09-11 00:12:22
---

想写这么一款插件已经很久了，最大的动因是经常看英文文档，对于一些比较陌生的单词，尤其是专有名词，总是需要结合上下文才能理解意思。这个时候如果我选中这个词，页面中所有同样的词都能标出来，我就能快速找到这个专有名词的上下文进行理解了。

其实这个功能很简单，就是 Chrome 自带的搜索功能，但是由于 Chrome 插件并不能直接调用 Chrome 自带的搜索功能，所以我就用简单的 JavaScript 模拟了这一操作。

<a href="https://github.com/jasonslyvia/In-page-Highlighter" target="_blank">English Version</a>
<h2>使用示例</h2>
选择前：

<img alt="inpage highlighter usage" src="https://github-camo.global.ssl.fastly.net/54d3ff98adc30dc3893d7fd06b2566f1dcc46cb0/687474703a2f2f7777342e73696e61696d672e636e2f6d773639302f38333165393338356a7731653868376a6e393374676a323070753064776469652e6a7067" />

选择后：（假设选中了「C89」，通过双击或鼠标选择均可）

<img alt="inpage highlighter usage" src="https://github-camo.global.ssl.fastly.net/a5e87a4ee06575f9c3a345a2f55cc11cd12ab844/687474703a2f2f7777322e73696e61696d672e636e2f6d773639302f38333165393338356a7731653868376a6d317039366a323070713064783431342e6a7067" />
<h2>使用方法</h2>
普通用户：请前往 Chrome 商店下载  <a href="https://chrome.google.com/webstore/detail/in-page-highlighter/jieapbldippnhiccagafbdbhipaaanei" target="_blank">In-page Highlighter</a>

开发者：
<ol>
	<li>前往 <a href="https://github.com/jasonslyvia/In-page-Highlighter" target="_blank">GitHub 页面</a>下载整个项目为 .zip 文档</li>
	<li>将压缩包中的内容解压到任意合适的目录下</li>
	<li>打开 Google Chrome 的设置页，进入「扩展程序」</li>
	<li>确保右上方的的「开发者模式」被勾选</li>
	<li>点击「加载正在开发的扩展程序」，找到之前解压的目录点确定即可</li>
</ol>
<h2>移植说明</h2>
本插件专门为 Google Chrome 编写，不过可以通过简单的修改移植到 Mozilla Firefox 平台。如果你需要将其移植到 Internet Explorer 或其他可能没有完全实现 ECMAScript 特性的浏览器平台，则需要对以下的语句进行修改（在 j.js 中）
<pre class="lang:js decode:true">String.prototype.trim
document.querySelectorAll
document.createTreeWalker
String.prototype.split(RegExp)</pre>
<h2>开发进度</h2>
<ul>
	<li>[x] 增加可选配置</li>
	<li>[x] 增加多国语言</li>
	<li>[  ] 修复已知 BUG</li>
</ul>
<h2>开源协议</h2>
本项目采用 MIT 协议发布