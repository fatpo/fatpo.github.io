# 1、生成证书

## 1.1、生成服务端证书
```text
# 为服务端的证书取一个基本的名字。
$ FILENAME=server
# 生成公钥私钥对。
$ openssl genrsa -out $FILENAME.key 1024
# 生成一个自签名的证书，会提示你输入国家代号、姓名等，或者按下回车键跳过输入提示。
$ openssl req -new -key $FILENAME.key -x509 -days 3653 -out $FILENAME.crt
# 用刚生成的密钥文件和证书文件来生成PEM文件。
$ cat $FILENAME.key $FILENAME.crt >$FILENAME.pem
```

## 1.2、生成客户端证书
```text
# 为服务端的证书取一个基本的名字。
$ FILENAME=client
# 生成公钥私钥对。
$ openssl genrsa -out $FILENAME.key 1024
# 生成一个自签名的证书，会提示你输入国家代号、姓名等，或者按下回车键跳过输入提示。
$ openssl req -new -key $FILENAME.key -x509 -days 3653 -out $FILENAME.crt
# 用刚生成的密钥文件和证书文件来生成PEM文件。
$ cat $FILENAME.key $FILENAME.crt >$FILENAME.pem
```
除了 filename 变了下，其他都一样。


# 2、交换证书

* 复制 server.pem 到 SSL 服务端主机，复制 server.crt 到客户端主机
* 复制 client.pem 到 SSL 客户端主机，复制 client.crt 到服务端主机
* 最终：服务端有 server.pem、client.crt 两个文件，客户端有 client.pem 、server.crt 两个文件

# 3、socat创建Openssl服务端
监听 4433 ：
```text
socat openssl-listen:4433,reuseaddr,cert=server.pem,cafile=client.crt echo
```
注意：
```text
其实就是把 tcp-listen 替换成 openssl-listen
```

运行这个命令后，Socat 会在 4433 端口监听，并要求客户端进行身份验证。

# 4、socat在客户端创建一个加密链接
假设服务端的域名是：`server.domain.org`，连接它的 4433 端口：
```text
$ socat stdio openssl-connect:server.domain.org:4433,cert=client.pem,cafile=server.crt
test
test
```
这个命令用来建立一个到服务程序的安全的连接。如果服务端和客户端成功建立连接后，会回显在客户端输入的内容。


# 5、参考链接
* [What actually reuseaddr option does in socat?](https://stackoverflow.com/questions/75059280/what-actually-reuseaddr-option-does-in-socat)
* [奇妙的linux世界：Socat 入门教程](https://mp.weixin.qq.com/s/96GQcFN_Avjfu1zvJDVEyg)