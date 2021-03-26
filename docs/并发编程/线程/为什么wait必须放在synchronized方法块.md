## 背景
`object.wait()`正常用法：
```java
public class A {
    public static void main(String[] args) throws InterruptedException {
        Object obj = new Object();
        synchronized (obj) {
            obj.wait();
        }
    }
}
```
`object.wait()`不在同步块:
```java
public class A {
    public static void main(String[] args) throws InterruptedException {
        Object obj = new Object();
        obj.wait();
    }
}
```
会报错：
```
Exception in thread "main" java.lang.IllegalMonitorStateException
	at java.lang.Object.wait(Native Method)
	at java.lang.Object.wait(Object.java:502)
	at A.main(A.java:5)
```

---

`object.wait()` 即使在同步块，但是临界资源不是同一个对象:
```java
public class A {
    public static void main(String[] args) throws InterruptedException {
        Object obj = new Object();
        Object obj2 = new Object();
        synchronized (obj) {
            obj2.wait();
        }
    }
}
```
会报错：
```
Exception in thread "main" java.lang.IllegalMonitorStateException
	at java.lang.Object.wait(Native Method)
	at java.lang.Object.wait(Object.java:502)
	at A.main(A.java:7)
```
## 思考：为什么wait必须放在synchronized？

是的，我们一直知道 `object.wait()`, `object.notify()` , `object.notifyAll()` 需要在同步块中，否则会报错。
但好像之前从来未想过为啥？

其实答案就在 Javadoc 文档中，点开wait()函数，Javadoc有一堆解释：
```
  /**
     * Causes the current thread to wait until either another thread invokes the
     * {@link java.lang.Object#notify()} method or the
     * {@link java.lang.Object#notifyAll()} method for this object, or a
     * specified amount of time has elapsed.
     * <p>
     * The current thread must own this object's monitor.
     * <p>
     * This method causes the current thread (call it <var>T</var>) to
     * place itself in the wait set for this object and then to relinquish
     * any and all synchronization claims on this object. Thread <var>T</var>
     * becomes disabled for thread scheduling purposes and lies dormant
     * until one of four things happens:
     * <ul>
     * <li>Some other thread invokes the {@code notify} method for this
     * object and thread <var>T</var> happens to be arbitrarily chosen as
     * the thread to be awakened.
     * <li>Some other thread invokes the {@code notifyAll} method for this
     * object.
     * <li>Some other thread {@linkplain Thread#interrupt() interrupts}
     * thread <var>T</var>.
     * <li>The specified amount of real time has elapsed, more or less.  If
     * {@code timeout} is zero, however, then real time is not taken into
     * consideration and the thread simply waits until notified.
     * </ul>
     * The thread <var>T</var> is then removed from the wait set for this
     * object and re-enabled for thread scheduling. It then competes in the
     * usual manner with other threads for the right to synchronize on the
     * object; once it has gained control of the object, all its
     * synchronization claims on the object are restored to the status quo
     * ante - that is, to the situation as of the time that the {@code wait}
     * method was invoked. Thread <var>T</var> then returns from the
     * invocation of the {@code wait} method. Thus, on return from the
     * {@code wait} method, the synchronization state of the object and of
     * thread {@code T} is exactly as it was when the {@code wait} method
     * was invoked.
     * <p>
     * A thread can also wake up without being notified, interrupted, or
     * timing out, a so-called <i>spurious wakeup</i>.  While this will rarely
     * occur in practice, applications must guard against it by testing for
     * the condition that should have caused the thread to be awakened, and
     * continuing to wait if the condition is not satisfied.  In other words,
     * waits should always occur in loops, like this one:
     * <pre>
     *     synchronized (obj) {
     *         while (&lt;condition does not hold&gt;)
     *             obj.wait(timeout);
     *         ... // Perform action appropriate to condition
     *     }
     * </pre>
     * (For more information on this topic, see Section 3.2.3 in Doug Lea's
     * "Concurrent Programming in Java (Second Edition)" (Addison-Wesley,
     * 2000), or Item 50 in Joshua Bloch's "Effective Java Programming
     * Language Guide" (Addison-Wesley, 2001).
     *
     * <p>If the current thread is {@linkplain java.lang.Thread#interrupt()
     * interrupted} by any thread before or while it is waiting, then an
     * {@code InterruptedException} is thrown.  This exception is not
     * thrown until the lock status of this object has been restored as
     * described above.
     *
     * <p>
     * Note that the {@code wait} method, as it places the current thread
     * into the wait set for this object, unlocks only this object; any
     * other objects on which the current thread may be synchronized remain
     * locked while the thread waits.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @param      timeout   the maximum time to wait in milliseconds.
     * @throws  IllegalArgumentException      if the value of timeout is
     *               negative.
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of the object's monitor.
     * @throws  InterruptedException if any thread interrupted the
     *             current thread before or while the current thread
     *             was waiting for a notification.  The <i>interrupted
     *             status</i> of the current thread is cleared when
     *             this exception is thrown.
     * @see        java.lang.Object#notify()
     * @see        java.lang.Object#notifyAll()
     */
    public final native void wait(long timeout) throws InterruptedException;
```
小编有话说：
```
强烈建议阅读英文doc
```
首先我们要知道这个wait()函数要干嘛：
```
Causes the current thread to wait until either another thread invokes the
     * {@link java.lang.Object#notify()} method or the
     * {@link java.lang.Object#notifyAll()} method for this object, or a
     * specified amount of time has elapsed.
```
就是说：
```
要让当前线程进入等待状态，直到别的线程调用object.notify()或者object.notifyAll()来唤醒，或者时间到了也要被唤醒。
```
然后wait()必须拥有object的`monitor`:
```
The current thread must own this object's monitor.
```
拥有`object's monitor`是为了能失去`object's monitor`：
```
The thread
     * releases ownership of this monitor and waits until another thread
     * notifies threads waiting on this object's monitor to wake up
     * either through a call to the {@code notify} method or the
     * {@code notifyAll} method. 
```
没错，关键词就是`releases ownership`，释放`object's monitor`的拥有权。

--- 
再想到`synchronized`同步块的意义是：
```
让当前线程拥有临界资源（object）的锁（monitor）。
```

至此，基本上`为什么object.wait()非要在同步块`这个问题的答案就出来：
```
只有在同步块中，才能保证当前线程拥有 object.monitor，才能在wait()函数中失去这个 object.monitor。
```
否则：
```
你都不曾拥有，谈何失去？
```

## 再思考：jdk规定的这种object monitor机制是为了什么
我们知道 `object.wait()` 和 `object.notify()` 底层就是对`object monitor`的`获取`、`失去`的操作。

这么做有个好处，就是能防止`Lost Wake-Up Problem` !

什么是`Lost Wake-Up Problem`: 
```
线程休眠后，再也无法唤醒的问题。
```
国哥给你模拟一波`Lost Wake-Up Problem`：
写一个生产者线程：
```
count++;
object.notify();
```
写一个消费者线程：
```
while(count<=0){
    object.wait();
    count--;
}
```
如果不能保证`object's momitor`的先获取再失去，那么生产者、消费者两个线程代码执行顺序可能是：
```
生产者：count++;
消费者：while(count<=0){
生产者：object.notify();
消费者：   object.wait();
消费者：    count--;
消费者：}
```
生产者线程唤醒了一个还没休眠的消费者线程，导致唤醒无效后，消费者继续休眠，那么发生一个很可怕的事：
```
消费者将永远沉睡！
```

---

如果能保证`object's momitor`的先获取再失去，那么生产者、消费者两个线程代码执行顺序只可能是以下情况：

生产者拿到临界资源锁：
```
生产者：count++; // 拥有object's monitor
生产者：object.notify(); // 唤醒消费者线程，但是因为此时没有消费者（只能说这种编码有问题，要么就让消费者先运行）

消费者：while(count <= 0){ // 这里进不去while条件，至少不会消费者永远沉睡。
消费者：   object.wait();
消费者：   count--;
消费者：}
```
消费者拿到临界资源锁，因为count初始化是0，所以消费者线程走到`object.wait()`后让出了`object's monitor`：
```
消费者：while(count<=0){   // 拥有object's monitor
消费者：   object.wait();  // 这里让出了object's monitor

生产者：count++;  // 这里得到了object's monitor
生产者：object.notify(); //这里再唤醒消费者线程，并在执行完毕后，让出object's monitor


消费者：   count--; // 这里得到了object's monitor，继续往下执行
消费者：}
```
所以，`object's monitor`这种`获取`，`失去`机制能保证 `Lost Wake-Up Problem` 问题不复存焉~