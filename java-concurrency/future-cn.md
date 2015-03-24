# 简介
Future 是 Java 5 JUC 包中的一个接口，主要提供了三类功能：
## 任务结果的获取
这个功能由 get 方法提供，它有两种形式的重载。get 方法本身使用起来很简单，需要注意的是它所抛出的异常：

* ExecutionException 对 Callable 或 Runnable 所抛出的异常的封装，可以通过 ` Throwable.getCause()` 方法获得具体异常。
* CancellationException 在调用 get 时任务被通过 `Future.cancel()` 方法被取消所抛出的异常。这个是 运行时异常，但如果你有调用 `Future.cancel()` 的地方，那还是需要处理的。
* TimeoutException `V get(long timeout, TimeUnit unit)` 重载形式所抛出的超时异常。

## 任务取消

# 通过代码看 Future 的使用
我们先看一段代码，这个代码是《Java Concurrency in Practise》的 “Listing 6.13. Waiting for Image Download with Future.”。

```java
public class FutureRenderer {
	private final ExecutorService executor = ...;
	void renderPage(CharSequence source) {
		final List<ImageInfo> imageInfos = scanForImageInfo(source);
		Callable<List<ImageData>> task = new Callable<List<ImageData>>() {
			public List<ImageData> call() {
				List<ImageData> result = new ArrayList<ImageData>();
				for (ImageInfo imageInfo : imageInfos)
					result.add(imageInfo.downloadImage());
				return result;
			}
		};
			
		Future<List<ImageData>> future = executor.submit(task);
		
		renderText(source);
		
		try {
			List<ImageData> imageData = future.get();
			for (ImageData data : imageData)
				renderImage(data);
		} catch (InterruptedException e) {
			// Re-assert the thread's interrupted status
			Thread.currentThread().interrupt();
			// We don't need the result, so cancel the task too
			future.cancel(true);
		} catch (ExecutionException e) {
			throw launderThrowable(e.getCause());
		}
	}
}
```
	
这段代码模拟了一个 HTML 网页渲染的过程。整个渲染过程分成 HTML 文本的渲染和图片的下载及渲染。这段代码为了提高渲染效率，先提交图片的下载任务，然后在渲染文本，文本渲染完毕之后再去渲染图片。由于图片下载是 IO 密集操作，HTML 文本渲染是 CPU 密集操作，所以让两者并发运行可以提高效率。

# Future 的局限性
## 获取已完成的任务
看到这里，肯定会有人说，为什么只用一个线程去下载所有的图片。如果用多线程去下载图片，效率岂不是更高。的确是这样，但是在提交图片下载之后，如何去从多个 Future 那里获得下载结果呢？依次调用 Future.get() 是个解决办法，但是那样效率并不高，因为第一个有可能是下载速度最慢的，这样会拖累整个页面的渲染，因为我们希望下载完一个图片就渲染一个。

为了解决这个问题，我们可以这样写

	public void renderPage(CharSequence source) {
        List<ImageInfo> imageInfos = scanForImageInfo(source);

        Queue<Future<ImageData>> imageDownloadFutures = new LinkedList<Future<ImageData>>();
        for (final ImageInfo imageInfo : imageInfos) {
            Future<ImageData> future = executorService.submit(new Callable<ImageData>() {
                @Override
                public ImageData call() throws Exception {
                    return imageInfo.downloadImage();
                }
            });
            imageDownloadFutures.add(future);
        }

        renderText(source);

        Future<ImageData> future;
        while ((future = imageDownloadFutures.poll()) != null) {
            if (future.isDone()) {
                if (!future.isCancelled()) {
                    try {
                        renderImage(future.get());
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        // We don't need the result, so cancel the task too
                        future.cancel(true);
                    } catch (ExecutionException e) {
                        System.out.println(e.getMessage());
                        renderImage(ImageData.emptyImage());
                    }
                }
            } else {
                imageDownloadFutures.add(future);
            }

            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                System.out.println("Interrupt images download.");
            }
        }

        executorService.shutdownNow();
        System.out.println("Finish the page render.");
    }
	
这段代码是不是很长，其实我们不用这么辛苦，JDK 已经替我们考虑了这个问题。但是这个话题超出了本期范围，我会在接下来的文章里讲到如何更好地解决这个问题。

# Future 的使用范围
从上面的例子我们看到，Future 是有其局限性的。Future 主要功能在于获取任务执行结果和对异步任务的控制。但如果要获取批量任务的执行结果，从上面的例子我们已经可以看到，单使用 Future 是很不方便的。其原因在于：一是我们没有好的方法去获取第一个完成的任务；二是 Future.get 是阻塞方法，使用不当会造成线程的浪费。解决第一个问题可以用 CompletionService 解决，CompletionService 提供了一个 take() 阻塞方法，用以依次获取所有已完成的任务。对于第二个问题，可以用 Google Guava 库所提供的 ListeningExecutorService 和 ListenableFuture 来解决。这些都会在后面的介绍。

除了获取批量任务执行结果时不便，Future 另外一个不能做的事便是防止任务的重复提交。要做到这件事就需要 Future 最常见的一个实现类 FutureTask 了。《Java Concurrency in Practice》中的例子“Listing 5.19. Final Implementation of Memoizer”便展示了如何使用 FutureTask 做到这一点。