---
title: Nginx 负载均衡
date: 2017-05-13 15:17:37
updated:
tags: Nginx
---

# 集群

Nginx 标准 HTTP 模块 [ngx_http_upstream_module](http://nginx.org/en/docs/http/ngx_http_upstream_module.html) 内置了集群和负载均衡功能，使用其中的 `upstream` 配合 `proxy_pass` 指令即可快速实现一个集群：

```
http {
    upstream backend {
        server backend1.example.com       weight=5;
        server 127.0.0.1:8080             max_fails=3 fail_timeout=30s;
        server unix:/tmp/backend3;

        server backup1.example.com:8080   backup;
        server backup2.example.com:8080   down;
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
```

其中 `server` 指令的参数意义如下：

| 参数                | 描述                                 |
| ----------------- | ---------------------------------- |
| weight=number     | 设置服务器的轮询权重，默认为 1。用于后端服务器性能不均的情况。   |
| max_conns=number  | 设置被代理服务器的最大可用并发连接数限制，默认为 0，表示没有限制。 |
| max_fails=number  | 设置最大失败重试次数，默认为 1。设置为 0 表示禁用重试。     |
| fail_timeout=time | 设置失败时间，默认 10 秒。                    |
| backup            | 将服务器标记为备份服务器。当主服务器不可用时，它将被传递请求。    |
| down              | 将服务器标记为永久不可用。                      |

# 负载均衡

负载均衡（Load Balance），其意思就是将运算或存储负载按一定的算法分摊到多个运算或存储单元上，下面介绍 Nginx 三种常见的负载均衡方法：

## ip hash

使用 Nginx `ip_hash` 指令，配置如下：

```
upstream backend {
    ip_hash;

    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
    server backend4.example.com;
}
```

`ip_hash` 指令指定集群使用基于**客户端 IP 地址**的负载均衡方法。 客户端 IPv4 地址的前三个八位字节或整个 IPv6 地址用作哈希键。 该方法确保来自同一客户端的请求将始终传递到同一台服务器，除非此服务器不可用，客户端请求则将被**转发**到另一台服务器（多数情况下，始终是同一台服务器）。
如果其中一台服务器需要临时删除，则应使用 `down` 参数标记，以便保留当前客户端 IP 地址的哈希值。

## 一致性 hash

使用 Nginx `hash` 指令，配置如下：

```
upstream backend {
    hash $remote_addr consistent;

    server backend1.example.com;
    server backend2.example.com;
}
```

`hash` 指令指定集群使用基于**指定 hash 散列键**的负载均衡方法。散列键可以包含文本，变量及其组合。请注意，从集群中添加或删除服务器可能会导致大量键被重新映射到不同的服务器。

解决办法是使用 `consistent` 参数启用  [ketama](http://www.last.fm/user/RJ/journal/2007/04/10/392555/) 一致性 hash 算法。 该算法确保在添加或删除时，只会有少量键被重新映射到不同的服务器。这有助于为缓存服务器实现更高的缓存命中率。

## 会话保持

通过 cookie 实现客户端与后端服务器的会话保持，在一定条件下可以保证同一个客户端访问的都是同一个后端服务器。常见配置如下：

```
upstream backend {
    server backend1.example.com;
    server backend2.example.com;

    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```

下面这份配置在同一域名下有两个 location，分别对应了两组集群服务。为了分别实现会话保持，将 cookie 写入了对应的 path 下，避免 cookie 互相干扰，也减少了数据传输量：

```
http {
    upstream backend1 {
        server backup1.example.com:8080;
        server backup1.example.com:8081;
    	sticky cookie srv_backend1 path=/backend1;
    }

    upstream backend2 {
        server backup2.example.com:8080;
        server backup2.example.com:8081;
    	sticky cookie srv_backend2 path=/backend2;
    }
    
    server {
        server_name example.com;
        listen 80;
    
        location /backend1/ {
            proxy_pass http://backend1;
        }
        
        location /backend2/ {
            proxy_pass http://backend2;
        }
    }
}
```

# 健康检查

健康检查（Health Check）是保障集群可用性的重要手段，有三种常见的健康检查方法：

* 使用社区版 Nginx 的 `max_fails` 和 `fail_timeout` 指令进行被动式检查，详见：《[nginx中健康检查(health_check)机制深入分析](https://segmentfault.com/a/1190000002446630)》；
* 使用[商业版 Nginx Plus](http://nginx.com/products/) 进行主动式检查，缺点是要收费；
* 使用 Nginx 第三方模块编译，例如：[nginx_upstream_check_module](https://github.com/yaoweibin/nginx_upstream_check_module) ；
* 使用 [Tengine](http://tengine.taobao.org/) 内置的[主动式健康检查](http://tengine.taobao.org/document_cn/http_upstream_check_cn.html)功能。

# 常用变量

## $upstream_addr

该模块中很常用的一个变量，用于标识集群中服务器的 IP 和端口。

一般会加入到 Nginx 日志中，用于排查问题来源：

```
log_format  main  '"$http_x_forwarded_for" - "$upstream_addr" - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" $remote_addr $request_time_msec'
```

同时还可以脱敏后加入到响应头中，用于排查问题来源：

```
http {
    map $upstream_addr $short_address {
        ~^\d+\.\d+\.\d+\.(.*) '';
    }
  
    server {
        server_name example.com;
        listen 80;
        
        upstream backend {
            server 127.0.0.1:81;
            server 127.0.0.1:82;
        }
        
        location / {
            add_header X-From $short_address$1;
            proxy_pass http://backend/;
        }
    }
}
```
