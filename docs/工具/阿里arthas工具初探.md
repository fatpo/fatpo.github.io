# 1、启动arthas-demo
![](.阿里arthas工具初探_images/2c412ab4.png)

# 2、启动arthas-boot
在新的Terminal 2里，下载`arthas-boot.jar`，再用`java -jar`命令启动：
```dtd
wget https://arthas.aliyun.com/arthas-boot.jar;java -jar arthas-boot.jar
```
arthas-boot是Arthas的启动程序，它启动后，会列出所有的Java进程，用户可以选择需要诊断的目标进程。

选择第一个进程，输入 `1` ，再`Enter/回车`：
```dtd
1
```
Attach成功之后，会打印Arthas LOGO。输入 `help` 可以获取到更多的帮助信息。

![](.阿里arthas工具初探_images/ceb0ee01.png)

# 3、打开Dashboard
`dashboard` 命令可以查看当前系统的实时数据面板。
```dtd
dashboard
```
输入 `q` 或者 `Ctrl+C` 可以退出dashboard命令。

![](.阿里arthas工具初探_images/725705f4.png)

# 4、thread查看线程
`thread 1` 可以看`ID=1`的线程：
![](.阿里arthas工具初探_images/82d9ebc9.png)

# 5、sc查看JVM里已加载的类
可以通过 sc 命令来查找JVM里已加载的类：
```dtd
sc -d *MathGame
```
![](.阿里arthas工具初探_images/097b2c26.png)

# 6、Jad反编译类

可以通过 jad 命令来反编译代码：
```dtd
jad demo.MathGame
```
![](.阿里arthas工具初探_images/abe1bb5e.png)

# 7、watch看函数返回值
通过watch命令可以查看函数的参数/返回值/异常信息。
```dtd
watch demo.MathGame primeFactors returnObj
```
输入 `q` 或者 `Ctrl+C` 退出`watch`命令。
![](.阿里arthas工具初探_images/920aacb7.png)