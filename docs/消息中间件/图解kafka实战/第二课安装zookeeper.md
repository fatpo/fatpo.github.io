# 0、为何需要zk
zk是集群搞分布式的基石，很多开源项目，只要用到分布式，基本上都会用到zk，因为zk能帮你管理分布式的节点元信息。

像阿里的rocketMQ早期版本也是用zk来管理节点元信息的，只是后面改为nameServer，作用一致。

# 1、下载zk
下载地址1：
```
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz
```
下载地址2：
```
wget http://archive.apache.org/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz
```
随便找个地方解压，比如我的：
```text
/home/ubuntu/zk_home/zookeeper-3.4.12
```
把zkHome路径存入/etc/profile:
```text 
export ZOOKEEPER_HOME=/home/ubuntu/zk_home/zookeeper-3.4.12
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```
别忘记source /etc/profile 生效一下后，验证下:
```text
$ echo $ZOOKEEPER_HOME
/home/ubuntu/zk_home/zookeeper-3.4.12
```


# 2、模拟单机集群

## 2.1、复制配置文件zoo.cfg
配置文件1：
```text
➜  zookeeper-3.4.12 cat conf/zoo1.cfg 
server.0=172.17.0.10:2886:3886
server.1=172.17.0.10:2887:3887
server.2=172.17.0.10:2888:3888

# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/tmp/zookeeper/data1
dataLogDir=/tmp/zookeeper/log1
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```
配置文件2：
```text
➜  zookeeper-3.4.12 cat conf/zoo1.cfg 
server.0=172.17.0.10:2886:3886
server.1=172.17.0.10:2887:3887
server.2=172.17.0.10:2888:3888

# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/tmp/zookeeper/data2
dataLogDir=/tmp/zookeeper/log2
# the port at which the clients will connect
clientPort=2182
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```
配置文件3：
```text
➜  zookeeper-3.4.12 cat conf/zoo1.cfg 
server.0=172.17.0.10:2886:3886
server.1=172.17.0.10:2887:3887
server.2=172.17.0.10:2888:3888

# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/tmp/zookeeper/data3
dataLogDir=/tmp/zookeeper/log3
# the port at which the clients will connect
clientPort=2183
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

总而言之，关注这3个配置，三个zk实例要三个path+三个端口：
```text
clientPort=2181
dataDir=/tmp/zookeeper/data1
dataLogDir=/tmp/zookeeper/log1
```
## 2.3、创建目录
```text
mkdir -p  /tmp/zookeeper/data1/
mkdir -p  /tmp/zookeeper/data2/
mkdir -p  /tmp/zookeeper/data3/
mkdir -p  /tmp/zookeeper/log1/
mkdir -p  /tmp/zookeeper/log2/
mkdir -p  /tmp/zookeeper/log3/
```

## 2.3、创建myid
给zk实例1创建myid:
```text
➜  zookeeper-3.4.12 cat /tmp/zookeeper/data1/myid
0
```
给zk实例2创建myid:
```text
➜  zookeeper-3.4.12 cat /tmp/zookeeper/data2/myid
1
```
给zk实例3创建myid:
```text
➜  zookeeper-3.4.12 cat /tmp/zookeeper/data3/myid
2
```

## 2.4、启动zk

```text
bin/zkServer.sh start conf/zoo1.cfg
bin/zkServer.sh start conf/zoo2.cfg
bin/zkServer.sh start conf/zoo3.cfg
```

观察下进程确定有3个`QuorumPeerMain`，这就是zk实例，启动成功：
```text
➜  zookeeper-3.4.12 jps
14544 Jps
12816 QuorumPeerMain
12649 QuorumPeerMain
12750 QuorumPeerMain
```

