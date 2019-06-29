title: jvm类加载机制
catalog: true
author: wangmj
tags: []
categories: []
date: 2018-11-30 17:00:00
subtitle:
header-img:
---
### 类加载时机
类从被加载到虚拟机内存，到卸载出内存，要经历的**生命周期：**
加载(Loading)=>验证(Verification)=>准备(Preparing)=>解析(Resolving)=>初始化(Initialization)=>使用(Using)=>卸载(Unloading)，其中验证、准备、解析成为连接阶段，可以参考netty里面的过程来记忆，netty中接收数据后会先进行接收数据，此过程对应加载过程；当接收完数据我们要来看下协议是否符合我们定义的数据协议，包括魔数、版本号等等，此阶段对应验证阶段；准备阶段主要是开辟内存并且设置类变量初始值阶段；当我们验证数据通过后，我们会按照协议来解析命令，此时对应解析阶段；当解析完成后，当我们用到这个类的时候就会进行初始化。下面具体介绍下类加载的具体过程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190217193404859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtajc2NQ==,size_16,color_FFFFFF,t_70)
### 类加载过程
#### 加载
在加载阶段，虚拟机需要完成以下3件事情
- 通过类的全限定名将类转化成二进制流
- 将二进制流转化成运行时数据结构
- 在方法区中实例化一个java.lang.class对象，作为访问类的入口

加载没有完成连接可能已经开始，但是加载与连接的开始时间肯定是加载在前，连接在后；
#### 验证
验证阶段主要工作是验证类是否符合jvm规范，主要由下面四个内容来进行验证
- **文件格式验证**
主要包括魔数、版本号等等验证
- **元数据验证**
是否有父类子类、是否是抽象类、类中的字段等验证
- **字节码验证**
这个阶段主要是对类中的方法体进行验证
- **符号引用验证**
主要验证private、public等是否合法
#### 准备
准备阶段主要是对类中的变量分配内存并初始化值的过程，8中基本变量的初始值基本都为0，除了Boolean 变量为false；
#### 解析
解析过程是将符号引用转化为直接引用的过程，可以理解为将类和类中的变量解析成内存中的变量过程。
#### 初始化
初始化是执行类构造器方法的过程

### 类加载器
类加载器就是将类的全限定名转化成二进制流的Java外部实现的模块，类加载器应用在OSGI、热部署、代码加密等领域。
自定义类加载过程

```

import java.io.IOException;
import java.io.InputStream;

/**
 * @author wangmj
 * @since 2019/2/17
 */
public class ClassLoaderDemo {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        ClassLoader classLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                try {
                    String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                    InputStream inputStream = getClass().getResourceAsStream(fileName);
                    if (inputStream == null) {
                        return super.loadClass(name);
                    }
                    byte[] bytes = new byte[inputStream.available()];
                    inputStream.read(bytes);
                    return defineClass(name, bytes, 0, bytes.length);
                } catch (IOException e) {
                    e.printStackTrace();
                    throw new ClassNotFoundException(name);
                }
            }
        };
        Object instance = classLoader.loadClass("com.spring.cloud.moudle.study.demo.jvm.ClassLoaderDemo").newInstance();
        System.out.println(instance.getClass());
        System.out.println(instance instanceof com.spring.cloud.moudle.study.demo.jvm.ClassLoaderDemo);
    }
}
```
#### 双亲委派模式
双亲委派模式在加载类的时候不会先在自己定义的加载方法执行，而是先去父类中的classLoader方法加载，这样好处就是所有子类最终都会实现最顶层父类的类加载器，这样不会导致因不同之类定义不同的类加载器导致各个类的结果不一样的情况。
#### 非双亲委派模式
JNDI、JDBC等都是非双亲委派模式，JNDI需要加载外部代码来实现相应功能，但是启动类加载器不识别这些代码，只能由父类加载器调用子类的加载器来实现，违背了由下到上的原则，但是增加了灵活性。
OSGI、热部署等动态加载是进一步的非双亲委派模式，OSGI有bundle概念，每个bundle有自己的类加载器，当一个bundle被替换时，会同时替换类加载器，有点类似微服务的模块化