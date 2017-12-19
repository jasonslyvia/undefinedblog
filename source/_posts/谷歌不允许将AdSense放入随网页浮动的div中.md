---
title: 谷歌不允许将AdSense放入随网页浮动的div中
tags: 

  - css
  - google
permalink: adsense-not-allowed-in-fixed-float-div
id: 74
updated: '2012-06-27 22:26:27'
date: 2012-06-27 22:26:27
---

<p>当初发现通过 JavaScript 和 CSS 配合可以实现在页面的 offsetTop 达到一定的数值后让 div 层固定的技术，这也就是常见的侧边栏最后一项固定的效果。当时第一个想到的就是这么好的资源一定要用来放广告啊！</p>
<p>不过今天收到了谷歌的邮件，提醒我广告摆放有违规现象，需要在 72 个小时内整改。邮件全文如下：</p>
<blockquote>
<p>本电子邮件地址只作通知用途，不接收回信，因此请勿回复本邮件。<br /> -------------------------------------------------------------------------------------------------------------------------------</p>
<p>您好！</p>
<p>在最近一次对您的帐户进行的审核中，我们发现您目前展示 Google 广告的方式不符合我们的合作规范 (<a href="https://www.google.com/support/adsense/bin/answer.py?answer=48182&amp;stc=aspe-1pp-cn" target="_blank">https://www.google.com/support/adsense/bin/answer.py?answer=48182&amp;stc=aspe-1pp-cn</a>)。</p>
<p>--------------------------------------------------<br /> 示例网页：<a href="http://www.ppios.com/ios-5-1-1-upgrade-tutorial.html" target="_blank">http://www.ppios.com/ios-5-1-1-upgrade-tutorial.html</a></p>
<p>请注意：此网址仅是一个示例，相同的违规情况可能存在于您网站中的其他网页或您的联盟网络中的其他网站。</p>
<p>发现的违规情况：</p>
<p>刻意突出：发布商不得鼓励用户点击 Google 广告，也不得设法让用户过分关注广告单元。例如，您的网站不得使用图形、箭头等符号或误导性图片，将用户的注意力引导到广告上，<strong>也不得将 Google 广告放置在浮动框脚本中</strong>。您可以在我们的帮助中心找到有关此政策的详情 (<a href="https://www.google.com/adsense/support/bin/answer.py?hl=cn&amp;answer=115985" target="_blank">https://www.google.com/adsense/support/bin/answer.py?hl=cn&amp;answer=115985</a>)。</p>
<p>您需要进行的操作：请在 72 小时内对您的网页进行所有必要的更改。</p>
<p>如果在上述时间段内改正了违规行为，广告投放将不会受到影响。如果未进行相应的更改和/或在审核过程中发现了其他违规情况，我们将停止向您的网站投放广告。</p>
<p>帐户状态：有效</p>
<p>您的 AdSense 帐户仍然有效。但是，如果我们继续发现问题，则有可能停用您的整个帐户。因此，我们建议您抽出时间检查您的联盟网络中的其他网页，确保您的所有其他网页都符合我们的规范。</p>
</blockquote>
<p>搜索了一下，发现谷歌的 AdSense 讨论组里也有用户明确的提出了不应该把广告代码放到固定的 div 中，估计也是因此被谷歌警告了吧！虽然看到很多国内的网站包括糗事百科都曾这么做过，既然不允许，那只好乖乖改掉啦！</p>