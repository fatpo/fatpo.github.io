# 1、重试次数 retries

## 1.1、重试而不是抛给应用程序
retries 参数用来配置生产者重试的次数，默认值为0，即在发生异常的时候不进行任何重试动作。

消息在从生产者发出到成功写入服务器之前可能发生一些临时性的异常，比如网络抖动、leader 副本的选举等，
这种异常往往是可以自行恢复的，生产者可以通过配置 retries 大于0的值，以此通过内部重试来恢复而不是一味地将异常抛给生产者的应用程序。

如果重试达到设定的次数，那么生产者就会放弃重试并返回异常。

## 1.2、不能重试的场景
不过并不是所有的异常都是可以通过重试来解决的，比如消息太大，超过 `max.request.size` 参数配置的值时，这种方式就不可行了。
 

# 2、重试间隔 retry.backoff.ms

重试还和另一个参数 `retry.backoff.ms` 有关，这个参数的默认值为`100`，它用来设定两次重试之间的时间间隔，避免无效的频繁重试。

在配置 `retries` 和 `retry.backoff.ms` 之前，最好先估算一下可能的异常恢复时间，这样可以设定总的重试时间大于这个异常恢复时间，以此来`避免生产者过早地放弃重试`。

# 3、重试导致的无序

## 3.1、kakfa是有序的
Kafka 可以保证同一个分区中的消息是有序的。

如果生产者按照一定的顺序发送消息，那么这些消息也会顺序地写入分区，进而消费者也可以按照同样的顺序消费它们。

对于某些应用来说，顺序性非常重要，比如 MySQL 的 binlog 传输，如果出现错误就会造成非常严重的后果。

## 3.2、max.in.flight.requests.per.connection
在说乱序之前，国哥带你们先看一个生产者重要参数：`max.in.flight.requests.per.connection`: 
```text
The maximum number of unacknowledged requests the client will send on a single connection before blocking. 

Note that if this setting is set to be greater than 1 and there are failed sends, 
there is a risk of message re-ordering due to retries (i.e., if retries are enabled).
```
稍微翻译下就是：`允许还没收到ack的消息个数`，如果大于1，可能和会乱序哦，你想想看为啥？消息1失败了，消息2成功，消息1重发，是不是就乱了？

## 3.3、乱序解决方案

如果将 retries 参数配置为非零值，并且 `max.in.flight.requests.per.connection` 参数配置为大于1的值，那么就会出现错序的现象：
* 如果第一批次消息写入失败，而第二批次消息写入成功，那么生产者会重试发送第一批次的消息，
* 此时如果第一批次的消息写入成功，那么这两个批次的消息就出现了错序。

一般而言，在需要保证消息顺序的场合建议把参数 `max.in.flight.requests.per.connection 配置为1`，而不是把 retries 配置为0，不过这样也会影响整体的吞吐。
