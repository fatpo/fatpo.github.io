# 教程

版本： Nginx version 1.14.*

[https://www.cloudbooklet.com/how-to-block-website-visitors-from-certain-countries-using-nginx-geoip/](https://www.cloudbooklet.com/how-to-block-website-visitors-from-certain-countries-using-nginx-geoip/)


```text
sudo apt install libnginx-mod-http-geoip geoip-database libgeoip1

sudo vi /etc/nginx/conf.d/haha.conf

# 在server中添加
geoip_country /usr/share/GeoIP/GeoIP.dat;
map $geoip_country_code $allow_visit {
    default yes;
    US no;
    #You can add additional countries
}
```

需要两个数据集：
```text
/usr/share/GeoIP/GeoIP.dat;
/usr/share/GeoIP/GeoIPCity.dat;
```
我打包好放在： [Geo.zip](https://github.com/fatpo/fatpo.github.io/blob/docsify-push/docs/%E6%88%91%E5%B9%B2%E8%BF%90%E7%BB%B4%E9%82%A3%E4%BA%9B%E4%BA%8B/%E7%94%9F%E4%BA%A7%E5%AE%9E%E6%88%98/nginx%E6%A1%88%E4%BE%8B/Geo.zip)

自己上传到上述目录。

# 完整demo
/etc/nginx/conf.d/haha.conf 的完整配置如下：
```text
upstream myapp{
    server 127.0.0.1:8000 weight=1;
}

server {
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.haha.xyz/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.haha.xyz/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    # block the country
    if ($allowed_country = no) {
        return 403;
    }
    error_page 403 /403.html;
    location  /403.html {
      root /etc/nginx/html/; 
      allow all;  
    }	


    server_name api.haha.xyz;

    client_max_body_size 20m;

    
    location ~* (app-ads.txt|haha1.html|haha2.html|haha3.html)$ {
         root /data/haha/static/;
    }
    location /static/ {
        root /data/haha/;
    }

    location ~* ^/haha/ {
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



server {
    if ($host = api.haha.xyz) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen      80;
    listen [::]:80;	


    server_name api.haha.xyz;
    return 404; # managed by Certbot


}

```

/etc/nginx/nginx.conf 的完整配置如下：
```text
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##
    
        # geoip
        geoip_country /usr/share/GeoIP/GeoIP.dat;
        geoip_city /usr/share/GeoIP/GeoIPCity.dat;
      
        # geoip - map the list of denied countries
        map $geoip_country_code $allowed_country {
          default yes;
          # China
          CN no;
        }


	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}

```