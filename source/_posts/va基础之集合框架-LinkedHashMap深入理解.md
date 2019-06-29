title: java基础之集合框架--LinkedHashMap深入理解
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-01-10 16:36:00
subtitle:
header-img:
---
### java基础之集合框架--LinkedHashMap深入理解
在看LinkedHashMap时还是建议先回顾下HashMap--LinkedHashMap的父亲，是不是感觉HashMap很重要，哈哈~~，它就是很重要，可以把它比作一个美轮美奂的精美建筑，一切设计都那么恰到好处！
[ java基础之集合框架--HashMap深入理解及应用](http://blog.csdn.net/wmj765/article/details/78427680)

#### LinkedHashMap底层结构图
LinkedHashMap底层为数组和双向链表，它继承了HashMap,生成数组index的逻辑与hashMap完全相同，只不过linkedHashMap的Entry里面多了两个元素，before和after，并定义了一个不可序列化的header，存储时，将第一个元素赋值给header，并将before指向header，下个元素put时，将上个元素的after指向此元素，并将此元素的before指向上个元素，最终则形成了图中的结构。

![这里写图片描述](http://img.blog.csdn.net/20171117161131083?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd21qNzY1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


#### LinkedHashMap之 put方法源码分析
我们知道，linkedHashMap继承了HashMap,它没有重写父类的put方法，所以调用时还会调用父类的put，但是其重写了hashMap的newNode方法，下面是关键代码，当put新值得时候，其会调用newNode方法；

```
	//初始化Node
      Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        //形成双向链表
        linkNodeLast(p);
        return p;
    }

	//初始化linkedHashMap的Entry
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }

	//组成双向链表
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)//第一次put值将值赋值给head
            head = p;
        else {//双向链表
            p.before = last;
            last.after = p;
        }
    }
```

上面为put的关键代码，基本思路就是相较于hashMap的entry多加了三个属性，head、before、after；head为static变量，当第一次put值A得时候将A赋值给head，并将A.before指向head，A.after为null，第二次put值B时将B.before指向A，并将A.after指向B，B.after为null.......以此类推，即可形成双向链表。

#### LinkedHashMap之 遍历源码分析

遍历之前，先把head赋值给next元素，然后调用内部类nextNode()方法，然后回依次去掉用next.after来取元素，直到next.after=head结束；伪代码类似这种for (LinkedHashMap.Entry e = head; e != null; e = e.after)

```
    abstract class LinkedHashIterator {
        LinkedHashMap.Entry<K,V> next;
        LinkedHashMap.Entry<K,V> current;
        int expectedModCount;

        LinkedHashIterator() {
            next = head;//将next赋值给head
            expectedModCount = modCount;
            current = null;
        }

        public final boolean hasNext() {
            return next != null;
        }

		//遍历
        final LinkedHashMap.Entry<K,V> nextNode() {
            LinkedHashMap.Entry<K,V> e = next;
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (e == null)
                throw new NoSuchElementException();
            current = e;
            next = e.after;
            return e;
        }

        public final void remove() {
            Node<K,V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            K key = p.key;
            removeNode(hash(key), key, null, false, false);
            expectedModCount = modCount;
        }
    }
```

以上就是LinkedHashMap分析