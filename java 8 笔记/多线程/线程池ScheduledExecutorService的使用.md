# 线程池ScheduledExecutorService的使用

## 一、线程池介绍

- newSingleThreadExecutor：单线程池，同时只有一个线程在跑
- newCachedThreadPool() ：回收型线程池，可以重复利用之前创建过的线程，运行线程最大数是Integer.MAX_VALUE
- newFixedThreadPool() ：固定大小的线程池，跟回收型线程池类似，只是可以限制同时运行的线程数量
- newScheduleThreadPool()：创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行

## 二、ScheduledExecutorService的用法

### 延时任务

```java
        // 延时任务
        mScheduledExecutorService.schedule(threadFactory.newThread(new Runnable() {
            @Override
            public void run() {
                Log.e("lzp", "first task");
            }
        }), 1, TimeUnit.SECONDS);
```

### 循环任务

### scheduleAtFixedRate()

> 按照上一次任务的发起时间计算下一次任务的开始时间

```java
        mScheduledExecutorService.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                Log.e("lzp", "first:" + System.currentTimeMillis() / 1000);
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, 1, 1, TimeUnit.SECONDS);
```

![image-20230511102112269](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511102112269.png)


从log上看，我们的循环任务严格按照每一秒发起一次，sleep（3000）对于任务的开启是没有影响的，也就是以上一个任务的开始时间 + 延迟时间 = 下一个任务的开始时间

### scheduleWithFixedDelay()

> 以上一次任务的结束时间计算下一次任务的开始时间

```java
        mScheduledExecutorService.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                Log.e("lzp", "scheduleWithFixedDelay:" + System.currentTimeMillis() / 1000);
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, 1, 1, TimeUnit.SECONDS);
```

![image-20230511102129930](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511102129930.png)


从log上看，每一个任务的时间间隔是4秒，而不是我们设置的间隔1秒，任务要耗时3秒，两个时间相加正好是4秒，那么之前代码注释的解释就说的通了：以上一次任务的结束时间 + 延迟时间 = 下一次任务的开始时间