---
title: concurrent-programme-5-building-blocks
date: 2019-06-25 10:10:28
categories:
- Java
tags: 
- Java并发编程
---

## 同步容器

### 同步容器的问题

同步容器都是线程安全的。但是对于复合操作，可能需要额外的加锁操作。

```java
public static Object getLast(Vector v) {
	int index = v.size() - 1;
	return v.get(index);
}
public static Object deleteLast(Vector v) {
	int index = v.size() - 1;
	return v.remove(index);
}
```

这段代码看上去没有危害，但是对于调用者来说情况就不一样了。如果线程A执行了deleteLast方法后，线程B正好执行到getLast的return处，这样就会产生一个ArrayIndexOutOfBounds异常。可以对容器加锁避免这个问题。

```java
public static Object getLast(Vector v) {
	sychronized(v) {
		int index = v.size() - 1;
		return v.get(index);
	}
}
public static Object deleteLast(Vector v) {
    sychronized(v) {
		int index = v.size() - 1;
		return v.remove(index);
    }
}
```

```java
// 利用迭代器Itertor，可能会产生ArrayIndexOutOfBounds异常
for (Object obj : vector) {
	// ....
}

sychronized (vector) {
    for (Object obj : vector) {
		// ....
	}	
}

```

### 迭代器和ConcurrentModificationException

```java
List<Widget> widgets = Collections.sychronizedList(new ArrayList<>());
// 可能产生ConcurrentModificationException
for (Widget w : widgets) {
	doSomething(w);
}
```

有时widgets数量比较大，或者doSomething操作很耗时，对widgets加锁可能会降低性能，这是比较好的做法是将List复制一份进行迭代。但是需要根据需求和widgets的数量去权衡加锁或复制。

## 并发容器

同步容器利用容器自身进行串行访问，实现的线程安全，但是这样会严重削弱并发性。

并发容器就是为并发设计的。ConcurrentHashMap替代Map同步实现；读操作多时，使用CopyOnWriteArrayList替代List同步实现；新增Queue和BlockingQueue实现了队列；ConcurrentSkipListMap和ConcurrentSkipListSet作为SortedMap和SortedSet同步的替代品。

```wiki
用并发容器替换同步容易可以有效的提升高并发程序性能。
```

### ConcurrentHashMap

JDK7分离锁机制

## 阻塞队列和生产者-消费者模式

阻塞队列支持生产者-消费者设计模式。该模式不会发现一个工作就会执行，而是放在一个“TO DO”列表中。这种模式简化开发，解除了生产者和消费者之间相互依赖的代码，实现了解耦。最常见的生产者-消费者设计是将线程池和工作队列相结合的Executor。

## 阻塞和可中断的方法

线程可能会因为几个原因被阻塞或暂停：等待I/O操作结束，等待获得一个锁，等待Thread.sleep中唤醒，等待另一个线程的计算结果。

BlockingQueue的put和take方法会抛出受检查的InterruptedException，这与类库其他的一些方法是相同的，比如Thread.sleep。当一个方法能抛出InterruptedException时，表明这个方法是一个可阻塞方法。



























