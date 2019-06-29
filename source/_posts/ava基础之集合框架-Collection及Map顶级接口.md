title: java基础之集合框架--Collection及Map顶级接口
catalog: true
author: wangmj
header-img: Demo.png
tags: []
categories: []
date: 2017-04-18 18:29:00
subtitle:
---
# java基础之集合框架--Collection及Map顶级接口

最近看了百度的面试题，面试内容基本都是基础知识，好多问题没有答好，所以定个小目标，在年前争取把java主流的基础知识都巩固一遍；那就以使用最频繁的集合框架开始。


## Collection及Map类图
### collection类图
![Collection类图](http://img.blog.csdn.net/20171030131133723?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd21qNzY1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看出来，Collection下的子接口有三个分别为Set,List,Queue，其中Set为无序的且不可重复的集合，List为有序可重复的集合，queue为先入先出的队列。

### Map类图
![这里写图片描述](http://img.blog.csdn.net/20171030131940860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd21qNzY1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

collection接口主要包含的方法如下：add()、addAll()、contain()、remove()、hashCode()、size()、toArray()等方法

关于Collection接口方面百度的技术面试主要问题：
Collection有哪些子类或者子接口（上面图即是常用的）；
问：接口可以定义常量么？
答：接口肯定是可以定义常量的，并且只能定义常量，但是其实接口中定义公有常量是不推荐的，除非这个常量与这个接口及其实现的子类有非常密切的关系，如integer的最大值等；effectiveJava一书中也明确表示不建议定义常量接口，实现常量接口会把实现细节暴露到导出的API中，并会对实现此接口的配置类造成污染。
还有一些特别基础的问题如：抽象类与接口的区别；为什么要用接口；为什么是单继承等；
还有一个面试题说的是==与equals的区别（String a="s"），这个其实很重要，对下面介绍set、map等结构有非常重要的作用；下面就解析下这道题。

### equals与==区别
介绍之前，先说明下，java的类型包括两类，一个为基础数据类型，一个为引用类型；在没有重写equals的方法情况下，equals与==比较结果相同，都是对引用地址的比较；基础数据类型都重写了equals方法，拿string类来说，它重写了object的equals方法

```
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        //在不想等的情况下，判断是否为String类型，若为String类，则比较两个值是否想到
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```
下面来分析下怎么来比较两个内存地址是否相同呢？有些朋友误以为默认情况下，hashCode返回的就是对象的存储地址，事实上这种看法是不全面的，确实有些JVM在实现时是直接返回对象的存储地址，但是大多时候并不是这样，只能说可能存储地址有一定关联。下面是HotSpot JVM中生成hash散列值的实现：

```
static inline intptr_t get_next_hash(Thread * Self, oop obj) {
  intptr_t value = 0 ;
  if (hashCode == 0) {
     // This form uses an unguarded global Park-Miller RNG,
     // so it's possible for two threads to race and generate the same RNG.
     // On MP system we'll have lots of RW access to a global, so the
     // mechanism induces lots of coherency traffic.
     value = os::random() ;
  } else
  if (hashCode == 1) {
     // This variation has the property of being stable (idempotent)
     // between STW operations.  This can be useful in some of the 1-0
     // synchronization schemes.
     intptr_t addrBits = intptr_t(obj) >> 3 ;
     value = addrBits ^ (addrBits >> 5) ^ GVars.stwRandom ;
  } else
  if (hashCode == 2) {
     value = 1 ;            // for sensitivity testing
  } else
  if (hashCode == 3) {
     value = ++GVars.hcSequence ;
  } else
  if (hashCode == 4) {
     value = intptr_t(obj) ;
  } else {
     // Marsaglia's xor-shift scheme with thread-specific state
     // This is probably the best overall implementation -- we'll
     // likely make this the default in future releases.
     unsigned t = Self->_hashStateX ;
     t ^= (t << 11) ;
     Self->_hashStateX = Self->_hashStateY ;
     Self->_hashStateY = Self->_hashStateZ ;
     Self->_hashStateZ = Self->_hashStateW ;
     unsigned v = Self->_hashStateW ;
     v = (v ^ (v >> 19)) ^ (t ^ (t >> 8)) ;
     Self->_hashStateW = v ;
     value = v ;
  }
 
  value &= markOopDesc::hash_mask;
  if (value == 0) value = 0xBAD ;
  assert (value != markOopDesc::no_hash, "invariant") ;
  TEVENT (hashCode: GENERATE) ;
  return value;
}
```
　因此有人会说，可以直接根据hashcode值判断两个对象是否相等吗？肯定是不可以的，因为不同的对象可能会生成相同的hashcode值。虽然不能根据hashcode值判断两个对象是否相等，但是可以直接根据hashcode值判断两个对象不等，如果两个对象的hashcode值不等，则必定是两个不同的对象。如果要判断两个对象是否真正相等，必须通过equals方法。

　　也就是说对于两个对象，如果调用equals方法得到的结果为true，则两个对象的hashcode值必定相等；

　　如果equals方法得到的结果为false，则两个对象的hashcode值不一定不同；

　　如果两个对象的hashcode值不等，则equals方法得到的结果必定为false；

　　如果两个对象的hashcode值相等，则equals方法得到的结果未知。

以上即为Collection接口的基础知识，及百度一些面试题，下一章将介绍map接口及其子类，因为set是基于map实现的，懂得map的原理也就明白了set的原理