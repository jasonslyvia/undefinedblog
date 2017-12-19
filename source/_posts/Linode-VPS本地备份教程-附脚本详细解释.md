---
title: 'Linode VPS本地备份教程[附脚本详细解释]'
tags: 

  - bash
  - vps
permalink: linode-vpsben-di-bei-fen-jiao-cheng-fu-jiao-ben-xiang-xi-jie-shi
id: 108
updated: '2012-11-09 13:40:22'
date: 2012-11-09 13:40:22
---

<p>废话不多说，直接上代码，解释为 &ldquo;#&rdquo; 后面的文字，代码后面附上了详细的使用方法。</p>
<pre class="brush: bash;fontsize: 100; first-line: 1; ">#!/bin/bash
#basedir 为保存备份文件的目录名，无需更改
basedir=/backup/daily
PATH=/bin:/usr/bin:/sbin:/usr/sbin; export PATH
export LANG=C

#www 和 sqld 分别代表保存备份文件的两个子目录名，无需更改
wwwd=$basedir/www
sqld=$basedir/sql

#创建保存备份文件的目录
for dirs in $wwwd $sqld
do
[ ! -d "$dirs" ] &amp;&amp; mkdir -p $dirs
done

#将下面一行中的 YOURPASSWARD 更改为你的数据库密码
/usr/local/mysql/bin/mysqldump -uroot -pYOURPASSWARD --all-databases &gt; $sqld/mysql.$(date +%Y-%m-%d).tar.bz2

cd /home/

#将下面一行中的 wwwroot 改为你存放网站文件的目录，一般不是 wwwroot 就是 www
tar -jpc -f $wwwd/www.$(date +%Y-%m-%d).tar.bz2 wwwroot
rm -rf $sqld/mysql.$(date +%Y-%m-%d -d "2 days ago").tar.bz2
rm -rf $wwwq/www.$(date +%Y-%m-%d -d "2 days ago").tar.bz2
</pre>
<h2>Linode VPS 本地备份脚本使用方法</h2>
<p>1、将以上代码复制，打开 PuTTy（或其他 SSH 软件），连接到你的 VPS（注意修改需要修改的部分，详见注释）</p>
<p>2、输入</p>
<blockquote>
<p><strong>vim backup.sh</strong></p>
</blockquote>
<p>3、按下键盘上的 i 键进入编辑状态，此时在 PuTTy 的界面单击右键，刚才复制的代码就粘贴进去了</p>
<p>4、按下键盘的 ESC 键，再按下 SHIFT + ; （即输入 ： ），再输入 wq，按回车，就保存好了脚本</p>
<blockquote>
<p>相当于输入了 <strong>:wq</strong> 这三个字符</p>
</blockquote>
<p>5、输入</p>
<blockquote>
<p><strong>sh backup.sh</strong></p>
</blockquote>
<p>系统就开始自动执行备份脚本了</p>
<p>6、脚本执行结束后，在 /backup/daily/ 下出现两个文件夹，一个是 sql，一个是 www，分别储存着数据库备份文件和网站文件的备份。</p>