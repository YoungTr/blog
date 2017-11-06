---
title: 单例模式-静态内部类实现
date: 2017-11-06 09:07:49
tags: [设计模式]
categories: [设计模式]
---

饿汉式单例类不能实现延迟加载，不管将来用不用始终占据内存；懒汉式单例类线程安全控制烦琐，而且性能受影响。

一种更好的实现方式称之为**initialization-on-demand holder (IoDH，按需初始化持有者)**。在所有版本的Java中，这个习惯用法能够实现安全，高度并发的惰性初始化，并且性能良好。

```
public class Something {
    private Something() {}

    private static class LazyHolder {
        static final Something INSTANCE = new Something();
    }

    public static Something getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```

当JVM加载类 Something 时，Something 类将进行初始化。由于该类没有任何静态变量需要初始化，初始化随即完成。静态类 LazyHolder 只有在类 Something **首次**调用静态方法 getInstance 时 JVM 才会加载并初始化 LazyHolder 类。LazyHolder类初始化时执行外部类Something的（private）构造函数来初始化静态变量INSTANCE。由于类初始化阶段由 JLS 保证是串行的，即非并发的，在加载和初始化期间，在静态getInstance方法中不需要进一步的同步。由于初始化阶段在串行操作中写入静态变量INSTANCE，因此getInstance的所有后续并发调用将返回相同的正确初始化的INSTANCE，而不会产生任何额外的同步开销。

虽然这个实现是一个高效的线程安全的“单例”缓存，没有同步开销，并且比无约束同步更好，但是这个实现方式需要确保构造 Something 的时候不能出错。在大多数JVM实现中，如果Something构造失败，随后尝试从同一个类加载器初始化它将导致NoClassDefFoundError 错误。

### 参考 
1. [Initialization-on-demand holder idiom](https://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom)
2. [确保对象的唯一性——单例模式 （四）](http://blog.csdn.net/lovelion/article/details/7420888)