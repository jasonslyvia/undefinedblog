---
title: 如何让WaterMark Reloaded插件不对GIF图片添加水印
tags: 

  - wordpress
permalink: ru-he-rang-watermark-reloadedcha-jian-bu-dui-giftu-pian-tian-jia-shui-yin
id: 92
updated: '2012-08-24 12:50:10'
date: 2012-08-24 12:50:10
---

<p>WaterMark Reloaded 是一款很好的 Wordpress 水印插件，但是这款插件默认会给 GIF 格式的图片添加水印，而且水印效果非常不理想。其实只要简单的编辑一下这款插件，就可以取消 WaterMark Reloaded 针对 GIF 添加的水印。</p>
<p>进入 Wordpress 后台，选择【插件】-【编辑】，找到【Watermark Reloaded】，编辑</p>
<blockquote>
<p><strong>watermark-reloaded/watermark-reloaded.php</strong></p>
</blockquote>
<p>文件，搜索&ldquo;gif&rdquo;，在大概如下位置</p>
<p>[cc lang="php"]</p>
<p>/*添加图片水印*/<br /> private function imageAddImg($image, array $opt) {<br /> $src = dirname(__FILE__).'/'.$this-&gt;get_option('watermark_file');<br /> $info = getimagesize($src);<br /> $width = $info[0];<br /> $height = $info[1];<br /> switch ($info['mime']){<br /> case "image/jpeg" :<br /> $src = imagecreatefromjpeg($src);<br /> $ext_name = 'jpg';<br /> break;</p>
<p>//删除以下<br /> case 'image/gif' :<br /> $src = imagecreatefromgif($src);<br /> $ext_name = 'gif';<br /> break;</p>
<p>//删除以上<br /> case 'image/png' :<br /> $src = imagecreatefrompng($src);<br /> $ext_name = 'png';<br /> break;<br /> default :<br /> $src = imagecreate($width,$height);<br /> imagecolorallocate($src,0xff,0xff,0xff);<br /> }<br /> $offset = $this-&gt;calculateOffset($image,$opt);<br /> imagealphablending($image, true);<br /> imagecopy($image, $src, $offset['x'], $offset['y'], 0, 0, $width, $height);<br /> return $image;<br /> }<br /> [/cc]</p>
<p>删除上述代码中注释包围的部分，点击【更新文件】按钮即可。</p>