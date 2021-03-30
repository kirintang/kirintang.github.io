---
title: Nginx深入了解-进阶(二)
date: 2018-08-05 10:14:45
tags: nginx
---

Nginx作为代理服务.正向代理：代理对象为客户端.反向代理：代理对象为服务端.

<!-- more -->

- 反向代理

> 配置语法:

```nginx
Syntax：proxy_pass URL
Default：–
Context：location、if in location、limit_except
```

> 配置实例:

```nginx
#server1 
server { 
    ... 
    listen 8080; 
    server_name localhost; 
    ... 
    location / { 
        root /opt/htdocs/html; 
        index index.html index.htm index.php; 
    } 
} 

#server2 
server { 
    listen 80; 
    server_name localhost; 
    ... 
    location ~/reg$ { 
        proxy_pass http://127.0.0.1:8080; // 反向代理8080 
    } 
}
```

- 正向代理

如果我们只允许某一个特定的ip访问，则可要考虑使用正向代理来实现.

> 客户端服务配置实例:

```nginx
server { 
  listen 80; 
  server_name www.mantis.me; 
  ... 
  location / { 
    if ($http_x_forwarded_for !~* "114\.249\.225\.223") { // 只允许114.249.225.223访问return 403; 
    } 
  } 
}
```

> 114.249.225.233服务器配置:

```agin

```

```nginx
server { 
  listen 80; 
  server_name www.mantis.me; 
  ... 
  resolver 8.8.8.8; // dns 
  location / { 
    proxy_pass http://$http_host$request_uri; 
  } 
}
```

客户端使用代理工具配置代理服务器，例如mac系统自带、google扩展工具SwitchySharp等，配置相应的http代理服务器地址。

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190215115049.png)

在浏览器输入[www.mantis.me即可访问](http://www.mantis.xn--me-h95cksq64t60m./).

