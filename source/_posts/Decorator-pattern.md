---
title: 装饰者模式
date: 2017-11-27 09:01:48
tags: [设计模式]
categories: [设计模式]
---


### 1. 模式动机

一般有两种方式可以实现给一个类或对象增加行为：

* 继承机制，使用继承机制是给现有类添加功能的一种有效途径，通过继承一个现有类可以使得子类在拥有自身方法的同时还拥有父类的方法。

* 关联机制，即将一个类的对象嵌入另一个对象中，由另一个对象来决定是否调用嵌入对象的行为以便扩展自己的行为，我们称这个嵌入的对象为装饰器(Decorator)。

有些时候使用继承无法完成解决问题，比如，类数量爆炸，设计死板，以及基类加入的新功能并不适用于所有的子类。这个时候装饰类就是一个很好的选择。


### 2. 模式定义

**装饰者模式：**动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的替代方案。

![](/images/decor-uml.png)


### 3. 代码分析

* [Component] Beverage

```
public abstract class Beverage {

    String description = "Unknown Beverage";

    public String getDescription() {
        return description;
    }

    public abstract double cost();

}
```


* [ConcreteComponent] Espresso

```
public class Espresso extends Beverage {

    public Espresso() {
        description = "Espresso";
    }

    @Override
    public double cost() {
        return 1.99;
    }
}
```

* [Decorator] CondimentDecorator

```
public abstract class CondimentDecorator extends Beverage {

    public abstract String getDescription();

}
```

* [ConcreteDecorator] Mocha

```
public class Mocha extends CondimentDecorator {

    Beverage beverage;

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + ", Mocha";
    }

    @Override
    public double cost() {
        return 0.20 + beverage.cost();
    }
}
```

* [Client] StarbuzzCoffee

```
public class StarbuzzCoffee {

    public static void main(String args[]) {
        Beverage beverage = new Espresso();
        System.out.println(beverage.getDescription() + ",cost $" + beverage.cost());

        beverage = new Mocha(beverage);
        System.out.println(beverage.getDescription() + ",cost $" + beverage.cost());

    }
}

```

### 4. 真实世界的装饰者： Java I/O

![](/images/java-io-1.png)

![](/images/java-io-2.png)

Java I/O 也引出装饰者一个“缺点”：利用装饰者模式，常常会造成设计中有大量的小类，数量很大。

### 4.1 编写自己的 I/O 装饰者

将输入流中所有的大写字母转成小写字母?

```
public class LowerCaseInputStream extends FilterInputStream {
   
    protected LowerCaseInputStream(InputStream in) {
        super(in);
    }

    @Override
    public int read() throws IOException {
        int c = super.read();
        return c == -1 ? c : Character.toLowerCase(c);
    }

    @Override
    public int read(byte[] b, int off, int len) throws IOException {
        int result = super.read(b, off, len);

        for (int i = off; i < off + result; i++) {
            b[i] = (byte) Character.toLowerCase((char) b[i]);
        }
        return result;
    }
}
```


### 5. 总结

* 继承属于扩展形式之一，但不见得是达到弹性设计的最佳方式。
* 在设计中，应该允许行为可以被扩展，而无须修改现有代码。
* 组合和委托可用于在运行时动态地加上新的行为。
* 除了继承，装饰者模式也可以扩展行为。
* 装饰者模式意味着一群装饰者类，这些类用来包装具体组件。
* 装饰者类反映出被装饰的组件类型。
* 装饰者可以在被装饰者的行为前面与/或后面加上自己的行为，甚至将被装饰者的行为整个取代掉，而达到特定的目的。
* 可以用无数个装饰者包装一个组件。
* 装饰者会导致设计中出现很多小对象，如果过度使用，会让程序变得很复杂。

