
# 从一个故障说说Java的三个BlockingQueue
bluedavy-林昊(毕玄)，2016年5月4日

最近出了个故障，排查的时候耗费了很长的时间。
**回顾整个排查过程**，`经验主义在这里起了不好的作用`，直接导致了整个故障排查的时间非常长。
这个**故障的根本原因**在于**BlockingQueue用的有问题**，
顺带展开说说Java中常用的几个BlockingQueue：ArrayBlockingQueue、LinkedBlockingQueue和SynchronousQueue。


### 故障现象和排查
当时**故障的现象**是`应用处理请求的线程池满了，导致请求处理不了`。
于是**dump线程，看线程都在做什么，结果发现线程都block在写日志的地方**。
以前出现过很多次问题，去线程dump的时候看到也是一堆的block在写日志，但通常是别的原因引发的，
所以这次也是按照这样的经验，认为肯定不会是写日志这个地方的问题，于是各种排查…
折腾了N久后，`回过头看发现持有那把日志锁的地方是自己人写的代码，
那段代码在拿到了这个日志锁后，从线程堆栈上看，block在了ArrayBlockingQueue.put这个地方`。
于是翻看这段代码，结果发现这是个1024长度的BlockingQueue，
那就意味着`如果这个Queue被放了1024个对象的话，put就一定会被block住`，
而且其实翻代码的时候能看出写代码的同学是考虑到了BlockingQueue如果满了应该要处理的，代码里写着：
```java
if (blockingQueue.remainingCapacity() < 1) { //todo }
blockingQueue.put
```
这里有`两个悲催的问题`，一是`这个if判断完还是直接会走到put，而不是else`；
二是`竟然关键的满了后的处理逻辑还在//todo`...

另外我觉得`这段代码还反应了同学对BlockingQueue的接口不太熟`，要达到这个效果，不需要这样先去判断，
**更合适的做法是用blockingQueue.offer，返回false再做相应的异常处理。**


### BlockingQueue实现类

**BlockingQueue是在`生产/消费者模式`下经常会用到的数据结构**，
常用的主要是ArrayBlockingQueue、LinkedBlockingQueue和**SynchronousQueue**。

`ArrayBlockingQueue/LinkedBlockingQueue两者的最大不同`
主要在于`存放Queue中对象方式，一个是数组，一个是链表`，代码注释里也写到了两者的不同：
```
Linked queues typically have higher throughput than array-based queues
but less predictable performance in most concurrent applications.
基于链表的队列，具有更高的吞吐量，但性能较差。
```

**SynchronousQueue**是一个非常**特殊**的BlockingQueue，它的**模式**是
`在offer的时候，如果没有另外一个线程正在take或poll的话，那么offer就会失败；
在take的时候，如果没有另外的线程正好并发在offer，也会失败。`
这种特殊的模式**非常适合用来做要求`高响应`并且`线程数不固定`的线程池的Queue**。


### fail fast/快速失败/快速处理

对于`在线业务场景`而言，**`所有`的并发，外部访问`阻塞的地方`的一个真理就是一定要有`超时机制`。**
我不知道见过多少次`由于没有超时造成的在线业务的严重故障`，**在线业务最强调的是`快速处理掉一次请求`**，
所以**fail fast/快速失败是`在线业务系统设计，代码编写`中的最重要原则**。
按照这个原则，上面的代码`最起码明显犯的错误`就是`用put而不是带超时机制的offer`，
或者说**如果是不重要的场景，完全就应该直接`用offer，false了直接抛异常或记录下异常`即可。**


### BlockingQueue使用场景

对于**BlockingQueue**这种**场景**呢，除了**超时机制**外，还有一个是**队列长度一定要做限制**，
否则`默认的是Integer.MAX_VALUE`，`万一代码出点bug的话，内存就被玩挂了(OOM)。`

说到BlockingQueue，就还是要提下**BlockingQueue被用的最多的地方：`线程池`**。
Java的`ThreadPoolExecutor`中有个参数是BlockingQueue，如果这个地方用的是ArrayBlockingQueue或LinkedBlockingQueue，
而线程池的corePoolSize和maximumPoolSize不一样的话，`在corePoolSize线程满了后，这个时候线程池首先会做的是offer到BlockingQueue，成功的话就结束。`
这种场景同样`不符合在线业务的需求`，**在线业务更希望的是`快速处理`，而不是先排队**，
而且其实**在线业务最好是不要让请求堆在排队队列里，在线业务这样做很容易`引发雪崩`**，
**`超出处理能力范围直接拒绝抛错`是相对比较好的做法**，至于在前面页面上排队什么这个是可以的，那是另外一种**限流机制**。
```java
public class ThreadPoolExecutor extends AbstractExecutorService {

    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true)) // 1. 未超过核心线程池大小，新建线程
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) { // 2. 超过核心线程池大小，任务进入队列
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false)) // 3. 当队列满了，再新建线程
            reject(command); // 超过最大线程池大小，则拒绝执行
    }

}
```

所以说**在写高并发、分布式的代码时，除了系统设计外，`代码细节的功力`是非常非常重要的。**


[原文](http://hellojava.info/?p=464)

