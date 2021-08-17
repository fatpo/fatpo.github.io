# 1、git代码库

* [ssh git](git@github.com:fatpo/kafka-demo.git)
* [http git](https://github.com/fatpo/kafka-demo.git)
* [生产者客户端代码](https://github.com/fatpo/kafka-demo/blob/main/kakfa/src/main/java/ProducerDemoV2.java)


# 2、java生产者客户端正规化版本
改动点：
* 配置初始化正规，多一个 `initConfig()`
* 用kafka的常量`ProducerConfig.BOOTSTRAP_SERVERS_CONFIG`、`ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG`、`ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG`做Key减少人为失误
* 可以配置`client.id`，这是表示生产者客户端一个标识iD，如果不设置：KafkaProducer会自动生成一个非空字符串，内容形式如`producer-1`，`producer-2`

代码如下：
```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class ProducerDemoV2 {
    public static final String brokerList = "1.116.156.79:9092";
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
        return properties;
    }

    public static void main(String[] args) {
        Properties properties = initConfig();

        System.out.println("send a message: hello, Kafka!");
        KafkaProducer<String, String> producer =
                new KafkaProducer<>(properties);
        ProducerRecord<String, String> record =
                new ProducerRecord<>(topic, "hello, Kafka!");
        try {
            producer.send(record);
        } catch (Exception e) {
            e.printStackTrace();
        }
        producer.close();
    }
}
```