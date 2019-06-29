title: JVM系列-JVM运行时内存区域
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-08-11 16:54:00
subtitle:
header-img:
---
#### JVM运行时内存区域构成

 1. 程序计数器
 2. 虚拟机栈
 3. 本地方法栈
 4. 堆内存
 5. 方法区
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190128142701453.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
#### 程序计数器
程序计数器所占内存非常小，为**线程私有变量**，可以看住当前线程执行**行数**的记录，字节码解释器通过程序计数器来查找下一条指令，及线程恢复切换等操作。
#### Java虚拟机栈
Java虚拟机栈主要存放**基本类型变量**及**对象变量的内存地址**信息，为**线程私有变量**。每个方法在执行的时候都会创建一个栈帧(Stack Frame)用于存储局部变量表、方法出口等，此过程为压栈，当方法执行完毕会把此栈弹出，即出栈，所以其生命周期等同于变量的生命周期。
当申请栈的深度大于虚拟机所允许的深度，将抛出**StackOverflowError**
参数：**-Xss**
#### 本地方法栈
Java虚拟机栈包含本地房发栈，其主要存放native方法。
#### Java堆内存
堆内存是jvm管理的最大区域的内存，是线程共有的内存区域，主要存储对象变量；分为年轻代与年老代，年轻代分为Eden区域和From Survivor 和To Survivor区域，是垃圾回收主要回收管理的区域。
当申请的堆内存大于允许的堆内存空间将抛出**OutOfMemoryError**异常,设置堆内存大小参数：**-Xms20M**(最小堆内存) **-Xmx20M**(最大堆内存)
#### 方法区
主要存放类信息、常量数据、永久数据，抛出**OutOfMemoryError**异常
参数：**-MaxPermSize**

#### OutOfMemoryError异常制造
启动参数

```
-Xms20M -Xmx20M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=D:\file
```
程序：

```
/**
 * @author wangmj
 * @since 2019/1/9
 */
public class OOMTest {
    public static void main(String[] args) {
        List list = new ArrayList();
        while (true) {
            list.add(new HeapVo());
        }
    }
}

```
程序打印信息：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190128151229812.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)