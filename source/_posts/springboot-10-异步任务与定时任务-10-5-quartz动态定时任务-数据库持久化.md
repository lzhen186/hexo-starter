---
title: SpringBoot 10.异步任务与定时任务 10.5.quartz动态定时任务(数据库持久化)
date: 2022-07-03T08:43:14.346Z
tags: [springboot]
---
# 一、前言

在项目开发过程当中，某些定时任务，可能在运行一段时间之后，就不需要了，或者需要修改下定时任务的执行时间等等。
需要在代码当中进行修改然后重新打包发布，很麻烦。使用Quartz来实现的话不需要重新修改代码而达到要求。

# 二、原理

1. 使用quartz提供的API完成配置任务的增删改查
2. 将任务的配置保存在数据库中

# 三、配置

application.yml
在上一节中已经引入了maven依赖包,这里不再重复。直接spring属性下面加入quartz配置信息

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
  quartz:
    job-store-type: JDBC #数据库存储quartz任务配置
    jdbc:
      initialize-schema: NEVER #自动初始化表结构，第一次启动的时候这里写always
```

> 但可能是版本bug，有的时候自动建表不会生效，自己去quartz-scheduler-x.x.x.jar里面找一下建表sql脚本：classpath:org/quartz/impl/jdbcjobstore/tables_@@platform@@.sql，然后执行。

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200428132333.png)

# 四、动态配置代码实现

第一步 创建一个定时任务相关实体类用于保存定时任务相关信息到数据库当中

```java
@Data
public class QuartzBean {
    /** 任务id */
    private String id;

    /** 任务名称 */
    private String jobName;

    /** 任务执行类 */
    private String jobClass;

    /** 任务状态 启动还是暂停*/
    private Integer status;

    /** 任务运行时间表达式 */
    private String cronExpression;
}
```

第二步 创建定时任务暂停，修改，启动，单次启动工具类

```java
public class QuartzUtils {

    /**
     * 创建定时任务 定时任务创建之后默认启动状态
     * @param scheduler 调度器
     * @param quartzBean 定时任务信息类
     */
    @SuppressWarnings("unchecked")
    public static void createScheduleJob(Scheduler scheduler, QuartzBean quartzBean) throws ClassNotFoundException, SchedulerException {
            //获取到定时任务的执行类 必须是类的绝对路径名称
            //定时任务类需要是job类的具体实现 QuartzJobBean是job的抽象类。
            Class<? extends Job> jobClass = (Class<? extends Job>) Class.forName(quartzBean.getJobClass());
            // 构建定时任务信息
            JobDetail jobDetail = JobBuilder.newJob(jobClass).withIdentity(quartzBean.getJobName()).build();
            // 设置定时任务执行方式
            CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(quartzBean.getCronExpression());
            // 构建触发器trigger
            CronTrigger trigger = TriggerBuilder.newTrigger().withIdentity(quartzBean.getJobName()).withSchedule(scheduleBuilder).build();
            scheduler.scheduleJob(jobDetail, trigger);
    }

    /**
     * 根据任务名称暂停定时任务
     * @param scheduler 调度器
     * @param jobName 定时任务名称
     */
    public static void pauseScheduleJob(Scheduler scheduler, String jobName) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName);
        scheduler.pauseJob(jobKey);
    }

    /**
     * 根据任务名称恢复定时任务
     * @param scheduler 调度器
     * @param jobName 定时任务名称
     */
    public static void resumeScheduleJob(Scheduler scheduler, String jobName) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName);
        scheduler.resumeJob(jobKey);
    }

    /**
     * 根据任务名称立即运行一次定时任务
     * @param scheduler 调度器
     * @param jobName 定时任务名称
     */
    public static void runOnce(Scheduler scheduler, String jobName) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName);
        scheduler.triggerJob(jobKey);
    }

    /**
     * 更新定时任务
     * @param scheduler 调度器
     * @param quartzBean 定时任务信息类
     */
    public static void updateScheduleJob(Scheduler scheduler, QuartzBean quartzBean) throws SchedulerException {

            //获取到对应任务的触发器
            TriggerKey triggerKey = TriggerKey.triggerKey(quartzBean.getJobName());
            //设置定时任务执行方式
            CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(quartzBean.getCronExpression());
            //重新构建任务的触发器trigger
            CronTrigger trigger = (CronTrigger) scheduler.getTrigger(triggerKey);
            trigger = trigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(scheduleBuilder).build();
            //重置对应的job
            scheduler.rescheduleJob(triggerKey, trigger);
    }

    /**
     * 根据定时任务名称从调度器当中删除定时任务
     * @param scheduler 调度器
     * @param jobName 定时任务名称
     */
    public static void deleteScheduleJob(Scheduler scheduler, String jobName) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName);
        scheduler.deleteJob(jobKey);
    }
}
@Controller
@RequestMapping("/quartz/job/")
public class QuartzController {
    //注入任务调度
    @Resource
    private Scheduler scheduler;

    @PostMapping("/create")
    @ResponseBody
    public String createJob(@RequestBody QuartzBean quartzBean) throws SchedulerException, ClassNotFoundException {
        QuartzUtils.createScheduleJob(scheduler,quartzBean);
        return "已创建任务";//这里return不是生产级别代码，测试简单写一下
    }

    @PostMapping("/pause")
    @ResponseBody
    public String pauseJob(String jobName) throws SchedulerException {
        QuartzUtils.pauseScheduleJob (scheduler,jobName);
        return "已暂停成功";//这里return不是生产级别代码，测试简单写一下
    }

    @PostMapping("/run")
    @ResponseBody
    public String runOnce(String jobName) throws SchedulerException {
        QuartzUtils.runOnce (scheduler,jobName);
        return "运行任务" + jobName + "成功";//这里return不是生产级别代码，测试简单写一下
    }

    @PostMapping("/resume")
    @ResponseBody
    public String resume(String jobName) throws SchedulerException {
        QuartzUtils.resumeScheduleJob(scheduler,jobName);
        return "恢复定时任务成功：" + jobName;
    }

    @PostMapping("/update")
    @ResponseBody
    public String update(@RequestBody QuartzBean quartzBean) throws SchedulerException {
        QuartzUtils.updateScheduleJob(scheduler,quartzBean);
        return "更新定时任务调度信息成功";
    }
}
```