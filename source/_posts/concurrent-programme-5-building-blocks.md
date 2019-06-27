---
title: Java并发编程-构建块
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

BlockingQueue的put和take方法会抛出受检查的InterruptedException，这与类库其他的一些方法是相同的，比如Thread.sleep。**当一个方法能抛出InterruptedException时，表明这个方法是一个可阻塞方法**。进一步看，如果它可以被中断，将可以提前结束阻塞状态。

中断是一种**协作**机制。一个线程不能迫使其他线程停止正在做的事情，或者去做其他事情；当线程A中断线程B，只是让B在某个方便的停止点是停止。

当代码中调用一个会抛出InterruptedException的方法时，有两种基本选择：

**传递InterruptedException**，将异常抛出去。或者先捕获，进行特定的操作后在抛出异常。

**恢复中断**，先捕获异常，在当前线程中调用interrupt方法让上层代码知道中断已经发生。

```java
public class TaskRunnable implements Runnable {
	private BlockingQueue<Task> taskQueue = ...
	@Override
	public void run() {
		try {
			processTask(taskQueue.take());
		} catch (IntterruptException e) {
			Thread.currentThread().interrupt();
		}
	}
}
```

## Sychronizer

Sychronizer是一个对象，根据本身的状态调节线程控制流。阻塞队列可以扮演一个Sychronizer角色。其他类型的Sychronizer包括信号量（Semaphore），关卡（barrier）， 闭锁（latch）。

### 闭锁

用来确保特性活动在其他活动完成后才继续执行（考虑CountDownLatch的使用）。

### FutureTask

FutureTask可作为闭锁，当将FutureTask提交后，Future.get()会一直阻塞直到任务完成。

```java
public class SearchProductInfoHelper {
    ExecutorService executor = Executors.newFixedThreadPool(4);
    
	public  ProductInfo get(final String productId) throws InterruptedException, DataLoadExcepton {
        FutureTask<ProductInfo> task = new FutureTask<>(new Callable<ProductInfo>() {
            public ProductInfo call() throws Exception {
                return loadProduceInfo(productId);
            }
		});
        exexutor.execute(task);
        // 阻塞直到task完成
        try {
            return task.get();
        } catch (ExecutionException e) {
            Throwable cause = e.getCause();
            if (cause instanceof DataLoadExcepton) {
                throw (DataLoadExcepton) cause;
            } else {
                // 处理不同原因导致的错误
                throw launderThrowable(cause);
            }
        }
    }
}

private static RuntimeException launderThrowable(Throwable t) {
    if (t instanceof RuntimeException) {
        return (RuntimeException) t;
    } else if (t instanceof Error) {
        throw (Error) t;
    } else {
        throw new IllegalStateException("Not checked", t);
    }
}
```

### 信号量

控制能够同时访问某特定资源的活动的数量。可以用来实现资源池或给一个容器限定边界。（Semaphore）

### 关卡

闭锁是一次性使用对象，一旦到最终状态，就不能被重置了。

关卡（barrier）类似于闭锁，它们能够阻塞一组线程，直到某些事件发生。关卡与闭锁的关键不同在于，必须等待所有线程同时到达关卡才能继续处理。**闭锁等待的是事件**；**关卡等待的是线程**；

## 为计算结果建立高效、可伸缩的缓存

缓存看起来再简单不过。一个天真的高速缓存，即使能改进在单线程环境下的性能，也只不过将性能瓶颈转化为可伸缩的瓶颈。

```java
// 版本1
private Map<K, V> cache = new ConcurrentHashMap<>();
public V compute(K key) {
    V v = cache.get(key);
    if (v == null) {
        v = computeValue(key);
        cache.put(key, v);
    }
    return v;
}
```

![两个线程计算相同的值](/concurrent-programme-5-building-blocks/1561541075143.png)

```java
// 版本2
private Map<K, FutureTask<V>> cache = new ConcurrentHashMap<>();
public V compute(final K key) throws InterruptedException {
    FutureTask<V> f = cache.get(key);
    if (f == null) {
        Callable<V> eval = new Callable<>() {
          	public void call() throw Exception {
                return omputeValue(key);
            }  
        };
        FutureTask<V> ft = new FutureTask<V>();
        f = ft;
        ft.run();
        cache.put(key, ft);
    }
    // 会阻塞直到ft.run执行完成
    return f.get();
}
```

![偶发的时序导致两次计算相同的值](/concurrent-programme-5-building-blocks/1561541164752.png)

```java
// 最终版，高
private Map<K, FutureTask<V>> cache = new ConcurrentHashMap<>();
public V compute(final K key) throws InterruptedException {
    FutureTask<V> f = cache.get(key);
    if (f == null) {
        Callable<V> eval = new Callable<>() {
          	public void call() throw Exception {
                return omputeValue(key);
            }  
        };
        FutureTask<V> ft = new FutureTask<V>();
        f = cache.putIfAbsent(key, ft);
        if (f == null) {
        	f = ft;
        	ft.run();
        }
    }
    // 会阻塞直到ft.run执行完成
    return f.get();
}
```

























