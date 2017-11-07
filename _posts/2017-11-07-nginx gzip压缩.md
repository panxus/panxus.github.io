---
layout: post
title: nginx gzip压缩
date: 2017-11-07
categories: blog
tags: [gzip,nginx]
description: nginx gzip压缩
---



###### nginx开启gzip压缩网页

1. 简述
> 1.HTTP协议上的GZIP编码是一种用来改进web应 用程序性能的技术。大流量的WEB站点常常使用GZIP压缩技术来让用户感受更快的速度。这一般是指WWW服务器中安装的一个功能，当有人来访问这个服务 器中的网站时，服务器中的这个功能就将网页内容压缩后传输到来访的电脑浏览器中显示出来.一般对纯文本内容可压缩到原大小的40%.这样传输就快了，效果 就是你点击网址后会很快的显示出来.当然这也会增加服务器的负载. 一般服务器中都安装有这个功能模块的。

> 2.减少文件大小有两个明显的好处，一是可以减少存储空间，二是通过网络传输文件时，可以减少传输的时间。gzip 是在 Linux 系统中经常使用的一个对文件进行压缩和解压缩的命令，既方便又好用。

> 3.在Nginx安装完成之后，我们可以开启Gzip压缩功能，这里Nginx默认只能对text/html类型的文件进行压缩。


2. 配置文件配置
```
# 启用 gzip 压缩功能
gzip  on;    
# 默认值是1.1，就是说对HTTP/1.1协议的请求才会进行gzip压缩
gzip_http_version 1.1;    
gzip_vary on;    
# 压缩级别，1压缩比最小处理速度最快，9压缩比最大但处理最慢，同时也最消耗CPU,一般设置为2/3就可以了
gzip_comp_level 2;    
# nginx 做前端代理时启用该选项，表示无论后端服务器的headers头返回什么信息，都无条件启用压缩
# gzip_proxied any;    
# 什么类型的页面或文档启用压缩
gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript image/jpeg image/gif image/png;    
# 最小压缩的页面，如果页面过于小，可能会越压越大，这里规定大于1K的页面才启用压缩
gzip_min_length  1k;    
# 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流
gzip_buffers 4 16k;    
# 禁用IE6的gzip压缩
gzip_disable "MSIE [1-6].(?!.*SV1)";
```