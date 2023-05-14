# 教程

版本： Nginx version 1.14.*

[https://www.cloudbooklet.com/how-to-block-website-visitors-from-certain-countries-using-nginx-geoip/](https://www.cloudbooklet.com/how-to-block-website-visitors-from-certain-countries-using-nginx-geoip/)


```text
sudo apt install libnginx-mod-http-geoip geoip-database libgeoip1

sudo vi /etc/nginx/conf.d/xx.conf

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
我打包好放在： [Geo.zip](./Geo.zip)

