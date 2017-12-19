---
title: 配置nginx实现通过cookie-free域名发送静态资源
tags: 

  - cdn
  - cookie
  - nginx
permalink: cookie-free-domain-configuration-nginx
id: 142
updated: '2013-05-25 02:23:41'
date: 2013-05-25 02:23:41
---

最近真是闲，天天写博客……自从去百度实习后，我对自己在专业知识上的要求提升到了一个前所未有的高度，以前觉得能用就行，现在但凡可能有效率问题就会觉得抓狂。说到网页的加载效率，这个是我一直关注的东西，昨晚突然想起 <strong>cookie-free</strong> 这回事，一拍脑袋决定给网站的所有静态资源都搞成 cookie-free 的。
<h2>使用 cookie-free 域名的好处</h2>
当浏览器加载 HTML 文件中引用的静态资源 —— 如图片、外部 CSS、外部 JS 等 —— 时，若该资源所属域与当前页面相同，则会在 HTTP 头请求中加载当前域的 cookie 信息。

下面需要举个例子：
<p style="text-align: center;"><img class="size-full wp-image-207085 aligncenter" title="Let me give you a for instance" alt="for example" src="http://undefinedblog.com/wp-content/uploads/2013/05/for-example.jpg" width="300" height="200" /></p>
好吧，不是这个意思，认真的举个<del>栗子</del>例子：

你的网站为 http://www.whatever.com，启用了 Google Analytics 或百度统计或任意第三方统计代码。用户访问你的网站首页，首页 html 代码引用了 10 个图片文件，图片地址是 http://www.whatever.com/images/[0-9].jpg （此处有正则）。

因为 Google Analytics 在 http://www.whatever.com 这个域下设置了 cookie，浏览器在加载这些图片时，会把 Google Analytics 设置的 cookie 放在头信息里发过去。

<strong>本来一个 cookie 也没多大，顶多 1KB，但是如果你要加载 50 个图片（或其它静态文件），这样发送的 cookie 总量就多达 50KB 了。</strong>对于静态资源来说，发送这些 cookie 完全没有意义，所以我们不想让浏览器请求这些静态资源时发送 cookie。

下图是苹果教程网未设置 cookie-free 域名前请求某静态资源时 HTTP 请求头中包含 cookie 内容的惨状：

<img class="aligncenter size-full wp-image-207088" alt="未设置Cookie Free域名前的惨状" src="http://undefinedblog.com/wp-content/uploads/2013/05/1.png" width="767" height="341" />
<h2>配置nginx实现cookie-free域名</h2>
其实单独把 cookie-free 域名这个事儿单独拎出来说是一件很伤感的事情，因为有钱淫一般都会同时选择 VIP 套餐：CDN。使用 CDN，将静态资源分发在多台服务器上，用户在请求资源时智能判断哪个节点速度最快并返回，同时再启用一个 cookie-free 域名用来指向静态资源。

对于没有能力搞 CDN 的人，只能单独配置一下 cookie-free 域名了。没关系，等咱有了钱，服务器买 20 个，什么联通、电信、铁通、长宽、移动、电信通，通通给接上，带宽怎么着也得 20M，还得上下行速率对等。

好了不废话了，下面使用专业的语言描述一下要干的事：

前提条件：一台自己的服务器（VPS），安装了 nginx，已经有可用的 nginx 配置文件（假定 server_name 为 www.whatever.com），网站运转正常。

待办事项：修改 nginx 的配置文件，让 nginx 监听 cdn.whatever.com，并修改部分对静态文件的处理规则。
<h3>具体步骤</h3>
首先你要增加一个子域名（cdn.whatever.com）指向你的服务器，这个不用多说了吧……

然后需要编辑你的 nginx 配置文件，它原来大概可能长这样
<pre class="lang:c decode:true">server {
    listen      80;
    server_name www.whatever.com whatever.com;
    root        /root/to/your/site;

    location / {
        index   index.html index.php;
    }

    location ~* \.(gif|jpg|png)$ {
        expires 30d;
    }

    /*此处略去若干*/
}</pre>
你需要做的是首先修改 server_name 字段，让 nginx 同时监听 cdn.whatever.com。
<pre class="lang:c decode:true">    server_name www.whatever.com whatever.com cdn.whatever.com;</pre>
然后新增一个 location 块，写如下内容，并确保这个 location 块处于所有已存在的 location 块之前，即 nginx 需要优先处理这个 location 中的内容。
<pre class="lang:c decode:true">location ~* \.(?:js|css|png|jpg|jpeg|gif|ico)$ {
	expires max;
	add_header Pragma public;
	add_header Cache-Control "public, must-revalidate, proxy-revalidate";
	access_log off;
	log_not_found off;
	tcp_nodelay off;
	break;
}</pre>
搞定后保存 nginx 配置文件，然后重启 nginx。

最后一步，将网页中引用的所有 www.whatver.com 域下的静态文件通通改为 cdn.whatever.com。以上文所说的那个首页为例，新的图片地址为 http://cdn.whatver.com/images/[0-9].jpg。

下图为苹果教程网启用了 cookie-free 域名来发送静态资源后的 HTTP 头请求截图，是否清爽了很多。<strong>最重要的是，因为没有 cookie 的发送，用户的流量节省了很多，且网页的加载速度也节省了很多，我开心的笑了。</strong>（画外音：别傻了，能省几 KB 啊）
<p style="text-align: center;"><img class="aligncenter size-full wp-image-207092" title="设置Cookie free域名后世界清新了很多" alt="设置 Cookie free域名后世界清新了很多" src="http://undefinedblog.com/wp-content/uploads/2013/05/2.png" width="733" height="343" /></p>

<h2>小记</h2>
其实要实现 cookie-free，只需要换一个域名并保证当前页面没有给根域名设置 cookie 即可。比如我们虽然启用了 cdn.whatever.com，但是由于 Google Analytics（以下简称 GA） 设置 cookie 的域为 .whatever.com 而不是 .www.ppios.com，会导致请求 cdn.whatever.com 中的内容时依然会发送 cookie。具体 cookie 的设置情况请自行打开开发者工具查看。

这里再提一下关于限制 GA 设置 cookie 的内容，原则上 GA 应该是默认只设置当前域名的，比如你的页面 URL 是 http://www.whatever.com，则 GA 设置 cookie 的域就是 www.whatever.com；同理如果你的 URL 是 http://whatever.com，则 GA 会把 cookie 设置为 .whatever.com，这样你所有的子域名都会被这个 cookie 影响了。

解决方法就是修改 GA 统计代码，在
<pre class="lang:js decode:true ">_gaq.push(['_setAccount', 'UA-123456788-1']);
_gaq.push(['_trackPageview']);</pre>
<strong>这两行之前</strong>新增一行
<pre class="lang:js decode:true ">_gaq.push(['_setDomainName', 'www.whatever.com']);</pre>
这样就是告诉 GA 只准在 www.whatever.com 域下设置 cookie，关于这个设置顺序折腾了小半个晚上，大家一定要注意啊。
<h2>参考引用及相关阅读</h2>
上文所述 nginx 配置参考自 <a href="http://pastebin.com/RZSLJQ7D">http://pastebin.com/RZSLJQ7D</a>

nginx 官网 <a href="http://nginx.org/cn/">http://nginx.org/cn/</a>

关于 GA 配置的解惑 <a href="http://stackoverflow.com/questions/12599315/analytics-setdomainname-not-working-anymore">http://stackoverflow.com/questions/12599315/analytics-setdomainname-not-working-anymore</a>

Github 此前一篇关于 cookie 溢出攻击的好文 <a href="https://github.com/blog/1466-yummy-cookies-across-domains">https://github.com/blog/1466-yummy-cookies-across-domains</a>

我抬头看了一眼书架上的「HTTP 权威指南」，露出了不易察觉的微笑。