---
title: Nginx小记
date: 2022-04-14 23:41:21
tags: [Linux, Nginx]
categories: Linux
---


# nginx及其衍生版本

- nginx开源版：http://nginx.org
- nginx plus 商业版: https://www.nginx.com
- openresty: http://openresty.org
- tengine: http://tengine.taobao.org

# nginx默认目录结构

- `logs/`：日志目录；
- `conf/`：配置文件目录；
- `html/`：静态资源目录；
- `sbin/`：存放nginx可执行文件；

# nginx基础配置

## nginx最小配置

- `worker_processes` worker 进程数，根据CPU核心数进行配置；

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;
    
    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
}
```

<!-- more -->

## 虚拟主机

通过配置`server_name`可以同时在一台服务器部署多个虚拟主机。通过`server_name`区分不同虚拟主机，甚至可以做到不同web站点服务共享相同端口。

`server_name`支持多种匹配模式：

- 完整匹配（精确匹配）；
- 通配符匹配：如，`*.baidu.com`，`*.baidu.*`等；
- 正则匹配；

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;
    
    # 虚拟主机1
    server {
        listen       80;
    		# 通过配置server_name 可以同时在一台服务器部署多个虚拟主机；
        # 通过server_name区分不同虚拟主机，甚至可以做到不同web站点服务共享相同端口；
        # 注意 vhost1和vhost2都是用80端口！
        server_name  vhost1;  

        location / {
            root   /usr/share/nginx/www/vhost1/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/www/vhost1/html;
        }
    }
  
    # 虚拟主机2
    server {
        listen       80;
        # 通过server_name区分不同虚拟主机，甚至可以做到不同web站点服务共享相同端口；
        # 注意 vhost1和vhost2都是用80端口！
        server_name  vhost2;

        location / {
            root   /usr/share/nginx/www/vhost2/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/www/vhost2/html;
        }
    }
}
```

## 反向代理

`proxy_pass`参数配置

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;
    
    # 虚拟主机1
    server {
        listen       80;
        server_name  vhost1;  

        location / {
            # 配置反向代理
            proxy_pass http://www.baidu.com;
        }

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/www/vhost1/html;
        }
    }
}
```

## 负载均衡

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;
  
    # 定义服务器集群，集群名为httpds
    upstream httpds {
      server 172.17.0.2:80;
      server 172.17.0.3:80;
    }
    
    # 虚拟主机1
    server {
        listen       80;
        server_name  vhost1;  

        location / {
            # 配置反向代理
            proxy_pass http://httpds;
        }

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/www/vhost1/html;
        }
    }
}
```

### 权重

```
version: "3"
services:
  ngx:
    image: nginx:1.21.6
    container_name: ngx
    ports:
      - 80:80
    volumes:
      - ./www:/usr/share/nginx/www:ro
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./logs:/var/log/nginx:rw
    depends_on:
      - ng1
      - ng2
      - ng3
    networks:
      - ngx_net
  ng1:
    image: nginx:1.21.6
    container_name: ng1
    ports:
       - 8001:80
    volumes:
       - ./www/ng1/html:/usr/share/nginx/html:ro
    networks:
      - ngx_net
  ng2:
    image: nginx:1.21.6
    container_name: ng2
    ports:
      - 8002:80
    volumes:
       - ./www/ng2/html:/usr/share/nginx/html:ro
    networks:
      - ngx_net
  ng3:
    image: nginx:1.21.6
    container_name: ng3
    ports:
      - 8003:80
    volumes:
      - ./www/ng3/html:/usr/share/nginx/html:ro
    networks:
      - ngx_net
networks:
  ngx_net:
```

weight设置权重：

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;
  
    # 定义服务器集群，集群名为httpds
    upstream httpds {
      # weight设置权重
      server ng1:80 weight=7;
      server ng2:80 weight=2;
      server ng3:80 weight=1;
    }
    
    # 虚拟主机1
    server {
        listen       80;
        server_name  localhost;  

        location / {
            # 配置反向代理
            proxy_pass http://httpds;
        }

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/www/vhost1/html;
        }
    }
}
```

down参数指定服务器不参与负载，backup参数代表默认不启用，其他服务器down了才启用：

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;
  
    # 定义服务器集群，集群名为httpds
    upstream httpds {
      # weight设置权重
      server ng1:80 weight=7;
      server ng2:80 weight=2;
      server ng3:80 weight=1 backup;
    }
    
    # 虚拟主机1
    server {
        listen       80;
        server_name  localhost;  

        location / {
            # 配置反向代理
            proxy_pass http://httpds;
        }

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/www/vhost1/html;
        }
    }
}
```

## 动静分离

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;
  
    # 定义服务器集群，集群名为httpds
    upstream httpds {
      # weight设置权重
      server ng1:80 weight=7;
      server ng2:80 weight=2;
      server ng3:80 weight=1 backup;
    }
    
    # 虚拟主机1
    server {
        listen       80;
        server_name  localhost;  

        location / {
            # 配置反向代理
            proxy_pass http://httpds;
        }
     
        location /img {
					root html;
           index index.html index.htm;
        }
    
    		location /js {
					root html;
           index index.html index.htm;
        }
    
    		location /css {
					root html;
           index index.html index.htm;
        }

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/www/vhost1/html;
        }
    }
}
```

### location 正则参数

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;
  
    # 定义服务器集群，集群名为httpds
    upstream httpds {
      # weight设置权重
      server ng1:80 weight=7;
      server ng2:80 weight=2;
      server ng3:80 weight=1 backup;
    }
    
    # 虚拟主机1
    server {
        listen       80;
        server_name  localhost;  

        location / {
            # 配置反向代理
            proxy_pass http://httpds;
        }
     
    		# A regular expression is preceded with the tilde (~) for case-sensitive matching,
        # or the tilde-asterisk (~*) for case-insensitive matching. 
        location ~*/(img|js|css) {
					root html;
           index index.html index.htm;
        }

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/www/vhost1/html;
        }
    }
}
```

### URL rewrite

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;

    keepalive_timeout  65;
  
    # 定义服务器集群，集群名为httpds
    upstream httpds {
      # weight设置权重
      server ng1:80 weight=7;
      server ng2:80 weight=2;
      server ng3:80 weight=1 backup;
    }
    
    # 虚拟主机1
    server {
        listen       80;
        server_name  localhost;  

        location / {
            # 将/2.html 重写为 /index.jsp?pageNum=2
            rewrite ^/2.html$ /index.jsp?pageNum=2 break;
            # 配置反向代理
            proxy_pass http://httpds;
        }
     
    		# A regular expression is preceded with the tilde (~) for case-sensitive matching,
        # or the tilde-asterisk (~*) for case-insensitive matching. 
        location ~*/(img|js|css) {
					root html;
           index index.html index.htm;
        }

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/www/vhost1/html;
        }
    }
}
```

