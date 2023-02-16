# 1、科普

key & csr & crt， openssl 就是围着这 3 个转。

# 2、key & crt

## 2.1、key

KEY 是指密钥，通过一定长度的随机数，经过指定的加密算法处理，得到后的一个文本密钥文件。

文件中既包含了 公钥 Publish Key & Private key 两个部分。(`但是下面的例子并没 Publish Key`)

大概长这个样子：
```text
-----BEGIN RSA PRIVATE KEY-----
MIICXQIBAAKBgQCwTStUyng3mXX89JYnaf290gurEseCnbINvsFpdGSUgFtXv9Yy
5pPmSCXSTIjPiRjspF1hOS45S1Tt3EiErcYOqmOr0OVBoL920tWRCHUETmkm5u5S
rfrTJKn1SmrEx4gt9RJSthB6GLENp4N0HaEm8dZQ6tfDGpFbyj3L/Cvq6QIDAQAB
AoGAFzD/izbbG40/XRgbjHG/5DF2QXnF7uOpoW8/aAzckkBUQ7XDioyUVg2VlBVn
Rc2mDrMMaZapAvZq5KM+kt19GQaZI+x06phwP6KZ0E3PVt2rJheqjVM5CHa2A+3Y
76SnqOEatRRt1ZgG5di0VbnW1rjeiXIpLRWV+ePChWN+71ECQQDrD+JkXBr2aoJc
i4pVjCmLeozJHd/qkWj7pSWb/nhdfd4Ct4G8QUQM2sdsS8oOcG9b4mbMuX4rcvU6
jx7qEZw9AkEAwAFeLYchVuTLhGkenU1hqlCDZ8PjOYcPPfCCt1Xk2bbDu/dF/96m
3IS0xIV7X60ufjBsZfYk5nnJmgieGp2YHQJBAMxBfUQgFP3TB3xLdOVpaiBdWUDZ
yN0XhdZFZyzqLsVuviA2PXHMdMmGwouEQAvT/7AkR5fWB6DRv+4mt4JF0zECQC+Y
pzX2B4e409KRFGu+IPXNW6W/Y3aBSn/6PQ0hl8d4jPDtjUaudQK5Su5kgH7pOVtC
ubxU1jTj/9vVQwwqAOkCQQDqDH3NdM8aP6uPcJ6qtOduVBjZlJcQlqZFen8sUBjd
MefEtokDdrURnb4bqTbf9SeCiEcFS/lQkg7EoL78jySG
-----END RSA PRIVATE KEY-----
```


两个文件的头部标识也可见出差别。

有一个问题：RSA密钥都是一对，可这里只生成了私钥，那公钥在哪呢？

实际上公钥是根据私钥生成的，由于RSA非对称特性，私钥算出公钥很容易，公钥算出私钥就很难。生成公钥的指令为：
```text
$ openssl rsa -in ca.key -pubout -out ca.pub
```

## 2.2、crt
`CERTIFICATE` 的缩写。 和 key 是天生一对。

KEY 密钥文件需要严格保密，而 CRT 证书文件却是公开的。

大概长这个样子：
```text
-----BEGIN CERTIFICATE-----
MIICZjCCAc+gAwIBAgIULcbqda3JbL4fefJxafoC7ZHrZA8wDQYJKoZIhvcNAQEL
BQAwRTELMAkGA1UEBhMCQVUxEzARBgNVBAgMClNvbWUtU3RhdGUxITAfBgNVBAoM
GEludGVybmV0IFdpZGdpdHMgUHR5IEx0ZDAeFw0yMzAyMTMwNzAyMTJaFw0zMzAy
MTMwNzAyMTJaMEUxCzAJBgNVBAYTAkFVMRMwEQYDVQQIDApTb21lLVN0YXRlMSEw
HwYDVQQKDBhJbnRlcm5ldCBXaWRnaXRzIFB0eSBMdGQwgZ8wDQYJKoZIhvcNAQEB
BQADgY0AMIGJAoGBALBNK1TKeDeZdfz0lidp/b3SC6sSx4Kdsg2+wWl0ZJSAW1e/
1jLmk+ZIJdJMiM+JGOykXWE5LjlLVO3cSIStxg6qY6vQ5UGgv3bS1ZEIdQROaSbm
7lKt+tMkqfVKasTHiC31ElK2EHoYsQ2ng3QdoSbx1lDq18MakVvKPcv8K+rpAgMB
AAGjUzBRMB0GA1UdDgQWBBQKen5ZC8kk16b/kHu4h8Z1jTZhbzAfBgNVHSMEGDAW
gBQKen5ZC8kk16b/kHu4h8Z1jTZhbzAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3
DQEBCwUAA4GBAHEK+tU7VllC38FvD2TnaT8rEaR2rcDv3xUD71CQQqfTCCeNHljq
su4C0d5IKe6uR8mr0qaRoTgjDC6lDYXfEQx9M5uC47vC/J0aw+8Y6O0634nlFcGP
T6Sb7qChOgqJAaD06y71vxh1m9+FrBJIYz4Az9+P3JhceiTs8gsTqoqR
-----END CERTIFICATE-----
```

# 3、csr

## 3.1、ca
讲解 csr 之前，要介绍一个权威机构。

ca 是一个权威机构，证书中心，我们无脑信任它，也信任它颁发的证书。

## 3.2、csr

如果你想要一个 CA (Certificate Authority) 认证机构为你颁发CRT 证书的话，那你需要先提供一个 CSR 请求文件。

这个 CSR 请求文件中，会包含 Public Key 公钥，以及一些申请者信息 (DN Distingusied Name)。

## 3.3、csr 和 crt 的关系

* 1、SSL Server 自己生成一个 密钥 KEY，内置包含 Private Key/Public Key， 私钥加密，公钥解密；
* 2、Server 利用 密钥 生成一个请求文件 server.csr，请求文件中包含有 Server 的一些信息，如域名/申请者/公钥等；
* 3、Server 将请求文件 server.csr 递交给 CA，然后 CA 对其验明正身后，将用 ca.key 和 server.csr 加密生成 server.crt 证书；
* 4、Client（浏览器）访问 server，从 server 拿到一个server.crt证书，由于 ca.key 和 ca.crt 是一对，于是以后客户端通过使用 ca.crt 就可以解密认证 server.crt 证书了，解得出来，就说明这个 server 给的证书是 ca 认可的。

# 4、不带 ca 机构和 csr 玩（给 ca 要花钱的）

如果不带 ca 结构，我们也不用生成什么鬼 csr文件了，直接就是 .key + .cst 文件玩起来就行。

Self-signed 的证书也可以用来加密连接，只不过你的用户在访问你的网站的时候，将会提示出警告，说你的网站是不被信任的。
因此当你开发的服务，并不需要提供给其他用户使用的时候 (e.g. non-production or non-public servers)，你才可以使用 Self-signed 形式的证书。

## 4.1、没有 key 文件，生成 Self-Signed 的证书
以下命令将会生成一个 `2048-bits` 的 Private Key (self-ca.key) 和 Self-Signed 的 CRT 证书 (self-ca.crt)：
```text
openssl req  -newkey rsa:2048 -nodes -keyout self-ca.key -x509 -days 365 -out self-ca.crt
```
参数说明：
```text
-nodes  如果指定了此选项，创建私钥时则不会对其进行加密
-x509 此选项输出自签名证书。这通常用于生成测试证书或自签名根证书。添加到证书的扩展项（如果有）在配置文件中指定。除非使用set_serial选项，否则将使用较大的随机数作为序列号。
-days 365 表明生成的证书有效时间为 365 天
```

## 4.2、已有 key 文件，生成 Self-Signed 的证书
通过已有的密钥 Private Key 来产生 Self-Signed 证书

以下命令是通过已有的 Private Key (self-ca.key)，来生成一个 Self-Signed 的 CRT 证书 (self-ca.crt)：
```text
openssl req -key self-ca.key -new -x509 -days 365 -out self-ca.key
```


## 4.3、通过已有的密钥 Private Key & 请求文件 CSR 来产生 Self-Signed 证书

以下命令是通过已有的 Private Key (self-ca.key) 以及请求文件 (self-ca.csr)，来生成一个 Self-Signed 的 CRT 证书 (self-ca.crt)：
```text
openssl x509 -signkey self-ca.key -in self-ca.csr -req -days 365 -out self-ca.crt
```

## 4.4、通过已有的请求文件 CSR，通过 Self-Signed CA 来签发 CRT 证书

以下命令是通过已有的 Self-Signed CA 的证书 (self-ca.crt) 和密钥 (self-ca.key)，和请求文件 CSR (domain.csr) 来签发生成 CRT 证书 (domain.crt)：
```text
openssl x509 -req -CAcreateserial -days 365 -CA self-ca.crt -CAkey self-ca.key -in nginx.csr -out nginx.crt
```
这个命令就很像模拟 CA 中心的生成了。


# 5、证书验证

Openssl 默认生成的证书相关的文件都是 PEM 格式编码的，这部分通过 openssl 命令来解析这些文件。

## 5.1、查看请求文件 CSR
```text
openssl req -text -noout -verify -in domain.csr

```

## 5.2、查看证书文件 CRT
```
openssl x509 -text -noout -in domain.crt
```

## 5.3、校验证书文件 CRT 合法性
以下命令来校验 domain.crt 证书，是否是由 ca.crt 证书签发出来的：
```text
openssl verify -verbose -CAFile ca.crt domain.crt
```

## 5.4、校验 Private Key & CSR & CRT 三者是否是匹配
以下命令是分别提取出 Private Key (domain.key) & CSR (domain.csr) & CRT (domain.crt) 三者中包含的 Public Key，然后通过 md5 运算检查是否一致：
```text
openssl rsa -noout -modulus -in domain.key | openssl md5
openssl req -noout -modulus -in domain.csr | openssl md5
openssl x509 -noout -modulus -in domain.crt | openssl md5
```




# 6、原文搬运
* [玩转 SSL 证书](https://yakir-yang.github.io/2018/08/09/Tools-%E7%8E%A9%E8%BD%AC-SSL-%E8%AF%81%E4%B9%A6/)
* [OpenSSL中文手册之命令行详解（未完待续）](https://blog.csdn.net/liao20081228/article/details/77159039)
