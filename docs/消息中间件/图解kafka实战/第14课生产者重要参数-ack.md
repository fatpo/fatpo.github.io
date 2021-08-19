# 1、似曾相似
不知道是不是觉得在哪里见过类似的设定：
* ack = 0， 不管发送成功与否，acks 设置为0可以达到最大的吞吐量
* ack = 1， leader副本写入数据后，就算是发送成功，不管其他副本
* ack = -1或all， 全部副本接受完毕，才算写入kafka成功，acks 设置为-1可以达到最强的可靠性

或者ack可不可以等于2？比如：
* ack = 2， leader副本和1个follower副本写入成功即可，不管其他副本

# 2、ack=-1可能造成数据重复

-1（all）：producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。

但是如果在follower同步完成后，broker发送ack之前，leader发生故障，导致没有返回ack给Producer，

由于失败重试机制，又会给新选举出来的leader发送数据，造成数据重复。

# 3、ack=1可能造成数据丢失

1：producer等待broker的ack，partition的leader落盘成功后返回ack。

如果在follower同步成功之前leader故障，而由于已经返回了ack，系统默认新选举的leader已经有了数据，从而不会进行失败重试，那么将会丢失数据。


to be continued...

# 4、参考
[横趟！面试中遇到的 ZooKeeper 问题](https://www.mdnice.com/writing/fa3d3ba2607e44ef857a3954537232bb)