redisson在redis基础上提供了一系列的分布式Java对象和服务，如分布式锁，集合，同步器，分布式信号量等等。

[redisson官方网站](https://redisson.org/)，[redisson中文文档](https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95)

# 分布式锁的实现

redisson对分布式锁的实现十分完善，`public interface RLock extends Lock, RLockAsync`实现了`java.util.concurrent.locks.Lock`接口，提供了可重入锁，公平锁，读写锁等语义，还提供了Semaphore，CountDownLatch的支持。

并且也有MultiLock,RedLock，防止单结点故障问题。

先看分布式锁提供的功能：

```java
RLock lock = redisson.getLock("anyLock");

// 最常见的使用方法
lock.lock();

// 加锁以后10秒钟自动解锁，无需调用unlock方法手动解锁
lock.lock(10, TimeUnit.SECONDS);

// 异步获取锁
lock.lockAsync();
lock.lockAsync(10, TimeUnit.SECONDS);
Future<Boolean> res = lock.tryLockAsync(100, 10, TimeUnit.SECONDS);

// 构建公平锁
RLock fairLock = redisson.getFairLock("anyLock");

// 构建读写锁
RReadWriteLock rwlock = redisson.getReadWriteLock("anyRWLock");
rwlock.readLock().lock();
rwlock.writeLock().lock();
```

## 分布式锁的实现逻辑

1. 循环获取锁

   ```java
   /**
   * 获取分布式锁的重要方法
   */
   private void lock(long leaseTime, TimeUnit unit, boolean interruptibly) throws InterruptedException {
       // 当前线程ID
       long threadId = Thread.currentThread().getId();
       // 先去异步判断锁是否被释放
       Long ttl = tryAcquire(-1, leaseTime, unit, threadId);
       // lock acquired
       if (ttl == null) {
           return;
       }
   
       RFuture<RedissonLockEntry> future = subscribe(threadId);
       if (interruptibly) {
           commandExecutor.syncSubscriptionInterrupted(future);
       } else {
           commandExecutor.syncSubscription(future);
       }
   
       try {
           // 上面首次尝试没有获取锁后，这里不停循环 尝试获取锁
           while (true) {
               ttl = tryAcquire(-1, leaseTime, unit, threadId);
               // lock acquired 代表锁被释放了
               if (ttl == null) {
                   break;
               }
   
               // waiting for message
               if (ttl >= 0) {
                   try {
                       // ttl >= 0代表锁被其他请求占有，等待ttl时间
                       future.getNow().getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                   } catch (InterruptedException e) {
                       // 如果允许中断，就抛出InterruptedException
                       if (interruptibly) {
                           throw e;
                       }
                       future.getNow().getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                   }
               } else {
                   // 如果ttl为null或者 < 0表示锁已经释放，可以去获取了
                   if (interruptibly) {
                       future.getNow().getLatch().acquire();
                   } else {
                       future.getNow().getLatch().acquireUninterruptibly();
                   }
               }
           }
       } finally {
           unsubscribe(future, threadId);
       }
   }
   ```

2. 异步获取锁，成功后如果没有指定leaseTime注册watchdog定时任务进行续约

   ```java
   private <T> RFuture<Long> tryAcquireAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId) {
       RFuture<Long> ttlRemainingFuture;
       if (leaseTime != -1) {
           ttlRemainingFuture = tryLockInnerAsync(waitTime, leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
       } else {
           // leaseTime不传默认是30秒
           ttlRemainingFuture = tryLockInnerAsync(waitTime, internalLockLeaseTime,
                                                  TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
       }
       ttlRemainingFuture.onComplete((ttlRemaining, e) -> {
           if (e != null) {
               return;
           }
   
           // 如果获取锁没有抛出异常，且ttlRemaining不为空，
           // lock acquired
           if (ttlRemaining == null) {
               // 如果用户设置了leaseTime，就不延期了
               if (leaseTime != -1) {
                   internalLockLeaseTime = unit.toMillis(leaseTime);
               } else {
                   // 就注册watchdog后续进行锁延期
                   scheduleExpirationRenewal(threadId);
               }
           }
       });
       return ttlRemainingFuture;
   }
   
   /**
   * lua脚本去获取锁的情况
   */
   <T> RFuture<T> tryLockInnerAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
       // 如果hash不存在，就用hincrby为名为 的hash结构存入 id + ":" + threadId = 1这样一个键值对，并设置过期时间leaseTime
       return evalWriteAsync(getRawName(), LongCodec.INSTANCE, command,
                             "if (redis.call('exists', KEYS[1]) == 0) then " +
                             "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                             "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                             "return nil; " +
                             "end; " +
                             // 如果hash中已经存在id + ":" + threadId的键值对，就+1，实现锁的可重入性
                             "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                             "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                             "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                             "return nil; " +
                             "end; " +
                             "return redis.call('pttl', KEYS[1]);", // 如果已经存在，就返回hash的过期时间，后面要根据此判断锁的状态
                             Collections.singletonList(getRawName()), unit.toMillis(leaseTime), getLockName(threadId));
   }
   ```

   