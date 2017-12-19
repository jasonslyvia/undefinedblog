---
title: 如何在多个WordPress博客间共享用户信息
permalink: ru-he-zai-duo-ge-wordpressbo-ke-jian-gong-xiang-yong-hu-xin-xi
id: 7
updated: '2014-08-04 14:12:20'
date: 2011-10-03 09:01:54
tags:
---

> 请查看最新版本的[多个Wordpress实例共享用户信息](http://undefinedblog.com/wordpress-multiple-site-same-user/)

各位站长可能出于很多原因，都会建立一个或者多个WordPress站点，大部分是一个www主域名，和几个二级域名。

今天，就结合我一个晚上的实战经验，谈谈如何在同一个数据库下安装多个WordPress博客，并在这些Wordpress博客间共享用户信息和Cookies信息（在一个Wordpress博客登录后转到另一个Wordpress博客自动处于登录状态）。

##建立共享数据库的博客

按照正常步骤安装第一个博客，解析到主域名，放在服务器根目录

在安装子博客时，打开 wp-config.php，将其中

```php
$table_prefix = 'wp_';
```
改为

```php
$table_prefix = 'wp1_';
```
注：这里的“wp1_”随意更改，只要和第一个的设置不同即可
另：两个博客的WordPress版本号需一致

还是在子博客的 wp-config.php里，找到如下行

```php
/* 好了！请不要再继续编辑。请保存本文件。使用愉快！ */
```
在该行之上添加如下两行代码：

```php
define('CUSTOM_USER_TABLE', 'wp_users');
define('CUSTOM_USER_META_TABLE', 'wp_usermeta');
```

以上两行代码表明你的子博客将和你的主博客共享同样的用户数据，除此之外子博客的所有数据都是独立的（和主博客使用同一个数据库，不同的表）

安装子博客（根目录下子目录），进行调试，完成

##实现多个WordPress博客Cookies共享，一次登录任意切换

1. 首先确认你按照上面的步骤成功建立了共享数据库的两个博客
2. 下载[Root Cookie 插件](http://wordpress.org/extend/plugins/root-cookie/)，分别上传到主博客和子博客的 `/wp-content/plugins/`文件夹中，并都激活
3. 打开你主博客的设置表，在浏览器中输入如下地址并按回车
`htto://www.ppios.com/wp-admin/option.php` 请将`ppios.com`替换为你的域名
4. 找到 `AUTH_SALT` 和 `LOGGED_IN_SALT`的值，记录下来（它们的值应该是一长串毫无意义的字符串）
5. 打开子博客的 wp-config.php，找到对应的语句，进行替换：
```php
define('AUTH_SALT', 'some value with numbers and letters ');
define('LOGGED_IN_SALT', 'some value with numbers and letters ');
```

6. 打开主博客的 wp-config.php，从[WordPress Api](https://api.wordpress.org/secret-key/1.1/) 随机生成4个键的值，将其替换到主博客wp-config.php的对应项中
7. 打开子博客的 wp-config.php，同样更新以上4项
8. 最后一步，在子博客的 wp-config.php 中任意处添加如下内容：
```php
$baseurl = 'http://www.yourmainurl.com';//自行替换
$cookiehash = md5($baseurl);
define('COOKIEHASH', $cookiehash);
define ('AUTH_COOKIE', 'wordpress_'.COOKIEHASH);
define ('SECURE_AUTH_COOKIE', 'wordpress_sec_'.COOKIEHASH);
define ('LOGGED_IN_COOKIE','wordpress_logged_in_'.COOKIEHASH);
define ('TEST_COOKIE', 'wordpress_test_cookie');
```

9. 清空所有浏览器缓存和Cookies，登入主站，再打开子站，看看是不是自动登入了？

以上全部是个人实际操作的经验总结，欢迎各位交流！
