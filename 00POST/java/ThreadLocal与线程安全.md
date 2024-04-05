# 什么是线程安全
线程安全（Thread Safety）是指在多线程环境下，一个程序或者代码段能够在**并发**执行的情况下正确地执行，不会产生意外的结果。确保线程安全是多线程编程中一个非常重要的概念，因为在多线程环境下，多个线程可能同时访问和修改共享的数据，如果不进行适当的同步和保护，可能导致数据不一致、不确定的行为或者程序崩溃。
## 线程安全的关键点
1. **原子性（Atomicity）：** 原子操作是指一个操作是不可中断的，要么全部执行成功，要么全部不执行，不会出现部分执行的情况。在多线程环境下，确保某个操作是原子的可以避免竞态条件（Race Condition）。
2. **可见性（Visibility）：** 当一个线程对共享变量进行了修改，其他线程应该能够立即看到这个修改。Java中使用`volatile`关键字可以保证变量的可见性。
3. **有序性（Ordering）：** 确保指令执行的顺序按照程序的顺序来执行。在Java中，通过synchronized关键字、Lock接口等机制可以保证代码块的有序性。
4. **不变性（Immutability）：** 使用不可变对象或者通过其他手段确保对象在多线程环境中不能被修改，从而避免竞态条件。
## 实现线程安全
1. **加锁机制：** 使用`synchronized`关键字或者`ReentrantLock`等锁机制来确保在同一时刻只有一个线程能够访问共享资源，从而避免竞态条件。
2. **原子类：** Java提供了一些原子类（如`AtomicInteger`、`AtomicLong`等），这些类的操作是原子的，不需要额外的同步。
3. **线程局部变量（Thread-Local Variables）：** 使用`ThreadLocal`可以为每个线程提供独立的变量副本，避免了共享变量的修改冲突。
4. **不可变对象：** 创建不可变对象，确保对象一旦被创建就不能被修改，从而避免多线程环境中的数据竞争。
5. **使用并发集合：** Java提供了一些并发集合类（如`ConcurrentHashMap`、`CopyOnWriteArrayList`等），它们内部实现了线程安全机制，可以在多线程环境下安全地使用。
# ThreadLocal
1. `ThreadLocal` 是 Java 中的一个特殊类，用于在多线程环境中保持**变量的线程本地副本**。它并不直接保证线程安全，而是通过为每个线程提供独立的变量副本来避免线程间的竞争和共享。
2. `ThreadLocal` 使用一个特殊的数据结构来维护每个线程的变量副本，这样每个线程都可以独立地修改自己的副本而不会影响其他线程的副本。这种方式在一些场景下非常有用，比如在线程池中处理任务时，每个任务都可以独立地访问和修改自己的数据，而不用担心线程安全问题。
3. 使用`ThreadLocal`有两个作用：
	- 线程内资源全局共享
	- 线程间资源互相隔离
### 用法示例
```java
常用方法：get(),set(object),remove()

public class ThreadLocalExample {
    private static final ThreadLocal<Integer> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        // 在主线程设置 ThreadLocal 变量
        threadLocal.set(42);

        // 启动两个新线程，它们会访问各自独立的 ThreadLocal 变量副本
        Thread thread1 = new Thread(() -> {
            threadLocal.set(100);
            System.out.println("Thread 1: " + threadLocal.get());
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("Thread 2: " + threadLocal.get());
        });

        thread1.start();
        thread2.start();

        // 在主线程获取 ThreadLocal 变量
        System.out.println("Main Thread: " + threadLocal.get());
    }
}
//输出
Thread 1: 100
Thread 2: null
Main Thread: 42
```
## 原理
一个ThreadLocal内部维护了一个ThreadLocalMap，类似于HashMap，当在某个线程中调用其set方法时，会将该线程id作为键，传入参数作为值存入其中；
因为map的键必须唯一，所以每个线程只有一个存储的空间，如果需要存储多个变量，可以采用如下办法：
1. 创建一个包含多个变量的类，然后将这个类的实例存储在一个ThreadLocal中
2. 使用多个ThreadLocal，每个ThreadLocal保存不同的值
## 使用ThreadLocal注意事项
使用`ThreadLocal`时，需要注意一些问题以确保正确的使用和避免潜在的错误。下面是一些建议和注意事项：
1. **ThreadLocal的生命周期：** **ThreadLocal存储的数据与线程的生命周期相关联**。如果线程池中的线程被重用，可能会导致上一次的`ThreadLocal`数据残留。确保**在线程池中使用**`ThreadLocal`时，在任务执行结束后及时清理相关的`ThreadLocal`数据。
2. **手动清理ThreadLocal：** 在使用完`ThreadLocal`后，尽量手动调用`remove`方法清理`ThreadLocal`中的数据。可以使用`try-finally`块确保在任何情况下都能正确清理。
```java
    try {  使用ThreadLocal }
    finally {   threadLocalVariable.remove(); }
```
3. **避免内存泄漏：** 由于`ThreadLocal`的生命周期与线程相绑定，如果在应用中存在很多线程，并且频繁使用`ThreadLocal`，可能导致内存泄漏。确保使用完`ThreadLocal`后及时清理，以免过多的`ThreadLocal`实例占用内存。
4. **适当使用初始化值：** 可以通过`ThreadLocal`的`withInitial`方法在获取线程本地变量时设置默认值。这有助于避免在未设置值时返回`null`等不可预测的情况。
```java
private static ThreadLocal<String> threadLocalVariable = 
	ThreadLocal.withInitial(() -> "Default");
```
5. **不要滥用ThreadLocal：** `ThreadLocal`是一种方便的工具，但滥用可能导致代码不易理解，尤其是在大规模的并发环境中。在适当的情况下使用，不要过度依赖它。
6. **清理资源：** 如果`ThreadLocal`中存储的是一些需要手动释放的资源，确保在不再需要这些资源时进行释放，以防资源泄漏。
7. **考虑使用InheritableThreadLocal：** 如果线程池中的任务可能会创建子任务，并且需要共享父任务的`ThreadLocal`数据，可以考虑使用`InheritableThreadLocal`。它会使子线程继承父线程的`ThreadLocal`值。
## 阿里开源TTL
1. TTL（Transmittable Thread-Local）是阿里巴巴开源的一个Java工具库，用于解决跨线程传递`ThreadLocal`值的问题。在Java多线程编程中，`ThreadLocal`通常用于保存线程私有的变量，但当线程池或异步任务池中的线程复用时，`ThreadLocal`的值无法自动传递，可能导致问题。
2. TTL提供了一种解决方案，通过TTL可以在父线程中设置`ThreadLocal`的值，然后在子线程中获取到这个值。这样，即使线程被线程池复用，`ThreadLocal`的值也能够正确地传递。
3. 以下是TTL的一些特点和使用方法：
	-  **支持父子线程传递：** TTL通过修改`ThreadLocal`的底层实现，支持在父线程中设置`ThreadLocal`的值，然后在子线程中获取到这个值。
	-  **透明：** 使用TTL时，对原有的`ThreadLocal`代码基本没有入侵，通过TTL提供的API可以在需要的地方进行修饰。
	-  **无内存泄漏风险：** TTL会在子线程中自动清理不再需要的`ThreadLocal`值，避免了传统`ThreadLocal`可能存在的内存泄漏问题。
	-  **与线程池集成：** TTL对一些常见的线程池（如ThreadPoolExecutor）进行了集成，可以方便地与线程池搭配使用。
	-  **兼容性：** TTL兼容Java原生的`ThreadLocal` API，使用起来相对简便。