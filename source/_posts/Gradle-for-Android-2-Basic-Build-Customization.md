---
title: 'Gradle for Android-2:Basic Build Customization'
date: 2017-10-24 19:30:18
tags: [Gradle, Android]
category: [Gradle]
---

### 理解 Gradle 文件

Android Studio 会默认创建三个文件：

![gradle-file](/images/gradle-file.png)

**settings.gradle** 文件定义哪些模块包含在构建中。

**根目录 build.gradle** 文件可以配置一些选项，它会应用到项目中所有的模块中。它包含两个默认的代码块：

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

The allprojects block can be used to define properties that need to be applied to all modules. 


**模块（module） build.gradle** 文件包含 module 中应用的选项。它还可以覆盖顶级 build.gradle 的任何选项。

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


*minSdkVersion 和 targetSdkVersion* 跟 manifest 中 <user-sdk>元素功能对应。

*versionCode and versionName* 也跟 manifest 中定义的功能相同。

manifest 中所有的值会被 build 文件中对应值所覆盖，如果 build 文件中没有定义一些值，将会使用manifest 中对应元素的值。

*dependencies block*是标准的 Gradle 配置的一部分，它定义了 app 或者 library 所有的依赖。

### 使用 tasks 

命令

```
gradlew tasks
```

会列出所有可用的任务，这些任务包括 Android tasks，build tasks，build setup tasks，help tasks，install tasks，verification tasks 和 other tasks 。

### 基础任务

* **assemble** assembles the output(s) of the project
* **clean** cleans the output of the project
* **check** runs all the checks, usually unit tests and instrumentation tests
* build runs both assemble and check runs both assemble and check

### Android 任务

* **assemble** creates an APK for every build type
* **clean** removes all the build artifacts, such as the APK files
* **check** performs Lint checks and can abort the build if Lint detects an issue
* **build** runs both assemble and check


**assemble task** 默认有 assembleDebug 和 assembleRelease，也可以增加更多的构建类型。

Android Plugin 新增了一些任务：

* **connectedCheck** runs tests on a connected device or emulator
* **deviceCheck** is a placeholder task for other plugins to run tests on remote devices
* **installDebug** and installRelease install a specific version to a connected device or emulator
* All install tasks also have uninstall counterparts


### BuildConfg and resources

从 SDK tools revision 17 开始，构建工具会根据构建类型创建一个包含 DEBUG 静态变量的 BuildConfig 类。当你 debugging 的时候，这是一个很好的方法，比如说 Log 。

这些常量对于切换特性或设置服务器url很有用,比如

```
 	buildTypes {
        debug {
            buildConfigField "String", "API_URL", "\"http://debug.example.com/api\""
            buildConfigField "boolean", "LOG_HTTP_CALLS", "true"
        }

        release {
            buildConfigField "String", "API_URL", "\"http://release.example.com/api\""
            buildConfigField "boolean", "LOG_HTTP_CALLS", "false"
        }
    }
```

添加了 buildConfigField 后，就可以在代码中使用 BuildConfig.API_URL 和 BuildConfig.LOG_HTTP 常量了。

```
public final class BuildConfig {
  // Fields from build type: debug
  public static final String API_URL = "http://debug.example.com/api";
  public static final boolean LOG_HTTP_CALLS = true;
}
```

Android工具团队还增加了以类似方式提供资源的可能性:


```
    buildTypes {
        debug {"
            resValue "string", "app_name_gradle", "Example DEBUG"
        }

        release {
            resValue "string", "app_name_gradle", "Example"
        }
    }
```

可以这样使用：

```
getString(R.string.app_name_gradle)
```


### 项目范围的设置

如果在一个项目中有多个Android模块,各个模块通用的设置统一处理是很有必要的，这样就不用修改各自模块中的构建文件了。

可以在根目录的构建文件中这样设置：

```
allprojects {
	apply plugin: 'com.android.application'
	android {
		compileSdkVersion 22
		buildToolsVersion "22.0.1"
	}
}
```

但是这样就需要所有的模块都是 Android app 项目，一种更好的办法是在根目录构建文件中定义一些通用的值，这些值可以在不同的模块中使用。在Gradle中可以在Project对象上添加额外的临时属性。任何 build.gradle 文件的 ext block 中都可以定义额外的属性。

根目录的 build.gradle 文件中：

```
ext {
    compileSdkVersion = 25
    buildToolsVersion = "25.0.2"
}
```


在其他的模块的 build.gradle 中可以这么使用

```
android {
	compileSdkVersion rootProject.ext.compileSdkVersion
	buildToolsVersion rootProject.ext.buildToolsVersion
}
```
### 工程属性

三种最常用定义额外属性的方式：

* The ext block
* The gradle.properties file
* The -P command-line parameter


在 build.gradle 文件中可以这样定义：

```
ext {
    local = 'Hello from build.gradle'
}

task printProperties << {
    println local // Local extra property
    println propertiesFile // Property from file
    if (project.hasProperty('cmd')) {
        println cmd // Command line property
    }
}
```

在同目录下的 gradle.properties 文件中定义：

```
propertiesFile = Hello from gradle.properties
```

运行一下命名：

```
gradlew printProperties -Pcmd='Hello from the command line'
```

将会输出

```
:printProperties
Hello from build.gradle
Hello from gradle.properties
Hello from the command line
```

如果一个模块定义了一个属性,它将覆盖已经存在于顶级文件中对应属性。

### 默认任务

如果您运行Gradle而不指定任务，它将运行帮助任务，会打印了有关如何使用Gradle的一些信息。可以设置默认的task，甚至是多个tasks，在根目录的 build.gradle 文件中添加如下：

```
defaultTasks 'clean', 'assembleDebug'
```
当运行没有任何参数的Gradle包装器时，它将运行 clean 和 assembleDebug。

查看设置的默认任务：

```
gradlew tasks | grep "Default tasks"

```

Default tasks: clean, assembleDebug