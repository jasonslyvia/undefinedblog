---
title: iOS中根据域名获取IP地址的函数
permalink: ioszhong-gen-ju-yu-ming-huo-qu-ipdi-zhi-de-han-shu
id: 15
updated: '2011-10-23 21:40:01'
date: 2011-10-23 21:40:01
tags:
---

<p>OC兼容C语言，可以直接使用C语言的库函数<br /> [cc lang="c++"]<br /> +(NSString*)getIPAddressByHostName:(NSString*)strHostName<br /> {<br /> const char* szname = [strHostName UTF8String];<br /> struct hostent* phot ;<br /> @try<br /> {<br /> phot = gethostbyname(szname);<br /> }<br /> @catch (NSException * e)<br /> {<br /> return nil;<br /> }</p>
<p>struct in_addr ip_addr;<br /> memcpy(&amp;ip_addr,phot-&gt;h_addr_list[0],4);///h_addr_list[0]里4个字节,每个字节8位，此处为一个数组，一个域名对应多个ip地址或者本地时一个机器有多个网卡</p>
<p>char ip[20] = {0};<br /> inet_ntop(AF_INET, &amp;ip_addr, ip, sizeof(ip));</p>
<p>NSString* strIPAddress = [NSString stringWithUTF8String:ip];<br /> reurn strIPAddress;<br /> ｝<br /> [/cc]</p>