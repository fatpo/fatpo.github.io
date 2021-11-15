# 1、背景
操作系统：
```
root@fatpo:/var/log/nginx# cat /etc/issue
Ubuntu 18.04.5 LTS \n \l
```
默认带了一个logrotate:
```
root@fatpo:/var/log/nginx# ll /etc/logrotate.d/
total 52
drwxr-xr-x   2 root root 4096 Oct 21 12:45 ./
drwxr-xr-x 103 root root 4096 Oct 11 06:13 ../
-rw-r--r--   1 root root  120 Nov  2  2017 alternatives
-rw-r--r--   1 root root  126 Nov 20  2017 apport
-rw-r--r--   1 root root  173 Apr 20  2018 apt
-rw-r--r--   1 root root  112 Nov  2  2017 dpkg
-rw-r--r--   1 root root  146 Nov 23  2018 lxd
-rw-r--r--   1 root root  845 Jan 12  2018 mysql-server
-rw-r--r--   1 root root  375 Oct 21 12:45 nginx
-rw-r--r--   1 root root  124 Apr  2  2018 redis-server
-rw-r--r--   1 root root  501 Jan 14  2018 rsyslog
-rw-r--r--   1 root root  178 Aug 15  2017 ufw
-rw-r--r--   1 root root  235 Nov 25  2019 unattended-upgrades
```
我们看看这个默认的配置`cat /etc/logrotate.d/nginx`: 
```dtd
/var/log/nginx/*.log {
	daily
	missingok
	rotate 60
	dateext
    dateformat .%Y%m%d
    #compress
	#delaycompress
	notifempty
	create 0640 www-data adm
	sharedscripts
	prerotate
		if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
			run-parts /etc/logrotate.d/httpd-prerotate; \
		fi \
	endscript
	postrotate
		invoke-rc.d nginx rotate >/dev/null 2>&1
	endscript
}
```
配置我就不多说了，自己去查。
# 2、问题
```dtd
每天6点半才切割日志，我想要的是每天0点。
```
为什么？

定位到问题了：
```dtd
root@fatpo:/var/log/nginx# cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```
可以看到：`6点25`！！！
```dtd
logrotate 默认是6点25的crontab。。好吧，明白了。
```


# 3、解决
增加一个crontab:
```
59 23 * * * /usr/sbin/logrotate -f /etc/logrotate.d/nginx  >> /var/log/nginx/logrotate.log 2>&1
```
然后稍微修改下配置，只切割`access.log`和`error.log`：
```
root@fatpo:~# cat /etc/logrotate.d/nginx
/var/log/nginx/access.log {
	daily
	missingok
	rotate 60
	dateext
        dateformat .%Y%m%d
        #compress
	#delaycompress
	notifempty
	create 0640 www-data adm
	sharedscripts
	prerotate
		if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
			run-parts /etc/logrotate.d/httpd-prerotate; \
		fi \
	endscript
	postrotate
		/usr/sbin/nginx -s reload # invoke-rc.d nginx rotate >/dev/null 2>&1
	endscript
}

/var/log/nginx/error.log {
        daily
        missingok
        rotate 60
        dateext
        dateformat .%Y%m%d
        #compress
        #delaycompress
        notifempty
        create 0640 www-data adm
        sharedscripts
        prerotate
                if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
                        run-parts /etc/logrotate.d/httpd-prerotate; \
                fi \
        endscript
        postrotate
                /usr/sbin/nginx -s reload # invoke-rc.d nginx rotate >/dev/null 2>&1
        endscript
}
```
注意这里的`postrotate`，如果不重启nginx，那么日志并不是重新输入到access.log，因为句柄没改变。

ps：
```
nginx 必须写完全的路径，否则会识别不到。
```

# 4、django的日志也不对
时间：`2021年11月11日21:00:01`，django的日志切割一直使用内部的：
```
logging.handlers.TimedRotatingFileHandler
```
但是自从我搞了nginx的logrotate后，django的日志就一直不准了，总是前后两天的混在一起。

难受啊，感觉这nginx和django怎么会搞上呢？没理由啊。

后来想清楚了，那段时间，我不仅改了nginx配置，我还改了gunicorn的配置。

是不是我把gunicorn的django worker数量从1提升到3导致的？

查了下google，还真的是有这个可能！

搜了下发现好像没有什么解决方案，最好是不要用django自带的切割，用logrotate，好吧，先把django的日志配置改为：
改动前：
```
 'file_handler': {
    'level': 'INFO',
    'class': 'logging.handlers.TimedRotatingFileHandler',
    'filename': '%s/django.log' % LOG_DIR,
    'formatter': 'standard',
    'encoding': 'utf-8',
    'when': 'midnight',
    'interval': 1,
    'backupCount': 30,
}, 
```
改动后：
```
    'file_handler': {
        'level': 'INFO',
        'class': 'logging.handlers.WatchedFileHandler',
        'filename': '%s/django.log' % LOG_DIR,
        'formatter': 'standard',
        'encoding': 'utf-8'
    },  
```
然后再新建一个 `/etc/logrotate.d/myapp`:
```
root@fatpo:~/www/myapp/logs# cat /etc/logrotate.d/myapp
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
这里有点难点，我找不到 `gunicorn.pid`，所以我去supervisor的gunicorn启动参数制定了pid: 
```
--pid /root/www/myapp/logs/gunicorn.pid
```
整个supervisor文件如下：
```
cat /etc/supervisor/myapp.ini
[program:myapp]
user=root
environment= PATH="/usr/bin"
directory=/root/www/myapp/
#command=/usr/bin/python3 manage.py runserver
command=/usr/local/bin/gunicorn --reload -k gevent -w 3 -b 0.0.0.0:8000 --pid /root/www/myapp/logs/gunicorn.pid --error-logfile /root/www/myapp/logs/gunicorn.err --log-file /root/www/myapp/logs/gunicorn.log  --access-logfile  /root/www/myapp/logs/gunicorn_access.log  myapp.wsgi:application
#redirect_stderr=false
stdout_logfile=/root/www/myapp/logs/myapp-supervisor.log
stderr_logfile=/root/www/myapp/logs/myapp-supervisor.err
```

记得要配置crontab:
```
0 0 * * * /usr/sbin/logrotate -f /etc/logrotate.d/myapp  >> /root/www/myapp/logs/logrotate_myapp.log 2>&1
```
因为我在logrotate配置了`dateyesterday` ，所以我的crontab可以是当天`0 0`搞，否则就要设置`59 23`，这样子不雅观，还会丢失一分钟的日志。

# 5、参考
* [linux环境下使用logrotate工具实现nginx日志切割](https://zhuanlan.zhihu.com/p/24880144)
* [linux下日志定时轮询的流程详解](https://cloud.tencent.com/developer/article/1720635)
* [Specify the time of daily log rotate](https://askubuntu.com/questions/24503/specify-the-time-of-daily-log-rotate)