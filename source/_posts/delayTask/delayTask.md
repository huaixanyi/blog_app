---
title: 延时任务task
comments: true
aside: true
top_img: false
date: 2022-01-17 10:09:24
tags:
description: 延时任务task
mathjax:
katex:
categories:
cover: false
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
                .withIdentity("search", "order").build();

        // 创建触发器 每3秒钟执行一次
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("orderTrigger", "order")
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
        System.out.println("查询订单数据库。。。");
        System.out.println("处理订单(删除，通知)。。。");
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
> 该方案是利用JDK自带的DelayQueue来实现，这是一个无界阻塞队列，该队列只有在延迟期满的时候才能从中获取元素，放入DelayQueue中的对象，必须实现Delayed接口。
> 基本原理：DelayQueue是一个没有边界BlockingQueue实现，加入其中的元素必需实现Delayed接口。当生产者线程调用put之类的方法加入元素时，会触发Delayed接口中的compareTo方法进行排序，也就是说队列中元素的顺序是按照到期时间排序的，而非它们进入队列的顺序。排在队列头部的元素是最早到期的，越往后到期时间越晚。消费者线程查看队列头部的元素，注意是查看不是取出。然后调用元素的getDelay方法，如果此方法返回的值小０或者等于０，则消费者线程会从队列中取出此元素，并进行处理。如果getDelay方法返回的值大于0，则消费者线程wait返回的时间值后，再从队列头部取出元素，此时元素应该已经到期。消费者线程的数量要够，处理任务的速度要快。否则，队列中的到期元素无法被及时取出并处理，造成任务延期、队列元素堆积等情况。

* poll():获取并移除队列的超时元素，没有则返回空
* take():获取并移除队列的超时元素，如果没有则wait当前线程，直到有元素满足超时条件，返回结果。

> 代码实现

* 方案一

```java
@Data
public class OrderDelay implements Delayed {
    private String orderId;
    private long timeout;

    OrderDelay(String orderId, long timeout) {
        this.orderId = orderId;

        this.timeout = timeout + System.nanoTime();
    }

    public int compareTo(Delayed other) {
        if (other == this) {

            return 0;
        }

        OrderDelay t = (OrderDelay) other;

        long d = (getDelay(TimeUnit.NANOSECONDS) -
                t.getDelay(TimeUnit.NANOSECONDS));

        return (d == 0) ? 0 : ((d < 0) ? (-1) : 1);
    }

    // 返回距离你自定义的超时时间还有多少
    public long getDelay(TimeUnit unit) {
        return unit.convert(timeout - System.nanoTime(), TimeUnit.NANOSECONDS);
    }

    void print() {
        System.out.println(orderId + "编号的订单要删除啦。。。。");
    }
}
@Configuration
public class JobBean {

    public volatile static DelayQueue<OrderDelay> queue = new DelayQueue< OrderDelay >();
    @Bean
    public ExecutorService executorJob(){
        // TODO 从数据库/redis初始化订单信息
//        initOrderQueue();
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        executorService.execute(()->{
            log.info("开启自动取消订单job,当前时间 = {}", System.currentTimeMillis());
            while (true) {
                System.out.println("time:" + System.currentTimeMillis());
                try {
                    // 获取指定订单信息
                    OrderDelay order = queue.take();
                    order.print();
                    // 从队列中删除该数据
//                    queue.remove(order);
                    log.info("订单" + order.getOrderId() + "超时取消，取消时间 = {}", System.currentTimeMillis());
                    log.info("Initial Size = {}", queue.size());
                } catch (InterruptedException e) {
                    System.out.println("time:break:" + System.currentTimeMillis());
                    break;
                }
            }
        });
        return executorService;
    }
}
```

* 方案二

```java
@Slf4j
@Component
public class OrderDelayQueueJob implements CommandLineRunner {
    public volatile static DelayQueue < OrderDelay > queue = new DelayQueue< OrderDelay >();
    public void orderTask() {
        log.info("开启自动取消订单job,当前时间 = {}", System.currentTimeMillis());
        while (true) {
            System.out.println("time:" + System.currentTimeMillis());
            try {
                // 获取指定订单信息
                OrderDelay order = queue.take();
                // 从队列中删除该数据
                order.print();
//                queue.remove(order);
                log.info("订单" + order.getOrderId() + "超时取消，取消时间 = {}", System.currentTimeMillis());
                log.info("Initial Size = {}", queue.size());
            } catch (InterruptedException e) {
                System.out.println("time:break:" + System.currentTimeMillis());
                break;
            }
        }
    }

    @Override
    public void run(String... args) {
        // 自动取消订单开启
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(this::orderTask);
    }
}
```

> 优缺点

* 优点:
效率高,任务触发时间延迟低。

* 缺点:
服务器重启后，数据全部消失，怕宕机
集群扩展相当麻烦
因为内存条件限制的原因，比如下单未付款的订单数太多，那么很容易就出现OOM异常
代码复杂度较高

#### 3.redis

> 利用redis的zset,zset是一个有序集合，每一个元素(member)都关联了一个score,通过score排序来取集合中的值

* 添加元素:ZADD key score member [[score member] [score member] …]
* 按顺序查询元素:ZRANGE key start stop [WITHSCORES]
* 查询元素：score:ZSCORE key member
* 移除元素:ZREM key member [member …]



# Have fun ^_^
---