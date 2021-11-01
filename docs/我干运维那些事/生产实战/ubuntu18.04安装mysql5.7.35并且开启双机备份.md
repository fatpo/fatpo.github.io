# 1、打算安装一个mysql
```text
select version();

5.7.35-0ubuntu0.18.04.1
```

# 2、安装mysql

```text
sudo apt-get update
sudo apt-get install mysql-server
```
如果要指定版本，先搜索：
```text
➜  ~ sudo apt-cache  madison mysql-server
mysql-server | 5.7.36-0ubuntu0.18.04.1 | http://mirrors.tencentyun.com/ubuntu bionic-security/main amd64 Packages
mysql-server | 5.7.36-0ubuntu0.18.04.1 | http://mirrors.tencentyun.com/ubuntu bionic-security/main i386 Packages
mysql-server | 5.7.36-0ubuntu0.18.04.1 | http://mirrors.tencentyun.com/ubuntu bionic-updates/main amd64 Packages
mysql-server | 5.7.36-0ubuntu0.18.04.1 | http://mirrors.tencentyun.com/ubuntu bionic-updates/main i386 Packages
mysql-server | 5.7.21-1ubuntu1 | http://mirrors.tencentyun.com/ubuntu bionic/main amd64 Packages
mysql-server | 5.7.21-1ubuntu1 | http://mirrors.tencentyun.com/ubuntu bionic/main i386 Packages
mysql-5.7 | 5.7.21-1ubuntu1 | http://mirrors.tencentyun.com/ubuntu bionic/main Sources
mysql-5.7 | 5.7.36-0ubuntu0.18.04.1 | http://mirrors.tencentyun.com/ubuntu bionic-security/main Sources
mysql-5.7 | 5.7.36-0ubuntu0.18.04.1 | http://mirrors.tencentyun.com/ubuntu bionic-updates/main Sources
```
再指定版本安装：
```text
➜  ~ sudo apt-get install  mysql-server=5.7.36-0ubuntu0.18.04.1
Reading package lists... Done
Building dependency tree       
Reading state information... Done
mysql-server is already the newest version (5.7.36-0ubuntu0.18.04.1).
0 upgraded, 0 newly installed, 0 to remove and 148 not upgraded.
```
其中`apt-cache  madison`的`madison` 很有意思，好像是来自 `Debian`，从SO找到的解释如下：
```text
The madison command was added in apt 0.5.20. 

It produces output that's similar to a then-existing tool called madison which was used by Debian server administrators. 

Several of these tools had names which were common female forenames, I don't know if there's a specific history behind that.

The madison tool no longer exists but there's a partial reimplementation 
called madison-lite (querying a local package archive, like the original), 
as well as a script called rmadison in devscripts which queries remote servers.

apt-cache madison is not emphasized because most of what it displays is also available through apt-cache showpkg and apt-cache policy.
```

 
# 3、修改密码
```text
$ mysql -u root

mysql> USE mysql; 
mysql> update user set authentication_string=PASSWORD('your_password_here') where user='root';
mysql> update user set plugin="mysql_native_password" where User='root';
mysql> flush privileges;
mysql> quit
```

# 4、数据库备份

主机A在跑应用，我打算在主机B每天定时备份下主机A的数据。
```text
➜  shells cat bakdb.sh 
#!/bin/bash 
time=$(date "+%Y-%m-%d %H:%M:%S")
echo 'back db at ' "${time}"

# 从远程服务器A拉回来数据库备份
ssh root@140.1.2.3  "mysqldump --add-drop-table -uroot -p123456  myappdb | gzip -9" > dblocal.sql.gz

# 导入到服务器B的相同数据库
gunzip < dblocal.sql.gz | mysql  -uroot -p123456 myappdb

# 把数据库备份一波
datestr=$(date "+%Y%m%d")
mv dblocal.sql.gz dblocal.sql.gz.${datestr}

# 删除远古备份 
ls  $(pwd)/dblocal.sql.gz* |sort -nr | awk 'NR>30' | xargs rm -f 
```
然后`crontab`:
```text
# m h  dom mon dow   command
0 11 * * * /bin/bash /home/ubuntu/shells/bakdb.sh >> /home/ubuntu/shells/bakdb.log 2>&1
```
这里主要感谢StackOverflow的大神，提供的远程备份：
```text
ssh root@140.1.2.3 "mysqldump --add-drop-table -uroot -p123456  myappdb | gzip -9" > dblocal.sql.gz
```

# 5、参考
* [Ubuntu18.04 安装MySQL](https://blog.csdn.net/weixx3/article/details/80782479)
* [Why apt madison?](https://unix.stackexchange.com/questions/276037/why-apt-madison)
* [mysql_用命令行备份数据库](https://www.cnblogs.com/hellangels333/p/9059770.html)
* [mysqldump-via-ssh-to-local-computer](https://stackoverflow.com/questions/40024280/mysqldump-via-ssh-to-local-computer)