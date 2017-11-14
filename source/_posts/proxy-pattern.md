---
title: 代理模式
date: 2017-10-17 09:21:16
tags: [设计模式]
categories: [设计模式]
---

### 1. 模式动机

在某些情况下，某个客户不想或者不能直接引用某一个对象，比如它们在不同的Java虚拟机（JVM）堆中（在不同的地址空间运行），就可以通过一个称之为“代理”的第三者来实现间接引用。

### 2. 模式定义


**代理模式**为另一个对象提供一个替身或占位符以控制对这个对象的访问。

代理控制访问？

* 远程代理控制访问远程对象。
* 虚拟代理控制访问创建开销大的资源。
* 保护代理基于权限控制对资源的访问。


### 3. 模式分析

**UML**

![](/images/proxy-uml.jpg)

**远程代理 Remote**

![远程代理](/images/remote-proxy.png)

**虚拟代理 Virtual Proxy**

![虚拟代理](/images/virtual-proxy.png)


```
import java.util.*;
 
interface Image {
    public void displayImage();
}

//on System A 
class RealImage implements Image {
    private String filename;
    public RealImage(String filename) { 
        this.filename = filename;
        loadImageFromDisk();
    }

    private void loadImageFromDisk() {
        System.out.println("Loading   " + filename);
    }

    public void displayImage() { 
        System.out.println("Displaying " + filename); 
    }
}

//on System B 
class ProxyImage implements Image {
    private String filename;
    private Image image;
 
    public ProxyImage(String filename) { 
        this.filename = filename; 
    }
    public void displayImage() {
        if(image == null)
              image = new RealImage(filename);
        image.displayImage();
    }
}
 
class ProxyExample {
    public static void main(String[] args) {
        Image image1 = new ProxyImage("HiRes_10MB_Photo1");
        Image image2 = new ProxyImage("HiRes_10MB_Photo2");     
        
        image1.displayImage(); // loading necessary
        image2.displayImage(); // loading necessary
    }
}
```

程序的输出为：

```
Loading    HiRes_10MB_Photo1
Displaying HiRes_10MB_Photo1
Loading    HiRes_10MB_Photo2
Displaying HiRes_10MB_Photo2
```



**动态代理**

Java 在 java.lang.reflect 包中有自己的代理支持，利用这个包可以在运行时动态地创建一个代理类，实现一个或者多个接口，并将方法的调用转发到所指定的类。因为实际的代理类是在运行时创建的，称这个Java技术为：动态代理。

![动态代理 UML](/images/dynamic-proxy.png)


**PersonBean 接口**

```
public interface PersonBean {
    String getName();

    String getGender();

    String getInterests();

    int getHotOrNotRating();

    void setName(String name);

    void setGender(String gender);

    void setInterests(String interests);

    void setHotOrNotRating(int rating);

}
```

**PersonBeanImpl 实现类**

```
public class PersonBeanImpl implements PersonBean {

    String name;
    String gender;
    String interests;
    int rating;
    int ratingCount;

    @Override
    public String getName() {
        return name;
    }

    @Override
    public String getGender() {
        return gender;
    }

    @Override
    public String getInterests() {
        return interests;
    }

    @Override
    public int getHotOrNotRating() {
        if (ratingCount == 0) {
            return 0;
        }

        return rating / ratingCount;
    }

    @Override
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public void setGender(String gender) {
        this.gender = gender;
    }

    @Override
    public void setInterests(String interests) {
        this.interests = interests;
    }

    @Override
    public void setHotOrNotRating(int rating) {
        this.rating += rating;
        this.ratingCount++;
    }
}
```


**保护代理**

这个例子中，根据定义 PersonBean 的方式，任何客户都可以调用任何方法，任何人都可以篡改被人的数据，这显然是不合理的。

什么是保护代理？这是一种根据访问权限决定客户可否访问对象的代理。


**为 PersonBean 创建动态代理**

这里，顾客不可以改变自己的 HotOrNot 评分，也不可以改变其他顾客的个人信息。

要修正这些问题，必须要创建两个代理：一个访问自己的 PersonBean 对象，另一个访问另一个顾客的 PersonBean 对象。这样，代理就可以控制在每一种情况下允许哪一种请求。

Java API的动态代理会为我们创建两个代理，我们只需要提供Handler来处理代理转来的方法。

**步骤一：创建InvocationHandler**

这里创建两个 InvocatonHandler(调用处理器)，其中一个给拥有者使用，另一个给非拥有者使用。*InvocatonHandler：*当代理的方法被调用时，代理会把这个调用转发给 InvocatonHandler,InvocatonHandler 会调用 invoke 方法。

**OwnerInvocationHandler**

```
public class OwnerInvocationHandler implements InvocationHandler {

    PersonBean personBean;

    public OwnerInvocationHandler(PersonBean personBean) {
        this.personBean = personBean;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        try {
            if (method.getName().startsWith("get")) {
                return method.invoke(personBean, args);
            } else if (method.getName().equals("setHotOrNotRating")) {
                throw new IllegalAccessException();
            } else if (method.getName().startsWith("set")) {
                return method.invoke(personBean, args);
            }
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

        return null;
    }
}
```

**步骤二：创建Proxy类并实例化Proxy对象**

```
	PersonBean getOwnerProxy(PersonBean person) {
 		
        return (PersonBean) Proxy.newProxyInstance( 
            	person.getClass().getClassLoader(),
            	person.getClass().getInterfaces(),
                new OwnerInvocationHandler(person));
	}
```


**测试**


```

public class MatchMakingTestDrive {
    Hashtable datingDB = new Hashtable();

    public static void main(String[] args) {
        MatchMakingTestDrive test = new MatchMakingTestDrive();
        test.drive();
    }

    public MatchMakingTestDrive() {
        initializeDatabase();
    }

    public void drive() {
        PersonBean joe = getPersonFromDatabase("Joe Javabean");
        PersonBean ownerProxy = getOwnerProxy(joe);
        System.out.println("Name is " + ownerProxy.getName());
        ownerProxy.setInterests("bowling, Go");
        System.out.println("Interests set from owner proxy");
        try {
            ownerProxy.setHotOrNotRating(10);  //这样应该是行不通的！！！
        } catch (Exception e) {
            System.out.println("Can't set rating from owner proxy");
        }
        System.out.println("Rating is " + ownerProxy.getHotOrNotRating());

        PersonBean nonOwnerProxy = getNonOwnerProxy(joe);
        System.out.println("Name is " + nonOwnerProxy.getName());
        try {
            nonOwnerProxy.setInterests("bowling, Go"); //这样应该是行不通的！！！
        } catch (Exception e) {
            System.out.println("Can't set interests from non owner proxy");
        }
        nonOwnerProxy.setHotOrNotRating(3);
        System.out.println("Rating set from non owner proxy");
        System.out.println("Rating is " + nonOwnerProxy.getHotOrNotRating());
    }

    PersonBean getOwnerProxy(PersonBean person) {

        return (PersonBean) Proxy.newProxyInstance(
                person.getClass().getClassLoader(),
                person.getClass().getInterfaces(),
                new OwnerInvocationHandler(person));
    }

    PersonBean getNonOwnerProxy(PersonBean person) {

        return (PersonBean) Proxy.newProxyInstance(
                person.getClass().getClassLoader(),
                person.getClass().getInterfaces(),
                new NonOwnerInvocationHandler(person));
    }

    PersonBean getPersonFromDatabase(String name) {
        return (PersonBean) datingDB.get(name);
    }

    void initializeDatabase() {
        PersonBean joe = new PersonBeanImpl();
        joe.setName("Joe Javabean");
        joe.setInterests("cars, computers, music");
        joe.setHotOrNotRating(7);
        datingDB.put(joe.getName(), joe);

        PersonBean kelly = new PersonBeanImpl();
        kelly.setName("Kelly Klosure");
        kelly.setInterests("ebay, movies, music");
        kelly.setHotOrNotRating(6);
        datingDB.put(kelly.getName(), kelly);
    }
}
```

**代码输出：**

```
Name is Joe Javabean
Interests set from owner proxy
Can't set rating from owner proxy
Rating is 7
Name is Joe Javabean
Can't set interests from non owner proxy
Rating set from non owner proxy
Rating is 5
```

### 4. 总结

* 代理模式为另一个对象提供代表，以便控制客户对对象的访问，管理访问的方式有许多种。
* 远程代理管理客户和远程对象之间的交互。
* 虚拟代理控制访问实例化开销大的对象。
* 保护代理基于调用者控制对象方法的访问。
* 代理模式有许多变体，例如：缓存代理，同步代理，防火墙代理和写入时复制代理。
* 代理在结构上类似装饰着，但是目的不同。
* 装饰着模式为对象加上行为，而代理则是控制访问。
* Java 内置的代理支持，可以根据需要建立动态代理，并将所有调用分配到所选的处理器。
* 就和其他包装着（wrapper）一样，代理会增加设计中类的数目增加。