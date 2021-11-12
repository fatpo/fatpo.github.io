# 1、背景
最近在搞那个`logrotate`，其中有一个操作就是需要重启服务，让文件刷新句柄。

其中配置`cat /etc/logrotate.d/myapp`：
```
/root/www/myapp/logs/django.log {
  daily
  rotate 30
  create
  dateext
  dateyesterday
  compress
  delaycompress
  notifempty
  missingok
  sharedscripts
  postrotate
     kill -USR1 $(cat /root/www/myapp/logs/gunicorn.pid)
  endscript
}
```
有一个 `kill`: 
```
kill -USR1 $(cat /root/www/myapp/logs/gunicorn.pid)
```
所以就想知道这个kill怎么才是最佳实践。

# 2、搜索资料

```
kill -HUP pid  pid 是进程标识。如果想要更改配置而不需停止并重新启动服务，请使用该命令。在对配置文件作必要的更改后，发出该命令以动态更新服务配置。

 根据约定，当您发送一个挂起信号（信号 1 或 HUP）时，大多数服务器进程（所有常用的进程）都会进行复位操作并重新加载它们的配置文件。清单 2 显示了向所有正在运行的 Web 服务器进程发送挂起信号的一种方法。
```


```
HUP(1)是让进程挂起，睡眠;

kill (9)六亲不认的杀掉

term(15)正常的退出进程

因为进程可能屏蔽某些信号，所以它们的用处也就不一样。。。
```