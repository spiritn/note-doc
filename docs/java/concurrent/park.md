
好的，这是一个非常经典的Java并发面试题。我们来详细解析 `LockSupport.park()` 和 `Thread.sleep()` 的区别。

## 一、核心区别对比

| 特性 | `LockSupport.park()` | `Thread.sleep()` |
|-----|---------------------|------------------|
| **所属包** | `java.util.concurrent.locks.LockSupport` | `java.lang.Thread` |
| **唤醒机制** | 需要显式调用 `unpark(thread)` | 超时自动唤醒，可被 `interrupt()` 中断 |
| **中断响应** | 被中断时**立即返回**，不会抛异常 | 被中断时**抛出InterruptedException** |
| **锁的状态** | **不会释放**已持有的锁 | **不会释放**已持有的锁 |
| **时间精度** | 无限期或纳秒级超时 | 毫秒级（或毫秒+纳秒） |
| **许可证机制** | 有（核心特性） | 无 |
| **面向对象** | 以**线程对象**为参数操作其他线程 | 静态方法，操作当前线程 |

---

## 二、深入理解 park/unpark 机制

### 许可证（Permit）概念
`LockSupport` 使用了一个隐式的"许可证"机制：
- 每个线程有一个许可证（最多为1）
- `unpark(thread)`：给指定线程发放一个许可证
- `park()`：消耗一个许可证，如果已有许可证则立即返回

### 代码示例
```java
import java.util.concurrent.locks.LockSupport;

public class ParkExample {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println("子线程开始执行");
            
            // 情况1：先unpark后park - 立即继续执行
            // LockSupport.unpark(Thread.currentThread());
            
            System.out.println("子线程即将park");
            LockSupport.park(); // 等待许可证
            System.out.println("子线程被唤醒");
        });

        thread.start();
        Thread.sleep(1000); // 主线程等待1秒
        
        System.out.println("主线程unpark子线程");
        LockSupport.unpark(thread); // 发放许可证
    }
}
```

---

## 三、使用场景和优势

### 什么时候使用 park/unpark？

1. **构建高级同步工具**
   ```java
   // 实现一个简单的锁
   class SimpleLock {
       private volatile Thread owner = null;
       
       public void lock() {
           Thread current = Thread.currentThread();
           while (!compareAndSetOwner(null, current)) {
               LockSupport.park(); // 等待锁释放
           }
       }
       
       public void unlock() {
           owner = null;
           LockSupport.unpark(waitingThread); // 唤醒等待线程
       }
   }
   ```

2. **精确的线程控制**
   ```java
   // 线程间精确协调，避免sleep的不确定性
   public class ThreadCoordination {
       private Thread workerThread;
       
       public void startWorker() {
           workerThread = new Thread(() -> {
               while (!Thread.currentThread().isInterrupted()) {
                   // 执行任务...
                   LockSupport.park(); // 等待下一轮调度
               }
           });
           workerThread.start();
       }
       
       public void wakeupWorker() {
           LockSupport.unpark(workerThread); // 精确唤醒
       }
   }
   ```

3. **性能敏感的等待**
   ```java
   // 在自旋锁中减少空转
   while (!canProceed()) {
       if (spinCount > MAX_SPINS) {
           LockSupport.parkNanos(1000); // 短暂park而不是忙等待
       } else {
           spinCount++;
       }
   }
   ```

### park 的优势

1. **更精确的线程控制**：可以精确唤醒特定线程
2. **不会抛中断异常**：代码更简洁，不需要处理受检异常
3. **先唤醒后等待仍有效**：`unpark` 发生在 `park` 之前时，`park` 不会阻塞
4. **高性能**：底层使用操作系统原语，比 `Object.wait()` 更高效

---

## 四、面试常见问题

### 基础问题
1. **park/sleep 的主要区别是什么？**
    - 唤醒机制、中断响应、许可证概念

2. **为什么 park 不会释放锁？**
    - 因为 park 是底层的线程阻塞原语，不涉及监视器锁

3. **park 和 wait 的区别？**
    - `wait()` 必须在 synchronized 块中使用，会释放锁
    - `park()` 可以在任何地方使用，不会释放锁

### 进阶问题
4. **如果先调用 unpark 再调用 park，会发生什么？**
    - 线程不会阻塞，因为许可证已经被消费

5. **park 被中断时会怎样？**
    - 立即返回，但不会抛异常，可以通过 `Thread.interrupted()` 检查中断状态

6. **如何在 park 时设置超时？**
    - 使用 `parkNanos(long nanos)` 或 `parkUntil(long deadline)`

### 实战问题
7. **下面代码的输出顺序是什么？**
   ```java
   Thread t = new Thread(() -> {
       LockSupport.unpark(Thread.currentThread());
       System.out.println("A");
       LockSupport.park();
       System.out.println("B");
   });
   t.start();
   ```
   **答案**：A 和 B 都会输出，因为先 unpark 后 park 不会阻塞

8. **设计题：用 park/unpark 实现生产者消费者**
   ```java
   class ParkBlockingQueue<T> {
       private final Queue<T> queue = new LinkedList<>();
       private final Thread consumerThread;
       
       public ParkBlockingQueue() {
           consumerThread = new Thread(() -> {
               while (!Thread.interrupted()) {
                   if (queue.isEmpty()) {
                       LockSupport.park();
                   }
                   T item = queue.poll();
                   // 处理item...
               }
           });
       }
       
       public void put(T item) {
           queue.offer(item);
           LockSupport.unpark(consumerThread);
       }
   }
   ```

---

## 五、最佳实践和注意事项

1. **总是检查中断状态**
   ```java
   LockSupport.park();
   if (Thread.interrupted()) {
       // 处理中断逻辑
   }
   ```

2. **避免长时间 park**
   ```java
   // 使用带超时的版本
   LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(5));
   ```

3. **结合 volatile 变量使用**
   ```java
   private volatile boolean condition = false;
   
   public void waitForCondition() {
       while (!condition) {
           LockSupport.park();
       }
   }
   ```

## 总结

- **`park/unpark`** 是更底层的线程阻塞机制，用于构建高级同步工具
- **`sleep`** 是简单的延时工具，适合简单的等待场景
- 在需要精确线程控制、构建锁、性能优化时选择 `park/unpark`
- 在简单延时场景使用 `sleep` 更直观

掌握这些区别对于深入理解Java并发编程和应对技术面试都非常重要！