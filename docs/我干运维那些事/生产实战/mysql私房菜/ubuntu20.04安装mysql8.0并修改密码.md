安装：
```text
apt-get install mysql-server
```

修改配置文件：
```text
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
在 [mysqld] 下添加：
```text
skip-grant-tables
```
重启mysql：
```text
systemctl restart mysq
```
进入 mysql修改密码：
```text
flush privileges;

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';

flush privileges;
```
done.

