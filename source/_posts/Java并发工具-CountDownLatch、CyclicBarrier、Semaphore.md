title: Java并发工具-CountDownLatch、CyclicBarrier、Semaphore
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-08-08 16:53:00
subtitle:
header-img:
---
### CountDownLatch等待线程完成
CountDownLatch是一个等待所有线程完成的工具，类似join()方法，初始化的时候会调用构造器传递线程个数，调用countDown方法将count值减一，调用await方法会等待知道count=0才会通过。写一个简单的例子方便理解

```
import java.util.concurrent.CountDownLatch;

/**
 * countDownLatch测试类
 *
 * @author wangmj
 * @since 2019/1/24
 */
public class CountDownLatchDemo {
    private static CountDownLatch countDownLatch = new CountDownLatch(2);

    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("线程1开始");
                countDownLatch.countDown();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("线程2开始");
                countDownLatch.countDown();
            }
        }).start();
        //此时会等待线程1和线程2执行完才会执行下面代码
        countDownLatch.await();
        System.out.println("3");
    }
}

```

### Semaphore 信号量
信号量是一个可以控制同时有几个线程执行，其他线程等待的工具类，就好像一个门同时只有3个人通过，其他人则在门外等候，当三个人进门之后其他人才会下次三个进入门

```

import java.util.concurrent.*;

/**
 * @author wangmj
 * @since 2019/1/24
 */
public class SemaphoreDemo {

    //同时只有三个线程执行
    private static Semaphore semaphore = new Semaphore(3);

    private static ExecutorService executor = Executors.newFixedThreadPool(10);

    public static void main(String[] args) {

        long start = System.nanoTime();

        for (int i = 0; i < 10; i++) {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        semaphore.acquire();
                        Thread.sleep(1000);
                        System.out.println("thread name:" + Thread.currentThread().getName()+" and nanoTime:"
                                + TimeUnit.NANOSECONDS.toSeconds(System.nanoTime()-start));
                        semaphore.release();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        executor.shutdown();
    }

}
```
运行结果如下

```
thread name:pool-1-thread-3 and nanoTime:1
thread name:pool-1-thread-2 and nanoTime:1
thread name:pool-1-thread-1 and nanoTime:1
thread name:pool-1-thread-5 and nanoTime:2
thread name:pool-1-thread-4 and nanoTime:2
thread name:pool-1-thread-6 and nanoTime:2
thread name:pool-1-thread-7 and nanoTime:3
thread name:pool-1-thread-8 and nanoTime:3
thread name:pool-1-thread-9 and nanoTime:3
thread name:pool-1-thread-10 and nanoTime:4
```
可以看到同一时间只允许三个线程运行，以上的原理都是基于AQS同步器实现的，可以参考我上两篇博客