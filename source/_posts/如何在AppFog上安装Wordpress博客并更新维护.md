---
title: 如何在AppFog上安装Wordpress博客并更新维护
tags: 

  - appfog
  - php
  - wordpress
permalink: install-wordpress-on-appfog
id: 114
updated: '2012-11-24 05:57:45'
date: 2012-11-24 05:57:45
---

<p>AppFog 前身是 PHPFog，是一家位于美国波特兰的 PaaS 提供商，其产品 AppFog 支持 PHP、Node.js、Ruby、Python、.NET 和 Java。即使是免费 plan 也有 1G RAM 和 100M 数据库可以用，此外还可以绑定自定义域名，比起 GAE 什么的好多了。AppFog 使用的是亚马逊的 AWS 服务器，因此选择新加坡节点在国内的访问速度一流。下面就一步步记录如何使用 AppFog 提供的服务。</p>
<h2>前期准备</h2>
<p>首先前往 AppFog 进行注册 &nbsp;<a href="https://console.appfog.com/signup" target="_blank"> 注册地址</a></p>
<p>成功后点击 Create App，点击 Wordpress，然后选择新加坡节点（Singapore），输入一个三级域名，这个将成为你应用的代号，如 myapp</p>
<p>这样你的第一个 AppFog 应用就建好了，跟 GAE 一样，你需要安装一个客户端来管理你的代码（其实是 Ruby 写的一个命令行程序，Mac 及 Linux 用户无需单独下载客户端）</p>
<h2>代码提交及维护</h2>
<p><a href="http://rubyinstaller.org/" target="_blank">Windows 下载地址</a></p>
<p>Windows 用户在安装好这个客户端（其实是Ruby虚拟机）后，Mac 及 Linux 用户直接在终端里输入</p>
<blockquote><p>gem update --system</p></blockquote>
<p>如果提示 connection timeout，你可能需要代理才能完成工作了。假设你已经安装了 GoAgent，那你可以输入这样的命令：</p>
<blockquote><p>gem update --system --http-proxy=http://127.0.0.1:8087</p></blockquote>
<p>这样就会自动通过代理来执行命令了</p>
<p>第一个命令执行成功 &mdash;&mdash; 其实就是检查了个版本 &mdash;&mdash; 后，再输入：</p>
<blockquote><p>gem install af</p></blockquote>
<p>需要代理的话还是加上上面那句，<strong>这里的 af 就是 AppFog 自己编写的一个基于 Ruby 的管理客户端</strong></p>
<p>安装的过程需要等待一段时间</p>
<p><img src="/media/ag1zfnBwaW9zZGV2bmV3cgwLEgVNZWRpYRjDHww/20121124055325.png" alt="appfog" width="60%" /></p>
<p>成功后你需要先去 AppFog 中将目前的代码下载下来 <a href="https://console.appfog.com/" target="_blank">后台地址</a></p>
<p>在项目左侧的【Update Source Code】，选择 Download Source Code，然后将下载好的压缩文件解压的任意目录（推荐根目录，如 D:myapp ）</p>
<p>然后对文件进行一些修改，比如修改 wp-config.php 中的 WPLANG 选项，将其定义为 zh_CN 即可让 Wordpress 显示中文</p>
<p>修改完后继续回到刚才安装好的 af，此时需要先登录</p>
<blockquote><p>输入 af login</p></blockquote>
<p>账号密码即为你的 AppFog 账号、密码</p>
<p>登录进去后，用 cd 命令进入你的源代码目录（本机），如&nbsp;</p>
<blockquote><p>cd D:myapp</p></blockquote>
<p>然后再输入</p>
<blockquote><p>af update myapp</p></blockquote>
<p>代码就会进行更新了，其实和 SVN 差不多</p>
