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
实际上，无论是初版还是升级版，都是用到了  `public ProducerRecord(String topic, V value);`，最简单的发送消息。

什么分区，什么时间戳，什么分区器key，暂时玩蛋去，用到再说。

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
这种异步要结合业务，在合适的时机，get()即可。

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