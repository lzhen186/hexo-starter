---
title: SpringBoot 10.异步任务与定时任务 10.1.实现Async异步任务
date: 2022-07-03T08:18:34.354Z
tags: [springboot]
---
# 1. 环境准备

在 `Spring Boot` 入口类上配置 `@EnableAsync` 注解开启异步处理。
创建任务抽象类 `AbstractTask`，并分别配置三个任务方法 `doTaskOne()`，`doTaskTwo()`，`doTaskThree()`。

```java
public abstract class AbstractTask {
    private static Random random = new Random();

    public void doTaskOne() throws Exception {
        System.out.println("开始做任务一");
        long start = currentTimeMillis();
        sleep(random.nextInt(10000));
        long end = currentTimeMillis();
        System.out.println("完成任务一，耗时：" + (end - start) + "毫秒");
    }

    public void doTaskTwo() throws Exception {
        System.out.println("开始做任务二");
        long start = currentTimeMillis();
        sleep(random.nextInt(10000));
        long end = currentTimeMillis();
        System.out.println("完成任务二，耗时：" + (end - start) + "毫秒");
    }

    public void doTaskThree() throws Exception {
        System.out.println("开始做任务三");
        long start = currentTimeMillis();
        sleep(random.nextInt(10000));
        long end = currentTimeMillis();
        System.out.println("完成任务三，耗时：" + (end - start) + "毫秒");
    }
}
```

# 2. 同步调用

下面通过一个简单示例来直观的理解什么是同步调用：

- 定义 `Task` 类，继承 `AbstractTask`，三个处理函数分别模拟三个执行任务的操作，操作消耗时间随机取（`10` 秒内）。

```java
@Component
public class SyncTask extends AbstractTask {
}
```

- 在 **单元测试** 用例中，注入 `SyncTask` 对象，并在测试用例中执行 `doTaskOne()`，`doTaskTwo()`，`doTaskThree()` 三个方法。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TaskTest {
    @Autowired
    private SyncTask task;

    @Test
    public void testSyncTasks() throws Exception {
        task.doTaskOne();
        task.doTaskTwo();
        task.doTaskThree();
    }
}
```

- 执行单元测试，可以看到类似如下输出：

```
开始做任务一
完成任务一，耗时：6720毫秒
开始做任务二
完成任务二，耗时：6604毫秒
开始做任务三
完成任务三，耗时：9448毫秒
```

任务一、任务二、任务三顺序的执行完了，换言之 `doTaskOne()`，`doTaskTwo()`，`doTaskThree()` 三个方法顺序的执行完成。

# 3. 异步调用

上述的 **同步调用** 虽然顺利的执行完了三个任务，但是可以看到 **执行时间比较长**，若这三个任务本身之间 **不存在依赖关系**，可以 **并发执行** 的话，同步调用在 **执行效率** 方面就比较差，可以考虑通过 **异步调用** 的方式来 **并发执行**。

- 创建 `AsyncTask`类，分别在方法上配置 `@Async` 注解，将原来的 **同步方法** 变为 **异步方法**。

```java
@Component
public class AsyncTask extends AbstractTask {
    @Async
    public void doTaskOne() throws Exception {
        super.doTaskOne();
    }

    @Async
    public void doTaskTwo() throws Exception {
        super.doTaskTwo();
    }

    @Async
    public void doTaskThree() throws Exception {
        super.doTaskThree();
    }
}
```

- 在 **单元测试** 用例中，注入 `AsyncTask` 对象，并在测试用例中执行 `doTaskOne()`，`doTaskTwo()`，`doTaskThree()` 三个方法。

```java
@Autowired
private AsyncTask asyncTask;

@Test
public void testAsyncTasks() throws Exception {
    asyncTask.doTaskOne();
    asyncTask.doTaskTwo();
    asyncTask.doTaskThree();
}
```

- 执行单元测试，可以看到类似如下输出：

```
开始做任务三
开始做任务一
开始做任务二
```

如果反复执行单元测试，可能会遇到各种不同的结果，比如：

1. 没有任何任务相关的输出
2. 有部分任务相关的输出
3. 乱序的任务相关的输出

原因是目前 `doTaskOne()`，`doTaskTwo()`，`doTaskThree()` 这三个方法已经 **异步执行** 了。主程序在 **异步调用** 之后，主程序并不会理会这三个函数是否执行完成了，由于没有其他需要执行的内容，所以程序就 **自动结束** 了，导致了 **不完整** 或是 **没有输出任务** 相关内容的情况。

> 注意：@Async所修饰的函数不要定义为static类型，这样异步调用不会生效。

# 4. 异步回调

为了让 `doTaskOne()`，`doTaskTwo()`，`doTaskThree()` 能正常结束，假设我们需要统计一下三个任务 **并发执行** 共耗时多少，这就需要等到上述三个函数都完成动用之后记录时间，并计算结果。

那么我们如何判断上述三个 **异步调用** 是否已经执行完成呢？我们需要使用 `Future` 来返回 **异步调用** 的 **结果**。

- 创建 `AsyncCallBackTask` 类，声明 `doTaskOneCallback()`，`doTaskTwoCallback()`，`doTaskThreeCallback()` 三个方法，对原有的三个方法进行包装。

```java
@Component
public class AsyncCallBackTask extends AbstractTask {
    @Async
    public Future<String> doTaskOneCallback() throws Exception {
        super.doTaskOne();
        return new AsyncResult<>("任务一完成");
    }

    @Async
    public Future<String> doTaskTwoCallback() throws Exception {
        super.doTaskTwo();
        return new AsyncResult<>("任务二完成");
    }

    @Async
    public Future<String> doTaskThreeCallback() throws Exception {
        super.doTaskThree();
        return new AsyncResult<>("任务三完成");
    }
}
```

- 在 **单元测试** 用例中，注入 `AsyncCallBackTask` 对象，并在测试用例中执行 `doTaskOneCallback()`，`doTaskTwoCallback()`，`doTaskThreeCallback()` 三个方法。循环调用 `Future` 的 `isDone()` 方法等待三个 **并发任务** 执行完成，记录最终执行时间。

```java
@Autowired
private AsyncCallBackTask asyncCallBackTask;

@Test
public void testAsyncCallbackTask() throws Exception {
    long start = currentTimeMillis();
    Future<String> task1 = asyncCallBackTask.doTaskOneCallback();
    Future<String> task2 = asyncCallBackTask.doTaskTwoCallback();
    Future<String> task3 = asyncCallBackTask.doTaskThreeCallback();

    // 三个任务都调用完成，退出循环等待
    while (!task1.isDone() || !task2.isDone() || !task3.isDone()) {
        sleep(1000);
    }

    long end = currentTimeMillis();
    System.out.println("任务全部完成，总耗时：" + (end - start) + "毫秒");
}
```

看看都做了哪些改变：

- 在测试用例一开始记录开始时间；
- 在调用三个异步函数的时候，返回Future类型的结果对象；
- 在调用完三个异步函数之后，开启一个循环，根据返回的Future对象来判断三个异步函数是否都结束了。若都结束，就结束循环；若没有都结束，就等1秒后再判断。
- 跳出循环之后，根据结束时间 - 开始时间，计算出三个任务并发执行的总耗时。

执行一下上述的单元测试，可以看到如下结果：

```
开始做任务三
开始做任务一
开始做任务二
完成任务二，耗时：2572毫秒
完成任务一，耗时：7333毫秒
完成任务三，耗时：7647毫秒
任务全部完成，总耗时：8013毫秒
```

可以看到，通过 **异步调用**，让任务一、任务二、任务三 **并发执行**，有效的 **减少** 了程序的 **运行总时间**。