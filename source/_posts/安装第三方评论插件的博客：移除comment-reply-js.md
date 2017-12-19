---
title: 安装第三方评论插件的博客：移除comment-reply.js
tags: 

  - wordpress
permalink: remove-comment-reply
id: 133
updated: '2013-04-25 20:35:34'
date: 2013-02-01 00:28:18
---

对于网站来说，加载越少的东西越好，HTTP 请求数越少越好。如果你的博客已经安装了 DISQUS 、多说等第三方评论插件，则 Wordpress 自带的 comment-reply.js 将没有用。使用以下代码禁止系统加载它：
<pre class="lang:php decode:true brush: php;fontsize: 100; first-line: 1;">// 将如下代码放置到主题目录下的 functions.php 中即可
function clean_header(){
	wp_deregister_script( 'comment-reply' );
         }
add_action('init','clean_header');</pre>