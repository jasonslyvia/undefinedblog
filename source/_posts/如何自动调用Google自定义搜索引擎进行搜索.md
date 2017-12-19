---
title: 如何自动调用Google自定义搜索引擎进行搜索
tags: 

  - google
permalink: auto-cse
id: 117
updated: '2013-04-25 20:39:17'
date: 2012-11-29 00:57:55
---

<a href="https://www.google.com/cse/" target="_blank">谷歌自定义搜索引擎</a>（CSE）为我们提供了很好的谷歌搜索支持，作为对博客搜索程序的扩充，Google搜索很好的完成了更高阶的搜索任务。然而试想这样一个场景，用户首先使用了Wordpress自带的搜索引擎进行搜索，并没有获取到想要的结果，假设你在搜索结果界面又放了一个Google自定义搜索框，用户还需要再次键入关键字点击搜索，才能转到Google自定义搜索结果页面。那么有没有办法让Google自定义搜索引擎自动进行搜索呢？答案是肯定的，我们需要借助一些简单的Google CSE API。

关于如何申请 CSE 这里不再赘述，假设你已经申请好了 CSE。

<strong>HTML</strong>
<pre class="brush: xml;fontsize: 100; first-line: 1;">&lt;div id="cse-search-form" style="width: 100%;display:none;"&gt;Loading&lt;/div&gt;</pre>
<strong>JS</strong>
<pre class="lang:js decode:true brush: jscript;fontsize: 100; first-line: 1;">&lt;script src="http://www.google.com.hk/jsapi" type="text/javascript"&gt;&lt;/script&gt;
&lt;script type="text/javascript"&gt;
  google.load('search', '1', {language : 'zh-CN'});
  google.setOnLoadCallback(function() {
	//获取需要搜索的关键词并执行搜索，如下两行为关键代码
	var query = '苹果教程网';
	customSearchControl.execute(query);
	var customSearchOptions = {};  var customSearchControl = new google.search.CustomSearchControl(
	  '001212031995655897722:skb-lu6f7zo', customSearchOptions);
	customSearchControl.setResultSetSize(google.search.Search.SMALL_RESULTSET);
	var options = new google.search.DrawOptions();
	options.setSearchFormRoot('cse-search-form');
	options.setAutoComplete(true);
	customSearchControl.setAutoCompletionId('001212031995655897722:skb-lu6f7zo+qptype:1');
	customSearchControl.draw('cse', options);
  }, true);
&lt;/script&gt;</pre>
<strong>搜索结果</strong>
<pre class="brush: xml;fontsize: 100; first-line: 1;">&lt;link rel="stylesheet" href="http://www.google.com/cse/style/look/default.css" type="text/css" /&gt;
&lt;div id="cse" style="width:100%;"&gt;&lt;/div&gt;</pre>
简单来说，整个功能的核心语句就是
<pre class="lang:js decode:true brush: jscript;fontsize: 100; first-line: 1;">customSearchControl.execute(query);</pre>
自行判断在合适的位置调用该语句即可实现自动搜索（无需用户点搜索按钮，关键词需要预先设置好，即query变量）