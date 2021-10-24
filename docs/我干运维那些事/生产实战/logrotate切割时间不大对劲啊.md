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
我们看看 `cat /etc/logrotate.d/nginx`: 
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
root@vsim:/var/log/nginx# cat /etc/crontab
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
59 23 * * * logrotate -vf /etc/logrotate.d/nginx
```
`cat /etc/logrotate.d/nginx`:
```dtd
root@fatpo:/var/log/nginx# cat /etc/logrotate.d/nginx
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
		nginx -s reload # invoke-rc.d nginx rotate >/dev/null 2>&1
	endscript
}
```
注意这里的`postrotate`，如果不重启nginx，那么日志并不是重新输入到access.log，因为句柄没改变。

# 3、参考
* [linux环境下使用logrotate工具实现nginx日志切割](https://zhuanlan.zhihu.com/p/24880144)
* [linux下日志定时轮询的流程详解](https://cloud.tencent.com/developer/article/1720635)
* [Specify the time of daily log rotate](https://askubuntu.com/questions/24503/specify-the-time-of-daily-log-rotate)