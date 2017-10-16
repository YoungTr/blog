---
title: 模板方法
date: 2017-10-16 11:40:58
tags: [设计模式]
categories: [设计模式]
---

### 1. 模式动机

当两个类的一些算法、逻辑存在很大的相似性，可以使用模板方法在超类中管理这个算法，改算法只存在于一个地方。超类专注在算法本身，而由子类提供完整的实现。


### 2. 模式定义

#### 2.1 **模板方法模式**在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法的某些步骤。


模板就是一个方法，这个方法定义一组步骤，其中的任何步骤都可以是抽象的，由子类负责实现。这可以确保算法的结构保持不变，同时由子类提供部分实现。

#### 2.2 ** UML图**

![UML](/images/templateuml.png)

### 2.3 代码分析

```
/**
 * 抽象类，作为基类,其子类必须实现其操作
 */
public abstract class CaffeineBeverage {

	/**
	 * 这就是模板方法。它被声明为 final，以免子类改变这个算法的顺序
	 * 模板方法定义了一连串的步骤，每个步骤由一个方法代表
	 */
	final void prepareRecipe() {
		boilWater();
		brew();
		pourInCup();
		addCondiments();
	}

	/**
	 * 原语操作，具体子类必须实现她们
	 */
	abstract void brew();
	abstract void addCondiments();
 
	void boilWater() {
		System.out.println("Boiling water");
	}
  
	void pourInCup() {
		System.out.println("Pouring into cup");
	}
}
```

### 3. 钩子方法 hook

我们可以有“默认不做事的方法”，称这种方法为“hook”（钩子）。子类可以视情况决定要不要覆盖它们。

对模板方法进行挂钩

```
public abstract class CaffeineBeverageWithHook {
 
	void prepareRecipe() {
		boilWater();
		brew();
		pourInCup();
		// 这里加一个用户是否想要调料的判断方法
		if (customerWantsCondiments()) {
			addCondiments();
		}
	}
 
	abstract void brew();
 
	abstract void addCondiments();
 
	void boilWater() {
		System.out.println("Boiling water");
	}
 
	void pourInCup() {
		System.out.println("Pouring into cup");
	}

	/**
	 * 这就是一个钩子，子类可以选择是否覆盖这个方法
	 * 默认返回 true
	 * @return
	 */
	boolean customerWantsCondiments() {
		return true;
	}
}
```

### 4. 模板方法的体现

模板方法模式是一个很常见的模式，比如 Java 数组类就提供了一个方便的模板方法用来排序。

```
Arrays.sort() 
```

```
private static void mergeSort(Object[] src,
                                  Object[] dest,
                                  int low,
                                  int high,
                                  int off) {
        int length = high - low;

        // Insertion sort on smallest arrays
        if (length < INSERTIONSORT_THRESHOLD) {
            for (int i=low; i<high; i++)
                for (int j=i; j>low &&
                         ((Comparable) dest[j-1]).compareTo(dest[j])>0; j--)
                    swap(dest, j, j-1);
            return;
        }
```


mergeSort 就是一个模板方法，包含了排序算法，此算法依赖于 compareto 方法的实现来完成算法。

要实现这一点，设计者利用了 Comparable 接口，必须实现这个接口，提供这个接口所声明的方法，也就是 compareTo() 。


注意，这个 Arrays 类 sort() 方法的设计者受到一些约束。通常我们无法设计一个类继承 Java 数组，而 sort 方法希望能够适用于所有的数组（每个数组都是不同的类）。所以定义了一个静态方法，而由被排序的对象内的每个元素自行提供比较大小的算法部分。这虽然不是教科书上的模板方法，但它的实现任然符合模板方法模式的精神。再者由于不需要继承数组就可以使用这个算法，这样使得排序变得更有弹性、更有用。


### 5. 总结

* “模板方法”定义了算法的步骤，把这些步骤的实现延迟到子类。
* 模板方法模式提供了一种代码复用的重要技巧。
* 模板方法的抽象类可以定义具体方法，抽象方法和钩子。
* 钩子是一种方法，它在抽象类中不做事，或者只做默认的事情，子类可以选择要不要去覆盖它。
* 为了防止子类改变模板方法中的算法，可以将模板方法声明为 final 。