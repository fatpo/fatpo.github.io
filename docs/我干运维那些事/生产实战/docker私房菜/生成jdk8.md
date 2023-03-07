
# 下载
```text
docker pull openjdk:8
```

# 跑起来
因为我是用来联网的，网络是 `mynetwork`，而且我还要目录映射，所以能跑的容器是：
```text
docker run --name jdk8 -v /Users/fatpo/Desktop:/tmp/ -it -d --network mynetwork openjdk:8 /bin/sh
```
纯跑起来：
```text
docker run --name jdk8  -it -d openjdk:8 /bin/sh
```

# 记录坑
一般来说直接：
```text
docker run --name jdk8  -d openjdk:8
```
会马上挂，原因是 dockerfile 没有一个进程挂着，所以会挂。

所以我们让容器启动后，马上跑一个进程，容器就正常使用：
```text
docker run --name jdk8  -it -d openjdk:8 /bin/sh
```