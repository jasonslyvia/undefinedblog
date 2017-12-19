---
title: 解决使用 ES6 写测试时 chai 的报错
permalink: chai-throw-error-when-use-babel-transform-es6-syntax
id: 171
updated: '2015-04-23 10:38:16'
date: 2015-04-23 10:30:56
tags:
---

最近基本所有的项目都开始使用 ES6 语法来写。一是因为很多 ES6 语法确实简洁很多（如箭头函数），二是因为向 ES5 甚至 ES3 兼容的 transpiler 都比较成熟，不用担心兼容性的问题。

但是最近在用 ES 6 写测试代码时，chai 会报如下错误：

```bash
TypeError: Unable to access callee of strict mode function
```

经过一番研究我发现如下几个点：

 1. chai 不支持 strict mode
 2. Babel 在将 ES6 module 转换为 ES5 时会自动添加 `"use strict";` 指令（这也是 ES 6 的规范）

因为我的测试代码也是用 ES 6 编写，Babel 在转换时给测试代码添加了严格模式的指令，导致 chai 的 expect 方法报错了。

解决方法比较简单：

 1. 在使用 Babel 转换 ES6 代码时可以传入参数 `{blacklist: strict}`，如果使用命令行，可以使用 `babel xxx.js --blacklist=strict` （在正常的业务代码中不建议使用该参数）
 2. 如果使用了 Webpack 或 Browserify，要在配置文件中忽略掉 `node_modules` 文件夹
 
 
最近工作繁忙，博客更新的太慢……