# 1、消息实体类
```java
public class ProducerRecord<K, V> {
    private final String topic; //主题
    private final Integer partition; //分区号
    private final Headers headers; //消息头部
    private final K key; //键
    private final V value; //值
    private final Long timestamp; //消息的时间戳
    //省略其他成员方法和构造方法
}
```

实际上，我认为上述的属性都非常有必要了解下：
* topic 主题
* partition 主题下第几个分区
* value 消息body
* key 分到主题下哪个分区可以随机，也可以指定，指定用分区器，分区器用到的是key
* headers kafka有大用，后续介绍
* timestamp kafka日志要做顺序排序

# 2、消息实体类的构造函数
```java
class ProducerRecord{
    public ProducerRecord(String topic, Integer partition, Long timestamp, 
                          K key, V value, Iterable<Header> headers);
    public ProducerRecord(String topic, Integer partition, Long timestamp,
                          K key, V value);
    public ProducerRecord(String topic, Integer partition, K key, V value, 
                          Iterable<Header> headers);
    public ProducerRecord(String topic, Integer partition, K key, V value);
    public ProducerRecord(String topic, K key, V value);
    public ProducerRecord(String topic, V value);
}
```
实际上，前面java生产者客户端无论是[初版](https://github.com/fatpo/kafka-demo/blob/main/kakfa/src/main/java/ProducerDemo.java)还是[规范版](https://github.com/fatpo/kafka-demo/blob/main/kakfa/src/main/java/ProducerDemoV2.java)，都是用到了
```
public ProducerRecord(String topic, V value);
```
最简单的发送消息模式，什么分区，什么时间戳，什么分区器key，暂时玩蛋去，用到再说。

# 3、发送消息的2种模式
```java
class KafkaProducer{
    public Future<RecordMetadata> send(ProducerRecord<K, V> record);

    public Future<RecordMetadata> send(ProducerRecord<K, V> record, 
                                   Callback callback);
}
```

## 3.1、同步
```java
class Demo{
    public static void main(String[] args) {
        try {
            producer.send(record).get();
        } catch (ExecutionException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## 3.2、异步-自己拿
```java
class Demo{
    public static void main(String[] args) {
        try {
            Future<RecordMetadata> future = producer.send(record);
            RecordMetadata metadata = future.get();
            System.out.println(metadata.topic() + "-" +
                    metadata.partition() + ":" + metadata.offset());
        } catch (ExecutionException | InterruptedException e) {
            e.printStackTrace();
        }
    }   
}
```
这种异步要结合业务，在合适的时机，get()即可，返回`RecordMetadata`，这是一个存放了`topic`,`partition`,`offset`的结构体。

但是异步代码，`合适时机`很难把握，你怎么知道它什么时候准备好数据？不知道的。

最好是让它来通知你。

## 3.3、异步-callback

```java

class Demo{
    public static void main(String[] args) {
        producer.send(record, new Callback() {
            @Override
            public void onCompletion(RecordMetadata metadata, Exception exception) {
                if (exception != null) {
                    exception.printStackTrace();
                } else {
                    System.out.println(metadata.topic() + "-" +
                            metadata.partition() + ":" + metadata.offset());
                }
            }
        });
    }   
}
```

实际上，根据国哥的经验，在异步处理上，用callback比future要可读性好，稍微绕一下而已。

# 4、代码示例
完整代码如下：
```java
import org.apache.kafka.clients.producer.*;

import java.util.Properties;
import java.util.concurrent.Future;

public class ProducerDemoV4Async {
    public static final String brokerList = "1.116.156.79:9092,1.116.156.79:9093,1.116.156.79:9094";
    public static final String topic = "test2";

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

    public static void main(String[] args) {
        Properties properties = initConfig();

        System.out.println("send a message: hello, Kafka!");
        KafkaProducer<String, String> producer =
                new KafkaProducer<>(properties);
        ProducerRecord<String, String> record =
                new ProducerRecord<>(topic, "hello, Kafka!");
        // callback方式
        try {
            producer.send(record, new Callback() {
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception != null) {
                        exception.printStackTrace();
                    } else {
                        System.out.println("[send callback -1] topic: " + metadata.topic() + ", partition: " +
                                metadata.partition() + ", offset: " + metadata.offset());
                    }
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }

        // future方式
        try {
            Future<RecordMetadata> future = producer.send(record);
            RecordMetadata metadata = future.get();
            System.out.println("[send callback -2] topic: " + metadata.topic() + ", partition: " +
                    metadata.partition() + ", offset: " + metadata.offset());
        } catch (Exception e) {
            e.printStackTrace();
        }
        producer.close();
    }
}
```