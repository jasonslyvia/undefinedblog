---
title: 如何删除所有Wordpress自定义文章类型
tags: 

  - mysql
  - wordpress
permalink: ru-he-shan-chu-suo-you-wordpresszi-ding-yi-wen-zhang-lei-xing
id: 91
updated: '2012-08-22 17:44:21'
date: 2012-08-22 17:44:21
---

<p>由于之前<a href="http://www.ppios.com">苹果教程网</a>使用了知更鸟主题，并注册了三种自定义文章类型（bulletin, video, picture），后来换主题后自定义文章类型无法访问，但是任然存在于数据库中，并且使用 WP_Query 时依然会被调出来。比如显示相关文章时，可能依然会显示自定义类型的文章，而点击这些链接访问却会出现 404 错误。这对于搜索引擎和用户来说都是不好的体验，所以如果没有转换的需要，不如直接把这些自定义文章类型从数据库里删除。</p>
<h2>第一步：查看有哪些自定义文章类型</h2>
<p>使用 phpmyadmin，执行如下 SQL 语句</p>
<pre class="brush: sql;fontsize: 100; first-line: 1; ">SELECT&nbsp;DISTINCT&nbsp;post_type FROM&nbsp;`wp_posts`&nbsp;</pre>
<p>一般 Wordpress 自带的 post_type 是 post, page, revision 这些，确认你要删除的自定义文章类型</p>
<h2>第二步：删除Wordpress自定义文章</h2>
<pre class="brush: sql;fontsize: 100; first-line: 1; ">DELETE FROM wp_posts WHERE post_type = '要删除的post_type';</pre>
<p>执行该SQL语句即可删除指定的post_type，如果要删除多种则更换引号内的文字多次执行即可</p>
<h2>第三步：删除 post_meta</h2>
<p>删除自定义文章类型后顺便还要删除对应的 post_meta，只用执行一次如下 SQL 语句即可</p>
<pre class="brush: sql;fontsize: 100; first-line: 1; ">DELETE FROM wp_postmeta WHERE post_id NOT IN (SELECT id FROM wp_posts);</pre>