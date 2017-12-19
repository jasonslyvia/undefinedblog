---
title: VPS 常用 SSH 命令
tags: 

  - ssh
  - vps
permalink: vps-chang-yong-ssh-ming-ling
id: 90
updated: '2012-08-21 21:06:06'
date: 2012-08-21 21:06:06
---

<p><strong>重启nginx</strong></p>
<p>/usr/local/nginx/sbin/nginx -s reload</p>
<p><strong>修改 nginx conf</strong></p>
<p>vim /usr/local/nginx/conf/vhost/xxx.conf</p>
<p><strong>删除整个目录</strong></p>
<p>rm -rf /folder/</p>
<p><strong>拷贝整个目录</strong></p>
<p>cp -R /somefolder/. /someotherfolder</p>
<p><strong>修改所有文件权限(注意最后的斜杠和分号，如果想要修改目录则把 f 换为 d)</strong></p>
<p>&nbsp;</p>
<p><del>find -type -f -exec chmod 655 {} ;</del></p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>经过实践发现 -type 后面的参数不能加 '-'</p>
<p>find -type f -exec chmod 655 {} ;</p>
<p><strong>远程下载文件(用来远程下载wordpress程序)</strong></p>
<p>wget url</p>
<p><strong>解压缩</strong></p>
<p>unzip</p>