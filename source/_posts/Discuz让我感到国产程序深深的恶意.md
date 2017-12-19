---
title: Discuz让我感到国产程序深深的恶意
tags: 

  - 开源
permalink: discuz-disgusts-me
id: 156
updated: '2014-04-20 06:29:56'
date: 2014-02-27 02:36:29
---

月初的时候接到一个 Discuz 二次开发的活儿，虽然我没有开发过 Discuz，但作为国内两大论坛程序之一，想必不会差到哪儿去。但是做了近两周的开发后，我不得不说 Discuz 让我感到了国内开源社区深深的恶意。在这里不吐不快，记录下来供后人参考。

注：以下内容均为一家之辞，如有不当之处，欢迎指正。

首先应该说明，<strong>Discuz 并不是严格意义上的开源程序。</strong>但是作为国内人气如此之高的 PHP 论坛程序，作为被腾讯这样的互联网巨头收购的公司，Discuz 对二次开发以及整个二次开发社区实在是太不友好了！下面吐槽开始：
<h2>文档稀缺、不规范，入门困难</h2>
当你满心欢喜的下载了 Discuz 在本地部署成功后，第一件事当然是看官方文档了解整个程序的大致结构，包括程序入口、路由规则、核心文件是哪些等等，但是你在 Discuz 官网上能看到的只有 2011 年的零碎文章（http://www.discuz.net/portal.php?mod=list&amp;catid=12），这还是针对 Discuz X 2.5 的。我不清楚该版本和最新的 Discuz X 3.1 有多少差距，但是对于程序员来说，没有详细的、完整的、正确的文档，二次开发简直就是灾难。

更不要说 Discuz 那多如牛毛的数据表，要想摸清楚哪张表是干什么的着实需要一番功夫，所有你可以参考的就是 Discuz 官方提供的『数据字典』。

最后再说一句，这么大一个论坛程序，竟然没有一本完整的教程书。不需要你写的像国外那些技术书一样好，随便弄个《21天学会Discuz》之类的水平也行啊。关键是木有！
<h2>基本所有模板、插件均收费，且质量参差</h2>
好吧，勉强看看这些东拼西凑、毫无逻辑可言的『教程』，然后开始着手二次开发。一般 Discuz 二次开发，都需要自己定义一套新的样式，这在 Discuz 里叫做『模板』开发，放在 /template 下。

由于需要加快开发速度，我们选择先购买一套现有的模板进行二次开发。正如小标题所说，基本上 Discuz 上的所有模板都是收费的，虽然每一款收费模板均有对应的免费版，但是免费版根本看不出任何样式（我们购买的这款模板，免费版总共只有一个 header 的样式）。

在购买了这款模板后，灾难才真正开始。由于 Discuz 会自动合并所有的模板 CSS 文件（具体有一个特殊的机制，在 CSS 文件中根据特定的标识分块加载），我没办法看到这款模板的具体 CSS。而正式安装获得源码后，我傻眼了……满篇的 &lt;div class="clear"&gt;&lt;/div&gt; 啊有木有，不管有没有 float 都要给你加一个 clear 啊。让我这个前端工程师情何以堪！

除此之外整个 HTML 代码结构也是千奇百怪，没有缩进，没有注释，什么都没有……

到现在为止，我使用的模板已经被我改掉了 60 % 以上，这款模板买来就是为了帮我更快的熟悉 Discuz（还好是雇主掏钱）。
<h2>社区分享精神差</h2>
这才是最让我诟病的，你在搜索引擎使用『Discuz + 你的问题』搜索出来的结果，大部分是像这样的：

>楼主：我遇到了某某问题，blahblahblah

>楼主：顶顶顶，求大神

>楼主：怎么没有人啊

>路人甲：不知道，帮顶

>路人乙：这个问题你可以通过买模板解决（附上一个付费模板的地址）

>楼主：再次狂顶！

>......若干天后

>楼主：已解决，关帖

就这样，后人再搜索到同样的问题后，只看到前人这样的『恶意』，着实让人心寒。当然我这么说有点道德绑架，也有点玻璃心，我没有权利要求别人分享自己的知识。但是我认为，一个程序是否强大、生命力是否顽强，就是看社区中每一个开发者的素质和能力。如果大家都没有分享精神，那么这个程序是很难继续发展的。

在这里，不得不提到我熟悉的 Wordpress，整个社区的氛围实在是太好了，再配合 StackOverflow，基本可以做到没有你找不到的答案。
<h2>小礼物</h2>
以上吐槽完毕，为了给 Discuz 社区奉献一点爱心，我分享一个一键清除 Discuz 缓存的代码，经常调试模板的同学应该会用到。将以下脚本放到网站根目录，通过访问 http://你的域名/clear_cache?type=1,2,3 一键清除 Discuz 缓存。其中 type 可以取 1、2或3，多个值用英文逗号（,）分割，分别是：
<ul>
	<li>1： CSS缓存</li>
	<li>2： 模板缓存</li>
	<li>3： DIY模块缓存</li>
</ul>
使用该脚本会验证登录的 cookies，所以需要在登录 Discuz 后台后使用。

<a href="https://gist.github.com/jasonslyvia/9227297" target="_blank">Gist</a>
<pre class="lang:php decode:true">&lt;?php

require './source/class/class_core.php';
require './source/function/function_misc.php';
require './source/function/function_forum.php';
require './source/function/function_admincp.php';
require './source/function/function_cache.php';

$discuz = C::app();
$discuz-&gt;init();

$admincp = new discuz_admincp();
$admincp-&gt;core  = &amp; $discuz;
$admincp-&gt;init();

$type = explode(',', $_GET['type']);

    //更新数据缓存
    if(in_array(1, $type)) {
        updatecache();
        require_once libfile('function/group');
        $groupindex['randgroupdata'] = $randgroupdata = grouplist('lastupdate', array('ff.membernum', 'ff.icon'), 80);
        $groupindex['topgrouplist'] = $topgrouplist = grouplist('activity', array('f.commoncredits', 'ff.membernum', 'ff.icon'), 10);
        $groupindex['updateline'] = TIMESTAMP;
        $groupdata = C::t('forum_forum')-&gt;fetch_group_counter();
        $groupindex['todayposts'] = $groupdata['todayposts'];
        $groupindex['groupnum'] = $groupdata['groupnum'];
        savecache('groupindex', $groupindex);
        C::t('forum_groupfield')-&gt;truncate();
        savecache('forum_guide', '');
        if($_G['setting']['grid']['showgrid']) {
            savecache('grids', array());
        }

        echo "数据缓存更新成功&lt;br /&gt;";
    }
    //模板缓存
    if(in_array(2, $type)) {
        cleartemplatecache();
        echo "模板缓存更新成功&lt;br /&gt;";
    }
    //模块缓存
    if(in_array(3, $type)) {
        include_once libfile('function/block');
        blockclass_cache();

        echo "DIY模块缓存更新成功&lt;br /&gt;";
    }

    exit();

?&gt;</pre>
明日将抵杭州，新的开始。