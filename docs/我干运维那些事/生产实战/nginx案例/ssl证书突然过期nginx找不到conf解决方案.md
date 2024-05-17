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
结果看了下日志，果然renew失败了：
```text
root@v2txtnowoffline:~/www/project# "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh"
[Sat 18 Nov 2023 12:23:50 AM UTC] ===Starting cron===
[Sat 18 Nov 2023 12:23:50 AM UTC] Already uptodate!
[Sat 18 Nov 2023 12:23:50 AM UTC] Upgrade success!
[Sat 18 Nov 2023 12:23:50 AM UTC] Auto upgraded to: 3.0.7
[Sat 18 Nov 2023 12:23:50 AM UTC] Renew: 'abc.hehe.xyz'
[Sat 18 Nov 2023 12:23:50 AM UTC] Renew to Le_API=https://acme.zerossl.com/v2/DV90
[Sat 18 Nov 2023 12:23:51 AM UTC] Using CA: https://acme.zerossl.com/v2/DV90
[Sat 18 Nov 2023 12:23:51 AM UTC] Single domain='abc.hehe.xyz'
[Sat 18 Nov 2023 12:23:51 AM UTC] Getting domain auth token for each domain
[Sat 18 Nov 2023 12:23:53 AM UTC] Getting webroot for domain='abc.hehe.xyz'
[Sat 18 Nov 2023 12:23:53 AM UTC] Verifying: abc.hehe.xyz
[Sat 18 Nov 2023 12:23:53 AM UTC] Nginx mode for domain:abc.hehe.xyz
[Sat 18 Nov 2023 12:23:53 AM UTC] Can not find conf file for domain abc.hehe.xyz
[Sat 18 Nov 2023 12:23:53 AM UTC] Please add '--debug' or '--log' to check more details.
[Sat 18 Nov 2023 12:23:53 AM UTC] See: https://github.com/acmesh-official/acme.sh/wiki/How-to-debug-acme.sh
[Sat 18 Nov 2023 12:23:54 AM UTC] Error renew abc.hehe.xyz_ecc.

```
主要是找不到conf文件：`Can not find conf file for domain`，难以置信，我的文件明明在那里。


## 解决方案
尝试升级 acme：
```
"/root/.acme.sh"/acme.sh  --upgrade  --auto-upgrade
```
没用。

尝试升级 dev版本：
```text
acme.sh --upgrade -b dev
```
没用。

找到类似的github issue： 
```
https://github.com/acmesh-official/acme.sh/issues/1914
```
果然我不是第一个遇到这个问题的人，最终解决方案：
```text
acme.sh --issue --nginx /etc/nginx/sites-enabled/myconfig -d mydomain.com
```
