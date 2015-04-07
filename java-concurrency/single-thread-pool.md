# Single Thread Pool
## Motivation
The util class `Executors` has a factory method - `ExecutorService newSingleThreadExecutor()` to a single thread pool. Although in the most of conditions, the thread pool is used to make the system has a better performance and resposibility. But in a multi threads application, your code has to consider the thread safe. So `ExecutorService newSingleThreadExecutor()` has its usage. In a scenario which the system does not have a high payload or does not need to consider a high performance. Then in this scenario, a single thread pool can be used. With it, you will feel relax about the thread safe (It does not mean you do not need to consider it, just easier.

## Unconfigurable
We mentioned before that some parameters of `ThreadPoolExecutor` can be changed on the fly. So could we change the core size of the single thread pool to break its single thread behavior? The answer is no.

```java
@Test(expected = ClassCastException.class)
public void change_single_thread_pool_core_size() {
    ExecutorService single = Executors.newSingleThreadExecutor();
    ((ThreadPoolExecutor) single).setCorePoolSize(2);
}
```

If you write a code snippet like above, you will encounter `ClassCastException` just like the test method declared. The reason is that `Executors` wrapper the single thread pool to an other `ExecutorService` which does not have methods like `setCorePoolSize(int)`.

That provides us a way to create a `ThreadPoolExecutor` which cannot be changed its parameters. You can use `Executors.unconfigurableExecutorService(ExecutorService)` to wrapper your own `ThreadPoolExecutor` instance.