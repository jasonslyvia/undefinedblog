---
title: 使用群晖nas搭建私有git服务器
tags: 

  - git
  - nas
permalink: deploy-private-git-server-on-synology-nas
id: 152
updated: '2013-11-11 06:22:15'
date: 2013-11-11 06:22:15
---

在我不厌其烦的跟身边朋友解释什么是 nas（私人版的 dropbox，什么？啥是 dropbox？好吧，私人版的百度云！哦……）后，我愈加喜欢上了这款花了我不少血汗钱的小家伙。购买 nas 最大的优势在于有专业团队为你打磨整个使用流程，虽然我很欣赏自己动手组装 nas 的行为，但是本着术业有专攻（能偷懒就偷懒）的原则，我还是选择了群晖的 nas 解决方案以及附带的 DSM 系统。

群晖 nas 的系统 DSM 是基于 Linux 的，因此你也可以把 nas 看作一台小型的 Linux 服务器。DSM 提供了很多实用的套件供你开发 nas 的潜力，包括 PHP、Python、MySQL、Wordpress 等，同时也支持 Git 服务器。

今天这篇文章就讲讲如何使用群晖 nas 搭建私有 git 服务器。
<h2>为什么要搭建私有 git 服务器</h2>
在开始正题之前，再废话两句。用 git 的人都知道 GitHub，但是 GitHub 托管私有项目是要付费的。因此如果是小型项目，大家都在同一个局域网里开发（nas 支持 DDNS 功能，所以严格意义上来说是支持互联网接入的）的话，部署在 nas 上的 git 服务器就很方便了。一来是无需缴纳额外的代码托管费用，二来是接入速度神速，且不用担心偶尔被墙。

即使是个人项目，我也觉得有必要使用 git 服务器，本地一份，remote一份，多一份备份多一个安心嘛！
<h2>nas 搭建 git 服务器</h2>
1、进入 DSM 系统的『套件中心』，安装 Git Server

2、创建一名用户，名为『git』（名称随意，但是叫 git 比较好，该用户仅有 git shell 的执行权限，除此之外无任何权限）

3、进入 Git Server，勾选之前创建的『git』，使其『允许访问』

<img class="aligncenter size-full wp-image-207145" alt="synology git" src="http://undefinedblog.com/wp-content/uploads/2013/11/屏幕快照-2013-11-10-下午10.18.42.png" width="599" height="350" />

4、在『控制面板』- 『终端机』中开启 SSH 登录功能

5、ssh 登录到 nas，使用 root 用户
<pre class="lang:sh decode:true">ssh root@你的nas地址</pre>
6、选择一个文件夹作为你的 git 代码存储地址，我设置的是
<blockquote>/volume1/git_repos/</blockquote>
/volume1 指的是 nas 中第一块硬盘，git_repos 文件夹本身不存在，需要自行创建
<pre class="lang:sh decode:true">cd /volume1
mkdir git_repos</pre>
7、在 git_repos 文件夹中创建一个新的文件夹，作为某一个项目的 base，我们叫他 test
<pre class="lang:sh decode:true">mkdir test
cd test</pre>
8、初始化一个空的 git 项目，此时有两种方法可以选择

<strong>若你已经在本地初始化好了项目</strong>

需要 push 到 nas 上的 git 服务器，使用如下命令：
<pre class="lang:sh decode:true">git init --bare .git</pre>
然后在本地使用如下命令加本地代码 push 到 git 服务器
<pre class="lang:sh decode:true">git remote add nas ssh://git@你的nas地址/volume1/git_repos/test/.git
git push nas master</pre>
<strong>若你在本地还没有项目</strong>

可以使用如下命令：
<pre class="lang:sh decode:true">git init</pre>
然后在本地使用
<pre class="lang:sh decode:true">git clone ssh://git@你的nas地址/volume1/git_repos/test/.git</pre>
完成项目的初始化

9、需要注意一点关于权限的问题，使用 root 用户进行上述操作后 git 用户可能会提示无权进行文件修改。解决方法：
<pre class="lang:sh decode:true">cd /volume1
chown -R git:users git_repos</pre>
将所有 git_repos 极其子目录的拥有者改为用户 git 后，就不存在权限问题了。

群晖的 nas 还有很多有意思的功能，基于 Linux 的系统意味着无穷的可玩性，慢慢发掘吧！