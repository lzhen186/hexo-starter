---
title: SpringBoot 10.异步任务与定时任务 10.2.为异步任务规划线程池
date: 2022-07-03T08:25:21.904Z
tags: [springboot]
---
# 一、线程池的作用

1. 防止资源占用无限的扩张
2. 调用过程省去资源的创建和销毁所占用的时间

在上一节中，我们的一个异步任务打开了一个线程，完成后销毁。在高并发环境下，不断的分配新资源，可能导致系统资源耗尽。所以为了避免这个问题，我们为异步任务规划一个线程池。

# 二、定义线程池

在上述操作中，创建一个 **线程池配置类**`TaskConfiguration` ，并配置一个 **任务线程池对象**`taskExecutor`。

```java
@Configuration
public class TaskConfiguration {
    @Bean("taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setKeepAliveSeconds(60);
        executor.setThreadNamePrefix("taskExecutor-");
        executor.setRejectedExecutionHandler(new CallerRunsPolicy());
        return executor;
    }
}
```

上面我们通过使用 `ThreadPoolTaskExecutor` 创建了一个 **线程池**，同时设置了以下这些参数：

| 线程池属性                 | 属性的作用                                                   | 设置初始值       |
| :------------------------- | :----------------------------------------------------------- | :--------------- |
| 核心线程数                 | 线程池创建时候初始化的线程数                                 | 10               |
| 最大线程数                 | 线程池最大的线程数，只有在缓冲队列满了之后，才会申请超过核心线程数的线程 | 20               |
| 缓冲队列                   | 用来缓冲执行任务的队列                                       | 200              |
| 允许线程的空闲时间         | 当超过了核心线程之外的线程，在空闲时间到达之后会被销毁       | 60秒             |
| 线程池名的前缀             | 可以用于定位处理任务所在的线程池                             | taskExecutor-    |
| 线程池对拒绝任务的处理策略 | 这里采用CallerRunsPolicy策略，当线程池没有处理能力的时候，该策略会直接在execute方法的调用线程中运行被拒绝的任务；如果执行程序已关闭，则会丢弃该任务 | CallerRunsPolicy |

- 创建 `AsyncExecutorTask`类，三个任务的配置和 `AsyncTask` 一样，不同的是 `@Async` 注解需要指定前面配置的 **线程池的名称**`taskExecutor`。

```java
@Component
public class AsyncExecutorTask extends AbstractTask {
    @Async("taskExecutor")
    public Future<String> doTaskOneCallback() throws Exception {
        super.doTaskOne();
        System.out.println("任务一，当前线程：" + Thread.currentThread().getName());
        return new AsyncResult<>("任务一完成");
    }

    @Async("taskExecutor")
    public Future<String> doTaskTwoCallback() throws Exception {
        super.doTaskTwo();
        System.out.println("任务二，当前线程：" + Thread.currentThread().getName());
        return new AsyncResult<>("任务二完成");
    }

    @Async("taskExecutor")
    public Future<String> doTaskThreeCallback() throws Exception {
        super.doTaskThree();
        System.out.println("任务三，当前线程：" + Thread.currentThread().getName());
        return new AsyncResult<>("任务三完成");
    }
}
```

- 在 **单元测试** 用例中，注入 `AsyncExecutorTask` 对象，并在测试用例中执行 `doTaskOne()`，`doTaskTwo()`，`doTaskThree()` 三个方法。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class AsyncExecutorTaskTest {
    @Autowired
    private AsyncExecutorTask task;

    @Test
    public void testAsyncExecutorTask() throws Exception {
        task.doTaskOneCallback();
        task.doTaskTwoCallback();
        task.doTaskThreeCallback();

        sleep(30 * 1000L);
    }
}
```

执行一下上述的 **单元测试**，可以看到如下结果：

```
开始做任务一
开始做任务三
开始做任务二
完成任务二，耗时：3905毫秒
任务二，当前线程：taskExecutor-2
完成任务一，耗时：6184毫秒
任务一，当前线程：taskExecutor-1
完成任务三，耗时：9737毫秒
任务三，当前线程：taskExecutor-3
```

执行上面的单元测试，观察到 **任务线程池** 的 **线程池名的前缀** 被打印，说明 **线程池** 成功执行 **异步任务**！

# 三、优雅地关闭线程池

> 由于在应用关闭的时候异步任务还在执行，导致类似 **数据库连接池** 这样的对象一并被 **销毁了**，当 **异步任务** 中对 **数据库** 进行操作就会出错。

解决方案如下，重新设置线程池配置对象，新增线程池 `setWaitForTasksToCompleteOnShutdown()` 和 `setAwaitTerminationSeconds()` 配置：

```java
@Bean("taskExecutor")
public Executor taskExecutor() {
    ThreadPoolTaskScheduler executor = new ThreadPoolTaskScheduler();
    executor.setPoolSize(20);
    executor.setThreadNamePrefix("taskExecutor-");
    executor.setWaitForTasksToCompleteOnShutdown(true);
    executor.setAwaitTerminationSeconds(60);
    return executor;
}
```

- **setWaitForTasksToCompleteOnShutdown(true):** 该方法用来设置 **线程池关闭** 的时候 **等待** 所有任务都完成后，再继续 **销毁** 其他的 `Bean`，这样这些 **异步任务** 的 **销毁** 就会先于 **数据库连接池对象** 的销毁。
- **setAwaitTerminationSeconds(60):** 该方法用来设置线程池中 **任务的等待时间**，如果超过这个时间还没有销毁就 **强制销毁**，以确保应用最后能够被关闭，而不是阻塞住。