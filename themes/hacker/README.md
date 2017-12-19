# Hacker | [中文版文档](/README_zh-CN.md)
[![Open Source Love](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://github.com/ellerbrock/open-source-badge/)  [![GPL Licence](https://badges.frapsoft.com/os/gpl/gpl.svg?v=103)](https://opensource.org/licenses/GPL-2.0)  


__Hacker is a simple blog theme focused on writing. In such a trend of complex typography, choose the return to origins, focusing on writing this matter.__  

The beginning is [moyo](http://liuxinyu.me/) created a theme of Wordpress , by DaraW transplanted to Hexo.

## Demo
You can refer to my blog: [DaraW](http://blog.daraw.cn/)  

![](https://ooo.0o0.ooo/2016/08/04/57a306f56bee2.png
)

## Installation
Firstly get the theme files, `git clone` or `download zip` both are ok.  

Create a folder named `Hacker` in the folder `themes`, and copy all the theme files to the folder `Hacker`.  

Then apply the theme in the hexo global configuration file `_config.yml`:

```yaml
theme: Hacker
```
Now all are in order, just enjoy~

__Notice: After every update, you'd better run command `hexo clean` to clean cache files before Hexo generating, in case of some problems cache files bring.__


## Configure
### Enable comments and Google Analytics
In the theme configuration file `_config.yml`:

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


`duoshuo`: `boolean`, use duoshuo or not;  
`duoshuo_name`: `string`, your duoshup ID, please don't use other people's IDs。

`disqus`: `boolean`, use disqus or not;  
`disqus_shortname`: your disqus site shortname.

`googleTrackId`: your Google Analytics ID, Hacker will not use Google Analytics if it's empty.

### Enable Categories and Tags pages
Categories Page: run `hexo new page categories`，then modify the generated file `source/categories/index.md`：
``` markdown
title: categories
date: 2017-01-30 19:16:17
layout: "categories"
---  
```
If you need to close comments of this page , you can add a line `comments: false`; `title` corresponds to the title of the page.

Tags Page: run `hexo new page tags`，then modify the generated file `source/tags/index.md`：
``` markdown
title: tags
date: 2017-01-30 19:16:17
layout: "tags"
---  
```
Configuration is the same as Categories Page.  

Add links to the menu: Edit the `_config.yml` file of the theme, add `Categories: /categories` and `Tags: /tags` in `menu` like this：
``` yml
menu:
  Home: /
  Archives: /archives
  Categories: /categories
  Tags: /tags
```

## Update
### v1.1.0
* Add support for closing article comments ([issue#14](https://github.com/CodeDaraW/Hacker/issues/14))
* Add support for categories and tags ([issue#7](https://github.com/CodeDaraW/Hacker/issues/7))

### v1.0.1
* fix incorrect comment link on the home page

### v1.0
* fix bug caused by subdirectory ([issue#10](https://github.com/CodeDaraW/Hacker/issues/10))
* fix display of `code` tag


### v0.3.
* Refactor ejs template files
* Replace css with stylus
* Add English Version README


### v0.2
* Remove some useless css
* Fix bug that icon still shows when there are no categories or tags
* Rewrite the archive index page
* Change the display of code block


## License
GNU GPL(General Public License) v2.0
