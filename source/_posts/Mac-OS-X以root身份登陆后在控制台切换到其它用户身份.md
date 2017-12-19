---
title: Mac OS X以root身份登陆后在控制台切换到其它用户身份
tags: 

  - mac
  - root
permalink: mac-os-x-root-as-normal
id: 123
updated: '2012-12-13 02:07:42'
date: 2012-12-13 02:07:42
---

<p>产生这样的问题是因为我喜欢使用 root 用户登陆 Mac 系统（避免各种乱七八糟的权限问题，暂时没有时间就理清楚 *nix 系统的权限问题），但是在用控制台安装 brew 执行脚本时，会提示：</p>
<blockquote>
<p>Don't run this as root</p>
</blockquote>
<p>平时都是需要管理员权限，现在反而不能用管理员权限。用 *nix 都知道提权时 su/sudo，那切换用户呢？其实很简单</p>
<blockquote>
<p>su - 用户名</p>
</blockquote>
<p>即可，注意横岗和用户名中间还有空格</p>