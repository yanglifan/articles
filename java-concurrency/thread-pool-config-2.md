# Thread Pool: More Configuration

Except the core pool size, the maximum pool size and the work queue, `ThreadPoolExecutor` has some another parameters: `keepAliveTime`, `threadFactory` and `RejectedExecutionHandler`.

## keepAliveTime
This parameter used to control how long an idle worker thread can alive. By default, only the thread over the core pool size will be affect by this parameter. But you can use the method `allowCoreThreadTimeOut` to let the core worker thread can be affected by this parameter.

`Executors.newFixedThreadPool(int nThreads)` sets keepAliveTime as 0, because its core size is equal with max size, so keepAliveTime is meaningless.

## threadFactory
Sometimes we need to do some custom thread configuration. We can set an own `ThreadFactory` to do that.

With `ThreadFactory`, we can set the name, priority, `UncaughtExceptionHandler`, daemon on a thread.

### Thread name
Set your own thread name will help you to debug.
```
"MyThreadPool-10@494" prio=5 tid=0x14 nid=NA sleeping
  java.lang.Thread.State: TIMED_WAITING
	  at java.lang.Thread.sleep(Thread.java:-1)
	  at me.yanglifan.demoall.concurrency.threadpool.CustomThreadFactory$2.run(CustomThreadFactory.java:25)
	  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	  at java.lang.Thread.run(Thread.java:745)
```

## RejectedExecutionHandler
If one `ThreadPoolExecutor`'s work queue is full and the number of worker thread reaches the max pool size, then any more tasks will be rejected. And the policy of the reject action will be defined in one `RejectedExecutionHandler`.

### Workers and the queue are full
As mentioned,  `Executors.newFixedThreadPool` will use a unbounded work queue - `new LinkedBlockingQueue()`.  This will have two problems: 

1. Memory leak
1. Store too many unprocessed tasks in the queue will cause data lost when the application crashed. 

So we should be cautious to use an unbounded `BlockingQueue`. If the work queue is a bounded one, then we should consider the `RejectedExecutionHandler`. Since when no more work thread can be created and the work queue is no more slot. A new submitted task will be handled by `RejectedExecutionHandler`.

And another condition which will make the task to be handled by `RejectedExecutionHandler` is after`ThreadPoolExecutor` has been shutdown, then if a new task would still been submitted to this `ThreadPoolExecutor`, this task will be handled by `RejectedExecutionHandler`

### RejectedExecutionHandler Implementations

* `AbortPolicy` is the default implementation of `RejectedExecutionHandler` used by `ThreadPoolExecutor`. `AbortPolicy` will simply throw a `RejectedExecutionException` to reject the task execution. So by default, if one `ThreadPoolExecutor`'s worker thread and work queue are both full, `execute` and `submit` method will be failed. So you need to define a policy to handle this exception.
* `CallerRunsPolicy` will make the task executed by the caller thread. So that will make the task execution to be as the synchronous style.
* `DiscardOldestPolicy` and `DiscardPolicy` are very easy to be understood. They can be used in some unimportant task executions. But can we discard an important task? The answer seems to be very clear, of course not. But actually in a distributed application, consider that there will have message queue middleware like RabbitMQ or Apache Kafka or JMS implentation. A message queue middleware will not remove a message and persist it, unless you reply a response that you have finished the task process. So in a distributed environment, if you discard a task execution, the message queue middleware will re-deliver this message (task) to another server node. So from the whole application view, the message (task) actually is still persisted and not missed.

### Implement your own RejectedExecutionHandler
The implementations provide by JDK are very very simple. It is reasonable to implement your own `RejectedExecutionHandler` to add the logic like write log, send an alert, or send an event to fire the auto ops system to scale the service.