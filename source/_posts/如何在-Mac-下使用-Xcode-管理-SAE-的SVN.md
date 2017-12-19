---
title: 如何在 Mac 下使用 Xcode 管理 SAE 的SVN
tags: 

  - mac
  - xcode
permalink: ru-he-zai-mac-xia-shi-yong-xcode-guan-li-sae-de-svn
id: 49
updated: '2012-02-10 08:31:04'
date: 2012-02-10 08:31:04
---

<p>现在新浪的SAE全面推行SVN方式部署代码了，下面大概讲一下如何在Xcode下部署新浪SAE的SVN，网上的资料都稍微有些复杂，我根据网上的资料进行了一定的简化。</p>
<p>1、创建一个新的应用，假设名为 ppios</p>
<p>2、打开Xcode，点击右上方的 Organizer，选择 Repositories，点击左下方的 + 号</p>
<p>3、点 [Add Repository]</p>
<p>4、弹出对话框中，name随意填，location填</p>
<blockquote>
<p>https://svn.sinaapp.com/<span style="color: #ff0000;">ppios &nbsp; &nbsp;(替换成你的应用名称)</span></p>
</blockquote>
<p>5、type选[subversion]，再点next</p>
<p>6、这个时候Xcode会提示输入用户名密码，这个是你的SAE账号密码，不是微博的</p>
<p>7、这个时候SAE的repo已经建好了，如果提示&ldquo;unable to load revisions&rdquo;，请打开终端，输入以下命令</p>
<blockquote>
<p>svn ls https://svn.sinaapp.com/<span style="color: #ff0000;">ppios</span></p>
</blockquote>
<p>终端会提示&ldquo;Error vaildating server &nbsp;certificate&rdquo;，输入 p 即可（Permanently Accept）</p>
<p>8、回到Xcode 发现已经可以正常加载所有的修改数据了</p>