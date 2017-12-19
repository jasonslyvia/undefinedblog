---
title: 'CentOS无缝升级nginx[完整命令及自动升级脚本]'
permalink: seamlessly-upgrade-nginx-on-centos
id: 139
updated: '2013-05-13 16:38:16'
date: 2013-05-13 16:34:13
tags:
---

这两天网站的服务器总是出现 503，ssh 进去用 top 命令看了一下似乎也没有特别严重的资源消耗，加上最近看到 nginx 又爆出很多漏洞，看着现在服务器上老版本的 nginx 我倒吸了一口凉气，决定升级。

先交代一下本次升级的环境：
<ul>
	<li>系统： CentOS 5.9   32位</li>
	<li>nginx 安装位置： /usr/local/nginx（<strong>不同的主机安装位置不同，使用以下代码时请注意做必要的替换</strong>）</li>
</ul>
<h2>手动升级</h2>
按照如下方法一步步下载、编译、替换 nginx 即可完成升级，对现有的 nginx 配置文件不会造成任何影响。

首先要做的当然是备份了，我采用的备份方法是将 nginx 目录下的所有文件打包压缩为一个名为 nginx.tar.gz 的文件
<pre class="lang:sh decode:true" title="其中 /usr/local/nginx 是nginx的安装目录">tar -zcvf nginx.tar.gz /usr/local/nginx/.</pre>
然后远程下载最新稳定版的 nginx
<pre class="lang:sh decode:true">wget http://nginx.org/download/nginx-1.4.1.tar.gz</pre>
<a href="http://nginx.org/en/download.html" target="_blank"> Nginx 下载地址</a>

将下载好的文件解压缩
<pre class="lang:sh decode:true">tar -zxvf nginx-1.4.1.tar.gz</pre>
解压后得到目录 nginx-1.4.1，进入该目录
<pre class="lang:sh decode:true">cd nginx-1.4.1</pre>
接下来我们需要编译 nginx 的源码，在编译之前确保你的主机安装了必须的编译工具
<pre class="lang:sh decode:true">yum install gcc openssl-devel pcre-devel zlib-devel</pre>
如果你不清楚编译时的选项，可以参考现有 nginx 的编译选项，查看方法
<pre class="lang:sh decode:true">/usr/local/nginx/sbin/nginx -V</pre>
得到结果如下
<pre class="lang:sh decode:true">nginx version: nginx/某版本号
built by gcc 4.1.2 20080704 (Red Hat 4.1.2-54)
TLS SNI support disabled
configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-ipv6</pre>
其中 configure arguments 就是当前运行的 nginx 编译时的命令，找到这些命令后我们开始编译新的 nginx
<pre class="lang:sh decode:true">./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-ipv6 &amp;&amp; make</pre>
编译完成后，在当前目录的 objs 目录下就是我们需要的 nginx 可执行文件了，只需覆盖即可完成升级。在覆盖老版本的 nginx 之前，首先将其重命名以免无法覆盖
<pre class="lang:sh decode:true">mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old</pre>
然后将新编译好的 nginx 复制到 nginx 的目录中
<pre class="lang:sh decode:true">cp objs/nginx /usr/local/nginx/sbin/nginx</pre>
最后重启 nginx 完成升级
<pre class="lang:sh decode:true">/usr/local/nginx/sbin/nginx -s reload</pre>
使用上面介绍过的命令查看 nginx 版本，显示为 1.4.1。
<h2>自动升级</h2>
如果你使用了 lnmp 一键包的安装环境（lnmp.org），可以考虑直接使用写好的 nginx 自动升级 shell 脚本。

将脚本下载到本地并执行该脚本
<pre class="lang:sh decode:true">wget soft.vpser.net/lnmp/upgrade_nginx.sh;sh upgrade_nginx.sh</pre>
输入你想要升级的版本号，比如 1.4.1 或 1.5.0 然后脚本就会自动进行备份及升级工作了，原理与上述「手动升级」代码一致。