---
title: '在Windows环境下使用C#快速进行GTK程序开发'
tags: 

  - c#
  - gtk
permalink: gtk-sharp
id: 141
updated: '2013-05-24 01:26:00'
date: 2013-05-24 01:26:00
---

其实这是一个挺操蛋的需求，如果我想用 C# 开发，界面干嘛非要用 GTK？如果要用 GTK，为什么要用 C# 这种跨平台兼容性不如 C++ 的语言？所以在开始正文前，先简单的介绍一下写这篇文章的原因。

本学期，我有幸上了一节名为——呃，不好意思记不起来了——的课，主要是一位德高望重的教授给我们照着念他自己写的一本教材，教材内容涉及了 Linux 基础知识、MySQL、GCC、GTK 等若干内容。期末的作业就是按照学号分配上述内容，我分到了 MySQL。按理说写个 MySQL 应用是相当简单的事，一个下午就能写个简易的 CMS 出来。

然而教授要求了，<strong>如果你的作品需要带界面，那就必须要用 GTK。</strong>于是我就不得不去写 GTK 程序，比起在 Linux 下拿 C 写，我更倾向于用更熟悉的语言解决这个脑残的需求。

最后废话一行，本来想用 JavaScript 的 GNOME 绑定 <a href="https://live.gnome.org/Seed" target="_blank">Seed</a>，不过配置和编译的过程太复杂。既然找到了更方便的 GTK# 和 Windows 下的神级 IDE Xamarin Studio，没有理由不用简单的。好了，下面开始正文。
<h2>GTK# 及 MonoDevelop</h2>
<a href="http://www.mono-project.com/GtkSharp" target="_blank">GTK#</a>（GTK Sharp）顾名思义就是 C# 的 GNOME 的绑定，主要实现的是 GTK+ 的绑定。

<a href="http://monodevelop.com/" target="_blank">MonoDevelop</a> 是一款跨平台的 IDE，用于 C# 及 .NET 语言的编写及调试，用起来跟 Visual Studio 没太大差别。MonoDevelop 的程序名称其实叫做 Xamarin Studio，最大的特点就是内置了 GTK 的图形化编辑界面，你可以像编写 Windows Form 程序那样通过拖拖拽拽完成界面设计和事件绑定，跟 <a href="http://glade.gnome.org/" target="_blank">Glade</a> 类似却比 Glade 更强大。

这些东西，都是开源的，貌似都是 GPL 协议。
<h2>简单开始</h2>
首先下载 Xamarin Studio，<a href="http://monodevelop.com/Download" target="_blank">下载地址</a>

如果你的电脑里没有装<strong> .NET Framework 4.0</strong> 或更高版本，则需要先安装之。同时，安装完 .NET 后还需要安装 <strong>GTK# for .NET。</strong>

以上内容均可以在下载链接中找到，下载后依次安装即可。

第一次运行 Xamarin Studio 的界面如下图，依次点击「New」-「C#」-「GTK# 2.0工程」新建一个 GTK 项目。

[caption id="attachment_207076" align="aligncenter" width="570"]<a href="http://undefinedblog.com/wp-content/uploads/2013/05/gtk-sharp.jpg"><img class=" wp-image-207076  " alt="Xamarin Studio" src="http://undefinedblog.com/wp-content/uploads/2013/05/gtk-sharp.jpg" width="570" height="331" /></a> 第一次运行Xamarin Studio[/caption]

在左侧「用户界面」-「MainWindow」双击，调出用户界面的设计界面，同时在右侧可以切换工具栏，这里可以看到你熟悉的各种控件。

[caption id="attachment_207078" align="aligncenter" width="609"]<a href="http://undefinedblog.com/wp-content/uploads/2013/05/xamarin-studio.png"><img class="size-full wp-image-207078" alt="xamarin studio编辑用户界面" src="http://undefinedblog.com/wp-content/uploads/2013/05/xamarin-studio.png" width="609" height="385" /></a> Xamarin Studio编辑用户界面[/caption]

GTK 的界面设计稍微不同于 Windows Form 或 WPF，具体应该怎么设计这里不多赘述了。只说明一点：<strong>先拖一些容器进去，这些容器用来控制界面的版式和布局，再在容器内添加具体的空间，如按钮、文本框等。</strong>

如何给控件绑定事件？假设你已经往界面里拖入了一个按钮，单击这个按钮，然后找到右侧的「属性」边栏，切换到「信号」，找到「Button Signals」。

[caption id="attachment_207079" align="aligncenter" width="287"]<a href="http://undefinedblog.com/wp-content/uploads/2013/05/binding-event-for-gtk-sharp.png"><img class="size-full wp-image-207079" alt="为控件绑定事件" src="http://undefinedblog.com/wp-content/uploads/2013/05/binding-event-for-gtk-sharp.png" width="287" height="318" /></a> 为控件绑定事件[/caption]

其中 「Clicked」对应的就是按钮的点击事件，单击「Click here to add ...」激活输入框，输入为该按钮绑定的事件函数，例如 <strong>btn_clicked</strong>。绑定事件后你能看到该项对应显示效果被加粗了。

这个时候，点击下方的「Source」切换到代码界面，就能看到 IDE 已经自动为刚才绑定的事件生成了函数，在函数里实现相关功能即可。

[caption id="attachment_207080" align="aligncenter" width="604"]<a href="http://undefinedblog.com/wp-content/uploads/2013/05/gtk-button-clicked.png"><img class="size-full wp-image-207080" alt="Xamarin Studio 代码编辑界面" src="http://undefinedblog.com/wp-content/uploads/2013/05/gtk-button-clicked.png" width="604" height="506" /></a> Xamarin Studio 代码编辑界面[/caption]
<h2>一些有用的资源</h2>
<ul>
	<li>一个简单的视频教程，演示了怎么利用 Xamarin Studio 创建一个带界面的程序，需翻墙
<a href="http://monodevelop.com/Documentation/Creating_a_simple_user_interface_with_MonoDevelop">http://monodevelop.com/Documentation/Creating_a_simple_user_interface_with_MonoDevelop</a></li>
	<li>GTK# 在线文档
<a href="http://docs.go-mono.com/?link=N%3aGtk">http://docs.go-mono.com/?link=N%3aGtk</a></li>
</ul>