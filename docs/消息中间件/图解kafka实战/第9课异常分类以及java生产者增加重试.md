# 1、kafka的异常

## 1.1、可重试

 * NetworkException
    表示网络异常，这个有可能是由于网络瞬时故障而导致的异常
 * LeaderNotAvailableException
    表示分区的 leader 副本不可用，这个异常通常发生在 leader 副本下线而新的 leader 副本选举完成之前，重试之后可以重新恢复
 * UnknownTopicOrPartitionException
    主题分区不认识，可能是刚好在创建，未能同步过来，重试
 * NotEnoughReplicasException
    * 没有足够的副本，可能是在节点调整，重试
 * NotCoordinatorException 
    * 没协调员，可能是选举还没结束，重试

## 1.2、不可重试
 * RecordTooLargeException 
    消息body过大，不会重试的
    
# 2、java生产者增加重试配置
```

```