---
title: Nginx深入了解-基础(三)
date: 2018-07-19 13:43:25
tags: nginx
---

Nginx有很多开源的第三方模块在实际生产应用中能方便我们使用，包括官方模块和第三方模块。这里介绍下常见的一些模块的使用。



<!-- more -->

- http_stub_status_modules配置

```nginx
Syntax：stub_status
Default：–
Context：server,location
```

> 配置实例:

```nginx
server {
    ...
    location /mystatus {
        stub_status;
    }
}
```

- random_index_module展示随机的首页

```nginx
Syntax：random_index on | off
Default：random_index off
Context：location
```

> 配置实例:

```nginx
server {
    ...
    location / {
        root /opt/app/code;
        random_index on;
    }
}
```

- http_sub_module html内容替换,只能替换第一个

```nginx
Syntax：sub_filter string replacement;
Default：–
Context：http,server,location
```

> 配置实例:

```nginx
server { 
  ...
  location / { 
    root ...; 
    index index.html index.php; 
    sub_filter '替换前的内容' '替换后的内容'; 
  } 
}
```

```nginx
Syntax：sub_filter_last_modified on|off;主要用于缓存
Default：sub_filter_last_modified off;
Content：http,server,location
```

```nginx
Syntax：sub_filter_once on|off; 全局/非全局替换
Default：sub_filter_once on;
Context：http,server,location
```

> 配置实例:

```nginx
server { 
  ... 
  location / { 
    root ...; 
    index index.html index.php; 
    sub_filter '替换前的内容' '替换后的内容'; 
    sub_filter_once off; // 全部替换 
  } 
}
```

- Nginx的请求限制

```nginx
连接频率限制：limit_conn_module
请求频率限制：limit_req_module
```

> 连接限制

```nginx
Syntax：limit_conn_zone key zone=name:size;
Default：–
Context：http
```

```nginx
Syntax：limit_conn zone number;// 需要基于limit_conn_zone
Default：–
Context：http,server,location
```

> 请求限制

```nginx
Syntax：limit_req_zone key zone=name:size rate=rate;
Default：–
Context：http
```

```nginx
Syntax：limit_req zone=name [burst=number][nodelay];// 需要基于limit_req_zone
Default：–
Context：http,server,location
```

> 配置实例:

```nginx
http { 
  .... 
  limit_conn_zone $binanry_remote_addr zone=conn_zone:1m; 
  limit_req_zone $binanry_remote_addr zone=req_zone:1m rate=1r/s; 
} 
server { 
  ... 
  location / { 
    ... 
    limit_conn conn_zone 1; 
    limit_req zone=req_zone burst=3 nodelay; 
    limit_req zone=req_zone burst=3; 
    limit_req zone=req_zone; 
  } 
}
```

