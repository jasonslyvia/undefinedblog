---
title: 如何给IE6IE7360用户启用指定的Wordpress主题
tags: 

  - wordpress
permalink: ru-he-gei-ie6ie7360yong-hu-qi-yong-zhi-ding-de-wordpresszhu-ti
id: 79
updated: '2012-07-14 00:22:55'
date: 2012-07-14 00:22:55
---

<p>首先呼吁大家马上抛弃IE6和IE7，最好连IE一并都抛弃了！！！！</p>
<p>下面解决实际问题，怎么让Wordpress给这些不兼容现有主题的浏览器指定一个可以兼容的主题呢？答案是使用 IE Theme 插件，当然，这款插件只可以指定 IE 浏览器，实际上在中国有很多国产浏览器也是用 IE 内核，同样显示有问题，比如 360，所以需要修改一下这款插件的部分内容，让360等浏览器也可以正常显示兼容主题。</p>
<p>编辑 IE Theme 插件中的 ie-theme.php 文件，将其中的 ie_init() 函数修改如下：</p>
<pre class="brush: php;fontsize: 100; first-line: 1; ">function ie_init(){
global $ie, $iever;
$ie = get_option("ie");
$ie['purl']=WP_PLUGIN_URL.'/'.str_replace(basename(__FILE__),"",plugin_basename(__FILE__));
$ie['path']=WP_PLUGIN_DIR.'/'.str_replace(basename(__FILE__),"",plugin_basename(__FILE__));

if( preg_match("/msie.{0,1}([0-9]{1,2}).([0-9]{1,3})/i",$_SERVER["HTTP_USER_AGENT"],$ver) ){
$iever = $ver[1];

//添加如下if语句，其中360ee即360浏览器特有的User-agent
if( preg_match("/360ee/i",$_SERVER["HTTP_USER_AGENT"]) == 1 )
$iever = 6;
}
else $iever=0;
}</pre>
<p>&nbsp;</p>
<p>如果你想增加其他的国产浏览器，比如滕旭TT之类的，只用把特定的UA加入if判断即可。最后再鄙视一下IE浏览器！！</p>
<p>测试方法：使用IE9或Chrome或Firefox访问ppiOS显示正常主题，使用IE6IE7360 IE兼容模式显示IE兼容主题，主题的Banner就是很明显的警告字样：你的浏览器版本太低了！！</p>