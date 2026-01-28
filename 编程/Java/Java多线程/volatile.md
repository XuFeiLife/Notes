[核心特性](#核心特性)
[为什么需要volatile](#为什么需要volatile)
[volatile vs synchronized](#volatile%20vs%20synchronized)
[常见应用场景](#常见应用场景)

`volatile` 是一个非常重要但常被误解的关键字。它主要用于多线程环境下，解决变量在多个线程之间的**可见性**和**有序性**问题。

# 核心特性

## 可见性

在多线程环境下，每个线程都有自己的本地缓存（Working Memory）。当一个线程修改了普通变量时，其他线程可能无法立即看到这个修改。

使用 `volatile` 修饰变量后：

- **立即刷回主存**：当一个线程修改了变量，它会立即被写回主内存。
- **失效通知**：其他线程在读取该变量时，会强制从主内存读取最新值，而不是从自己的缓存中读

## 有序性

编译器和处理器为了提高性能，往往会对指令进行**重排序**。`volatile` 通过加入“内存屏障”（Memory Barrier）来禁止指令重排序，确保代码执行顺序符合逻辑预期。

# 为什么需要volatile

代码场景
``` java
boolean stop = false;

// 线程 A：执行任务
while (!stop) {
    // 做一些事...
}

// 线程 B：停止任务
stop = true;
```
**问题：** 即使线程 B 将 `stop` 改为 `true`，线程 A 仍可能因为缓存了 `stop = false` 而陷入死循环。 **解决：** 给 `stop` 加上 `volatile` 关键字，线程 A 就能立刻感知到 `stop` 的变化。

# volatile vs synchronized

[synchronized](synchronized.md)

| **特性**   | **volatile**           | **synchronized** |
| -------- | ---------------------- | ---------------- |
| **开销**   | 极低（轻量级）                | 较高（重量级，涉及锁竞争）    |
| **可见性**  | 保证                     | 保证               |
| **原子性**  | **不保证** (例如 `i++` 不安全) | 保证               |
| **使用场景** | 状态标记、双重检查锁定            | 复杂的临界区操作         |
**注意：** `volatile` 不能保证原子性。如果你执行 `count++`，这实际上包含“读-改-写”三个步骤，`volatile` 无法阻止多个线程在中间阶段互相覆盖，这种情况下必须使用 `synchronized` 或 `AtomicInteger`。

# 常见应用场景
- **状态标志位**：如上面提到的 `stop` 标志，用于优雅地关闭线程。
- **单例模式（Double-Check Locking）**：防止对象初始化时的指令重排导致拿到一个未初始化的对象。
```java
public class Singleton {
    private static volatile Singleton instance; // 必须加 volatile

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
