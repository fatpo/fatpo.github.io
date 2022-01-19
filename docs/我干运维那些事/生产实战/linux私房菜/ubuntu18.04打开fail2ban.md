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
sudo nano /etc/fail2ban/jail.local
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