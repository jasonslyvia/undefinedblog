---
title: 在Micolog中使用多说
tags: 

  - duoshuo
  - micolog
permalink: using-duoshuo-in-micolog
id: 119
updated: '2012-11-29 19:42:19'
date: 2012-11-29 19:42:19
---

<p>多说是一个很好的社会化评论系统，由于 Micolog 自身的评论功能实在太烂，所以一狠心直接换了多说。以前用多说都是在 Wordpress 平台中，直接装插件就可以，那如何在 Micolog 中使用多说呢？</p>
<p>首先你需要去多说申请一个新的站点 &nbsp; <a href="http://duoshuo.com/create-site/" target="_blank">点这里申请</a></p>
<p>然后将获取到的代码放到主题的 comment.html 中，注意进行如下的修改：</p>
<p>在评论框的显示层中，添加 data-thread-key 等多项参数，可以让多说更好的整合进 Micolog，更智能的识别文章和分页。<br />
&nbsp;</p>
<pre class="brush: xml;fontsize: 100; first-line: 1; ">&lt;!-- Duoshuo Comment BEGIN --&gt;
&lt;div class="ds-thread" data-thread-key="{{entry.key}}" data-title="{{entry.title}}" data-author-key="{{entry.author}}" data-category="{{category.name}}" data-url="{{ entry.link }}"&gt;&lt;/div&gt;
&lt;script type="text/javascript"&gt;
var duoshuoQuery = {short_name:"devppios"};
(function() {
	var ds = document.createElement('script');
	ds.type = 'text/javascript';ds.async = true;
	ds.src = 'http://static.duoshuo.com/embed.js';
	ds.charset = 'UTF-8';
	(document.getElementsByTagName('head')[0]
	|| document.getElementsByTagName('body')[0]).appendChild(ds);
})();
&lt;/script&gt;
&lt;!-- Duoshuo Comment END --&gt;</pre>
<p>&nbsp;<br />
使用以上代码完全替换掉 comment.html 中原有的所有代码即可。</p>
