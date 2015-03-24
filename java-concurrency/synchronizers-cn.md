��ԭ��д��һƪ��ͬ��������£�������Ϊ OSChina �༭����Ե�ʣ���ʽ�����ˣ�������дһƪ��ԭ����ɾ�����ղ�ԭ�ĵ����ѶԲ�ס����

���˵һ�� JUC �е�ͬ����������Ҫ�ĳ�Ա��`CountDownLatch`��`CyclicBarrier` �� `Semaphore`����֪����û�г�ѧ�߾��������������ֲ�̫�üǣ����������� JUC �н�Ϊ���õ�ͬ������ͨ�����ǿ��Է����ʵ�ֺܶ��߳�֮��Э���Ĺ��ܡ�������Ĵ������ JDK �ĵ���
 
# CountDownLatch 
ֱ��������ǵ�����(CountDown)����(Latch)������������˵�����ŵ���˼����˼�������ֹǰ�������������ָ `CountDownLatch.await()` �����ڵ�����Ϊ0֮ǰ��������ǰ�̡߳�

## ����
`CountDownLatch` �����ú� `Thread.join()` �������ƣ�������һ���̺߳�����һ���̵߳�Э�������磬���߳�����һ���֮ǰ��Ҫһϵ�е�׼��������ֻ����Щ׼����������ɣ����̲߳��ܼ������Ĺ�������Щ׼�������˴˶��������Կ��Բ���ִ��������ٶȡ�����������¾Ϳ���ʹ�� `CountDownLatch` Э���߳�֮��ĵ����ˡ���ֱ�Ӵ����̵߳������Java 5.0 ֮ǰ�������ǿ���ʹ�� `Thread.join()`���� JUC ���ֺ���Ϊ�̳߳��е��̲߳���ֱ�ӱ����ã����Ծͱ���ʹ�� `CountDownLatch` �ˡ�
 
## ʾ��
�����������ӿ������Ϊ F1 ������ά�޹��̣�ֻ�� startSignal �����Ա�ʾͣ�����������ֲ�̫���ϣ������´�֮��ά�޹��ſ�ʼ�ɻֻ�е����й�����ɹ���֮���������ܼ�����

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

�� `startSignal.await()` �������̣߳��� `startSignal.countDown()` ������֮������ `Worker` �߳̿�ʼִ�� `doWork()` ���������� `Worker��doWork()` �Ǽ���ͬʱ��ʼִ�еġ��� `Worker.doWork()` ִ����Ϻ󣬵��� `doneSignal.countDown()`�������� `Worker` �߳�ִ�����֮�����̼߳���ִ�С�

# CyclicBarrier 
`CyclicBarrier` ���������ѭ��դ����ѭ���ϰ�ʲô�ģ������е��Ť�ġ����Ի��Ǳ����ˣ�ֻ����᲻���Դ�����������Ҫ�ķ�������һ����`await()`��`await()` ����ÿ������һ�Σ�����������1��������ס��ǰ�̡߳�����������0ʱ����������������ڴ� `CyclicBarrier` �����������߳̿�ʼ���С�����֮������ٴε��� `await()` �������������ֻ��� N-1����һ�����¿�ʼ������� Cyclic �ĺ������ڡ�

`CyclicBarrier` ��ʹ�ò����ѣ�����Ҫע��������ص��쳣�����˳������쳣��`CyclicBarrier.await()` �������׳�һ�����е� `BrokenBarrierException`������쳣�����ڵ�ĳ���߳��ڵȴ��� `CyclicBarrier` ʱ���жϻ�ʱ������ʱ������ͬ������� `CyclicBarrier` �ϵȴ����̱߳���ܵ� `BrokenBarrierException`����˼����˵��ͬ־�ǣ�����ˣ��и�С����Ѿ����ˣ���������������п��ܻ�һֱ����ȥ�����и��ظ��Ұɡ�

`CyclicBarrier.await()` �������з���ֵ��������ʾ��ǰ�߳��ǵڼ���������� Barrier ���̡߳�
 
�� `CountDownLatch` һ����`CyclicBarrier` ͬ�����Կ����ڹ��캯�����趨�ܼ���ֵ���� `CountDownLatch` ��ͬ���ǣ�`CyclicBarrier` �Ĺ��캯�������Խ���һ�� `Runnable`������ `CyclicBarrier` ���ͷ�ʱִ�С�

> NOTE: CyclicBarrier �Ĺ���Ҳ������ CountDownLatch ��ʵ��
 
## ʾ��
CyclicBarrier ��Ӧ�ã���Ȼ��������ӻ��� CountDownLatch Ҳ�ǿ���ʵ�ֵģ��ܼ򵥣��Ͳ�˵��ôд�ˣ� 

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

## `CyclicBarrier` �� `CountDownLatch` ���÷��ϵĲ�ͬ
`CountDownLatch` ������һ���̺߳���һ�����߳�֮��Ĺ���Э����һ�����̵߳ȴ�һ�鹤���̵߳�������ϲż�������ִ����ʹ�� `CountDownLatch` ����Ҫ������`CyclicBarrier` ����һ������̣߳�����һ���߳���Ҫ��һ��ʱ����ϴ��һ�£�����ͬʱ��ʼһ�����������⣬`CyclicBarrier` ��ѭ�����Ժ͹��캯�������ܵ� `Runnable` ����Ҳ�� `CountDownLatch` �����߱��ġ�
 
# Semaphore 
`Semaphore` ֱ�����ź��������ܳ������������������⡣��Ȼ����Ϊ�ڼ������ѧ��������������Ѿã����Բ����Ҹġ����Ĺ��ܱȽϺ���⣬����ͨ�����캯���趨һ����������ɣ�Ȼ��ͨ�� `acquire` ���������ɣ�`release` �����ͷ���ɡ������� `tryAcquire` �� `acquireUninterruptibly` ���������Ը����Լ�����Ҫѡ��
 
## ʾ����`Semaphore` ������Դ����
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

�������ʾ���� Semaphore ���÷�ûʲô�ɶི�ġ���Ҫ���Ե���������������ͬ������������������Ӧ��ûʲôӰ�죬��Ϊ��Ҫ�Ƕ�һ�� boolean ������һ�� O(n) �Ĳ���������ÿ��ѭ������Ĳ����ܼ򵥣������ٶȺܿ졣������֪�� JUC �����̳߳صĿ�������ô���ģ����˲��ţ���û�����ǿ�Դ���룬�ÿտ�������֪����Ҳ����˵˵��
 
# ���һ�仰�ܽ�
`CountDownLatch` ����ʹһ���̵߳���һ���̶߳��������ټ����ܣ�`CyclicBarrier` �ܹ�ʹһ���߳���һ��ʱ����ϴﵽͬ����������һ��ʼִ��ȫ���������һ��������ͬʱ�����ǿ���ѭ��ʹ�õģ�`Semaphore` ��ֻ����һ���������߳�ͬʱִ��һ������