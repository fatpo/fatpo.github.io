## 背景

nginx版本：

```text
nginx version: nginx/1.18.0 (Ubuntu)
```

ubuntu 版本：

```text
18.04
```

突然ssl证书失效了，很奇怪啊，因为我一直有自动更新的，crontab -l:

```text
49 0 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
```

感觉这个acme不是很靠谱，还是用更加可靠的`certbot`来处理。

## 教程

卸载 acme:

```text
/root/.acme.sh/acme.sh --uninstall
```

安装certbot:

```text
sudo apt-get update
sudo apt-get install certbot
```

安装certbot nginx插件：

```text
sudo apt-get install python3-certbot-nginx
```

获取证书：

```text
sudo certbot certonly --nginx -d your_domain.com
```

拿到对应的路径：

```text
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/haha.abc.xyz/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/haha.abc.xyz/privkey.pem
   Your cert will expire on 2024-02-16. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

路径：

```text
/etc/letsencrypt/live/haha.abc.xyz/fullchain.pem
/etc/letsencrypt/live/haha.abc.xyz/privkey.pem
```

修改对应domain的nginx.conf：

```text
server {
    listen      80;
    listen [::]:80;
    listen 443 ssl;

    server_name haha.abc.xyz;

    ssl_certificate    /etc/letsencrypt/live/haha.abc.xyz/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/haha.abc.xyz/privkey.pem;
    
    #ssl_certificate   /root/.acme.sh/haha.abc.xyz_ecc/fullchain.cer;
    #ssl_certificate_key  /root/.acme.sh/haha.abc.xyz_ecc/haha.abc.xyz.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    client_max_body_size 20m;

    location ~* \.(html|gif|jpg|jpeg|png|css|js|ico)$ {
        root /data/www/MyBackEnd/static/;
    }
    location /static/ {
        root /data/www/MyBackEnd/;
    }

    location ~* ^/(GGUser)/ {
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
}

```

重启nginx：

```text
nginx -t
nginx -s reload
```

定时刷新`crontab -e` :

```text
# 每天的0:00和12:00
0 0,12 * * * certbot renew
```
