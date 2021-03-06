---
title: SpringBoot 10.异步任务与定时任务 10.3.通过@Scheduled实现定时任务
date: 2022-07-03T08:32:42.727Z
tags: [springboot]
---
# 一、开启定时任务方法

Scheduled定时任务是Spring boot自身提供的功能，所以不需要引入Maven依赖包
在项目入口main方法上加注解

```java
@EnableScheduling //开启定时任务
```

# 二、不同定时方式的解析

### 1.fixedDelay和fixedRate，单位是毫秒，它们的区别就是：

fixedRate就是每隔多长时间执行一次。(开始------->X时间------>再开始)。如果间隔时间小于任务执行时间，上一次任务执行完成下一次任务就立即执行。如果间隔时间大于任务执行时间，就按照每隔X时间运行一次。
而fixedDelay是当任务执行完毕后一段时间再次执行。（开始--->结束(隔一分钟)开始----->结束）。上一次执行任务未完成，下一次任务不会开始。

### 2.cron表达式：灵活

**举例说明**

| 表达式         | 说明                                                        |
| :------------- | :---------------------------------------------------------- |
| 0 0 3 * * ?    | 每天3点执行                                                 |
| 0 5 3 * * ?    | 每天3点5分执行                                              |
| 0 5 3 ? * *    | 每天3点5分执行，与上面作用相同                              |
| 0 5/10 3 * * ? | 每天3点的 5分，15分，25分，35分，45分，55分这几个时间点执行 |
| 0 10 3 ? * 1   | 每周星期天，3点10分 执行，注：1表示星期天                   |
| 0 10 3 ? * 1#3 | 每个月的第三个星期，星期天 执行，#号只能出现在星期的位置    |

- 第一位，表示秒，取值0-59
- 第二位，表示分，取值0-59
- 第三位，表示小时，取值0-23
- 第四位，日期天/日，取值1-31
- 第五位，日期月份，取值1-12
- 第六位，星期，取值1-7，星期一，星期二...，注：不是第1周，第二周的意思，另外：1表示星期天，2表示星期一。
- 第七位，年份，可以留空，取值1970-2099

**cron中，还有一些特殊的符号，含义如下：**
(*)星号：可以理解为每的意思，每秒，每分，每天，每月，每年...
(?)问号：问号只能出现在日期和星期这两个位置。
(-)减号：表达一个范围，如在小时字段中使用“10-12”，则表示从10到12点，即10,11,12
(,)逗号：表达一个列表值，如在星期字段中使用“1,2,4”，则表示星期一，星期二，星期四
(/)斜杠：如：x/y，x是开始值，y是步长，比如在第一位（秒） 0/15就是，从0秒开始，每15秒，最后就是0，15，30，45，60 另：*/y，等同于0/y

cron表达式在线：http://cron.qqe2.com/

# 三、实现定时任务

```java
@Component
public class ScheduledJobs {
  
    //表示方法执行完成后5秒再开始执行
    @Scheduled(fixedDelay=5000)
    public void fixedDelayJob() throws InterruptedException{
        System.out.println("fixedDelay 开始:" + new Date());
        Thread.sleep(10 * 1000);
        System.out.println("fixedDelay 结束:" + new Date());
    }
    
    //表示每隔3秒
    @Scheduled(fixedRate=3000)
    public void fixedRateJob()throws InterruptedException{
        System.out.println("===========fixedRate 开始:" + new Date());
        Thread.sleep(5 * 1000);
        System.out.println("===========fixedRate 结束:" + new Date());
    }

    //表示每隔10秒执行一次
    @Scheduled(cron="0/10 * * * * ? ")
    public void cronJob(){
        System.out.println("=========================== ...>>cron...." + new Date());
    }
}
```

运行结果如下：从运行结果上看，并未按照预期的时间规律运行。仔细看线程打印，竟然所有的定时任务使用的都是一个线程，所以彼此互相影响。

```
===========fixedRate 结束:Tue Jul 09 19:53:04 CST 2019pool-1-thread-1
fixedDelay 开始:Tue Jul 09 19:53:04 CST 2019pool-1-thread-1
fixedDelay 结束:Tue Jul 09 19:53:14 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:14 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:16 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:16 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:18 CST 2019pool-1-thread-1
=========================== ...>>cron....Tue Jul 09 19:53:18 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:18 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:20 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:20 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:22 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:22 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:24 CST 2019pool-1-thread-1
fixedDelay 开始:Tue Jul 09 19:53:24 CST 2019pool-1-thread-1
fixedDelay 结束:Tue Jul 09 19:53:34 CST 2019pool-1-thread-1
=========================== ...>>cron....Tue Jul 09 19:53:34 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:34 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:36 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:36 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:38 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:38 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:40 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:40 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:42 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:42 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:44 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:44 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:46 CST 2019pool-1-thread-1
===========fixedRate 开始:Tue Jul 09 19:53:46 CST 2019pool-1-thread-1
===========fixedRate 结束:Tue Jul 09 19:53:48 CST 2019pool-1-thread-1
fixedDelay 开始:Tue Jul 09 19:53:48 CST 2019pool-1-thread-1
fixedDelay 结束:Tue Jul 09 19:53:58 CST 2019pool-1-thread-1
=========================== ...>>cron....Tue Jul 09 19:53:58 CST 2019pool-1-thread-1
```

# 四、解决定时任务单线程运行的问题

```java
@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {
 
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(scheduledTaskExecutor());
    }
 
    @Bean
    public Executor scheduledTaskExecutor() {
        return Executors.newScheduledThreadPool(3); //指定线程池大小
    }
}
```

再次运行上面的程序，运行时间规律就符合期望了。