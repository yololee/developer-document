# CountDownLatch的使用

## 一、简介

CountDownLatch是一个同步工具类，用来协调多个线程之间的同步，或者说起到线程之间的通信（而不是用作互斥的作用）

CountDownLatch能够使一个线程在等待另外一些线程完成各自工作之后，再继续执行使用一个计数器进行实现，计数器初始值为线程的数量。当每一个线程完成自己任务后，计数器的值就会减一。当计数器的值为0时，表示所有的线程都已经完成一些任务，然后在CountDownLatch上等待的线程就可以恢复执行接下来的任务

## 二、主要方法

| 方法                                             | 解释                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| public void countDown()                          | 递减锁存器的计数，如果计数到达零，则释放所有等待的线程。如果当前计数大于零，则将计数减少 |
| public boolean await(long timeout,TimeUnit unit) | 使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断或超出了指定的等待时间。如果当前计数为零，则此方法立刻返回true值。 |

## 三、俩种经典用法

> 1.某一线程在开始运行前等待n个线程执行完毕

将 CountDownLatch 的计数器初始化为n ：new CountDownLatch(n)，每当一个任务线程执行完毕，就将计数器减1 countdownlatch.countDown()，当计数器的值变为0时，在CountDownLatch上 await() 的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行

> 2.实现多个线程开始执行任务的最大并行性

注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的 CountDownLatch 对象，将其计数器初始化为 1 ：new CountDownLatch(1)，多个线程在开始执行任务前首先 coundownlatch.await()，当主线程调用 countDown() 时，计数器变为0，多个线程同时被唤醒

## 三、springboot中集成线程池使用countDownLatch

ExecutorConfig.java

```java
@Configuration
@EnableAsync
public class ExecutorConfig {

    @Bean()
    public Executor asyncServiceExecutor() {

        return new ThreadPoolExecutor(10, 20, 0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<>(1024), new ThreadFactoryBuilder().build(), new ThreadPoolExecutor.AbortPolicy());
    }
}
```

CountDownLatchService.java

```java
@Service
public class CountDownLatchService {

    @Autowired
    private ThreadService threadService;

    public void handleData() {
        CountDownLatch countDownLatch = new CountDownLatch(10);
        try {
            for (int i = 0; i < 10; i++) {
                threadService.asyncHandData(i, countDownLatch);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //latch.countDown()放入进finally中，这样即使报错，直接记录跨过，也不会影响主线程的执行
            countDownLatch.countDown();
        }

        try {
            //latch.await();等待所有子线程完成之后，在继续执行
            //CountDownLatch计数器不归0的时候，主线程不执行
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("主线程执行完毕");
    }

}
```

ThreadService.java

```java
@Component
public class ThreadService {

    @Async
    public void asyncHandData(int i, CountDownLatch countDownLatch) {
        try {
            System.out.println(i);
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            //latch.countDown()放入进finally中，这样即使报错，直接记录跨过，也不会影响主线程的执行
            countDownLatch.countDown();
        }
    }
}
```