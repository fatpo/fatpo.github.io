# 1、背景
偶然看到《Java并发编程实战》关于多线程的3个问题：
* 安全性问题
* 活跃性问题
* 性能问题

作者讲解极好，可以记录。

# 2、安全性问题
经典问题，扣除库存：
```
public int decrement(){
 return --count;//count初始库存为10
}
```
多个线程同时操作，可以触发超发超卖，这是不可允许的。

安全性保证方案：
* 使用synchronize内置锁
* ReentrantLock显式锁的加锁机制
* 使用线程安全的原子类
    * 不安全：
        * ArrayList、HashMap等
            * 调用迭代器的next()方法获取下一个元素时，会先通过checkForComodification()方法检查modCount和expectedModCount是否相等，若不相等则抛出ConcurrentModificationException
            * 每次调用add或remove等方法都会使modCount加1
            * expectedModCount是迭代器的属性，在迭代器实例创建时被赋与和遍历前modCount相等的值
        * SimpleDateFormat，内部代码没有对共享变量进行同步
            * 要么SimpleDateFormat + ThreadLocal搭配使用
            * 要么就使用更安全更便捷的：Java8的LocalDateTime和DateTimeFormatter
    * 安全：
        * CopyOnWriteArrayList、ConcurrentHashMap
       
* 以及采用CAS的方式等

# 3、活跃性问题
* 死锁：
    * 怎么预防死锁：
        * 保证线程加锁条件顺序一直，要么都是ABC,不要一个ABC，一个BAC
        * 尽量引入加锁超时机制：
            * Lock接口的 tryLock(long time, TimeUnit unit)
        * 正确释放锁：
            * finally子句必须干净：
                * finally{ //先执行业务代码 //再 lock.unlock();} 也许不是一个好主意，有可能锁无法释放
* 活锁
    * 只是执行太慢而已，迟迟没反馈。
* 饥饿
    * 优先级太低，一直轮不到它。（Java 中优先级分为 1 到 10，1 最低，10 最高）


# 4、性能问题
线程切来上下文是需要消耗CPU的。

好的线程使用方案：
* 不要将线程池作为局部变量使用
    * 不仅没有利用线程池的优势，反而还会耗费线程池所需的更多资源
    * 线程池如无必要，必须是全局变量
* 谨慎使用默认的线程池静态方法
    * Executors.newFixedThreadPool(int);     //创建固定容量大小的线程池
    * Executors.newSingleThreadExecutor();   //创建容量为1的线程池
        * newFixedThreadPool和newSingleThreadExecutor在运行的线程数超过corePoolSize时，后来的请求会都被放到阻塞队列中等待，
        * 因为阻塞队列设置的过大，后来请求不能快速失败而长时间阻塞，就可能造成请求端的线程池被打满，拖垮整个服务。
    * Executors.newCachedThreadPool();       //创建一个线程池，线程池容量大小为Integer.MAX_VALUE
        * newCachedThreadPool将corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，阻塞队列使用的SynchronousQueue，SynchronousQueue不会保存等待执行的任务
        * 所以newCachedThreadPool是来了任务就创建线程运行，而maximumPoolSize相当于无限的设置，使得创建的线程数可能会将机器内存占满。
* 最好是根据业务需求自定义线程池参数
    * I/O密集型
        * 最佳线程数=CPU数/(1-阻塞系数); 
        * 阻塞系数=线程等待时间/(线程等待时间+CPU处理时间)
        * IO密集型任务的CPU处理时间往往远小于线程等待时间，所以阻塞系数一般认为在0.8-0.9之间
        * 4核CPU，阻塞系数 = 0.9， corePoolSize = 4 / (1-0.9) = 40 个！
    * CPU密集型
        * 最佳线程数 = CPU核数+1个线程
        * 4核CPU，corePoolSize=5个！

# 5、参考
* [程序员段飞-这些线程安全的坑，你在工作中踩了么？](https://juejin.cn/post/7060707944534376478)    
