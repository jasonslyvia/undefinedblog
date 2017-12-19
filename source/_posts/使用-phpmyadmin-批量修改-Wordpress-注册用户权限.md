---
title: 使用 phpmyadmin 批量修改 Wordpress 注册用户权限
tags: 

  - mysql
  - sql
permalink: shi-yong-phpmyadmin-pi-liang-xiu-gai-wordpress-zhu-ce-yong-hu-quan-xian
id: 70
updated: '2012-06-12 20:30:17'
date: 2012-06-12 20:30:17
---

<p>想到要修改注册用户权限是因为前两天 ppiOS问答 被一位恶意用户发布了一篇恶意的黑帽SEO文章。虽然我已经在 ppiOS问答 后台设置了用户提交的问题默认处于草稿状态，但是这个恶意发文章的用户显然是了解 Wordpress 机制的。尽管没有提供后台接口，但是他还是成功的跳过管理员的审核发布了一篇软文。我发现后第一时间就想到：糟糕，用户权限没有分配好！</p>
<p>之前不了解 Wordpress 的用户注册机制，为了方便用户提问我把新用户注册后的角色统一的设置成了作者，没想到这次却被人利用了这个漏洞。痛定思痛，我研究了一下 Wordpress 中关于用户权限的内容，在 wp-usermeta 中，结构比较特殊，但是结合<a href="http://www.shinephp.com/how-to-change-wordpress-user-role-capabilities/" target="_blank">另一篇</a>文章的指点，还是参悟出了批量修改用户权限的方法。</p>
<p>执行如下的 SQL 语句可以显示你的 Wordpress 中所有用户的权限设置：</p>
<p><img src="http://ww1.sinaimg.cn/mw690/868702cfjw1dtvq0kotw5j.jpg" alt="wordpress数据库 查询语句" /></p>
<p>小插曲：服务器竟然检测到我贴的 SQL 语句，在预览文章时提示 405 NOT ALLOWED，估计是避免 SQL 注入吧！ ASERVER 真有意思！</p>
<p>在 ppiOS 问答中查询结果如图：</p>
<p><img src="http://ww1.sinaimg.cn/mw690/868702cfjw1dtvq0l3l2dj.jpg" alt="wordpress 用户 数据库" /></p>
<p>可见，管理员的身份标示为</p>
<blockquote>
<p>a:1:{s:13:"administrator";s:1:"1";}</p>
</blockquote>
<p>而作者的身份标示则是</p>
<blockquote>
<p>a:1:{s:6:"author";s:1:"1";}</p>
</blockquote>
<p>我们的最终目的是把作者角色的用户全部转换为贡献者（contributor，只能提交文章，不能改变文章状态，即不能发表），那么 contributor 的字段是什么呢？</p>
<blockquote>
<p>a:1:{s:11:"contributor";s:1:"1";}</p>
</blockquote>
<p>因此我们要做的是把所有用户除了管理员的权限全部改为贡献者，SQL 语句如下：</p>
<p><img src="http://ww4.sinaimg.cn/mw690/868702cfjw1dtvq0lkmqxj.jpg" alt="wordpress" /></p>
<p>修改完悲催的发现，原来 Wordpress 还是有方便的图形化界面来批量修改权限的。在后点，【用户】选项卡，右上方的下拉菜单，【显示选项】，选择一次显示的用户数从20改到200就差不多了&hellip;&hellip;然后再批量修改就好&hellip;&hellip;</p>