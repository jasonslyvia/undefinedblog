---
title: 使用 ASIHttpRequest 模拟登陆正方教务系统的几点心得
tags: 

  - asihttprequest
  - 模拟登录
  - 正方教务系统
permalink: >-
  shi-yong-asihttprequest-mo-ni-deng-lu-zheng-fang-jiao-wu-xi-tong-de-ji-dian-xin-de
id: 102
updated: '2013-04-25 20:40:44'
date: 2012-09-26 02:54:28
---

终于成功了！终于成功在 iOS 系统上模拟登陆了正方教务系统，这样我的 正方教务系统 iOS 客户端计划很快就会实现了！感谢 Chrome，感谢 ASIHttpRequest，感谢 StackOverFlow，感谢 Google，感谢 Apple！

其实有模拟登陆这个想法很久了，但是当时（大约1年前）的自己知识储备完全不够，根本不理解各种 HTTP 协议、状态码、header 什么的，cookies 也不懂，做网站的这一年多来学习的 B/S 知识真是大大的拓展了我的视野！当然不可否认的是将来学习的路还很长，还很远，我要继续努力！

这篇文章主要记录我使用 ASIHttpRequest 框架成功登陆正方教务系统（含验证码输入功能）的过程中的几点心得，记下来和大家一起分享！

1、忽视 __VIEWSTATE 参数，这个参数在提交表单是会自动提交，在 html 中它是一个 hidden 的 input 标签，这个 __VIEWSTATE 貌似是一个 ASP.NET 专属的 XX 玩意儿，如果你没有在 POST 操作中增加这一项，那么铁定不能成功。我的解决方法是第一次直接获取登陆界面，在 responseString 中利用子串把 __VIEWSTATE 的值提取并保存出来，好在提交表单时提交。
<pre class="lang:objc decode:true brush: cpp;fontsize: 100; first-line: 1;">NSString *preVIEW = [request responseString];

NSRange range = [preVIEW rangeOfString:@"&lt;input type="hidden" name="__VIEWSTATE" value=""];

self.code = [preVIEW substringWithRange:NSMakeRange(range.location + 47, 48)];</pre>
当然，我知道这样的代码一点都不美，但是它有效，而且不需要引入第三方正则库……所以，暂且就这样吧！、
<blockquote>更新：目前 iOS 已经内置正则支持，使用正则匹配来获取 viewstate 值是更好的方法。</blockquote>
2、忽视 URLENCODE，提交登陆表单时进行 URLENCODE 是一个基本的做法，其实 ASIHTTPREQUEST 已经帮我们实现了 URLENCODE。但是！但是！正方教务系统的 charset 是 gb2312，而 ASIHttpRequest 的默认编码格式是 UTF8，这样你 encode 完 post 过去依然无法登陆
<pre class="lang:objc decode:true brush: cpp;fontsize: 100; first-line: 1;">NSStringEncoding enc = CFStringConvertEncodingToNSStringEncoding (kCFStringEncodingGB_18030_2000);

[request setStringEncoding:enc];</pre>
加上这两句后再用 ASIFormDataRequest 提交数据时 urlencode 就会自动用 gb2312 了

3、cookie 提交，由于现在的教务系统需要输入验证码，而获取验证码会导致原来的 Session 失效，这个问题也困扰了我很久，最后的解决方法是在请求验证码（包括最后提交表单时）在 header 中设置 【Cookie】 为 Session ，并且同时提交 cookies（这个 cookies 在第一次发送请求并获得返回数据时保存）
<pre class="lang:objc decode:true brush: cpp;fontsize: 100; first-line: 1;">//第一次请求数据时保存 cookie

self.cookies = [request responseCookies];

//请求验证码时发送 cookies 同时在 header 设置 Session

NSArray *headerValue = [NSArray arrayWithObjects:self.session, nil];

NSArray *headerKey = [NSArray arrayWithObjects:@"Cookie", nil];

[request setRequestHeaders:[NSMutableDictionary dictionaryWithObjects:headerValue forKeys:headerKey]];

[request setRequestCookies:[self.cookies mutableCopy]];

//最后表单提交时同理</pre>
以上三点为主要收获，另外还有一些在实践中解决的困惑也记下来，免得大家重蹈覆辙：

1、在模拟提交表单时，并不需要设置太多的 header，这样反而会导致服务器出现 500 的 responsestatuscode。一般来说 header 里设置个 cookie 就够了。

2、若模拟登陆后存在一个页面跳转，ASIHttpRequest 会自动帮你跳转。比如登陆正方教务系统时，表单提交的 action 对象是 default2.aspx。如果登陆成功会在 header 中返回 location :/ xl.aspx=1xxxxxxx ，这个时候 ASIHttpRequest 会自动获取新的地址的数据并返回。这个功能可以通过设置进行取消，但是具体还没有研究。

基本就是这么多，在网上几乎搜不到关于正方教务系统的模拟登陆 demo，等我的客户端完全做好后会考虑开源给大家共同学习、进步！