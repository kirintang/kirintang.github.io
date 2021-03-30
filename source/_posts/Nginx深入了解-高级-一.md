---
title: Nginx深入了解-高级(二)
date: 2018-08-13 14:08:20
tags: nginx
---

Nginx的静态处理能力很强，但是动态处理能力不足，因此，在企业中常用动静分离技术。动静分离技术其实是采用代理的方式，在server{}段中加入带正则匹配的location来指定匹配项针对PHP的动静分离：静态页面交给Nginx处理，动态页面交给PHP-FPM模块或Apache处理。在Nginx的配置中，是通过location配置段配合正则匹配实现静态与动态页面的不同处理方式。

<!-- more -->

假如我们有一个页面，既有静态资源如图片、js文件等，也有动态的ajax请求，如下：

```html
<!DOCTYPE html> 
<html> 
  .... 
  <div> 
    <img src="https://www.mantis.me/img/abc.jpg"> 
  </div> 
  <script> 
    $.ajax{ 
      type: POST, 
      url: "https://www.mantis.me/user/add.php", 
      dataType: JSON, 
      success: function() {} 
    } 
  </script> 
  <script src="https://www.mantis.me/static/js/main.js"></script> 
</html>
```

我们可以通过nginx正则匹配和location将静态资源和动态请求分离：

```nginx
upstream api { 
  server 127.0.0.1:8080; 
} 

server { 
  listen 80; 
  server_name localhost; 
  access_log /var/logs/nginx/access.log main; 
  root /opt/app/code; 
  
  location ~ \.php$ { 
    proxy_pass http://api; index index.html index.htm index.php; 
  } 
  location ~ \.(jpg|jpeg|png|gif)$ { 
    expires 1h; gzip on; 
  } 
  ...... 
}
```

