---
title: 如何让 BWP External Links 忽略指定的域名
tags: 

  - wordpress
  - 插件
permalink: ru-he-rang-bwp-external-links-hu-lue-zhi-ding-de-yu-ming
id: 97
updated: '2012-09-07 21:29:08'
date: 2012-09-07 21:29:08
---

<p>BWP External Links 是 Wordpress 平台一款很好的外链管理插件，它允许你设置所有链接向站外的链接为 nofollow 甚至新窗口打开，但是这款插件只能够过滤子域名，却不能过滤某些特定的域名。比如你的网站里有淘宝客的域名，也会被&nbsp;BWP External Links 识别为外链，导致进行一个新的页面之类。</p>
<p>其实只要进行简单的修改，就能够实现让&nbsp;BWP External Links 忽略指定的域名（以taobao.com为例）</p>
<p>打开 Wordpress 后台，【插件】-【编辑】，选择&nbsp;BWP External Links -&nbsp;<strong>bwp-external-links/includes/class-bwp-external-links.php</strong></p>
<p>在大约 430 行左右找到函数 is_local，在函数体内增加一个 strpos 判断，如果域名里含有&ldquo;taobao&rdquo;，则自动返回 true，即指明这个域名为本地域名。</p>
<pre class="brush: php;fontsize: 100; first-line: 1; ">function is_local($url, $home)
{

#以下为添加内容
if(strpos($url, 'taobao') !== false){
return true;
}

$以上为添加内容
$url = strtolower($url);
// Treat http and https as the same scheme
$home = (strpos($home, 'https://') === 0) ? ('http' . substr($home, 5)) : $home;
$url = (strpos($url, 'https://') === 0) ? ('http' . substr($url, 5)) : $url;
// Compare the URLs
if (!($is_local = (strpos($url, $home) === 0)))
{
// If there is no scheme, then it's probably a relative, local link
$scheme = substr($url, 0, strpos($url, ':'));
$is_local = !$scheme || ($scheme &amp;&amp; !preg_match('/^[a-z0-9.]{2,16}$/iu', $scheme));
}
// Not local, now check forced local domains
if (!$is_local &amp;&amp; $this-&gt;options['input_forced_local'])
$is_local = $this-&gt;match_domain($url, $this-&gt;parse_options('input_forced_local', false));

return $is_local;
}</pre>
<p>如果想过滤过个域名，则写成</p>
<pre class="brush: php;fontsize: 100; first-line: 1; ">if(strpos($url, 'xxx') !== false || strpos($url, 'ooo') !== false)
</pre>
<p>即可</p>