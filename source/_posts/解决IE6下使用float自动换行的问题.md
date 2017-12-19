---
title: 解决IE6下使用float自动换行的问题
tags: 

  - css
  - ie6
permalink: jie-jue-ie6xia-shi-yong-floatzi-dong-huan-xing-de-wen-ti
id: 36
updated: '2011-12-15 05:59:52'
date: 2011-12-15 05:59:52
---

<p>IE6实在是太蛋疼了，各种问题&hellip;&hellip;</p>
<p>若遇到在其他浏览器，IE8+下均正常，在IE6下元素就会错位、自动换行的问题，请把含有float标签的元素放在该层的第一位</p>
<p>如：</p>
<p>&lt;div&gt;something&lt;/div&gt;&lt;div style="float:right"&gt;something else&lt;/div&gt;</p>
<p>改为</p>
<p>&lt;div style="float:right"&gt;something else&lt;/div&gt;&lt;div&gt;something&lt;/div&gt;</p>
<p>另外若同一行里其他元素使用了 position:relative也会导致自动换行、元素下沉的问题</p>