# 1、ack是似曾相似的配置
不知道是不是觉得在哪里见过类似的设定：
* ack = 0， 不管发送成功与否，ack 设置为0可以达到最大的吞吐量
* ack = 1， leader副本写入数据后，就算是发送成功，不管其他副本
* ack = -1或all， 全部副本接受完毕，才算写入kafka成功，ack 设置为-1可以达到最强的可靠性

是否有额外的参数配置，比如ack可不可以等于2？这种配置我总感觉在哪里见过，是哪个中间件呢，比如：zk？rabbitmq？
* ack = 2， leader副本和1个follower副本写入成功即可，不管其他副本
* ack = 3， leader副本和2个follower副本写入成功即可，不管其他副本


# 2、ack可能出现的面试题
## 2.1、kafka如何保证宕机的时候数据不丢失？
`多副本机制`是任何一个优秀的分布式系统都要具备的功能，假设一个topic有3个分区，每个分区有2个副本。

万一某台kafka的broker宕机了，没关系的，如果宕机的broker里面有leader副本，那么剩下的机器保存有follower副本，可以选举成为leader副本，继续向外提供服务。

这样的多副本冗余机制，可以保证任何一台机器挂掉，都不会导致数据彻底丢失，因为起码还是有副本在别的机器上的。

## 2.2、多副本之间数据如何同步？
leader副本对外提供写请求，leader副本保持最新数据。

当一个partitioner要写入数据时，都是往leader副本写数据。

follower副本会不断向leader副本发起同步请求，`尝试` 拉取最新数据。

## 2.3、ISR是什么？
众所周知，leader副本是保持最新数据的副本。

所有和leader副本同步完成的follower副本+leader副本本身，都算是`ISR` = In-Sync Replicas，已同步副本。

因为客观原因，follower副本可能有网络延迟？可能JVM遇到stop the world？总之就是有可能在`规定时间内`没同步成功，那么就会移除ISR，进入`OSR(Out-Sync Replicas)`。

## 2.4、ack三种参数是什么?
* ack = 0， 不管发送成功与否，ack 设置为0可以达到最大的吞吐量
* ack = 1， `kafka默认配置`，leader副本写入数据后，就算是发送成功，不管其他副本是否同步
* ack = -1或all， 全部副本接受完毕，才算写入kafka成功，ack 设置为-1可以达到最强的可靠性。

## 2.5、ack=-1是否一定不会丢数据?
答案：不是的。

因为可能ISR里面只有一个leader副本，它没有备份，所以leader副本收到消息后还没处理就宕机了，数据也会丢失。


## 2.6、ack=-1可能造成数据重复，是真的吗？

ack=-1（all）：producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。

但是如果在follower同步完成后，broker发送ack之前，leader发生故障，导致没有返回ack给Producer，

由于失败重试机制，又会给新选举出来的leader发送数据，造成数据重复。

## 2.7、ack=1可能造成数据丢失，是真的吗？

ack=1：producer等待broker的ack，partition的leader落盘成功后返回ack。

如果在follower同步成功之前leader故障，而由于已经返回了ack，系统默认新选举的leader已经有了数据，从而不会进行失败重试，那么将会丢失数据。


# 3、参考
[横趟！面试中遇到的 ZooKeeper 问题](https://www.mdnice.com/writing/fa3d3ba2607e44ef857a3954537232bb)