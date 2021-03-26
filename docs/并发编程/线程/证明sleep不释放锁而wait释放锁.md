# 写在前面
建议先看 [课程二：Java线程状态转移图](https://fatpo.github.io/#/并发编程/线程/课程二：Java线程状态转移图) ，否则您可能会对sleep和wait没什么概念。
```
Thread.sleep(long) vs object.wait()
```
为了方便讨论，我们把下面的代码称之为临界方法区：
```
 synchronized (object) {
     // sth code
 }
```
小编有话说：
```
临界方法区不用我多说吧，嘿嘿，就是同一时间只能进一个线程的代码块（类似马桶同时只能有一个人上，吧？）
```

---
# 思考：如何证明sleep没有释放锁，而wait释放锁了

小编有话说：
```
不同于传统面试题，问sleep释放锁了吗？wait释放锁了吗？
而是直接告诉你答案，但是让你去证明它，如果你理解够深的，是可以通过小demo来实现的。
```
这里先说一下锁和线程的关系：
```
锁：   一般是object，对象来着，对象头markword有4字节，比如偏向锁时markword有23bit表示线程ID。
线程： 你可以理解为多个少年，抢夺一个貌美如画的少女，少女就是锁，少年就是线程。
```


## 先证明 object.wait() 可以释放锁
```java
public class A {

    public static void main(String[] args) throws InterruptedException {
        Object object = new Object();
        long start = System.currentTimeMillis();

        Thread t1 = new Thread() {
            @Override
            public void run() {
                synchronized (object) {
                    // 执行一段复杂的操作，使得线程2迟迟拿不到锁，进不了临界方法区
                    int cnt = 0;
                    for (int i = 0; i < 100000000; i++) {
                        cnt++;
                        cnt %= 121312;
                    }
                    System.out.println("线程1：在执行前半段代码，cnt：" + cnt);

                    try {
                        object.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("线程1：在执行后半段代码");
                }
            }
        };
        t1.start();

        Thread t2 = new Thread() {
            @Override
            public void run() {
                System.out.println("线程2：等待进入临界方法区！！！" + (System.currentTimeMillis() - start) + " ms");
                synchronized (object) {
                    System.out.println("线程2：终于进入临界区，之前等待了锁：" + (System.currentTimeMillis() - start) + " ms");
                    System.out.println("线程2：发出信号，让线程1继续执行...");
                    object.notify();
                }
                System.out.println("线程2：离开临界方法区！！！整个过程花了：" + (System.currentTimeMillis() - start) + " ms");
            }
        };
        t2.start();

        t1.join();
        t2.join();
    }

}
```
运行结果：
```
线程2：等待进入临界方法区！！！1 ms
线程1：在执行前半段代码，cnt：38912
线程2：终于进入临界区，之前等待了锁：526 ms
线程2：发出信号，让线程1继续执行...
线程2：离开临界方法区！！！整个过程花了：526 ms
线程1：在执行后半段代码
```
小编有话说：
```
两个线程都是 synchronized (object){} ，能看到线程t1和t2抢夺同一个锁：object！
```
因为t1执行比较早，所以先拿到object锁，导致t2拿不到锁，t2处于BLOCKED状态，被挡在`临界方法区`，挡了：
```
526ms
```
然后t1执行一半，调用了object.wait()，让出了CPU，<b>并且释放了锁，否则t2进不去临界方法区</b>：
```
线程1：在执行前半段代码，cnt：38912
线程2：终于进入临界区，之前等待了锁：526 ms
```
最后，t2完事后调用了object.notify()，唤醒线程t1，使得t1重新进入就绪队列。

因为就绪队列就线程t1一个，线程t1再被调度进入RUNNING状态，表面上看起来就是线程t1继续执行了后半部分代码：
```
线程1：在执行后半段代码
```

## 再证明 Thread.sleep(long) 不释放锁
```java
public class A {

    public static void main(String[] args) throws InterruptedException {
        Object object = new Object();
        long start = System.currentTimeMillis();

        Thread t1 = new Thread() {
            @Override
            public void run() {
                synchronized (object) {
                    // 执行一段复杂的操作，使得线程2迟迟拿不到锁，进不了临界方法区
                    int cnt = 0;
                    for (int i = 0; i < 100000000; i++) {
                        cnt++;
                        cnt %= 121312;
                    }
                    System.out.println("线程1：在执行前半段代码，cnt：" + cnt);

                    try {
                        Thread.sleep(10000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("线程1：在执行后半段代码");
                }
            }
        };
        t1.start();

        Thread t2 = new Thread() {
            @Override
            public void run() {
                System.out.println("线程2：等待进入临界方法区！！！" + (System.currentTimeMillis() - start) + " ms");
                synchronized (object) {
                    System.out.println("线程2：终于进入临界区，之前等待了锁：" + (System.currentTimeMillis() - start) + " ms");
                    System.out.println("线程2：发出信号，让线程1继续执行...");
                    object.notify();
                }
                System.out.println("线程2：离开临界方法区！！！整个过程花了：" + (System.currentTimeMillis() - start) + " ms");
            }
        };
        t2.start();

        t1.join();
        t2.join();
    }

}

```
运行结果：
```
线程2：等待进入临界方法区！！！3 ms
线程1：在执行前半段代码，cnt：38912
线程1：在执行后半段代码
线程2：终于进入临界区，之前等待了锁：10507 ms
线程2：发出信号，让线程1继续执行...
线程2：离开临界方法区！！！整个过程花了：10507 ms
```
代码几乎不变，object.wait()替换成Thread.sleep(1000)，我们让线程t1睡10秒：
```
Thread.sleep(10000);
```
能明显看到，线程1完全执行完毕后才执行线程2，线程2全程被阻塞在临界方法区门口：
```
线程2：终于进入临界区，之前等待了锁：10507 ms
```
小编有话说：
```
线程t2真可怜，人家线程t1就是让出CPU,进入TIMED_WAITING状态，也轮不到你玩。

毕竟锁未释放，你进不去！进不去就是进不去。
```