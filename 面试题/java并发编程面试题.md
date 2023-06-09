# java并发编程面试题

## 基础知识

### 并发编程的优缺点

```
优点：
    充分利用多核CPU的计算能力
    方便进行业务拆分，提升系统并发能力和性能
缺点：
    内存泄漏、上下文切换、线程安全、死锁    
```

### 什么是线程安全,servlet 是线程安全吗

```
线程安全:如果你的代码在多线程下执行和在单线程下执行永远都能获得一样的结果，那么你的代码就是线程安全的
    
Servlet 不是线程安全的，servlet 是单实例多线程的，当多个线程同时访问同一个方法，是不能保证共享变量的线程安全性的
SpringMVC 的 Controller 是线程安全的吗？不是的，和 Servlet 类似的处理流程,Servlet 和 SpringMVC 需要考虑线程安全问题，但是性能可以提升不用处理太多的 gc，可以使用 ThreadLocal 来处理多线程的问题    
```

### 并发编程三要素是什么？在 Java 程序中怎么保证多线程的运行安全

```
原子性：一个或多个操作要么全部执行成功要么全部执行失败。
可见性：一个线程对共享变量的修改，另一个线程能够立刻看到。
有序性：程序执行的顺序按照代码的先后顺序执行，避免指令重排。

出现线程安全问题的原因：
线程切换带来的原子性问题
缓存导致的可见性问题
编译优化带来的有序性问题
    
解决办法：
JDK Atomic开头的原子类、synchronized、lock，可以解决原子性问题
volatile、synchronized、lock，可以解决可见性问题
volatile、Happens-Before 规则可以解决有序性问题 
```

### 并行和并发的区别

```
串行：多个任务在一个线程上按顺序执行
并发：多个任务在一个 CPU 核上按细分的时间片轮流(交替)执行，从逻辑上来看那些任务是同时执行。
并行：单位时间内，多个 CPU 同时处理多个任务，是真正意义上的“同时进行”。
```

## 线程和进程的区别

### 什么是线程和进程

```
进程:
一个在内存中运行的应用程序。每个进程都有自己独立的一块内存空间，一个进程可以有多个线程，比如在Windows系统中，一个运行的xx.exe就是一个进程。

线程:
进程中的一个执行任务（控制单元），负责当前进程中程序的执行。一个进程至少有一个线程，一个进程可以运行多个线程，多个线程可共享数据
```

### 守护线程和用户线程

```
用户 (User) 线程：运行在前台，执行具体的任务，如程序的主线程、连接网络的子线程等都是用户线程
守护 (Daemon) 线程：运行在后台，为其他前台线程服务
```

### 什么是线程死锁

```
死锁：死锁是指两个或两个以上的进程（线程）在执行过程中，由于竞争资源或者由于彼此通信而造成的
一种阻塞的现象，若无外力作用，它们都将无法推进下去。
```

### 形成死锁的四个必要条件是什么

```
-互斥条件：在一段时间内某资源只由一个进程占用。如果此时还有其它进程请求资源，就只能等
待，直至占有资源的进程用毕释放
    
-占有且等待条件：指进程已经保持至少一个资源，但又提出了新的资源请求，而该资源已被其它进
程占有，此时请求进程阻塞，但又对自己已获得的其它资源保持不放
    
-不可抢占条件：别人已经占有了某项资源，你不能因为自己也需要该资源，就去把别人的资源抢过来 
    
-循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。（比如一个进程集合，A在等B，B在等C，C在等A）    
```

## 创建线程的三种方式

### 创建线程的方式

```
1、继承Thread类
   --写一个Thread类的子类
   --重写run()方法
   --创建Thread类的子类对象
   --调用start()方法,开启线程
2、实现Runnable接口
   --写一个Runnable接口的子类
   --重写run()方法
   --创建Thread对象,把Runnable接口的子类对象作为参数传递
   --调用start()方法,开启线程
3、实现Callable接口
   --写一个Callable接口的子类
   --重写call()方法.有返回值
   --创建FutureTask的对象,将Callable的子类对象作为参数传递
   --创建Thread对象,把FutureTask对象作为参数传递
   --调用start()方法,开启线程
4、使用Executor框架来创建线程池
    
创建线程方式的不同点:
1、继承Thread类:
	由于类的单继承性,继承Thread类后不可以再继承其他类
2、实现Runnable接口:
	扩展性较强,可以继承其他类,同时还可以实现多个接口
3、实现Callable接口:
	线程执行完之后有返回值
4、线程池:
	我们自己频繁地去创建和销毁线程比较消耗系统资源,同时也比较浪费时间.
    当创建一个线程池,其实就是创建了一个能够存储线程的容器,需要执行线程任务时,就从线程池中拿一个线程出来用,用完之后再还给线程池.
```

### runnable 和 callable的区别

```
相同点：
1、都是接口
2、都可以编写多线程程序
3、都采用Thread.start()启动线程
不同点
1、Runnable 接口 run 方法无返回值；Callable 接口 call 方法有返回值，是个泛型，和Future、FutureTask配合可以用来获取异步执行的结果
2、Runnable 接口 run 方法只能抛出运行时异常，且无法捕获处理；Callable 接口 call 方法允许抛出异常，可以获取异常信息
```

### start()方法和run()方法的区别

```
 调用 start 方法方可启动线程并使线程进入就绪状态，而 run 方法只是 thread 的一个普通方法
调用，还是在主线程里执行
```

## 线程的状态和基本操作

### 线程的几种可用状态

```
NEW（新建状态）： 
    ⾄今尚未启动的线程处于这种状态  ，还没有调用start方法
RUNNABLE（就绪状态）：
    正在 Java 虚拟机中执⾏的线程处于这种状态  ，调用了start方法，还没有抢夺CPU的执行权
RUNNING(运行状态)：抢夺到了cpu的执行权，执行run()方法的线程执行体    
BLOCKED（阻塞状态）：由于某种原因放弃cpu的执行权，直到线程可以进入可运行状态再次获取cpu的执行权
	等待阻塞：执行了wait()方法，jvm把线程放入到等待队列中
	同步阻塞：运行的线程获取对象的同步锁时，，该同步锁被其他线程占用
    其他阻塞：调用了sleep
TERMINATED（结束状态）：已退出的线程处于这种状态。
```

### Java 中用到的线程调度算法是什么

```
线程调度是指按照特定机制为多个线程分配 CPU 的使用权。Java 虚拟机的一项任务就是负责线程的调度。

有两种调度模型：分时调度模型和抢占式调度模型。Java虚拟机采用抢占式调度模型

-分时调度模型:让所有的线程轮流获得 cpu 的使用权，并且平均分配每个线程占用的CPU的时间片
    
-抢占式调度模型:Java虚拟机采用抢占式调度模型，是指优先让可运行池中优先级高的线程占用CPU，如果可
运行池中的线程优先级相同，那么就随机选择一个线程，使其占用CPU。处于运行状态的线程会一直运行，直至它不得不放弃 CPU  
```

### 请说出与线程同步以及线程调度相关的方法

```
（1） wait()：使一个线程处于等待状态，并且释放所持有的对象的锁；

（2）sleep()：使一个正在运行的线程处于睡眠状态，是一个静态方法，sleep() 不释放锁，调用此方法要处理 InterruptedException 异常；

（3）yield()：使当前线程从运行状态变为就绪状态；

（4）notify()：唤醒一个处于等待状态的线程，当然在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由 JVM 确定唤醒哪个线程，而且与优先级无关；

（5）notifyAll()：唤醒所有处于等待状态的线程，该方法并不是将对象的锁给所有线程，而是让它们竞争，只有获得锁的线程才能进入就绪状态；
```

### sleep() 和 wait() 有什么区别

```
	sleep的作用是让线程休眠指定的时间，在时间到达时恢复。	 
  wait()的作用是是线程就进入到一个和该对象相关的等待池中（进入等待队列，也就是阻塞的一种，叫等待阻塞），同时释放对象锁，并让出CPU资源，待指定时间结束后返还得到对象锁。等待的线程只是被激活，但是必须得再次获得锁才能继续往下执行，也就是说只要锁没被释放，原等待线程因为为获取锁仍然无法继续执行。
    
补充：
1.属于不同的两个类，sleep()方法是线程类（Thread）的静态方法，wait()方法是Object类里的方法。
2.sleep()方法不会释放锁，wait()方法释放对象锁。
3.sleep()方法可以在任何地方使用，wait()方法则只能在同步方法或同步块中使用。
4.sleep()必须捕获异常，wait()方法、notify()方法和notiftAll()方法不需要捕获异常。
5.sleep()使线程进入阻塞状态（线程睡眠），wait()方法使线程进入等待队列（线程挂起），也就是阻塞类别不同。
6.它们都可以被interrupted方法中断。
```

### Thread 类中的 yield 方法有什么作用

```
使当前线程从执行状态（运行状态）变为可执行态（就绪状态)    
```

### sleep()方法和 yield()方法有什么区别

```
1、sleep()方法给其他线程运行机会时不考虑线程的优先级，因此会给低优先级的线程以运行的机会；yield()方法只会给相同优先级或更高优先级的线程以运行的机会
2、线程执行 sleep()方法后转入阻塞（blocked）状态，而执行 yield()方法后转入就绪（ready）状态 
3、sleep()方法声明抛出 InterruptedException，而 yield()方法没有声明任何异常    
```

### notify() 和 notifyAll() 有什么区别

```
如果线程调用了对象的 wait()方法，那么线程便会处于该对象的等待池中，等待池中的线程不会去竞争该对象的锁

notifyAll() 会唤醒所有的线程，notify() 只会唤醒一个线程
notifyAll() 调用后，会将全部线程由等待池移到锁池，然后参与锁的竞争，竞争成功则继续执行，
如果不成功则留在锁池等待锁被释放后再次参与竞争。而 notify()只会唤醒一个线程，具体唤醒哪
一个线程由虚拟机控制    
```

## 并发关键字

### synchronized 的作用

```
在 Java 中，synchronized 关键字是用来控制线程同步的，就是在多线程的环境下，synchronized 修饰的代码段不被多个线程同时执行。synchronized 可以修饰静态方法，实例方法和代码块
```

### synchronized的使用方式

```
修饰静态方法：对当前类对象加锁，进入同步代码前要获得当前类对象的锁

修饰实例方法：对当前实例对象加锁，进入同步代码前要获得当前实例对象的锁

修饰代码块：如果synchronized括号里面的是对象，锁的就是实例对象；如果括号里面的是class类，锁的是类
```

### synchronized 和 Lock 的区别

```
首先synchronized是Java关键字，Lock是 Java 接口；

synchronized 可以给方法、代码块加锁；而 lock 只能给代码块加锁。

synchronized 不需要手动获取锁和释放锁，使用简单，发生异常会自动释放锁，不会造成死锁；而 lock 需要自己加锁和释放锁，如果使用不当没有 unLock()去释放锁可能造成死锁。

通过 Lock 可以知道有没有成功获取锁，而 synchronized 却无法办到。
```

### synchronized 和 ReentrantLock 区别

```
相同点：
两者都是可重入锁

不同点：
1、本质：synchronized 是关键字，ReetrantLock是
2、加锁和释放锁：ReentrantLock 必须手动获取与释放锁，而 synchronized 不需要手动开启和释放锁
3、作用域：ReentrantLock 只能给代码块加锁，而 synchronized 可以给方法、代码块加锁。
```

### volatile 关键字的作用

```
Java 提供了 volatile 关键字来保证可见性和禁止指令重排（有序性）。volatile 提供 happens-before 的保证，同时确保一个线程对共享变量的修改能对其他线程是可见的。当一个共享变量被 volatile 修饰时，它会保证修改的值会立即被更新到内存，当有其他线程需要读取时，它会去内存中读取新值。
    
volatile 常用于多线程环境下的单次操作(单次读或者单次写)。    
```

### volatile 能保证原子性吗？volatile 能使得一个非原子操作变成原子操作吗

```
关键字volatile的主要作用是使变量在多个线程间可见，但无法保证原子性，所以对于多个线程访问共享变量需要加锁进行同步。

虽然volatile只能保证可见性不能保证原子性，但用volatile修饰long和double可以保证其操作原子性。
```

### synchronized 和 volatile 的区别是什么

```
synchronized 表示只有一个线程可以获取对象的锁，执行代码，阻塞其他线程。

volatile 表示变量在 CPU 的寄存器中是不确定的，必须从主存中读取。保证多线程环境下变量的可见性和禁止指令重排序
    
区别：
    1.使用范围：volatile 是变量修饰符；synchronized 可以修饰方法和代码块
    2.并发编程三要素：volatile保证可见性，不能保证原子性，synchronized 则可以可见性和原子性
    3.阻塞：volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞
```

## Lock体系

### Lock 接口

```
Lock 接口比同步方法和同步块提供了更具扩展性的锁操作。整体上来说 Lock 是 synchronized 的扩展版，Lock 提供了无条件的、可轮询的(tryLock 方法)、定时的(tryLock 带参方法)、可中断的(lockInterruptibly)、可多条件队列的(newCondition 方法)锁操作
```

### 乐观锁和悲观锁的理解

> 乐观锁

```
乐观锁：假设不会发生并发冲突，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于 write_condition 机制是乐观锁。在 Java中 java.util.concurrent.atomic 包下面的原子操作类就是使用了乐观锁的一种实现方式 CAS
```

> 悲观锁

```
悲观锁：假定会发生并发冲突，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。再比如 jdk1.6 以前的同步原语 synchronized 关键字的实现也是悲观锁
```

### 乐观锁的实现方式

> 版本号机制

```
版本号机制：一般是在数据表中加上一个版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读到的version值与当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功
```

> CAS算法

```
java 中的 Compare and Swap 即 CAS ，当多个线程尝试使用 CAS 同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值（B）。如果内存地址里面的值 V 和 预期原值 A 的值是一样的，那么就将内存里面的值 V 更新成新值 B。CAS是通过无限循环来获取数据的，如果在第一轮循环中，a 线程获取地址里面的值被 b 线程修改了，那么 a 线程需要自旋，到下次循环才有可能机会执行。
```

### 什么是可重入锁（ReentrantLock）

```
可重入锁:当前线程获取该锁再次获取时不会被阻塞

在java关键字synchronized隐式支持重入性，synchronized通过获取自增，释放自减的方式实现重入

重入性的实现原理:
以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。
```

### ReadWriteLock 是什么

```
ReadWriteLock 是一个读写锁接口，读写锁是用来提升并发程序性能的锁分离技术，ReentrantReadWriteLock 是 ReadWriteLock 接口的一个具体实现，实现了读写的分离，读锁是共享的，写锁是独占的，读和读之间不会互斥，读和写、写和读、写和写之间才会互斥，提升了读写的性能。

而读写锁有以下三个重要的特性：
1、公平选择性：支持非公平（默认）和公平的锁获取方式，吞吐量还是非公平优于公平。
2、可重入：读锁和写锁都支持线程可重入。
3、锁降级：遵循先获取写入锁，然后获取读取锁，最后释放写入锁。写锁能够降级成为读锁，但是，从读取锁升级到写入锁是不可能的。
```

## ThreadLocal

### ThreadLocal 是什么

```
ThreadLocal 是一个本地线程局部变量工具类，在每个线程中都创建了一个 ThreadLocalMap 对象，简单说 ThreadLocal 就是一种以空间换时间的做法，每个线程可以访问自己内部 ThreadLocalMap 对象内的 value。通过这种方式，避免资源在多线程间共享。

原理：线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程间共享。Java提供ThreadLocal类来支持线程局部变量，是一种实现线程安全的方式。但是在管理环境下（如 web 服务器）使用线程局部变量的时候要特别小心，在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。任何线程局部变量一旦在工作完成后没有释放，Java 应用就存在内存泄露的风险
```

### ThreadLocal造成内存泄漏的原因

```
ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用，而 value 是强引用。所以，如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，ThreadLocalMap 中就会出现key为null的Entry。假如我们不做任何措施的话，value 永远无法被GC 回收，这个时候就可能会产生内存泄露
```

### ThreadLocal内存泄漏解决方案

```
每次使用完ThreadLocal，都调用它的remove()方法，清除数据。

在使用线程池的情况下，没有及时清理ThreadLocal，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用ThreadLocal就跟加锁完要解锁一样，用完就清理
```

## 线程池

### 什么是线程池

```
线程池顾名思义就是事先创建若干个可执行的线程放入一个池（容器）中，需要的时候从池中获取，线程不用自行创建，使用完毕不需要销毁线程而是放回池中，从而减少创建和销毁线程对象的开销
    
四种线程池：
    1、newSingleThreadExecutor：创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行
    
    2、newFixedThreadPool：创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。如果希望在服务器上使用线程池，建议使用 newFixedThreadPool方法来创建线程池，这样能获得更好的性能
    
    3、newCachedThreadPool：创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60 秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说 JVM）能够创建的最大线程大小
    
    4、newScheduledThreadPool：创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求
```

### 线程池中 submit() 和 execute() 方法有什么区别

```
接收参数：execute()只能执行 Runnable 类型的任务。submit()可以执行 Runnable 和 Callable 类型的任务

返回值：submit()方法可以获取异步计算结果 Future 对象，而execute()没有

异常处理：submit()方便Exception处理
```

### 有哪几种创建线程池的方式

```
使用 Executors 工具类创建线程池
     ExecutorService executorService = Executors.newSingleThreadExecutor();

使用ThreadPoolExecutor构造函数创建线程池
     ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 5, 200, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(5));
```

### ThreadPoolExecutor构造函数重要参数分析

```
ThreadPoolExecutor 3 个最重要的参数
1、corePoolSize ：核心线程数，定义了最小可以同时运行的线程数。
2、maximumPoolSize ：线程池中允许存在的最大工作线程数。
3、workQueue：工作队列的长度。当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，任务就会被存放在队列中。

ThreadPoolExecutor 其他常见参数
1、keepAliveTime：线程池中的线程数大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime才会被回收销毁；
2、unit ：keepAliveTime 参数的时间单位。
3、threadFactory：创建新线程的线程工厂
4、handler ：当工作队列已满并且同时运行的线程数达到最大工作线程数时，新加入的任务就会走拒绝策略
```

### ThreadPoolExecutor拒绝策略

```
ThreadPoolExecutor.AbortPolicy（默认）：抛出 RejectedExecutionException来拒绝新任务的处理。

ThreadPoolExecutor.CallerRunsPolicy：用调用者所在的线程来执行任务。但是这种策略会降低对于新任务提交速度，影响程序的整体性能。

ThreadPoolExecutor.DiscardPolicy：不处理新任务，直接丢弃掉。

ThreadPoolExecutor.DiscardOldestPolicy：丢弃最早的未处理的任务。
```

