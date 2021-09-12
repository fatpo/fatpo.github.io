# 1、写这个笔记的背景
程序员工作中会遇到一个问题`Java线程安全`，具体可以看我的笔记：[什么是Java线程安全](https://fatpo.github.io/#/JVM/Java线程/什么是Java线程安全)。

这个问题的根源是来自`JMM`，具体可以看我的笔记：[JMM内存模型](https://fatpo.github.io/#/JVM/JVM整体架构/JMM内存模型) 。

解决线程安全有3个原则：`原子性`，`有序性`，`可见性`。

怎么保证这3个原则，Java给我们提供了工具：`synchronized`， 这就是为什么国哥要写这篇笔记的原因。


# 2、synchronized保证方法或代码块操作的原子性
这里我就直接抄袭参考链接1的：
```text
Synchronized 保证⽅法内部或代码块内部资源（数据）的互斥访问。
即同⼀时间、由同⼀个Monitor（监视锁） 监视的代码，最多只能有⼀个线程在访问。

被 Synchronized 关键字描述的方法或代码块在多线程环境下同一时间只能由一个线程进行访问。

在持有当前 Monitor 的线程执行完成之前，其他线程想要调用相关方法就必须进行排队，
知道持有持有当前 Monitor 的线程执行结束，释放 Monitor ，下一个线程才可获取 Monitor 执行。

如果存在多个 Monitor 的情况时，多个 Monitor 之间是不互斥的。

多个 Monitor 的情况出现在自定义多个锁分别来描述不同的方法或代码块，
Synchronized 在描述代码块时可以指定自定义 Monitor ，默认为 this 即当前类。
```

动图描述 Monitor 工作机制：
![](imgs/synchronized-monitor-1.awebp)
![](imgs/synchronized-monitor-2.awebp)



# 3、保证监视资源的可见性
这里我就直接抄袭参考链接1的：
```text
保证多线程环境下对监视资源的数据同步。即任何线程在获取到 Monitor 后的第⼀时间，会先将共享内存中的数据复制到⾃⼰的缓存中；

任何线程在释放 Monitor 的第⼀时间，会先将缓存中的数据复制到共享内存中。
```

# 4、保证线程间操作的有序性
这里我就直接抄袭参考链接1的：
```text
Synchronized 的原子性保证了由其描述的方法或代码操作具有有序性，同一时间只能由最多只能有一个线程访问，不会触发 JMM 指令重排机制。
```

# 5、总结
当然我们上面的都是抄袭且算是知道这么一回事，有个monitor监控这块代码，然后monitor只能被一个线程持有什么的。

那么具体的底层原理，我建议看 [参考2](http://www.hollischuang.com/archives/1883)，从源码角度解读 synchronized怎么做到的，不过我相信，面试问这些底层源码了，要么就是面试官炫技，要么就是面试官刁难，要么就是你简历写精通JVM。

参考2的意思，简单来说就是从汇编代码可以看到：`monitorenter`和`monitorexit`两个指令，前者是加锁，后者是释放锁：
```text
每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行monitorenter）后，该计数器自增变为 1 ，

当同一个线程再次获得该对象的锁的时候，计数器再次自增。

当同一个线程释放锁（执行monitorexit指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。
```

# 6、参考
* [掘金: 图解Java线程安全](https://juejin.cn/post/6844903890224152584?share_token=5a50f615-9135-4e98-83a8-a062ff673f7b)
* [hollischuang: 深入理解多线程（一）——Synchronized的实现原理](http://www.hollischuang.com/archives/1883)
