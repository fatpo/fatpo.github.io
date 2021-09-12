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

# 5、参考
* [掘金: 图解Java线程安全](https://juejin.cn/post/6844903890224152584?share_token=5a50f615-9135-4e98-83a8-a062ff673f7b)
