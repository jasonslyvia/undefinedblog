---
title: 从写 node.js 爬虫说起
permalink: talking-about-nodejs-crawler
id: 149
updated: '2014-04-14 12:26:40'
date: 2013-11-04 04:53:05
tags:
---

前一阵子在学校内网填报奖学金时意外发现一个漏洞可以查看任意学生的详细资料，包括父母、成绩、身高、体重<del>、感情状态</del>等……这些个信息如果我没记错的话是大学入学时自己亲手填的，所以真实性很高。

本着科学研究、有漏洞不用过期作废（其实我高度怀疑学校会不会修复这个漏洞，见之前<a title="看上去很美 为什么JSONP跨域获取cookie登录老师邮箱不现实" href="http://undefinedblog.com/2012/12/jsonp-cross-domain-get-cookie/" target="_blank">研究学校内网邮箱盗取cookie的文章</a>）的精神，我觉得把这些信息都爬下来好好研究一番。刚好马上要毕业了想做个数据可视化方向的毕设，这下源数据不都有了么！

总而言之，言而总之，突然就想用 node.js 写这么一个玩意儿，下面就要好好的说说写 node.js 爬虫这几天我的收获。

其实写爬虫的思路十分简单：
<ol>
	<li>按照一定的规律发送 HTTP 请求获得页面 HTML 源码（必要时需要加上一定的 HTTP 头信息，比如 cookie 或 referer 之类）</li>
	<li>利用正则匹配或第三方模块解析 HTML 代码，提取有效数据</li>
	<li>将数据持久化到数据库中</li>
</ol>
但是真正写起这个爬虫来，我还是遇到了很多的问题（和自己的基础不扎实也有很大的关系，node.js 并没有怎么认真的学过）。主要还是 node.js 的异步和回调知识没有完全掌握，导致在写代码的过程中走了很多弯路。
<h2>模块化</h2>
模块化对于 node.js 程序是至关重要的，不能像原来写 PHP 那样所有的代码都扔到一个文件里（当然这只是我个人的恶习），所以一开始就要分析这个爬虫需要实现的功能，并大致的划分了三个模块。
<ul>
	<li>主程序，调用爬虫模块和持久化模块实现完整的爬虫功能</li>
	<li>爬虫模块，根据传来的数据发送请求，解析 HTML 并提取有用数据，返回一个对象</li>
	<li>持久化模块，接受一个对象，将其中的内容储存到数据库中</li>
</ul>
模块化也带来了困扰了我一个下午的问题：<strong>模块之间的异步调用导致数据错误</strong>。其实我至今都不太明白问题到底出在哪儿，鉴于脚本语言不那么方便的调试功能，暂时还没有深入研究。

另外一点需要注意的是，模块化时尽量慎用全局对象来储存数据，因为可能你这个模块的一个功能还没有结束，这个全局变量已经被修改了。
<h2>Control Flow</h2>
这个东西很难翻译，直译叫控制流（吗）。众所周知，node.js 的核心思想就是异步，但是异步多了就会产生好几层嵌套，代码实在难看。这个时候，你需要借助一些 Control Flow 模块来重新整理你的逻辑。在这里就要推荐开发社区十分活跃，用起来也很顺手的 async.js（[https://github.com/caolan/async/](https://github.com/caolan/async/)）。

async 提供了很多实用的方法，我在写爬虫时主要用到了
<ul>
	<li>async.eachSeries(arr, fn, callback)  依次把 arr 中的每一个元素传给 fn，若 fn 回调没有返回错误对象就继续传下一个，否则把错误对象传给 callback，循环结束</li>
	<li>async.parallel(fn[, fn] , callback)  当所有的 fn 都执行完成后执行 callback</li>
</ul>
这些控制流方法给爬虫的开发工作带来了很大的方便。考虑这么一个应用场景，你需要把若干条数据插入数据库（属于同一个学生），你需要在所有数据都插入完成后才能返回结果，那么如何保证所有的插入操作都结束了呢？只能是层层回调保证，如果用 async.parallel 就方便多了。

这里再多提一句，本来保证所有的插入都完成这个操作可以在 SQL 层实现，即 transaction，但是 node-mysql 截止我使用的时候还是没有很好的支持 transaction，所以只有自己手动用代码保证了。
<h2>解析 HTML</h2>
在解析过程中也遇到一些问题，这里一并记录下来。

最基本的发送 HTTP 请求获得 HTML 代码，使用 node 自带的 http.request 功能即可。如果是爬简单的内容，比如获得某个指定 id 元素中的内容（常见于抓去商品价格），那么正则足以完成任务。但是对于复杂的页面，尤其是数据项较多的页面，使用 DOM 会更加方便高效。

而 node.js 最好的 DOM 实现非 cheerio（[https://github.com/MatthewMueller/cheerio](https://github.com/MatthewMueller/cheerio) ）莫属了。其实 cheerio 应该算是 jQuery 的一个针对 DOM 操作优化和精简的子集，包含了 DOM 操作的大部分内容，去除了其它不必要的内容。使用 cheerio 你就可以像用普通 jQuery 选择器那样选择你需要的内容。
<h3>下载图片</h3>
在爬数据时，我们可能还需要下载图片。其实下载图片的方式和普通的网页没有太大的区别，但是有一点让我吃了苦头。

注意下面代码中言辞激烈的注释，那就是我年轻时犯下的错误……
<pre class="lang:js decode:true">  var req = http.request(options, function(res){

    //初始化数据！！！
    var binImage = '';

    res.setEncoding('binary');
    res.on('data', function(chunk){
      binImage += chunk;
    });

    res.on('end', function(){

      if (!binImage) {
        console.log('image data is null');
        return null;
      }

      fs.writeFile(imageFolder + filename, binImage, 'binary', function(err){
        if (err) {
          console.log('image writing error:' + err.message);
          return null;
        }
        else{
          console.log('image ' + filename + ' saved');
          return filename;
        }
      });
    });

    res.on('error', function(e){
      console.log('image downloading response error:' + e.message);
      return null;
    });
  });

  req.end();</pre>
<h3>GBK 转码</h3>
另外一个值得说明的问题就是 node.js 爬虫在爬 GBK 编码内容时转码的问题，其实这个问题很好解决，但是新手可能会绕弯路。这里就把源码全部奉上：
<pre class="lang:js decode:true"> var req = http.request(options, function(res) {
   res.setEncoding('binary');
   res.on('data', function (chunk) {
    html += chunk;
   });

   res.on('end', function(){
    //转换编码
    html = iconv.decode(html, 'gbk');
   });
  });

  req.end();</pre>
这里我使用的转码库是 iconv-lite（[https://github.com/ashtuchkin/iconv-lite](https://github.com/ashtuchkin/iconv-lite)），完美支持 GBK 和 GB2312 等双字节编码。
<h2>小结</h2>
其实这个爬虫还有很大的提升空间，包括如何合理的利用 node 的异步功能加快爬取的速度（现在的速度基本上也就是 1 秒一个页面，主要时间花费在了 HTTP 请求上），如何更好的组织代码等。

当然，现在手上有了这么多数据，如何好好的分析一下也是需要认真考虑的。

<img class="size-full wp-image-207139" alt="nodejs crawler" src="http://undefinedblog.com/wp-content/uploads/2013/11/屏幕快照-2013-11-03-下午8.49.05.png" width="684" height="480" />

如果你有兴趣研究这个项目的完整代码，移步[bjut_crawler](https://github.com/jasonslyvia/bjut_crawler)，顺便也研究一下你们的学校是否有这样的漏洞！