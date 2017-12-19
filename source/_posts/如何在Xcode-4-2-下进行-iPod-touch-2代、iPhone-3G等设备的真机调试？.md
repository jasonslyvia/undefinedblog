---
title: 如何在Xcode 4.2 下进行 iPod touch 2代、iPhone 3G等设备的真机调试？
tags: 

  - xcode
permalink: >-
  ru-he-zai-xcode-4-2-xia-jin-xing-ipod-touch-2dai-iphone-3gdeng-she-bei-de-zhen-ji-diao-shi
id: 50
updated: '2012-02-13 18:31:30'
date: 2012-02-13 18:31:30
---

<p>Xcode 4.2 默认的 SDK 设置是 iOS 5.0，如果想要在 Xcode 4.2下调试低版本的 iOS 设备或者低端的 iOS 设备如 iPhone 3G、iPod touch 2/3 等，则可能出现各种各样的问题。</p>
<p>问题的解决方法是：</p>
<p>1、打开 Xcode ，点击菜单[Xcode] - [Prefernces]，切换到[Downloads]选项卡，下载[iOS 4.0-4.1 Device Debugging Support]，或者 iOS 3.0版本，具体根据自己的需要进行</p>
<p>2、新建一个项目，点击左侧的项目名称，在右侧的栏目中，切换到左侧的[Targets]，然后切换到右侧的[Build Settings]</p>
<p>展开[Architectures]属性一栏，将属性从[Standard (armv7)]改为[Others]，在展开的栏目中添加[armv6]和[armv7]两厢</p>
<p>3、同样设置[Valid Architecture]为 [armv6]和[armv7]</p>
<p>4、将[iOS Deployment Target]设置为你想要部署的<span style="color: #ff0000;">最低iOS版本</span></p>
<p><img class="aligncenter" src="http://ww2.sinaimg.cn/mw600/868702cfjw1dq0tobb0jgj.jpg" alt="" width="591" height="680" /></p>
<p>5、打开项目的 info.plist文件，一般在[Resources]文件夹中，完整删除[Required device capabilities]这一项即可</p>
<p><img class="aligncenter" src="http://ww2.sinaimg.cn/mw600/868702cfjw1dq0toc9pd3j.jpg" alt="" width="600" height="282" /></p>