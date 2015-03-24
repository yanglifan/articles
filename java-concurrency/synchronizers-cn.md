（原来写过一篇相同标题的文章，不过因为 OSChina 编辑器的缘故，格式改乱了，所以重写一篇。原文已删除，收藏原文的朋友对不住。）

这次说一下 JUC 中的同步器三个主要的成员：`CountDownLatch`、`CyclicBarrier` 和 `Semaphore`（不知道有没有初学者觉得这三个的名字不太好记）。这三个是 JUC 中较为常用的同步器，通过它们可以方便地实现很多线程之间协作的功能。（下面的代码出自 JDK 文档）
 
# CountDownLatch 
直译过来就是倒计数(CountDown)门闩(Latch)。倒计数不用说，门闩的意思顾名思义就是阻止前进。在这里就是指 `CountDownLatch.await()` 方法在倒计数为0之前会阻塞当前线程。

## 作用
`CountDownLatch` 的作用和 `Thread.join()` 方法类似，可用于一组线程和另外一组线程的协作。例如，主线程在做一项工作之前需要一系列的准备工作，只有这些准备工作都完成，主线程才能继续它的工作。这些准备工作彼此独立，所以可以并发执行以提高速度。在这个场景下就可以使用 `CountDownLatch` 协调线程之间的调度了。在直接创建线程的年代（Java 5.0 之前），我们可以使用 `Thread.join()`。在 JUC 出现后，因为线程池中的线程不能直接被引用，所以就必须使用 `CountDownLatch` 了。
 
## 示例
下面的这个例子可以理解为 F1 赛车的维修过程，只有 startSignal （可以表示停车，可能名字不太贴合）命令下达之后，维修工才开始干活，只有等所有工人完成工作之后，赛车才能继续。

```java
class Driver { // ...
    void main() throws InterruptedException {
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(N);
 
        for (int i = 0; i < N; ++i) // create and start threads
            new Thread(new Worker(startSignal, doneSignal)).start();
 
        doSomethingElse();            // don't let run yet
        startSignal.countDown();      // let all threads proceed
        doSomethingElse();
        doneSignal.await();           // wait for all to finish
    }
}
 
class Worker implements Runnable {
    private final CountDownLatch startSignal;
    private final CountDownLatch doneSignal;
    Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
        this.startSignal = startSignal;
        this.doneSignal = doneSignal;
    }
    public void run() {
        try {
            startSignal.await();
            doWork();
            doneSignal.countDown();
        } catch (InterruptedException ex) {} // return;
    }
 
    void doWork() { ... }
}
```

当 `startSignal.await()` 会阻塞线程，当 `startSignal.countDown()` 被调用之后，所有 `Worker` 线程开始执行 `doWork()` 方法，所以 `Worker。doWork()` 是几乎同时开始执行的。当 `Worker.doWork()` 执行完毕后，调用 `doneSignal.countDown()`，在所有 `Worker` 线程执行完毕之后，主线程继续执行。

# CyclicBarrier 
`CyclicBarrier` 翻译过来叫循环栅栏、循环障碍什么的（还是有点别扭的。所以还是别翻译了，只可意会不可言传啊）。它主要的方法就是一个：`await()`。`await()` 方法每被调用一次，计数便会减少1，并阻塞住当前线程。当计数减至0时，阻塞解除，所有在此 `CyclicBarrier` 上面阻塞的线程开始运行。在这之后，如果再次调用 `await()` 方法，计数就又会变成 N-1，新一轮重新开始，这便是 Cyclic 的含义所在。

`CyclicBarrier` 的使用并不难，但需要注意它所相关的异常。除了常见的异常，`CyclicBarrier.await()` 方法会抛出一个独有的 `BrokenBarrierException`。这个异常发生在当某个线程在等待本 `CyclicBarrier` 时被中断或超时或被重置时，其它同样在这个 `CyclicBarrier` 上等待的线程便会受到 `BrokenBarrierException`。意思就是说，同志们，别等了，有个小伙伴已经挂了，咱们如果继续等有可能会一直等下去，所有各回各家吧。

`CyclicBarrier.await()` 方法带有返回值，用来表示当前线程是第几个到达这个 Barrier 的线程。
 
和 `CountDownLatch` 一样，`CyclicBarrier` 同样可以可以在构造函数中设定总计数值。与 `CountDownLatch` 不同的是，`CyclicBarrier` 的构造函数还可以接受一个 `Runnable`，会在 `CyclicBarrier` 被释放时执行。

> NOTE: CyclicBarrier 的功能也可以由 CountDownLatch 来实现
 
## 示例
CyclicBarrier 的应用（当然，这个例子换成 CountDownLatch 也是可以实现的，很简单，就不说怎么写了） 

```java
class Solver {
    final int N;
    final float[][] data;
    final CyclicBarrier barrier;
 
    class Worker implements Runnable {
        int myRow;
        Worker(int row) { myRow = row; }
        public void run() {
            while (!done()) {
                processRow(myRow);
 
                try {
                    barrier.await();
                } catch (InterruptedException ex) {
                    return;
                } catch (BrokenBarrierException ex) {
                    return;
                }
            }
        }
    }
 
    public Solver(float[][] matrix) {
        data = matrix;
        N = matrix.length;
        barrier = new CyclicBarrier(N, new Runnable() {
                public void run() {
                    mergeRows(...);
                }
            });
        for (int i = 0; i < N; ++i)
            new Thread(new Worker(i)).start();
 
        waitUntilDone();
    }
}
```

## `CyclicBarrier` 和 `CountDownLatch` 在用法上的不同
`CountDownLatch` 适用于一组线程和另一个主线程之间的工作协作。一个主线程等待一组工作线程的任务完毕才继续它的执行是使用 `CountDownLatch` 的主要场景；`CyclicBarrier` 用于一组或几组线程，比如一组线程需要在一个时间点上达成一致，例如同时开始一个工作。另外，`CyclicBarrier` 的循环特性和构造函数所接受的 `Runnable` 参数也是 `CountDownLatch` 所不具备的。
 
# Semaphore 
`Semaphore` 直译是信号量，可能称它是许可量更容易理解。当然，因为在计算机科学中这个名字由来已久，所以不能乱改。它的功能比较好理解，就是通过构造函数设定一个数量的许可，然后通过 `acquire` 方法获得许可，`release` 方法释放许可。它还有 `tryAcquire` 和 `acquireUninterruptibly` 方法，可以根据自己的需要选择
 
## 示例：`Semaphore` 控制资源访问
```java
class Pool {
    private static final int MAX_AVAILABLE = 100;
    private final Semaphore available = new Semaphore(MAX_AVAILABLE, true);
 
    public Object getItem() throws InterruptedException {
        available.acquire();
        return getNextAvailableItem();
    }
 
    public void putItem(Object x) {
        if (markAsUnused(x))
            available.release();
    }
 
    // Not a particularly efficient data structure; just for demo
 
    protected Object[] items = ... whatever kinds of items being managed
    protected boolean[] used = new boolean[MAX_AVAILABLE];
 
    protected synchronized Object getNextAvailableItem() {
        for (int i = 0; i < MAX_AVAILABLE; ++i) {
            if (!used[i]) {
                used[i] = true;
                return items[i];
            }
        }
        return null; // not reached
    }
 
    protected synchronized boolean markAsUnused(Object item) {
        for (int i = 0; i < MAX_AVAILABLE; ++i) {
            if (item == items[i]) {
                if (used[i]) {
                    used[i] = false;
                    return true;
                } else
                    return false;
            }
        }
        return false;
    }
}
```

上面这个示例中 Semaphore 的用法没什么可多讲的。需要留言的是这里面有两个同步方法，不过对吞吐应该没什么影响，因为主要是对一个 boolean 数组做一下 O(n) 的操作，而且每个循环里面的操作很简单，所以速度很快。不过不知道 JUC 里面线程池的控制是怎么做的，本人不才，还没看过那块源代码，得空看看，有知道的也可以说说。
 
# 最后一句话总结
`CountDownLatch` 是能使一组线程等另一组线程都跑完了再继续跑；`CyclicBarrier` 能够使一组线程在一个时间点上达到同步，可以是一起开始执行全部任务或者一部分任务。同时，它是可以循环使用的；`Semaphore` 是只允许一定数量的线程同时执行一段任务。