---
title: Nginx介绍及配置
date: 2019-05-15 15:10:39
tags: 服务器
keywords:
description:
---
<script type="text/javascript" src="/js/src/bai.js"></script>
## 前言

> Nginx是一款**自由的、开源的、高性能的HTTP服务器和反向代理服务器**；同时也是一个IMAP、POP3、SMTP代理服务器；Nginx可以作为一个HTTP服务器进行网站的发布处理，另外Nginx可以作为**反向代理**进行**负载均衡**的实现。
> **专为性能优化而开发**，性能是其最重要的考量,实现上非常注重效率。它支持内核Poll模型，能经受高负载的考验,有报告表明能支持高达 50,000个并发连接数。
> **具有很高的稳定性。** 其它HTTP服务器，当遇到访问的峰值，或者有人恶意发起慢速连接时，也很可能会导致服务器物理内存耗尽频繁交换，失去响应，只能重启服务器。例如当前apache一旦上到200个以上进程，web响应速度就明显非常缓慢了。而Nginx采取了分阶段资源分配技术，使得它的CPU与内存占用率非常低。nginx官方表示保持10,000个没有活动的连接，它只占2.5M内存，所以类似DOS这样的攻击对nginx来说基本上是毫无用处的。就稳定性而言,nginx比lighthttpd更胜一筹。
> **支持热部署。** 它的启动特别容易, 并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动。你还能够在不间断服务的情况下，对软件版本进行进行升级。



## 特点

### 反向代理
> 说反向代理之前，我们先看看正向代理，正向代理也是大家最常接触的到的代理模式

#### 什么是正向代理？
![](/images/05_ng/0.png)

正向代理最大的特点是客户端非常明确要访问的服务器地址；服务器只清楚请求来自哪个代理服务器，而不清楚来自哪个具体的客户端；正向代理模式屏蔽或者隐藏了真实客户端信息。
正向代理，"它代理的是客户端，代客户端发出请求"，是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。

**正向代理用途：**
（1）访问原来无法访问的资源，类似翻墙操作如在国内访问Google
（2）可以做缓存，加速访问资源
（3）对客户端访问授权，上网进行认证
（4）代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息

#### 什么是反向代理

反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器；并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。
通常的代理服务器，只用于代理内部网络对Internet的连接请求，客户机必须指定代理服务器,并将本来要直接发送到Web服务器上的http请求发送到代理服务器中。当一个代理服务器能够代理外部网络上的主机，访问内部网络时，这种代理服务的方式称为反向代理服务。
        
![](/images/05_ng/1.png)


我们在实际项目操作时，正向代理和反向代理很有可能会存在在一个应用场景中，正向代理代理客户端的请求去访问目标服务器，目标服务器是一个反向单利服务器，反向代理了多台真实的业务处理服务器。具体的拓扑图如下：
![](/images/05_ng/2.png)

### 负载均衡

> 将服务器接收到的请求按照规则分发的过程，称为负载均衡

Nginx作为反向代理服务器的角色，Nginx反向代理服务器接收到的客户端发送的请求数量，就是我们说的负载量。请求数量按照一定的规则进行分发到不同的服务器处理的规则，就是一种均衡规则。

#### Nginx调度算法方式

##### weight轮询(默认，常用)
接收到的请求按照权重分配到不同的后端服务器，即使在使用过程中，某一台后端服务器宕机，Nginx会自动将该服务器剔除出队列，请求受理情况不会受到任何影响。 这种方式下，可以给不同的后端服务器设置一个权重值(weight)，用于调整不同的服务器上请求的分配率；权重数据越大，被分配到请求的几率越大；该权重值，主要是针对实际工作环境中不同的后端服务器硬件配置进行调整的。
```
upstream nodes {

server 192.168.10.1:8668 weight=5;

server 192.168.10.2:8668 weight=10;

}
```
weight和请求数量成正比，主要用于上游服务器配置不均衡的情况。下面的配置中，192.168.10.2机器的请求量是192.168.10.1机器请求量的2倍。
##### ip_hash（常用）
每个请求按照发起客户端的ip的hash结果进行匹配，这样的算法下一个固定ip地址的客户端总会访问到同一个后端服务器，这也在一定程度上解决了集群部署环境下session共享的问题。
```
upstream nodes {
   ip_hash;
   server 192.168.10.1:8668;
   server 192.168.10.2:8668;
}
```



##### fair
智能调整调度算法，动态的根据后端服务器的请求处理到响应的时间进行均衡分配，响应时间短处理效率高的服务器分配到请求的概率高，响应时间长处理效率低的服务器分配到的请求少；结合了前两者的优点的一种调度算法。但是需要注意的是Nginx默认不支持fair算法，如果要使用这种调度算法，请安装upstream_fair模块。

```
upstream nodes {
  server 192.168.10.1:8668;
  server 192.168.10.2:8668;
  fair;
}
```
按上游服务器的响应时间来分配请求。响应时间短的优先分配。


##### url_hash
按照访问的url的hash结果分配请求，每个请求的url会指向后端固定的某个服务器，可以在Nginx作为静态服务器的情况下提高缓存效率。同样要注意Nginx默认不支持这种调度算法，要使用的话需要安装Nginx的hash软件包。
注意：在upstream中加入hash语句。server语句中不能写入weight等其他的參数，hash_method是使用的hash算法。

```
upstream nodes {
      server 192.168.10.1:8668;
      server 192.168.10.2:8668;
      hash $request_uri;
      hash_method crc32;
}
```
#### upstream中常用的配置项

down：表示当前的server不參与负载均衡。
weight：默觉得1，weight越大，负载的权重就越大。
max_fails ：请求失败的次数默觉得1。
fail_timeout : max_fails次失败后，暂停请求此台服务器的时间。
backup： 其他全部的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。
```
upstream nodes {
    ip_hash;server 192.168.10.1:8668 down;
    server 192.168.10.2:8668 weight=2;
    server 192.168.10.3:8668;
    server 192.168.10.4:8668 backup;
}
```

## Nginx命令
### Windows环境

##### 启动：
```
C:\server\nginx-1.0.2>start nginx
```
```
C:\server\nginx-1.0.2>nginx.exe
```
##### 停止：
stop是快速停止nginx；
```
C:\server\nginx-1.0.2>nginx.exe -s stop
```
quit是完整有序的停止nginx，并保存相关信息
```
C:\server\nginx-1.0.2>nginx.exe -s quit
```
##### 重新加载：
当配置信息修改，需要重新载入这些配置时使用此命令
```
C:\server\nginx-1.0.2>nginx.exe -s reload
```
##### 重新打开日志文件：
```
C:\server\nginx-1.0.2>nginx.exe -s reopen
```
##### 查看Nginx版本：
```
C:\server\nginx-1.0.2>nginx -v
```
### Linux环境
##### 启动
```
/usr/local/nginx/sbin/nginx  
```
##### 启动载入当前配置
```
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```
##### 测试配置正确性
```
/usr/local/nginx/sbin/nginx -t
```
##### 重新加载
```
/usr/local/nginx/sbin/nginx -s reload   
```
##### 退出
```
/usr/local/nginx/sbin/nginx -s stop  
```
  
##### 保持未结束的进程后退出
```
/usr/local/nginx/sbin/nginx -s quit  
```
##### 日志重新选择
```
/usr/local/nginx/sbin/nginx -s reopen 
```

##### 查看Nginx版本
```
/usr/local/nginx/sbin/nginx -v
```



## Nginx配置

### 配置文件结构

```
...              #全局块

events {         #events块
   ...
}

http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```


#### 全局块
配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
#### events块
配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
#### http块
可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
#### server块
配置虚拟主机的相关参数，一个http中可以有多个server。
#### location块
配置请求的路由，以及各种页面的处理情况。

### 详细介绍
```

########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页    
     server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
} 
```
#### 默认配置
```

# 用户和用户组
#user www www;
#user nobody;
# nginx进程数，一般设置与CPU核数一致
worker_processes  1;

# 错误日志
#error_log  logs/error.log;
#error_log  logs/error.log  debug;
#error_log  logs/error.log  info;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  warn;
#error_log  logs/error.log  error;
#error_log  logs/error.log  crit;

# 进程文件
#pid        logs/nginx.pid;

# 工作模式连接数上限
events {
    worker_connections  1024; #单个进程最大连接数（最大连接数=连接数*进程数）
}


http {
    include       mime.types; # 文件扩展名与类型映射表
    default_type  application/octet-stream; # 默认文件类型

    # 日志格式设定
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    # 定义本虚拟主机的访问日志
    #access_log  logs/access.log  main;

	# sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    sendfile        on;#开启高效文件传输模式
    #tcp_nopush     on;#防止网络阻塞

    #keepalive_timeout  0;
    keepalive_timeout  65;#长连接超时时间，单位是秒

    #gzip  on; #开启gzip压缩输出 

    server {
        listen       80; # 监听端口
        server_name  localhost; # 域名可以有多个，用空格隔开

        #charset koi8-r; # 默认编码

        #access_log  logs/host.access.log  main; # 访问日志

        location / {
            root   html;
            index  index.html index.htm;
        }

        # 错误页面
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        # 错误页面转发
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html; #根目录
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        # 
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;  #请求转向http://127.0.0.1 定义的服务器列表
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        # FastCGI是解析PHP的处理器
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    # 多个域名映射同一个服务器配置
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server ssl 证书配置
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

### 参考
[https://www.cnblogs.com/knowledgesea/p/5175711.html](https://www.cnblogs.com/knowledgesea/p/5175711.html)
