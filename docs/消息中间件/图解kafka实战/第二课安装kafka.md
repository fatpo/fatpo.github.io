# 安装zookeeper
自己去百度安装
```
zookeeper-3.4.12.tar.gz 
```

启动zk:
```
> zkServer.sh start
JMX enabled by default
Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

查看状态：
```
> zkServer.sh status
JMX enabled by default
Using config: /opt/zookeeper-3.4.12/bin/../conf/zoo.cfg
Mode: Standalone
```


# 安装kafka
自己去百度安装
```
kafka_2.11-2.0.0.tgz 
```

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
log.dirs=/tmp/kafka-logs/1
```
config/server2.properties关键配置:
```
broker.id=1
listeners=PLAINTEXT://localhost:9093
log.dirs=/tmp/kafka-logs/2
```
config/server3.properties关键配置:
```
broker.id=2
listeners=PLAINTEXT://localhost:9094
log.dirs=/tmp/kafka-logs/3
```


启动3个kafka：
```
➜  kafka_2.11-2.0.0 bin/kafka-server-start.sh config/server1.properties &
➜  kafka_2.11-2.0.0 bin/kafka-server-start.sh config/server2.properties &
➜  kafka_2.11-2.0.0 bin/kafka-server-start.sh config/server3.properties &
```

# 创建topic
```
➜  kafka_2.11-2.0.0 bin/kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic topic-demo --replication-factor 3 --partitions 4

Created topic "topic-demo".
```
# 查看topic
```
➜  kafka_2.11-2.0.0 bin/kafka-topics.sh --zookeeper localhost:2181/kafka --describe --topic topic-demo
Topic:topic-demo	PartitionCount:4	ReplicationFactor:3	Configs:
	Topic: topic-demo	Partition: 0	Leader: 2	Replicas: 2,0,1	Isr: 2,0,1
	Topic: topic-demo	Partition: 1	Leader: 0	Replicas: 0,1,2	Isr: 0,1,2
	Topic: topic-demo	Partition: 2	Leader: 1	Replicas: 1,2,0	Isr: 1,2,0
	Topic: topic-demo	Partition: 3	Leader: 2	Replicas: 2,1,0	Isr: 2,1,0
```

# 生产消息
```
➜  kafka_2.11-2.0.0 bin/kafka-console-producer.sh --broker-list localhost:9092 --topic topic-demo
>hello
>fatpo
>
```

# 消费消息
```
➜  kafka_2.11-2.0.0 bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic-demo
hello
fatpo
```