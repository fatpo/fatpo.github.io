直接上案例：
```
# 配置 HTTP 重定向到 HTTPS
server {
    listen 80;
    server_name rmdata.com;
    return 301 https://$server_name$request_uri;
}

# 配置 HTTPS
server {
    listen 443 ssl;
    server_name rmdata.com;

    # SSL 证书和密钥
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    # 配置 SSL 安全协议和密码套件
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";

    # 配置前端静态文件服务
    location / {
        root /var/www/html/build;
        index index.html;

        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
        add_header Access-Control-Allow-Headers "Authorization,Content-Type,Accept";
        if ($request_method = 'OPTIONS') {
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
            add_header Access-Control-Allow-Headers "Authorization,Content-Type,Accept";
            return 204;
        }
    }

    # 配置后台服务代理
    location /api {
        proxy_pass http://127.0.0.1:8090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
        add_header Access-Control-Allow-Headers "Authorization,Content-Type,Accept";
        if ($request_method = 'OPTIONS') {
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
            add_header Access-Control-Allow-Headers "Authorization,Content-Type,Accept";
            return 204;
        }
    }
}

```