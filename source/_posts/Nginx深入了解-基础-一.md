---
title: Nginx深入了解-基础(一)
date: 2018-07-17 15:22:04
tags: nginx
---

Centos下Nginx安装的正确姿势；Nginx安装有很多种方式，但是在centos下如何能够快速且按照nginx官方标准的安装nginx呢？

<!-- more -->

首先登录```nginx```官方网站:[http://nginx.org，点击右侧download菜单，选择底部Pre-Built](http://nginx.org，点击右侧download菜单，选择底部Pre-Built/) Packages对应的```stable version```（墙裂建议）找到对应的```Centos```操作系统，复制

```bash
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
```

这段代码。

进入我们的机器在``` /etc/yum.repos.d/``` 目录下新建 ```nginx.repo ```文件，将上面代码粘贴，注意要修改的地方有两处，我们是在``` centos ```下安装的，所以修改其中的``` OS ```为 ```centos``` ，同时修改``` OSRELEASE ```为我们的 ```centos ```版本号，比如使用的是``` centos7.2``` 我们将其修改为7



至此，我们的 ```nginx``` 对应的官方``` yum ```源就配置好了，我们使用``` yum list|grep nginx ```来查看对应的 ```nginx ```信息。我们可以看到最新的稳定 ```nginx ```版本:

```bash
collectd-nginx.x86_64 5.8.0-4.el7 epel munin-nginx.noarch 2.0.33-1.el7 epel 
nextcloud-nginx.noarch 10.0.4-2.el7 epel 
nginx.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-all-modules.noarch 1:1.12.2-2.el7 epel 
nginx-debug.x86_64 1:1.8.0-1.el7.ngx nginx 
nginx-debuginfo.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-filesystem.noarch 1:1.12.2-2.el7 epel 
nginx-mod-http-geoip.x86_64 1:1.12.2-2.el7 epel 
nginx-mod-http-image-filter.x86_64 1:1.12.2-2.el7 epel 
nginx-mod-http-perl.x86_64 1:1.12.2-2.el7 epel 
nginx-mod-http-xslt-filter.x86_64 1:1.12.2-2.el7 epel 
nginx-mod-mail.x86_64 1:1.12.2-2.el7 epel 
nginx-mod-stream.x86_64 1:1.12.2-2.el7 epel 
nginx-module-geoip.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-module-geoip-debuginfo.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-module-image-filter.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-module-image-filter-debuginfo.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-module-njs.x86_64 1:1.14.0.0.2.2-1.el7_4.ngx nginx 
nginx-module-njs-debuginfo.x86_64 1:1.14.0.0.2.2-1.el7_4.ngx nginx 
nginx-module-perl.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-module-perl-debuginfo.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-module-xslt.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-module-xslt-debuginfo.x86_64 1:1.14.0-1.el7_4.ngx nginx 
nginx-nr-agent.noarch 2.0.0-12.el7.ngx nginx 
owncloud-nginx.noarch 9.1.5-1.el7 epel 
pcp-pmda-nginx.x86_64 3.11.8-7.el7 base 
python2-certbot-nginx.noarch 0.25.1-1.el7 epel
```

上述看到的是最新版本`nginx.x86_64/1:1.14.0-1.el7_4.ngx`，直接使用`yum install nginx`就可以安装了。安装完成后使用`nginx -v`和`nginx -V`查看对应的版本信息和配置信息。

