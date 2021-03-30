---
title: Nginx深入了解-基础(二)
date: 2018-07-17 13:56:27
tags: nginx
---

Nginx的安装及相关参数介绍。yum安装nginx后可通过命令rpm -pl nginx查看相关的安装目录。如下:

<!-- more -->

- 参数介绍

| 路径                                                         | 类型           | 作用                                                         |
| ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ |
| /etc/logrotate.d/nginx                                       | 配置文件       | Nginx日志轮转，用于logrotate服务的日志切割                   |
| /etc/nginx /etc/nginx/nginx.conf /etc/nginx/conf.d /etc/nginx/conf.d/default.conf | 目录，配置文件 | nginx主配置文件                                              |
| /etc/nginx/fastcgi_params /etc/nginx/uwsgi_params /etc/nginx/scgi_params | 配置文件       | 编码转换映射转换文件                                         |
| /etc/nginx/mime.types                                        | 配置文件       | 设置http协议的content-type与扩展名对应关系                   |
| /usr/lib/systemd/system/nginx-debug.service /usr/lib/systemd/system/nginx.service    /etc/sysconfig/nginx /etc/sysconfig/nginx-debug | 配置文件       | 用于配置系统守护进程管理器管理方式                           |
| /usr/lib64/nginx/modules /etc/nginx/modules                  | 目录           | nginx模块目录                                                |
| /usr/sbin/nginx /usr/sbin/nginx-debug                        | 命令           | nginx服务的启动管理的终端命令                                |
| /usr/share/doc/nginx-版本号 /usr/share/doc/nginx-版本号/COPYRIGHT /usr/share/man/man8/nginx.8.gz | 文件、目录     | ginx手册、帮助文件                                           |
| /var/cache/nginx                                             | 目录           | nginx的缓存目录，nginx布景可以用作服务代理，还可以用来做缓存服务 |
| /var/log/nginx                                               | 目录           | nginx日志目                                                  |

> 使用```nginx -V```查看配置参数:

```bash
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```

| 编译                                                         | 作用                                      |
| ------------------------------------------------------------ | ----------------------------------------- |
| –prefix=/etc/nginx \ –sbin-path=/usr/sbin/nginx \ –modules-path=/usr/lib64/nginx/modules \ –conf-path=/etc/nginx/nginx.conf \ –error-log-path=/var/log/nginx/error.log –http-log-path=/var/log/nginx/access.log \ –pid-path=/var/run/nginx.pid \ –lock-path=/var/run/nginx.lock | 安装目的目录或路径                        |
| –http-client-body-temp-path=/var/cache/nginx/client_temp \ –http-proxy-temp-path=/var/cache/nginx/proxy_temp \ –http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \ –http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \ –http-scgi-temp-path=/var/cache/nginx/scgi_temp | 执行对应的模块时，nginx所保留的临时性文件 |
| –user=nginx \ –group=nginx                                   | 设定nginx进程启动的用户和用户组           |

- ```nginx.conf```的配置相关

> 第一块

| 参数             | 说明                          |
| ---------------- | ----------------------------- |
| user             | 设置nginx服务的系统使用用户   |
| worker_processes | 工作进程数，与cpu核数保持一致 |
| error_log        | nginxd的错误日志              |
| pid              | nginx服务启动时候的pid        |

> 第二块

| 模块   | 参数               | 说明                   |
| ------ | ------------------ | ---------------------- |
| events | worker_connections | 每个进程允许最大连接数 |
|        | use                | 工作进程数             |

> 第三块

```nginx
http { 
    include /etc/nginx/mime.types; // http的content-type设置 
    default_type application/octet-stream; 
    log_format main '$remote_addr - $remote_user [$time_local] "$request" ' '$status $body_bytes_sent "$http_referer" ' '"$http_user_agent" "$http_x_forwarded_for"';   
    access_log /var/log/nginx/access.log main; 
    sendfile on; #tcp_nopush on; 
    keepalive_timeout 65; #gzip on; 
    // 标准的方式是将多个server配置成多个.conf文件放在/etc/nginx/conf.d下include进来 
    #include /etc/nginx/conf.d/*.conf; 
    server { 
    		listen 80; 
    		server_name localhost; 
    		localtion / { 
      		root .... ;
          index .... ; 
        } 
        error_page .... 
        localtion = /50x.tml { 
      			..... 
    		} 
  	} 
  	server { .... } 
  	server { .... } 
}
```

```log_format```定义access_log的日志内容格式，可自定义。例如添加```user_agent```信息：```$http_User_Agent```.

