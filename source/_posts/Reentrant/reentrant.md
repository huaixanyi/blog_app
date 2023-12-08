---
title: ReentrantLock可重入锁tryLock和lock使用场景
comments: true
aside: true
top_img: false
date: 2022-01-18 10:09:24
tags:
description: ReentrantLock可重入锁tryLock和lock使用场景
mathjax:
katex:
categories:
top: true
cover: http://42.192.149.99:9999/down/YK6LeoksZfw5?fname=/360wallpaper.jpg
video: true
---
### 背景
* 生成订单30分钟未支付，则自动取消
* 生成订单60秒后,给用户发短信

### 方案
#### 1.数据库轮训
> 该方案通常是在`小型项目中使用`，即通过一个线程定时的去扫描数据库，通过订单时间来判断是否有超时的订单，然后进行update或delete等操作
> 类似于定时任务

* 引入maven依赖
quartz：英[kwɔːts] 美[kwɔːrts]
```xml
<dependency>
  <groupId>org.quartz-scheduler</groupId>
  <artifactId>quartz</artifactId>
  <version>2.2.2</version>
</dependency>
```

* 代码实现
```java
@Configuration
public class JobBean {

    @Bean(initMethod = "start")
    public Scheduler schedulerJob() throws Exception{
        // 创建任务
        JobDetail jobDetail = JobBuilder.newJob(StockJobService.class)
                .withIdentity("job1", "group1").build();

        // 创建触发器 每3秒钟执行一次
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("trigger1", "group3")
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                        .withIntervalInSeconds(3)
                        .repeatForever())
                .build();

        Scheduler scheduler = new StdSchedulerFactory().getScheduler();

        // 将任务及其触发器放入调度器
        scheduler.scheduleJob(jobDetail, trigger);

        // 调度器开始调度任务
        return scheduler;
    }

}
public class StockJobService implements Job {

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println("要去数据库扫描啦。。。");
    }
}
```

* 优缺点
 优点: 简单易行，支持集群操作
 缺点:
 对服务器内存消耗大
 存在延迟，比如你每隔3分钟扫描一次，那最坏的延迟时间就是3分钟
 假设你的订单有几千万条，每隔几分钟这样扫描一次，数据库损耗极大

#### 2.JDK的延迟队列
> 该方案是利用JDK自带的DelayQueue来实现，这是一个无界阻塞队列，该队列只有在延迟期满的时候才能从中获取元素，放入DelayQueue中的对象，是必须实现Delayed接口的。
> 基本原理：DelayQueue是一个没有边界BlockingQueue实现，加入其中的元素必需实现Delayed接口。当生产者线程调用put之类的方法加入元素时，会触发Delayed接口中的compareTo方法进行排序，也就是说队列中元素的顺序是按照到期时间排序的，而非它们进入队列的顺序。排在队列头部的元素是最早到期的，越往后到期时间越晚。消费者线程查看队列头部的元素，注意是查看不是取出。然后调用元素的getDelay方法，如果此方法返回的值小０或者等于０，则消费者线程会从队列中取出此元素，并进行处理。如果getDelay方法返回的值大于0，则消费者线程wait返回的时间值后，再从队列头部取出元素，此时元素应该已经到期。消费者线程的数量要够，处理任务的速度要快。否则，队列中的到期元素无法被及时取出并处理，造成任务延期、队列元素堆积等情况。

* Poll():获取并移除队列的超时元素，没有则返回空
* take():获取并移除队列的超时元素，如果没有则wait当前线程，直到有元素满足超时条件，返回结果。

> 代码实现
```java
@Configuration
public class JobBean {

    @Bean(initMethod = "start")
    public Scheduler schedulerJob() throws Exception{
        // 创建任务
        JobDetail jobDetail = JobBuilder.newJob(StockJobService.class)
                .withIdentity("job1", "group1").build();

        // 创建触发器 每3秒钟执行一次
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("trigger1", "group3")
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                        .withIntervalInSeconds(3)
                        .repeatForever())
                .build();

        Scheduler scheduler = new StdSchedulerFactory().getScheduler();

        // 将任务及其触发器放入调度器
        scheduler.scheduleJob(jobDetail, trigger);

        // 调度器开始调度任务
        return scheduler;
    }

}
public class StockJobService implements Job {

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println("要去数据库扫描啦。。。");
    }
}
```


# Have fun ^_^
---