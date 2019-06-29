title: java concurrent包下的锁
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-07-29 16:52:00
subtitle:
header-img:
---
### concurrent包下锁的种类
- 接口：Lock、ReadWriteLock、Condition
- 实现类：ReentrantLock、ReentrantReadWriteLock、StampedLock
- 抽象类：AbstractQueuedSychronizer

简单梳理下锁的关系ReentrantLock实现了Lock接口，并内部实现了AQS同步器；ReentrantReadWriteLock实现了ReadWriteLock接口，并有内部类实现类AQS同步器，可以看出无论上面两种锁实现的关键都是AQS同步器，足以看出想要深入理解锁，必须先掌握AQS同步器，可以查看我的上一篇介绍AQS的博客【[同步队列-AbstractQueuedSyncronizer](https://blog.csdn.net/wmj765/article/details/86539873)】下面将逐一介绍并写demo来理解锁的使用方法。

#### ReentrantLock可重入锁分析及使用
- volatile int state 状态值，很重要的一个参数，来标识同步状态，可以先将其理解为当前已经获取锁的次数(不准确，为入门者方便理解，后面会具体分析)。
- sync extends AQS 内部类，继承同步器，需要自己重写tryAcquire()、tryRelease()或者tryAcquireShared()、tryReleaseShared方法；一个是独占锁的实现，一个是可共享的实现

##### 共享锁与独占锁

独占锁的实现上篇博客写了个例子，这里在简单总结下，独占就相当于同时只有一个线程可以获取到锁，其他线程则阻塞在队列中，此时只需在获取锁的时候判断state值是否为0，若为0，则cas自旋将1赋值给state，若为1，则将当前线程放入到队列中，内部结构为双向链表。
共享锁即同时可以有多个线程获取到锁（CountDownLatch），思路其实和独占锁大体相同，只不过state值将不是以1位分界，下面我自己实现了一个简单的共享锁

```
import java.util.concurrent.locks.AbstractQueuedSynchronizer;

/**
 * 自定义共享锁，继承AQS，重写tryAcquireShared/tryReleaseShared
 *
 * @author wangmj
 * @since 2019/1/18
 */
public class CountDownLock {
    private final class sync extends AbstractQueuedSynchronizer {
        //自旋CAS将state设置成剩余可进入线程数
        @Override
        protected int tryAcquireShared(int reduce) {
            for (; ; ) {
                //当前状态值
                int current = getState();
                //剩余可进入的线程
                int rest = current - reduce;
                //当rest<0将线程任务放入队列等待
                if (rest < 0 || compareAndSetState(current, rest)) {
                    return rest;
                }
            }
        }


        @Override
        protected boolean tryReleaseShared(int increment) {
            for (; ; ) {
                int current = getState();
                int newCount = current + increment;
                if (newCount > 2)
                    throw new IllegalArgumentException();
                if (compareAndSetState(current, newCount)) {
                    return true;
                }
            }
        }

        sync(int count) {
            if (count < 0)
                throw new IllegalMonitorStateException();
            setState(count);
        }
    }

    private final sync sync = new sync(2);

    public void lock() {
        sync.acquireShared(1);
    }

    public void unLock() {
        sync.releaseShared(1);
    }
}
```
上面代码固定实现了可以2个线程共享的功能，继续写一段测试代码来验证是否成功

```
/**
 * 共享锁测试类 结果应为两个线程同时获取到锁，然后等待1S，即线程成对出现
 *
 * @author wangmj
 * @since 2019/1/18
 */
public class CountDownLockTest {

    static class task implements Runnable {
        CountDownLock lock = new CountDownLock();

        @Override
        public void run() {
            lock.lock();
            try {
                Thread.sleep(1000);
                System.out.println("currentThread" + Thread.currentThread().getName());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unLock();
            }
        }
    }


    public static void main(String[] args) throws InterruptedException {
        task task = new task();
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(task);
            thread.start();
        }
    }
}
```
结果是线程成对出现，并间隔一秒，即同时可以有两个线程来获取到锁

##### 公平锁与非公平锁
公平锁与非公平锁的主要区别是公平锁会完全按照先入先出的顺序来执行等待队列中的线程，而非公平锁则在获取锁的时候会先尝试当前线程获取到锁，若当前线程获取锁失败，则加入等待队列后和公平锁没有区别，下面我写了个简单的例子来看公平锁和非公平锁

```
import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 继承ReentrantLock，重写getQueuedThreads,将当前队列反转成正序
 *
 * @author wangmj
 * @since 2019/1/19
 */
public class RentrantLockDemo extends ReentrantLock {

    public RentrantLockDemo(boolean fair) {
        super(fair);
    }

    @Override
    public Collection<Thread> getQueuedThreads() {
        List list = new ArrayList(super.getQueuedThreads());
        Collections.reverse(list);
        return list;
    }
}
```
继续写一个测试类，来测试公平锁与非公平锁

```
/**
 * 测试公平锁与非公平锁的区别
 *
 * @author wangmj
 * @since 2019/1/19
 */
public class ReentrantLockTest {
    private static RentrantLockDemo fairLock = new RentrantLockDemo(true);
    private static RentrantLockDemo unfairLock = new RentrantLockDemo(false);


    static class Task implements Runnable {
        private RentrantLockDemo lock;

        public Task(RentrantLockDemo lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            lock.lock();
            try {
                Thread.sleep(100);
                System.out.println("currentThread" + Thread.currentThread().getName());
                System.out.println("fairThreads:" + lock.getQueuedThreads());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {

        Task task = new Task(unfairLock);
        for (int i = 0; i < 100; i++) {
            Thread thread = new Thread(task);
            thread.start();
        }
    }
}
```
测试结果如下

```
currentThreadThread-0
fairThreads:[Thread[Thread-1,5,main], Thread[Thread-2,5,main], Thread[Thread-3,5,main], Thread[Thread-4,5,main]]
currentThreadThread-1
fairThreads:[Thread[Thread-2,5,main], Thread[Thread-3,5,main], Thread[Thread-4,5,main]]
currentThreadThread-2
fairThreads:[Thread[Thread-3,5,main], Thread[Thread-4,5,main]]
currentThreadThread-3
fairThreads:[Thread[Thread-4,5,main]]
currentThreadThread-4
fairThreads:[]
```
可以看出，线程会按照等待队列中的顺序依次执行，而非公平锁则不会完全按照顺序来执行