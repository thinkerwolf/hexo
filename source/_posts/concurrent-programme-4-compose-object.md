---
title: Java并发编程-组合对象
date: 2019-06-24 14:00:03
categories:
- Java
tags: 
- Java并发编程
---

## 实例限制

就是将一个对象封装在另一个对象里面，访问被封装代码的路径都是可知，实例限制于各种锁相结合可以确保已线程安全的方式访问被封装的对象。同时要确保被封装的对象不会溢出到不可控的代码中。

限制对象方式有固定到单个线程中、使用线程本地变量、私有域中等等。

```java
// 私有域实例限制
@ThreadSafe
public class PersonSet {
	private final Set<Person> pset = new HashSet<>();
	public sychronized Person addPerson(Person p) {
		pset.add(p);
	}
	public sychronized boolean containPerson(Person p) {
		return pset.contains(p); 
	}
}
```

### 监视器模式

使用私有对象锁，而不是对象内部锁，可以有很多好处。私有对象锁外部程序无法获取，程序代码足够简单。

```java
public class PrivateLock {
	private final Object lock = new Object();
	void doSomethine() {
		sychronized(lock) {
			// ...
		}
	}
}
```

## 委托线程安全

几乎所有的对象都是组合对象，当凭空构建一个类，或者使用非线程安全对象组装一个类时，Java监视器很有用。但是对象的组件都是线程安全的，这样的对象是线程安全的吗？答案是依据情况而定。下面是Java监视器和委托模式下线程安全对比。

```java
// 简单的汽车追踪器类
@ThreadSafe
public class MonitorVehicleTracker {
	private final Map<String, MutablePoint> locations;
	public MonitorVehicleTracker(Map<String, MutablePoint> locations) {
		this.locations = deepCo;
	}
	public sychronized Map<String, MutablePoint> getLocations() {
		return deepCopy(locations);
	}
	public sychronized void setLocation(String id, int x, int y) {
		MutablePoint p = locations.get(id);
		if (p == null) {
			return;
		}
		p.x = x;
		p.y = y;
	}
	public sychronized MutablePoint getLocation(String id) {
		return locations.get(id);
	}
	private Map<String, MutablePoint> deepCopy() {
		return Collections.unmodifiableMap(new HashMap<>(locations));
	}
}

public class MutablePoint {
	int x, y;
	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}
} 
```

```java
// 委托模式，将线程安全委托给ConcurrentHashMap
@ThreadSafe
public class DelegatingVehicleTracker {
	private final Map<String, Point> locations;
	private final Map<String, Point> unmodifiableLocations;
	public DelegatingVehicleTracker(Map<String, Point> points) {
		this.locations = new ConcurrentHashMap<>(points);
		this.unmodifiableLocations = Collections.unmodifiableMap(locations);
	}
	public Map<String, MutablePoint> getLocations() {
		return unmodifiableLocations;
	}
	public sychronized void setLocation(String id, int x, int y) {
		locations.replace(id, new Pont(x, y));
	}
	public sychronized MutablePoint getLocation(String id) {
		return locations.get(id);
	}
}
public class Point{
	private final int x, y;
	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}
}
```

### 非状态依赖变量

```java
@ThreadSafe
public class VisualComponent {
	private final List<KeyListener> keyListeners = new CopyOnWriteArrayList<>();
	private final List<MouseListener> mouseListeners = new CopyOnWriteArrayList<>();
	public void addKeyListener(KeyListener listener) {
		keyListeners.add(listener);
	}
	public void removeKeyListener(KeyListener listener) {
		keyListeners.remove(listener);
	}
	public void addMouseListener(MouseListener listener) {
		mouseListeners.add(listener);
	}
	public void removeMouseListener(MouseListener listener) {
		mouseListeners.remove(listener);
	}
}
```

VisualComponent中两个域都是线程安全，且两者之间没有互不影响，所以是线程安全的。

### 委托无法胜任

```java
@NoThreadSafe
public class NumberRange {
	// 不变约束 lower <= upper
	private final AtomicInteger lower = new AtomicInteger();
	private final AtomicInteger upper = new AtomicInteger();
	public void setLow(int i) {
		// 不安全的检查在运行
		if (i > upper.get()) {
			throw new RuntimeException("");
		}
		lower.set(i);
	}
	public void setUp(int low) {
		// 不安全的检查在运行
		if (i < lower.get()) {
			throw new RuntimeException("");
		}
		upper.set(low);
	}
}
```

NumberRange不是线程安全的，它没有保护好约束lower和upper的不变约束。虽然lower和upper虽然是线程安全的，但是检查再运行的操作破坏了这种约束。非常类似于volatile的描述规则。

### 发布底层状态变量

如果一个状态变量是线程安全的，没有任何不变约束限制它的值，并且没有任何状态转换限制它的操作，那么它可以被安全发布。

## 向已有的线程安全类添加功能

### 客户端枷锁

```java
@NotThreadSafe
public class ListHelper<E> {
	public List<E> list = Collections.sychronizedList(new ArrayList<E>());
	// 与list内部操作使用的不是同一把锁
    public sychronized boolean putIfAbsent(E element) {
		boolean c = list.contains(element);
		if (!c) {
			list.add(element);
		}
		return !c;
	}
}
```

这个类不是线程安全的，就是因为list和ListHelper使用的锁是不同的，虽然putIfAbsent方法是同步的，但是list的其他操作使用的锁与这个不同，所以依然无法保证线程安全。

```java
// 正确的客户端加锁
@ThreadSafe
public class ListHelper<E> {
	public List<E> list = Collections.sychronizedList(new ArrayList<E>());
	public boolean putIfAbsent(E element) {
		// 使用list本身的锁进行同步
        sychronized (list) {
            boolean c = list.contains(element);
            if (!c) {
                list.add(element);
            }
            return !c;
		}
	}
}
```

### 组合

```java
// 使用组合实现
public class ImprovedList<E> {
    private final List<E> list;
    public class ImprovedList(List<E> list) {
        this.list = list;
    }
    public sychronized boolean putIfAbsent(E element) {
            boolean c = list.contains(element);
            if (!c) {
                list.add(element);
            }
            return !c;
	}
    public sychronized void clear() {
        list.clear();
    }
}
```

ImprovedList使用Java监视器模式封装了List，使用自身的锁同步机制，不同关心传入的List内部同步与否。虽然可能在性能上有微弱的损失，但是在代码的健壮性上相比客户端加锁更加健壮。



