# 常见工具类
1. juc核心**AQS**：AbstractQueuedSynchronizer和synchronized的monitor类似，实现了一套管程控制框架，用volatile int的state作为锁对象并记录重入次数，CAS修改state来上锁/释放，同样实现了entryList和waitSet供线程阻塞等待。
2. **ReentrantLock**：在AQS基础上完成的和synchronzied类似的锁，优点在于可尝试tryLock、可超时、可打断、可设置为公平锁、可通过Condition灵活添加waitSet。性能上由于synchronized加入锁升级和自旋优化后差别不大。
3. **ReentrantReadWriteLock**：读写锁，读-读不互斥，提高资源共享效率。state高16位记录读锁，低16位记录写锁，阻塞队列中读线程被唤醒时递归唤醒后继读线程。
4. **Semaphore**：信号量，限制并行访问共享资源的线程的个数，类似于抢车位。可用于单机限流，acquire申请资源访问权限，realese释放，state为最大并行线程数。
5. **CountDownLatch**：用于线程间同步协作，一个线程await等待其它线程完成任务时countDown，state为count数。
6. **CyclicBarrier**：循环栅栏，构造参数int parties、 Runnable barrierAction。state为parties，子任务线程await等待时state-1，state为0时调用barrierAction的run方法；可循环使用。
7. **Thread Pool**：线程池也在JUC里
# ReentrantLock
## 原理
ReentrantLock有一个**内部类Sync**继承了AQS，AQS作为一个构建同步锁的框架，维护了阻塞队列和锁状态state，同时ReentrantLock实现了Lock，可以作为锁使用
这样理解：lock是一个锁的用途，比如lock（）和unlock（）上锁和放锁，是一种抽象的概念锁，而AQS是一个构建锁的框架，通过这个框架中的一些构造才能真正制造出一把能用的锁
## 各机制的实现
 1. **可尝试** `tryLock()`：
    - 当调用 `tryLock()` 时，会尝试获取锁，如果锁是可用的，则立即获取锁并返回 `true`，否则立即返回 `false`，而不会阻塞当前线程。
    - 使用 `compareAndSet` 或 `CAS` 操作尝试将锁的状态从无锁改为有锁，成功则获取锁。
2. **可超时** `tryLock(long time, TimeUnit unit)`：
    - 类似于 `tryLock()`，但是在指定的时间范围内尝试获取锁。如果在指定时间内获取到了锁，则返回 `true`，否则超时后返回 `false`。
    - 使用 `ReentrantLock` 的 `Sync` 内部类的 `tryAcquireNanos` 方法，通过计时的方式实现。
3. **可打断** `lockInterruptibly()`：
    - `lockInterruptibly()` 方法是阻塞的，如果当前线程被中断，则会抛出 `InterruptedException` 异常，同时释放已经获取的锁。
    - 在内部实现中，`ReentrantLock` 使用了 `AbstractQueuedSynchronizer`（AQS）的 `acquireInterruptibly()` 方法，该方法支持可打断的阻塞。
4. **公平锁和非公平锁**：
    - 在公平锁模式下，锁的获取是按照请求的顺序进行的，先请求的线程先获得锁；在非公平锁模式下，锁的获取是非公平的，它允许在同一时间点内请求锁的线程立即获取锁，而不管其他线程是否等待。
    - 在 `ReentrantLock` 的构造函数中，可以通过传递参数来选择公平或非公平模式，对应的参数是 `true`（公平锁）和 `false`（非公平锁）。公平锁会维护一个等待队列，按照线程请求锁的顺序来获取锁，而非公平锁则更倾向于允许后来的线程立即获取锁，提高了性能。
 5. **可重入性**（Reentrancy）：
    - `ReentrantLock` 支持可重入性，即同一个线程可以多次获得同一把锁而不会死锁。
    - 在内部实现中，`ReentrantLock` 使用了一个计数器来记录同一个线程对锁的获取次数，每次释放锁时计数器减一，直到计数器归零才完全释放锁
6. 补充说明：
- `ReentrantLock` 的核心实现在于其内部的 `Sync` 类，它的两个子类分别是 `NonfairSync` 和 `FairSync`，用于实现非公平锁和公平锁。
- `tryLock()`、可超时`tryLock(long timeout, TimeUnit unit)`、可打断的机制主要是在 `NonfairSync` 中实现的。
- 公平锁的实现主要在 `FairSync` 中，通过队列维护等待线程，按照请求锁的顺序获取锁。。
# Condition
提供了**线程等待和通知**的机制，可以用于实现更灵活的线程协作。
Condition是一个接口，AQS内部实现了它（ConditionObject内部类），并提供了一个获取ConditionObject实例的方法
ReentrantLock继承了AQS，其内部类Sync通过AQS提供的方法获取ConditionObject实例，**每个 ReentrantLock对象都可以有多个关联的Condition**（条件），这样可以在同一个锁上实现多个等待队列，每个队列用于不同的条件，可以实现特定条件下的线程唤醒，也就是“一锁多用”，满足某种条件时，唤醒一部分等待队列中的线程
## Condition 接口主要的方法：
1. **await()：**
    - 使当前线程等待，同时释放当前锁。
    - 当其他线程调用相同对象上的 `signal()` 或 `signalAll()` 方法时，被等待的线程将被唤醒，重新竞争锁。
2. **awaitUninterruptibly()：**
    - 与 `await()` 类似，但不响应中断。
3. **signal()：**
    - 唤醒等待在此条件上的一个线程。
    - 被唤醒的线程将重新尝试获取锁。
4. **signalAll()：**
    - 唤醒等待在此条件上的所有线程。
5. **await(long time, TimeUnit unit)：**
    - 在指定的时间内等待，超时后自动唤醒。

Condition 的 await 和 signal 与 Object 的 wait 和 notify 有什么区别
1. **关联对象**：
    - `Condition` 是与 `ReentrantLock` 相关联的，通过 `ReentrantLock.newCondition()` 创建。一个 `ReentrantLock` 可以有多个与之相关联的 `Condition`，每个用于不同的等待和通知。
    - `Object` 的 `wait` 和 `notify` 是与每个对象关联的，任何对象都可以调用它们。
2. **等待和通知的粒度**：
    - `Condition` 允许精确地选择哪些线程被唤醒，因为一个 `ReentrantLock` 可以有多个 `Condition`。使用 `await` 和 `signal` 可以指定具体的 `Condition`。
    - `Object` 的 `wait` 和 `notify` 是全局的，当调用 `notify` 时，所有在该对象上等待的线程都有可能被唤醒。
3. **可中断性**：
    - `Condition` 的 `await` 方法支持线程等待期间的中断，即线程在等待时可以响应中断。
    - `Object` 的 `wait` 方法在等待时无法响应中断，只能等待被通知或超时。
4. **等待队列管理**：
    - `Condition` 提供了更多的等待队列管理方法，如 `awaitNanos`、`awaitUntil` 等，以及支持等待队列中指定条件的线程。
    - `Object` 的 `wait` 方法较为简单，只支持基本的等待和唤醒。