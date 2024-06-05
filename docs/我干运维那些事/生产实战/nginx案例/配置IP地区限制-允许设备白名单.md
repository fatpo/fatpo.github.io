# 教程

Nginx version 1.14.*

Ubuntu: 18.04

# demo
假设有2个设备ID想加入到IP限制的白名单中：
```text
aaa-bbb-ccc-ddd
aaa-bbb-ccc-eee
```

总的 nginx.conf如下：
```text
upstream myapp{
    server 127.0.0.1:8000 weight=1;
}

server {
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.aabbcc.live/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.aabbcc.live/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    error_page 403 /403.html;
    location  /403.html {
      root /etc/nginx/html/;
      allow all;
    }

    server_name api.aabbcc.live;

    client_max_body_size 20m;


    location /script/ {
        alias /tmp/www/myapp/Client/;
        autoindex_exact_size on;
        autoindex_localtime on;
        charset utf-8,gbk;
    }

    location ~* (app-ads.txt|haha.html)$ {
         root /tmp/www/myapp/static/;
    }
    location ~* \.(js|ico)$ {
        root /tmp/www/myapp/static/;
    }
    location /static/ {
        root /tmp/www/myapp/;
    }

    location ~* ^/appapi/stat/ {
        proxy_set_header        REMOTE_ADDR     $proxy_add_x_forwarded_for;

        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
        add_header Access-Control-Allow-Headers 'DNT, X-Mx-ReqToken, Keep-Alive, User-Agent, X-Requested-With, If-Modified-Since, Cache-Control, Content-Type, Authorization';

        if ($request_method = 'OPTIONS') {
            return 204;
        }

        proxy_set_header   Host             $host:8000;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Cookies          $http_cookies;
        proxy_pass http://myapp;
    }

    location ~* ^/appapi/ {
        proxy_set_header        REMOTE_ADDR     $proxy_add_x_forwarded_for;

        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
        add_header Access-Control-Allow-Headers 'DNT, X-Mx-ReqToken, Keep-Alive, User-Agent, X-Requested-With, If-Modified-Since, Cache-Control, Content-Type, Authorization';

        if ($request_method = 'OPTIONS') {
            return 204;
        }

        set $CHECK "";

        if ($allowed_country = no) {
                set $CHECK CN;
        }
    
            if ($http_x_uuid ~ aaa-bbb-ccc-ddd|aaa-bbb-ccc-eee) {
                set $CHECK "${CHECK}-WHITE";
        }
    
        if ($CHECK = CN) {
                return 403;
        }
        # block country
        #if ($allowed_country = no) {
        #    return 403;
        #}

        proxy_set_header   Host             $host:8000;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Cookies          $http_cookies;
        proxy_pass http://myapp;
    }

}



server {
    if ($host = api.aabbcc.live) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen      80;
    listen [::]:80;


    server_name api.aabbcc.live;
    return 404; # managed by Certbot


}

```