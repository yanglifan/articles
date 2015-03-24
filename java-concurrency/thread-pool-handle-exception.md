# Thread Pool: Handle Exception

## Handle Exception
There are some developers use a thread or a thread pool to execute a task directly, like following:

```java
aThreadPoolExecutor.execute(new Runnable() {
    public void run() {
        // Some code
    }
});
```

How about If there is no `try...catch` block in the above "some code"? One result may be the runtime exception happened in the "some code" will not be tracked. Why? Because in Java, every thread has a default uncaught exception handler. If you do not do any configuration, a uncaught exception (RuntimeException) will be output to `System.err` by a default uncaught exception handler. That is why when you write a multithread cmd application, you can see the error stack in the console. But in a server application, an runtime exception cannot be output to the log file automatically. So you can see the code in a lot of server applications like following:

```java
aThreadPoolExecutor.execute(new Runnable() {
    public void run() {
        try {
	    // Some code
        catch (Exception e) {
            LOGGER.error("xxx", e);
        }
    }
});
```

The above looks like a little trivial if you have to write like this everytime. Actually, just like mentioned at above, Java already has a mechanism to handle the exception thrown in a thread:
```java
Thread.setDefaultUncaughtExceptionHandler((t, e) -> LOGGER.error("======== NOTE ========", e));
Executors.newSingleThreadExecutor().execute(() -> {
    throw new RuntimeException("My runtime exception");
});
```
In the above code snippet, for short, I use Java 8 Lambda syntax. But it will not affect too much. In the class `Thread`, there is a static method `setDefaultUncaughtExceptionHandler`. You can use this method to set your own ex handler for all threads in this JVM process. In the above sample, the handler will write the exception information into the log. So in every task executed in thread pools, no more try...catch.

UncaughtExceptionHandler has three level, for all threads, for a thread group and for a per thread. So for a thread pool, you can use a `ThreadFactory` to set a custom `UncaughtExceptionHandler` for every worker thread in this thread pool.

### execute() and submit()
But the story about handle exception is not finished yet. Let me see a code snippet:
```java
Thread.setDefaultUncaughtExceptionHandler((t, e) -> LOGGER.error("======== NOTE ========", e));
Executors.newSingleThreadExecutor().submit((Runnable) () -> {
    throw new RuntimeException("My runtime exception");
});
```
In the above sample, could the runtime exception be logged? The answer is no, unless get the `Future` result:
```java
Thread.setDefaultUncaughtExceptionHandler((t, e) -> LOGGER.error("======== NOTE ========", e));
Future f = Executors.newSingleThreadExecutor().submit((Runnable) () -> {
    throw new RuntimeException("My runtime exception");
});
f.get(); // Exception will be logged
```