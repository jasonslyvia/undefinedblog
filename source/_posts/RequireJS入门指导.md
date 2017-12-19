---
title: RequireJS入门指导
tags: 

  - javascript
  - require js
permalink: primer-for-require-js
id: 136
updated: '2013-04-25 20:33:30'
date: 2013-03-27 00:42:15
---

最近在百度实习做的一个项目用到了 <strong>Require JS </strong>这个库，之前从来没有了解过，经过一番大概的搜索后找到一篇非常不错的文章，<strong>看完后能够让你对 Require JS 的运行机制、使用方法以及为什么使用 Require JS 有一个基础的认识</strong>。下面我就对这篇文章进行一个简单的翻译，如果有错漏的地方希望指正。

原文链接：<a href="http://netmvc.blogspot.be/2012/11/javascript-modularity-with-requirejs.html">http://netmvc.blogspot.be/2012/11/javascript-modularity-with-requirejs.html</a>
<h2>谁适合读这篇文章</h2>
对 JavaScript 有一定的了解，写过一定规模的 JavaScript 代码，尤其能够掌握 JavaScript 匿名函数、回调等概念，并希望了解 Require JS 的用法、概念、意义等。
<h2>一些基本信息</h2>
<strong>Require JS</strong>(<a href="http://requirejs.org/">官网</a>) 是一个 JavaScript 文件或模块的加载器，它可以根据不同页面对不同模块的需求，按照你的设置依次加载并执行所有的 JavaScript 文件。使用 Require JS 主要有两个目的：
<ul>
	<li>解耦代码，方便代码重用和管理</li>
	<li>加速页面加载，内置优化引擎（需服务器支持），自动异步加载依赖项，将所有需要加载的文件自动合并为一个文件，减少 HTTP 请求</li>
</ul>
Require JS 同时支持整合 jQuery、CommonJS、Node JS 等，而且对浏览器有不错的兼容性（IE 6+，你懂的）。

&nbsp;
<h2>没有 Require JS 的时候</h2>
下面我们将以一个没有使用 Require JS 的模块开始，讲讲如何将你的代码解耦并使用 Require JS 来加载：
<pre class="lang:xhtml decode:true">&lt;html xmlns="http://www.w3.org/1999/xhtml"&gt;
&lt;head&gt;
    &lt;title&gt;Phase 1&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;div&gt;
        &lt;h1&gt;Modular Demo 1&lt;/h1&gt;
    &lt;/div&gt;
    &lt;div id="messagebox"&gt;&lt;/div&gt;

    &lt;script src="../Scripts/jquery-1.8.2.min.js" type="text/javascript"&gt;&lt;/script&gt;

    &lt;script type="text/javascript"&gt;

        var baseUrl = '/api/messenger/';

        function ShowMessage(id) {
            $.ajax({
                url: baseUrl + id,
                type: 'GET',
                dataType: 'json',
                success: function(data) {
                    $("#messagebox").html(data);
                }
            });
        }

        var id = 55;
        ShowMessage(id);

    &lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;</pre>
以上是一个很简单而且很常见的 ajax 请求页面，可以看到这个页面里的代码虽然简单，但是耦合度很高，下面我们将其中的 JavaScript 代码解耦，把不同的模块独立出来。
<pre class="lang:js decode:true brush: jscript;fontsize: 100; first-line: 1;">//模块1，全局设置模块
        var config = (function() {
            var baseUrl = '/api/messenger/';

            return {
                baseUrl: baseUrl
            };
        })();

//模块2，请求数据模块
        var dataservice = (function($, config) {
            var callApi = function (url, type, callback) {
                    $.ajax({
                        url: url,
                        type: type,
                        dataType: 'json',
                        success: function (data) {
                            callback(data);
                        }
                    });
                },
                getMessage = function (id, callback) {
                    url = config.baseUrl + id;
                    callApi(url, 'GET', callback);
                };

            return {
                getMessage: getMessage
            };
        })($, config);

//模块3，封装的一个 messenger 模块，显示 ajax 放回的数据
        var messenger = (function ($, dataservice) {
            var showMessage = function(id) {
                dataservice.getMessage(id, function(message) {
                    $("#messagebox").html(message);
                });
            };

            return {
                showMessage: showMessage
            };
        })($, dataservice);

//模块4，匿名函数，传 id, 调用 messenger 的showMessage() 方法 （直接执行）
        (function (messenger) {
            var id = 55;
            messenger.showMessage(id);
        })(messenger);</pre>
将这些代码放到原来的、没有重构前的页面中去，将实现同样的效果。但是我们将这些功能独立出来后，就可以很好的重复利用部分代码。

下面假设我们将这 4 个模块放到 4 个独立的 js 文件中：
<ul>
	<li>模块1：config.js</li>
	<li>模块2：dataservice.js</li>
	<li>模块3：messenger.js</li>
	<li>模块4：main.js</li>
</ul>
这样，在我们的 html 页面中，我们需要依次加载这 4 个模块：
<pre class="brush: xml;fontsize: 100; first-line: 1;">&lt;html xmlns="http://www.w3.org/1999/xhtml"&gt;
&lt;head&gt;
    &lt;title&gt;Phase 1&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;div&gt;
        &lt;h1&gt;Modular Demo 1&lt;/h1&gt;
    &lt;/div&gt;
    &lt;div id="messagebox"&gt;&lt;/div&gt;

    &lt;script src="../Scripts/jquery-1.8.2.min.js" type="text/javascript"&gt;&lt;/script&gt;

    &lt;script src="config.js" type="text/javascript"&gt;&lt;/script&gt;
    &lt;script src="dataservice.js" type="text/javascript"&gt;&lt;/script&gt;
    &lt;script src="messenger.js" type="text/javascript"&gt;&lt;/script&gt;
    &lt;script src="main.js" type="text/javascript"&gt;&lt;/script&gt;

&lt;/body&gt;
&lt;/html&gt;</pre>
这个时候，代码一样可以工作的非常顺利，没有任何问题。但是需要考虑一个问题：<strong>js 文件的加载顺序</strong>。当一个模块2需要依赖模块1中的功能时，必须要求模块1加载完成后再执行模块2中的代码。对于简单的代码（如本文涉及的仅4个模块）还好考虑，若有数十个模块需要加载，那么依赖的顺序一旦处理出错，很可能就会报 xxx is undefined 错误。

这个时候就是 Require JS 需要大显身手的时候。
<h2>Require JS 的基础操作</h2>
<h3>定义模块</h3>
<pre class="lang:js decode:true brush: jscript;fontsize: 100; first-line: 1;">define(['jquery'],
    function ($) {
        var text = 'I am a module',
            showMessage = function() {
                $("#messagebox").html(text);
            };

        return {
            showMessage: showMessage
        };
    }
);</pre>
可以看到，一个 JavaScript 模块（如上面提到的 config、dataservice、messenger 等），都被一个 define() 函数包起来。

其中 define() 函数接受 3 个参数：
<ul>
	<li>模块名</li>
	<li>依赖项，以数组形式传递</li>
	<li>模块内容</li>
</ul>
但是注意，上面的模块定义中，我们并没有传 <span style="text-decoration: underline;">模块名</span> 这个参数，理由如下：

Require JS 会自动根据文件名（不含后缀名，即不含 ".js"）来给定义的模块命名，这样在最后优化（将所有的 js 文件合并为一个）时就能自动调用所有的文件。如果你显示的给定义的一个模块命名了，那么在日后进行修改时（比如改动了文件名或模块名），Require JS 将无法自动加载。因此最佳实践是：<strong>不要显式的定义模块名，让文件名成为默认的模块名，方便 Require JS 管理。</strong>

说完了第一个参数，再看看第二个参数 <span style="text-decoration: underline;">依赖项</span>。依赖项我们传递的是一个字符串数组，每个字符串均代表要实现本模块的功能，需要预先加载的其他模块的模块名。如上例中我们的依赖项就是 jquery。

最后一个参数就是我们需要定义的模块内容。在上例中，我们的模块是一个 function，其中函数的形参即为对依赖项的引用。比如我们的模块有两个依赖项：
<pre class="lang:js decode:true brush: jscript;fontsize: 100; first-line: 1;">define(['moduleA','moduleB'],function(a,b){
        //a.blabla(); b.blabla = ooxx;
});</pre>
就可以在模块中分别使用形参来调用我们依赖的模块。<strong>需要注意的一点是，依赖项的书写顺序和在模块定义的函数中的形参顺序要一致。</strong>

另外，Require JS 定义的模块并不一定是 function，也可以是普通的对象，详见 <a href="http://requirejs.org/docs/api.html#define">Require JS 文档</a>。

<strong>将解耦后的代码改写为 Require JS 模块</strong>

我们在上面把一个简单的 ajax 调用解耦成 4 个独立的模块，下面我们就把这 4 个模块根据 Require JS 定义模块的风格进行改写。
<pre class="lang:js decode:true brush: jscript;fontsize: 100; first-line: 1;">//config.js
define([],
    function () {
        var baseUrl = '/api/messenger/';

        return {
            baseUrl: baseUrl
        };
    }
);

//dataservice.js
define(['jquery', 'config'],
    function ($, config) {
        var
            callApi = function (url, type, callback) {
                $.ajax({
                    url: url,
                    type: type,
                    dataType: 'json',
                    success: function (data) {
                        callback(data);
                    }
                });
            },

            getMessage = function (id, callback) {
                url = config.baseUrl + id;
                callApi(url, 'GET', callback);
            };

        return {
            getMessage: getMessage
        };
    }
);

//messenger.js
define(['jquery', 'dataservice'],
    function ($, dataservice) {
        var showMessage = function (id) {
            dataservice.getMessage(id, function (message) {
                $("#messagebox").html(message);
            });
        };

        return {
            showMessage: showMessage
        };
    }
);

//main.js
(function() {
    requirejs.config(
        {
            paths: {
                'jquery': '../Scripts/jquery-1.8.2.min'
            }
        }
    );

    require(
        ['messenger'],
        function(messenger) {
            var id = 55;
            messenger.showMessage(id);
        }
    );
})();</pre>
你可能注意到了 main.js 中定义模块的方式似乎不同于其它 3 个模块。没错，因为严格意义上来说，main.js 是调用了其它 3 个模块的功能并最终获得数据。因此下面就要引出怎么在定义了 Require JS 模块后最终设置和调用的问题。
<h3>设置和调用模块</h3>
Require JS 对于模块的设置和调用提供了两种方式：

1、定义一个 script 标签，并增加 data-main 属性，指向我们的设置文件；同时添加 src 属性，指向 require-js.js 文件（即主库）
<pre class="brush: jscript;fontsize: 100; first-line: 1;">&lt;script data-main="main.js" src="require-js.js"&gt;&lt;/script&gt;</pre>
2、先引入 Require JS 库，再显式的设置
<pre class="brush: bash;fontsize: 100; first-line: 1; crayon-selected">&lt;script src="require-js.js"&gt;&lt;/script&gt;
&lt;script&gt;
require({
        baseUrl : 'scripts', //设置查找所有模块的根目录
        paths:'',//如果在 baseUrl 下没有找到相应的模块，则加上 paths 继续查找
        shim:{},//加载依赖项，传对象
});
require(['dataservice.js']); //加载完上述所有依赖项后，再加载自己编写的模块
&lt;/script&gt;</pre>
详细参数见 <a href="http://requirejs.org/docs/api.html#config">Require JS 文档</a>。

以上就是 Require JS 的一些简单使用方法和实践，话再说回 iOS 开发……很快回归！