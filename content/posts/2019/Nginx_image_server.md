+++
title = "Nginx 搭建图片服务器及緩存設置"
date = 2019-08-09T23:13:00+08:00
lastmod = 2022-04-10T01:49:18+08:00
tags = ["Nginx"]
categories = ["Nginx"]
draft = false
toc = true
+++

## Nginx 搭建图片服务器及緩存設置 {#nginx-搭建图片服务器及緩存設置}

-   2019-08-09 23:13:04


### 准备工作 {#准备工作}

安装 Nginx 依赖

```sh
yum -y install pcre-devel openssl openssl-devel
```

如果不安装在执行=./configure=时会出现=./configure: error: the HTTP rewrite module requires the PCRE library.=错误


### 安装 Nginx {#安装-nginx}

1.  下载 Nginx 安装包 [Nginx
    官网下载地址](https://nginx.org/en/download.html)

2.  解压 Nginx

<!--listend-->

```sh
  tar -zxvf nginx-1.14.2.tar.gz
  cd nginx-1.14.2
  ./configure
  make && make install
```

在配置/编译/安装过程中可能会出现缺少依赖的问题,
这个根据个人遇到的问题自行 Google 吧,这里就不一一介绍了


### 修改配置 {#修改配置}


#### 修改工作进程数配置 {#修改工作进程数配置}

一般根据 CPU 核心数设置, CPU 有几个核心就设置成几, 比如我的服务器有 8
个核心就设置为 8

```text
  worker_processes  8;
```


#### 修改 events 模块中包含 nginx 中所有处理连接的设置 {#修改-events-模块中包含-nginx-中所有处理连接的设置}

常用配置项如下:

```text
  events{
    use epoll;
    worker_connections 20000;
  }
```

-   详细说明 use epoll; 使用 epoll 的 I/O 模型(值得注意的是如果你不知道
    Nginx 该使用哪种轮询方法的话，它会选择一个最适合你操作系统的)

-   补充说明:

    与 apache 相类，nginx 针对不同的操作系统，有不同的事件模型

    1.  标准事件模型 Select、poll
        属于标准事件模型，如果当前系统不存在更有效的方法，nginx 会选择
        select 或 poll
    2.  高效事件模型 Kqueue：使用于 FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0
        和 MacOS X.使用双处理器的 MacOS X 系统使用 kqueue
        可能会造成内核崩溃。 Epoll:使用于 Linux 内核 2.6 版本及以后的系统。
        /dev/poll：使用于 Solaris 7 11/99+, HP/UX 11.22+ (eventport), IRIX
        6.5.15+ 和 Tru64 UNIX 5.1A+。 Eventport：使用于 Solaris 10.
        为了防止出现内核崩溃的问题， 有必要安装安全补丁

    查看 linux 版本号可以使用 cat /proc/version 命令

    ```sh
        cat /proc/version
    ```

输出如下:

```text
Linux version 2.6.32-431.el6.x86_64 (mockbuild@c6b8.bsys.dev.centos.org) (gcc version 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC) ) #1 SMP Fri Nov 22 03:15:09 UTC 2013
```

worker_connections 2000;

工作进程的最大连接数量 理论上每台 nginx 服务器的最大连接数为
worker_processes\*worker_connections worker_processes 为我们再 main
中开启的进程数


#### 修改 Http 配置 {#修改-http-配置}

```text
    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    proxy_connect_timeout 10;
    proxy_read_timeout 180;
    proxy_send_timeout 5;
    proxy_buffer_size 16k;
    proxy_buffers 4 32k;
    proxy_busy_buffers_size 96k;
    proxy_temp_file_write_size 96k;
    proxy_temp_path /tmp/temp_dir;
    proxy_cache_path /tmp/cache levels=1:2 keys_zone=cache_one:100m inactive=1d max_size=40g;
```


#### 修改 Server 配置 {#修改-server-配置}

```text
server {
        listen  90;
        server_name localhost;
        root /usr/local/nginx/html;

        location / {

        }
    }

    #此Server负责内网访问
    server{
     listen        80;
       server_name  xxx.example.com;
       root /usr/local/nginx/html/;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            #index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
     location ~ .*\.(gif|jpg|png|css|js)(.*) {
                proxy_pass http://192.168.xxx.xxx:xx;
                proxy_redirect off;
                proxy_set_header Host $host:xxxx;
                proxy_cache cache_one;
                proxy_cache_valid 200 302 24h;
                proxy_cache_valid 301 30d;
                proxy_cache_valid any 5m;
                expires 90d;
                add_header wall  "It's been cached!!!";
        }
      error_page   500 502 503 504  /50x.html;
               location = /50x.html {
               root   html;
        }

    }

    # 此Server负责内网访问
    server {
        listen       80;
        server_name  localhost;
        root /usr/local/nginx/html/;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            #index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        # 这个配置是重点: 配置缓存的时间及 wall
       location ~ .*\.(gif|jpg|png|css|js)(.*) {
                proxy_pass http://192.168.xxx.xxx:xx;
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_cache cache_one;
                proxy_cache_valid 200 302 24h;
                proxy_cache_valid 301 30d;
                proxy_cache_valid any 5m;
                expires 90d;
                add_header wall  "It's been cached!!!";
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
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
```

至此,Nginx 图片缓存服务器就已经配置完毕了,现在你可以启动 Nginx
测试是否成功; 测试方法:
访问一张存在的图片,只后将图片改名或者删除,再次刷新页面,如果仍然可以请求到图片,则证明成功


### 附录 {#附录}


#### 附录 1: 完整配置文件 {#附录-1-完整配置文件}

```text
#user  nobody;
worker_processes  8;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    proxy_connect_timeout 10;
    proxy_read_timeout 180;
    proxy_send_timeout 5;
    proxy_buffer_size 16k;
    proxy_buffers 4 32k;
    proxy_busy_buffers_size 96k;
    proxy_temp_file_write_size 96k;
    proxy_temp_path /tmp/temp_dir;
    proxy_cache_path /tmp/cache levels=1:2 keys_zone=cache_one:100m inactive=1d max_size=40g;

    server {
        listen  90;
        server_name localhost;
        root /usr/local/nginx/html;

        location / {

        }
    }


    server{
       listen        80;
       server_name  xxxx.example.com;
       root /usr/local/nginx/html/;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            #index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
     location ~ .*\.(gif|jpg|png|css|js)(.*) {
                proxy_pass http://192.168.x.xxx:xx;
                proxy_redirect off;
                proxy_set_header Host $host:xxxx;
                proxy_cache cache_one;
                proxy_cache_valid 200 302 24h;
                proxy_cache_valid 301 30d;
                proxy_cache_valid any 5m;
                expires 90d;
                add_header wall  "It's been cached!!!";
        }
      error_page   500 502 503 504  /50x.html;
               location = /50x.html {
               root   html;
        }

    }

    server {
        listen       80;
        server_name  localhost;
     root /usr/local/nginx/html/;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            #index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
     location ~ .*\.(gif|jpg|png|css|js)(.*) {
                proxy_pass http://192.168.x.xxx:xx;
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_cache cache_one;
                proxy_cache_valid 200 302 24h;
                proxy_cache_valid 301 30d;
                proxy_cache_valid any 5m;
                expires 90d;
                add_header wall  "It's been cached!!!";
        }


        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
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
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
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


### 附录 2 nginx.conf 配置文件详解 {#附录-2-nginx.conf-配置文件详解}


#### nginx.conf 配置结构 {#nginx.conf-配置结构}

```text
... #全局块
events { #events块
...
}

http #http块
{
    ... #http全局块

    server #server块
    {
        ... #server全局块
        location [PATTERN] #location块
        {
            ...
        }
        location [PATTERN]
        {
            ...
        }每个指令必须有分号结束
     }

    server
    {
        ...
    }

    ... #http全局块
}
```

1.  main 全局块：配置影响 nginx 全局的指令。一般有运行 nginx
    服务器的用户组，nginx 进程 pid
    存放路径，日志存放路径，配置文件引入，允许生成 worker process 数等。
2.  events 块：配置影响 nginx
    服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
3.  http 块：可以嵌套多个
    server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type
    定义，日志自定义，是否使用 sendfile
    传输文件，连接超时时间，单连接请求数等。
4.  server 块：配置虚拟主机的相关参数，一个 http 中可以有多个 server。
5.  location 块：配置请求的路由，以及各种页面的处理情况。

不同模块指令关系：server 继承 main；location 继承 server；upstream
既不会继承指令也不会被继承，它有自己的特殊指令，不需要在其他地方的应用


### 附录 3:nginx.conf 基本配置模板 {#附录-3nginx.conf-基本配置模板}

每个指令必须有分号结束

```text
  ########### 每个指令必须有分号结束。#################

  #配置用户或者组，默认为nobody nobody。
  #user administrator administrators;
  #允许生成的进程数，默认为1
  #worker_processes 2;
  #指定nginx进程运行文件存放地址
  #pid /nginx/pid/nginx.pid;
  #制定错误日志路径，级别。这个设置可以放入全局块，http块，server块，级别依次为：debug|info|notice|warn|error|crit|alert|emerg
  error_log log/error.log debug;

  #工作模式及连接数上限
  events {
  #设置网路连接序列化，防止惊群现象发生，默认为on
    accept_mutex on;
  #设置一个进程是否同时接受多个网络连接，默认为off
    multi_accept on;
  #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
  #use epoll;
  #单个work进程允许的最大连接数，默认为512
    worker_connections 1024;
  }

  #http服务器
  http {
  #文件扩展名与文件类型映射表。设定mime类型(邮件支持类型),类型由mime.types文件定义
  #include /usr/local/etc/nginx/conf/mime.types;
    include mime.types;
  #默认文件类型，默认为text/plain
    default_type application/octet-stream;

  #取消服务访问日志
  #access_log off;
  #自定义日志格式
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for';
  #设置访问日志路径和格式。"log/"该路径为nginx日志的相对路径，mac下是/usr/local/var/log/。combined为日志格式的默认值
    access_log log/access.log myFormat;
    rewrite_log on;

  #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。（sendfile系统调用不需要将数据拷贝或者映射到应用程序地址空间中去）
    sendfile on;
  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    sendfile_max_chunk 100k;

  #连接超时时间，默认为75s，可以在http，server，location块。
    keepalive_timeout 65;

  #gzip压缩开关
  #gzip on;

    tcp_nodelay on;

  #设定实际的服务器列表
    upstream mysvr1 {
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup; #热备(其它所有的非backup机器down或者忙的时候，请求backup机器))
    }
    upstream mysvr2 {
  #weigth参数表示权值，权值越高被分配到的几率越大
      server 192.168.1.11:80 weight=5;
      server 192.168.1.12:80 weight=1;
      server 192.168.1.13:80 weight=6;
    }
    upstream https-svr {
  #每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题
      ip_hash;
      server 192.168.1.11:90;
      server 192.168.1.12:90;
    }

  #error_page 404 https://www.baidu.com; #错误页

  #HTTP服务器

  # 静态资源一般放在nginx所在主机
    server {
        listen 80; #监听HTTP端口
        server_name 127.0.0.1; #监听地址
        keepalive_requests 120; #单连接请求上限次数
        set $doc_root_dir "/Users/doing/IdeaProjects/edu-front-2.0"; #设置server里全局变量
        #index index.html;  #定义首页索引文件的名称
        location ~*^.+$ { #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
            root $doc_root_dir; #静态资源根目录
            proxy_pass http://mysvr1; #请求转向“mysvr1”定义的服务器列表
            #deny 127.0.0.1; #拒绝的ip
            #allow 172.18.5.54; #允许的ip
        }
    }

  #http
    server {
        listen 80;
        server_name www.helloworld.com; #监听基于域名的虚拟主机。可有多个，可以使用正则表达式和通配符
        charset utf-8; #编码格式
        set $static_root_dir "/Users/doing/static";
        location /app1 { #反向代理的路径（和upstream绑定），location后面设置映射的路径
            proxy_pass http://zp_server1;
        }
        location /app2 {
            proxy_pass http://zp_server2;
        }
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {  #静态文件，nginx自己处理
            root $static_root_dir;
            expires 30d; #静态资源过时间30天
        }
        location ~ /\.ht {  #禁止访问 .htxxx 文件
            deny all;
        }
        location = /do_not_delete.html { #直接简单粗暴的返回状态码及内容文本
            return 200 "hello.";
        }

  # 指定某些路径使用https访问(使用正则表达式匹配路径+重写uri路径)
        location ~* /http* { #路径匹配规则：如localhost/http、localhost/httpsss等等
  #rewrite只能对域名后边的除去传递的参数外的字符串起作用，例如www.c.com/proxy/html/api/msg?method=1&para=2只能对/proxy/html/api/msg重写。
  #rewrite 规则 定向路径 重写类型;
  #rewrite后面的参数是一个简单的正则。$1代表正则中的第一个()。
  #$host是nginx内置全局变量，代表请求的主机名
  #重写规则permanent表示返回301永久重定向
            rewrite ^/(.*)$ https://$host/$1 permanent;
        }

  #错误处理页面（可选择性配置）
  #error_page 404 /404.html;
  #error_page 500 502 503 504 /50x.html;

  #以下是一些反向代理的配置(可选择性配置)
  #proxy_redirect off;
  #proxy_set_header Host $host; #proxy_set_header用于设置发送到后端服务器的request的请求头
  #proxy_set_header X-Real-IP $remote_addr;
  #proxy_set_header X-Forwarded-For $remote_addr; #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
  #proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
  #proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
  #proxy_read_timeout 90; #连接成功后，后端服务器响应时间(代理接收超时)
  #proxy_buffer_size 4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
  #proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
  #proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
  #proxy_temp_file_write_size 64k; #设定缓存文件夹大小，大于这个值，将从upstream服务器传

  #client_max_body_size 10m; #允许客户端请求的最大单文件字节数
  #client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数


    }

  #https
  #(1)HTTPS的固定端口号是443，不同于HTTP的80端口；
  #(2)SSL标准需要引入安全证书，所以在 nginx.conf 中你需要指定证书和它对应的 key
    server {
      listen 443;
      server_name  www.hellohttps1.com www.hellohttps2.com;
      set $geek_web_root "/Users/doing/IdeaProjects/backend-geek-web";
      ssl_certificate      /usr/local/etc/nginx/ssl-key/ssl.crt; #ssl证书文件位置(常见证书文件格式为：crt/pem)
      ssl_certificate_key  /usr/local/etc/nginx/ssl-key/ssl.key; #ssl证书key位置
      location /passport {
        send_timeout 90;
        proxy_connect_timeout 50;
        proxy_send_timeout 90;
        proxy_read_timeout 90;
        proxy_pass http://https-svr;
      }
      location ~ ^/(res|lib)/ {
          root $geek_web_root;
          expires 7d;
  #add_header用于为后端服务器返回的response添加请求头，这里通过add_header实现CROS跨域请求服务器
          add_header Access-Control-Allow-Origin *;
      }
  #ssl配置参数（选择性配置）
      ssl_session_cache shared:SSL:1m;
      ssl_session_timeout 5m;
    }

  #配置访问控制：每个IP一秒钟只处理一个请求，超出的请求会被delayed
  #语法：limit_req_zone  $session_variable  zone=name:size  rate=rate (为session会话状态分配一个大小为size的内存存储区，限制了每秒（分、小时）只接受rate个IP的频率)
    limit_req_zone  $binary_remote_addr zone=req_one:10m   rate=1r/s nodelay;
    location /pay {
          proxy_set_header Host $http_host;
          proxy_set_header X-Real_IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  #访问控制：limit_req zone=name [burst=number] [nodelay];
          limit_req zone=req_one burst=5; #burst=5表示超出的请求(被delayed)如果超过5个，那些请求会被终止（默认返回503）
          proxy_pass http://mysvr1;
    }

  #可以把子配置文件放到/usr/local/etc/nginx/servers/路径下，通过include引入
    include /usr/local/etc/nginx/servers/*.conf;

  }
```


#### 内置全局变量 {#内置全局变量}

```text
  $args ：这个变量等于请求行中的参数，同$query_string
  $content_length ： 请求头中的Content-length字段。
  $content_type ： 请求头中的Content-Type字段。
  $document_root ： 当前请求在root指令中指定的值。
  $host ： 请求主机头字段，否则为服务器名称。
  $http_user_agent ： 客户端agent信息
  $http_cookie ： 客户端cookie信息
  $limit_rate ： 这个变量可以限制连接速率。
  $request_method ： 客户端请求的动作，通常为GET或POST。
  $remote_addr ： 客户端的IP地址。
  $remote_port ： 客户端的端口。
  $remote_user ： 已经经过Auth Basic Module验证的用户名。
  $request_filename ： 当前请求的文件路径，由root或alias指令与URI请求生成。
  $scheme ： HTTP方法（如http，https）。
  $server_protocol ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
  $server_addr ： 服务器地址，在完成一次系统调用后可以确定这个值。
  $server_name ： 服务器名称。
  $server_port ： 请求到达服务器的端口号。
  $request_uri ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
  $uri ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
  $document_uri ： 与$uri相同。
```


### 参考链接 {#参考链接}

附录模块转载自: [nginx
快速入门之配置篇](https://zhuanlan.zhihu.com/p/31202053)
[Nginx 配置详解](http://www.cnblogs.com/knowledgesea/p/5175711.html)
[Nginx 简易教程](http://www.cnblogs.com/jingmoxukong/p/5945200.html)
[Nginx 配置总结](http://blog.p2hp.com/archives/4493)
