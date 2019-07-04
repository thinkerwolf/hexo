---
title: Java并发编程-应用线程池
date: 2019-07-03 15:34:58
categories:
- Java
tags: 
- Java并发编程
---

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    extensions: ["tex2jax.js"],
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
      <!--$表示行内元素，$$表示块状元素 -->
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
      processEscapes: true
    },
    "HTML-CSS": { availableFonts: ["TeX"] }
  });
</script>
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js">
</script>

## 任务和策略间的隐性耦合

尽管Executer框架为制定和修改执行策略提供了相当大的灵活度，但是并非单个策略适合所有的任务。以下类型的任务需要明确指定一个策略。

**依赖性任务**。大多数运行良好的运行任务是独立的。但是如果提交的任务之间有相互依赖，这就对执行策略带来了约束。

**线程限制的任务**。单线程化的Executor对比任意的线程池，可以对同步作出更强的承诺。

**响应时间敏感的任务**。GUI对应用程序的响应时间是很敏感的，如果将一个用户操作提交到一个单线程化Executor或一个只包含少量线程的线程池中，会削弱Executor的响应性。

**使用ThreadLocal任务**。ThreadLocal会为每个线程保存一个私有的变量。但是Executor框架在需求不高的时候会重用线程，在需求高的是创建新的线程，如果抛出异常就会被全新的工作线程取代。在线程池中，不应该用ThreadLocal去传递任务间数值。

### 线程饥饿死锁

如果一个线程池中的一个任务依赖其他任务的执行，就可能产生死锁。在单一线程的Executor中更易发生。父任务在等待子任务完成，而因为子任务和父任务在同一个线程中执行，子任务同时也在等父任务完成，造成**线程饥饿死锁**（Thread starvation deadlock）。下面就是饥饿死锁的一个例子。

```java
public class ThreadPoolDeadLock {
    ExecutorService exec = Executors.newFixedThreadPool(1);
    public class RenderPageTask implements Callable<String> {
        public String call() throw Exception {
            Future<String> head, foot;
            head = exec.submit(new Callable<String> {
                 public String call() throw Exception { 
                 	return "header";
                 }
            });
            foot = exec.submit(new Callable<String> {
                 public String call() throw Exception { 
                 	return "footer";
                 }
            });
            // 产生死锁，等待子任务完成
            return head.get() + " page " + foot.get(); 
        }
    }
    public Future<String> execute() {
        return exec.submit(new RenderPageTask());
    }
}
```

### 耗时操作

任务时间过长阻塞线程，就算不产生死锁也会影响程序的响应性能。利用库中一些限时阻塞方法可以解决此类问题，比如BlockingQueu.put、poll，Thread.join，CountdownLatch.await都有限时阻塞的版本。

## 线程池大小的选定

定制线程池的大小并不是多精密的活，需要避免就是不要"过大"和“过小”。在一个有N个处理器的系统执行密集型的任务通常使用（N+1）个线程。I/O操作和阻塞性任务就需要更大线程池。

在一个基准负载下，使用几种不同大小的线程池运行应用程序，观察CPU利用水平。

给定以下定义：

$$
N_{cpu} = CPU数量
$$

$$
U_{cpu}=目标CPU使用率
$$

$$
W/C=等待时间与计算时间的比率
$$

为了保持处理器达到期望的使用率，最优的池大小为：
$$
N_{threads}=N_{cpu}\times U_{cpu} \times (1+W/C)
$$

## 配置ThreadPoolExecutor

### 线程的创建和销毁

核心池大小（core pool size），最大池大小（max pool size）和存活时间（keep alive time）共同管理线程池大小。线程池的数量永远保持这核心池数量直到创建到最大池数量都不会创建新的线程。如果提交的任务超过最大任务数量且线程池数量已达最大，任务将会被拒绝。如果一个线程闲置了超过存活时间，会回收此线程，这样通过调节核心池大小和存活时间可以让资源运用在别的地方。

### 管理队列任务

有时会遇到客户端抛给线程池的任务过多，超出了线程池的可以处理的速度，这是也有可能造成资源耗尽的情况，即便没有耗尽内存，也有可能导致程序的响应时间变得糟糕。

ThreadPoolExecutor允许使用一个BlockingQueue去做任务队列。任务队列排列有三种方式：无限队列、优先队列和同步移交三种方式。

newFixedThreadPool和newSingleThreadPool使用的是无限的LinkedBlockingQueue，工作队列会无限的增加。一个稳妥的方式是使用有限队列，当队列满了之后还有相应的饱和拒绝策略处理任务（RejectedExecutionHandler）。

对于庞大的无限的池，可以使用SynchronousQueue，完全绕开队列，将任务直接从生产者移交给工作线程。为了把一个任务放入SynchronousQueue中，必须有另外一个线程正在等待接收移交的任务。如果没有这样的线程，只要线程数不大于最大池数，就会创建一个新的线程，否则就会拒绝任务。只有池是无限的，或者接受的任务可以被拒绝。SynchronousQueue才是一个有价值的选择。

### 饱和策略

当一个有限队列充满后。饱和策略开始起作用。ThreadPoolExecutor的饱和策略可以通过调用setRejectedExecutionHandler去修改。JDK提供了几种不同的RejectedExecutionHandler：中止（默认，AbortPolicy），调用者有限（CallerRunsPolicy），遗弃（DiscardPolicy），遗弃最旧的（DiscardOldestPolicy）。

### 线程工厂

线程池是通过一个线程工程（ThreadFactory）去新建线程，在新建线程池时，可以指定自己的实现的线程工厂，对新建的线程进行个性化定制。比如线程名称，线程优先级等等。

如果需要使用安全策略为某些特定代码授予权限，可以利用Executors中privilegedThreadFactory方法提供的线程工厂。

## 扩展ThreadPoolExecutor

ThreadPoolExecutor的设计是可扩展的，其提供几个“钩子”函数让子类覆写：beforeExecute、afterExecute和terminate。可以利用beforeExecute和afterExecute去添加一些日志等操作。

## 并行递归算法

如果一个循环的每次迭代都是相互独立的，我们不必等到迭代都完成后再一起处理，我们可以使用Executor将一个顺序执行转化为并行执行。比如网页图片的下载和加载。树的深度优先遍历等。