---
title: Gradle for Android-1:Getting Started with Gradle and Android Studio
date: 2017-10-24 14:32:01
tags: [Gradle, Android]
category: [Gradle]
---


### 1. 开始使用 Gradle 和 Android Studio

#### Understanding Gradle basics

Android 项目中要使用 Gradle，需要创建一个 “build.gradle” 的脚本文件。

#### Projects and tasks

Gradle 中最重要的两个概念 projects 和 tasks 。 每个构建（build）至少有一个 project ，每个 project 包含一个或多个 tasks 。 每个 build.gradle 代表一个 project 。

### The build confguration file

Android 中 “build.gradle” 文件中必须要有一些元素：

```
buildscript {
    repositories {
        jcenter() //仓库
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
    }
}
```

plugin 

```
apply plugin: 'com.android.application'
```

```
apply plugin: 'com.android.library'
```

编译的SDK 和 buildTools 版本

```
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
}
```

### The project structure

![gradle-project-structure](/images/gradle-project-structure.png)


source set *is a group of source fles, which are compiled and executed together*.

![gradle-source-set](/images/gradle-source-set.png)


### Getting started with the Gradle Wrapper

Gradle Wrapper 未安装 Gradle 也可以使用，gradlew会委托gradle命令来做相应的事情。

创建 gradle wrapper，在 build.gradle 中添加如下 task

```
task wrapper(type: Wrapper) {
gradleVersion = '2.4'
}
```

执行 **gradle wrapper**创建 wrapper 文件：

![gradle-gradlew-wrapper](/images/gradle-gradlew-wrapper.png)

build.gradle 默认有 wrapper 任务，也可以指定 gradle 的版本

```
gradle wrapper --gradle-version 2.4
```

### Running basic build tasks

```
gradlew tasks
```

将会列出所有可用的任务，添加 --all 参数，将会为每个任务列出更详细的概述。

```
gradlew assembleDebug
```

这个任务会创建一个 debug 版本的 app，默认路径为 MyApp/app/build/
outputs/apk 

**Abbreviated task names**

Gradle还提供了简化的驼峰任务名称作为快捷方式，当然这个驼峰简称必须使唯一的，比如，

```
gradlew assDeb
```

或者

```
gradlew aD
```
效果与 

```
gradlew assembleDebug
```

一样。

### Keeping the old project structure

从 Elicpse  导入项目时，所有的源都将驻留在同一个文件夹中，所以需要告诉Gradle所有这些组件都可以在src文件夹中找到。

```
android {
	sourceSets {
		main {
			manifest.srcFile 'AndroidManifest.xml'
			java.srcDirs = ['src']
			resources.srcDirs = ['src']
			aidl.srcDirs = ['src']
			renderscript.srcDirs = ['src']
			res.srcDirs = ['res']
			assets.srcDirs = ['assets']
		}
	androidTest.setRoot('tests')
	}
}
```

如果需要依赖**JAR 文件**，需要告诉 Gradle 这些依赖文件的位置，通常会把 JAR 文件放在 libs 文件夹中：

```
dependencies {
	compile fileTree(dir: 'libs', include: ['*.jar'])
}
```

在项目根目录创建 “setting.gradle” 文件：

```
include: ':app'
```

它的目的是告诉Gradle在构建中包含 app 模块。

在根目录和 module 目录中都需要一个 “build.gradle” 文件。