---
title: 单例模式
date: 2017-10-24 10:25:12
tags: [设计模式]
categories: [设计模式]
---

### 1. 模式动机

有一些对象其实我们只需要一个，比方说：线程池（threadpool）、缓存(cache)、日志对象、驱动程序等对象。

### 2. 模式定义

**单例模式**确保一个类只有一个实例，并提供一个全局访问点。

### 3. 模式分析

**MUL**

![singleton-uml](/images/singleton-uml.png)

**代码分析**

**预先加载创建实例**
 
```
public class Singleton {
	//在静态初始化器（static initializer）创建单例
	private static Singleton uniqueInstance = new Singleton();
 
	private Singleton() {}
 
	public static Singleton getInstance() {
		return uniqueInstance;
	}
}
```

JVM 在加载这个类时马上创建此唯一的单例实例。JVM 保证在任何线程访问 uniqueInstance 静态变量之前，一定先创建此实例。

**用“双重检查加锁，延时加载”**


```
public class Singleton {
	private volatile static Singleton uniqueInstance;
 
	private Singleton() {}
 
	public static Singleton getInstance() {
		if (uniqueInstance == null) {
			synchronized (Singleton.class) {
				if (uniqueInstance == null) {
					uniqueInstance = new Singleton();
				}
			}
		}
		return uniqueInstance;
	}
}
```

### 4. 总结

* 单例模式确保程序中一个类最多只有一个实例，并提供访问这个实例的全局点。
* 在 Java 中实现单例模式需要私有构造器，一个静态方法和一个静态变量。
* 注意多线程的问题。
* 使用多个类加载器，可能导致单例失效而产生多个实例。
