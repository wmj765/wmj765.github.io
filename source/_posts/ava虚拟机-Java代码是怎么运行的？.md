title: java虚拟机-- Java代码是怎么运行的？
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-05-20 16:42:00
subtitle:
header-img:
---
从虚拟机角度来说，其将Java代码编码为class文件，并放在虚拟机中的方法区。

运行时，当执行一个方法的时候，虚拟机会生产一个栈帧，可以不连续，当方法执行结束，将此栈帧弹出.

虚拟机包含 堆、方法区。PC寄存器、Java方法栈、本地方法栈(用 C++ 写的 )
