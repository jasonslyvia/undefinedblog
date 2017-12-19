---
title: 修改Wordpress插件的技巧
tags: 

  - wordpress
  - 插件
permalink: xiu-gai-wordpresscha-jian-de-ji-qiao
id: 109
updated: '2012-11-12 16:27:13'
date: 2012-11-12 16:27:13
---

<p>关于 Wordpress 外链问题，我一直使用 BWP External Links 处理，功能很强大，但是并不能满足我的全部需求。比如在排除列表方面，这款插件只能选择忽略一些或者全部子域名，但是我想有些站外域名也被排除掉就无法实现。这个问题之前已经写过<a title="如何让 BWP External Links 忽略指定的域名" href="/2012/09/%e5%a6%82%e4%bd%95%e8%ae%a9-bwp-external-links-%e5%bf%bd%e7%95%a5%e6%8c%87%e5%ae%9a%e7%9a%84%e5%9f%9f%e5%90%8d/" target="_blank">一篇文章</a>解决了，不再赘述。</p>
<p>想起 修改Wordpress插件的技巧 这个话题，是因为今天又动了一下 BWP External Links，因为一些原因我希望给所有的链接在跳转时进行 Base64 加密，在一个中间页进行解密并判断。插件本身提供了给所有站外域名添加前缀的功能，但是不支持对链接进行加密，所以需要修改。</p>
<p>因为我不喜欢用 Wordpress 后台的插件修改功能（这也给后来发生的问题买下来伏笔），所以直接用 FTP 进到 plugins 目录里编辑&nbsp;class-bwp-external-links.php 文件，大约在 1071 行是关于添加前缀的，源代码如下：</p>
<pre class="brush: php;fontsize: 100; first-line: 1; ">if (!$is_local &amp;&amp; $external_prefix)
$new_link = preg_replace('/href=(?:"|')([^"']*)(?:"|')/iu', 'href="' . $external_prefix . '$1"', $new_link);</pre>
<p>&nbsp;</p>
<p>我们希望把匹配出来的结果进行加密后再进行替换，所以 preg_replace 是无法实现的，需要用到 preg_replace_callback，具体修改方法不再赘述。</p>
<p>最好的修改Wordpress插件的方法（不考虑在 Wordpress 后台修改）是：<span style="color: #800000;"><strong>先停止使用插件，然后编辑插件文件，最后尝试重新启用。</strong></span></p>
<p>这样如果插件本身有 BUG，Wordpress 会阻止插件启用，这样就避免了插件的 BUG 带来的问题。</p>