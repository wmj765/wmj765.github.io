title: jvm系列--内存分配和回收策略
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-10-29 16:57:00
subtitle:
header-img:
---
### 对象优先在Eden区分配
对象的内存分配，大方向上就是在堆上进行分配，当Eden区中没有足够的空间分配时，会进行一次minor GC--年轻代GC。
写一段测试代码来测试下
参数：

```
-Xmx20M -Xms20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+UseSerialGC
```

```
/**
 * @author wangmj
 * @since 2019/1/29
 */
public class EdenGCDemo {
    private static final int _1MB = 1024 * 1024;

    public static void main(String[] args) {
        byte[] allocation1, allocation2, allocation3, allocation4;

        allocation1 = new byte[2 * _1MB];
        allocation2 = new byte[2 * _1MB];
        allocation3 = new byte[2 * _1MB];
        //发生minor Gc
        allocation4 = new byte[4 * _1MB];
    }
}
```
运行结果

```
[GC (Allocation Failure) [DefNew: 6270K->705K(9216K), 0.0334506 secs] 6270K->4801K(19456K), 0.0335107 secs] [Times: user=0.00 sys=0.00, real=0.03 secs] 
Heap
 def new generation   total 9216K, used 7172K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  78% used [0x00000000fec00000, 0x00000000ff250ed0, 0x00000000ff400000)
  from space 1024K,  68% used [0x00000000ff500000, 0x00000000ff5b04e0, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 4096K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  40% used [0x00000000ff600000, 0x00000000ffa00020, 0x00000000ffa00200, 0x0000000100000000)
 Metaspace       used 3421K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 370K, capacity 388K, committed 512K, reserved 1048576K
```
可以看出发生了一次minor GC，Eden内存由6M变成几乎为0，堆内存由6M变成了5M(?为甚么堆内存会减小，没想明白？)。我们看到form Survivor内存增加了将近1M， 老年代内存增加了4M。说明最后的4M直接被分配到了老年代。
当老年代所剩的内存区域小于新生代区域并且小于平均每次收集新生代区域的平均值则会发生majorGC/fullGC,我们将上面代码简单修改下即可验证

```
/**
 * @author wangmj
 * @since 2019/1/29
 */
public class EdenGCDemo {
    private static final int _1MB = 1024 * 1024;

    public static void main(String[] args) {
        byte[] allocation1, allocation2, allocation3, allocation4;

        allocation1 = new byte[2 * _1MB];
        allocation2 = new byte[3 * _1MB];
        //此时发生minorGC
        allocation3 = new byte[2 * _1MB];
        //发生majorGC
        allocation4 = new byte[4 * _1MB];
    }
}
```
执行结果

```
"C:\Program Files\Java\jdk1.8.0_161\bin\java" -Xmx20M -Xms20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 "-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\lib\idea_rt.jar=51703:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\bin" -Dfile.encoding=UTF-8 -classpath "C:\Program Files\Java\jdk1.8.0_161\jre\lib\charsets.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\deploy.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\access-bridge-64.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\cldrdata.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\dnsns.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\jaccess.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\jfxrt.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\localedata.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\nashorn.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\sunec.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\sunjce_provider.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\sunmscapi.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\sunpkcs11.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\ext\zipfs.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\javaws.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\jce.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\jfr.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\jfxswt.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\jsse.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\management-agent.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\plugin.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\resources.jar;C:\Program Files\Java\jdk1.8.0_161\jre\lib\rt.jar;D:\workspace\personalpj\spring-cloud-servers\spring-cloud-moudle\spring-cloud-moudle-study\target\classes" com.spring.cloud.moudle.study.demo.jvm.EdenGCDemo
[GC (Allocation Failure) [PSYoungGen: 7306K->840K(9216K)] 7306K->5968K(19456K), 0.0032098 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 840K->0K(9216K)] [ParOldGen: 5128K->5824K(10240K)] 5968K->5824K(19456K), [Metaspace: 3397K->3397K(1056768K)], 0.0074300 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 9216K, used 6468K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 78% used [0x00000000ff600000,0x00000000ffc510c8,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 ParOldGen       total 10240K, used 5824K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 56% used [0x00000000fec00000,0x00000000ff1b02c8,0x00000000ff600000)
 Metaspace       used 3428K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 372K, capacity 388K, committed 512K, reserved 1048576K
```
可以看出先是发生了一次minorGC然后又发生了一次fullGC
### 大对象直接进入老年代
当对象足够大以至于Eden区域没有连续的足够区域来容纳，对象将直接分配到老年代，我们写段代码来测试

### 长期存活的对象将放入到老年代中
当年龄超过15岁的对象会被放入到老年代中