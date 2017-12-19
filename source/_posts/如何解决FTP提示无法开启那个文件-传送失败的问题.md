---
title: 如何解决FTP提示无法开启那个文件 传送失败的问题
tags: 

  - ftp
permalink: >-
  ru-he-jie-jue-ftpti-shi-wu-fa-kai-qi-na-ge-wen-jian-chuan-song-shi-bai-de-wen-ti
id: 110
updated: '2012-11-13 11:03:32'
date: 2012-11-13 11:03:32
---

<p>换用 VPS 后，各种关于权限的问题就接踵而至了。之前说过记得把目录下的所有文件夹赋予权限 644，目录赋予权限 755，但是最近又遇到另一个问题，跟权限扯上了关系。</p>
<p>当需要用 SSH 处理一些文件时，比如用 wget 直接远程下载 wordpress 压缩包然后用 unzip 解压，这样操作后的目录无法使用 FTP 上传文件，当你尝试上传的时候会提示</p>
<blockquote>
<p><strong>无法开启那个文件????<br /> </strong><strong>传送失败</strong></p>
</blockquote>
<p>即使权限看起来没有问题，但是依然会遇到各种错误。</p>
<p>这可能是目录权限设置问题或ftp管理上的用户设置问题。</p>
<p>面板上用户需要的权限是 www</p>
<p>打开 SSH 输入如下命令，其中 /home/wwwroot/ 是你 VPS 存放网站文件的目录</p>
<blockquote>
<p><strong>chown www:www -R /home/wwwroot/</strong></p>
</blockquote>
<p>即可解决提示无法开启那个文件 传送失败的问题</p>