## 应用场景

`AQS` 是一个同步并发框架，基于它可以写出很多酷炫的轮子。 阻塞线程`blocking queue`就是其中一个轮子。

`blocking queue`旨在解决线程通信，是解决生产者-消费者问题的最佳利器，由`java.util.concurrent`包提供。

`blocking queue`的应用场景包括：
* 线程池
* rocketmq
* spring-eureka

## 概要

根据队列类型可分为：
* 有界队列
* 无界队列

根据数据结构可分为：
* <b>ArrayBlockingQueue</b> - 基于数组的有界队列
* <b>LinkedBlockingQueue</b> - 基于链表的可选有界队列
* <b>PriorityBlockingQueue</b> - 优先级堆支持的无界优先级队列
* <b>DelayQueue</b> - 由优先级堆支持的、基于时间的调度队列
 
 

## 代码应用

做一个有生产者，消费者的demo，其中生产者不断产生随机数，消费者不断消费之。

可控制队列最大值是5。

```java
import java.util.Random;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class B {

    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);

        int producerCnt = 6;
        int consumerCnt = 4;
        Thread[] producers = new Thread[producerCnt];
        Thread[] consumers = new Thread[consumerCnt];
        for (int i = 0; i < producerCnt; i++) {
            producers[i] = new Thread(new Producer(queue, i));
            producers[i].start();
        }
        for (int i = 0; i < consumerCnt; i++) {
            consumers[i] = new Thread(new Consumer(queue, i));
            consumers[i].start();
        }
    }


}

class Producer implements Runnable {

    private BlockingQueue<Integer> queue;

    private int index;

    public Producer(BlockingQueue<Integer> queue, int index) {
        this.queue = queue;
        this.index = index;
    }

    @Override
    public void run() {
        while (true) {
            // 生产一个数字
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Integer r = new Random().nextInt(100);
            System.out.println("生产者" + index + "生产了：" + r + "， 队列长度：" + queue.size());
            try {
                queue.put(r);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}


class Consumer implements Runnable {

    private BlockingQueue<Integer> queue;

    private int index;

    public Consumer(BlockingQueue<Integer> queue, int index) {
        this.queue = queue;
        this.index = index;
    }

    @Override
    public void run() {
        while (true) {
            try {
                Integer take = queue.take();
                Thread.sleep(1000);
                System.out.println("消费者" + index + "消费了：" + take + "， 队列长度：" + queue.size());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
运行结果：
```dtd
消费者3消费了：33， 队列长度：2
消费者2消费了：21， 队列长度：2
消费者1消费了：16， 队列长度：0
消费者0消费了：41， 队列长度：2
生产者0生产了：64， 队列长度：0
生产者5生产了：75， 队列长度：1
生产者4生产了：44， 队列长度：2
生产者2生产了：40， 队列长度：3
生产者3生产了：33， 队列长度：4
生产者1生产了：5， 队列长度：5
消费者3消费了：30， 队列长度：4
消费者2消费了：86， 队列长度：4
消费者1消费了：64， 队列长度：2
消费者3消费了：44， 队列长度：2
消费者0消费了：75， 队列长度：2
消费者2消费了：40， 队列长度：0
生产者3生产了：57， 队列长度：0
生产者1生产了：72， 队列长度：0
生产者2生产了：17， 队列长度：0
生产者4生产了：35， 队列长度：1
生产者0生产了：76， 队列长度：0
```


## 源码剖析

to be continued...


## 参考链接
[知乎：面试必考AQS-await和signal的实现原理](https://zhuanlan.zhihu.com/p/113493890)
[知乎：面试必考AQS-排它锁的申请与释放](https://zhuanlan.zhihu.com/p/111350666)