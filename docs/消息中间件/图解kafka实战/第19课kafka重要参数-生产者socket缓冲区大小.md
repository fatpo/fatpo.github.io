# 1、生产者接收socket消息缓冲区大小
`receive.buffer.bytes` : 

这个参数用来设置 Socket 发送消息缓冲区（SO_SNDBUF）的大小，默认值为 32768（B），即`32KB`。

与 receive.buffer.bytes 参数一样，如果设置为-1，则使用操作系统的默认值。

```
The size of the TCP receive buffer (SO_RCVBUF) to use when reading data. 

If the value is -1, the OS default will be used.
```


# 2、生产者发送socket消息缓冲区大小
`send.buffer.bytes`: 

这个参数用来设置 Socket 接收消息缓冲区（SO_RECBUF）的大小，默认值为 131072（B），即`128KB`。

如果设置为-1，则使用操作系统的默认值。

如果 Producer 与 Kafka 处于不同的机房，则可以适地调大这个参数值。
