JDK 5 的并发包中提供了很多类，这些类提供了比原有的并发机制更好的性能和伸缩性。要想理解这些类的工作机理，那就不得不提到 CAS。CAS 全称是 Compare and Swap，是指现代主流 CPU 都支持的一种指令，这个指令能为多线程编程带来更好的性能（稍后会详细介绍）。另外一个可能会被当做 CAS 的是 Compare and Set，是指 JDK 5 并发包中广泛使用的一种基于 Compare and Swap 的并发算法。严格说 CAS 仅指代前者。

###Compare and Swap###
在这里，CAS 指的是现代 CPU 广泛支持的一种对内存中的共享数据进行操作的一种特殊指令。这个指令会对内存中的共享数据做原子的读写操作。简单介绍一下这个指令的操作过程：首先，CPU 会将内存中将要被更改的数据与期望的值做比较。然后，当这两个值相等时，CPU 才会将内存中的数值替换为新的值。否则便不做操作。最后，CPU 会将旧的数值返回。这一系列的操作是原子的。它们虽然看似复杂，但却是 Java 5 并发机制优于原有锁机制的根本。简单来说，CAS 的含义是“我认为原有的值应该是什么，如果是，则将原有的值更新为新值，否则不做修改，并告诉我原来的值是多少”（这段描述引自《Java Concurrency in Practice》）

###Compare and Set###
在理解了什么是 Compare and Swap 之后，理解 Compare and Set 便很容易了。Compare and Set 就是利用 Compare and Swap 实现的非阻塞的线程安全的写操作算法。它的实现过程如下：首先读取你要更改的数据的原值，然后将其和你要更新成的值作为 Compare and Swap 操作的两个参数，如果 Compare and Swap 的返回值和原值不同，便重复这一过程，直至成功。写成伪代码如下

    int old;
    int new;
    do {
        old = value.get();
        new = doSomeCalcBasedOn(old)
    while (value.compareAndSwap(old, new));

Compare and Set 是一个非阻塞的算法，这是它的优势。但是有一个问题就是存在这样的可能，在每次读操作之后，写操作之前，都有另外一个线程更改了原先的值，这样 Compare and Set 操作就会不停地重试。但这样的可能只存在于理论，在实际中很少发生。

Compare and Set 广泛使用在 Java 5 中的 Atomic 类中，其它的诸如 ReetrantLock、Semaphore 等的类也通过 AbstractQueuedSynchronizer 类间接地使用了 Compare and Set。
