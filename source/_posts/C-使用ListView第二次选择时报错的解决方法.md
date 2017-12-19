---
title: 'C#使用ListView第二次选择时报错的解决方法'
permalink: cshi-yong-listviewdi-er-ci-xuan-ze-shi-bao-cuo-de-jie-jue-fang-fa
id: 18
updated: '2011-10-26 04:46:37'
date: 2011-10-26 04:46:37
tags:
---

<p>在C#中使用ListView，当单击一个item时一切正常，但是再点击另一个item时提示：</p>
<blockquote>
<p>&ldquo;System.ArgumentOutOfRangeException&rdquo;类型的未经处理的异常出现在 System.Windows.Forms.dll 中。<br /> 其他信息: InvalidArgument=&ldquo;0&rdquo;的值对于&ldquo;index&rdquo;无效。</p>
</blockquote>
<p><strong>问题原因</strong><br /> listView 的selectindexChange事件每次都要执行两次,第一次将选择的数目,就是selectitems.count清空,然后才重新指定,所以在第二次里selectitems.count = 0 ,也就是没有选择任何项,所以报了index异常.<br /> <strong>解决办法</strong><br /> 在selectindexchange事件中,用If语句屏蔽一地此indexchange的一地此执行<br /> <strong>修正后的代码</strong><br /> [cc lang="c#"]<br /> private void listView1_SelectedIndexChanged(object sender, EventArgs e)<br /> {<br /> if (listView1.SelectedItems.Count == 0)<br /> {<br /> //屏蔽第一次改变<br /> }<br /> else<br /> {<br /> MessageBox.Show(listView1.SelectedItems[0].Text);<br /> }<br /> }<br /> [/cc]</p>