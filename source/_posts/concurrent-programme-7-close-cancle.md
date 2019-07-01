---
title: Java并发编程-关闭与取消
date: 2019-06-27 16:12:13
categories:
- Java
tags: 
- Java并发编程
---

## 任务取消

在Java中，没有哪一种方式用来停止线程是绝对安全的。只有通过相互协作的机制，使代码和任务遵循一个统一的协议，用来进行任务取消。

```java
@ThreadSafe
public class BrokenPrimeGenerator implements Runnable {
    private List<BigInteger> primes = new ArrayList<>();
    private volitile boolean cancel = false;
    @Override
    public void run() {
        BigInteger p = BigInteger.ONE;
        while (!cancel) {
            p = p.nextProablePrime();
            sychronized (primes) {
                // 如果primes.add()是个阻塞的函数
                // 就算将cancle设置true，此任务也可能取消不了。
                primes.add(p);
            }
        }
    }
    public void setCancel(boolean can) {
        this.cancel = can; 
    }
}
```

### 中断

每一个线程都有一个boolean类型的中断状态；在中断的时候，这个状态设置为true。**interrupt**方法中断目标线程，并且**isInterrupted**返回目标线程的中断状态。静态**interrupted**方法仅仅能够**清除**当前线程的中断状态（需要谨慎使用，因为调用后会清除中断状态）；**这是清除中断状态的唯一方法**。

```wiki
在API和语言规范中，并没有把中断和任何取消语义绑定起来。使用中断来处理取消之外的任何事情都是不明智的，很难支撑起大型应用。
```

阻塞库函数Object.wait()和Thread.sleep()函数，试图检测线程何时中断，并提前结束返回。**它们对中断的响应是，清除中断状态，并抛出InterruptedException异常**。

```wiki
调用interrupt不意味着必然停止目标线程正在运行的工作；
它仅传递请求中断的消息。
```

有一些方法对这样的请求很重视，比如wait、sleep、join方法，当它们接受到中断请求会抛出一个异常，或者调用时中断状态已经被设置了。**所以在良好的方法中应该能够完全忽略这样的请求，只要在合适的位置调用中断方法，其他的留给调用代码去处理**。

```java
@ThreadSafe
class PrimeGenerator extend Thread {
	private BlockQueue<BigInteger> primes;
    public PrimeGenerator(BlockQueue<BigInteger> primes) {
        this.primes = primes;
    }
    @Override
    public void run() {
        try {
            while (!Thread.currentThread().isInterrupted()) {
                BigInteger p = BigInteger.ONE;
                p = p.nextProablePrime();
                // put方法会阻塞
                primes.put(p);
            }
        } catch (InterruptException e) {
            // 允许退出线程....
        }
    }
    public void cancel() {
        this.interrupt();
    }
}
```

```wiki
中断通常是取消任务最好的方法。
```

### 中断策略

区分**任务**和**线程**对中断很重要，一个单一的中断请求可能有一个或者一个以上的接收者。在线程池中中断一个线程，意味着取消当前任务，并关闭当前线程。

任务不会拥有自己的线程，而是租借线程运行。代码并不是线程的所有者，这时中断线程就需要格外小心保存中断状态。大多数库函数只是抛出InterruptedException作为中断响应，它们绝不可能自己独占一个线程，所以大多数线程库比如Executor都实现了自己的取消策略。

**注意保存线程的中断状态**，如果对中断不仅仅是把InterruptedException传递给调用者，还应该在处理InterruptedException后调用Thread.currentThread().interrupt()保存中断状态。

```wiki
因为每个线程都有自己中断策略，所以不能自己去中断（Interrupt）线程，除非知道线程对程序意味着什么。
```

### 响应中断

两种响应中断使用策略：

- 传递异常，使你的方法也是可中断的阻塞方法。
- 保存中断状态，上层调用栈的代码进行处理。

```java
// 传递异常
private BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>();
public Task getNextTask() throws InterruptedException {
	return taskQueue.take();
}
```

大多数代码不知道自己在哪个线程中运行，所以需要将线程的中断状态保存下来，并且可以进一步说明中断的线程。

```java
// 保存中断状态
private BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>();
public Task getNextTask()  {
	boolean interrupted = false;
	try {
        // 不可取消的任务保存中断状态。
		while (true) {
			try {
				return taskQueue.task();
			} catch (InterruptedException e) {
				interrupted = true;
			}
		}
	} finally {
		if (interrupted) {
			Thread.currentThread().interrupt();
		}
	}

}
```

### 通过Future取消任务

```java
public static void timedRun(Runnable r, long timeout, TimeUnit unit) throws InterruptedException {
	Future<?> f = taskExe.submit(r);
     try {
         f.get(timeout, unit);
     } catch (TimeoutException e) {
         // 下面的任务会被取消
     } catch (ExecutionException e) {
         // 执行错误，抛出
         throw launcherException(e);
     } finally {
         f.cancel(true);
     }
}
```



### 处理不可中断阻塞

很多阻塞方法通过提前返回和抛出InterruptedException来实现对中断的响应。但是并不是所有的阻塞方法都响应中断，对于这些方法可以使用与中断类似的手段，来停止这些线程。

**java.io的同步Socket I/O**。InputStream和OutputStream都不响应中断，通过关闭底层Socket，可以让read和write方法抛出SocketException。

```java
ServerSocket ss;
try {
    ss = new ServerSocket(8888);
} catch (Exception e) {
    throw new RuntimeException(e);
}
for (;;) {
    Socket s;
    try {
        s = ss.accept();
    } catch (Exception e) {
        continue;
    }
    Runnable task = new Runnable() {
      	@Override
        public void run() {
            BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            try {
                handleInput(br);
            } catch (SocketException e) {
                // 处理中断
                Thread.currentThread().interrupt();
            }
        }  
    };
    new Thread(task).start();
    // 通过关闭Socket起到发送线程中断信号
    socket.close();
}
```

**java.nio的同步I/O**。中断一个等待InterruptibleChannel的线程，会导致抛出CloseByInterruptException，并关闭链路。关闭一个InterruptibleChannel也会导致多个阻塞链路操作上的线程抛出AsychronousCloseException。

**Selector的异步I/O**。如果一个线程阻塞与Select.select()方法，close方法会导致它抛出ClosedSelectorException提前返回。

**获得锁。**一个线程在等待内部锁。显示Lock提供了lockInterruptibly方法，允许等待一个锁，并仍然响应中断。

```java
public lockInterruptibly() throws InterruptedException;
```

### 使用newTaskFor封装取消任务

```java
public interface CancelableCallable<T> extends Callable<T> {
    void cancel();
    RunnaableFuture<T> newTask();
}
public class CancelableExecutor extends ThreadPoolExecutor {
	//...
	protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        if (callable instanceof CancelableCallable) {
            return ((CancelableCallable) callable).newTask();
        } else {
            return super.newTaskFor(callable);
        }
    }
}
```



## 停止基于线程的服务





































