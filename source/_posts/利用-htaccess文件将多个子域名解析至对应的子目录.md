---
title: 利用.htaccess文件将多个子域名解析至对应的子目录
tags: 

  - htaccess
permalink: >-
  li-yong-htaccesswen-jian-jiang-duo-ge-zi-yu-ming-jie-xi-zhi-dui-ying-de-zi-mu-lu
id: 53
updated: '2012-04-22 06:23:13'
date: 2012-04-22 06:23:13
---

<p>对于不支持子域名解析但是支持 .htaccess 的主机来说，这个功能就非常有用了</p>
<p>假设有主域名 ppios.com，子域名 yspx.ppios.com 和 ask.ppios.com，设置结果为访问 repo.ppios.com 时自动解析到 www.ppios.com/repo/ 文件夹中</p>
<p>下面是详细设置：<br /> <strong>在根目录下的 .htaccess</strong></p>
<blockquote>
<p>RewriteEngine On<br /> RewriteCond %{HTTP_HOST} ^((www|<span style="color: #008000;">yspx</span>).)?<span style="color: #800000;">ppios</span>.com$<br /> RewriteCond %{REQUEST_URI} !^/<span style="color: #008000;">yspx</span>/<br /> RewriteCond %{REQUEST_FILENAME} !-f<br /> RewriteCond %{REQUEST_FILENAME} !-d<br /> RewriteRule ^(.*)$ /<span style="color: #008000;">yspx</span>/$1<br /> RewriteCond %{HTTP_HOST} ^((www|<span style="color: #008000;">yspx</span>).)?<span style="color: #800000;">ppios</span>.com$<br /> RewriteRule ^(/)?$ <span style="color: #008000;">yspx</span>/<span style="color: #0000ff;">index.html</span> [L]</p>
<p>RewriteCond %{HTTP_HOST} ^((www|<span style="color: #008000;">repo</span>).)?<span style="color: #800000;">ppios</span>.com$<br /> RewriteCond %{REQUEST_URI} !^/<span style="color: #008000;">repo</span>/<br /> RewriteCond %{REQUEST_FILENAME} !-f<br /> RewriteCond %{REQUEST_FILENAME} !-d<br /> RewriteRule ^(.*)$ /<span style="color: #008000;">repo</span>/$1<br /> RewriteCond %{HTTP_HOST} ^((www|repo).)?ppios.com$<br /> RewriteRule ^(/)?$ repo/ [L]</p>
</blockquote>
<p>注：标红区域请自行替换成自己的域名（若不是.com则需同时替换.com部分），绿色区域请自行替换为自己的子域名（子目录名），蓝色区域请自行设置子目录对应的首页类型，比如 repo 文件夹下主页是 index.html，则设置为 repo/index.html，也可不设置，只写 repo/</p>
<p><strong>在对应子目录下的 .htaccess</strong></p>
<blockquote>
<pre>RewriteEngine On
RewriteBase /<span style="color: #008000;">repo</span>
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /<span style="color: #008000;">repo</span>/ [L]</pre>
</blockquote>
<pre>注：绿色部分自行替换为对应的子目录名，要在每一个子目录下都放置对应的 .htaccess 文件</pre>
<pre>这样就完成了通过设置 .htaccess 来将子域名解析到子目录的工作</pre>