---
title: 如何让Grunt在Wordpress主题开发中发挥作用
tags: 

  - grunt
  - wordpress
permalink: grunt-wordpress
id: 155
updated: '2014-03-12 08:28:27'
date: 2014-01-23 13:21:53
---

<a href="https://github.com/jasonslyvia/grunt-wordpress">English Version</a>

Grunt (http://gruntjs.cn/) 是目前前端开发领域非常热门的自动部署工具，它本身被定义为『JavaScript任务运行器』。借助强大的 npm，你可以使用 Grunt 自动完成 JS文件及CSS文件自动压缩及合并、静态资源文件自动添加MD5后缀甚至能自动替换HTML代码中引用的JS、CSS经压缩加MD5后缀之后的文件名，真正做到一键部署。

但是 Grunt 社区中的大部分插件都是以新型 webapp 为基础开发的，他们大多基于 Angular、Backbone 等前端 MVC 框架，文件结构与资源引用方式与 Wordpress 主题有着很大的差别，因此直接使用已有的 Grunt 插件实现 Wordpress 主题的自动部署并不容易。

本文将着重讨论如何让 Grunt 在 Wordpress 主题开发及部署中发挥作用，真正利用好这样一款前端开发神器。

<strong>阅读本文需要有一定的 Grunt 使用基础</strong>
<h2>最终目的</h2>
一个常规的 Wordpress 主题目录结构应该如下图所示，在 header.php 中有若干 css 文件的引用，在 footer.php 中应该有若干 js 文件的引用。
<pre><code>|
+- your_theme_name
|   +- header.php
|   +- footer.php
|   +- style.css
|   +- js
|      +- all your js files
|   +- css
|      +- all your css files</code></pre>
通过本文介绍的部署方法，最终实现如下效果：
<pre><code>|
+- your_theme_name
|   +- header.php
|   +- footer.php
|   +- style.min.a2312abe.css
|   +- js-dist
|      +- all your js files that are minified and revved
|   +- css-dist
|      +- all your css files that are minified and revved
</code></pre>
即所有的静态资源都已经被压缩、合并、添加MD5戳，同时在 header.php 及 footer.php中，原本引用的 js 和 css 文件名也被替换为了压缩合并后的文件名，真正实现一键部署。
<h2>Wordpress 主题使用 Grunt 的问题</h2>
在进入主题之前，先说明一下为什么 Wordpress 主题开发无法直接使用 Grunt 的某些功能，以便你能更好的了解这篇文章想要解决的问题及思路。
<h3>资源引用方式不同</h3>
在新型的 webapp 中，一般直接使用相对路径引用静态资源即可，例如：
<pre class="lang:xhtml decode:true">&lt;link rel="stylesheet" type="text/css" href="css/main.css" /&gt;</pre>
但是在 Wordpress 中，我们一般需要使用绝对路径：
<pre class="lang:php decode:true">&lt;?php define("URL", get_template_directory_uri()); ?&gt;
...
...
&lt;link rel="stylesheet" type="text/css" href="&lt;?php echo URL;?&gt;/css/main.css" /&gt;</pre>
这样就直接导致了在使用如 usemin 等 Grunt 插件时无法正确替换静态资源的问题。
<h3>特殊的 style.css</h3>
开发过 Wordpress 主题的同学都知道，Wordpress 是通过每个主题目录下的 style.css 文件来识别这个主题的。因此，不同于普通的 webapp 所有 css 可以放在同一个文件夹下，Wordpress 主题里的 style.css 必须放在主题的根目录下，这样为 Grunt 的任务配置带来了麻烦。

同时 style.css 还需要在开发保存一个特殊的注释块，才能被 Wordpress 正确识别。
<h2>部署思路</h2>
考虑 Grunt 的特性以及 Wordpress 主题的特殊性，我们需要对现有的 Wordpress 主题源代码文件进行一些简单的修改：
<ul>
	<li>在 header.php 及 footer.php 中修改资源的引用方式，改为相对路径引用，即去掉原有的 &lt;?php echo URL;?&gt; 部分</li>
	<li>将 header.php 修改为 header-src.php，同理 footer.php 修改为 footer-src.php，这样做的目的是每次使用 Grunt 部署后自动在当前目录生成可用的 footer.php 和 header.php</li>
	<li>同上，将 style.css 修改为 style-src.php</li>
	<li>如果你在 header.php 及 footer.php 之外的其它 php 文件中也引用了需要压缩等操作的静态资源，使用同样的方法命名他们</li>
</ul>
同时为了避免源文件与处理后的文件混杂在一起，我们需要对文件结构进行一定的调整。主要是将压缩后的文件放在新的文件夹里，如原本在 ./js 文件夹下的 js 文件将被放在 ./js-dist 文件夹下，当然文件夹名可以自定。
<h2>配置 Grunt</h2>
在修改好 Wordpress 源文件后，我们开始配置 Grunt。下面是我自己使用的一份 Gruntfile 配置文件，供大家参考。再次说明，如果你不理解下面的配置内容，请参考 Grunt 的<a href="http://gruntjs.cn/getting-started/" target="_blank">新手上路</a>页面。

说明：当你处于开发环境时，运行 <span class="lang:xhtml decode:true  crayon-inline ">grunt build </span> 命令；当你需要发布时，运行 <span class="lang:xhtml decode:true  crayon-inline ">grunt dist</span>  命令。目前存在一个问题，即每次修改后都要运行 <span class="lang:xhtml decode:true  crayon-inline ">grunt build</span>  命令才能看到效果，你可以结合更多的 grunt 插件监控文件状态更改，然后自动运行 <span class="lang:xhtml decode:true  crayon-inline ">grunt build</span>  命令。
<pre class="lang:js decode:true">module.exports = function(grunt) {

  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),

    concat: {
      dist: {
        src: ['js/common.js', 'js/grid.js'],
        dest: 'js/main.js'
      }
    },

    uglify: {
      dist: {
        files: [{
          expand: true,
          cwd: 'js',
          src: ['*.js'],
          dest: '.tmp/js',
          ext: '.min.js'
        }]
      }
    },

    cssmin: {
      //处理普通css文件
      dist: {
        files: [{
          expand: true,
          cwd: 'css',
          src: ['*.css'],
          dest: '.tmp/css',
          ext: '.min.css'
        }]
      },

      //处理style.css
      specialStyle: {
        options: {
          banner: '/*\n'+
                      'Theme Name: Canon\n'+
                      'Theme URI: http://xiaoshelang.ppios.com/\n'+
                      'Description: 小摄郎 Wordpress 主题\n'+
                      'Author: YangSen\n'+
                      'Author URI: http://undefinedblog.com/\n'+
                      'Version: 1.0\n'+
                      '私人主题，非开源，保留所有权利。\n'+
                      'Private theme, not open-sourced, all rights reserved.\n'+
                  '*/\n'
        },
        files: {
          ".tmp/css/style.min.css": "style-src.css"
        }
      }
    },

    filerev: {
        js: {
          src: '.tmp/js/*.js',
          dest: 'js-dist'
        },
        css: {
          src: ['.tmp/css/*.css'],
          dest: 'css-dist'
        }
    },

    copy: {
      build: {
        files: [
        {
          src: "style-src.css",
          dest: ".tmp/css/style.css"
        },
        {
          expand: true,
          src: ["css/*.css"],
          dest: ".tmp"
        },
        {
          expand: true,
          src: ["js/*.js"],
          dest: ".tmp"
        }]
      }
    },

    wpreplace: {
      options: {
        templatePath: '/wp-content/themes/canon/',
        jsPath: 'js-dist',
        cssPath: 'css-dist',
        concat: []
      },

      dist: {
        src: ['header-src.php', 'footer-src.php'],
        options: {
          concat: [{
            src: ['common.js', 'grid.js'],
            dest: ['main.js']
          }]
        }
      },

      build: {
        src: ['header-src.php', 'footer-src.php']
      }
    },

    clean: {
      beforeBuild: ['js-dist',
                    'css-dist',
                    'js/main.js',
                    "style*.css",
                    "!style-src.css"],
      afterBuild: ['.tmp']
    }

  });

  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-concat');
  grunt.loadNpmTasks('grunt-contrib-cssmin');
  grunt.loadNpmTasks('grunt-contrib-clean');
  grunt.loadNpmTasks('grunt-contrib-copy');
  grunt.loadNpmTasks('grunt-filerev');
  grunt.loadNpmTasks('grunt-wp-replace');

  grunt.registerTask('dist', ['clean:beforeBuild',
                                 'concat',
                                 'uglify',
                                 'cssmin',
                                 'filerev',
                                 'clean:afterBuild',
                                 'wpreplace:dist'
                                 ]);

  grunt.registerTask('build', ['clean:beforeBuild',
                                'copy:build',
                                'filerev',
                                'wpreplace:build',
                                'clean:afterBuild']);

};</pre>
其中 <a href="https://github.com/jasonslyvia/grunt-wp-replace" target="_blank">grunt-wp-replace</a> 是我自己写的一个 Grunt 插件，核心功能就是将 php 文件中的静态资源引用替换为新生成的文件名。

对于 wpreplace 这个 task  的配置简单的说明一下：
<pre class="lang:js decode:true">/*
 ...
*/
  wpreplace: {
    options: {
      //主题的URI
      templatePath: '/wp-content/themes/your-theme-name/',
      //存放js静态文件的文件夹名
      jsPath: 'js-dist',
      //存放css
      cssPath: 'css-dist',
      //如果你使用了concat任务合并了js或css文件，按照以下格式指明他们，wpreplace会自动完成替换工作
      concat: [{
        src: ['a.js', 'b.js'],
        dest: ['concated.js']
      }]
    },
    your_target: {
      //需要处理的文件，如果还有别的文件可以自行添加，但是需要统一加上 -src 的后缀
      src: ['header-src.php', 'footer-src.php']
    }
/*
...
*/</pre>
<h2>小结</h2>
至此，整个 Grunt 的配置过程就结束了。当然，我只是按照自己的业务需要使用了部分的 Grunt 任务，你还可以添加你自己需要的任务，来更好的部署自己的 Wordpress 主题。

这篇文章是我对利用 Grunt 部署 Wordpress 主题的一些愚见，希望能够抛砖引玉，找到更好的方法。