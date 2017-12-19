---
title: 如何在低版本的Xcode中使用高版本的SDK
tags: 

  - xcode
permalink: use-higher-version-sdks-in-lower-version-xcode
id: 73
updated: '2012-06-26 22:25:23'
date: 2012-06-26 22:25:23
---

<p>这几天需要在 iOS 上做一个关于 OpenGL ES 的课设，想要真机调试时才发现我的 4.2 版本的 Xcode 最高只支持 iOS 5.0 的 SDK，而我手上的两部测试设备都已经升级到了 iOS 5.1.1。虽然备份的有 shsh 可以降回 iOS 5.0，但是总觉得太麻烦了，于是打算升级到 Xcode 4.3.2。（Xcode 4.3.1 及以上版本支持 iOS 5.1.1 调试，仅限 Lion 系统）</p>
<p>然而下载好 Xcode 4.3.2 的 DMG 文件后又悲催的发现，我的 OS X 是 10.7，而 Xcode 4.3.2 需要 10.7.3 及更高版本的 OS X，莫非还要升级系统，太麻烦了！</p>
<p>搜索了一下发现有人遇到了同样的问题，而解决方法如下：</p>
<p>0、测试环境 Mac OS X 10.7 + Xcode 4.2，欲实现效果：<strong>在 Xcode 4.2 下实现 5.1.1 真机调试</strong></p>
<p>1、下载 Xcode 4.3.1 及更高版本的 DMG 文件</p>
<p>2、右键单击下载好的 DMG 文件，选择【浏览】（记不太清是什么选项了，总之是可以浏览 DMG 内部文件的一个选项）</p>
<p>3、在新的文件浏览窗口里，定位到</p>
<blockquote>
<p>/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/5.1 (9B176)</p>
</blockquote>
<p>4、将<strong>&nbsp;5.1(9B176)&nbsp;</strong>这个文件夹复制到你的 Xcode 对应目录中</p>
<blockquote>
<p>/Developer/Platforms/iPhoneOS.platform/DeviceSupport</p>
</blockquote>
<p>5、同样把 iOS 5.1 （或 5.1.1 ）的 SDK 复制到 Xcode 的对应目录</p>
<blockquote>
<p>/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS5.1.sdk</p>
</blockquote>
<p>6、如果打开了 Xcode 就彻底关闭 Xcode 后重启，然后选择你的 5.1.1 的机器，直接进行编译即可</p>
<p>已知的问题是可能在编译时 Xcode 提示无法启动线程，但是应用已经传到了设备内，再手动从设备端打开即可，这样的坏处时在真机调试时无法实时获得调试信息。因此本文只能是权宜之计，还是升级到苹果要求的版本再开发为好！</p>