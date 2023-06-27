# ThreadLocal原理详解和使用场景

## 1、概述

ThreadLocal意为线程本地变量，用于解决多线程并发时访问共享变量的问题。

所谓的共享变量指的是在堆中的实例、静态属性和数组；对于共享数据的访问受Java的内存模型（JMM）的控制，其模型如下

![image-20230627091319116](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230627091319116.png)

每个线程都会有属于自己的本地内存，在堆（也就是上图的主内存）中的变量在被线程使用的时候会被复制一个副本线程的本地内存中，当线程修改了共享变量之后就会通过JMM管理控制写会到主内存中。

很明显，在多线程的场景下，当有多个线程对共享变量进行修改的时候，就会出现线程安全问题，即数据不一致问题。常用的解决方法是对访问共享变量的代码加锁（synchronized或者Lock）。但是这种方式对性能的耗费比较大。在JDK1.2中引入了ThreadLocal类，来修饰共享变量，使每个线程都单独拥有一份共享变量，这样就可以做到线程之间对于共享变量的隔离问题。

简单的说就是，<font color ='red'>一个ThreadLocal在一个线程中是共享的，在不同线程之间又是隔离的（每个线程都只能看到自己线程的值）</font>

## 2、API介绍

- set(T value)：将当前线程的此线程局部变量的副本设置为指定的值
- get()：返回当前线程的此现场局部变量的副本中的值
- remove()：删除此现场局部变量的当前线程的值
- initialValue()：返回此现场局部变量的当前线程的初始值

```java
public class ThreadLocalTest {
 
	private static ThreadLocal<Integer> num = new ThreadLocal<Integer>() {
		// 重写这个方法，可以修改“线程变量”的初始值，默认是null
		@Override
		protected Integer initialValue() {
			return 0;
		}
	};
 
	public static void main(String[] args) {
		// 创建一号线程
		new Thread(new Runnable() {
			@Override
			public void run() {
				// 在一号线程中将ThreadLocal变量设置为1
				num.set(1);
				System.out.println("一号线程中ThreadLocal变量中保存的值为：" + num.get());
			}
		}).start();
 
		// 创建二号线程
		new Thread(new Runnable() {
			@Override
			public void run() {
				num.set(2);
				System.out.println("二号线程中ThreadLocal变量中保存的值为：" + num.get());
			}
		}).start();
 
		//为了让一二号线程执行完毕，让主线程睡500ms
		try {
			Thread.sleep(500);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		System.out.println("主线程中ThreadLocal变量中保存的值：" + num.get());
	}
}
```

数据结果：

```java
一号线程中ThreadLocal变量中保存的值为：1
二号线程中ThreadLocal变量中保存的值为：2
主线程中ThreadLocal变量中保存的值：0  
```

程序结果重点看的是主线程输出的是0，如果是一个普通变量，在一号线程和二号线程中将普通变量设置为1和2，那么在一二号线程执行完毕后在打印这个变量，输出的值肯定是1或者2。但使用ThreadLocal变量通过两个线程赋值后，在主线程程中输出的却是初始值0。在这也就是为什么“一个ThreadLocal在一个线程中是共享的，在不同线程之间又是隔离的”，每个线程都只能看到自己线程的值，这也就是ThreadLocal的核心作用：<font color ='red'>实现线程范围的局部变量</font>

## 3、原理

**每个Thread对象都有一个ThreadLocalMap，当创建一个ThreadLocal的时候，就会将该ThreadLocal对象添加到该Map中，其中键就是ThreadLocal，值可以是任意类型**

ThreadLocal存入一个值，实际上是向当前线程对象中的**ThreadLocalMap**存入值，ThreadLocalMap我们可以简单的理解成一个Map，而向这个Map存值的key就是ThreadLocal实例本身

```java
public void set(T value) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 获取当前线程的threadLocals字段
    ThreadLocalMap map = getMap(t);
    // 判断线程的threadLocals是否初始化了
    if (map != null) {
      //这里的this就是当前调用set方法的ThreadLocal实例
        map.set(this, value);
    } else {
        // 没有则创建一个ThreadLocalMap对象进行初始化
        createMap(t, value);
    }
}

```

> 总结：

JDK8之后，每个Thread维护一个ThreadLocalMap对象，这个Map的key是ThreadLocal实例本身，value是存储的值要隔离的变量，是泛型，其具体过程如下

- 每个Thread线程内部都有一个Map（ThreadLocalMap::threadlocals）
- Map里面存储ThreadLocal对象（key）和线程的变量副本（value）
- Thread内部的Map由ThreadLocal维护，由ThreadLocal负责向map获取和设置变量值
- 对于不同的线程，每次获取副本值时，别的线程不能获取当前线程的副本值，就形成了数据之间的隔离

## 4、ThreadLocal内存泄露问题

内存泄露问题：指程序中动态分配的堆内存由于某种原因没有被释放或者无法释放，造成系统内存的浪费，导致程序运行速度减慢或者系统奔溃等严重后果

ThreadLocal的内存泄露问题一般考虑和Entry对象有关，在上面的Entry定义可以看出ThreadLocal::Entry被弱引用所修饰。**JVM会将弱引用修饰的对象在下次垃圾回收中清除掉。**这样就可以实现ThreadLocal的生命周期和线程的生命周期解绑。

使用ThreadLocal造成内存泄露的问题是因为：`ThreadLocalMap的生命周期与Thread一致，如果不手动清除掉Entry对象的话就可能会造成内存泄露问题。`**因此，需要我们在每次在使用完之后需要手动的remove掉Entry对象**

## 5、ThreadLocal的应用场景

>  场景一：在重入方法中替代参数的显式传递

假如在我们的业务方法中需要调用其他方法，同时其他方法都需要用到同一个对象时，可以使用ThreadLocal替代参数的传递或者static静态全局变量。这是因为使用参数传递造成代码的耦合度高，使用静态全局变量在多线程环境下不安全。当该对象用ThreadLocal包装过后，就可以保证在该线程中独此一份，同时和其他线程隔离

> 场景二：全局存储用户信息

可以尝试使用ThreadLocal替代Session的使用，当用户要访问需要授权的接口的时候，可以现在拦截器中将用户的Token存入ThreadLocal中；之后在本次访问中任何需要用户用户信息的都可以直接冲ThreadLocal中拿取数据

> 场景三：解决线程安全问题

赖于ThreadLocal本身的特性，对于需要进行线程隔离的变量可以使用ThreadLocal进行封装

