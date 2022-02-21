* **1、Java创建对象的几种方式**
* **2、Java反射是什么**
* **3、Java创建线程池的流程和参数**
* **4、Java线程的状态流转**
* **5、为什么必须在同步的方法/块内调用wait()，notify()，notifyAll()**
* **6、Java的HashMap底层原理**
* **7、Java的类的加载机制**


## 1、Java创建对象的几种方式
* 使用`new关键字`: Student student = new Student();
* 使用`Class类的newInstance`方法: Student student2 = (Student)Class.forName("根路径.Student").newInstance();
* 使用`Constructor类的newInstance`方法: Constructor<Student> constructor = Student.class.getInstance(); Student stu = constructor.newInstance();
* 使用`clone`方法: Student stu2 = <Student>stu.clone();
* 使用`反序列化`方法: ObjectInputStream in = new ObjectInputStream (new FileInputStream("data.obj"));  Student stu3 = (Student)in.readObject();
  * 使用`反射`方法: 通过new创建对象的效率比较高。通过反射时，先找查找类资源，使用类加载器创建，过程比较繁琐，所以效率较低。

## 2、Java反射是什么
* 反射是什么
  * 反射机制是在`运行时`，对于任意一个类，都能够知道这个类的所有属性和方法；
  * 对于任意个对象，都能够调用它的任意一个方法。
  * 在java中，只要给定类的名字，就可以通过反射机制来获得类的所有信息。
  * 这种动态获取的信息以及动态调用对象的方法的功能称为Java语言的反射机制。
* 哪里用到了反射
  * jdbc: Class.forName("com.mysql.jdbc.Driver.class")
  * 框架：spring, hibernate 等
* 反射实现机制
  * 第一步：获取class对象！
  * 第二步：创造实例
    * 使用class对象的newInstance()方法来创建该 Class 对象对应类的实例（如果有空构造函数）
    * 使用Class对象获取指定的 Constructor 对象，再调用 Constructor 对象的 newInstance()方法来创建 Class 对象对应类的实例,通过这种方法可以选定构造方法创建实例
* 获取class对象的几种方法
  * 通过Object类的getClass方法来获取
    * System.out.println(list.getClass().getName()); // java.util.ArrayList
  * 使用.class的方式
    * Class clazz = String.class; System.out.println(clazz.getName()); // java.lang.String
  * 使用Class.forName方法（**最安全+性能最好**）
    * Class clazz = Class.forName("org.whatisjava.reflect.Foo");
* 反射的优缺点
  * 优点
    * 能够运行时动态获取类的实例，提高灵活性；
    * 与动态编译结合
  * 缺点
    * 使用反射性能较低，需要解析字节码，将内存中的对象进行解析
        * 通过setAccessible(true)关闭JDK的安全检查来提升反射速度；
        * 多次创建一个类的实例时，有缓存会快很多
    * 相对不安全，破坏了封装性（因为通过反射可以获得私有方法和属性）

## 3、Java创建线程池的流程和参数
* 参数
  * maximumPoolSize： 线程池最大能创建的线程数量
  * corePoolSize：核心线程数
    * 当已创建线程数 < corePoolSize时，即使此时线程池中存在空闲线程，也会创建新线程，这些线程称为核心线程。 
    * 当已创建线程数 == corePoolSize时，又有任务来，会把任务放到阻塞队列中排队。 
    * 当阻塞队列已满，且已创建线程数<maximumPoolSize，创建非核心线程执行任务。 
    * 当阻塞队列已满，且已创建线程数==maximumPoolSize，使用拒绝策略。
  * keepAliveTime + unit：线程存活时间
    * 默认当线程池中已创建线程数 > corePoolSize时，keepAliveTime才会起作用。
    * 即一个空闲线程存活到keepAliveTime了，就会被销毁，直到已创建线程数 == corePoolSize。
  * workQueue：阻塞队列
    * ArrayBlockingQueue
    * LinkedBlockingQueue
    * PriorityBlockingQueue
    * SynchronousQueue
  * handler：拒绝策略
    * ThreadPoolExecutor.AbortPolicy: 丢弃任务并抛出RejectedExecutionException异常
    * ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
    * ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
    * ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
* Executors工具类创建线程池
  * 可变数量 newCachedThreadPool
  * 固定数量 newFixedThreadPool
  * 固定一个 newSingleThreadExecutor
  * 定时线程池 newScheduledThreadPool
* 为什么阿里巴巴不让用Executors工具类创建线程池
  * 用ThreadPoolExecutor强迫开发了解业务场景选择最合适的参数配置，自定义线程池不容易出错。
  * newFixedThreadPool和newSingleThreadExecutor的请求队列（LinkedBlockingQueue最大是21亿)）可能是无限大，可能堆积大量的线程，OOM警告
  * newCachedThreadPool的maxPoolSize无限大，可能创建无限多线程，OOM警告

## 4、Java线程的状态流转
* 状态：
  * NEW： 线程创建完成未启动
  * RUNNABLE：可运行状态，可能正在执行中，也可能等待cpu分配资源
  * BLOCKED： 阻塞状态，等待监视器锁定时被阻止的状态，调用synchronized 同步块或方法；或者调用wait之后重新进入 Synchronized 同步块或方法
  * WAITING：等待状态，调用 Object.wait，Thread.join，LockSupport.park 方法进入等待状态；
    * 另一个线程可以用Object.notify或 Object.notifyAll 唤起wait的线程，Thread.join只能等待指定线程结束，LockSupport.unpark
  * TIMED_WAITING：具有等待时间的等待状态，调用以下方法进入该状态；
    * Thread.sleep(1000)、Object.wait(1000)、 Thread.join(1000)、LockSupport.parkNanos(1000)、LockSupport.parkUntil(1000) ; 等待时间完毕进入runnable状态
  * TERMINATED：终止，线程执行完毕

## 5、为什么必须在同步的方法/块内调用wait()，notify()，notifyAll()
* 1、一个线程调用一个对象A的wait()之后,就需要另外一个线程来调用A的notify() 来唤醒,这样的成对使用是一种线程间的通讯
* 2、另一方面 wait()操作的调用一定是到达某种特定状态来，而这种特定状态是由其他线程 调用notify（），notifyAll（） 向外发出的 已到达特定状态的信号。该状态是线程之间的通信信道，并且是 共享可变状态。
* 3、Synchronize 保证多线程状态下 ，只有一个线程来操作 wait（），notify（），notifyAll（），任意意时刻只能被唯一的一个获得了对象实例锁的线程调用
* 为什么wait()和notify()需要搭配synchonized关键字使用
  * synchronized：
    * Java中每一个对象都可以成为一个监视器（Monitor）, 该Monitor由一个锁（lock）, 一个等待队列（waiting queue ）, 一个入口队列( entry queue).
    * 对于一个对象的方法， 如果没有synchronized关键字， 该方法可以被任意数量的线程，在任意时刻调用。
    * 对于添加了synchronized关键字的方法，任意时刻只能被唯一的一个获得了对象实例锁的线程调用。
    * synchronized用于实现多线程的同步操作

## 6、Java的HashMap底层原理
* jdk1.7: 
  * Entry<K,V>数组 + 链表 + 头插法（头擦法可能在多线程并发下扩容会出现死循环）
* jdk1.8: 
  * 元素个数 <= 8，Node<K, V>数组 + 链表 + 尾插法
  * 元素个数 > 8，Node<K, V>数组 + 红黑树 + 尾插法
  * 为什么是8，工业统计出来的，8这个阈值，刚好是插入效率和查询效率在两个数据结构的平衡点。
    

## 7、Java的类加载器
* 分类
    * 规范
        * Bootstrap ClassLoader
        * Other
    * Hotspot实现
        * Bootstrap ClassLoader，{JAVAHOME}/lib, 采用C++编写，无法作为对象被程序引用！
        * Other
            * Extension ClassLoader, {JAVAHOME}/lib/ext, 采用Java编写，可以被程序引用，继承自Java.lang.ClassLoader
            * Application ClassLoader, classpath/java.class.path, 采用Java编写，可以被程序引用，继承自Java.lang.ClassLoader
            * User ClassLoader, 任意来源, 采用Java编写，可以被程序引用，继承自Java.lang.ClassLoader
* 两个问题？
    * 不同的类加载器，好像除了读取二进制的动作和范围不一样，那么后续的加载逻辑是否也不一样？
        * 后续都是由 java.lang.ClassLoader.defineClass() 统一处理，开发者没有操作的空间。
    * 遇到名字一样的类，这么多类加载器会不会产生混乱呢？
        * 不会! JVM规定了不同的类加载器有唯一的命名空间！同一个类加载器下，如果之前加载过了，不会重复加载。
* 双亲委派
    * 类加载器遇到需要加载的类时，优先让父亲去加载，父亲没有的话，再让父亲的父亲去加载，如果还没有，就让父亲委派自己去加载。
    * 打破双亲委派： 重载 ClassLoader的loadClass()函数。用于JDK多家厂商的接入等场景，他们怎么打破的，是直接让Bootstrap加载器去委派Application加载器，没有走双亲委派。

## 8、Java的类的加载机制

* 加载（`就是把class的二进制流加载到内存`）
    * 1、通过类的全限顶名来获取定义此类的二进制字节流
    * 2、将这个类字节流代表的静态存储结构转为方法区的运行时数据结构
    * 3、在堆中生成一个代表此类的java.lang.Class对象，作为访问方法区这些数据结构的入口
* 链接
    * 验证（`主要目的是不要危害虚拟机`）
        * 文件格式验证
            * 是否是以魔法数字：OxCAFFBABE开头
            * 主次版本号是否在当前虚拟机接受范围内
        * 元数据验证
            * 是否有父类，理论上所有的class除了Object都应该有父类
            * 是否继承了final类
        * 字节码验证
            * 跳转指令是否指向正确位置
            * 类型转换是否有效
        * 符号引用验证
            * 符号引用的直接引用是否存在
    * 准备
        * 为类的静态变量分配内存，并初始化默认值
            * 特殊情况：如果类静态变量的字段属性表中存在ConstantValue属性，则直接执行赋值语句
                * 类静态变量为基本数据类型，并且用final修饰
                * 类静态变量为String类型，并且用final修饰，并且以字面量的形式赋值
    * 解析
        * 将类、接口、字段和方法的符号引用转为直接引用
        * 个人理解：`在编译的时候，每个java类都会被编译成一个class文件，编译的时候虚拟机并不知道所引用类的地址，只能用符号引用来代替。解析阶段就是把这个符号引用改为真正的地址的阶段！`
* 初始化
    * 初始化阶段是执行类构造器`<client>`方法的过程。
    * `<client>`方法是由编译器自动收集类中的类变量的赋值操作、静态语句块中的语句合并而成。
    * 虚拟机会保证`<client>`方法执行前，父类的`<client>`方法已经执行完毕。
    * 如果一个类中没有静态比变量赋值、没有静态语句块，那么将不会生成<client>方法。
* 使用
* 卸载