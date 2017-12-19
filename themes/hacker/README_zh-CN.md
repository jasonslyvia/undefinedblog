# Hacker | [English Docs](/README.md)
[![Open Source Love](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://github.com/ellerbrock/open-source-badge/)  [![GPL Licence](https://badges.frapsoft.com/os/gpl/gpl.svg?v=103)](https://opensource.org/licenses/GPL-2.0)  


__Hacker是一款专注于写作的简洁博客主题。在如此讲究复杂排版的趋势下，选择回归本源，专注于写作这件事。__  

一开始是[moyo](http://liuxinyu.me/)为Wordpress所创作的一个主题，由DaraW移植到Hexo。

## Demo
参考我的博客：[DaraW](http://blog.daraw.cn/)  

![](https://ooo.0o0.ooo/2016/08/04/57a306f56bee2.png
)

## 安装
获得主题文件， `git clone`或者`download zip`均可；  

在`themes`文件夹中创建文件夹`Hacker`，将主题文件都复制粘贴至`Hacker`文件夹；  

然后在hexo全局配置文件`_config.yml`中应用主题：

```yaml
theme: Hacker
```
这样就安装好了，开始享受吧~

__注意：版本更新后建议在hexo生成前执行一次`hexo clean`，清除以前的缓存，避免带来的一些莫名其妙的问题。__

## 配置
### 启用评论和谷歌分析
在主题配置文件`_config.yml`中：

```yaml
# duoshuo comment
duoshuo: true
duoshuo_name:

# disqus comment
disqus: false
disqus_shortname:

# google analytics
googleTrackId:
```


`duoshuo`: `boolean`，是否开启多说评论；  
`duoshuo_name`: `string`，你的多说用户ID，请不要使用别人的ID。  

`disqus`: `boolean`，是否开启disqus评论；  
`disqus_shortname`: `string`，你的disqus site shortname。  

`googleTrackId`: `string`，为谷歌分析的个人ID，留空则为不使用谷歌分析。

### 启用分类和标签页面
分类功能：执行`hexo new page categories`，然后修改生成的`source/categories/index.md`：
``` markdown
title: categories
date: 2017-01-30 19:16:17
layout: "categories"
---  
```
如果你需要关闭该页的评论，可以添加一行`comments: false`；`title`对应的则是该页的标题。  

标签功能：同理，执行`hexo new page tags`，然后修改生成的`source/tags/index.md`：
``` markdown
title: tags
date: 2017-01-30 19:16:17
layout: "tags"
---  
```
配置同分类功能。

在菜单中添加链接：编辑主题的`_config.yml`，在`menu`中添加`Categories: /categories`和`Tags: /tags`，如下：
``` yml
menu:
  Home: /
  Archives: /archives
  Categories: /categories
  Tags: /tags
```

## 更新
### v1.1.0
* 增加对关闭文章评论的支持([issue#14](https://github.com/CodeDaraW/Hacker/issues/14))
* 增加对分类和标签的支持([issue#7](https://github.com/CodeDaraW/Hacker/issues/7))

### v1.0.1
* 修复了主页上错误的评论链接

### v1.0
* 修复从文件夹导致的bug([issue#10](https://github.com/CodeDaraW/Hacker/issues/10))
* 修复`code`标签的显示效果


### v0.3
* 重构ejs模板
* 改用stylus
* 添加英文文档


### v0.2
* 去除部分无用css和重复css
* 修复无分类/标签依然出现icon的bug
* 重写归档列表页面
* 修改代码块样式


## 协议
GNU GPL(General Public License) v2.0
