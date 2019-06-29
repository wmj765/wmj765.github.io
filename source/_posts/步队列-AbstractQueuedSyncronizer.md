title: 同步队列-AbstractQueuedSyncronizer
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-07-19 16:51:00
subtitle:
header-img:
---
### 根据AQS自己实现一个独占锁

```
/**
 * 独占锁自定义实现
 *
 * @author wangmj
 * @since 2019/1/18
 */
public class MutexLock {
    //自定义内部类继承AQS
    //独占锁只需实现tryAcquire与tryRelease方法
    //共享锁需要实现tryAcquireShared与tryReleaseShared方法
    private static class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire(int arg) {
            //cas将state变为arg
            if (compareAndSetState(0, arg)) {
                //设置独占线程为当前线程
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }


        @Override
        protected boolean tryRelease(int arg) {
            //若状态值为0，则说明锁已释放
            if (getState() == 0)
                throw new IllegalMonitorStateException();
            //设置独占线程为空，并将state更新为0
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }
    }

    private final Sync sync = new Sync();

    public void lock() {
        sync.acquire(1);
    }

    public void unlock() {
        sync.release(1);
    }

}

```
以上代码实现了一个简单的独占锁，下面我们写个测试类来测试下是否生效

```
/**
 * 独占锁测试类
 *
 * @author wangmj
 * @since 2019/1/15
 */
public class MuxaLockTest {
    //安全数
    private int safeCount = 0;
    //不安全数
    private int unsafeCount = 0;
    //自定义独占锁
    private MutexLock lock = new MutexLock();

    public static void main(String[] args) throws InterruptedException {
        MuxaLockTest lockTest = new MuxaLockTest();
        for (int j = 0; j < 100; j++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int i = 0; i < 1000; i++) {
                        lockTest.safeCount();
                        lockTest.unsafeCount();
                    }
                }
            }).start();
        }

        //等待使所有线程都执行完毕
        Thread.sleep(2000);

        System.out.println("safeCount = " + lockTest.safeCount);
        System.out.println("unsafeCount = " + lockTest.unsafeCount);

    }

    /**
     * 加锁，线程安全
     */
    private void safeCount() {
        lock.lock();
        try {
            safeCount++;
        } finally {
            lock.unlock();
        }
    }

    /**
     * 不加锁，会造成线程安全问题
     */
    private void unsafeCount() {
        unsafeCount++;
    }
}
```
测试结果若下

```
safeCount = 100000
unsafeCount = 99935
```
说明我们自己定义的独占锁生效，能让线程串行化执行。那么AQS的原理是什么呢？推荐大家看个文章，精髓就是下面这个图片。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190118143707728.png)

内部维护了一个队列，当tryAcquire获取失败时，会将此线程放入队列尾部，并且将线程park()等待，释放的时候会先将头结点unpark()，并将第二个节点变成头结点，相当于出队操作；中间有很多设计巧妙的细节，如放入队列的时候会先尝试快速直接将任务放置到队尾操作

```
//快速将任务放置到队尾
if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
```
当快速失败的时候，会CAS自旋将队列入队，方法如下

```
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```
AQS原理连接如下，讲的很详细很到位
[Java并发之AQS详解](http://www.cnblogs.com/waterystone/p/4920797.html)