---
title: '淘宝客API正确调用方法[PHP]'
tags: 

  - api
  - php
  - 淘宝客
permalink: invoke-taobaoke-api-in-correctly
id: 128
updated: '2013-04-25 20:38:08'
date: 2013-01-06 21:10:50
---

好吧，又是一篇跟 iOS 开发无关的文章……我有罪……

不过话说回淘宝客 API 的调用，还是很坑爹的，光是各种参数的拼凑就花了我两天的时间去琢磨，还有 timestamp 变成一个莫名其妙的符号的问题……总之就是各种闹心。记得当初解决这个问题的时候网上相关的内容甚少，不知道现在有没有贴出相关的使用方法，如果没有的话……这篇文章就算积了德了。

<a href="http://open.taobao.com/doc/api_cat_detail.htm?spm=0.0.0.42.B7Gt8w&amp;cat_id=38&amp;category_id=102">淘宝客 API 官方地址</a>

下面就贴代码，注释里会解释用法（以 taobao.taobaoke.items.get 这个 API 为例）

看下面的代码前假设你已经申请了淘宝开放平台的账号，知道啥是 pid，啥是 app_key，啥是 app_secret
<pre class="lang:php decode:true brush: php;fontsize: 100; first-line: 1;">//拼凑请求 api 的地址
//参数为需要查询的商品名称
function para($keyword){
	$para = array(
		'timestamp'=&gt;date('Y-m-d H:i:s'), //设置 timestamp 参数，无需变更
		'v'=&gt;'2.0',//api 版本无需变更
		'app_key'=&gt;'12345678',//app_key，输你自己的！
		'method'=&gt;'taobao.taobaoke.items.get',//使用的 api 类型，自己看着调吧
		'partner_id'=&gt;'top-apitools',//貌似不用改，忘记了= =
		'format'=&gt;'json',//返回格式，json 或 xml
		'sort'=&gt;'credit_desc',//排序类型，有很多，credit_desc 是按信用降序，具体看官方文档
		'keyword'=&gt;$keyword,//要查找的商品名称
		'pid'=&gt;'12345678',// pid，输自己的！
		'fields'=&gt;"title,pic_url,price,click_url"//需要返回的数据类型，由于我只需要标题、图片、价格和链接，所以我只填了这4个，具体还可以返回什么看官方文档！
	);
	return $para;
}

//生成签名，这个步骤最坑爹
function set_sign($keyword){
	$secret = "ooxxooxooxx1234ooxxooxx1234";//secret，输自己的！！
	$para1 = para($keyword);//拼凑参数
	ksort($para1);//排序，官方要求，不是为了美观好不好= =
	foreach($para1 as $key =&gt; $value){//URL 拼起来，为了下面加密生成签名用
		$uri .= $key . $value;
	}
	$sign = strtoupper(md5($secret. $uri));//加密一炮，这个要求也是淘宝官方的，把secret和uri拼起来，然后md5，再全部取大写
	return $sign;
}

//请求数据
function get_result($sign, $para){
	$pa = "";
	foreach($para as $key =&gt; $value){
		if($key == 'keyword' || $key == 'timestamp')//注意 urlencode
			$value = urlencode($value);
		$pa .= $key . '=' .$value . '&amp;';
	}
	$pa = substr($pa, 0, -1);
	$url = "http://gw.api.taobao.com/router/rest?sign=". $sign.'&amp;'. $pa;
	$ch = curl_init($url);
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
	$result = curl_exec($ch);
	curl_close($ch);
	$r = json_decode($result);//返回数据结果，爱咋用咋用！
}</pre>
具体用法也很简单，直接调用 get_result 函数
<pre class="lang:php decode:true brush: php;fontsize: 100; first-line: 1;">get_result(set_sign("iphone"), para("iphone"));//突然发现get_result这个函数设计的也很坑爹，懒得优化了，将就着用吧！</pre>
20130425更新：接到淘宝的通知，说我的淘宝客 API 调用频率过低，要封掉我的权限……