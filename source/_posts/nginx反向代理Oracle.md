---
title: nginx反向代理Oracle
date: 2020-12-03 14:27:43
tags: [Nginx]
categories: [Nginx]
comments: true
---
微服务架构下众多服务分布在不同的服务器，连接同一个数据库，造成安全隐患。一旦其中一台服务被劫持，数据库就存在被攻击，被劫持的可能。

为每个服务提供单独的数据库成本太高，数据库网关还未建设，计划通过nginx反向代理tcp连接，实现数据库访问。
# 升级nginx
nginx增加tcp模块使用`--with-stream`

## 查询nginx原有的模块
查看configure
>/nginx/sbin/nginx -V

configure arguments如下
>configure arguments: --prefix=/home/nginx --with-http_stub_status_module --with-http_ssl_module

<!--more-->
## 编译SSL模块
切换到源码包，在之前的configure arguments后面增加stream模块`--with-stream`
>./configure --prefix=/home/nginx --with-http_stub_status_module --with-http_ssl_module --with-stream

配置完成后，运行命令`make`,这里不要进行`make install`，否则就是覆盖安装。

## 覆盖Nginx
备份原有Nginx
>cp /nginx/sbin/nginx /nginx/sbin/nginx-bak

关闭进程nginx master进程，将刚编译好的nginx复制到sbin下
>cp /home/soft/nginx/objs/nginx /home/nginx/sbin
>
启动nginx，查看安装模块
>/nginx/sbin/nginx -V

# 配置Nginx Config

在nginx.config中配置stream模块

 ```bash
#修改工作进程数量为cpu数
worker_processes  4;
stream {
      #日志
      log_format proxy '[$time_iso8601]---remote-ip:$remote_addr,'
                  'protocol:$protocol,status:$status,byte_sent:$bytes_sent,byte_received:$bytes_received,'
                  'session_time:$session_time,upstream_addr:$upstream_addr,'
                  'upstream_bytes_sent:$upstream_bytes_sent,upstream_bytes_received:$upstream_bytes_received,upstream_connect_time:$upstream_connect_time';
 
     access_log  logs/stream.access.log proxy;
     error_log   logs/stream.error.log error;
 
     #代理oracle数据库
     upstream wxdoracle{
         server 10.xx.xx.xx:1521;
     }
 
     server {
         listen 10521 so_keepalive=on;
         proxy_connect_timeout 50s;#连接事件增加
         #proxy_timeout 24h; #不设置timeout事件
         proxy_pass wxdoracle;
         tcp_nodelay on;
         proxy_buffer_size 1024k;
     }
}
```
重启nginx
```bash
nginx -s reload
```

