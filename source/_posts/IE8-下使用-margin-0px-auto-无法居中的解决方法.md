---
title: 'IE8 下使用 margin:0px auto 无法居中的解决方法'
tags: 

  - css
  - ie8
permalink: ie8-xia-shi-yong-margin0px-auto-wu-fa-ju-zhong-de-jie-jue-fang-fa
id: 48
updated: '2012-02-03 09:18:33'
date: 2012-02-03 09:18:33
---

<p><strong>源代码：</strong></p>
<p>[cc lang="html"]</p>
<div style="margin: 0px auto; width: 200px;">//想要居中的元素</div>
<p>[/cc]</p>
<p>在FF、Chrome下都可以完美居中，但是在IE8下就不行，解决方法让如下：</p>
<p><strong>新代码：</strong><br /> [cc lang="html"]</p>
<div style="text-align: center; width: 300px; overflow: hidden;">
<div style="margin: 0px auto;">//想要居中的元素</div>
</div>
<p>[/cc]<br /> 具体改动是添加一个</p>
<div>标签将想要居中的元素括起来，并且保证大
<div>的宽度大于想要居中的
<div>，并添加text-align和overflow属性
<p>&nbsp;</p>
</div>
</div>
</div>