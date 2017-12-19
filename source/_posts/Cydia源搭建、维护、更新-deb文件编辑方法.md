---
title: Cydia源搭建、维护、更新 deb文件编辑方法
tags: 

  - cydia
  - linux
permalink: cydiayuan-da-jian-wei-hu-geng-xin
id: 52
updated: '2012-04-22 04:58:26'
date: 2012-04-22 04:58:26
---

<p>这个个人技术博客已经很久没有更新了，基本失去了它本来的意义，逐步恢复吧！</p>
<p>记录一下今晚到现在学到的知识：<span style="color: #800000;">Cydia搭建的方法</span> 和 <span style="color: #800000;">利用.htaccess文件将二级域名解析到子目录</span></p>
<h2>DEB文件编辑方法</h2>
<p>注：适用于Mac系统</p>
<p>1、将下载到本地的 .deb 文件解压缩（使用 BetterZip软件），得到&nbsp;control.tar.gz 和&nbsp;data.tar.gz 两个文件</p>
<p>2、同时选中这两个文件双击解压，得到 control 文件和一个 data.tar</p>
<p>3、新建一个 DEBIAN 文件夹，将 control 文件放入</p>
<p>4、编辑 control 文件中的信息</p>
<p>5、打开控制台，使用 cd 命令切换到当前目录</p>
<p>6、使用</p>
<blockquote>
<p>find . -name '*.DS_Store' -type f -delete</p>
</blockquote>
<p>命令删除 .DS_Store 文件</p>
<p>7、拷贝文件夹的名称，打开控制台，切换到当前目录</p>
<p>8、输入命令</p>
<blockquote>
<p>chmod 0755 当前目录名</p>
<p>chmod 0755 当前目录名/DEBIAN</p>
<p>chmod 0755 当前目录名/DEBIAN/control</p>
<p>dpkg-deb -b 当前目录名</p>
</blockquote>
<h2>Cydia搭建及维护的方法</h2>
<p>具体的搭建方法不多说了，基本是参考这个视频：http://www.youtube.com/watch?v=9eHKvCqBDBQ</p>
<p>主要记录一下<strong>如何添加和更新.deb文件</strong></p>
<p>1、将所有的.deb文件放到 /repo/debs 文件夹中</p>
<p>2、在【终端】里输入命令</p>
<blockquote>
<p>cd repo</p>
</blockquote>
<p>3、继续输入命令</p>
<blockquote>
<p>dpkg-scanpackages debs / &gt; Packages</p>
</blockquote>
<p>4、第3步生成了集合所有deb文件信息的 <strong>Packages</strong> 文件</p>
<p>5、利用 bzip2 生成 Cydia 可以识别的文件格式，输入命令</p>
<blockquote>
<p>bzip2 -fks Packages</p>
</blockquote>
<p>6、将所有文件上传到服务器即可</p>
<p>注：Release 文件为整个Cydia源的描述</p>