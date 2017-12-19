---
title: OS X Apache 部署完全指南
tags: 

  - apache2
  - os x
permalink: configure-os-x-apache
id: 146
updated: '2013-09-25 21:04:23'
date: 2013-09-25 21:04:23
---

每次配置 Apache 都是痛苦的经历，几乎很少有一次成功的时候，且不说 Apache 配置文件中的各种 Listen、DocumentRoot、Directory、Order、Allow、Options、deny、grant 指令，就是基础的 *nix 权限问题就能搞到我头大。

下面将我在 OS X 下利用系统自带 Apache 2 部署 web 开发环境的过程全部记录下来，方便自己回头查看，也希望能帮助到有需要的朋友。
<h2>基本信息</h2>
<ul>
	<li>Apache 所在目录：/etc/apache2</li>
	<li>Apache 配置文件：/etc/apache2/httpd.conf</li>
	<li>Apache 虚拟主机配置文件：/etc/apache2/extra/http-vhost.conf</li>
	<li>Apache 操作指令： sudo apachectl [start|stop|restart]   // 启动|停止|重启</li>
	<li>hosts 文件：/etc/hosts</li>
</ul>
下面的内容将不再一一指出每一个目录或文件对应的地址是什么。
<h2>部署思路</h2>
为不同的 WEB 项目指定不同的虚拟主机，同时绑定不同的『域名』
<h2>部署方法</h2>
1、编辑 apache 配置文件，找到
<pre class="lang:sh decode:true">#Virtual hosts
#Include /private/etc/apache2/extra/httpd-vhosts.conf</pre>
一段，将 Include 前面的 # 去掉，保存

2、编辑虚拟主机配置文件，在末尾添加新的虚拟主机配置
<pre class="lang:sh decode:true">&lt;VirtualHost *:80&gt;
    ServerAdmin youremail@email.com
    ServerName your_server_name
    DocumentRoot "path_to_your_document_root"
    ErrorLog "/etc/apache2/error.log"
    &lt;Directory "path_to_your_document_root"&gt;
        Options All Indexes FollowSymLinks
        Satisfy Any
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;</pre>
<ul>
	<li>ServerAdmin：你的邮箱地址，可以随便填，不影响</li>
	<li>ServerName：要与你在 hosts 中添加的名称一致，比如你希望在浏览器输入 mywebsite.xxx 就能访问这个虚拟主机，就需要先在 hosts 中增加一行 127.0.0.1   mywebsite.xxx，再将 ServerName 设置为 mywebsite.xxx</li>
	<li>DocumentRoot 及 Directory 后引号中的内容：你的所有代码存放的目录</li>
</ul>
编辑完成后保存（Require all granted 是 Apache 2.4.3 以后新增的特性）

3、重启 apache，试试输入你绑定好的域名访问你的虚拟主机
<h2>常见问题</h2>
0、检查你的配置文件，通过
<pre class="lang:sh decode:true">sudo apachectl -S</pre>
命令能够检查你当前虚拟主机的配置文件是否有语法错误活或其它错误，如果有则解决。

常见的配置出错问题包括在 OS X 系统下使用了含有空格的目录，Apache 会自动为带有 \ 的目录增加多一个 \，即成为 \\，导致目录无法被识别。正确的配置方法是若目录含有空格，则直接输入空格，无需转义即可。

1、出现 403 Forbidden

首先查看错误日志
<pre class="lang:sh decode:true">tail -f /etc/apache2/error.log</pre>
看看具体是哪个目录不能访问，如果连代码所在的根目录都不能访问的话，说明权限设置有问题
<pre class="lang:sh decode:true">chown username:group -R *</pre>
上面的 username 是你的用户名，如果不确定的话可以在终端输入命令『whoami』查询；group 是用户所在的组，一般来说 OS X 默认都是『staff』，如果不确定可以通过命令『groups 你的用户名』查询。通过 chown 这个命令已经递归的将你代码所在文件夹内的所有文件的所有权设置为当前用户。

如果依然出错，则需要给上级目录设置执行权限
<pre class="lang:sh decode:true">chmod +x ..</pre>
本操作需要递归进行到 / ，即系统根目录。
<h2>小知识</h2>
<h3>如何让局域网内的其它电脑或终端访问本机的服务器？</h3>
首先编辑 apache 配置文件，找到 <span class="lang:sh decode:true  crayon-inline">Listen 80</span> 将其改为<span class="lang:sh decode:true  crayon-inline">Listen *:80</span>，其次在你的虚拟主机配置文件中，找到<span class="lang:sh decode:true  crayon-inline">NameVirtualHost 80</span>，改为<span class="lang:sh decode:true  crayon-inline">NameVirtualHost *:80</span>，同时将<span class="lang:sh decode:true  crayon-inline">&lt;VirtualHost 80&gt;</span> 改为 <span class="lang:sh decode:true  crayon-inline">&lt;VirtualHost *:80&gt;</span> 即可。

这样配置的好处是比如你在本地开了一台虚拟主机跑 Windows 系统，你希望测试网站在 Windows 环境下的兼容性，则直接在虚拟机中输入本机 IP 地址就能访问了。