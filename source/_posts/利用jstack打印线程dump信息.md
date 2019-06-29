title: 利用jstack打印线程dump信息
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-07-01 16:47:00
subtitle:
header-img:
---
1.进入到jdk/bin目录

2.jstack pid > /home/data/dump

3.查看各线程状态下的数量



grep java.lang.Thread.State /home/data/dump9001 | awk '{print $2$3$4$5}' | sort |uniq -c



4.分析：37个线程处于WAITING(onobjecmonitor)状态

5.打开dump文件具体查看waiting状态线程都在干什么



