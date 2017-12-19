---
title: 正式将博客迁移到GAE 改用Micolog
tags: 

  - gae
  - micolog
  - python
permalink: migrate-to-gae-using-micolog
id: 111
updated: '2012-11-22 23:06:35'
date: 2012-11-22 23:06:35
---

<p>想把博客换到别的主机主要是因为这两天 VPS 经常 502 Bad Gateway，想想也是，在一个 VPS 上放了 5 个 Wordpress，日 PV 5000，对于一个 512 M 的 VPS 来说压力还是有点大。本来准备买个便宜点儿的虚拟主机将就用用，但是突然想到 GAE 每天有免费的 1G 流量。对于我这个每天 IP 不到 100 的小博客来说，GAE 完全够用了。</p>
<p>因此昨晚决定把博客迁到 GAE 上，但是遇到了这么几个问题：</p>
<h2>GAE 不支持 PHP，而博客原来使用的是 Wordpress</h2>
<p>有两种解决方案：更换 Python 博客程序或者在 GAE 上跑 Java 虚拟机来解释 PHP</p>
<p>经过一番挣扎后我选择了前者，虽然两者对我来说都没有接触过，但是想想觉得后者还是麻烦一些。</p>
<p>所以我就搜索到了 Micolog，看起来似乎作者就是照着 Wordpress 做的，因此基本功能上和 Wordpress 没有太多的区别。各种模板文件打开能够看到 Wordpress 主题文件的影子，虽然 Python 语法让我看着头晕&hellip;&hellip;</p>
<h2>一些针对 Micolog 的 Mod</h2>
<ul>
<li>上传好博客程序后，从 Wordpress 导出了所有的文章、页面和评论，然后用 Micolog 原生的一个插件导入，但是遇到一些问题。Wordpress 默认会去掉文章中的 p 标签，这导致文章导入 Micolog 后没有 p 标签，对于板式来说简直就是灾难。</li>
<li>另外 Micolog 默认不支持自动生成摘要，所以在首页显示文章列表时会自动输出全文，这可不好。</li>
<li>主题的样式不是很好看</li>
<li>友情链接不能只在主页显示</li>
</ul>
<p>这些问题今天都被我解决了，抽空慢慢总结放出。</p>
<p>最后比较担心的是伟大的 GFW，虽然 GAE 可以绑定自定义域名，虽然现在还能正确访问。但是不知道哪天 GFW 一不高兴就给墙了，到时候后就得另想办法了！</p>
<p>&nbsp;</p>