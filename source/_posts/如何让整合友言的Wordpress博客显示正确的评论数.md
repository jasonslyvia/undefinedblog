---
title: 如何让整合友言的Wordpress博客显示正确的评论数
permalink: >-
  ru-he-rang-zheng-he-you-yan-de-wordpressbo-ke-xian-shi-zheng-que-de-ping-lun-shu
id: 25
updated: '2011-11-26 04:03:08'
date: 2011-11-26 04:03:08
tags:
---

<p>友言（uyan.cc）是一个社会化评论聚合网站，使用友言提供的社交评论框代替Wordpress自带的评论框，可以让你的网站与SNS连接的更紧，也方便网友直接在SNS里对网站的内容进行评论。<br />
但是由于友言使用了&lt;iframe&gt;结构，且评论机制不同于Wordpress原生评论，因此Wordpress主题自带的显示评论计数的函数就没用了。下面ppiOS就教大家如何解决这一问题。</p>
<h2>步骤一</h2>
<p>将以下代码保存到footer.php中&lt;/body&gt;标签之前<br />
[cc lang="javascript"]<br />
<!-- UY BEGIN --><br />
<script charset="gb2312" type="text/javascript" language="javascript">// <![CDATA[
(function () {
        var uyan_amount = document.createElement('script');
        uyan_amount.type = 'text/javascript';
        uyan_amount.src = 'http://uyan.cc/js/countFrame.js';
        (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(uyan_amount);
    }());
// ]]></script><br />
<!-- UY END -->[/cc]</p>
<h2>步骤二</h2>
<p>在需要显示评论计数的地方插入如下代码<br />
[cc lang="php"]<a href="<?php echo wp_get_shortlink(); ?>" uyan_identify="true" >0条评论</a>[/cc]<br />
友言会自动判断本篇文章的评论数并修复为正确的数量，完成！</p>
