---
title: 'Gradle for Android-2:Basic Build Customization'
date: 2017-10-24 19:30:18
tags: [Gradle, Android]
category: [Gradle]
---

### Understanding the Gradle files

Android Studio 会默认创建三个文件：

![gradle-file](/images/gradle-file.png)

**The seetings file** 定义哪些模块包含在构建中。

**The top-level build file**可以配置一些选项，它会应用到项目中所有的模块中。它包含两个默认的代码块：

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

The allprojects block can be used to defne properties that need to be applied to all modules. 


**The module build file**包含 module 中应用的选项。它还可以覆盖顶级 build.gradle 的任何选项。

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 25
    buildToolsVersion '25.0.3'

    defaultConfig {
        applicationId "uk.co.senab.photoview.sample"
        minSdkVersion 14
        targetSdkVersion 25
        versionCode 100
        versionName "1.0"
    }

	buildTypes {
		release {
			minifyEnabled false
			proguardFiles getDefaultProguardFile
			('proguard-android.txt'), 'proguard-rules.pro'
		}
	}
}

dependencies {
	compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:25.3.1'
}

```

**defaultConfig block**配置了 app 的核心属性，这些属性会覆盖 AndroidManifest.xml 中对应的条目。

*applicationId* 会覆盖 manifest 中的 package name，但是 applicationId 和 package name 有很多不同。

在 Gradle 作为 Android 的构建系统之前，AndroidManifest.xml 中的 package name 有两种目的：它是应用的唯一标识和用作R资源类中的包的名称，现在 applicationId 将作为设备应用的唯一标识。这样便于构建不同版本的app，而使用同一 package name 的R资源。


*applicationId 和 targetSdkVersion* 跟 manifest 中 <user-sdk>元素很相似。

*versionCode and versionName* 也跟 manifest 中定义的功能相同。

manifest 中所有的值会被 build 文件中对应值所覆盖，如果 build 文件中没有定义一些值，将会使用manifest 中对应元素的值。

*dependencies block*是标准的 Gradle 配置的一部分，它定义了 app 或者 library 所有的依赖。