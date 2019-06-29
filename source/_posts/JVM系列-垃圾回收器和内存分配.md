title: JVM系列--垃圾回收器和内存分配
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-08-28 16:55:00
subtitle:
header-img:
---
### 判断对象是否存活

 - 引用计数法
 - 可达性分析算法
 ##### 引用计数法：
 引用计数法主要为记录一个对象被引用的次数，当引用数大于0，则不会被垃圾回收；对象没被一次引用，则计数加1，对象失效时计数减一；这个方法对jvm回收来说非常高效，但是不会很准确，假如说两个对象互相引用，即使这两个对象没有其他地方应用，也会一直存在，不会被回收。
 ##### 可达性分析
 可达性分析主要由JVM根据GC root为跟，依次向下找其引用的对象，所有在这个引用链上的对象都不会被回收，一些对象即使有引用链，但是其根节点不是GC Root对象，其也会被回收。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129155249143.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
GC Root的对象包括以下四种
- 虚拟机栈所引用的对象
- 静态变量所引用的对象
- 常量所引用的对象
- 本地方法栈所引用的对象

对象即使没有在引用链上，也不会一次被判死刑，其会在finalize()方法中有一次escape gc 的机会。
判断一个对象是否有finalize()逃生机会有两种，
- 对象是否重写了finalize()方法
- 对象是否经历过finalize()方法

只有重写finalize()方法并且没有执行过finalize()方法才会有机会拯救自己，下面写一个程序验证。

```
/**
 * 对象重写finalize() escape GC
 * @author wangmj
 * @since 2019/1/28
 */
public class FinalizeEscapeGc {
    public static FinalizeEscapeGc HOOK = null;

    public void isAlive() {
        System.out.println("i am alive");
    }

    @Override
    protected void finalize() throws Throwable {
        System.out.println("finalize executed start");
        super.finalize();
        FinalizeEscapeGc.HOOK = this;
        System.out.println("finalize executed end");
    }

    public static void main(String[] args) throws InterruptedException {
        HOOK = new FinalizeEscapeGc();
        //对象没有可达性时，只有一次escape机会，
        HOOK = null;
        //会首先判断对象是否有必要执行finalize方法
        //当对象没有覆盖finalize()方法或者对象没有调用过finalize()方法则认为没有必要执行
        System.gc();
        Thread.sleep(500);
        //由于覆盖了finalize()方法并且第一次调用，所有对象成功escape
        if (HOOK != null) {
            HOOK.isAlive();
        }else {
            System.out.println("i am dead");
        }


        HOOK = null;
        System.gc();
        Thread.sleep(500);
        //对象已经调用一次，则不会再次调用finalize
        if (HOOK != null) {
            HOOK.isAlive();
        }else {
            System.out.println("i am dead");
        }
    }
}
```
执行结果

```
finalize executed start
finalize executed end
i am alive
i am dead
```
### 垃圾收集算法
- 标记清除算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129160615534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
步骤：首先遍历查询所有有引用的对象进行标记，第二步遍历堆，把所有未标记的对象进行清除
缺点：会产生内存碎片
- 复制算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019012916072959.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
步骤：需要同样大小的内存，先遍历所有的引用对象并进行复制，复制的同时进行碎片的整理，为了解决两倍大小的内存，设置堆内存的时候设置成Eden区和Survivor区域，复制的时候将存活的对象复制到Survivor区域
缺点：需要两倍大小的内存
- 标记-整理算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129160942705.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
步骤；结合上面两种的算法，先遍历所有有引用的对象并对其标记；遍历所有堆中没有被引用的对象		    并清除；对内存进行重新整理；
缺点：复杂性提高
