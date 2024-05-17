# 1、背景
操作系统：
```
root@fatpo:/var/log/nginx# cat /etc/issue
Ubuntu 18.04.5 LTS \n \l
```

# 2、nginx 日志切割配置
`cat /etc/logrotate.d/nginx`: 
```dtd
/var/log/nginx/access.log {
        daily
        missingok
        rotate 60
        dateext
        dateyesterday
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
关键参数：`dateyesterday` 然后使用昨天的日期。