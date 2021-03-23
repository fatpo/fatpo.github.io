# 写在前面
建议先看 [课程一：Linux操作系统线程状态转移图](https://fatpo.github.io/#/并发编程/线程/课程一：Linux操作系统线程状态转移图) ，否则您可能会对下面的几个状态感到懵逼~

特别是 JVM的 RUNNABLE 和 操作系统里面的 RUNNING + READY 的关系。

---

# 枚举java线程状态

- <b>NEW</b> - this state represents a new thread which is not yet started.
- <b>RUNNABLE</b> - this state represents a thread which is executing in the underlying JVM. Here executing in JVM doesn't mean that the thread is always executing in the OS as well - it may wait for a resource from the Operating system like the processor while being in this state.
- <b>BLOCKED</b> - this state represents a thread which has been blocked and is waiting for a moniotor to enter/re-enter a synchronized block/method. A thread gets into this state after calling Object.wait method.
- <b>WAITING</b> - this state represnts a thread in the waiting state and this wait is over only when some other thread performs some appropriate action. A thread can get into this state either by calling - Object.wait (without timeout), Thread.join (without timeout), or LockSupport.park methods.
- <b>TIMED_WAITING</b> - this state represents a thread which is required to wait at max for a specified time limit. A thread can get into this state by calling either of these methods: Thread.sleep, Object.wait (with timeout specified), Thread.join (with timeout specified), LockSupport.parkNanos, LockSupport.parkUntil
- <b>TERMINATED</b> - this state reprents a thread which has completed its execution either by returning from the run() method after completing the execution OR by throwing an exception which propagated from the run() method and hence caused the termination of the thread.

小编有话说:
```
这一段请务必看英文doc，很多中文资料翻译都怪怪的。
```

---
这里有意思的点是 RUNNABLE 它并不是表示线程一直在占用处理器执行：
```
doesn't mean that the thread is always executing in the OS as well
```
它还能表示等待某些资源，如等待CPU调度，等待IO响应：
```
it may wait for a resource from the Operating system like the processor
```
所以JAVA竟然把非WAITING和非BLOCKED的线程状态都当成RUNNABLE了，这里和操作系统的线程状态，还真的不一样呢！（具体哪里不一样，看后续~）

---

NEW 和 TERMINATED 就不说了，一个开始一个结束，但是有些朋友可能会对WAITING 和 BLOCKED 表示疑惑：
```
看起来两个都是在等sth ？ 那么它们的区别是啥呢？
```
具体区别看后续的状态转移图~

# 状态转移图


![](../imgs/2021-03-23-Java线程转移图.png)

---

看了状态转移图后，这里解答下WAITING (TIMED_WAITING同理，我们就不赘言) 和 BLOCKED的区别。
先说为啥有人会问两者的区别：
```
他们觉得这2个状态很像，所以容易迷惑。
```
为啥这2个状态很像：
```
都是在等待着什么。
```
但如果看懂状态转移图后，你就会发现，其实两者内在逻辑是完全不同的：
```
WAITING: 业务逻辑层面的调度（程序员可以控制），理论上来说等一个信号的触发，可以等待很久。

BLOCKED: 因为并发临界资源的争夺，导致只有一个线程能占用资源，其它线程只能干等着，它巴不得能马上占用处理器，越快越好。
         不会有程序员专门让一个线程来BLOCKED住，程序员巴不得马上、立刻都给爷执行完毕，所以无法控制。
         严格上来说，一般并发程度不高的话，BLOCKED的时间不会太久。
```