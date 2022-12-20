# @Scheduled 注解与线程池

---

[toc]



## 默认配置

默认情况下，Spring 使用一个线程来处理所有定时任务：

```java
package org.springframework.scheduling.config;

//...
import java.util.concurrent.Executors;
//...

/**
 * Helper bean for registering tasks with a {@link TaskScheduler}, typically using cron
 * expressions.
 */
public class ScheduledTaskRegistrar implements ScheduledTaskHolder, InitializingBean, DisposableBean {
    /**
	 * Calls {@link #scheduleTasks()} at bean construction time.
	 */
	@Override
	public void afterPropertiesSet() {
		scheduleTasks();
	}
    
    /**
	 * Schedule all registered tasks against the underlying
	 * {@linkplain #setTaskScheduler(TaskScheduler) task scheduler}.
	 */
	@SuppressWarnings("deprecation")
	protected void scheduleTasks() {
		if (this.taskScheduler == null) {
			this.localExecutor = Executors.newSingleThreadScheduledExecutor(); // 默认采用单线程处理所有任务
			this.taskScheduler = new ConcurrentTaskScheduler(this.localExecutor);
		}
        // ...
	}
}
```

```java
package java.util.concurrent;

//...

public class Executors {
    /**
     * Creates a single-threaded executor that can schedule commands
     * to run after a given delay, or to execute periodically.
     * (Note however that if this single
     * thread terminates due to a failure during execution prior to
     * shutdown, a new one will take its place if needed to execute
     * subsequent tasks.)  Tasks are guaranteed to execute
     * sequentially, and no more than one task will be active at any
     * given time. Unlike the otherwise equivalent
     * {@code newScheduledThreadPool(1)} the returned executor is
     * guaranteed not to be reconfigurable to use additional threads.
     * @return the newly created scheduled executor
     */
    public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
        return new DelegatedScheduledExecutorService
            (new ScheduledThreadPoolExecutor(1));
    }    
}
```

由于是单个线程，定时任务的实际执行周期可能会与预期的不一样。在最糟糕的情况下，可能会因为某个定时任务的阻塞导致其他定时任务无法执行。



例子：

```java
@Slf4j
@Service
public class MyScheduledService {

    /**
     * Task1：每隔1秒处理一次，处理时长10秒
     */
    @Scheduled(cron = "0/1 * * * * ? ")
    public void task1() {
        log.info("Task1：start");
        aLongTimeTask(10000);
        log.info("Task1：end\n");
    }

    /**
     * Task2：每隔1秒处理一次，处理时长1秒
     */
    @Scheduled(cron = "0/1 * * * * ? ")
    public void task2() {
        log.info("Task2：start");
        aLongTimeTask(1000);
        log.info("Task2：end\n");
    }

    private void aLongTimeTask(long time) {
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

预期的结果是 task2 每隔一秒处理一次，然而，因为是单线程，必须等 task1 处理完才能处理 task2，所以实际上 task2 大约十秒才会执行一次：

```
2022-12-20 19:25:16.014  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task1：start
2022-12-20 19:25:26.025  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task1：end

2022-12-20 19:25:26.025  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task2：start
2022-12-20 19:25:27.037  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task2：end

2022-12-20 19:25:27.037  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task1：start
2022-12-20 19:25:37.052  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task1：end

2022-12-20 19:25:37.052  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task2：start
2022-12-20 19:25:38.065  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task2：end

2022-12-20 19:25:38.065  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task1：start
2022-12-20 19:25:48.074  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task1：end

2022-12-20 19:25:48.074  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task2：start
2022-12-20 19:25:49.080  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task2：end

2022-12-20 19:25:49.080  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task1：start
2022-12-20 19:25:59.083  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task1：end

2022-12-20 19:25:59.083  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task2：start
2022-12-20 19:26:00.090  INFO 21252 --- [   scheduling-1] c.x.s.service.a.MyScheduledService       : Task2：end
```

可以看到，处理定时任务的使用是 “scheduling-1” 这个线程。



## 优化方向

-   **方向1：改变任务自身执行方式**

    从任务自身入手，改变任务的执行方式，例如，异步执行、提交给线程池执行

    

-   **方向2：改变Spring处理定时任务的方式**

    从 Spring 处理定时任务的方式入手，使用自定义的线程池替换默认的线程池



## 实践记录

#### 实践1：使用异步的方式处理任务

```java
package com.xzy.scheduling.service.c;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

/**
 * 说明：
 * <p>1.使用默认的SingleThreadScheduledExecutor处理定时任务</p>
 * <p>2.将定时任务改为异步执行</p>
 *
 * @author xzy
 * @date 2022/12/8  21:28
 */
@Slf4j
@Service
@EnableAsync
@Configuration
public class MyScheduledService  {

    /**
     * Task1：每隔1秒处理一次，处理时长10秒
     */
    @Async
    @Scheduled(cron = "0/1 * * * * ? ")
    public void task1() {
        log.info("Task1：start");
        aLongTimeTask(10000);
        log.info("Task1：end\n");
    }

    /**
     * Task2：每隔1秒处理一次，处理时长1秒
     */
    @Async
    @Scheduled(cron = "0/1 * * * * ? ")
    public void task2() {
        log.info("Task2：start");
        aLongTimeTask(1000);
        log.info("Task2：end\n");
    }

    private void aLongTimeTask(long time) {
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

效果：Task2 每隔一秒处理一次，与预期相符

```
2022-12-20 19:45:26.041  INFO 5688 --- [         task-2] c.x.s.service.c.MyScheduledService       : Task1：start
2022-12-20 19:45:26.041  INFO 5688 --- [         task-1] c.x.s.service.c.MyScheduledService       : Task2：start
2022-12-20 19:45:27.002  INFO 5688 --- [         task-3] c.x.s.service.c.MyScheduledService       : Task1：start
2022-12-20 19:45:27.002  INFO 5688 --- [         task-4] c.x.s.service.c.MyScheduledService       : Task2：start
2022-12-20 19:45:27.049  INFO 5688 --- [         task-1] c.x.s.service.c.MyScheduledService       : Task2：end

2022-12-20 19:45:28.014  INFO 5688 --- [         task-4] c.x.s.service.c.MyScheduledService       : Task2：end

2022-12-20 19:45:28.014  INFO 5688 --- [         task-5] c.x.s.service.c.MyScheduledService       : Task2：start
2022-12-20 19:45:28.015  INFO 5688 --- [         task-6] c.x.s.service.c.MyScheduledService       : Task1：start
2022-12-20 19:45:29.013  INFO 5688 --- [         task-7] c.x.s.service.c.MyScheduledService       : Task1：start
2022-12-20 19:45:29.014  INFO 5688 --- [         task-8] c.x.s.service.c.MyScheduledService       : Task2：start
2022-12-20 19:45:29.028  INFO 5688 --- [         task-5] c.x.s.service.c.MyScheduledService       : Task2：end

2022-12-20 19:45:30.007  INFO 5688 --- [         task-1] c.x.s.service.c.MyScheduledService       : Task2：start
2022-12-20 19:45:30.007  INFO 5688 --- [         task-4] c.x.s.service.c.MyScheduledService       : Task1：start
2022-12-20 19:45:30.023  INFO 5688 --- [         task-8] c.x.s.service.c.MyScheduledService       : Task2：end

2022-12-20 19:45:31.004  INFO 5688 --- [         task-5] c.x.s.service.c.MyScheduledService       : Task2：start
2022-12-20 19:45:31.004  INFO 5688 --- [         task-8] c.x.s.service.c.MyScheduledService       : Task1：start
2022-12-20 19:45:31.019  INFO 5688 --- [         task-1] c.x.s.service.c.MyScheduledService       : Task2：end

2022-12-20 19:45:32.016  INFO 5688 --- [         task-5] c.x.s.service.c.MyScheduledService       : Task2：end

(略)
```

<u>似乎，@Async最终也是将任务交给线程池处理</u>

>   TODO：探究@Async与线程池的关系



#### 实践2：将任务提交给线程池处理

```java
/**
 * 说明：
 * <p>1.使用默认的SingleThreadScheduledExecutor处理定时任务</p>
 * <p>2.将定时任务的执行交给自定义的线程池</p>
 *
 * @author xzy
 * @date 2022/12/8  21:28
 */
@Slf4j
@Service
public class MyScheduledService {

    private final ExecutorService executorService = Executors.newFixedThreadPool(5); // 5个核心线程

    /**
     * Task1：每隔1秒处理一次，处理时长10秒
     */
    @Scheduled(cron = "0/1 * * * * ? ")
    public void task1() {
        log.info("Task1：start");
        executorService.execute(() -> {
            aLongTimeTask(10000);
            log.info("Task1：end\n");
        });
    }

    /**
     * Task2：每隔1秒处理一次，处理时长1秒
     */
    @Scheduled(cron = "0/1 * * * * ? ")
    public void task2() {
        log.info("Task2：start");
        executorService.execute(() -> {
            aLongTimeTask(1000);
            log.info("Task2：end\n");
        });
    }

    private void aLongTimeTask(long time) {
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

效果：Task2 每隔一秒处理一次，与预期相符

```
2022-12-20 19:51:56.005  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task1：start
2022-12-20 19:51:56.005  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task2：start
2022-12-20 19:51:57.003  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task2：start
2022-12-20 19:51:57.003  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task1：start
2022-12-20 19:51:57.019  INFO 8424 --- [pool-1-thread-2] c.x.s.service.d.MyScheduledService       : Task2：end

2022-12-20 19:51:58.008  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task1：start
2022-12-20 19:51:58.008  INFO 8424 --- [pool-1-thread-3] c.x.s.service.d.MyScheduledService       : Task2：end

2022-12-20 19:51:58.008  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task2：start
2022-12-20 19:51:59.001  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task2：start
2022-12-20 19:51:59.001  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task1：start
2022-12-20 19:51:59.017  INFO 8424 --- [pool-1-thread-2] c.x.s.service.d.MyScheduledService       : Task2：end

2022-12-20 19:52:00.014  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task2：start
2022-12-20 19:52:00.014  INFO 8424 --- [pool-1-thread-3] c.x.s.service.d.MyScheduledService       : Task2：end

2022-12-20 19:52:00.014  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task1：start
2022-12-20 19:52:01.015  INFO 8424 --- [pool-1-thread-3] c.x.s.service.d.MyScheduledService       : Task2：end

2022-12-20 19:52:01.015  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task2：start
2022-12-20 19:52:01.015  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task1：start
2022-12-20 19:52:02.003  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task2：start
2022-12-20 19:52:02.003  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task1：start
2022-12-20 19:52:03.011  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task2：start
2022-12-20 19:52:03.011  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task1：start
2022-12-20 19:52:04.006  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task2：start
2022-12-20 19:52:04.006  INFO 8424 --- [   scheduling-1] c.x.s.service.d.MyScheduledService       : Task1：start


（略）
```

可以看到，任务的触发是都是“scheduling-1”线程完成的，但任务的执行是由自定义线程池完成的。



#### 实践3：使用自定义线程池替换默认线程池

```java
package com.xzy.scheduling.service.b;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;
import org.springframework.stereotype.Service;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 说明：使用自定义的线程池处理定时任务
 *
 * @author xzy
 * @date 2022/12/8  21:28
 */
@Slf4j
@Service
@Configuration
public class MyScheduledService implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        // 自定义线程池：4个核心线程
        ExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(4);
        scheduledTaskRegistrar.setScheduler(scheduledExecutorService);
    }

    /**
     * Task1：每隔1秒处理一次，处理时长10秒
     */
    @Scheduled(cron = "0/1 * * * * ? ")
    public void task1() {
        log.info("Task1：start");
        aLongTimeTask(10000);
        log.info("Task1：end\n");
    }

    /**
     * Task2：每隔1秒处理一次，处理时长1秒
     */
    @Scheduled(cron = "0/1 * * * * ? ")
    public void task2() {
        log.info("Task2：start");
        aLongTimeTask(1000);
        log.info("Task2：end\n");
    }

    private void aLongTimeTask(long time) {
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

效果：Task2 每隔一秒处理一次，与预期相符

```
2022-12-20 19:54:15.009  INFO 3052 --- [pool-1-thread-1] c.x.s.service.b.MyScheduledService       : Task1：start
2022-12-20 19:54:15.009  INFO 3052 --- [pool-1-thread-2] c.x.s.service.b.MyScheduledService       : Task2：start
2022-12-20 19:54:16.019  INFO 3052 --- [pool-1-thread-2] c.x.s.service.b.MyScheduledService       : Task2：end

2022-12-20 19:54:17.013  INFO 3052 --- [pool-1-thread-2] c.x.s.service.b.MyScheduledService       : Task2：start
2022-12-20 19:54:18.021  INFO 3052 --- [pool-1-thread-2] c.x.s.service.b.MyScheduledService       : Task2：end

2022-12-20 19:54:19.015  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：start
2022-12-20 19:54:20.019  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：end

2022-12-20 19:54:21.015  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：start
2022-12-20 19:54:22.029  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：end

2022-12-20 19:54:23.004  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：start
2022-12-20 19:54:24.013  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：end

2022-12-20 19:54:25.011  INFO 3052 --- [pool-1-thread-1] c.x.s.service.b.MyScheduledService       : Task1：end

2022-12-20 19:54:25.011  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：start
2022-12-20 19:54:26.009  INFO 3052 --- [pool-1-thread-1] c.x.s.service.b.MyScheduledService       : Task1：start
2022-12-20 19:54:26.025  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：end

2022-12-20 19:54:27.007  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：start
2022-12-20 19:54:28.021  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：end

2022-12-20 19:54:29.003  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：start
2022-12-20 19:54:30.015  INFO 3052 --- [pool-1-thread-3] c.x.s.service.b.MyScheduledService       : Task2：end

（略）
```