---
title: 解决Wordpress comment-page- 导致的 description 重复问题
tags: 

  - seo
  - wordpress
permalink: jie-jue-wordpress-comment-page-dao-zhi-de-description-zhong-fu-wen-ti
id: 96
updated: '2012-08-30 20:39:29'
date: 2012-08-30 20:39:29
---

<p>今天用 Google Webmaster 检查站点信息时发现很多页面 description 信息重复的提示。</p>
<p>可以看到，是一个文章页和一个文章页的 comment-page 页重复了，访问这个 comment-page 页，发现跟原来的文章页没有任何区别。重复的页面对搜索引擎来说就是灾难！这个 comment-page 究竟是干嘛的呢？</p>
<p>原来只是 Wordpress 在后台的【讨论】设置里，有一项【分页显示评论】，就是这个选项让 Wordpress 产生了这些 comment-page。又因为我的站点已经启用了社会化评论多说插件，根本不存在评论需要在不同页面显示的问题，所以果断取消选中这一项。</p>
<p>这虽然解决了最根本的问题，但是搜索引擎的反应总是慢一拍，所以我们需要把这些 comment-page 重定向到对应的文章页。</p>
<p>由于我启用了 Redirection 插件，所以就直接在插件界面设置了，其它遇到这个问题的朋友也可以在 Apache 设置 .htaccess 或 Nginx 设置 .conf</p>
<p><del>其中应该把【原始URL】中的 "/i" 去掉，这是我测试的结果。</del></p>
<p>更新，转向规则为</p>
<blockquote>
<p>&nbsp;/(.+?).html(/comment-page-d) &nbsp;=&gt; $1</p>
<p>其中 .html 为我设置的自定义固定链接后缀</p>
</blockquote>
<p>至此，应该就能完美解决因为 comment-page 导致重复页面和 description 的问题了！</p>