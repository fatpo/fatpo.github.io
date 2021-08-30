# 1、request.timeout.ms

生产者等待broker回应acks的最长时间，30000ms，`30s`。

请求超时之后可以选择进行重试。

# 2、retries重试次数

# 3、关于生产者-broker-副本的思考

看到《图解kafka实战》书上写：
```text
这个参数用来配置 Producer 等待请求响应的最长时间，默认值为30000（ms）。

请求超时之后可以选择进行重试。

注意这个参数需要比 broker 端参数 replica.lag.time.max.ms 的值要大，
这样可以减少因客户端重试而引起的消息重复的概率。
```
我不是很理解这句话的意思，但是作者这么写，肯定是遇到了坑后的经验之谈。

先科普下`replica.lag.time.max.ms`参数：
```text
If a follower hasn't sent any fetch requests or hasn't consumed up to the leaders log end offset 
for at least this time, the leader will remove the follower from isr.

如果从副本在规定时间内没有处理任何请求或者没有消费到leader副本的LEO，那么就把这个从副本踢出ISR。
```

|情景|备注|评论|
|:---|---|---|
|情况1| 然后假设我broker副本间复制限制最大限制30s。<br>生产者传输到broker花了1s。<br>副本同步最限制最大10s。<br>副本同步实际花了1s。|这种情况是最佳的，最普通的，相安无事。|
|情况2| 然后假设我broker副本间复制限制最大限制30s。<br>生产者传输到broker花了20s。<br>副本同步最限制最大10s。<br>副本同步实际花了1s。|这种情况是生产者不用重试，follower因为长时间没收到请求被剔出ISR，生产者无感知，等follower消费速度跟上后重加入ISR再复制这条消息。不会重复|
|情况3| 然后假设我broker副本间复制限制最大限制30s。<br>生产者传输到broker花了35s。<br>副本同步最限制最大50s。<br>副本同步实际花了1s。|生产者超过30s阈值，要重试1次，一共发送2条，follower副本没超过时间阈值50s，消费了两条一样的数据。|
|情况4| 然后假设我broker副本间复制限制最大限制30s。<br>生产者传输到broker花了35s。<br>副本同步最限制最大10s。<br>副本同步实际花了1s。|生产者超过30s阈值，要重试1次，一共发送2条，这种情况是最佳的，最普通的，相安无事。|

----

情况一：

然后假设我broker副本间复制限制最大限制30s。

生产者传输到broker花了1s。

副本同步最限制大10s。

副本同步实际花了1s。

这种情况是最佳的，最普通的，相安无事。

----

假设我生产者到broker花了20s，并没有超过30s的限制，所以这是规则允许的，不用重试。

然后假设我broker副本间复制限制最大限制10s。

那么因为生产者到broker已经花了20s了，副本已经20s没有处理请求，超过了10s的最大限制，那么follower副本就会被踢出ISR。
leader副本接受这条信息，等待follower副本跟上消费速度后加入ISR，继续复制这条消息到follower副本。

所以这个

-----

然后假设我broker副本间复制限制最大限制50s。生产者传输到broker花了20s。

那么因为生产者到broker已经花了20s，副本20s没有处理请求，没有超过50s的最大限制，那么follower副本依旧在ISR。

-----

然后假设我broker副本间复制限制最大限制50s。生产者传输到broker花了35s。

那么因为生产者到broker已经花了35s，副本35s没有处理请求，没有超过50s的最大限制，那么follower副本依旧在ISR。
但是生产者发送请求到broker因为超过了30s，所以它会重试，所以就发送了第二条信息到broker，假设这次发送很快就到了。
此时follower副本假设还是在ISR，那么它一共收到了2条一样的命令。



