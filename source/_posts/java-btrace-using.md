---
title: 不重启JVM，替换已加载的类
date: 2019-07-03 11:02:50
tags: 
	- Java
categories:
	- Java
---

[美团技术团队]: https://tech.meituan.com/	"美团技术团队"

前几天看博客在美团技术团队的一篇博客，[Java动态追踪技术探究](https://tech.meituan.com/2019/02/28/java-dynamic-trace.html)，所以写下此文。

## 问题

在排查线上程序代码出问题的时候，可以通过远程debug或者查看日志去定位问题。但是在远程debug没开、出问题的代码没有打印日志而且不可重启服务器的情况下该如何去定位问题。

如果能加载一个已经加载类的信息，然后修改字节码中目标方法所在的区域，然后重新加载这个类，这个方法区中的对象的行为就改变了。

## java.lang.instrument.Instrumentation

### retransformClasses(Class<?>... classes)

这个方法就是将给定的Class重新改变。当类被初始化加载后或者被重新定义后，初始的类文件字节可以被用**java.lang.instrument.ClassFileTransformer**转变。重新变化的步骤如下：

- 从初始class文件字节码开始
- 针对**canRetransform**为false，ClassFileTransformer transform方法返回上一次加载的或者重新定义的字节码。
- 针对**canRetransform**为true，ClassFileTransformer transform方法返回被转化后的字节码。
- 重新转化后的class文件字节码会被加载为新的class定义。

这个方法不会让它的初始化者重新运行。静态变量的值仍然是上一次调用的值。被重新转化的类的实例并不收到影响。重新转化可能影响方法体、常量池和属性。重新转化不能添加、移除、重命名域或方法，不能改变方法签名或改变继承。

### redefineClasses(ClassDefinition... definitions) 

这个方法用来直接替换一个class的定义，并不会牵扯到存在的class文件字节码。如果重新定义的方法有很活跃的栈帧，这些活跃的栈帧继续运行原始方法的字节，重新定义的方法应用在新的调用上。与retransformClasses方法相同的限制。

## 直接操作字节码

ASM框架是一个直接操作字节码的框架。cglib、spring等框架中字节码的操作都是基于这个框架。

## Btrace

[Btrace](https://github.com/btraceio/btrace)是基于ASM、Java Instrument API和Java Attach API开发的可动态追踪的工具。

### Btrace使用

Btrace的使用可以在github上的[wiki page](https://github.com/btraceio/btrace/wiki)查看。Btrace的动态操作使用过脚本插入的。我们可以看几个Btrace官方的脚本。

这个脚本是监控线程启动数的脚本。在线程启动时增加线程计数，然后以2000ms的频率打印当前的已经启动线程数。

```java
@BTrace public class ThreadCounter {
    // create a jvmstat counter using @Export
    @Export private static long count;
    @OnMethod(
        clazz="java.lang.Thread",
        method="start"
    ) 
    public static void onnewThread(@Self Thread t) {
        // updating counter is easy. Just assign to
        // the static field!
        count++;
    }
    @OnTimer(2000) 
    public static void ontimer() {
        // we can access counter as "count" as well
        // as from jvmstat counter directly.
        println(count);
        // or equivalently ...
        println(Counters.perfLong("btrace.com.sun.btrace.samples.ThreadCounter.count"));
    }
}
```

这个脚本是检测JVM内存的脚本，每隔4000ms打印堆和非堆的内存使用情况。

```java
@BTrace public class Memory {
    @OnTimer(4000)
    public static void printMem() {
        println("Heap:");
        println(Sys.Memory.heapUsage());
        println("Non-Heap:");
        println(Sys.Memory.nonHeapUsage());
    }
}
```

有个这两个Btrace例子脚本，我们也可以自己去写相应的脚本去实现自己想要的功能。

