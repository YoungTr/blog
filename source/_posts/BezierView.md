---
title: BezierView
date: 2017-09-20 21:13:33
tags:  [Android, View]
categories: [Android自定义View]
---

### 简单学习了贝塞尔曲线的原理和使用

二阶贝塞尔曲线在Android中的API为：quadTo()和rQuadTo()，三阶贝塞尔曲线在Android中的API为：cubicTo()和rCubicTo().

#### 1. 一阶贝塞尔曲线和水波纹效果的使用

<img src="https://youngtr.github.io/images/Bezier.gif" width="50%" height="50%">

关键代码

 		mPath.reset();
        mPath.moveTo(-mWaveLength + mXOffset, mCenterY);
        for (int i = 0; i < mWaveCount; i++) {
            mPath.quadTo((-mWaveLength * 3 / 4) + (i * mWaveLength) + mXOffset, mCenterY + mYOffset, (-mWaveLength / 2) + (i * mWaveLength) + mXOffset, mCenterY);
            mPath.quadTo((-mWaveLength / 4) + (i * mWaveLength) + mXOffset, mCenterY - mYOffset, i * mWaveLength + mXOffset, mCenterY);
        }
        mPath.lineTo(mWidth, mHeight);
        mPath.lineTo(0, mHeight);
        mPath.close();
        canvas.drawPath(mPath, mWavePaint);

#### 2. 曲线的拟合

<img src="https://youngtr.github.io/images/bezierview.png" width="50%" height="50%">

#### 3. 粘性拖动控件

<img src="https://youngtr.github.io/images/drag.gif" width="50%" height="50%">


**[源码BezierView](https://github.com/YoungTr/BezierView)**

#### 参考

[贝塞尔曲线原理 http://www.cnblogs.com/hnfxs/p/3148483.html](http://www.cnblogs.com/hnfxs/p/3148483.html)

[Android QQ小红点的实现 http://blog.csdn.net/mabeijianxi/article/details/50560361](http://blog.csdn.net/mabeijianxi/article/details/50560361)

[贝塞尔曲线开发的艺术 http://www.jianshu.com/p/55c721887568](http://www.jianshu.com/p/55c721887568)
