---
title: Nginx深入了解-进阶(一)
date: 2018-07-23 10:22:57
tags: nginx
---

Nginx用来作为静态资源web服务；CDN、浏览器缓存、跨域、防盗链等。

<!-- more -->

非服务器动态运行生成的文件:

| 类型         | 种类                |
| ------------ | ------------------- |
| 浏览器端渲染 | HTML、CSS、JS       |
| 图片         | JPG、GIF、JPEG、PNG |
| 视频         | FLV、MPEG           |
| 文件         | TXT等               |

#### 一，静态资源CDN

- 配置语法-文件读取

```nginx
Syntax：sendfile on|off
Default：sendfile off
Context：http、server、location、if on location
```

- 配置语法-tcp_nopush(在sendfile开启的情况下，提高网络包的传输效率)

```nginx
Syntax：tcp_nopush on|off
Default：tcp_nopush off
Context：http、server、location
```

- 配置语法-tcp_nodelay(在keepalive连接下，提高网络包的传输实时性)

```nginx
Syntax：tcp_nodelay on|off
Default：tcp_nodelay off
Context：http、server、location
```

- 配置语法-压缩(压缩传输文件)

```nginx
Syntax：gzip on|off
Default：gzip off
Context：http、server、location、if on location
```

```nginx
Syntax：gzip_comp_level
Default：gzip_comp_lvel 1;
Context：http、server、location
```

```nginx
syntax：gzip_http_version 1.0|1.1
Default：gzip_http_version 1.1
Context：http、server、location
```

- 扩展Nginx压缩模块
  - http_gzip_static_module-预读gzip功能
  - http_gunzip_module-应用支持gunzip压缩方式
- 配置实例:

```nginx
server { 
  ... 
  sendfile on; 
  location ~ .*\.(jpg|png|gif)$ { 
    gzip on; 
    gzip_http_version 1.1; 
    gzip_comp_level 2; 
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png; 
  } 
  location ~ .*\.(txt|xml)$ { 
    gzip on; 
    gzip_http_version 1.1; 
    gzip_comp_level 1; 
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png; 
  } 
  location ~ ^/download { 
    gzip on; tcp_nopush on; 
    gzip_static on; 
    ... 
  } 
}
```

#### 二，浏览器缓存

| 优先级 | 机制                    | 参数                            |
| ------ | ----------------------- | ------------------------------- |
| 1      | 校验是否过期            | Expires、Cache-Control(max-age) |
| 2      | 协议中的Etag头信息校验  | Etag                            |
| 3      | Last-Modified头信息校验 | Last-Modified                   |

- 配置实例

```nginx
server {
    ...
    expires 24h;
}
```

#### 三，跨域访问

通俗地讲，跨域访问就是在访问某一个网址([www.mantis.me](http://www.mantis.me/)) 的同时访问另一个网址([www.xxx.com](http://www.xxx.com/)) ，对于浏览器来说这是不允许的，因为容易出现CRSF攻击，浏览器会阻止这种行为。

- 配置语法:

```nginx
Syntax：add_header name value [always]
Default：–
Context：http、server、location、if in location
```

浏览器会检查Access-Control-Allow-Origin对应的值，提示错误：

```http
XMLHttpRequest cannot load http://www.mantis.me/. The ‘Access-Control-Allow-Origin’ header has a value ‘http://www.xxx.com' that is not equal to the supplied origin. Origin ‘http://www.mantis.me/' is therefore not allowed access.
```

- 配置实例:

```nginx
server { 
  ... 
  location ~ .*\.(html|htm)$ { 
    ... 
    add_header Access-Control-Allow-Origin www.xxx.com; // 允许所有可以设置为*(不建议，容易被跨域攻击) 
    add_header Access-Control-Allow-Methods POST,GET,OPTIONS,PUT,DELETE; 
  } 
}
```

#### 四，防盗链

防止资源被盗用，其他网站盗用我们网站的资源链接信息导致资源信息被盗用同时还有可能导致我们服务器的压力增大。

- http_refer防盗链配置

```nginx
Syntax：valid_referers none|blocked|server_names|string…;
Default：—
Context：server、location
```

- 配置实例:

```nginx
server { 
  location ~ .*\.(jpg|jpeg|png|gif)$ { 
    ... 
    valid_referers blocked www.mantis.me ~/baidu\./;// 这里只允许refer头为www.mantis.me的地址和baidu搜索过来的，可以便于seo优化 
    if ($invalid_referer) { 
      return 403; 
    } 
  } 
}
```

