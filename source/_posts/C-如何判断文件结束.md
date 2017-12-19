---
title: 'C#如何判断文件结束'
permalink: cru-he-pan-duan-wen-jian-jie-shu
id: 17
updated: '2011-10-26 03:40:50'
date: 2011-10-26 03:40:50
tags:
---

<p>若使用StreamReader类：<br /> Read方法，到达尾端没有字符可以读到时，返回0<br /> Peek方法，如果没有字符可以读取，返回-1<br /> ReadLine方法,到达尾端时，返回null<br /> ReadBlock方法，到达尾端，没有字符可以再读时，返回0<br /> ReadToEnd方法，一次读完<br /> [cc lang="c#"]<br /> StreamReader r = new StreamReader("test.txt");<br /> while(r.ReadLine!=null)<br /> {<br /> //do sth here<br /> }<br /> [/cc]</p>