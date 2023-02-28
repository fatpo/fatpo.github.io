# 直接上案例
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

# 如何生成证书
## 安装证书工具
```text
sudo apt-get update
sudo apt-get install openssl
```

## 生成自签名证书和密钥
```text
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/server.key -out /etc/nginx/ssl/server.crt
```
在生成证书时，您需要提供一些信息，例如国家、省份、城市、组织和域名等。其中，Common Name 项应该填写您的服务器 IP 地址或您的主机名。

## 将证书和私钥添加到 Nginx 配置文件中
```text
server {
    listen 443 ssl;
    server_name rmdata.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    # 其他配置
}
```
