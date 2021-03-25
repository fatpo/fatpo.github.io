## 写在前面

先看看linux中进程长什么样子：

![](imgs/2021-03-25-zlVZV6.png)

其中:

- Text Section: A Process, sometimes known as the Text Section, also includes the current activity represented by the value of the Program Counter. 
- Stack: The Stack contains the temporary data, such as function parameters, returns addresses, and local variables. 
- Data Section: Contains the global variable. 
- Heap Section: Dynamically allocated memory to process during its run time. 

解释下：
- Text Section: 放代码（我记得以前上课的时候说是放代码...） 
- Stack: 放一些临时的数据如返回地址、局部变量、函数入参（类似JVM的栈）
- Data Section: 全局数据（类似JVM的方法区）
- Heap Section: 动态分配的内存（类似JVM的堆）



## linux 进程状态
- <b> New (Create) </b> – In this step, the process is about to be created but not yet created, it is the program which is present in secondary memory that will be picked up by OS to create the process.
- <b> Ready </b> – New -> Ready to run. After the creation of a process, the process enters the ready state i.e. the process is loaded into the main memory. The process here is ready to run and is waiting to get the CPU time for its execution. Processes that are ready for execution by the CPU are maintained in a queue for ready processes.
- <b> Run </b> – The process is chosen by CPU for execution and the instructions within the process are executed by any one of the available CPU cores.
- <b> Blocked or wait </b> – Whenever the process requests access to I/O or needs input from the user or needs access to a critical region(the lock for which is already acquired) it enters the blocked or wait state. The process continues to wait in the main memory and does not require CPU. Once the I/O operation is completed the process goes to the ready state.
- <b> Terminated or completed  </b>– Process is killed as well as PCB is deleted.
- <b> Suspend ready </b> – Process that was initially in the ready state but were swapped out of main memory(refer Virtual Memory topic) and placed onto external storage by scheduler are said to be in suspend ready state. The process will transition back to ready state whenever the process is again brought onto the main memory.
- <b> Suspend wait or suspend blocked </b> – Similar to suspend ready but uses the process which was performing I/O operation and lack of main memory caused them to move to secondary memory. When work is finished it may go to suspend ready.
小编有话说：
```
上面虽然密密麻麻，但仍强烈建议阅读英文doc，因各厂商设计不同，可能状态不一致，但是基本上大同小异：开始、等待运行、运行、等待、结束。
```
`linux进程状态` vs `java 线程状态`：
```
1、都有开始和结束

2、java线程的runnable = linux进程的ready + running

3、linux进程多了： Suspended Ready + Suspended Block
```

linux进程状态转移图：
![](imgs/2021-03-25-2W6f3t.png)

看了上图，你可能注意到了，进程可以划分为2个大的状态：
```
main memory vs secondary memory
```
再看那个`New`状态的描述：
```
New (Create) – In this step, the process is about to be created but not yet created, it is the program which is present in secondary memory that will be picked up by OS to create the process.
刚创建的进程，就在 secondary memory
```
关于`secondary momery`还要再看`Suspended Ready` 和 `Suspended Block`:
```
进程状态本来处于ready状态，但是被切换到 secondary momery 后，就变成了 Suspended Ready。
```
类似的 `Suspended Block`：
```
performing I/O operation and lack of main memory caused them to move to secondary memory.
```
如果它在执行IO操作又不在 `main memory` 那么就是 `suspended block`状态。

