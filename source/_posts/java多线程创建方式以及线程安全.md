---
title: java多线程创建方式以及线程安全
top: false
cover: false
toc: true
mathjax: true
date: 2021-05-12 10:25:02
password:
summary: 本文主要介绍多线程的一些知识。
tags:
- 原创
- 多线程
categories:
- java
---
## 什么是线程

 - 线程被称为轻量级进程，是程序执行的最小单位，它是指在程序执行过程中，能够执行代码的一个执行单位。每个程序程序都至少有一个线程，也即是程序本身。

## 线程的状态
 - `新建（New）`：创建后尚未启动的线程处于这种状态
 - `运行（Runable）`：Runable包括了操作系统线程状态的Running和Ready，也就是处于此状态的线程有可能正在执行，也有可能正在等待着CPU为它分配执行时间。
 - `等待（Wating）`：处于这种状态的线程不会被分配CPU执行时间。等待状态又分为无限期等待和有限期等待，处于无限期等待的线程需要被其他线程显示地唤醒，没有设置Timeout参数的Object.wait()、没有设置Timeout参数的Thread.join()方法都会使线程进入无限期等待状态；有限期等待状态无须等待被其他线程显示地唤醒，在一定时间之后它们会由系统自动唤醒，Thread.sleep()、设置了Timeout参数的Object.wait()、设置了Timeout参数的Thread.join()方法都会使线程进入有限期等待状态。
 - `阻塞（Blocked）`：线程被阻塞了，“阻塞状态”与”等待状态“的区别是：”阻塞状态“在等待着获取到一个排他锁，这个时间将在另外一个线程放弃这个锁的时候发生；而”等待状态“则是在等待一段时间或者唤醒动作的发生。在程序等待进入同步区域的时候，线程将进入这种状态。
 - `结束（Terminated）`：已终止线程的线程状态，线程已经结束执行。
![线程的生命周期](https://img-blog.csdnimg.cn/20210511133223945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDE3Mjc2,size_16,color_FFFFFF,t_70)


## 多线程创建方法

### 继承Thread

```java
/**
 * @Author GocChin
 * @Date 2021/5/11 11:56
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript:
 */
class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println(currentThread().getName()+"运行了");
    }
}
class Test{
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        System.out.println(Thread.currentThread().getName()+"：运行了");
        myThread.start();
    }
}

```
### 实现Runable接口创建多线程

```java
/**
 * @Author GocChin
 * @Date 2021/5/11 12:37
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript: 实现Runable接口的方式创建多线程
 * 1.创建一个实现了Runable接口的类
 * 2.实现类去实现Runable中的抽象方法，run();
 * 3.创建实现类的对象
 * 4.将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
 * 5.通过Thread类的对象调用start()
 */
class MThread implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i<100;i++){
            if (i%2!=0){
                System.out.println(i);
            }
        }
    }
}
public class ThreadTest1 {
    public static void main(String[] args) {
        //3.创建实现类的对象
        MThread mThread = new MThread();
        //4.将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
        Thread thread = new Thread(mThread);
        thread.start();
    }
}

```

 Thread和Runable创建多线程对比

 	开发中：优先使用Runable
 	1.实现的方式没有类的单继承的局限性。
	2.实现的方式跟适合处理多个线程有共享数据的情况。
	联系：Thread类中也实现了Runable，两种方式都需要重写run()。

###  实现Callable接口

```coffeescript
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * @Author GocChin
 * @Date 2021/5/11 13:03
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript:
 */
class MCallable implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        int sum=0;
        for(int i=0;i<100;i++){
            sum+=i;
        }
        return sum;
    }
}
public class CallableTest {
    public static void main(String[] args) {
        //执行Callable 方式，需要FutureTask 实现实现，用于接收运算结果
        FutureTask<Integer> integerFutureTask = new FutureTask<Integer>(new MCallable());
       
        new Thread(integerFutureTask).start();
        //接受线程运算后的结果
        Integer integer = null;
        try {
            integer = integerFutureTask.get();
            System.out.println(integer);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

    }
}

```
与Runable相比，Callable功能更强大
		

	相比run()方法可以有返回值
	方法可以抛出异常
	支持泛型的返回值
	需要借助FutureTask类，比如获取返回结果


### 使用线程池进行创建
 **线程池创建的好处**
 

 - 提高响应速度（减少了创建新线程的时间）
 - 降低资源消耗（重复利用线程池中线程，不需要每次都创建）
 - 便于线程管理：
	 - corePoolSize:核心线程池的大小
	 - maximumPoolSize:最大线程数
	 - keepAliveTime:线程没有任务时最多保持多长时间后悔中止
```coffeescript
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @Author GocChin
 * @Date 2021/5/11 13:10
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript:
 */
class Thread1 implements Runnable{

    @Override
    public void run() {
        for (int i=1;i<30;i++){
            System.out.println(Thread.currentThread().getName() + ":" + i);
        }
    }
}
public class ThreadPool {
    public static void main(String[] args) {
        //创建线程池
        ExecutorService executorService= Executors.newFixedThreadPool(10);
        Thread1 threadPool = new Thread1();
        for (int i=0;i<5;i++){
            //为线程池分配任务
            executorService.submit(threadPool);
        }
        //关闭线程池
        executorService.shutdown();
    }
}

```

## Thread中的常用方法

 - start()：启动当前线程；调用当前线程的run();
 - run()：通常需要重写Thread类中的此方法，将创建的线程要执行的操作声明在此方法中。
 - currentThread()：静态方法，返回当前代码的线程。
 - getName()：获取当前线程的名字。
 - setName()：设置当前线程的名字。
 - yield()：释放当前cpu的执行权，切换线程执行。
 - join()：在线程a中调用线程b的join(),此时线程a会进入阻塞状态，知道线程b完全执行完毕，线程a 才结束阻塞状态。
 - stop()：强制线程生命期结束。（过时了，不建议使用）
 - isAlive()：判断线程是否还活着。
 - sleep(long millitime)：让当前线程睡眠指定的事milltime毫秒。在指定的millitime毫秒时间内，当前线程是阻塞状态。
 ## 线程的优先级
  #####  线程的优先级等级
  
 - MAX_PRIORITY:10
 - MIN_PRIORITY:1
 - NORM_PRIORITY:5

  ##### 涉及的方法
  

 - getPriority()：返回线程的优先值
 - setPriority(int newPriority)：改变线程的优先级

 ##### 说明

 - 线程创建时继承父线程的优先级
 - 低优先级知识获得调度的概率低，并非一定是在高优先级线程之后才被调用

 ## 线程的同步
![问题](https://img-blog.csdnimg.cn/20210511133641454.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDE3Mjc2,size_16,color_FFFFFF,t_70)

 ### 多线程卖票
 
基于实现Runable的方式实现多线程买票
```java
package demo2;

/**
 * @Author GocChin
 * @Date 2021/5/11 13:37
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript: 创建三个窗口买票，总票数为100张，使用Runable接口的方式
 *      存在线程安全问题，待解决
 */
class Thread2 implements Runnable{

    private  int ticket=100;
    @Override
    public void run() {
        while (true){
            if (ticket>0) {
                System.out.println(Thread.currentThread().getName() + ":买票，票号为：" + ticket);
                ticket--;
            }else {
                break;
            }
        }
    }
}
public class Test1 {
    public static void main(String[] args) {
        Thread2 thread2 = new Thread2();
        Thread t1 = new Thread(thread2);
        Thread t2 = new Thread(thread2);
        Thread t3 = new Thread(thread2);
        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");
        t1.start();
        t2.start();
        t3.start();
    }
}

```
实现结果，存在重复的票
![实现结果](https://img-blog.csdnimg.cn/20210511134337217.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDE3Mjc2,size_16,color_FFFFFF,t_70)
如果在买票方法中加入sleep函数

```java
 public void run() {
        while (true){
            if (ticket>0) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + ":买票，票号为：" + ticket);
                ticket--;
            }else {
                break;
            }
        }
    }
```
则运行结果可能会出现-1，表示也是不正常的
![出现-1的结果](https://img-blog.csdnimg.cn/2021051113481650.png)

理想情况![卖票理想状态](https://img-blog.csdnimg.cn/20210511135148558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDE3Mjc2,size_16,color_FFFFFF,t_70)
极端情况
![极端情况](https://img-blog.csdnimg.cn/20210511135322506.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDE3Mjc2,size_16,color_FFFFFF,t_70)
**在java中，我们通过同步机制，来解决线程的安全问题。**

### 同步代码块

```java
synchronized(同步监视器){
	//需要被同步的代码
}
```
说明

 - 操作共享数据的代码就是需要被同步的代码。
 - 共享数据：多个线程共同操作的变量，比如本题中的ticket就是共享数据。
 - 同步监视器：俗称：锁。任何一个类的对象都可以充当锁。要求：`多个线程必须要共用统一把锁`。
 - 同步的方式，解决了线程的安全问题---好处。但是操作同步代码时，只能有一个线程参与，其他线程等待。相当于是一个单线程的过程，效率低。-----局限性
 - 使用Runable接口创建多线程的方式中，可以使用`this`关键字；在继承Thread类中创建多线程中，慎用`this`充当同步监视器，可以考虑使用当前类充当同步监视器。`Class clazz = Windows.class`    因此 类也是一个对象
 - 包裹操作共享数据的代码 `不能多也不能少`
 

**修改之后的代码：**

```coffeescript
package demo2;

/**
 * @Author GocChin
 * @Date 2021/5/11 13:37
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript: 创建三个窗口买票，总票数为100张，使用Runable接口的方式
 *      存在线程安全问题，待解决
 */
class Thread2 implements Runnable{

    private  int ticket=100;

    Object object = new Object();
    @Override
    public void run() {
        while (true){
            synchronized(object) { //括号中的内容可以直接使用当前对象this去充当
                if (ticket > 0) {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ":买票，票号为：" + ticket);
                    ticket--;
                } else {
                    break;
                }
            }
        }
    }
}
public class Test1 {
    public static void main(String[] args) {
        Thread2 thread2 = new Thread2();
        Thread t1 = new Thread(thread2);
        Thread t2 = new Thread(thread2);
        Thread t3 = new Thread(thread2);
        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");
        t1.start();
        t2.start();
        t3.start();
    }
}

```
**结果**
![运行结果](https://img-blog.csdnimg.cn/20210511140703443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDE3Mjc2,size_16,color_FFFFFF,t_70)
**继承Thread的方式，去使用同步代码块，需要将声明的锁对象设为statci，否则创建的对象的同步监视器不唯一，就无法实现。**

```java
package demo2;

/**
 * @Author GocChin
 * @Date 2021/5/11 14:45
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript:
 */
class WindowsTest2 extends Thread{
    private static int ticket=100;
    private static   Object obj = new Object();

    @Override
    public void run() {
        while (true){
            synchronized (obj){ //这里不能使用this去充当，可以直接写一个Test.class   类也是对象
                if (ticket>0){
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(getName()+":买票，票号为："+ticket);
                    ticket--;
                }else {
                    break;
                }
            }
        }
    }
}
public class  Test2{
    public static void main(String[] args) {
        WindowsTest2 w1 = new WindowsTest2();
        WindowsTest2 w2 = new WindowsTest2();
        WindowsTest2 w3 = new WindowsTest2();
        w1.setName("窗口一");
        w2.setName("窗口二");
        w3.setName("窗口三");
        w1.start();
        w2.start();
        w3.start();
    }
}

```

### 同步方法
如果操作共享数据的代码`完整的声明在一个方法`中，可以将此方法声明为同步的。




---
通过实现Runable的方式实现同步方法。
```java
package demo2;

/**
 * @Author GocChin
 * @Date 2021/5/11 13:37
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript: 创建三个窗口买票，总票数为100张，使用Runable接口的方式
 * 存在线程安全问题，待解决
 */
class Thread3 implements Runnable {

    private int ticket = 100;


    @Override
    public void run() {
        while (true) {
            show();
        }

    }
    private synchronized void show(){
        if (ticket > 0) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ":买票，票号为：" + ticket);
            ticket--;
        }
    }
}

public class Test3 {
    public static void main(String[] args) {
        Thread3 thread3 = new Thread3();
        Thread t1 = new Thread(thread3);
        Thread t2 = new Thread(thread3);
        Thread t3 = new Thread(thread3);
        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");
        t1.start();
        t2.start();
        t3.start();
    }
}

```
通过实现继承Thread的方式实现同步方法。使用的同步监视器是this，则不唯一，就会报错。所以将该方法定义为static。当前的同步换时期就变成`Test4.class`了

```java
package demo2;

/**
 * @Author GocChin
 * @Date 2021/5/11 14:45
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript:
 */
class WindowsTest4 extends Thread{
    private static int ticket=100;
    private static   Object obj = new Object();

    @Override
    public void run() {
        while (true){
            show();
        }

    }
    public static synchronized void show(){//同步监视器不是this了，而是当前的类
//    public synchronized void show(){//同步监视器是this  ，t1,t2,t3
        if (ticket>0){
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":买票，票号为："+ticket);
            ticket--;
        }
    }
}
public class  Test4{
    public static void main(String[] args) {
        WindowsTest4 w1 = new WindowsTest4();
        WindowsTest4 w2 = new WindowsTest4();
        WindowsTest4 w3 = new WindowsTest4();
        w1.setName("窗口一");
        w2.setName("窗口二");
        w3.setName("窗口三");
        w1.start();
        w2.start();
        w3.start();
    }
}

```
---
总结

 - 同步方法仍然设计到同步监视器，只是不需要我们去显示的声明。
 - 非静态的同步方法，同步监视器是：this静态的同步方法中，同步监视器是类本身。
### Lock锁解决线程安全问题
---
 #### synchronize与lock的异同
 相同
 
 - 都可以解决线程安全问题
 
不同
 - synchronize机制在执行相应的同步代码以后，自动的释放同步监视器；Lock需要手动的启动同步lock(),同时结束同步也需要手动的实现unlock()。


建议优先使用顺序
Lock------>同步代码块（已经进入了方法体，分配了相应资源）---->同步方法（在方法体之外）
```java
package demo2;

import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author GocChin
 * @Date 2021/5/11 15:58
 * @Blog: itdfq.com
 * @QQ: 909256107
 * @Descript:
 */
class Lock1 implements Runnable{
    private int ticket=50;

    //1.实例化
    private ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        while(true){
            try {
                //2.调用lock锁定方法
                lock.lock();
                if (ticket>0){
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()+"售票，票号为："+ticket);
                    ticket--;
                }else{
                    break;
                }
            } finally {
                //3.调用解锁方法
                lock.unlock();
            }
        }
    }
}
public class LockTest1 {
    public static void main(String[] args) {
        Lock1 lock1 = new Lock1();
        Thread t1 = new Thread(lock1);
        Thread t2 = new Thread(lock1);
        Thread t3 = new Thread(lock1);
        t1.setName("窗口一");
        t2.setName("窗口二");
        t3.setName("窗口三");
        t1.start();
        t2.start();
        t3.start();

    }
}

```
