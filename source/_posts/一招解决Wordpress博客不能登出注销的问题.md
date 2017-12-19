---
title: 一招解决Wordpress博客不能登出注销的问题
permalink: '-zhao-jie-jue-wordpressbo-ke-bu-neng-deng-chu-zhu-xiao-de-wen-ti'
id: 8
updated: '2011-10-11 20:53:15'
date: 2011-10-11 20:53:15
tags:
---

<p>之前把两个Wordpress博客的用户数据共享了，但是之后却出现了Wordpress登陆了不能登出的问题，着实恼火。<br /> 在Google了很久之后，找到了如下的解决方法：<br /> 在登出（注销链接）处找到如下代码：<br /> [cc lang="php"]<!--?php echo get_option('siteurl'); ?-->[/cc]<br /> 注：可能不同的主题登出的代码不同，你可以把鼠标移动到登出链接上，查看登出的链接，然后在你的代码中查找相关数据<br /> 然后将上面那行代码替换为：<br /> [cc lang="php"]<!--?php echo wp_logout_url(get_permalink()) ?-->[/cc]<br /> 这样就可以完美解决无法登出的问题了！</p>