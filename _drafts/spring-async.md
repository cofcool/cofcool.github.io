# Spring Async

org.springframework.scheduling.annotation.AsyncResult
org.springframework.aop.interceptor.AsyncExecutionAspectSupport
org.springframework.aop.interceptor.AsyncExecutionInterceptor

## 定时任务

org.springframework.scheduling.config.ScheduledTaskRegistrar

各个类型的任务都继承 org.springframework.scheduling.config.Task

CronTask 依靠 org.springframework.scheduling.concurrent.ReschedulingRunnable 实现定时执行，最终调用 java.util.concurrent.ScheduledExecutorService#schedule(java.lang.Runnable, long, java.util.concurrent.TimeUnit)。执行本次任务要结束时 org.springframework.scheduling.Trigger 计算出到下次任务的时间间隔