---
title: 不要信任CNNIC证书
tags: 

  - https
  - security
  - ssl
permalink: do-not-trust-cnnic-certificate
id: 132
updated: '2013-01-28 14:00:18'
date: 2013-01-28 14:00:18
---

<h2>CNNIC 是干什么的</h2>
<p>CNNIC 不是企业、不是事业、不是社团。截至2001年7月，它业已卖掉12万个cn域名，CNNIC仅此一项，年毛收入就可以达到近4000万元，而且这个数字还在飞速增长之中。另外，CNNIC还在做中文域名注册、流量认证、通用网址注册、认证培训等十分赚钱的业务。</p>
<p>上级主管单位：工信部（没错，就是搞绿坝的那货）[1]</p>
<h2>CNNIC证书是干什么的</h2>
<p>浏览网站时，你可能注意到部分网址是 https 开头的，这表明你与该网站服务器之间的链接是加密的，其他人无法获取的具体内容信息。为了建立安全链接，网站的服务器一般都会使用 SSL 证书，这些证书由第三方机构颁布，保证了权威性和安全性。[2]</p>
<p>CNNIC证书也可以看做是这样一个证书，当你和一个网站建立安全连接时，如果使用的证书是 CNNIC 证书，系统（浏览器）也会默认这个连接是安全的。</p>
<h2>为什么不能信任CNNIC证书</h2>
<p>首先，CNNIC本身就不是好鸟：[3]</p>
<ul>
<li>长期发布流氓软件</li>
<li>域名管理混乱</li>
<li>强行霸占域名</li>
<li>禁止个人注册域名</li>
<li>政策朝令夕改</li>
</ul>
<p>其次，如果你信任了 CNNIC 证书，可能导致：[4]</p>
<ul>
<li>中间人攻击（非常重要，CNNIC证书配合GFW，走遍天下无敌手）</li>
<li>ActiveX攻击</li>
</ul>
<p>最后，CNNIC 证书已经被 Windows、Mozilla 等大厂商广泛接受被作为系统内置默认可信任的证书。因此，我们需要手动清理 CNNIC 证书。[5]<br /> <img class="origin_image" style="border-style: none; margin: 6px 0px; display: block; overflow: hidden; max-width: 100%; height: auto; cursor: -webkit-zoom-in;" title="点击查看原图" src="http://p3.zhimg.com/22/42/22425cd89e089ea16ecedd42e04ecbcf_m.jpg" alt="" /></p>
<h2>参考资料</h2>
<p>[1][3]&nbsp;<a href="http://program-think.blogspot.com/2010/02/about-cnnic.html">http://program-think.blogspot.com/2010/02/about-cnnic.html</a><br /> [2]&nbsp;<a href="http://www.ioshin.com/digitalsignature/SSL%20and%20HTTPS.html">HTTPS,HTTP和SSL有什么联系-iOShin SSL证书专题</a><br /> [4]&nbsp;<a href="http://blog.csdn.net/program_think/article/details/5309699">CNNIC证书的危害及各种清除方法</a><br /> 如何清除 CNNIC 证书见参考资料 4</p>