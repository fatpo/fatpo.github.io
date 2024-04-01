# 背景
如果已经打开了ssh无密码登录，再加上fail2ban就万无一失了。

关闭ssh密码登录教程可以参考国哥写的另外一篇：[传送门](https://fatpo.github.io/#/我干运维那些事/生产实战/linux私房菜/ubuntu18.04取消ssh密码登录)

# 教程
他山之石： [https://www.linuxidc.com/Linux/2018-11/155450.htm](https://www.linuxidc.com/Linux/2018-11/155450.htm)

大家可以去原博客点赞。

===================

安装fail2ban很简单。登录到您的Ubuntu服务器并更新/升级。请注意，如果在此过程中升级内核，则必须重新启动服务器（因此在重新启动可行时运行此服务器）。

要更新和升级服务器，请发出以下命令：
```
sudo apt-get update
sudo apt-get upgrade
```

完成上述命令后，重新启动服务器（如有必要）。

可以使用单个命令安装fail2ban：
```
sudo apt-get install -y fail2ban
```

当该命令完成时，fail2ban准备好了。您将要使用以下命令启动并启用该服务：
```
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

配置jail

接下来我们将为SSH登录尝试配置一个jail。在/etc fail2ban目录中，您将找到jail.conf文件。不要编辑此文件。相反，我们将创建一个新文件jail.local，它将覆盖jail.conf中的任何类似设置。我们的新jail配置将监视/var/log/auth.log，使用fail2ban sshd过滤器，将SSH端口设置为22，并将最大重试次数设置为3.为此，请发出命令：
```
sudo vi /etc/fail2ban/jail.local
```

在此新文件中，粘贴以下内容：
```
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
```

保存并关闭该文件。使用以下命令重新启动fail2ban：
```
sudo systemctl restart fail2ban
```

此时，如果有人试图通过SSH登录您的Ubuntu服务器，并且失败了三次，那么将通过iptables阻止其IP地址阻止它们进入。


# 检查是否开启服务
看服务是否running：
```text
root@VM-0-10-ubuntu:~# sudo systemctl status fail2ban

● fail2ban.service - Fail2Ban Service
   Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2024-04-01 22:23:11 CST; 22s ago
     Docs: man:fail2ban(1)
 Main PID: 30052 (fail2ban-server)
    Tasks: 3 (limit: 2125)
   CGroup: /system.slice/fail2ban.service
           └─30052 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

```

看服务是否有日志：
```text
root@VM-0-10-ubuntu:~# tail -f /var/log/fail2ban.log
2024-04-01 22:23:14,754 fail2ban.filter         [30052]: INFO    [sshd] Found 45.180.136.12 - 2024-04-01 22:21:42
2024-04-01 22:23:14,755 fail2ban.filter         [30052]: INFO    [sshd] Found 43.134.180.14 - 2024-04-01 22:22:11
2024-04-01 22:23:14,755 fail2ban.filter         [30052]: INFO    [sshd] Found 43.128.69.133 - 2024-04-01 22:22:34
2024-04-01 22:23:14,756 fail2ban.filter         [30052]: INFO    [sshd] Found 43.154.90.94 - 2024-04-01 22:22:47
2024-04-01 22:23:14,756 fail2ban.filter         [30052]: INFO    [sshd] Found 45.180.136.12 - 2024-04-01 22:23:00
2024-04-01 22:23:15,238 fail2ban.actions        [30052]: NOTICE  [sshd] Ban 45.180.136.12
2024-04-01 22:23:15,279 fail2ban.actions        [30052]: NOTICE  [sshd] Ban 43.128.69.133
2024-04-01 22:23:15,293 fail2ban.actions        [30052]: NOTICE  [sshd] Ban 43.154.90.94
2024-04-01 22:23:15,302 fail2ban.actions        [30052]: NOTICE  [sshd] Ban 43.134.180.14
2024-04-01 22:23:47,287 fail2ban.filter         [30052]: INFO    [sshd] Found 103.186.1.76 - 2024-04-01 22:23:47

```


# 如果安装失败发现是格式问题
查看错误日志：
```text
sudo journalctl -u fail2ban

Apr 01 01:57:58 VM-0-10-ubuntu systemd[1]: Starting Fail2Ban Service...
Apr 01 01:57:58 VM-0-10-ubuntu systemd[1]: Started Fail2Ban Service.
Apr 01 01:57:59 VM-0-10-ubuntu fail2ban-server[1359]: Traceback (most recent call last):
Apr 01 01:57:59 VM-0-10-ubuntu fail2ban-server[1359]:   File "/usr/bin/fail2ban-server", line 34, in <module>
Apr 01 01:57:59 VM-0-10-ubuntu fail2ban-server[1359]:     from fail2ban.client.fail2banserver import exec_command_line, sys
Apr 01 01:57:59 VM-0-10-ubuntu fail2ban-server[1359]:   File "/usr/lib/python3/dist-packages/fail2ban/client/fail2banserver.py", line 173
Apr 01 01:57:59 VM-0-10-ubuntu fail2ban-server[1359]:     async = self._conf.get("async", False)
Apr 01 01:57:59 VM-0-10-ubuntu fail2ban-server[1359]:           ^
Apr 01 01:57:59 VM-0-10-ubuntu fail2ban-server[1359]: SyntaxError: invalid syntax
Apr 01 01:58:00 VM-0-10-ubuntu systemd[1]: fail2ban.service: Main process exited, code=exited, status=1/FAILURE
Apr 01 01:58:00 VM-0-10-ubuntu systemd[1]: fail2ban.service: Failed with result 'exit-code'.

```
啊？语法错误？

解决方案（把关键词async替换成is_async）：
```text
sed -i 's/\basync\b/is_async/g' /usr/lib/python3/dist-packages/fail2ban/client/fail2banserver.py
sed -i 's/\basync\b/is_async/g' /usr/lib/python3/dist-packages/fail2ban/client/fail2banclient.py
```
重启：
```text
sudo systemctl start fail2ban  
sudo systemctl enable fail2ban
```

重新看服务状态。