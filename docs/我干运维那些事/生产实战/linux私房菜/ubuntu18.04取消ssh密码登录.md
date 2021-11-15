# 关闭密码登录
处于安全考虑，不然密码猜测。

在服务端的sshd服务配置中，修改配置`vi /etc/ssh/sshd_config`，如下：
```text
PasswordAuthentication no
PubkeyAuthentication yes
```

重启:
```text
service sshd restart
```