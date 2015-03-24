# Thread Pool: Core Size, Max Size and Work Queue

Though in Java 6, 7 and 8, many new features have been imported into concurrency utils. Thread pool classes introduced by Java 5 are always most common used concurrency tools by Java programmers. It looks like easy to use them but master them is not an easy work. So today I will write something about how to use Java thread pools better.

`ThreadPoolExecutor` has three, very important parameters: the number of core threads, the number of maximum threads and the work queue which type is `BlockingQueue`:

```java
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue)
```

To be a programmer who is familiar with Java concurrency, you should know the relationship between these three parameters. After a `ThreadPoolExecutor` initialized, there is no worker thread. Along with tasks are submitted into this `ThreadPoolExecutor`, a new worker thread will be created, no matter whether there is idle core worker thread or not, till this number reaches the core pool size. Then new arrived tasks will be stored in the work queue. Util this queue is full, then new worker threads will be created again. The number of worker threads will increase to the maximum pool size.

So a common mistake is to create a `ThreadPoolExecutor` with a small core pool size, a large maximum pool size and a large size work queue like `LinkedBlockingQueue`. Then expected when a huge mount of tasks come in, the number of worker threads will increase to the maximum pool size soon. This will not happen, unless the work queue is full. But in the most of cases, before the work queue is full, OutOfMemoryError will happen.

So the factory method `Executors.newFixedThreadPool(int nThreads)` will create a `ThreadPoolExecutor` instance whose the core size and maximum size are same, otherwise the maximum pool size is meaningless.

## newFixedThreadPool
The method `Executors.newFixedThreadPool(int nThreads)` will create a `ThreadPoolExecutor` like this:
```
new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
```

The threads are bounded but the work queue is unbounded. So if the payloads are exceeded the process ability, the work queue will store more and more tasks. If this situation keeps all the time, the full work queue will cause memory problems. Even an empty task, if the number is up to 20 million, it still would use 1GB memory. So there is no server be able to store `Integer.MAX_VALUE` tasks, which is around 2 billion.

## newCachedThreadPool
The method `Executors.newCachedThreadPool` will create a `ThreadPoolExecutor` like this:
```
new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>())
```
The core pool size of `ThreadPoolExecutor` can be set as 0. The `ThreadPoolExecutor` instance created by `Executors.newCachedThreadPool()` is this type of `ThreadPoolExecutor`. But this type of use a special `BlockingQueue` - `SynchronousQueue`. `SynchronousQueue` actually is a transfer, it can store items but it always said that "I am full". So this behavior will cause `ThreadPoolExecutor` to create a new worker thread to poll tasks from `SynchronousQueue` then run them.

Its threads are unbounded but the work queue is bound (just a handoff queue.), so it will more dangerous if use it to handle a huge mount of tasks directly.

## Quiz
* `new ThreadPoolExecutor(0, 10, new ArrayBlockingQueue(100))`