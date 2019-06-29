title: java基础之集合框架--HashSet深入理解
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-01-01 16:34:00
subtitle:
header-img:
---
###HashSet深入理解
前几天分析了HashMap,看懂hashMap后，再来看HashSet就简单多了，其实HashSet底层就初始化时即调用了HashMap，在看HashSet时建议先回顾下HashMap
[ java基础之集合框架--HashMap深入理解及应用](http://blog.csdn.net/wmj765/article/details/78427680)

#### HashSet源码分析
我们直接来看下hashset的源码，可以看到当我们初始化对象时，其实是初始化了一个hashmap，其他的有参构造器也分别调用了hashMap的有参构造方法，代码中就不展示了。

```
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    //存放HashMap的value，我们add时传的是key，value为new Object()
    private static final Object PRESENT = new Object();

    //无参构造器，其实就是一个hashMap
    public HashSet() {
        map = new HashMap<>();
    }
```
#### HashSet常用方法
再来看看HashSet的常用方法，size方法、add方法、遍历等等，其实都是调用map中的方法，其他方法就不一一列举了，在add方法时，map.put方法，key为add的对象，value为new　Object()；

```
//大小
public int size() {
        return map.size();
    }
    
//添加元素，若不重复返回true
public boolean add(E e) {
        return map.put(e, PRESENT)==null;

//遍历
public Iterator<E> iterator() {
        return map.keySet().iterator();
    }
```

其实只要我们get到hashMap，其实同时你也完全了解了HashSet；
最后有一个问题大家思考下，有一组学生，想要放进一个集合中，放在什么集合中能保证取出来的时候有序？HashSet/HashMap可以么