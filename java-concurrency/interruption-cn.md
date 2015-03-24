��ν�߳��жϣ���ʵ������ֹһ���̡߳���ʹ�� Java �߳�ʱ�������߳����������������ܶ�ʱ��Ҳ��Ҫ��ǰ����һ���̵߳�ִ�й��̡�Thread ������һ���� start() ���Ӧ�� stop() ���������Դ��ⲿ����һ���̵߳�ִ�С�������������Ǽ����Ƽ�ʹ�õģ���Ϊ���ⲿǿ�н���һ���̵߳�ִ�У��ᵼ�²���Ԥ֪�Ĵ�����Ϊ�����������ڴ����ʱ�����һ���̵߳�ִ�С�

���ԣ��� Java �̻߳����У���������һ�ֽ����̵߳ķ�ʽ���Ǿ����жϡ��жϼ����֮�������߳��ⲿ��������һ�����ֵ�����߳��ڲ���ִ��ʱ���Լ�����ֵ������֪���߳��Ƿ�Ӧ�ý����ˡ�

# 1. �ܹ������жϵķ���
���� Thread.interrupt() �������⣬���� JDK �еķ���Ҳ�������жϣ�Ҳ��ͨ������ `Thread.interrupt()` ��ʵ�ֵģ���

* `FutureTask.cancel()`
* `ExecutorService.shutdownNow()` �������������̳߳��������̵߳��жϷ��������������ǿ��еĻ��������еġ��� `ExecutorService.shutdown()` ����ֻ���жϿ��е��̡߳�

����ֻ�Ǿ����� JDK ��Ӧ�õ����߳��жϵ����ӣ����������ӻ��кܶ࣬�Ͳ�һһ�о��ˡ���Ȼ��Ϊ������Ӧ�жϣ�������д�� Runnable �� Callable �����У�����ͨ�� `Thread.isInterrupted()`��`Thread.interrupted()` ���������߲��� `InterruptedException` �ȵ��ж��쳣�������߳��жϲ����������߳��ǲ���������ǰ�����ġ�

# 2. �ܹ����жϵķ���
�� JDK ���������Ϳ���У�����Ӧ�жϵķ����Ǻܶ�ġ������г����������� JDK ������Ӧ�жϵķ�����

* Thread.sleep()
* Object.wait()
* BlockingQueue.put(), BlockingQueue.take()
* ReentrantLock.lockInterruptibly(), Condition.await()
* ServerSocketChannel.accept(), SocketChannel.open()

�ȵ�

JDK ������Ӧ�жϵķ��������϶����׳��쳣����Щ���������ɱ���Ϊ���ࣺһ���ǲ�����صģ�һ���� IO ��صġ�

# 3. �жϵĴ���
## 3.1 ���� InterruptedException��Ҳ���������ж��쳣��
`InterruptedException` ��������жϱ�����ʽ��������δ��� `InterruptedException` ���Ϊ Java �ж�֪ʶ�еı��޿Ρ����ⷽ�� IBM developerWorks ����ƪ���½��ĺܺã���������Ĳο������л��г����ӡ�������Ͷ���ƪ������һ���ܽᣬ��λ���Ϳ���ȥ���Ǳ������Ի��ϸ��֪ʶ��

���� InterruptedException �������¼��ַ�ʽ������ʹ�õĴ���������� [Java ������ʵ��: ���� InterruptedException](http://www.ibm.com/developerworks/cn/java/j-jtp05236.html)����

### 3.1.1 ֱ�������׳�
���쳣�����κδ���ֱ������÷����ĵ�����

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

### 3.1.2 �� catch ������������׳�
��Ϊ `InterruptedException` ���׳������Ϸ���ִ�У�ʹ���ڽ��еĹ���ֻ���һ���֡�����Щ����£������Ҫ��������ع��Ĵ��������������������Ҫ�� catch ���н��д���֮���������׳� `InterruptedException`��

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

### 3.1.3 ���׳� InterruptedException ʱҪ�ָ��ж�״̬
�ܶ�ʱ����������ʵ�ֵĽӿڶ�������ƣ���ܿ����޷��׳� `InterruptedException`������ʵ�� `Runnable` �ӿ��Ա�дҵ����롣��ʱ������޷��������׳� `InterruptedException` �ˡ���ʱ��Ӧ��ʹ�� `Thread.currentThread().interrupt()` ����ȥ�ָ��ж�״̬����Ϊ�����������׳� `InterruptedException` ʱ�������ǰ�̵߳��ж�״̬�������ʱ���ָ��ж�״̬��Ҳ���׳� `InterruptedException`�����ж���Ϣ��ᶪʧ���ϲ������Ҳ���޷���֪�жϵķ������������п��ܵ��������޷���ȷ��ֹ�������ʽ��

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

���������Ҫ����жϵ��������ڲ������첽�������������ֹ�����ԣ������Ĵ������̵� `InterruptedException` �����׳�ʱ��������������޷�����ȷ��ֹ�������ʽ����Ҳ���Բ��ٻָ��жϡ�

���� Runnable �����ӣ���Ҷ�֪�� `Runnable` �ܶ�ʱ���������̳߳��У����̳߳��ṩ�߳������С�����ͨ������Ϊ����Ķ���������ʹ�õģ�Ҳ����˵���̳߳غ� `Runnable` ʵ��֮�䣬û�б�ĵ��ò��ˡ���ô�� try-catch `InterruptedException` ֮�󣬱�ɲ����ڻָ��߳��ж��ˣ���Ȼ������лع��ȵ����󣬻�����Ҫʵ�ֵģ�����Ϊͨ���쳣�Ĳ������Ѿ�������ȷ��ֹ�߳��ˡ�

���ǣ���������������������д�ģ����� `InterruptedException` �ķ����ᱻ�����ġ����̳߳���ķ������á������� A, B ����������A �� B �������ã�A �в��� `InterruptedException` ��û�лָ��߳��жϣ��� B ������һ������ѭ����ͨ������߳��ж����˳��������ڵ��� A ֮���и������ķ������Ǳ������߳��޷�������������ֹ�����������

### 3.1.4 �Լ��׳� InterruptedException
��ʱ������Ҫ���������С��ش����һ�� `InterruptedException` �Ա�ʾ�жϵķ�������Ȼ�������ʱ����Ҳ��Ҫʹ�� `Thread.isInterrupted()` �� `Thread.interrupted()` ������жϵķ������Ǿ��������������е���һ������������ء���ʵ����ǰ��Ĳ�������֪�����׳� `InterruptedException` ʱ���߳��ж�״̬��Ҫ�������Ϊʲô���Լ�����ɣ��Ǻǣ�������� `Thread.interrupted()` �����á�

_��ʵ����Ҫ�ǿ� JDK Դ���룬�ͻᷢ�֣�JDK �в�����Ҳ����ô����_

> **NOTE: ʹ�� Thread.interrupt() �� InterruptedException �е����ַ�����ʾ�жϣ�**�����ᵽ��һ����������ڽӿڵ����ƶ��޷��׳� InterruptedException����ʱ�����ѡ��ֻ���� Thread.interrupt() �ָ��жϡ��������������������ʱ���Ƽ�ʹ�� InterruptedException ����ʾ�жϡ������������׳� InterruptedException ʱ���������ڸ��ߵ����ߣ�������������ܻỨ�Ѻܶ��ʱ�䣬�������ͨ���߳��ж�����ֹ���á�ͨ�� InterruptedException ����ʾ�жϣ��������������ӦҲ��Ѹ�١�

## 3.2 �޷����жϵ����
### 3.2.1 synchronized
������ synchronized �������������޷����жϵģ������Ҫ���Ա��жϵ���������ʹ�� Java 5 concurrent �е� Lock��

### 3.2.2 Java IO�������� NIO �� AIO��
�� Java IO �����˽���˶�֪����`ServerSocket.accept()` ��һ���������������� `Thread.interrupt()` ��������Ӱ�졣Ҫ����ֹ `ServerSocket.accept()` �ĵȴ���Ψһ�������ǵ��� close() ������ 

## 3.3 �߳��ж��� Java IO
����˵����ͳ�� Java IO �е����������޷���Ӧ�߳��жϣ���Ϊ Java IO ���ֵ�ʱ��û���жϻ��ơ��� Java 1.4 ����� Java NIO �Ѿ����Ժܺõ�֧�� Java �߳��ж��ˡ�

�����ʹ�� XXXChannel���߳��жϻᵼ�� accept(), open() �����������׳� ClosedByInterruptException��

# 4. �ο�

* [Java ������ʵ��: ���� InterruptedException](http://www.ibm.com/developerworks/cn/java/j-jtp05236.html)
* [�̹߳����ģ������̵߳��жϻ���](http://ifeve.com/thread-manager-4/)
* [��ϸ����Java�жϻ���](http://www.infoq.com/cn/articles/java-interrupt-mechanism)
* [����ȡ��(Cancellation)](http://ifeve.com/cancellation/)