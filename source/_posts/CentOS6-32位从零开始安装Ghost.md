---
title: CentOS6 32位从零开始安装Ghost
permalink: centos-installing-ghost
id: 2
updated: '2015-04-14 17:08:08'
date: 2014-03-20 20:05:08
tags:
---

为了换上传说中写作体验超级好的 [Ghost](http://ghost.org)，我下了狠心把 VPS 上的所有内容全部删除，把系统从 CentOS 5 升级到 CentOS 6，在各种教程和手册的指导下花费了一个下午的时间终于配置好了 Ghost。

一句话总结：**能把 Ghost 在 VPS 上配置好的人都值得拿高薪，太麻烦了！**

下面的内容包括了我这一下午折腾的所有操作，你可以根据自己的需要选择阅读。当然如果你也想从头开始安装 Ghost，可以按顺序阅读本文。

##基本信息

本次操作涉及以下内容：

 - 操作系统：CentOS6 32位（查看方式：`cat /etc/*release*`）
 - node版本：0.10.26
 - Ghost版本：0.4.1
 - nginx版本：1.4.7
 - MySQL版本：5.5

操作步骤：

 1. 通过 yum 安装 node
 2. 通过 yum 安装 nginx
 3. 通过 yum 安装 MySQL*
 4. 下载 Ghost 源码
 5. 配置 MySQL
 6. 配置 nginx
 7. 配置 Ghost
 8. Wordpress原有数据导入
 
*Ghost 支持 MySQL 或 SQLite3 数据库，为了日后扩展或迁移方便，我决定使用 MySQL。若你选择使用 SQLite3，可以跳过 MySQL 的相关步骤，同时在 Ghost 的 `config.js` 中使用自己的配置。
 
##编译node

原本我的 VPS 上快乐的运行着 LNMP 环境，本以为再直接装上 node 即可，谁知道 yum 无法直接安装 node。

根据 node 官方的 [wiki](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)， CentOS 需要先安装 epel 的 yum repo 才能使用 yum 安装 node。

```
#更新 yum 的 repo list
rpm -Uvh http://download-i2.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
#同时安装 node 和 npm
sudo yum install nodejs npm --enablerepo=epel
```
安装完成后，分别使用`node -v` 和 `npm -v` 来确认 node 和 npm 以及功能安装成功。

>其实是在写这篇博客的时候才发现可以直接通过 yum 装，我自己是直接编译的 node 源码……

##安装nginx

安装 nginx 的步骤和 node 类似，更新源列表然后直接安装。

````
#下载 repo 列表
wget http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
#通过 rpm 添加列表
rpm -ivh nginx-release-centos-6-0.el6.ngx.noarch.rpm
#通过 yum 安装 nginx
yum install nginx
#将 nginx 设置为开机启动（可选，推荐）
chkconfig nginx on
#启动 nginx 服务
service nginx start
```

下面是一些常用的 nginx 相关文件地址：

 - 主配置文件：/etc/nginx/nginx.conf
 - 默认主机配置文件：/etc/nginx/conf.d/default.conf
 - 默认日志目录：/var/log/nginx/
 
##安装MySQL

步骤依然类似，不再添加注释。

```
rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm

rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

yum --enablerepo=remi,remi-test install mysql mysql-server

service mysqld start

chkconfig mysqld on
```

##安装 Ghost

这个简单，因为 node 环境已经部署好，运行 Ghost 只需要将 Ghost 的源码下载并解压就好了。

这里我把 `/var/www` 设置为服务器的根目录，你可以根据你的喜好进行设置，但是注意在下面的脚本中替换相关路径。

```
cd /var && mkdir www && cd www
wget http://dl.ghostchina.com/Ghost-0.4.1.zip && unzip Ghost*
```

现在 `/var/www` 目录结构如下（使用`ls`查看）：
```
.
├── config.js
├── content
├── core
├── Gruntfile.js
├── index.js
├── LICENSE
├── node_modules
├── package.json
└── README.md
```

##配置MySQL

MySQL 默认用户名为 `root`，默认密码为空，这是十分不安全的。因此我们要对 MySQL 进行一些简单的安全配置，然后为 Ghost 创建一个专门的数据库和专门的用户（使用root用户来连接Ghost使用的数据库是十分不安全的做法！）。

`mysql_secure_installation`

运行上述命令可以进入 MySQL 的交互式安装程序，基本流程是：

 1. 输入安装 MySQL 时指定的 root 密码，一般直接按回车
 2. 是否改变 root 密码，输入 y
 3. 输入新的 root 密码
 4. 是否删除匿名用户，输入 y
 5. 是否禁止 root 远程登录，输入 y
 6. 是否删除默认的 test 数据库，输入 y
 7. 是否马上应用最新的设置，输入 y
 
至此整个 MySQL 的安装过程结束。下一步开始创建为 Ghost 专用的用户名和数据库。

```
#登录MySQL
mysql -u root -p 你的密码

#创建名为ghost的用户并新建名为ghost的数据库，同时给ghost用户授予ghost数据库的所有权限
CREATE DATABASE ghost;
GRANT ALL PRIVILEGES ON ghost.* To 'ghost'@'127.0.0.1' IDENTIFIED BY '为ghost用户设置一个与root不同的密码';
```
##配置nginx

安装完 nginx 后，它的主配置文件在 `/etc/nginx/nginx.conf`，但是这里面没有太多要修改的东西。

我们主要修改 `/etc/nginx/conf.d/default.conf`，我最终的配置如下，见注释。

```

server {
    listen       80;
    #你的域名
    server_name  undefinedblog.com;

	#访问日志地址
    access_log  /var/log/nginx/log/host.access.log  main;

    #后续Wordpress原有数据导入章节会详细说明
    location /wp-content/ {
        root /var/www/content/images;
    }

	#下面几个location都是让nginx直接serve静态文件
    location ~ ^/(img/|css/|lib/|vendor/|fonts/|robots.txt|humans.txt) {
      root /var/www/core/client/assets;
      access_log off;
      expires max;
    }

    location ~ ^/(shared/|built/) {
      root /var/www/core;
      access_log off;
      expires max;
    }

    location ~ ^/(favicon.ico) {
      root /var/www/core/shared;
      access_log off;
      expires max;
    }

    location ~ ^/(content/images/) {
      root /var/www;
      access_log off;
      expires max;
    }

	#后续Wordpress原有数据导入会详细讲解
    rewrite ^/(\d+)/(\d+)/(.*)$ http://undefinedblog.com/$3 permanent;

	#核心block，将请求proxy到Ghost实例上
    #其中端口可以在Ghost的config.js中修改，但要保持一致
    location / {
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        Host      $http_host;
        proxy_pass              http://127.0.0.1:2368;
    }
}
```
修改完配置后执行`service nginx restart`，如果没有报错说明配置是没有问题的。

##配置Ghost

做了这么多的铺垫工作，终于要开始配置 Ghost 了。打开 `/var/www/config.js`（如果不存在该文件，则先执行`mv config-sample.js config.js`将默认的配置文件重命名）。

打开`config.js`后，直接修改`production`模块。

```
...
production: {
    url: 'http://undefinedblog.com', //替换为你自己的域名。
    mail: {},
    database: {
        client: 'mysql',
        connection: {
            host     : '127.0.0.1',
            user     : 'ghost', //上面配置过
            password : 'ghost对应的密码', //上面配置过
            database : 'ghost', //我们前面为 Ghost 创建的数据库名称
            charset  : 'utf8'
        }
    },
    server: {
        host: '127.0.0.1',
        port: '2368'//若修改该端口记得在nginx中做相应改变
    }
},
...
```
配置完成后，使用`npm`安装所有的依赖。
```
npm install --production
```
>关于为什么加上 `--production` 这个flag，Ghost 官网给出了解释。虽然我们现在直接在编辑Ghost的源码，看起来像是在`develop`，但实际上如果你不加`--production`，npm安装依赖时会安装一堆开发Ghost核心时才需要的文件，这些文件在运行Ghost的时候是不需要的。

安装完所有的依赖，可以运行 Ghost 的了。如果我们直接执行 `node index.js`，当你的ssh 断开后 node 也会停止运行，所以我们需要另一个模块`forever`，让我们的 node 程序一直执行。

```
npm install forever -g
```

安装完 forever 后，再执行下面的命令启动 Ghost：

```
NODE_ENV=production forever start index.js
```

如果不出意外，输入你的域名就能看到 Ghost 的界面了。当然，一般都是会出现意外的，这个时候建议你查看如下内容：

 - nginx 错误日志，`/var/log/nginx/error.log`
 - Ghost 代码所在目录的权限
 - Ghost 的配置是否正确
 - 数据库能否正常连接
 
##Wordpress原有数据导入

对于从 Wordpress 转过来的人，一般会遇到两个问题：

 1. 原有博文数据的导入
 2. 原有附件（图片）的导入
 3. 原链接到新链接的跳转
 
下面分享一下我的解决方案。

###博文数据导入

对于Wordpress数据导入到Ghost，你可以安装一款 Wordpress 插件 `Ghost`，然后在后台`工具`一栏导出 Ghost 支持的数据格式。

注意：目前支持导出的内容包括标题、作者、日期、内容、链接（permalink，slug）、标签，不支持其它的 postmeta。

我在 Ghost 导入数据时（http://你的ghost博客地址/ghost/debug ），遇到提示 slug 长度不能大于 150 的提示。我看了一下 Wordpress 导出的数据中，如果 slug 含有中文会被默认 urlencode，导致长度过大。

系统的默认提示为
>"Property 'slug' exceeds maximum length of 150."

我写了一个简单的 node 脚本处理这个问题：

```
var fs = require('fs');

//data.json是原来导出的文件
fs.readFile('data.json', {encoding: 'utf-8'}, function(err, data){
    if(err) throw err;
    data = JSON.parse(data);
    data.data.posts.forEach(function(v){
        v.slug = decodeURIComponent(v.slug);
    });
    
	//data-new.json是处理后的文件，在Ghost后台导入该文件即可
    fs.writeFile('data-new.json', JSON.stringify(data), function (err){
        if(err) throw err;
        console.log('solved!');
    });
});
```

###图片的导入

首先将你Wordpress里的`/wp-content/uploads`目录压缩后，然后在 `/var/www/content/images`下新建一个目录名为`wp-content`，然后压缩包上传到 Ghost 所在的 `/var/www/content/images` 目录中，最后解压缩。

新的目录结果如下：

```
.
├── data
│   └── README.md
├── images
│   ├── README.md
│   └── wp-content
├── plugins
│   └── README.md
└── themes
    └── casper
```

然后就没有然后了，在上一步配置 nginx 的时候我们加入的 
```
location /wp-content/ {
    root /var/www/content/images;
}
```
语句，意思就是请求 uri 包含 `/wp-content/` 时，从 `/var/www/content/images/` 中获取数据，举个例子。

原请求：

`/wp-content/uploads/2014/01/abc.jpg`

经过 nginx 处理，变成了：

`/var/www/content/images/wp-content/uploads/2014/01/abc.jpg`

这样图片就能顺滑的显示了。

###原链接跳转

我原来在 Wordpress 中设置的链接形式是：

`/%year%/%month%/%post_name%/`

但是 Ghost 只支持：

`/%year%/%month%/%day%/%post_name%/` （在Ghost后台设置）

或

`/%post_name%/`

两种，因此我配置了 nginx 对请求进行简单的跳转。

```
rewrite ^/(\d+)/(\d+)/(.*)$ http://undefinedblog.com/$3 permanent;
```

nginx 的跳转语法比较简单，都是基本的正则，你可以根据自己的需求进行调整。

##后续工作

这就是我花费了一个下午的成果，加上一个晚上来写这篇博客。

目前的 Ghost 还不太成熟，下一步我准备自己写一个简单的主题，包含以下功能：

 - 自动集成语法高亮
 - 输出 meta tag 方便 seo
 - 支持 IE8 及更高版本浏览器
 - 将日期格式换成 yyyy-mm
 - 等
 
 **update 2014/04/20**
 
 新的主题已经做好了，请参考[alarm the Ghost theme](https://github.com/jasonslyvia/alarm)
 
##参考链接
 - [1] [Ghost 官网](http://ghost.org)
 - [2] [Ghost 中国](http://www.ghostchina.com/)
 - [3] [node wiki](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)
 - [4] [ghost nginx 静态文件缓存](https://ghost.org/forum/installation/181-nginx-setup-with-static-file-serving/)
 - [5] [MySQL创建用户及授权](http://stackoverflow.com/questions/1720244/create-new-user-in-mysql-and-give-it-full-access-to-one-database)
 - [6] [MySQL安装](http://stackoverflow.com/questions/9361720/update-mysql-version-from-5-1-to-5-5-in-centos-6-2)