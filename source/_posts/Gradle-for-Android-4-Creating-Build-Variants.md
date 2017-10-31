---
layout: w
title: Gradle for Android-4.创建构建变体(Creating Build Variants)
date: 2017-10-31 13:51:42
tags:  [Gradle, Android]
category: [Gradle]
---

当开发应用程序时，通常有几个不同的版本。最常见的情况是，具有用于测试应用程序的测试版本，以及生产版本。通常，这些不同的版本有不同的设置信息，比如测试 URL 可能与生产环境的不同。

### 构建类型（Build types）

在 Gradle 的 Android 插件中，构建类型用于定义应该如何构建应用程序或库。每个构建类型都可以指定是否应该包括调试符号，应用程序ID必须是什么，是否应该删除未使用的资源，等等。可以在buildTypes块中定义构建类型：

```
android {
    buildTypes {
        release {
            minifyEnabled false
			proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
	}
}
```

新模块的默认build.gradle文件会构建一个名为release的构建类型。此构建类型不仅禁用删除未使用的资源（通过将minifyEnabled设置为false）并定义默认 ProGuard 配置文件的位置。

发布构建类型(release build)不是为项目创建的唯一构建类型。默认情况下，每个模块都有一个调试类型（debug build）。它设置为明确的默认值，但可以通过将其包含在 buildTypes 块中来更改其配置，并覆盖要更改的属性。


### 创建构建类型

可以在 buildTypes 块中创建一个新的 build type：

```
android {
    buildTypes {
        staging{
            applicationIdSuffix ".staging"
            versionNameSuffix "-staging"
            buildConfigField "String", "API_URL", "\"http://staging.example.com/api\""
        }
	}
}
```

分段构建类型(staging build type)为应用程序ID设置了一个新的后缀，使其与调试和发行版本的应用程序ID不同。假设具有默认构建配置，加上分段构建类型，构建类型的应用程序ID如下所示：

* **Debug:** com.package
* **Release:** com.pacakge
* **Staging：** com.package.staging

这意味着能够在同一设备上同时安装分段版本和发行版本，而不会引起任何冲突。分段构建类型还具有版本名称后缀，可用于区分同一设备上的多个应用程序版本。

创建新的构建类型时，并不总是从头开始,可以初始化复制另一个构建类型的属性:

```
android {
    buildTypes {
        staging.initWith(buildTypes.debug)
        staging{
            applicationIdSuffix ".staging"
            versionNameSuffix "-staging"
            buildConfigField "String", "API_URL", "\"http://staging.example.com/api\""
        }
	}
}
```

initWith() 方法创建一个新的构建类型，并将所有属性从现有构建类型复制到新创建的类型。通过在新的构建类型对象中简单地将它们定义，可以覆盖属性或定义额外的属性。


### 资源集合（Source sets）

当创建一个新的构建类型，Gradle 会创建一个新的资源集合（source set）。

默认情况下，源集目录被假定为具有与构建类型相同的名称。 当定义新的构建类型时，该目录不会自动创建，必须自己收到创建，然后才能使用构建类型自定义的源代码和资源。

![source-set](/images/gradle-source-set.png)

这样就可以覆盖特定构建类型的某些属性，将自定义代码添加到某些构建类型，并将自定义布局或字符串添加到不同的构建类型。

资源在使用不同的源集合时以特殊方式处理。Drawables 和布局文件将完全覆盖主源集中具有相同名称的资源，但是value目录中的文件（如strings.xml）则不会。Gradle 会将构建类型资源的内容与主要资源合并。

main source set，strings.xml:

```
<resources>
	<string name="app_name">TypesAndFlavors</string>
	<string name="hello_world">Hello world!</string>
</resources>
```

staging build type source set,strings.xml:

```
<resources>
    <string name="app_name">TypesAndFlavors</string>
</resources>
```

合并的strings.xml文件将会像这样：

```
<resources>
	<string name="app_name">TypesAndFlavors STAGING</string>
	<string name="hello_world">Hello world!</string>
</resources>
```

当构建不是暂存的构建类型时，最终的strings.xml文件将只是来自主源集的strings.xml文件。

清单文件(manifest file)也是这样，如果为构建类型创建清单文件，不需要将整个清单文件从主源集中复制，只需添加所需的标签即可。Android插件会将清单文件合并在一起。


### 依赖

每种构建类型都可以有自己的依赖关系。Gradle自动为每种构建类型创建新的依赖关系配置。例如，如果只在调试版本要添加日志框架，，可以这样做：

```
dependencies {
	compile fileTree(dir: 'libs', include: ['*.jar'])
	compile 'com.android.support:appcompat-v7:22.2.0'
	debugCompile 'de.mindpipe.android:android-logging-log4j:1.0.3'
}
```

### Product ﬂavors 产品风格

与用于配置相同应用程序或库的多个不同构建的构建类型相反，产品风格被用于创建不同版本的同一个应用程序。典型的例子是一个有免费和付费版本的应用程序。另一个常见的情况是，一个代理商可以为几个客户端构建具有相同功能的应用程序，其中只有品牌信息改变。这在出租车行业或银行应用程序中非常常见，其中一家公司创建了可以针对同一类别的所有客户端重复使用的应用程序。唯一改变的是主要的颜色，标志和后端的URL。产品风格（product flavors）大大简化了基于相同代码的应用程序的不同版本的过程。

*如果您不确定是否需要新的版本类型或新产品，您应该问自己是否要创建内部使用的同一个应用程序的新版本，或是新版APK发布到Google Play。 如果您需要一个全新的应用程序，需要与您现有的应用程序分开发布，那么使用 product flavors 是一个很好的方法。 否则，您应该坚持使用构建类型。*

[Why are build types distinct from product flavors?](https://stackoverflow.com/questions/27905934/why-are-build-types-distinct-from-product-flavors "Why are build types distinct from product flavors?")


### 创建产品风格

使用 productFlavor ：

```
    productFlavors {
        red {
            applicationId "com.gradleforandroid.red"
            versionCode 3
            resValue "dimen", "flavor_textSize", "20dp"
        }
        blue {
            applicationId "com.gradleforandroid.blue"
            versonCode 4
			minSdkVersion 14
            resValue "dimen", "flavor_textSize", "10dp"
        }
    }
```

Product flavors 具有不同于构建类型的属性。这是因为product flavors是ProductFlavor类的对象，就像所有构建脚本中存在的defaultConfig对象一样。这意味着defaultConfig和所有的product flavors共享相同的属性。

### Source sets

product flavors 有自己的源集（source set）文件夹，文件夹名为 flavor name 跟 build type name，例如 blueRelease，可以为不同的 flavor 设置不同的 app 图标。

组合文件夹的组件将具有比构建类型文件夹和product flavor文件夹中的组件更高的优先级。

