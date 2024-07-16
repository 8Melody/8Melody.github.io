---
layout: post
title: "Nginx反向代理处理跨域"
subtitle: "Nginx proxy_pass"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Nginx
---

> 记录使用Nginx反向代理解决前端跨域问题。

# Nginx反向代理解决前端跨域问题

> No 'Access-Control-Allow-Origin' header is present on the requested resource.

前言：最近在抓包快手用户结构及视频的时候，使用`Fiddler`抓完包，反编译破解`sig`算法之后，使用H5做了一个分析工具，在请求的过程中，出现了如上错误，当然这也是意料之中的事情。

这里使用了某台服务器的`Nginx`来解决这个问题，利用`Nginx`的反向代理设置响应头。

```javascript
location ^~ /rest/ {
       add_header Access-Control-Allow-Origin *;
       add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
       add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';

       if ($request_method = 'OPTIONS') {
          return 204;
        }
        
       proxy_pass http://api.gifshow.com/rest/;
    }

```

具体参见[HTTP访问控制。](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)

