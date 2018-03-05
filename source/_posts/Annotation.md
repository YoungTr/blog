---
title: 注解
date: 2018-03-02 10:03:36
tags: [Java]
categories: [Java]
---

注解（也被称为元数据）为在代码中添加信息提供了一种形式化的方法，使我们可以在某个时刻非常方便地使用这些数据。

## 1. 基本语法

### 1.1 定义注解

注解的定义看起来很像接口的定义，与其他任何Java接口一样，注解也会编译成class文件。

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
}
```

在注解中，一般都会包含一些元素以表示某些值。当分析处理注解时，程序或工具可以利用这些值。注解的元素看起来像接口中的方法，唯一的区别是可以为其指定默认值。

没有元素的注解称为标记注解（mark annotation），例如上例中的 @Test。

一个简单的注解，可以用它来跟踪一个项目中的用例。

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    public int id();

    public String description() default "no description";
}
```

在下面的类中，有三个方法被注解为用例：

```
public class PasswordUtil {

    @UseCase(id = 47, description =
            "Passwords must contain at least on number")
    public boolean validatePassword(String password) {
        return (password.matches("\\w*\\d\\w*"));
    }

    @UseCase(id = 48)
    public String encryptPassword(String password) {
        return new StringBuilder(password).reverse().toString();
    }

    @UseCase(id=49,description =
    "New password can't quea")
    public boolean checkForNewPassword(List<String> prevPasswords, String password) {
        return !prevPasswords.contains(password);
    }
}
```

### 1.2 元注解

Java 目前只内置了三种标准注解，以及四种元注解（mate-annotation）。


**@Target** 表示该注解用于什么地方，可能的 ElementType 参数包括：

* **CONSTRUCTOR**：构造器的声明
* **FIELD**：域声明（包括 enum 实例）
* **LOCAL_VARIABLE**：局部变量声明
* **METHOD**：方法声明
* **PACKAGE**：包声明
* **PARAMETER**：参数声明
* **TYPE**：类、接口（包括注解类型）或 enum 声明

**@Retention** 表示需要在什么级别保存该注解信息，可选的 RetentionPolicy 参数包括：

* **SOURCE**：注解将被编译器丢弃
* **CLASS**：注解在 class 文件中可用，但会被 JVM 丢弃
* **RUNTIME**：VM将在运行期也保留注解，因此可以通过反射机制读取注解信息 


**@Documented** 将此注解包含在 Javadoc 中。

**@Inherited** 允许子类继承父类中的注解。

## 2. 编写注解处理器

使用注解的过程中，很重要的一个部分就是创建与使用*注解处理器*。

用一个注解处理器来读取 PasswordUtils 类，并使用反射机制查找 @UseCase 标记。

