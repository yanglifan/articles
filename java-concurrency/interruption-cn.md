所谓线程中断，其实就是终止一个线程。在使用 Java 线程时，除了线程自行正常结束，很多时候也需要提前结束一个线程的执行过程。Thread 类中有一个与 start() 相对应的 stop() 方法，可以从外部结束一个线程的执行。但是这个方法是极不推荐使用的，因为从外部强行结束一个线程的执行，会导致不可预知的错误，因为这样往往会在错误的时间结束一个线程的执行。

所以，在 Java 线程机制中，就有了另一种结束线程的方式，那就是中断。中断简而言之就是让线程外部可以设置一个标记值，而线程内部在执行时可以检查这个值，来获知此线程是否应该结束了。

# 1. 能够设置中断的方法
除了 Thread.interrupt() 方法以外，下列 JDK 中的方法也会设置中断（也是通过调用 `Thread.interrupt()` 来实现的）：

* `FutureTask.cancel()`
* `ExecutorService.shutdownNow()` 这个方法会调用线程池中所有线程的中断方法，不论它们是空闲的还是运行中的。而 `ExecutorService.shutdown()` 方法只能中断空闲的线程。

上面只是举两个 JDK 中应用到了线程中断的例子，这样的例子还有很多，就不一一列举了。当然，为了能响应中断，在你所写的 Runnable 或 Callable 代码中，必须通过 `Thread.isInterrupted()`、`Thread.interrupted()` 方法，或者捕获 `InterruptedException` 等的中断异常来发现线程中断并处理，否则线程是不会自行提前结束的。

# 2. 能够被中断的方法
在 JDK 和其它类库和框架中，能相应中断的方法是很多的。下面列出几个常见的 JDK 中能响应中断的方法：

* Thread.sleep()
* Object.wait()
* BlockingQueue.put(), BlockingQueue.take()
* ReentrantLock.lockInterruptibly(), Condition.await()
* ServerSocketChannel.accept(), SocketChannel.open()

等等

JDK 中能响应中断的方法基本上都是抛出异常。这些方法基本可被分为两类：一类是并发相关的，一类是 IO 相关的。

# 3. 中断的处理
## 3.1 处理 InterruptedException（也包括其它中断异常）
`InterruptedException` 是最常见的中断表现形式。所以如何处理 `InterruptedException` 便成为 Java 中断知识中的必修课。在这方面 IBM developerWorks 上有篇文章讲的很好，我在下面的参考文章中会列出链接。我这里就对这篇文章做一个总结，各位看客可以去读那边文章以获得细节知识。

处理 InterruptedException 可有以下几种方式（下面使用的代码均引用自 [Java 理论与实践: 处理 InterruptedException](http://www.ibm.com/developerworks/cn/java/j-jtp05236.html)）：

### 3.1.1 直接向上抛出
将异常不做任何处理，直接抛向该方法的调用者

    <!-- lang: java -->
    public class TaskQueue {
        private static final int MAX_TASKS = 1000;
    
        private BlockingQueue<Task> queue 
            = new LinkedBlockingQueue<Task>(MAX_TASKS);
    
        public void putTask(Task r) throws InterruptedException { 
            queue.put(r);
        }
    
        public Task getTask() throws InterruptedException { 
            return queue.take();
        }
    }

### 3.1.2 在 catch 中做处理后在抛出
因为 `InterruptedException` 的抛出，会打断方法执行，使正在进行的工作只完成一部分。在有些情况下，你就需要进行诸如回滚的处理。所以在这种情况便需要在 catch 块中进行处理之后在向上抛出 `InterruptedException`。

    <!-- lang: java -->
    public class PlayerMatcher {
        private PlayerSource players;
    
        public PlayerMatcher(PlayerSource players) { 
            this.players = players; 
        }
    
        public void matchPlayers() throws InterruptedException { 
            try {
                 Player playerOne, playerTwo;
                 while (true) {
                     playerOne = playerTwo = null;
                     // Wait for two players to arrive and start a new game
                     playerOne = players.waitForPlayer(); // could throw IE
                     playerTwo = players.waitForPlayer(); // could throw IE
                     startNewGame(playerOne, playerTwo);
                 }
             }
             catch (InterruptedException e) {  
                 // If we got one player and were interrupted, put that player back
                 if (playerOne != null)
                     players.addFirst(playerOne);
                 // Then propagate the exception
                 throw e;
             }
        }
    }

### 3.1.3 不抛出 InterruptedException 时要恢复中断状态
很多时候，由于你所实现的接口定义的限制，你很可能无法抛出 `InterruptedException`。例如实现 `Runnable` 接口以编写业务代码。这时，你就无法再向上抛出 `InterruptedException` 了。此时你应该使用 `Thread.currentThread().interrupt()` 方法去恢复中断状态。因为阻塞方法在抛出 `InterruptedException` 时会清除当前线程的中断状态，如果此时不恢复中断状态，也不抛出 `InterruptedException`，那中断信息便会丢失，上层调用者也就无法得知中断的发生。这样便有可能导致任务无法正确终止的情况方式。

    <!-- lang: java -->
    public class TaskRunner implements Runnable {
        private BlockingQueue<Task> queue;
    
        public TaskRunner(BlockingQueue<Task> queue) { 
            this.queue = queue; 
        }
    
        public void run() { 
            try {
                 while (true) {
                     Task task = queue.take(10, TimeUnit.SECONDS);
                     task.execute();
                 }
             }
             catch (InterruptedException e) { 
                 // Restore the interrupted status
                 Thread.currentThread().interrupt();
             }
        }
    }

在这里，我们要清楚中断的意义在于并发或异步场景下任务的终止。所以，如果你的代码在吞掉 `InterruptedException` 而不抛出时并不会造成任务无法被正确终止的情况方式，那也可以不再恢复中断。

还是 Runnable 的例子，大家都知道 `Runnable` 很多时候是用在线程池中，由线程池提供线程来运行。并且通常是作为任务的顶层容器来使用的，也就是说在线程池和 `Runnable` 实现之间，没有别的调用层了。那么在 try-catch `InterruptedException` 之后，便可不用在恢复线程中断了（当然，如果有回滚等的需求，还是需要实现的）。因为通过异常的捕获，你已经可以正确终止线程了。

但是，如果不是上述情况，你所写的，捕获 `InterruptedException` 的方法会被其它的、非线程池类的方法调用。例如有 A, B 两个方法，A 被 B 方法调用，A 中捕获 `InterruptedException` 后没有恢复线程中断，而 B 方法是一个无限循环，通过检查线程中断来退出，或者在调用 A 之后有个阻塞的方法。那便会造成线程无法按照期望被终止的情况发生。

### 3.1.4 自己抛出 InterruptedException
有时候，你需要“无中生有”地创造出一个 `InterruptedException` 以表示中断的发生。当然，在这个时候，你也需要使用 `Thread.isInterrupted()` 或 `Thread.interrupted()` 来检测中断的发生。那究竟是用这两者中的哪一个，还是随便呢。其实看了前面的部分我们知道，抛出 `InterruptedException` 时，线程中断状态需要被清除（为什么？自己想想吧，呵呵）。这就是 `Thread.interrupted()` 的作用。

_其实，你要是看 JDK 源代码，就会发现，JDK 中并发类也是这么做的_

> **NOTE: 使用 Thread.interrupt() 和 InterruptedException 中的哪种方法表示中断？**上面提到了一种情况是由于接口的限制而无法抛出 InterruptedException，这时你别无选择，只能用 Thread.interrupt() 恢复中断。除了这种情况，其它的时候推荐使用 InterruptedException 来表示中断。当方法声明抛出 InterruptedException 时，它就是在告诉调用者，我这个方法可能会花费很多的时间，而你可以通过线程中断来终止调用。通过 InterruptedException 来表示中断，含义更清晰，反应也更迅速。

## 3.2 无法被中断的情况
### 3.2.1 synchronized
阻塞在 synchronized 的内置锁上是无法被中断的，如果需要可以被中断的锁。可以使用 Java 5 concurrent 中的 Lock。

### 3.2.2 Java IO（不包含 NIO 和 AIO）
对 Java IO 少有了解的人都知道，`ServerSocket.accept()` 是一个阻塞方法，但是 `Thread.interrupt()` 对它毫无影响。要想终止 `ServerSocket.accept()` 的等待，唯一方法就是调用 close() 方法。 

## 3.3 线程中断与 Java IO
上面说到传统的 Java IO 中的阻塞方法无法响应线程中断，因为 Java IO 出现的时候还没有中断机制。在 Java 1.4 引入的 Java NIO 已经可以很好地支持 Java 线程中断了。

如果是使用 XXXChannel，线程中断会导致 accept(), open() 等阻塞方法抛出 ClosedByInterruptException。

# 4. 参考

* [Java 理论与实践: 处理 InterruptedException](http://www.ibm.com/developerworks/cn/java/j-jtp05236.html)
* [线程管理（四）操作线程的中断机制](http://ifeve.com/thread-manager-4/)
* [详细分析Java中断机制](http://www.infoq.com/cn/articles/java-interrupt-mechanism)
* [任务取消(Cancellation)](http://ifeve.com/cancellation/)