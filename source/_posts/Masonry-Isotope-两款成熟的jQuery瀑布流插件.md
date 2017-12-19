---
title: Masonry Isotope 两款成熟的jQuery瀑布流插件
tags:

  - jquery
  - 插件
  - 瀑布流
permalink: masonry-iostope-jquery-waterfall-plugin
id: 127
updated: '2013-04-25 20:36:53'
date: 2013-01-02 22:44:14
---

在开发一个网站的过程中需要用到瀑布流的样式，虽然用 Javascript 实现瀑布流并不复杂，但是我本无意在前端方向发展，所以本着能偷懒就偷懒的原则，坚决的选择了第三方插件。经过一番搜索，有两款 jQuery 插件比较符合我的要求：<strong>Masonry </strong>和 <strong>Isotope</strong>。其实 Isotope 可以看成是 Masonry 的升级加强版，当然代价是 Isotope 商用是需要购买许可的，而 Mansory 是 MIT Licence。

<a href="http://masonry.desandro.com/">Masonry 主页</a>     <a href="http://isotope.metafizzy.co">Isotope 主页</a>
<h2>Masonry 和 Isotope 有什么不同</h2>
两者都支持将一个 div 中的某一类元素按照瀑布流的样式排列，但是 Isotope 支持更多的功能，其中最重要的是 Filter，也就是根据 class 单独显示某一类的 div。由于我对瀑布流的需求在 Filter 处已经打住，所以没有更深的研究，如果你有兴趣的话可以对比一下二者的 docs，可以看到更多的不同。

二者都支持无限滚动（Infinite Scroll）模式，这对于瀑布流来说是很重要的一个功能。

另外一点就是前面说的 Licence 不同，大家一定要注意，以免日后有纠纷。
<h2>基本用法</h2>
由于 Masonry 和 Isotope 的语法差不太多，我以 Isotope 为例简单的说明怎么使用这款插件

首先当然是加载 jQuery 和 Isotope 了（Masonry 需要 jQuery 1.4 及更新，Isotope 需要 jQuery 1.4.3 及更新）
<h3>html 部分</h3>
<pre class="brush: xml;fontsize: 100; first-line: 1;">&lt;div id="containter"&gt;
    &lt;div class="item happy"&gt;blahblah&lt;/div&gt;
    &lt;div class="item happy"&gt;blahblah&lt;/div&gt;
    &lt;div class="item"&gt;blahblah&lt;/div&gt;
    &lt;div class="item"&gt;blahblah&lt;/div&gt;
    &lt;div class="item"&gt;blahblah&lt;/div&gt;
    &lt;div class="item"&gt;blahblah&lt;/div&gt;
&lt;/div&gt;</pre>
<h3>jQuery 部分</h3>
<pre class="lang:js decode:true brush: bash;fontsize: 100; first-line: 1;">jQuery('#container').imagesLoaded(function(){
    jQuery(this).isotope({
        itemSelector : '.item',
	animationEngine : "jquery"
    });
});</pre>
注意到在调用 isotope 之前我们将其 wrap 进了 imagesLoaded 方法，因为一般来说瀑布流的元素项（class 为 item 的 div）中都会有大量的图片，而 isotope 是根据每个元素相的高度动态安排位置的（即 masonry 模式），如果图片还没有加载完就调用了安排布局的方法，效果就不完美了。

此外看到 isotope 的设置接受的是对象，具体还可以设置哪些参数请参考 isotope 的文档，这里只做一个基本的演示。
<h2>过滤数据 Filter</h2>
<pre class="lang:js decode:true brush: bash;fontsize: 100; first-line: 1; crayon-selected">jQuery('#container').isotope({filter : '.happy'});</pre>
调用以上代码就可以实现只显示 class 为 happy 的层，其它层自动隐藏。由于我们在初始化 isotope 时设置了动画引擎，所以这些操作会自动添加动画，相当方便省心呐！

那么如何恢复显示所有的 div 呢？简单，filter 的值设为空即可， {filter : ''}
<h2>无限滚动 Infinite Scroll</h2>
通过调用 isotope 的 appended 方法来调用（masonry 同样支持）

同时需要 <a href="http://www.infinite-scroll.com/">Infinite Scroll</a> 插件（各种插件= =），当然我个人觉得通过简单的 offsetTop 之类的就可以判断是否加载了

当滑动到页面最底部时用 ajax 请求数据，然后用 appended 的方法插入 container 即可

写完才觉得，这个博客离 <strong>iOS开发</strong> 越来越远了…………
