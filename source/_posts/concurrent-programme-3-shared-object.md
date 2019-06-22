---
title: Java并发编程-共享对象
date: 2019-06-22 09:57:44
categories:
- Java
tags: 
- Java并发编程
---

## 可见性

可见性相当微妙，发生的错误可能与直觉大相径庭。在单线程环境中，向一个变量写入值，然后在没有干涉的情况下读取这个值，很自然的会希望得到相同的值。但是当读写发生在不同的线程中，情况可能就不一样了。为了确保跨线程的内存可见性，必须使用同步机制。

```
public class NonVisibility {

	static boolean ready = false;
	static int num = 0;

	static class ReadThread extends Thread {
		@Override
		public void run() {
			while (!ready) {
				Thread.yield();
			}
			System.err.println(num);
		}
	}

	public static void main(String[] args) {
		new ReadThread().start();
		num = 42;
		ready = true;
	}
}
```

**“重排序”**现象，在单个线程中，只要对结果不会产生影响，就不能保证其中的操作会严格按照写定的顺序执行-即使重排序会对其他线程造成影响。

### 过期数据

在NonVisibility中，过期数据导致打印错误，在生产环境中，过期值可能导致程序的崩溃，脏数据的产生，错误的计算或者无限的循环。

### 非原子的64位操作

非volitile的long和double数据在JVM中允许分开成两个32位进行操作，这时使用volitile或者同步机制可以解决。

### 锁和可见性

内置锁可以用来确保一个线程以某种可预见的方式看到另一个线程的影响，当B执行到与A相同的锁监视的同步块时，A同步块之前所做的事情，对B都是可见的。如果没有同步，就没有这样的保证。

```
锁不仅仅是同步互斥的，也可以是内存可见的。
为了保证所有线程都能看到共享的、可变变量的最新值，读取和写入线程必须使用公共的锁进行同步。
```

{% asset_img 1561172279707.png 同步对可见性的保证 %}

### volitile变量

当一个域声明为volatile类型后，编译器和运行时会监视这个变量：它是共享的，对它的操作不会与其他内存操作一起被重排序。volatile变量不会缓存到寄存器或处理器其他地方。所以读取volatile变量时，总是返回最新的数据。

理解volatile变量时，可以想象其与下面的代码功能大致类似。只不过get和set方法取代了对volatile变量的读写操作。但是访问volatile变量的操作不会加锁，也不会有执行线程的阻塞，所以volatile相对sychronized而言只是一种轻量级的同步机制。

```java
public int value;
public sychronized int get() {
	return value;
}
public sychronized void set(int value) {
	this.value = value;
}
```

从内存可见角度看，写入volatile变量就像退出了同步块，读取volatile变量就像进入同步块。但是不推荐依赖volatile变量来控制可见性，volatile极其脆弱而且并不直观。

```wiki
只有当volatile变量能够简化实现和同步策略的验证，才使用它们。
正确使用volatile变量的方式：
用于确保它们所引用的对象状态的可见性，或者用于表示重要的生命周期事件的发生。
```

volatile变量固然方面，但也存在限制。通常volatile被当做标识完成、中断、状态的标记使用。使用volatile必须格外小心，比如volatile不能让自增操作（count++）原子化，除非只有一个线程进行操作。

```
加锁可以保证可见性和原子性，但是volatile只能保证可见性。
```

## 发布和溢出

发布（publish）一个对象是其能够被当前范围之外的代码所使用。又是需要确保对象内部状态不被暴露。如果变量发布了内部状态可能危及到封装性，并使程序难以维持稳定；如果发布对象，还没有完成构造，同样危及线程安全。一个对象在尚未准备好就进行发布，就称为溢出。下面为对象溢出的例子。

```java
// 发布对象
public static final Map<Integer, String> map;
public void init() {
	map = new HashMap<>();
}
```

```java
// 允许内部可变数据溢出
class UsafeState{
    private String[] states = new String[]{"XA", "TCC"};
    public String[] getStates() {
        return states;
    }
}
```

```java
// 隐式地允许this引用溢出，因为内部被包含了隐式的引用
class Escape {
    public Escape(EventSource source) {
   		source.addEventListener(new EventListener() {
          	public void onClick(Event event) {
                doSomethine(event);
            }  
        });    
    }
}
```

### 安全构建实践

对象至于在构造函数返回后，才是一个可预言、稳定的状态。如果this引用在构造过程中溢出，这样的对象被认为是"没有正确构建的"。

```wiki
不要让this引用在构造期间溢出。
```

一个常见的导致this引用在构造期间溢出的常见错误，是在构造函数中启动一个线程。无论是显示的（通过它传递给构造函数）还是隐式的，this引用几乎总被新线程共享。在构造函数创建线程没有错，但是最好不要先启动它，在构造函数结束后通过一个start方法进行启动。

如果要在构造器中增加监听或者启动线程，可以使用一个私有函数或者工厂方法。

```java
public class SafeListener {
    private final EventListener listener;
    public SafeListener() {
        this.listener = new EventListener() {
            public void onClick(Event e) {
                doSomethin(e);
            }
        };
    }
    public static SafeListener newInstance(EventSource source) {
        SafeListener sl = new SafeListener();
        source.addListener(sl.listener);
        return sl;
    }
}
```

## 线程封闭

线程封闭是实现线程安全的最简单的方式之一。当对象封闭在一个线程中，这种做法自动称为线程安全的。

Swing发展了线程封闭技术。Swing的可视化组件和数据模型并不是线程安全的，通过将它们限制到Swing的事件分发线程中实现线程安全。

### Ad-hoc线程限制

指维护线程限制性的任务全部落在实现上。因为没有可见性修饰符与本地变量等语言特性协助将对象限制在目标线程上，所以这种方法很容易出错。鉴于ad-hoc线程限制具有易损性，应当节制使用它。用一种线程限制的强形式（栈限制或ThreadLocal）取代它。

栈限制是线程限制的一种特例，只能通过本地变量才能触及对象。其他线程无法访问。与ad-hoc相比更容易维护，更健壮。

```java
public int loadTheArk(Collection<Animal> candidates) {
	SortedSet<Animal> animals;
    int numPairs=0;
    // animals 限制在方法中，不要让它们逸出！
    animals = new TreeSet<>();
    animals.addAll(candidates);
    .....
}
```

维护对象引用的栈限制，需要保证引用的对象没有逸出。在线程内部上下文使用非线程安全的对象仍然可以保证线程的安全性。但是一线开发任务编码的那一刻需要清楚的文档化，防止后期维护人员错误的放任对象溢出。

### ThreadLocal

通常用于可变的单例或全局变量设计中，出现共享。每个线程单独维护一个变量，这样就可以防止并发问题。

```
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<>() {
	public Connection initialValue() {
		return DriverManager.getConnection(DB_URL);
	}
}
public static Connection getConnection() {
	return connectionHolder.get();
}
```

在Netty的ByteBuf中，就是利用ThreadLocal去进行byte数组的分配，防止接受请求频繁创建byte数组，这样既可以节省内存、又可以并发问题。

ThreadLocal很容易滥用：比如将他们所封闭的数据作为全局变量的许可证。线程本地变量会降低重用性，引入隐晦的类间耦合，应当谨慎的使用。

## 不可变性

不可变对象永远是线程安全的。final关键字是构成不可变对象的一部分，被final修饰的对象仍然可能是可变的。

##  安全发布















