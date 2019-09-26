(`) 代码... 代码... 代码... (`)

# Nginx简述

**Nginx 是一个开源且高性能、可靠的HTTP中间件，代理服务。**

# Nginx的优势。

```
IO多路复用。**  IO复用解决的就是一个并发问题 **
```

# nginx 日志

```
+ 包括：
    error.log     // 主要记录nginxhttp错误请求的状态
    access_log    // 记录nginx的访问状态，分析每一次访问的请求
+ 实现log配置依赖 — log_format配置
```

# nginx 错误日志输出 tail -f 错误日志地址

# nginx access.log 日志输入 tail -n 200 查看log地址

# nginx 变量

- 将 arg作为参数_之后的参数作为参数值访问
- HTTP请求变量 - arg_PARAMETER
- http_HEADER request请求的header进行输出
- sent_http_HEADER 服务端返回给客户端的header
- 比如记录 User_Agent是： '$http_user_agent'; //以$符号开头 http_ 然后 写入user_agent 将本身的-转为_

# nginx 模块

nginx 官方模块 查看官方支持模块nginx -V 带有--with- 前缀的

- 可以通过这个查询nginx 握手数，链接数 ，活跃数。总的访问数 --with-http_stub_status_module

  ```
        location /mustatus {
            stub_status;
        }
  ```

- 在目录中选择一个随机主页 --with-http_random_index_module

  ```
        location / {
            root /Users/hebei/Desktop/index/rem;
            random_index on;
            # index index.html rem.html;
        }
  ```

- http 内容替换 --with-http_sub_module<br>
  syntax: sub_filter string replacement; default: -; context: http, server, location;

  ```
        location / {
            root /Users/hebei/Desktop/index/rem;
            sub_filter '<h1>helloword' '<h1>HELLOWORD'; # 单个替换
            sub_filter_once off;    # 开启全局替换
        }
  ```

- nginx的请求限制

  ```
        server {
            limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
            #              表示客户端地址          大小    速率：表示每秒一个
            limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;
            location  / {
                root /Users/hebei/Desktop/index/rem;
                # 连接的限制
                limit_conn conn_zone 1;

                # 引用请求限制
                #limit_req zone=req_zone;
                # 对客户端起到一个访问限数的作用，有三个是延时的，其他的直接返回;
                #limit_req zone=req_zone burst=3 nodelay;
            }

        }
  ```

- 压力测试 ab -n 40 -c 20 <http://shan.hebei.com/>

- 访问控制，限制IP访问

  ```
        location ~ ^/admin.html {
            root /Users/hebei/Desktop/index/root;
            # 限制自己的ip不可以访问
            # deny 127.0.0.1;
            # 允许所有的ip访问
            # allow all;

            # 限制只允许自己的ip访问
            allow 127.0.0.1;
            # 不允许其他的ip访问
            deny all ;

            index index.html index.htm;
        }
  ```

- 用户访问控制模块 http_access_module 访问页面的时候需要输入用户名密码 进行访问

  ```
        # 创建用户名，密码  htpasswd -c 文件 用户名
        auth_basic "自己定义的字符串！";
        auth_basic_user_file /usr/local/etc/nginx/auth_conf;
  ```

- 基于用户的信任登陆 http_auth_basic_module

# gzip 开启nignx 压缩

```
    location ~ .*\.(txt|xml)$ {
        gzip on;    # 开启压缩
        gzip_http_version  1.1; # 压缩版本好
        gzip_comp_level 1;  # 压缩倍数
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/gif image/png; # 压缩文件类型

        root /Users/hebei/Desktop/www/doc;
    }
    # 开启下载模式
    # 首先将本地文件进行手动压缩 gzip ./文件地址；生成.gz文件
    location ~ ^/download {
        gzip_static on; # 为on值时，地址栏不需要加压缩之后的gz后缀名
        tcp_nopush on;
        root /Users/hebei/Desktop/www;
    }
```

# nginx 设置缓存

```
    location ~ .*\.(html|htm) {
        expires 24h; # 设置这个字段 创建缓存 cache-control:***
        root /Users/hebei/Desktop/www;
    }
```

# nginx 跨域访问

```
    location ~ .*\.(html|htm) {
        add_header Access-Control-Allow-Origin http://10.240.64.227:8000;
        add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
    }
```

# 防盗链

```
+ 目的：防止资源被盗用
+ 防盗链设置思路
    首要方式：区别哪些请求是非正常的用户请求
+ 基于http_refer放到链配置
    location / {
        # valid_referers 表示允许哪些信息来访问
        # none 表示允许没有带referer信息访问
        # blocked 表示referer信息不是标准的http://方式来请求。允许
        # 允许ip的方式来访问    
        valid_referers none blocked 116.62.103.228;
        if ($invalid_referer) {
            return 403;
        }
    }
```

# nginx 负载均衡

```
    # 请求会轮询这几个服务端口
    # weight=5 轮询的权重，加的越大权重越大
    # down  当前的server不参与负载均衡
    # backup 预留的备份服务器
    # max_fails 允许请求失败的次数
    # fail_timeout 经过max_fails失败后，服务暂停的时间
    # max_conns 限制最大的接收的连接数
    upstream shan {
        ip_hash;    # 每个请求按访问ip的hash结果分配,这样来自同一个ip的固定访问一个后端服务器
        server 127.0.0.1:3333   weight=5;  
        server 127.0.0.1:3334;
        server 127.0.0.1:3335;
    }
    server {
        location / {
            proxy_pass http://shan;
        }
    }
```

# 缓存 proxy_cache

```
    http {
        # proxy_cache_path /usr..... # 缓存存储路径
        # levels=1:2;   # 缓存的目录进行分级，1:2就是2层目录进行分级
        # keys_zone=shan_cache:10m  # 是开辟一个定义缓存空间的名字在下方proxy_path 就是使用这个名字 10m 就是10兆的大小
        # max_size 这个目录最大是多大
        # inactive=60m 表示在60分钟内没有访问到，就会做一个清理
        # use_temp_path=off 用来存放临时文件，一般建议关掉，否则需要新建目录
        proxy_cache_path /usr/local/var/run/nginx/shan_temp levels=1:2 keys_zone=shan_cache:10m max_size=10g inactive=60m use_temp_path=off;
        server {
            # 如果请求的地址中有 (url3|login|register|password\/reset) 这些地址，则赋值$cookie_nocache变量为1;
            if ($request_uri ~ ^/(url3|login|register|password\/reset)) {
                set $cookie_nocache 1;
            }
            location / {
                proxy_cache shan_cache; # 开启缓存空间
                # 表示对200 和304的头信息 是12个小时过期;
                proxy_cache_valid 200 304 12h;
                # 除了200 和 304之外的都是10分钟过期
                proxy_cache_valid any 10m;
                # 改变key对应的参数
                proxy_cache_key $host$uri$is_args$args;

                #
                proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
                proxy_no_cache $http_pragma $http_authorization;


                # 添加默认头信息
                add_header Nginx-Cache "$upstream_cache_status";
                # 如果upstream负载均衡有一台服务器有问题 就跳过访问下一台
                proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
            }
        }
    };
    + 清理指定的url
        方式一：rm -rf 缓存目录
```
