# 1、背景
最近在搞那个`logrotate`，其中有一个操作就是需要重启服务，让文件刷新句柄。

其中配置`cat /etc/logrotate.d/myapp`：
```
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
有一个 `kill`: 
```
kill -USR1 $(cat /root/www/myapp/logs/gunicorn.pid)
```
所以就想知道这个kill怎么才是最佳实践。

# 2、搜索资料
网上很多资料都是互相抄来抄去，这里我摘抄下关键的：
```
kill -HUP pid  pid 是进程标识。如果想要更改配置而不需停止并重新启动服务，请使用该命令。在对配置文件作必要的更改后，发出该命令以动态更新服务配置。

 根据约定，当您发送一个挂起信号（信号 1 或 HUP）时，大多数服务器进程（所有常用的进程）都会进行复位操作并重新加载它们的配置文件。清单 2 显示了向所有正在运行的 Web 服务器进程发送挂起信号的一种方法。
```


```
HUP(1)是让进程挂起，睡眠;

kill (9)六亲不认的杀掉

term(15)正常的退出进程

因为进程可能屏蔽某些信号，所以它们的用处也就不一样。。。
```

# 3、kill -USR1 {pid}
这段解释来自`What does kill -USR1 %1 do?`： 
```
kill -USR1 %1 sends the "user-defined signal #1" (a.k.a. "SIGUSR1") to the first background child process of the current shell process. 

If that background process has set up a signal-handler function for the USR1 signal, 

that function will be run. If the target process doesn't have a signal-handler for that signal, the target process will terminate.
```
它凑巧解释了我们的`Waht does kill -USR1 {pid} do?` 问题。

看来 `-USR1` 是真的可以存在的。

# 4、终于找到man的解释
首先科普下man: [man的最佳实践](https://fatpo.github.io/#/我干运维那些事/生产实战/linux私房菜/man的最佳实践)

根据`man 7 signal`可以查到：
可以查看到:
* kill -HUP {PID}: Hangup detected on controlling terminal or death of controlling process
* kill -USR1 {PID}: SIGUSR1   30,10,16    Term    User-defined signal 1

man解释太官方且不够明了，信息比较少。

# 5、stackexchange 的神级回答
让我们看看来自`stackexchange`的关于`Program behavior when kill -HUP is recieved?`的星标回答：
```
Read its documentation. 

That's the only way. 

As Keith already wrote, the original meaning of SIGHUP was that the user had lost access to the program, 

and so interactive programs should die. 

Daemons — programs that don't interact directly with the user — 

have no need for this behavior and instead often reload their configuration files when they receive SIGHUP. 

But these are just conventions.
```

看到没有！！ `instead often reload their configuration files` + `But these are just conventions.` 

重载配置，这只是约定俗成的！！

果然kill -HUP 能够reload配置，并没有写到官方文档，只是约定俗成的！！

至此，国哥彻底解开了为什么用 `kill -HUP {pid}` 去重载配置的原因。

回答还提到，为什么HUP信号会重载配置，让我们去读读源码就知道了：
```
If you have the source, you can read that, too. Or if you only have the binary, you can try disassembling it, 

look for sigaction calls that set up a signal handler for SIGHUP, 

and try to figure out what those signal handlers are doing. 

It'll be easier arranging not to send SIGHUP to that program in the first place.
```


# 6、参考
* [What does kill -USR1 %1 do?](https://superuser.com/questions/1607829/what-does-kill-usr1-1-do)
* [Program behavior when kill -HUP is recieved?](https://unix.stackexchange.com/questions/15601/program-behavior-when-kill-hup-is-recieved)