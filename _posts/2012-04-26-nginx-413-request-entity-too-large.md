---
layout: post
title: "Nginx错误：413 Request Entity Too Large"
description: ""
category: 
tags: ['Nginx']
---
{% include JB/setup %}

查了一下[官网wiki](http://wiki.nginx.org/HttpCoreModule#client_max_body_size)后发现，原来nginx最大body size默认设为1MB，当用户上传的文件大于这个值就会导致 413 Request Entity Too Large 这个错误。

1MB的默认值对于文件上传来说显得有点小，不过我们可以通过 client_max_body_size 这个设置项修改。

编辑nginx.conf文件，在文件的http段中加上 `client_max_body_size 20M;`

    http {
      …
      client_max_body_size 20M;
      …
    }

保存后退出，接下来重新加载新的配置：

    NGINX_HOME/sbin/nginx -t ＃ 正常会提示syntax is ok，test is successful之类
    NGINX_HOME/sbin/nginx -s reload
