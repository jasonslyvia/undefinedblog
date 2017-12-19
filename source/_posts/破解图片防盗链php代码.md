---
title: 破解图片防盗链php代码
tags: 

  - 图片
permalink: po-jie-tu-pian-fang-dao-lian-phpdai-ma
id: 22
updated: '2011-11-19 01:01:01'
date: 2011-11-19 01:01:01
---

<p>1、复制如下代码，保存为ppios.php</p>
<p>&nbsp;</p>
<pre class="brush: php;fontsize: 100; first-line: 1; ">&lt;?
 $p=$_GET['p'];
$pics=file($p);
for($i=0;$i&lt; count($pics);$i++)
{
echo $pics[$i];
}
?&gt;</pre>
<p><br /> 2、将ppios.php上传到网站根目录<br /> 3、在需要显示防盗链的图片时，将图片地址替换为</p>
<pre class="brush: php;fontsize: 100; first-line: 1; ">http://你的域名/ppios.php?p=图片地址</pre>
<p><br /> 例如网易博客某图片的地址为<br /> http://img6.ph.126.net/lEKhvNMxJgvtn359bLV1zA==/2399292701498720326.jpg<br /> <img src="http://img6.ph.126.net/lEKhvNMxJgvtn359bLV1zA==/2399292701498720326.jpg" alt="" /><br /> 直接插入会显示防盗链<br /> 将该地址替换为<br /> http://www.ppios.com/ppios.php?p=http://img6.ph.126.net/lEKhvNMxJgvtn359bLV1zA==/2399292701498720326.jpg<br /> 即可正常显示<br /> <img src="http://www.ppios.com/i.php?p=http://img6.ph.126.net/lEKhvNMxJgvtn359bLV1zA==/2399292701498720326.jpg" alt="" /></p>
<p>&nbsp;</p>
<p>&nbsp;</p>