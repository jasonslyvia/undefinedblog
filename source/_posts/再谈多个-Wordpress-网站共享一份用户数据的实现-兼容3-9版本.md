---
title: '再谈多个 Wordpress 网站共享一份用户数据的实现[兼容3.9版本]'
permalink: wordpress-multiple-site-same-user
id: 160
updated: '2015-02-02 17:24:08'
date: 2014-08-02 04:51:04
tags:
---

在若干年以前，我刚开始折腾Wordpress没多久的时候，就自己摸索过**[多个Wordpress网站共享一份数据表的实现方法](http://undefinedblog.com/ru-he-zai-duo-ge-wordpressbo-ke-jian-gong-xiang-yong-hu-xin-xi/)**。这种看起来好像很高大上的类SSO功能，能够给用户在多个网站之间提供快速、无缝、透明的登录体验。

举个很简单的例子，原本有一个Wordpress网站 `http://www.example.com（后称网站A）`，你突然想增加一个博客子站点`http://blog.example.com（后称网站B）`，那么原本在`网站A`注册的用户当然不想重新去`网站B`再次注册。这个时候就体现出本文要探讨的问题的价值了。

废话不多说，直接开始！

>以下例子中所有的域名example.com请自行替换成你的域名

>阅读本文的前提是你已经有基本的网站部署知识，会部署单个Wordpress网站，知道如何给Wordpress配置数据库信息，了解Wordpress目录结构。

##部署网站A
1. 如果还没有部署Wordpress，先按照常规方式下载源文件、配置环境（这里不赘述）。**记录好填入的数据库名、数据库用户名、数据库密码等信息，后续步骤要使用。**（备用数据A）

2. 编辑`wp-config.php`，在任意位置添加如下内容：

```php
define('COOKIE_DOMAIN', 'example.com');
define('COOKIEPATH', '/');
```
同时找到有一排形如`define('XXX_KEY', 'xxxasdas')`的内容（共8行），这里是设置一些salt，用来给cookie加密。如果没有也没有关系，有的话复制这些内容，下面的步骤要使用（备用数据B）。

##部署网站B
1. 在网站B的`wp-config.php`文件中，填写与网站A一致的数据库信息（备用数据A）

2. 与网站A一样，在任意位置添加如下内容：

```php
    define('COOKIE_DOMAIN', 'example.com');
    define('COOKIEPATH', '/');
```

3. 若存在备用数据B（各种define的'XX_KEY'），将这些段落完整的粘贴到网站B的`wp-config.php`中。

4. 将`$table_prefix  = 'wp_';`改为`$table_prefix  = 'xxx_';`（`xxx`是任意不同于`wp`的字符串）

5. 添加如下内容；
```php
define('CUSTOM_USER_TABLE', 'wp_users');
define('CUSTOM_USER_META_TABLE', 'wp_postmeta');
```

##关键步骤，修改Wordpress核心文件
我知道，这很恶心，但是我debug了一晚上后，已经实在没有精力去想一个优雅的解决方案了。请各位Wordpress达人提供一个hook吧……

按理说根据上述配置就可以实现用户在`www.example.com`登录后打开`blog.example.com`也自动处于登录状态了，但是现实是无情的。即使你发现两个站点下均存在`wordpress_logged_in_xxxx`的cookie，但是Wordpress就是不能实现这个cookie，调用`is_user_logged_in()`也是返回false。

经过我各种跟踪调试，最终定位到内核文件`wp-includes/default-constants.php`中的一个常量`COOKIEHASH `是罪魁祸首。

源代码如下：

```php
define( 'COOKIEHASH', md5( $siteurl ) );
```

**先提供解决方案，再讲原理**

将 `$siteurl` 改成你的顶级域名字符串，即

```php
define( 'COOKIEHASH', md5( 'example.com' ) );
```

这下整个世界都清爽了，cookie们终于愉快的在同一个域下同步了！

##原理解释

为什么一个简单的常量会导致cookie同步失败呢？我们来逐层的恢复这个递归。

>下面的这些函数并不是定义在同一个文件中，这里为了逻辑清晰将它们列在一起。

```php
define( 'COOKIEHASH', md5( $siteurl ) );
...
define('LOGGED_IN_COOKIE', 'wordpress_logged_in_' . COOKIEHASH);
...
wp_parse_auth_cookie($cookie='', $scheme='');
//该函数返回根据 $scheme 读取 $_COOKIE 中的内容并返回结果，
//这里的 $scheme 是常量 "logged_in"，函数内部使用 switch
//将 "logged_in" map 到了 LOGGED_IN_COOKIE 这个常量
...
wp_validate_auth_cookie($cookie, $scheme)
//根据上面的函数返回的结果，进一步判断cookie是否有效，并返回 user_id
...
add_filter( 'determine_current_user', 'wp_validate_auth_cookie',1);
//定义了一个filter，当apply的时候执行上述函数
...
get_currentuserinfo()
//这是很常用的一个内部函数，在这个函数里apply了上述filter
...
wp_get_current_user、is_user_logged_in等
//这些函数都调用了上述函数

```
分析到这里整个Wordpress的cookie解析流程也清晰了，问题很好定位，就是因为`网站A`和`网站B`的`$siteurl`不同，导致常量定义不同，导致读不到cookie，最终导致了cookie不能跨子域共用。

以上。