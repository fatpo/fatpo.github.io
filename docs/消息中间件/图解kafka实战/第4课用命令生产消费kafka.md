# 1、创建一个topic
```text
➜  kafka_2.11-2.0.0 bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test2 --replication-factor 3 --partitions 4
Created topic "test2".
```
检查下是否成功：
```text
➜  kafka_2.11-2.0.0 bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test2                                    
Topic:test2	PartitionCount:4	ReplicationFactor:3	Configs:
	Topic: test2	Partition: 0	Leader: 2	Replicas: 2,0,1	Isr: 2,0,1
	Topic: test2	Partition: 1	Leader: 0	Replicas: 0,1,2	Isr: 0,1,2
	Topic: test2	Partition: 2	Leader: 1	Replicas: 1,2,0	Isr: 1,2,0
	Topic: test2	Partition: 3	Leader: 2	Replicas: 2,1,0	Isr: 2,1,0
```

# 2、消费者监听
```text
?  kafka_2.11-2.0.0 bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test2

```
此时此刻可以看到，命令处于挂起状态。

# 3、生产者生产
```text
➜  kafka_2.11-2.0.0 bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test2  
>
```
此时此刻可以看到，命令处于挂起状态。

随便输入点什么：
```text
➜  kafka_2.11-2.0.0 bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test2  
>hello
>world

```

# 4、消费者消费
```text
➜  kafka_2.11-2.0.0 bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test2
hello
world

```