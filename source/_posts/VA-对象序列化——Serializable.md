title: JAVA 对象序列化——Serializable
catalog: true
author: wangmj
tags: []
categories: []
date: 2017-11-29 16:33:00
subtitle:
header-img:
---
##JAVA 对象序列化——Serializable
上一篇分析HashMap时，每次put新值得时候将key、value放在一个transient Node[] table数组里面，大家注意到前面的关键字transient了吗？他代表什么意义呢；我们知道Map接口实现了顶级接口Serializable，表示Map对象的所有属性都是可序列化的，但是加上transient关键字就表示此属性不在被序列号。
### serializable作用
Java的对象序列化是指将那些实现了Serializable接口的对象转换成一个字符序列，并能够在以后将这个字节序列完全恢复为原来的对象。这一过程甚至可通过网络进行，这意味着序列化机制能自动弥补不同操作系统之间的差异。 只要对象实现了Serializable接口（记住，这个接口只是一个标记接口，不包含任何的方法）
### serializable简单实现
先建立一个简单类，实现Serializable接口
```
public class Test implements Serializable {
    private static final long serializeId = 129183856L;
    private int i;
    private String s;

    public Test(int i, String s) {
        this.i = i;
        this.s = s;
    }

    @Override
    public String toString() {
        return "Test{" +
                "i=" + i +
                ", s='" + s + '\'' +
                '}';
    }
}
```
如果我们想要序列化一个对象，首先要创建某些OutputStream(如FileOutputStream、ByteArrayOutputStream等)，然后将这些OutputStream封装在一个ObjectOutputStream中。这时候，只需要调用writeObject()方法就可以将对象序列化，并将其发送给OutputStream（记住：对象的序列化是基于字节的，不能使用Reader和Writer等基于字符的层次结构）。而饭序列的过程（即将一个序列还原成为一个对象），需要将一个InputStream(如FileInputstream、ByteArrayInputStream等)封装在ObjectInputStream内，然后调用readObject()即可。

```
public class SerializeMain {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Test test = new Test(1, "ssss");
        System.out.println(test);
        //序列化操作
        ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("test.out"));
        os.writeObject("======");
        os.writeObject(test);
        os.close();
		//反序列化操作
        ObjectInputStream oi = new ObjectInputStream((new FileInputStream("test.out")));
        String s = (String) oi.readObject();
        Test t = (Test) oi.readObject();
        System.out.println(s);
        System.out.println(t);
    }
}
```
这时打印出信息为

```
======
Test{i=1, s='ssss'}
```

若我们将Test类s属性改成private transient String s，再次运行主程序，此时我们打印的信息为

```
======
Test{i=1, s='null'}
```

从上面结果能够看出来经过transient关键字修饰的字段是不能够被序列化的。

### 为什么Node[]要加transient关键字
怎么理解? 看一下HashMap.get()/put()知道, 读写Map是根据Object.hashcode()来确定从哪个bucket读/写. 而Object.hashcode()是native方法, 不同的JVM里可能是不一样的.

打个比方说, 向HashMap存一个entry, key为 字符串"STRING", 在第一个java程序里, "STRING"的hashcode()为1, 存入第1号bucket; 在第二个java程序里, "STRING"的hashcode()有可能就是2, 存入第2号bucket. 如果用默认的串行化(Entry[] table不用transient), 那么这个HashMap从第一个java程序里通过串行化导入第二个java程序后, 其内存分布是一样的. 这就不对了. HashMap现在的readObject和writeObject是把内容 输出/输入, 把HashMap重新生成出来.