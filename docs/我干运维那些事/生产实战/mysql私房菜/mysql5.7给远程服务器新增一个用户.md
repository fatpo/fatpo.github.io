## 背景
mysql:
```text
mysql> select version();
+-----------------------------+
| version()                   |
+-----------------------------+
| 5.7.41-0ubuntu0.18.04.1-log |
+-----------------------------+
1 row in set (0.00 sec)

```

我有两台服务器，10.1.96.1 ，10.1.96.2，其中mysql在10.1.96.2上面，我能保证mysql运行正常，bind-address=0.0.0.0，并且密码正确，但就是报错了，AccessDenied.

checking:
```text
1. ping ok
2. telnet ok
3. mysql password ok
4. mysql bind-address ok
```
but `mysql -h 10.1.96.2 -uroot -p` failed with wrong pass? 

## 解决方案
直接创建一个新的用户给10.1.96.1 使用
```text
CREATE USER 'new_user'@'10.1.96.1' IDENTIFIED BY 'password';

GRANT ALL PRIVILEGES ON my_db.* TO 'new_user'@'10.1.96.1';

FLUSH PRIVILEGES;
```