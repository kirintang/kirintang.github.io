---
title: Nginx深入了解-基础(四)
date: 2018-07-20 10:21:03
tags: nginx
---

Nginx的访问控制。有两种方式可以来进行webserver的访问控制：一种是基于IP的访问控制-http_access_module;另一种是基于用户的信任登录-http_auth_basic_module.

<!-- more -->

- http_access_module

```nginx
Syntax：allow address|CIDR|unix:|all;// ip地址|网段|socket方式|允许所有
Default：–
Context：http,server,location,limit_except
```

相对应的deny方式

```nginx
Syntax：deny address|CIDR|unix:|all;
Default：–
Context：http,server,location,limit_except
```

> 配置实例:

```nginx
server { 
  ... 
  location / { 
    ... 
    deny 127.0.0.1; allow all; 
  } 
  location ~ ^/admin { 
    ... 
    allow 127.0.0.1; deny all; 
  } 
}
```

http_access_module是有局限性的，当客户端使用cdn代理时，nginx读取客户端的ip是通过remote_addr来识别的，识别到的ip此时是cdn代理的ip，而不是客户端真实的ip.



方式一：可以使用http_x_forwarded_for解决。

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190215115219.png)

方式二：结合geo模块

方式三：通过http自定义变量传递

- http_auth_basic_module

```nginx
Syntax：auth_basic string|off;
Default：auth_basic off;
Context：http,server,location,limit_except
```

官方文档：[http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)

```nginx
Syntax：auth_basic_user_file file;// 使用文件密码信息
Default：–
Context：http,server,location,limit_except
```

按照官网可以使用htpasswd方式生成对应的文件:

```bash
[dc2-user@10-254-0-193 nginx]$ sudo htpasswd -c ./auth_conf mantis 
[dc2-user@10-254-0-193 nginx]$ more ./auth_conf mantis:$apr1$dnrF/7bE$gaMkEYvWB2KYmaG0cQcoS0
```

配置:

```nginx
server { 
  ... 
  location ~ ^/admin { 
    ... 
    auth_basic "Auth access deny!"; 
    auth_basic_user_file /etc/nginx/auth_conf; 
    ... 
  } 
}
```

局限性：

一，用户信息依赖文件
二，操作管理机械，效率低下

解决方式：

一，使用Lua实现验证
二，Nginx和LDAP打通，利用nginx-auth-ldap模块