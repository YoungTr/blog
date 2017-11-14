---
title: Java的引用-StrongReference、WeakReference、SoftRefrence
date: 2017-11-07 09:16:01
tags: [Java]
categories: [转载, Java]
---


本文转载自 [http://blog.csdn.net/mxbhxx/article/details/9111711](http://blog.csdn.net/mxbhxx/article/details/9111711)

### 1. StrongReference

StrongReference 是 Java 的默认引用实现,  它会尽可能长时间的存活于 JVM 内， 当没有任何对象指向它时 GC 执行后将会被回收。

```
@Test  
public void strongReference() {  
    Object referent = new Object();  
      
    /** 
     * 通过赋值创建 StrongReference  
     */  
    Object strongReference = referent;  
      
    assertSame(referent, strongReference);  
      
    referent = null;  
    System.gc();  
      
    /** 
     * StrongReference 在 GC 后不会被回收 
     */  
    assertNotNull(strongReference);  
} 
```


### 2. WeakReference & WeakHashMap

WeakReference，是一个弱引用,  当所引用的对象在 JVM 内不再有强引用时, GC 后 weak reference 将会被自动回收 


```
public void weakReference() {  
    Object referent = new Object();  
    WeakReference<Object> weakRerference = new WeakReference<Object>(referent);  
  
    assertSame(referent, weakRerference.get());  
      
    referent = null;  
    System.gc();  
      
    /** 
     * 一旦没有指向 referent 的强引用, weak reference 在 GC 后会被自动回收 
     */  
    assertNull(weakRerference.get());  
} 
```

WeakHashMap 使用 WeakReference 作为 key， 一旦没有指向 key 的强引用, WeakHashMap 在 GC 后将自动删除相关的 entry 

```
public void weakHashMap() throws InterruptedException {  
    Map<Object, Object> weakHashMap = new WeakHashMap<Object, Object>();  
    Object key = new Object();  
    Object value = new Object();  
    weakHashMap.put(key, value);  
  
    assertTrue(weakHashMap.containsValue(value));  
      
    key = null;  
    System.gc();  
      
    /** 
     * 等待无效 entries 进入 ReferenceQueue 以便下一次调用 getTable 时被清理 
     */  
    Thread.sleep(1000);  
      
    /** 
     * 一旦没有指向 key 的强引用, WeakHashMap 在 GC 后将自动删除相关的 entry 
     */  
    assertFalse(weakHashMap.containsValue(value));  
}  
```

### 3. SoftReference 

SoftReference 于 WeakReference 的特性基本一致， 最大的区别在于 SoftReference 会尽可能长的保留引用直到 JVM 内存不足时才会被回收(虚拟机保证), 这一特性使得 SoftReference 非常适合缓存应用。

```
public void softReference() {  
    Object referent = new Object();  
    SoftReference<Object> softRerference = new SoftReference<Object>(referent);  
  
    assertNotNull(softRerference.get());  
      
    referent = null;  
    System.gc();  
      
    /** 
     *  soft references 只有在 jvm OutOfMemory 之前才会被回收, 所以它非常适合缓存应用 
     */  
    assertNotNull(softRerference.get());  
}
```

**Stack Voerflow 相关回答**

[What's the difference between SoftReference and WeakReference in Java?](https://stackoverflow.com/questions/299659/whats-the-difference-between-softreference-and-weakreference-in-java#)

