---
title: 在Mac OS X下使用终端连接OpenShift的SSH并部署应用
tags: 

  - openshift
  - paas
permalink: ssh-openshift-using-mac-os-x
id: 130
updated: '2013-01-20 17:52:22'
date: 2013-01-20 17:52:22
---

<p>今天折腾了一整天 OpenShift，早就听说 OpenShift 的大名，却一直无缘一试。从今天早上 10 点到现在（下午5点30分），终于摸透了 Mac 下使用 Terminal（终端）SSH 登陆 OpenShift 的方法，记下来与大家分享。</p>
<h2>什么是 OpenShift</h2>
<p>由大名鼎鼎的 RedHat 推出的一款 PaaS 服务，有免费的 Plan，支持创建 3 个应用，支持 PHP、Pyhton、Ruby、NodeJS 等语言。</p>
<h2>使用 SSH 连接 OpenShift</h2>
<p>阅读下面的文字假设你已有以下准备：</p>
<ul>
<li>已经注册 OpenShift 账号，并创建了一个 Gear（即一个应用）</li>
<li>使用 Mac OS X 10.6 及更高版本的系统</li>
<li>了解 vim 的基本使用方法</li>
<li>已经通过终端安装了 rhc（OpenShift 的部署工具，安装指令 gem install rhc）</li>
</ul>
<p>首先，先使用 rhc 检测一下当前的状态，打开终端，输入</p>
<blockquote>
<p>rhc-chk</p>
</blockquote>
<p>需要先输入你的 OpenShift 对应的密码，回车。</p>
<p>一般会遇到如下的结果:</p>
<blockquote>
<p>Password: *********</p>
<p>Analyzing system</p>
<p>....F.F</p>
<p>=================================================</p>
<p>|| &nbsp;Your system did not pass all of the tests &nbsp;||</p>
<p>=================================================</p>
<p><strong><span style="color: #808000;">1) Your public key is not loaded into a running ssh-agent: /var/root/.ssh/id_rsa.pub</span></strong></p>
<p><strong><span style="color: #808000;"><span style="white-space: pre;"> </span>If this is your only error, your connection may still work, depending on your SSH configuration.</span></strong></p>
<p><span style="color: #800080;">2) Cannot SSH into your app: xxxxx-ooooo.rhcloud.com.</span></p>
</blockquote>
<p>需要明确的是，如果你运行 rhc-chk 一切正常，那么就无需继续看下去了，直接使用原本的 git / ssh 的操作指令进行管理和部署即可。</p>
<p>下面的内容针对解决上述两个问题。</p>
<h2><strong>问题 1：Your public key is not loaded into a running ssh-agent</strong></h2>
<p><strong>首先我们需要创建 SSH 需要用到的公钥和私钥</strong></p>
<p>如果你已经创建，可以使用如下命令查看：</p>
<blockquote>
<p>ls ~/.ssh/</p>
</blockquote>
<p>若结果找两个形如 xx_rsa 和 xx_rsa.pub 两个文件则不需要执行如下的生成步骤，否则继续</p>
<p>输入命令</p>
<blockquote>
<p>ssh-keygen</p>
</blockquote>
<p>一路回车，最后会提示成功创建</p>
<p><strong>下面需要为秘钥和目录设置正确的权限</strong></p>
<p>依次输入命令：</p>
<blockquote>
<p>chmod 700 ~/.ssh</p>
<p>chmod 600 ~/.ssh/id_rsa*</p>
</blockquote>
<p>这样就设置了正确的权限。</p>
<p><strong>将公钥内容提交到 OpenShift 控制面板</strong></p>
<p>输入如下命令</p>
<blockquote>
<p>vim ~/.ssh/id_rsa.pub</p>
</blockquote>
<p>会打开刚才生成的秘钥中公钥的内容，全选复制。打开 OpenShift 控制面板</p>
<blockquote>
<p><a href="https://openshift.redhat.com/app/account" target="_blank">https://openshift.redhat.com/app/account</a></p>
</blockquote>
<p>将全部内容粘贴到右下角【Public Keys】的那个文本框里。若已经提交了公钥，则建议删除现有的公钥再提交新的公钥。</p>
<p><strong>接下来添加 ssh key 到 ssh-agent 中</strong></p>
<p>输入如下命令</p>
<blockquote>
<p>ssh-add ~/.ssh/id_rsa</p>
</blockquote>
<p>控制台回返回如下信息：</p>
<blockquote>
<p>Enter passphrase for /var/root/.ssh/id_rsa:&nbsp;</p>
</blockquote>
<p>之前我们在创建秘钥时并没有设置 passphrase，所以直接按回车即可。系统返回：</p>
<blockquote>
<p>Identity added: /var/root/.ssh/id_rsa (/var/root/.ssh/id_rsa)</p>
</blockquote>
<p><strong>一劳永逸，将 key 添加到 config 中</strong></p>
<p>打开控制台，输入如下指令</p>
<blockquote>
<p>vim ~/.ssh/config</p>
</blockquote>
<p>在打开的新界面立输入以下内容（提示，按下 i 键进入编辑模式，按下 ESC 键退出编辑模式，按下 :wq 保存退出&hellip;&hellip; 说过了要有 vim 基础的哈）</p>
<p><span style="color: #000000;"> </span></p>
<blockquote>
<p>Host *.rhcloud.com</p>
<p>IdentityFile ~/.ssh/id_rsa</p>
<p>VerifyHostKeyDNS yes</p>
<p>StrictHostKeyChecking no</p>
</blockquote>
<p>保存后整个为 ssh 添加秘钥的过程就就解决了。现在再运行 rhc-chk 命令肯定不会再出现&nbsp;<span style="color: #000000;">Your public key is not loaded into a running ssh-agent 这样的错误了。</span></p>
<h2><span style="color: #000000;">问题2：</span><span style="color: #000000;">Cannot SSH into your app: xxxxx-ooooo.rhcloud.com.</span></h2>
<p><span style="color: #000000;">其实这个问题很简单，你的这个应用的 IP 地址 SSH 被封了，虽然能 ping 通，但是 SSH 连不上。</span></p>
<p>解决方法：删除这个应用，重新创建。</p>
<p>&nbsp;</p>
<p>&nbsp;</p>