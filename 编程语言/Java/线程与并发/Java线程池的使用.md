# Java 线程池的使用

创建线程池的 API：

```java
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler)
```

这几个核心参数的作用：

- `corePoolSize` 为线程池的基本大小。
- `maximumPoolSize` 为线程池最大线程大小。
- `keepAliveTime` 和 `unit` 则是线程空闲后的存活时间。
- `workQueue` 用于存放任务的阻塞队列。
- `handler` 当队列和最大线程池都满了之后的饱和策略。

了解了这几个参数再来看看实际的运用。







































