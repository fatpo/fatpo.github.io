# 1、kafka的异常

## 1.1、可重试

* NetworkException
    表示网络异常，这个有可能是由于网络瞬时故障而导致的异常
* LeaderNotAvailableException
    表示分区的 leader 副本不可用，这个异常通常发生在 leader 副本下线而新的 leader 副本选举完成之前，重试之后可以重新恢复
* UnknownTopicOrPartitionException
    主题分区不认识，可能是刚好在创建，未能同步过来，重试
* NotEnoughReplicasException
    没有足够的副本，可能是在节点调整，重试
* NotCoordinatorException 
    没协调员，可能是选举还没结束，重试

## 1.2、不可重试
不可重试异常：
* RecordTooLargeException  消息body过大，不会重试的
    

# 2、java生产者增加重试配置
主要是初始化配置时增加retry配置：
```java
properties.put(ProducerConfig.RETRIES_CONFIG, 10);
```
初始配置代码：
```java
public static Properties initConfig() {
    Properties properties = new Properties();
    // 诸如“key.serializer”、“max.request.size”、“interceptor.classes”之类的字符串经常由于人为因素而书写错误
    // kafka 帮我们提供了一些constant常量
    properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
    properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
            "org.apache.kafka.common.serialization.StringSerializer");
    properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
            "org.apache.kafka.common.serialization.StringSerializer");
    properties.put(ProducerConfig.CLIENT_ID_CONFIG, "producer.client.id.demo");
    properties.put(ProducerConfig.RETRIES_CONFIG, 10);
    return properties;
}
```

[整的代码点这里](https://github.com/fatpo/kafka-demo/blob/main/kakfa/src/main/java/ProducerDemoV3Retry.java)