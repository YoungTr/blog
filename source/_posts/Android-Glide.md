---
title: Android 图片加载框架 Glide
date: 2017-10-18 18:34:53
tags: [开源框架]
category: [开源框架]
---

[Github 地址 https://github.com/bumptech/glide](https://github.com/bumptech/glide "Glide github 地址")

[中文文档 https://muyangmin.github.io/glide-docs-cn/](https://muyangmin.github.io/glide-docs-cn/ "中文文档")

[郭霖的博客-Glide最全解析 http://blog.csdn.net/column/details/15318.html](http://blog.csdn.net/column/details/15318.html "郭霖的博客-Glide最全解析")

## 1. 基本用法

**Gradle**

```
repositories {
  mavenCentral()
  maven { url 'https://maven.google.com' }
}

dependencies {
    compile 'com.github.bumptech.glide:glide:4.2.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.2.0'
}
```


**API**


```
Glide.with(fragment)
    .load(url)
    .into(imageView);
```

**Glide V4 Generated API**

Glide v4 使用 注解处理器 (Annotation Processor) 来生成出一个 API，在 Application 模块中可使用该流式 API 一次性调用到 RequestBuilder， RequestOptions 和集成库中所有的选项。

在 Application 模块中包含一个 AppGlideModule 的实现：

```
@GlideModule
public final class MyAppGlideModule extends AppGlideModule {}
```

**使用 Generated API**

```
 GlideApp.with(this)
                .asBitmap() //指定图片格式 link asGif() asDrawable()
                .load(IMAGE_URL)
                .placeholder(R.drawable.loading)//loading 占位符
                .error(R.drawable.error)        //error 占位符
                .diskCacheStrategy(DiskCacheStrategy.NONE)//禁用缓存功能
                .override(200, 200)  //指定大小
                .centerCrop() //裁剪方式  link centerInside() fitCenter()
                .into(mImageView);
```