---
title: iPhone开发 如何从App内部加载html文件
permalink: iphonekai-fa-ru-he-cong-appnei-bu-jia-zai-htmlwen-jian
id: 10
updated: '2011-10-18 18:55:32'
date: 2011-10-18 18:55:32
tags:
---

<p>[cc lang="c++"]<br />
- (void)loadDocument:(NSString*)docName {</p>
<p>    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);<br />
    //NSString *documentsDirectory = [paths objectAtIndex:0];    该方法为从Documnent中加载<br />
    //NSString *path = [documentsDirectory stringByAppendingPathComponent:docName];<br />
    NSString *mainBundleDirectory = [[NSBundle mainBundle] bundlePath];//该方法为把html文件添加到App内部<br />
    NSString *path = [mainBundleDirectory  stringByAppendingPathComponent:docName];<br />
    NSURL *url = [NSURL fileURLWithPath:path];<br />
    NSURLRequest *request = [NSURLRequest requestWithURL:url];</p>
<p>    self.myWebView.scalesPageToFit = YES;</p>
<p>    [self.myWebView loadRequest:request];<br />
}<br />
[/cc]</p>
