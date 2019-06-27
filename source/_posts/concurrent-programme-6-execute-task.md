---
title: Java并发编程-任务执行
date: 2019-06-27 10:16:47
categories:
- Java
tags: 
- Java并发编程
---

## 线程中执行任务

### 顺序执行任务

```java
public class SingleThreadWebServer {
	public static void main(String[] args) {
        ServerSocket ss = new ServerSocket(80);
        while(true) {
            Socket s = ss.accept();
            handleRequest(s);
        }
    }
} 
```

### 创建线程执行任务

```java
public class MultiThreadWebServer {
	public static void main(String[] args) {
        ServerSocket ss = new ServerSocket(80);
        while(true) {
            Socket s = ss.accept();
            new Thread(new Runnable() {
            	 handleRequest(s);
            }).start();
        }
    }
} 
```

## Executor框架

顺序执行会产生糟糕的响应性和吞吐量，“每任务每线程”会给资源管理器带来麻烦。

Executor接口将任务提交与任务执行实现了解耦。如果要在程序中实现生产者-消费者模式，最好的方法就是使用Executor框架。

### 执行策略

- 任务在什么线程中执行。
- 任务以什么顺序（FIFO，LIFO）执行。
- 多少个任务并发执行。
- 多少个任务在等待队列中。
- 系统过载，应该放弃哪些任务，并如何通知程序知晓。
- 任务失败后的处理。

### 线程池

重用已创建的线程，避免了线程创建、消亡产生的开销。通过创建适当大小的线程池，可以保持计算机CPU足够忙碌。还可以防止过多线程之间资源的相互竞争，导致应用内存耗尽或失败。

### Executor生命周期

ExecutorService接口扩展了Executor接口，增加关闭终止等操作。

```java
public interface ExecutorService extends Executor {
    void shutdown();
    List<Runnable> shutdownNow();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

```

Executor有运行、关闭和终止三种状态。Executor刚创建时是运行状态，调用shutdown方法先进入关闭状态，等待所有任务执行完进入终止状态。调用shutdownNow直接进入终止状态。

## 寻找可强化的并行性

合理的任务边界优势并不容易区分，可能一个请求进入之后开可以进一步细分成不同的部分。下面是使用并行方式渲染网页的文字和图片。

```java
public class Render {
	private ExecutorService executor = Executors.newFixedThreadPool(4);
	public void renderPage(CharSequence source) {
		final List<ImageInfo> infos = scanImageInfo(source);
		Callable<List<ImageData>> task = new Callable<>() {
			public List<ImageData> call() throws Exception {
				List<ImageData> ids = new ArrayList<>();
                  for (ImageInfo ii : infos) {
                      ids.add(downloadImage(ii));
                  }
			}
		};
         Future<List<ImageData>> f = executor.submit(task);
        renderText(source);
        try {
            List<ImageData> ids = f.get();
            renderImages(ids);
        } catch (InterruptedException e) {
    
        } catch (ExecutionException e) {
            
        }    
	}
}
```



### 并行运行异类任务的局限性

比如网页渲染，将**渲染文本**和**下载图片**的两个任务并行执行，如果渲染文本的速度远远快于下载图片的速度，那么最后的速度与顺序执行没什么区别。因此试图通过并行运行异类任务还需要很多额外的工作。

```wiki
大量相互独立且同类的任务并行运行，会将任务分配大不同的线程中，这样才能获得性能的提升。
```

### CompletionService：Executor遇到BlockingQueue

```java
public class Render {
	private ExecutorService executor = Executors.newFixedThreadPool(4);
	public void renderPage(CharSequence source) {
		final List<ImageInfo> infos = scanImageInfo(source);
         CompletionService<ImageData> imageExe = new CompletionService<>(executor);
		for (ImageInfo info : infos) {
             Callable<ImageData> task = new Callable<>() {
                 public ImageData call() throws Exception {
                 	return downloadImage(info);
                 }
             };
            imageExe.submit(task);
        }
        renderText(source);
        for (int t = 0; t < infos.size(); t++) {
            Future<ImageData> f = imageExe.take();
            renderImage(f.get());
        }
    }
}
```





















