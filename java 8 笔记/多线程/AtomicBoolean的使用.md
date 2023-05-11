# AtomicBoolean的使用

## 一、简介

AtomicBoolean是java.util.concurrent.atomic包下的原子变量，这个包里面提供了一组原子类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由JVM从等待队列中选择一个另一个线程进入

AtomicBoolean，在这个Boolean值的变化的时候不允许在之间插入，保持操作的原子性

> 方法：compareAndSet(boolean expect, boolean update)
>
> 作用：
>
> - 比较AtomicBoolean和expect的值，如果一致，执行方法内的语句。其实就是一个if语句
> - 把AtomicBoolean的值设成update  比较的是这两件事是一气呵成的，这连个动作之间不会被打断，任何内部或者外部的语句都不可能在两个动作之间运行

## 二、案例

在多线程环境中，我们通过判断一个boolan变量的值，然后修改该变量的值，之后进行操作；存在一个问题就是，多个线程可能都读到该变量的值是符合条件的，然后都去修改了变量的值；其实只需要一个线程执行就可以了，主要的原因就是因为if判断和set值是两个操作，这里面存在线程安全的问题

### 使用基本的boolean类型

```java
public class BarWorker implements Runnable {
 
    private String name;
 
    private static boolean exists = false;
 
    public BarWorker(String name) {
        this.name = name;
    }
 
    @Override
    public void run() {
        if (!exists) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            exists = true;
            System.out.println(name + ":enter");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(name + ":leave");
            exists = false;
        } else {
            System.out.println(name + ":give up");
        }
    }
 
    public static void main(String[] args) {
        BarWorker bar1 = new BarWorker("bar1");
        BarWorker bar2 = new BarWorker("bar2");
        new Thread(bar1).start();
        new Thread(bar2).start();
 
    }
}
```

上面为了模拟if判断和赋值操作的原子性，故意在之间设置了个时间间隔；执行结果

```java
bar1:enter
bar2:enter
bar1:leave
bar2:leave
```

从执行结果可以看出，两个线程都执行了对应的操作

### 使用AtomicBoolean

```java
public class AtomaticTest implements Runnable {
 
    private String name;
 
    private static AtomicBoolean exists = new AtomicBoolean(false);
 
    public AtomaticTest(String name) {
        this.name = name;
    }
 
    @Override
    public void run() {
        if(exists.compareAndSet(false, true)) {
            System.out.println(name + ":enter");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(name + ":leave");
            exists.set(false);
        }else{
            System.out.println(name +":give up");
        }
    }
 
    public static void main(String[] args) {
        AtomaticTest atomatic1 = new AtomaticTest("bar1");
        AtomaticTest atomatic2 = new AtomaticTest("bar2");
        new Thread(atomatic1).start();
        new Thread(atomatic2).start();
    }
}
```

执行结果

```java
bar2:enter
bar1:give up
bar2:leave
```

可见只执行了一个线程

