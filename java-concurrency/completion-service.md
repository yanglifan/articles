# CompletionService
## Motivation
When we submit (execute will cannot get the return) a group of tasks, then we may need to get results of them all. How to do this? Loop the `Future` and invoke `get()` method on every `Future` instance? Is it trivial? So JDK provides a wrapper to make the asynchronous execution result can be get easily.

## Demo
```java
ExecutorService executorService = ExecutorBuilder.newBuilder().build();
CompletionService<Integer> completionService = new ExecutorCompletionService<>(executorService);
 
IntStream.range(0, 10).forEach(i -> completionService.submit(() -> {
    Thread.sleep(1000);
    return i;
}));
 
Future<Integer> future = completionService.poll(3, TimeUnit.SECONDS); // InterruptedException
int sum = 0;
while (future != null) {
    sum += future.get(); // ExecutionException
    future = completionService.poll(3, TimeUnit.SECONDS);
}
 
assertThat(sum, is(45));
```