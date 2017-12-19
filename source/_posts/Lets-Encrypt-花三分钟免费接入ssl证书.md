---
title: Lets Encrypt 花三分钟免费接入ssl证书
permalink: lets-encrypt
id: 180
updated: '2016-05-01 19:59:54'
date: 2016-01-03 05:58:24
tags:
---

2016-05-01 19:58:46 更新：最近很多朋友说我的 https 证书过期了，于是重新按照 DO 的[这篇教程](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04)重新配置了一遍。不得不说，DO 写教程的水平确实一流！建议移步学习。

---

很久很久以前我还在当个人站长的时候，就一直琢磨着给网站买个 ssl 证书，这样用户访问的时候浏览器上的小绿锁能给人一种非常安全的印象。然而导致我最终没有上 https 的原因有：

1. 证书好贵 
2. https 会影响 SEO（据说） 
3. 懒（~~这才是真正的原因~~）

直到前段时间发现了 [Lets Encrypt 项目](https://letsencrypt.org/)，一个倡导互联网上所有网站都该使用 https 的组织，提供免费的 ssl 证书服务。

![ssl certificate price in godaddy](http://ww3.sinaimg.cn/bmiddle/831e9385gw1ezlvzdzwsoj20a70fb75d.jpg)

上图是 GoDaddy 的 ssl 证书报价，换句话说，Lets Encrypt 免费送你价值 62.99 刀的证书。

当然，这里提到价格并不是为了让大家薅羊毛，而是为了体现 Lets Encrypt 组织的高尚目标。希望看到本文的同学，能够享受免费 ssl 证书带来了安全和稳定，而不是产生牟利等不当想法。

使用的方法也很简单，以 nginx 为例，直接使用 Github 上 xdtianyu 同学提供的[脚本](https://github.com/xdtianyu/scripts/tree/master/lets-encrypt)即可。

## 操作步骤

切换到你希望保存证书及私钥的目录，我选择当前用户的 home，即 `~`。

> 下文中所有 `/path/to/` 请替换成你服务器上的实际路径；`your_domain` 同理替换成你自己的域名

下载脚本及配置文件

```bash
cd ~
wget https://raw.githubusercontent.com/xdtianyu/scripts/master/lets-encrypt/letsencrypt.conf
wget https://raw.githubusercontent.com/xdtianyu/scripts/master/lets-encrypt/letsencrypt.sh
chmod +x letsencrypt.sh
```

编辑配置文件，将第2、3、4行中的 Example.com 替换成你自己的域名

```bash
vim letsencrypt.conf
```

执行 letsencrypt 脚本，获取证书及私钥

```bash
./letsencrypt.sh ./letsencrypt.conf
```

添加 nginx alias

```bash
server {
  ...

  location /.well-known/ {
    alias /path/to/.well-known/;
  }

  ...
}
```

修改 nginx 配置，加载私钥和证书
```bash
server {
   listen 443 ssl;
   ssl_certificate /path/to/your_domain.chained.crt;
   ssl_certificate_key /path/to/your_domain.com.key;

   ...
}
```

全部操作完成后，重启 nginx 即可。


```bash
service nginx restart
#或 
/etc/init.d/nginx -s reload
```

## 额外配置

首先，Lets Encrypt 的证书需要每月更新，因此你需要一个 cron 命令定时运行上述脚本

```bash
0 0 1 * * ~/letsencrypt.sh ~/letsencrypt.conf >> /var/log/lets-encrypt.log 2>&1
```

其次，如果你希望重定向所有的非 https 到 https 访问，可以在 nginx 配置中添加如下 server block

```bash
server {
    listen 80;
    server_name your_domain.com;
    return https://your_domain.com$request_uri;
}
```

总的来说，Lets Encrypt 所倡导免费及普遍的 ssl 证书有利于整个互联网环境的安全化。对于某些靠流量劫持广告的 ISP 来说，苦日子要来了。
