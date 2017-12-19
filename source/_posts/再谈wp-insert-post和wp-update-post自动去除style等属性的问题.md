---
title: 再谈wp_insert_post和wp_update_post自动去除style等属性的问题
tags: 

  - wordpress
permalink: >-
  zai-tan-wp_insert_posthe-wp_update_postzi-dong-qu-chu-styledeng-shu-xing-de-wen-ti
id: 104
updated: '2012-10-04 23:00:21'
date: 2012-10-04 23:00:21
---

<p>之前因为调用 wp_insert_post 后查看文章时发现所有的 style 属性都被去除了，Google 之后发现是 kses 的问题，网上说只要修改一个名为 allowedposttags 的 global 变量即可，因此我总结了一下发表了&nbsp;<a title="wp_insert_post 自动清除 style 等属性的解决方法" href="/2012/09/wp_insert_post-%e8%87%aa%e5%8a%a8%e6%b8%85%e9%99%a4-style-%e7%ad%89%e5%b1%9e%e6%80%a7%e7%9a%84%e8%a7%a3%e5%86%b3%e6%96%b9%e6%b3%95/" target="_blank">wp_insert_post 自动清除 style 等属性的解决方法</a>&nbsp;这篇文章，现在想想还是不严谨的。</p>
<p>按照这篇文章的做法并不能避免使用上述两个函数去除 style 标签的问题，具体原因今天空下来之后又研究了一下，<strong>原来是涉及到一个权限的问题。</strong></p>
<p>如果以管理员的身份去调用 wp_insert_post 或 wp_update_post，就不会有这些问题，而普通的用户调用时这些标签依然会被去除。</p>
<p>解决方法是在需要调用者两个函数的地方先取消 Wordpress 检查合法标签的 hook，即按照以下方式调用：</p>
<pre class="brush: php;fontsize: 100; first-line: 1; ">//去除钩子

kses_remove_filters();

wp_insert_post($post);

//添加钩子

kses_init_filters();
</pre>
<p>当然这只是一种临时的解决方案，如果你确认 UGC 不会产生问题的话，或者能确保所有的函数调用都在自己的控制范围内，那么这样解决也未尝不是一种方法！</p>
<p>参考：<a href="http://wordpress.org/support/topic/bypass-sanitize_post-from-wp_insert_post">http://wordpress.org/support/topic/bypass-sanitize_post-from-wp_insert_post</a></p>