title: jvm系列- 垃圾收集器
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-10-21 16:56:00
subtitle:
header-img:
---
### 垃圾收集器
#### 	serial收集器
serial收集器是最原始的收集器，其是一个单线程收集器，当其垃圾回收时会"stop the world"，暂停所有程序，但是其在CPU核数较少的时候会比较高效，因为单线程减少了线程切换的开销。
看图明白其原理
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129161914822.png)
#### ParNew 收集器
ParNew收集器实际上是serial收集器的多线程版，其主要是在server模式下的新生代收集器，因为只有ParNew和serial收集器可以和CMS收集器共同使用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129162429994.png)
#### Parallel Scavenge收集器
Parallel Scavenge收集器是一个新生代收集器，也是一个多线程收集器，看起来和ParNew收集器一样，但是他的关注点不一样
其主要关注的是可控制的吞吐量，被称为吞吐量优先收集器，可以设置在一定时间内完成垃圾回收，但是不是设置的时间越小越好，当时间越小则会造成收集频率增加，新生代内存动态减少等后果，jdk1.8默认使用的是这种收集器
#### Serial old 收集器
是一种老年代的收集器，是单线程收集器，使用标记-整理算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129163541280.png)
#### parallel old收集器
parallel old收集器是parallel scavenge老年代版本，是多线程线程收集器，采用的是标记-整理算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129171759546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
#### CMS收集器
是一种获取最短时间停顿的收集器，主要是“标记-清除”算法，主要由四个步骤
- 初始标记
- 并发标记
- 重新标记
- 并发清除
![在这里插入图片描述](https://img-blog.csdnimg.cn/201901291722033.png)
可以看出，初始标记和重复标记需要stop the world，而并发标记和并发清除不需要停顿

#### G1收集器
G1收集器是jdk1,7才开始投入使用的收集器，其和CMS收集器主要区别是可以精准预测停顿时间在什么时间段，其主要实现是讲堆分成若干个Region区域，对每个区域进行分区收集
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129172542722.png)
### 垃圾收集器参数总结
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019012917262952.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/2019012917264981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
