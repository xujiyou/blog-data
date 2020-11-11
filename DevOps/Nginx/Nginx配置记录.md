---
title: Nginx配置记录
date: 2020-06-16 15:07:57
tags:
---

记录一些用过的 Nginx 配置。

---

## Nginx 代理 Vue.js 静态网站

```
   server {
        listen       80;
        server_name     boot.serrhub.com;
        location / {
            root /opt/source/serrhub-front/dist;
            index index.html;
            add_header Access-Control-Allow-Origin *;
            try_files $uri $uri/ /index.html;
        }
    }
```

## Nginx 代理 Hexo 静态网站

```
    server {
        listen       443 ssl;
        server_name  xujiyou.work www.xujiyou.work;

        ssl_certificate "/etc/nginx/conf.d/cret/xujiyou-work-nginx-1214113354/xujiyou.work_chain.crt";
        ssl_certificate_key "/etc/nginx/conf.d/cret/xujiyou-work-nginx-1214113354/xujiyou.work_key.key";
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        location /README/index.html {
            root /opt/public/;
            index index.html;
            add_header Access-Control-Allow-Origin *;
        }

        location ^~/README/ {
            proxy_set_header Host $host;
            proxy_set_header  X-Real-IP        $remote_addr;
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_set_header X-NginX-Proxy true;
            #rewrite ^/README/(.*)$ /$1 break;
            if ($request_uri ~* \.md$) {
               rewrite ^/(.*)\.md$ /$1 break;
            }
            proxy_pass http://README/;
        }

        location ~ ^/.*/resource/.*$ {
            rewrite ^/(.*)/resource/(.*)$ https://xujiyou.work/resource/$2 break;
        }

        location / {
            root /opt/public/;
            index index.html;
            add_header Access-Control-Allow-Origin *;
           # try_files $uri $uri/ /index.html;
            if ( $request_uri = "/" ) {
                rewrite "/" https://xujiyou.work/README/index.html break;
            }
        }


        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

## Nginx 代理 Java 服务

```
    server {
        listen       443 ssl;
        server_name  boot.xujiyou.work;

        ssl_certificate "/etc/nginx/conf.d/cret/boot-xujiyou-work-nginx-1216114456/boot.xujiyou.work_chain.crt";
        ssl_certificate_key "/etc/nginx/conf.d/cret/boot-xujiyou-work-nginx-1216114456/boot.xujiyou.work_key.key";
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        location / {
            proxy_pass http://127.0.0.1:8080;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
```



## 设置上传大小

```
server {
        listen        80;
        server_name    www.S1.com;
        client_max_body_size 30M;

        location /api/ {
            proxy_pass http://127.0.0.1:8891/;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_send_timeout 3600s;
            proxy_read_timeout 3600s;
            proxy_connect_timeout 60s;
        }

        location / {
            root /opt/s1/www.S1.com;
            index index.html;
            add_header Access-Control-Allow-Origin *;
            try_files $uri $uri/ /index.html;
        }
    }
```

如果 proxy_pass的结尾有`/`， 则会把`/api/*`后面的路径直接拼接到后面，即移除api.

## 404 重写

```nginx
        location / {
            root   /data1/imgs;
            autoindex on;

            if ($request_uri ~* ^/all) {
                error_page 404 =200 @test;
            }

            if ($request_uri ~* ^/test) {
                error_page 404 =200 @minio;
            }

        }

        location @test {
            rewrite ^/all/(.*)$ /test/$1 permanent;
        }

        location @minio {
            rewrite ^/test/(.*)$ http://192.168.6.124:29000/test/$1 permanent;
        }
```



## 跨域

```nginx
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Headers x-token,Token,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization;
    add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
```



## Nginx 配置 WebSocket

```
        location ~*  ^/monitor/ws/.* {
            rewrite ^/monitor(.*)$ $1 break;
            add_header Access-Control-Allow-Origin *;
            proxy_pass http://127.0.0.1:8003;
            proxy_set_header Host $http_host;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
```

