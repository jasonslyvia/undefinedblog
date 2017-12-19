---
title: Ngnix：当静态资源不存在时重定向
tags: 

  - nginx
permalink: nginx-redirect-404-request-for-static-files
id: 134
updated: '2013-02-25 11:04:10'
date: 2013-02-25 11:04:10
---

<p>最近又没干啥&ldquo;正经&rdquo;事儿，倒是把苹果教程网重新装修了一遍，启用了 W3 TOTAL CACHE，配置了我两天终于达到了理想中的效果。</p>
<p>下面记录一下处理过程中遇到的一个小问题，<strong>当静态资源不存在时重定向的问题</strong>。</p>
<p>搞个网站真辛苦，写代码（旁白：还有 CSS ！！）就算了，还要会配置服务器，看各种参数= =</p>
<p>下面是正文：</p>
<h2>使用场景</h2>
<p>服务器 A（网站服务器，www.abc.com）：请求 cdn.abc.com/a.png</p>
<p>服务器 B（CDN 服务器，cdn.abc.com）：接受请求，发现 a.png 不存在，重定向至 www.abc.com/a.png</p>
<p>如果你使用过 W3TC，应该对这个比较熟悉，选择 【Self Hosted CDN】就是这么个效果。</p>
<h2>解决方案</h2>
<p>在服务器 B 对应的 nginx 配置文件（一般在 /usr/local/nginx/conf/nginx.conf）中的 server 大括号里增加如下内容：</p>
<p>&nbsp;</p>
<pre class="brush: bash;fontsize: 100; first-line: 1; ">location ^~ /static/
{
	try_files $uri @ppios;
}

location @ppios
{
	rewrite ^(.*)$ http://www.abc.com/$1;
}</pre>
<p>其中第一个 location 的意思是所有对 static 这个目录下的请求，如果你的文件就放在根目录，也可以写成 location /。甚至如果你只处理静态文件，可以写成 location ~/*/*(gif|png)$ 等等。[1]</p>
<p>括号里面的意思是尝试 serve 这个请求的文件，如果不成功就转到 ppios。</p>
<p>在 ppios 里，我们做了重定向，具体的语法就是标准的正则，不再赘述了。</p>
<h2>参考资料</h2>
<p>[1] Nginx Location&nbsp;<a href="http://wiki.nginx.org/HttpCoreModule#location" target="_blank">http://wiki.nginx.org/HttpCoreModule#location</a></p>
<p>[2] Nginx try_files&nbsp;<a href="http://wiki.nginx.org/HttpCoreModule#try_files" target="_blank">http://wiki.nginx.org/HttpCoreModule#try_files</a></p>
<p>&nbsp;</p>