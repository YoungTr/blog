---
title: View的工作原理
date: 2017-10-23 09:44:06
tags: [Android, Android源码分析]
categories: [Android源码分析]
---

### 1. ViewRootImpl

View 的绘制流程从 ViewRootImpl 的 performTraversals 方法开始，经过 measu、layout 和 draw 三个过程将一个 View 绘制出来。

![performTraversals](/images/performTraversals.png)

1. measure 过程决定了 View 的宽高，Measure 完成以后，可以通过 **getMeasuredWidth** 和 **getMeasureHeight** 方法来获取 View 测量后的宽高。

2. layout 过程决定了 View 四个顶点的位置，可以通过 **getWidth** 和 **getHeight** 方法得到最终宽高。

3. draw 过程决定了 View 的显示。


### 2. 理解 MeasureSpec

MeasureSpec 很大程度上决定了一个 View 的尺寸规格，这个工程还受父容器的影响，因为父容器影响 View 的 MeasureSpec 的创建过程。

#### 2.1 MeasureSpec

MeasureSpec 代表一个 32 位 int 值，**高2位**代表 SpecMode，**低30位**代表 SpecSize。 SpecMode 是指测量模式，SpecSize 是指在某种测量模式下的规格大小。

```
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;
        public static final int EXACTLY     = 1 << MODE_SHIFT;
        public static final int AT_MOST     = 2 << MODE_SHIFT;

        public static int makeMeasureSpec(int size, int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }

        public static int getMode(int measureSpec) {
            return (measureSpec & MODE_MASK);
        }

        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }

```

SpecMode 有三类：

**UNSPECIFIED**

父容器对 View 没有任何限制，要多大给多大。（一般用于系统内部，表示一种测量状态）

**EXACTLY**

父容器已经检测出 View 所需要的**精确大小**，View 的最终大小就是 SpecSize 所指定的值。它对应于 LayoutParams 中的 **match_parent** 和**具体数值**这两种模式。

**AT_MOST**

父容器指定一个可用大小即 SpecSize，View 的大小不能大于这个值。它对应于 LayoutParams 中的 **wrap_content** 。


#### 2.2 MeasureSpec 和 LayoutParams 的对应关系

在 View 测量的时候，系统会将 LayoutParams 在父容器的约束下转换成对应的 MeasureSpec，然后再根据这个 MeasureSpec 来确定 View 测量后的宽高。

需要注意，MeasureSpec 不是唯一由 LayoutParams 决定的，LayoutParams 需要和父容器一起才能决定 View 的 MeasureSpec。


对顶级 View（即 DecorView），其 MeasureSpec 由窗口的尺寸和其自身的 LayoutParams 来共同决定；对于普通 View，其 MeasureSpec 由父容器的 MeasureSpec 和自身的 LayoutParams 来共同决定。

**MeasureSpec 一旦确定后，onMeasure 中就可以确定 View 的测量宽高。**


对 DecorView 来说，ViewRootImpl 中 measureHierarchy 方法，其展示了 DecorView 的 MeasureSpec 的创建过程：

```
	childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
	childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
	performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
```

**desiredWindowWidth，desiredWindowHeight是屏幕的尺寸。**


getRootMeasureSpec 方法实现：

```
private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }
```

DecorView 的 MeasureSpec 遵守如下规则：
 
* LayoutParams.MATCH_PARENT：精确模式，大小就是窗口大小；
* LayoutParams.WRAP_CONTENT：最大模式，大小不定，但不能超过窗口大小；
* 固定大小（比如100dp）：精确模式，大小为 LayoutParams 中指定大小；


对于一般的 View，View 的 measure 由 ViewGroup 传递过来，ViewGroup 的 measureChildMargins 方法：

```
    protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

上述方法对子元素进行 measure ，在调用子元素的 measure 方法 之前会通过 getChildMeasureSpec 方法来获取子元素的 MeasureSpec 。

子元素的 MeasureSpec 的创建与父容器的 MeasureSpec 和子元素的 LayoutParams有关，还与 View 的 margin 及 padding 有关，具体情况可以看一下 ViewGroup 的 getChildMeasureSpec 方法：

```
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

上述方法主要作用是根据父容器的 MeasureSpec 同时结合 View 本身的 LayoutParams 来确定子元素的 MeasureSpec，参数中的 padding 是指父容器中已占用的空间大小，因此子元素可用大小为父容器的尺寸减去 padding 。

普通 View 的 MeasureSpec 创建规则

![view-measure-spec](/images/view-measure-spec.png)

* 当 View 采用固定宽高时，不管父容器的 MeasureSpec 是什么，View 的 MeasureSpec 都是 **MeasureSpec.EXACTLY** 其大小都是遵循 LayoutParams 中的大小。
* 当 View 的宽高是 match_parent 时，如果父容器是 MeasureSpec.EXACTLY 模式，那么 View 也是 MeasureSpec.EXACTLY 模式其大小是父容器的剩余空间；如果父容器是 MeasureSpec.AT_MOST 模式，那么 View 也是 MeasureSpec.AT_MOST 模式其大小不会超过父容器的剩余空间。
* 当 View 的宽高是 wrap_content 时，不管父容器是 MeasureSpec.EXACTLY 还是 MeasureSpec.AT_MOST ，View 总是 MeasureSpec.AT_MOST 模式并且大小不能超过父容器剩余空间。 


### 3. View 的工作流程

#### 3.1 measure 过程

如果是一个原始的 View ，那么通过 measure 就可以完成其测量；如果是一个 ViewGroup ，除了完成自己的测量，还会遍历去调用所有子元素的 measure 方法。