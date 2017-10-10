---
title: 适配器模式
date: 2017-10-09 09:19:00
tags: [设计模式]
categories: [设计模式]
---


### 1. 模式动机

假如你需要在欧洲国家使用美国制造的笔记本电脑，你可能需要使用一个交流电的适配器。它的作用就是将欧式插座转换成美式插座，好让美式插头可以插进这个插座得到电力——将一个接口转换成另一个接口，以符合客户的需求。

### 2. 模式解析

![模式解析](/images/adpter-pattern-relative.png)

客户使用适配器的过程如下：

1. 客户通过目标接口调用适配器的方法对适配器发出请求。
2. 适配器使用被适配者接口把请求转换成被适配者的一个或多个调用接口。
3. 客户接收到调用的结果，但并未察觉这一切是适配器在起转换作用。（客户和被适配者是解耦的，一个不知道另一个。）

### 3. 模式定义

**适配器模式**将一个类的接口，转换成客户期望的另一个接口。适配器让原来接口不兼容的类可以合作无间。

### 4. 模式结构

#### 4.1 适配器模式包括如下角色：

* Target：期待得到的接口
* Adapee：需要适配的接口
* Adapter：适配器类，核心
* Client：客户

#### 4.2 UML 图

![UML](/images/adapter-pattern-uml.png)

良好的OO设计原则：使用对象组合，以修改的接口包装被适配者，这样被适配者的任何子类，都可以搭配着适配器使用。

#### 4.3 代码分析

* 鸭子接口

```
public interface Duck {
    void quack();

    void fly();

}
```

* 绿头鸭是鸭子的子类

```
public class MallardDuck implements Duck {
    @Override
    public void quack() {
        Log.d(LOG_TAG, "Quack");
    }

    @Override
    public void fly() {
        Log.d(LOG_TAG, "I'm flying");
    }
}
```

* Turkey 接口

```
public interface Turkey {
    void gobble();

    void fly();
}
```

* Turkey 接口的一个具体实现

```
public class WildTurkey implements Turkey {
    @Override
    public void gobble() {
        Log.d(LOG_TAG, "Gobble gobble");
    }

    @Override
    public void fly() {
        Log.d(LOG_TAG, "I'm flying a short distance");
    }
}
```

* TurkeyAdapter

```
public class TurkeyAdapter implements Duck {

    private Turkey turkey;

    public TurkeyAdapter(Turkey turkey) {
        this.turkey = turkey;
    }

    @Override
    public void quack() {
        turkey.gobble();
    }

    @Override
    public void fly() {
        turkey.fly();
        turkey.fly();
    }
}
```

* 客户端

```
   WildTurkey wildTurkey = new WildTurkey();
        wildTurkey.gobble();
        wildTurkey.fly();
        Duck turkeyAdapter = new TurkeyAdapter(wildTurkey);
        turkeyAdapter.quack();
        turkeyAdapter.fly();
```

* 输出

```
The turkey say...
Gobble gobble
I'm flying a short distance
The duck say...
Gobble gobble
I'm flying a short distance
I'm flying a short distance

```

### 5. 总结

1. 当需要使用一个现有的类而其接口并不符合你的需要时，就使用适配器。
2. 适配器改变接口以符合客户的期望。
3. 适配器模式有两种形式：对象适配器和类适配器。类适配器需要用到多重继承。
4. 适配器将一个对象包装起来以改变其接口。