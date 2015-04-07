# Scheduled Thread Pool
This part is about `ScheduledThreadPoolExecutor`. Because most of programmers use this class frequently. So I will not introduce the usage of this class. I will introduce how this class be implemented.

## Nutshell
The timer function is the biggest difference between `ScheduledThreadPoolExecutor` and `ThreadPoolExecutor`. (I will mention `ScheduledThreadPoolExecutor` as the "scheduler" for short.) The scheduler has the following three methods to provide the timer function:

* `ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit)`
* `ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`
* `ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)`

So how these three methods been implemented?

The key is the *work queue*. Scheduler uses a special `BlockingQueue` - `DelayedWorkQueue` as its work queue. So Scheduler's construct does not provide the work queue parameter, because `DelayedWorkQueue` is the key and cannot be replaced by normal `BlockingQueue`.

## DelayedWorkQueue
`DelayedWorkQueue` stores instances (`ScheduledFutureTask`) in an array and with the binary heap arrange. Elments in this queue are sorted as the delay time asc. So the peek of the heap is first element which is expired and need to be executed. After the peek of the heap (`DelayedWorkQueue`) has been taken, the heap will be rebuild to be as a correct heap again. (The process in the algorithm called "heapify")

## Recur
When recurred schedule methods were invoked, the `Runnable` instance will be wrapped by `ScheduledFutureTask`, and the `period` attribute of `ScheduleFutureTask` instance will not be 0. In this way `ScheduledThreadPool` will know that this task will be executed periodically. So after the current execution iteration finished, this task will be added into the queue again. So this task can be executed periodically.