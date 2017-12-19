---
title: 怎么让Robots不屏蔽Google Adsense
tags: 

  - google
permalink: zen-yao-rang-robotsbu-ping-bi-google-adsense
id: 28
updated: '2011-11-29 20:59:45'
date: 2011-11-29 20:59:45
---

<p>有的站长为了提高网站文章页的权重，屏蔽了一些tag和category页，使用的语法是</p>
<p>User-agent: *<br /> Disallow: /wp-admin/<br /> Disallow: /wp-includes/<br /> Disallow: /wp-content/<br /> Disallow: /tag/<br /> Disallow: /category/<br /> Disallow: /i.php?url=*<br /> Disallow: /wp-*</p>
<p>但是这样带来一个问题就是凡是 Spider不能爬得页面，是不会显示Google Adsense的，那怎么既屏蔽这些页面又让Google Adsense显示在页面里呢？</p>
<p>在Robots.txt里加入这样一句话：</p>
<p>User-agent: Mediapartners-Google<br /> Disallow:</p>
<p>即可，Mediapartners-Google是Google Adsense的蜘蛛，这句话就是告诉GA的蜘蛛可以爬整个站的内容</p>
<p>参考链接：<a href="http://www.google.com/support/adsense/bin/answer.py?answer=10532&amp;topic=159">Google</a></p>