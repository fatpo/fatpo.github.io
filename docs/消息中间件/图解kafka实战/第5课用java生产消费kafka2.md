# 1、重试

KafkaProducer 中一般会发生两种类型的异常：可重试的异常和不可重试的异常。

## 1.1、可重试异常
* NetworkException
* LeaderNotAvailableException
* UnknownTopicOrPartitionException
* NotEnoughReplicasException
* NotCoordinatorException 

比如 `NetworkException` 表示网络异常，这个有可能是由于网络瞬时故障而导致的异常，可以通过重试解决；

又比如 `LeaderNotAvailableException` 表示分区的 leader 副本不可用，这个异常通常发生在 leader 副本下线而新的 leader 副本选举完成之前，重试之后可以重新恢复。

## 1.2、不可重试的异常
`RecordTooLargeException` 异常，暗示了所发送的消息太大，KafkaProducer 对此不会进行任何重试，直接抛出异常。

## 1.3、重试的套路
对于可重试的异常，如果配置了 retries 参数，那么只要在规定的重试次数内自行恢复了，就不会抛出异常。retries 参数的默认值为0。

配置方式参考如下：
```
props.put(ProducerConfig.RETRIES_CONFIG, 10);
```

# 2、回调

