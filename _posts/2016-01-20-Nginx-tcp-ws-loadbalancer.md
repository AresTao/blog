---
layout: post
title: 用nginx构建TCP/WS负载均衡proxy
keywords: Nginx
category : 架构
tags : [架构]
---
nginx的高性能就不多说了，各大互联网公司都对其有重用。我们常见的是用nginx作为http的proxy，来做一下接入控制和负载分担。现在有个需求是为TCP服务器或者WS服务器做proxy，其实nginx依然可以。

具体操作如下：

###安装pcre库(nginx的依赖库)

{% highlight XML %}
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.34.tar.gz  
# tar -zxvf pcre-8.34.tar.gz  
# cd pcre-8.34  
#./configure  
# make && make install  
# cd ..  
{% endhighlight %}

###下载nginx tcp模块

假设当前目录是/home/test/
{% highlight XML %}

git clone https://github.com/yaoweibin/nginx_tcp_proxy_module.git

patch -p1 < /home/test/nginx_tcp_proxy_module/tcp.patch
{% endhighlight %}

###安装nginx

{% highlight XML %}
# wget http://nginx.org/download/nginx-1.5.9.tar.gz  
# tar -zxvf nginx-1.5.9.tar.gz  
# cd nginx-1.5.9  
#./configure --user=test --group=test --prefix=/usr/local/nginx --with-http_stub_status_module --with-cc-opt='-O3' --with-cpu-opt=opteron --add-module=/home/test/nginx_tcp_proxy_module  
# make && make install  
# cd ..
{% endhighlight %}

###修改配置

sudo vim /usr/local/nginx/conf/nginx.conf

{% highlight XML %}
worker_processes  4;
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid;
events {
    use epoll;
    worker_connections  100000;
}
tcp {
    upstream tcp {
        server ip1:port1;
        server ip2:port2;
        check interval=3000 rise=2 fall=5 timeout=1000;
    }
    
    server {
        listen port;
        server_name ip;
        proxy_pass tcp;
    }
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [<ime_local] "$request" '
    #'$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
                          
    #access_log  logs/access.log  main;
                              
    sendfile        on;
    #tcp_nopush     on;
                                      
    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
                                                  
    upstream http {
        server ip3:port3;
        server ip4:port4;
        least_conn;
    }
    server {
        listen       port;
        server_name  ip/domain;

        location / {
            proxy_pass http://http;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
        }
    }
}
{% endhighlight %}

###启动
  
sudo /usr/local/nginx/sbin/nginx

如果libpcre.so.1报错，修改如下：

{% highlight XML %}
  
  如果是32位系统
  
  [root@lee ~]#  ln -s /usr/local/lib/libpcre.so.1 /lib
  
  如果是64位系统
  
  [root@lee ~]#  ln -s /usr/local/lib/libpcre.so.1 /lib64

{% endhighlight %}
