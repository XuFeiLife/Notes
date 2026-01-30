# 1. 概述

**java.util.concurrent (简称 JUC)** 是从 Java 5 (JDK 1.5) 开始引入的一组专门用于处理多线程编程的工具包。它由并发编程专家 **Doug Lea** 设计，旨在替代传统的 `synchronized` 和 `wait/notify` 机制，提供更高效、更灵活、更易用的并发编程模型。

JUC 的核心设计目标是：**高性能、低延迟、高伸缩性**。

# 2. 核心基石：AQS (AbstractQueuedSynchronizer)

JUC 的大部分组件（如 `ReentrantLock`, `Semaphore`, `CountDownLatch`）都建立在 **AQS** 框架之上。

- **原理**：AQS 维护一个 `volatile` 状态位（state）和一个 FIFO 的双向队列。
- **机制**：通过 **CAS** 操作修改状态位，如果修改失败，则将当前线程封装成节点放入等待队列。

# 3. 主要模块

## 3.1 原子变量类 (Atomic Classes)
位于 `java.util.concurrent.atomic` 包下。

- **特点**：利用底层 CPU 的 **CAS (Compare And Swap)** 指令，实现无锁（Lock-Free）的线程安全。
- **典型类**：`AtomicInteger`, `AtomicReference`, `LongAdder` (适合高竞争场景)。

## 3.2 显示锁(Locks)
位于 `java.util.concurrent.locks` 包下。

- **ReentrantLock**：支持公平锁/非公平锁、可中断获取锁、超时获取锁。
- **ReentrantReadWriteLock**：读写分离锁，大幅提升“读多写少”场景的并发能力。
- **StampedLock**：JDK 8 引入，支持乐观读，进一步提升性能。

## 3.3 并发容器 (Concurrent Collections)
替代传统的同步容器（如 `Hashtable`, `Vector`）。

- **ConcurrentHashMap**：高性能哈希表，JDK 8 采用 Node 数组 + 链表/红黑树 + CAS + synchronized 的细粒度锁。
- **CopyOnWriteArrayList**：写时复制，适用于读多写极少的场景。
- **BlockingQueue**：阻塞队列（如 `ArrayBlockingQueue`, `LinkedBlockingQueue`），是生产者-消费者模型的标准实现。

## 3.4  线程池 (Executor Framework)
JUC 提供了强大的线程管理机制，避免频繁创建/销毁线程的开销。

- **ThreadPoolExecutor**：线程池的核心实现类。
- **ScheduledThreadPoolExecutor**：支持定时及周期性任务执行。

## 3.5 同步协作工具类 (Synchronizers)
用于控制多个线程之间的协作流程：

- **CountDownLatch**：一人等众人（计数减至0时触发）。
- **CyclicBarrier**：众人互相等（所有人都到达屏障点时同时出发）。
- **Semaphore**：限流（控制同时访问特定资源的线程数量）。

# 4. JUC的优势

- **性能优化**：通过 CAS 和自旋锁减少了内核态与用户态切换的开销。
- **功能丰富**：提供了 `synchronized` 无法实现的“尝试获取锁”和“响应中断”。
- **安全性**：内置了大量成熟的并发模式，降低了开发者编写死锁、竞态条件代码的概率。

# 5. 对比

| **维度**   | **传统并发 (Early Java)**          | **JUC 并发**                                   |
| -------- | ------------------------------ | -------------------------------------------- |
| **锁定机制** | 关键字 `synchronized` (内置锁)       | `Lock` 接口 (显式锁)                              |
| **线程协作** | `wait()` / `notify()`          | `Condition` / `CountDownLatch`               |
| **容器**   | `Collections.synchronizedList` | `CopyOnWriteArrayList` / `ConcurrentHashMap` |
| **数值操作** | `volatile` + 锁                 | `AtomicInteger` (CAS)                        |
