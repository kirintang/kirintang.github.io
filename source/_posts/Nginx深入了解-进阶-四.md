---
title: Nginx深入了解-进阶(四)
date: 2018-08-13 09:32:50
tags: nginx
---

Nginx同样可以用来作为缓存服务；客户端浏览器缓存我们称之为客户端缓存，后端使用Redis、Memcache等缓存服务我们称之为后端缓存，同理Nginx作为缓存服务我们就称之为代理缓存。

<!-- more -->

#### 一，Nginx作为代理缓存的流程示意图：

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190215114209.png)

- 配置语法：

```nginx
Syntax：proxy_cache_path path [levels=levels]

[use_temp_path = on|off] keys_zone=name:size [inactive = time]

[max_size=size] \ [manager_files=number] \ [manager_sleep=time]

[manager_threshold=time] \ [loader_files=number]

[loader_sleep=time] \ [loader_threshold=time] \ [purger=on|off]

[purger_files=number] \ [purger_sleep=time]

[purger_threshold=time];

Default：–

Context：http
```

- proxy_cache配置语法

```nginx
Syntax：proxy_cache zone|off;

Default：proxy_cache off;

Context：http、server、location
```

- 缓存过期周期

```
Syntax：proxy_cache_valid [code…] time;

Default：–

Context：http、server、location
```

- 缓存维度

```nginx
Syntax：proxy_cache_key string;

Default：proxy_cache_key $schema$proxy_host$request_uri; // 协议+主机+url

Context：http、server、location
```

#### 二，配置实例

```nginx
http { 
    ......
    proxy_cache_path /var/cache levels=1:2 keys_zone=test_cache:10m max_size=10g inactive=60m use_temp_path=off; #60m是指60分钟，1:2两级目录，test_cache开辟的空间名称 
    server { 
        listen 80; 
        server_name localhost; 
        access_log /var/logs/access.log main; 
        location / { 
            proxy_cache test_cache; 
            proxy_cache_valid 200 304 12h; 
            proxy_cache_valid any 10m; 
            proxy_cache_key $host$uri$is_args$args; 
            add_header Nginx-Cache "$upstream_cache_status"; # 增加头信息 key(Nginx-Cache) value($upstream_cache_status) 
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504; # 当出现5xx，超时，错误等时，跳过直接访问下一台服务器 
            include proxy_params; 
        } 
    } 
}
```

#### 三，清理指定缓存信息
使用第三方模块ngx_cache_purge来实现。

#### 四，部分页面不缓存，比如登录注册页不希望缓存，可以使用proxy_no_cache实现

```nginx
Syntax：proxy_no_cache string …;

Default：—;

Context：http、server、location;
```

- 配置实例

```nginx
server { 
    ...... 
    if ($request_uri ~ ^/(login|register|password\/reset)) { 
        set $cookie_nocache 1; 
    } 
    location / { 
        proxy_cache test_cache; 
        proxy_cache_valid 200 304 12h; 
        proxy_cache_valid any 10m; 
        proxy_cache_key $host$uri$is_args$args; 
        proxy_no_cache $cookie_nocache $arg_nocache $arg_comment; 
        proxy_no_cache $http_pragma $http_authorization; 
        add_header Nginx-Cache "$upstream_cache_status"; # 增加头信息 key(Nginx-Cache) value($upstream_cache_status) 
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504; # 当出现5xx，超时，错误等时，跳过直接访问下一台服务器 
        include proxy_params; 
    } 
}
```

#### 五，大文件分片请求混存

- 优势：每个子请求收到的数据都会形成一个独立的文件，一个请求断了，其他请求不受影响。

- 缺点：当文件很大时或者slice很小时，可能会导致文件描述符耗尽等情况。