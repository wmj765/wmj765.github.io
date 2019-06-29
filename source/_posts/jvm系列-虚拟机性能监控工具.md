title: jvm系列--虚拟机性能监控工具
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-11-03 16:58:00
subtitle:
header-img:
---
### JDK命令行工具
#### jps
jps可以列出正在运行的虚拟机内存，是一款简单单一的工具
命令 jps [options] [hostid]
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130220130302.png)
#### jstat
虚拟机统计信息监视工具，是运行期地位虚拟机性能问题的首选工具，他可以显示本地或者远程的类加载、内存、垃圾收集等信息。
命令：jstat [option vmid [interval[s|ms] {count}]]
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130220509169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
运行 jstat -gc pid 250 20表示每250ms打印一次gc信息，共输出20次
常用的还有jstat -gcutil
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130220554689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
#### jmap
jmap 用于生成堆存储快照信息(一般为heapdump和dump文件)
命令：jmap [options] pid
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130221316217.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
#### jstack
java堆栈跟踪工具，主要用于生成当前线程快照信息，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130221521939.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
### jdk可视化工具
#### jConsole
是一款jdk自带的分析jvm性能的插件，启动程序在jdk/bin目录下
#### VisualVM
是一款非常优秀的jdk自带jvm性能插件