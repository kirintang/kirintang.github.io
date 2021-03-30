---
title: Nginx深入了解-高级(二)
date: 2018-08-18 14:08:20
tags: nginx
---

Nginx的Rewrite规则，实现url的重定向以及重写，用来实现特定场景的支持；比如特定页面、接口的转发等。

<!-- more -->

##### 一，场景

1）URL访问跳转，支持开发设计

 页面跳转、兼容性支持、展示效果等。

2）SEO优化

 伪静态改写，优化搜索引擎。

3）维护

 后台维护、流量转发等。

4）安全

 伪静态，将真实的页面进行伪装，让外部的爬虫、黑客攻击感觉不到是一个动态的页面。

##### 二，配置语法

> Syntax：rewrite regex replacement [flag];
>
> Default：-
>
> Context：server、location、if

eg：当网站需要维护时，可以指定一个页面，通过rewrite规则转发。

```nginx
rewrite ^(.*)$ /pages/main.html break;
```

##### 三，正则表达式

| 表达式 |                  描述                  |
| :----: | :------------------------------------: |
|   .    |       匹配除换行符以外的任意字符       |
|   ?    |              重复0次或1次              |
|   +    |            重复1次或更多次             |
|   *    |   最少连接数，哪个机器连接数少就分发   |
|   \d   |                匹配数字                |
|   ^    |            匹配字符串的开始            |
|   $    |            匹配字符串的结束            |
|  {n}   |                重复n次                 |
|  {n,}  |            重复n次或更多次             |
|  [c]   |             匹配单个字符c              |
| [a-z]  |       匹配a-z小写字母的任意一个        |
|   \    |                转义字符                |
|   ()   | 用于匹配括号之间的内容，通过$1、$2调用 |

eg：匹配user_agent信息里面MISE的

```nginx
if ($http_user_agent ~ MISE) {
    rewrite ^(.*)$ /mise/$1 break; # $1匹配()内的内容
}
```

##### 四，Nginx中的flag

|   flag    |                    描述                     |
| :-------: | :-----------------------------------------: |
|   last    |      停止rewrite检测，会再建立一次请求      |
|   break   |       停止rewrite检测，不会再建立请求       |
| redirect  | 返回302临时重定向，地址栏会显示跳转后的地址 |
| permanent | 返回301永久重定向，地址栏会显示跳转后的地址 |

eg：

```nginx
# last break 
server { 
  listen 80; 
  server_name localhost; 
  ... 
  location ~ ^/break { 
    rewrite ^/break /test/ break; 
  } 
  location ~ ^/last { 
    rewrite ^/last /test/ last; # 返回200 {"status":"success"} 
  } 
  location /test/ { 
    default_type application/json; 
    return 200 '{"status":"success"}'; 
  } 
}
```

eg:

```nginx
# redirect permanent
server { 
  listen 80; 
  server_name localhost; 
  ... 
  location ~ ^/mantis { 
    #rewrite ^/mantis https://www.mantis.me/ permanent; #301 
    rewrite ^/mantis https://www.mantis.me/ redirect; #302 
  } 
}
```

