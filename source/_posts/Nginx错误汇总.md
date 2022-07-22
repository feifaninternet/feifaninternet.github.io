-------------------------
title: Nginx错误汇总
tags: Nginx
categories: Expand
date: 2020-02-12 16:42:41
-------------------------


### Description
由于最近一段时间 nginx 总是提示 nginx error，所以我想把nginx出现的问题及解决方案或者思路汇总起来，方便以后遇到能快速应对 

### 1.nginx error: upstream prematurely closed connection while reading response header from upstream
偶尔出现nginx error错误，从上游读取响应头时上游提前关闭连接

1.去除nginx.conf中的长连接配置，即删除如下配置，重启nginx

   ```jshelllanguage
    keepalive_timeout  120s 120s;
   ```
   
2.或者再location中增加如下配置，重启nginx

   ```jshelllanguage
    location / {
    
    proxy_pass http://backend;
    
    proxy_http_version 1.1;                         # 设置http版本为1.1
    
    proxy_set_header Connection "";      # 设置Connection为长连接（默认为no）}
    
    }
   ```
3.如果2种方式都无效，尝试在nginx.conf中添加如下配置后重启nginx

   ```jshelllanguage
    keepalive_timeout  120s 120s;
    keepalive_requests 10000;    #默认100，在一个长连接上可以服务的最大请求数目,
                                  当达到最大请求数目且所有已有请求结束后，连接被关闭
   ```
4.如果还是没有效果，请检查node.js或者其他服务是否存在端口占用情况，修改端口，或者查看node后台日志是否有内存溢出问题，有的话可能需要升级node版本