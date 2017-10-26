---
title: Gradle for Android-3:Managing Dependencies
date: 2017-10-26 09:22:47
tags: [Gradle, Android]
category: [Gradle]
---

### 库（Repositories）

Gradle 并不会设置默认的存储库，在 Android Studio 中需要自己将依赖添加到存储块中：

```
    repositories {
        jcenter()
    }
```

Gradle支持三种不同类型的存储库：Maven，Ivy和静态文件或者目录。

依赖关系由三个元素标识：组，名称和版本(group,name,version)。

```
dependencies {
	compile 'com.google.code.gson:gson:2.3'
	compile 'com.squareup.retrofit:retrofit:1.9.0'
}
```

### 预配置的存储库

Gradle已经预配置了三个Maven仓库：JCenter，Maven Central和 local Maven 仓库:

```
repositories {
	mavenCentral()
	jcenter()
	mavenLocal()
}
```
其中，JCenter是Maven Central的超集。


Local Maven 仓库是的所有依赖项的本地缓存，也可以添加自己的依赖关系。默认情况下，本地存储库在主目录中下一个名为.m2的文件夹中。在Linux或Mac OS X上的路径
是〜/.m2 在Microsoft Windows上，是％UserProfile％\.m2


### 远程存储库

一些组织创建有趣的插件或库，更喜欢在自己的Maven或Ivy服务器上托管它们，而不是将它们发布到Maven Central或JCenter。我们可以这样添加这些依赖库：

```
repositories {
	maven {
	url "http://repo.acmecorp.com/maven2"
	}
}
```

### 本地仓库

可以在自己的硬盘驱动器或网络驱动器上运行Maven和Ivy存储库。要将这些添加到构建中，只需将URL配置到驱动器上位置的相对路径或绝对路径：

```
repositories {
	maven {
	url "../repo"
	}
}
```


默认情况下，创建新的Android项目依赖于Android支持库。 使用SDK管理器安装Google存储库时，会在硬盘 ANDROID_SDK/extras/google/m2repository 和ANDROID_SDK/extras/android/m2repository上创建两个Maven存储库。 Gradle可以从这里获得 Google 提供的 libraries，例如 Android Support Library 和Google Play Services 。

也可以使用flatDirs将常规目录添加为存储库：

```
repositories {
	flatDir {
	dirs 'aars'
	}
}
```

### 本地依赖（Local dependencies）

添加一些 jar 文件或者是 native library

### 文件依赖（File dependencies）

要将JAR文件添加为依赖关系，可以使用Gradle提供的文件方法：

```
dependencies {
	compile files('libs/domoarigato.jar')
}
```

如果有很多JAR文件，可以一次添加整个文件夹：

```
dependencies {
	compile fileTree('libs')
}
```

默认情况下，新创建的 Android 项目有一个libs文件夹，并声明它用于依赖。有一个过滤器，确保只使用JAR文件,而文件夹中的所有文件:

```
compile fileTree(dir: 'libs', include: ['*.jar'])
```


### 本地库 (Native libraries)

在模块目录创建一个 “jniLibs” 文件夹，然后创建不同平台的子文件夹，将 .so 文件加入到子文件中：


![gradle-native-libraries](/images/gradle-native-libraries.png)


```
android {
	sourceSets.main {
	jniLibs.srcDir 'src/main/libs'
	}
}
```

### 使用 .aar 文件


要将.aar文件作为依赖项添加，需要在应用程序模块中创建一个文件夹，将.aar文件复制到其中，并将该文件夹添加为存储库：

```
repositories {
	flatDir {
	dirs 'aars'
	}
}
```

也可以这样：

```
dependencies {
	compile(name:'libraryname', ext:'aar')
}
```

这将告诉Gradle寻找具有.aar扩展名的某个名称的库。


### 依赖概念（Dependency concepts）

### 配置（Confgurations）

有时可能需要使用仅存在于某些设备上的 SDK，比如特定厂商提供的蓝牙 SDK。 为了
可以编译代码，需要将 SDK 添加到编译类路径中，而不需要将 SDK 包含在 APK 中，因为已经存在那些设备上了。这是依赖配置的来源。

Gradle groups dependencies into configurations, which are just named sets of files.These are the standard configurations for an Android app or library:

* compile
* apk
* provided
* testCompile
* androidTestCompile

编译配置(**compile configuration**)是默认的，包含所有编译主要应用程序的依赖关系。这种配置下的所有都会被添加 classpath，还会被生成到 apk 中。

apk配置（**apk configuration**）中的依赖关系只会被添加到包中，不会添加到编译类路径（classpath）；提供配置（**provided configuration**）完全相反，它的依赖关系将不被打包。这两个配置只对 JAR 依赖关系有效，添加库项目（library projects）将导致错误。


**testCompile** 和 **androidTestCompile** 配置添加额外的测试库，比如 JUnit 或者 Espresso，这样这些框架只出现在测试 APK 中，不会出现在发布 APK 中。


除了这些标准配置之外，Android插件还为每个构建变体生成配置，从而可以将配置依赖关系添加到诸如 **debugCompile**，**releaseProvided** 等配置中。例如，如果仅将日志框架添加到调试版本中，这就很有用。