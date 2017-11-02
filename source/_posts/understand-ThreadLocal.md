---
title: 理解 ThreadLocal
date: 2017-11-01 17:23:39
tags: [Java, 线程]
categories: [Java]
---

Android 中，在 Hanlder 消息机制中可以看到 ThreadLocal 的身影，ThreadLocal并不是线程,通过 ThreadLocal 可以在不同线程中存储数据，对其他线程来说是无法获取的。

### 1. 简单示例

在不同线程中，打印 ThreadLocal 设置的线程的名字：

```
    private ThreadLocal<String> mStringThreadLocal = new ThreadLocal<>();


        mStringThreadLocal.set(Thread.currentThread().getName());
        Log.d(TAG, "[Thread#main]mStringThreadLocal.get()" + mStringThreadLocal.get());

        new Thread("Thread#1") {
            @Override
            public void run() {
                mStringThreadLocal.set(Thread.currentThread().getName());
                Log.d(TAG, "[Thread#1]mStringThreadLocal=" + mStringThreadLocal.get());
            }
        }.start();

        new Thread("Thread#2") {
            @Override
            public void run() {
                Log.d(TAG, "[Thread#2]mStringThreadLocal=" + mStringThreadLocal.get());
            }
        }.start();
    }
```

日志输出：

```
D/ThreadLocal: [Thread#main]mStringThreadLocal.get()main
D/ThreadLocal: [Thread#1]mStringThreadLocal=Thread#1
D/ThreadLocal: [Thread#2]mStringThreadLocal=null
```

可以看出，在 main 和 Thread1 中设置了当前线程的名字，分别打印了 main 和 Thread#1 ，刚好是设置的名字；而 Thread#2 线程没有设置值，则打印了 null 。

### 2. 源码分析

ThreadLocal 是一个泛型类，主要看一下 set 和 get 方法：

```
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

首先会获取当前线程 t，根据 t 对象获取一个 ThreadLocalMap 对象，如果 map 对象不为空，就将当前线程对象作为 key，当 value 存储在该 map 对象中；如果 map 对象为空，则执行 createMap 方法。

```
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```

getMap 方法很简单，返回 Thread 对象的 threaLocals 成员变量，该变量是 ThreadLocalMap 的一个实例。


```
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

createMap 方法则是为当前 Thread 对象的 threadLocals 变量赋值。

看一下 ThreadLocalMap：

```
static class ThreadLocalMap{
        private Entry[] table;

        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }

        private void set(ThreadLocal<?> key, Object value) {

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }

        private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }
}
```

ThreadLocalMap 是 ThreadLocal 的静态内部类，内部有一个 Entry 的数组变量，主要用于存储设置的值，可以看出 value 存储的位置是由 threadLocalHashCode 的值确定的。

Entry 比较简单，是 ThreadLocalMap 的静态内部类，其继承了 WeakReference :

```
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```

再看一下 ThreadLocal 的 get 方法：

```
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

可以看出如果 ThreadLocalMap 不为空，则调用 ThreadLocalMap 的 getEntry 方法，相应会返回 Entry 的 value。如果这个对象为null那么就返回初始值，初始值由ThreadLocal的initialValue方法来描述，默认情况下为null，也可以重写该方法：

```
    protected T initialValue() {
        return null;
    }
```

### 总结

简单总结一下：

1. 每一个线程（Thread）都有一个 ThreadLocalMap 成员变量，可以将 ThreadLocal 中设置的值存储在相应的线程中，每个线程各自管理自己的 ThreadLocalMap，线程可以正确访问到自己存储的对象。
2. 也可以自己定义一个静态的map，将当前thread作为key，将 value put到map中，应该也行，但事实上，ThreadLocal的实现刚好相反，它是在每个线程中有一个map，而将ThreadLocal实例作为key，这样每个map中的项数很少，而且当线程销毁时相应的东西也一起销毁了，速度也提升了很多。
3. ThreadLocal的应用场合，最适合的是按线程多实例（每个线程对应一个实例）的对象的访问，并且这个对象很多地方都要用到。


### 参考

[正确理解ThreadLocal](http://www.iteye.com/topic/103804 "正确理解ThreadLocal") 

[Android的消息机制之ThreadLocal的工作原理](http://blog.csdn.net/singwhatiwanna/article/details/48350919 "Android的消息机制之ThreadLocal的工作原理")








