---
title: 一键快速切换/还原DNS设置脚本
tags: 

  - dns
  - vbs
permalink: vbs-script-for-changing-dns
id: 140
updated: '2014-02-09 05:21:40'
date: 2013-05-17 02:54:10
---

学校的 DNS 越来越不给力了，打开一些网页时经常出现连接超时的现象。虽然不能确定 DNS 服务器到底出了啥问题，但是可以确定换用 114 或者 Google 的 DNS 肯定能够更快的请求到资源。

问题来了，尽管设置完新的 DNS 后可以用的很爽，但是某些时候需要用到内网资源，比如网关登录系统时，更换 DNS 将导致这些请求无法被正确转发。这时，不得不再手动的把 DNS 切换回默认设置。

<img class="aligncenter size-full wp-image-207067" title="这样来来回回的设置 DNS 真的很烦人" alt="Windows 7设置DNS" src="http://undefinedblog.com/wp-content/uploads/2013/05/Windows-DNS.png" width="419" height="434" />

这种无脑又费时费力的工作当然是程序员不能容忍的，于是，解决方案来了。
<h2>Windows 版</h2>
<h3>设置</h3>
<pre class="lang:vb decode:true">strComputer = "."
arrDNSServers = Array("114.114.114.114", "114.114.115.115")

Set objWMIService = GetObject("winmgmts:" _
  &amp; "{impersonationLevel=impersonate}!\\" &amp; strComputer &amp; "\root\cimv2")
Set Nics = objWMIService.ExecQuery _
  ("SELECT * FROM Win32_NetworkAdapterConfiguration WHERE IPEnabled = True")

For Each Nic In Nics
  intSetDNSServers = Nic.SetDNSServerSearchOrder(arrDNSServers)
  If intSetDNSServers = 0 Then
    WScript.Echo "已切换为114 DNS"
  End If
Next</pre>
<h3>恢复</h3>
<pre class="lang:vb decode:true">strComputer = "."

Set objWMIService = GetObject("winmgmts:" &amp; "{impersonationLevel=impersonate}!\\" &amp; strComputer &amp; "\root\cimv2")

Set colNetAdapters = objWMIService.ExecQuery("Select * from Win32_NetworkAdapterConfiguration where IPEnabled=TRUE")

For Each objNetAdapter In colNetAdapters
errEnable = objNetAdapter.EnableDHCP()
errDNS = objNetAdapter.SetDNSServerSearchOrder(null)
Next
If intSetDNSServers = 0 Then
WScript.Echo "已恢复为默认 DNS"
  End If</pre>
<h3>使用方法</h3>
将上述两个脚本分别保存为「设置DNS.vbs」和「恢复DNS.vbs」，当然文件名可以自拟，但是确保扩展名设置正确，然后双击运行即可。

如果系统默认没有关联 vbs 的解释引擎，可以打开注册表，找到「HKEY_CLASSES_ROOT」-「.vbs」，双击右侧「默认」键，将其值改为「VBSFile」后即可。

注意，「设置 DNS.vbs」中默认的 DNS 设置为 114 的 DNS，这个 DNS 在国内来说效果很不错了。如果你需要换用 Google 或其它的 DNS，修改第 2 行的内容即可。
<h2>OS X 版</h2>
<pre class="lang:sh decode:true">##设置DNS
sudo networksetup -setdnsservers Wi-Fi 114.114.114.114 114.114.115.115"
sudo networksetup -setdnsservers Wi-Fi 8.8.8.8 8.8.4.4
#恢复默认DNS
sudo networksetup -setdnsservers Wi-Fi empty</pre>