# 1、背景
2021年10月01日，凌晨四点多，我的服务器被黑了，损失了800美金。

在这里感恩下我的宝宝，他的哭声让我惊醒，否则我可能要损失几千美金。

更多细节可以看国哥另外一篇，[传送门](https://fatpo.github.io/#/我干运维那些事/django服务器IP限流的最佳实践)

# 2、nginx黑名单
版本号：
```
# nginx -v
nginx version: nginx/1.14.0 (Ubuntu)
```
在 `/etc/nginx/ip.black`下写入：
```dtd
# cat /etc/nginx/ip.black
deny 45.130.97.193;
deny 206.62.4.242;
deny 206.62.22.156;
deny 206.62.27.10;
deny 206.62.22.50;
deny 68.114.159.20;
```
这些IP全是坏人来的，一分钟请求大几十次的，我也不`打码`了。

在`/etc/nginx/nginx.conf`的http结构体下增加配置：
```dtd
http {
    #黑名单
    include ip.black;
}
```

重启下：
```dtd
nginx -t
nginx -s reload
```
# 3、参考

* [CSDN: Nginx基础配置之设置IP黑名单](https://blog.csdn.net/snow____man/article/details/83545922)
