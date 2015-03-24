# ���
Future �� Java 5 JUC ���е�һ���ӿڣ���Ҫ�ṩ�����๦�ܣ�
## �������Ļ�ȡ
��������� get �����ṩ������������ʽ�����ء�get ��������ʹ�������ܼ򵥣���Ҫע����������׳����쳣��

* ExecutionException �� Callable �� Runnable ���׳����쳣�ķ�װ������ͨ�� ` Throwable.getCause()` ������þ����쳣��
* CancellationException �ڵ��� get ʱ����ͨ�� `Future.cancel()` ������ȡ�����׳����쳣������� ����ʱ�쳣����������е��� `Future.cancel()` �ĵط����ǻ�����Ҫ����ġ�
* TimeoutException `V get(long timeout, TimeUnit unit)` ������ʽ���׳��ĳ�ʱ�쳣��

## ����ȡ��

# ͨ�����뿴 Future ��ʹ��
�����ȿ�һ�δ��룬��������ǡ�Java Concurrency in Practise���� ��Listing 6.13. Waiting for Image Download with Future.����

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
	
��δ���ģ����һ�� HTML ��ҳ��Ⱦ�Ĺ��̡�������Ⱦ���̷ֳ� HTML �ı�����Ⱦ��ͼƬ�����ؼ���Ⱦ����δ���Ϊ�������ȾЧ�ʣ����ύͼƬ����������Ȼ������Ⱦ�ı����ı���Ⱦ���֮����ȥ��ȾͼƬ������ͼƬ������ IO �ܼ�������HTML �ı���Ⱦ�� CPU �ܼ����������������߲������п������Ч�ʡ�

# Future �ľ�����
## ��ȡ����ɵ�����
��������϶�������˵��Ϊʲôֻ��һ���߳�ȥ�������е�ͼƬ������ö��߳�ȥ����ͼƬ��Ч�����Ǹ��ߡ���ȷ���������������ύͼƬ����֮�����ȥ�Ӷ�� Future ���������ؽ���أ����ε��� Future.get() �Ǹ�����취����������Ч�ʲ����ߣ���Ϊ��һ���п����������ٶ������ģ���������������ҳ�����Ⱦ����Ϊ����ϣ��������һ��ͼƬ����Ⱦһ����

Ϊ�˽��������⣬���ǿ�������д

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
	
��δ����ǲ��Ǻܳ�����ʵ���ǲ�����ô���࣬JDK �Ѿ������ǿ�����������⡣����������ⳬ���˱��ڷ�Χ���һ��ڽ������������ｲ����θ��õؽ��������⡣

# Future ��ʹ�÷�Χ
��������������ǿ�����Future ����������Եġ�Future ��Ҫ�������ڻ�ȡ����ִ�н���Ͷ��첽����Ŀ��ơ������Ҫ��ȡ���������ִ�н��������������������Ѿ����Կ�������ʹ�� Future �Ǻܲ�����ġ���ԭ�����ڣ�һ������û�кõķ���ȥ��ȡ��һ����ɵ����񣻶��� Future.get ������������ʹ�ò���������̵߳��˷ѡ������һ����������� CompletionService �����CompletionService �ṩ��һ�� take() �����������������λ�ȡ��������ɵ����񡣶��ڵڶ������⣬������ Google Guava �����ṩ�� ListeningExecutorService �� ListenableFuture ���������Щ�����ں���Ľ��ܡ�

���˻�ȡ��������ִ�н��ʱ���㣬Future ����һ�����������±��Ƿ�ֹ������ظ��ύ��Ҫ��������¾���Ҫ Future �����һ��ʵ���� FutureTask �ˡ���Java Concurrency in Practice���е����ӡ�Listing 5.19. Final Implementation of Memoizer����չʾ�����ʹ�� FutureTask ������һ�㡣