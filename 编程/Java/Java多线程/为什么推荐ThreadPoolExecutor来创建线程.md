[为什么不推荐 Executors？](#为什么不推荐%20Executors？)
[最推荐ThreadPoolExecutor](#最推荐ThreadPoolExecutor)

## 为什么不推荐 Executors？

 **1. newFixedThreadPool**
 使用无届队列LinkedBlockingQueue，任务堆积有OOM风险。
 
```java
Executors.newFixedThreadPool(n);

public static ExecutorService newFixedThreadPool(int nThreads) {  
    return new ThreadPoolExecutor(nThreads, nThreads,  
                                  0L, TimeUnit.MILLISECONDS,  
                                  new LinkedBlockingQueue<Runnable>());  
}

```


**2. newCachedThreadPool**
线程数无上限，高并发时CPU/内存容易打爆

```java
Executors.newCachedThreadPool();

public static ExecutorService newCachedThreadPool() {  
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,  
                                  60L, TimeUnit.SECONDS,  
                                  new SynchronousQueue<Runnable>());  
}

```

## 最推荐ThreadPoolExecutor
✔️ 线程数可控  
✔️ 队列可控  
✔️ 拒绝策略可控  
✔️ 可观测、可调优
[线程池七大核心参数](线程池七大核心参数.md)


```java

ExecutorService threadPool = new ThreadPoolExecutor(
        4,                      // corePoolSize
        8,                      // maximumPoolSize
        60L,                    // keepAliveTime
        TimeUnit.SECONDS,
        new ArrayBlockingQueue<>(100), // 有界队列
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.AbortPolicy()
);

public ThreadPoolExecutor(int corePoolSize,  
                          int maximumPoolSize,  
                          long keepAliveTime,  
                          TimeUnit unit,  
                          BlockingQueue<Runnable> workQueue) {  
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,  
         Executors.defaultThreadFactory(), defaultHandler);  
}

```

## SpringBoot中用法

配置+托管 
[[线程数怎么设置？]]

```java
@Bean
public ThreadPoolTaskExecutor taskExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(4);
    executor.setMaxPoolSize(8);
    executor.setQueueCapacity(100);
    executor.setThreadNamePrefix("biz-pool-");
    executor.setRejectedExecutionHandler(new    ThreadPoolExecutor.AbortPolicy());
    executor.initialize();
    return executor;
}

```
使用：
```java
@Async("taskExecutor")
public void doAsync() {
    // ...
}

```