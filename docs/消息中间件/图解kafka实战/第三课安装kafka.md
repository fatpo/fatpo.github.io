# 1、下载kafka
下载地址：
```
wget https://archive.apache.org/dist/kafka/2.0.0/kafka_2.11-2.0.0.tgz
```
随便找个地方解压，像我的是：
```text
/home/ubuntu/kafka_home/kafka_2.11-2.0.0
```
和zk一样，我们配置下kafka_home，vi /etc/profile:
```text
export KAFKA_HOME=/home/ubuntu/kafka_home/kafka_2.11-2.0.0
export PATH=$PATH:$KAFKA_HOME/bin
```
别忘记source /etc/profile 生效一下后，验证下:
```text
$ echo $KAFKA_HOME
/home/ubuntu/kafka_home/kafka_2.11-2.0.0
```

# 2、配置kafka
首先创建3个配置文件:
```
cp server.properties server1.properties
cp server.properties server2.properties
cp server.properties server3.properties
```

config/server1.properties关键配置:
```
broker.id=0
listeners=PLAINTEXT://localhost:9092
advertised.listeners=PLAINTEXT://172.17.0.10:9092
log.dirs=/tmp/kafka-logs1
zookeeper.connect=172.17.0.10:2181,172.17.0.10:2182,172.17.0.10:2183
```
config/server2.properties关键配置:
```
broker.id=1
listeners=PLAINTEXT://localhost:9093
advertised.listeners=PLAINTEXT://172.17.0.10:9093
log.dirs=/tmp/kafka-logs2
zookeeper.connect=172.17.0.10:2181,172.17.0.10:2182,172.17.0.10:2183
```
config/server3.properties关键配置:
```
broker.id=2
listeners=PLAINTEXT://localhost:9094
advertised.listeners=PLAINTEXT://172.17.0.10:9094
log.dirs=/tmp/kafka-logs3
zookeeper.connect=172.17.0.10:2181,172.17.0.10:2182,172.17.0.10:2183
```
创建3个日志目录：
```text
mkdir -p /tmp/kafka-logs1
mkdir -p /tmp/kafka-logs2
mkdir -p /tmp/kafka-logs3
```

# 3、启动kafka
## 3.1、单机集群启动
```text
➜  kafka_2.11-2.0.0 nohup bin/kafka-server-start.sh config/server1.properties > k1.log 2>&1 &
[1] 7579
➜  kafka_2.11-2.0.0 nohup bin/kafka-server-start.sh config/server2.properties > k2.log 2>&1 &
[2] 7952
➜  kafka_2.11-2.0.0 nohup bin/kafka-server-start.sh config/server3.properties > k3.log 2>&1 &
[3] 8296
```
检查下jps：
```text
➜  kafka_2.11-2.0.0 jps
7952 Kafka
29873 QuorumPeerMain
1490 ZooKeeperMain
8580 Jps
8296 Kafka
29914 QuorumPeerMain
29963 QuorumPeerMain
7579 Kafka
```


## 3.2、去zk检查kafka是否启动
进入zk：
```text
$ bin/zkCli.sh 
Connecting to localhost:2181
2021-08-14 19:40:45,315 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.12-e5259e437540f349646870ea94dc2658c4e44b3b, built on 03/27/2018 03:55 GMT
2021-08-14 19:40:45,320 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=localhost.localdomain
2021-08-14 19:40:45,320 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_292
2021-08-14 19:40:45,322 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Private Build
2021-08-14 19:40:45,322 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-8-openjdk-amd64/jre
2021-08-14 19:40:45,322 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../build/classes:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../build/lib/*.jar:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../lib/slf4j-log4j12-1.7.25.jar:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../lib/slf4j-api-1.7.25.jar:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../lib/netty-3.10.6.Final.jar:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../lib/log4j-1.2.17.jar:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../lib/jline-0.9.94.jar:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../lib/audience-annotations-0.5.0.jar:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../zookeeper-3.4.12.jar:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../src/java/lib/*.jar:/home/ubuntu/zk_home/zookeeper-3.4.12/bin/../conf:
2021-08-14 19:40:45,322 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib/x86_64-linux-gnu/jni:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu:/usr/lib/jni:/lib:/usr/lib
2021-08-14 19:40:45,323 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
2021-08-14 19:40:45,323 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2021-08-14 19:40:45,323 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
2021-08-14 19:40:45,323 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
2021-08-14 19:40:45,323 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=4.15.0-118-generic
2021-08-14 19:40:45,323 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=ubuntu
2021-08-14 19:40:45,323 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/home/ubuntu
2021-08-14 19:40:45,324 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/home/ubuntu/zk_home/zookeeper-3.4.12
2021-08-14 19:40:45,325 [myid:] - INFO  [main:ZooKeeper@441] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@67424e82
Welcome to ZooKeeper!
2021-08-14 19:40:45,345 [myid:] - INFO  [main-SendThread(localhost.localdomain:2181):ClientCnxn$SendThread@1028] - Opening socket connection to server localhost.localdomain/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2021-08-14 19:40:45,465 [myid:] - INFO  [main-SendThread(localhost.localdomain:2181):ClientCnxn$SendThread@878] - Socket connection established to localhost.localdomain/127.0.0.1:2181, initiating session
2021-08-14 19:40:45,481 [myid:] - INFO  [main-SendThread(localhost.localdomain:2181):ClientCnxn$SendThread@1302] - Session establishment complete on server localhost.localdomain/127.0.0.1:2181, sessionid = 0x15646eee000d, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] 
```

检查下 /brokers/ids 是否有3个brokerId：
```text
[zk: localhost:2181(CONNECTED) 16] ls /brokers/ids
[0, 1, 2]
[zk: localhost:2181(CONNECTED) 17] 
```

# 4、小试牛刀-创建一个topic

创建topic:
```
➜  kafka_2.11-2.0.0 bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test --replication-factor 1 --partitions 1 
Created topic "test".
```

还记得zk的作用吗？ 管理kafka节点元数据的，它能管理broker，也能管理topic，现在我们去zk查看下topic是否创建：
```text
[zk: localhost:2181(CONNECTED) 0] ls /brokers/topics
[test]
```

用zk命令查看topic：
```
➜  kafka_2.11-2.0.0 bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test 
Topic:test	PartitionCount:1	ReplicationFactor:1	Configs:
	Topic: test	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
```
