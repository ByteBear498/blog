---
date: 2019-08-08 21:57:00
status: public
tags: 
 - 并发
categoies:
 - java
title: Java并发测试CyclicBarrier与CountDownLatch
---



### CountDownLatch

相关的api

```java
//构造方法
public CountDownLatch(int count) 
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public void await() throws InterruptedException { };   
//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { }; 
//将count值减1
public void countDown() { }; 
```

简单来说它的执行逻辑是：先初始化`count`，当执行`countDown()`时`count`会减1，当`count`为零时之前因为`await()`被挂起的线程都会被唤醒。



示例代码：

```java
public static void main(String[] args) {
    int bash = 10;
    CountDownLatch countDownLatch = new CountDownLatch(1);
    for (int index = 0; index < bash; index++) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    countDownLatch.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Thread:" + Thread.currentThread().getName() + "准备完毕,time:" + System.currentTimeMillis());
            }
        }).start();
    }
    countDownLatch.countDown();
}
```

```
Thread:Thread-6准备完毕,time:1565252560307
Thread:Thread-2准备完毕,time:1565252560307
Thread:Thread-1准备完毕,time:1565252560307
Thread:Thread-9准备完毕,time:1565252560307
Thread:Thread-8准备完毕,time:1565252560307
Thread:Thread-5准备完毕,time:1565252560307
Thread:Thread-7准备完毕,time:1565252560307
Thread:Thread-4准备完毕,time:1565252560307
Thread:Thread-0准备完毕,time:1565252560307
Thread:Thread-3准备完毕,time:1565252560307
```



### CyclicBarrier

相关api

```java
//构造方法，参数parties指让多少个线程或者任务等待至barrier状态
public CyclicBarrier(int parties)
//用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；
public int await() throws InterruptedException, BrokenBarrierException { };
//让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { }; 
```



简单来说它的执行逻辑是：`await()`执行时间表示该线程进入barrier状态，当进行barrier状态的数据达到`parties`时，所有barrier状态下的线程会被唤醒



示例代码：

```java
public static void main(String[] args) {
    int bash = 10;
    CyclicBarrier cyclicBarrier = new CyclicBarrier(bash);
    for (int index = 0; index < bash; index++) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.println("Thread:" + Thread.currentThread().getName() + "准备完毕,time:" + System.currentTimeMillis());
            }
        }).start();
    }
}
```

```
Thread:Thread-8准备完毕,time:1565252611728
Thread:Thread-2准备完毕,time:1565252611729
Thread:Thread-1准备完毕,time:1565252611729
Thread:Thread-6准备完毕,time:1565252611728
Thread:Thread-5准备完毕,time:1565252611728
Thread:Thread-9准备完毕,time:1565252611728
Thread:Thread-7准备完毕,time:1565252611728
Thread:Thread-0准备完毕,time:1565252611728
Thread:Thread-3准备完毕,time:1565252611729
Thread:Thread-4准备完毕,time:1565252611729
```

