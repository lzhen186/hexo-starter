---
title: SpringBoot 10.异步任务与定时任务 10.4.quartz简单定时任务(内存持久化)
date: 2022-07-03T08:40:31.660Z
tags: [springboot]
---
# 一、 引入对应的 maven依赖

在 springboot2.0 后官方添加了 Quartz 框架的依赖，所以只需要在 pom 文件当中引入

```xml
      <!--引入quartz定时框架-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-quartz</artifactId>
        </dependency>
```

# 二、 创建一个任务类

由于 springboot2.0 自动进行了依赖所以创建的定时任务类直接继承 QuzrtzJobBean 就可以了，新建一个定时任务类：QuartzSimpleTask

```java
public class QuartzSimpleTask extends QuartzJobBean {
    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println("quartz简单的定时任务执行时间："+new Date().toLocaleString());
    }
}
```

# 三、创建 Quartz 定时配置类

将之前创建的定时任务添加到定时调度里面

```java
@Configuration
public class QuartzSimpleConfig {
    //指定具体的定时任务类
    @Bean
    public JobDetail uploadTaskDetail() {
        return JobBuilder.newJob(QuartzSimpleTask.class).withIdentity("QuartzSimpleTask").storeDurably().build();
    }

    @Bean
    public Trigger uploadTaskTrigger() {
        //这里设定触发执行的方式
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("*/5 * * * * ?");
        // 返回任务触发器
        return TriggerBuilder.newTrigger().forJob(uploadTaskDetail())
                .withIdentity("QuartzSimpleTask")
                .withSchedule(scheduleBuilder)
                .build();
    }
}
```

最后运行项目查看效果