# Nginx笔记
> wvv 20191216

配置反向代理服务器，实现不同协议共用端口，初步实现。

```nginx
#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;
	map $http_upgrade $connection_upgrade {
		default upgrade;
		'' close;
	}
	upstream websocket {
		server 127.0.0.1:61614;
	}
	upstream webserver {
		server 127.0.0.1:3000;
	}
    server {
        listen       80;
        server_name  localhost;
        access_log  logs/host.access.log  main;
		location / {  
			root /var/www/web/;
			index  index.html index.htm;
		}

        location /login {
            proxy_pass http://webserver/login;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
		location /ws {
			proxy_pass http://websocket;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
		}
    }
}
```