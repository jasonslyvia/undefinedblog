---
title: AppFog如何安装PhpMyAdmin
tags: 

  - appfog
  - mysql
permalink: install-phpmyadmin-on-appfog
id: 115
updated: '2012-11-24 06:50:50'
date: 2012-11-24 06:50:50
---

<p>AppFog 支持的语言种类很多，相应的支持的数据库格式也不少。但是 AppFog 不同于一般的虚拟主机，即使你选择了一键安装 Wordpress，也不会自动给你安装一个 PhpMyAdmin，因此我们需要手动安装 PhyMyAdmin。</p>
<p>其实安装的方法很简单，因为 AppFog 中的数据库可以在不同的 App 之间共享（bind 一下就好），所以大概思路就是新建一个 php 应用，然后在 Services 里 bind 原来的数据库，在上传 phpmyadmin 源码即可。</p>
<h2>AppFog上安装PhpMyAdmin</h2>
<p>1、新建 App，在控制台最上面点 Create App，选择 PHP</p>
<p>2、机房选 Singapore</p>
<p>3、名字随意起，比如 myappphpmyadmin</p>
<p>4、创建成功后在左侧的【Sevice】里找到你需要安装 phpmyadmin 的那个数据库，点 bind</p>
<p>5、下载 phpmyadmin 源码 &nbsp;<a href="http://www.phpmyadmin.net/home_page/downloads.php">http://www.phpmyadmin.net/home_page/downloads.php</a></p>
<p>6、按照之前讲过的<a href="/2012/11/install-wordpress-on-appfog/" target="_blank">AppFog部署方法</a>进行部署，然后通过系统自动分配的域名访问即可</p>