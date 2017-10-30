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


##### 3.1.1 View 的 measure 工程

View 的 measure 工程由其 measure 方法完成，该方法是一个 final 类型的方法，在该方法中会其调用 onMeasure 方法：

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```
setMeasuredDimension 方法会设置 View 的宽高，只需要看一下 getDefaultSize 方法：

```
	public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```

对于 MeasureSpec.AT_MOST 和 MeasureSpec.EXACTLY ，getDefaultSize 返回的大小就是 measureSpec 中的 specSize，而这个 specSize 就是 View 测量后的大小。

对 MeasureSpec.UNSPECIFIED ，View 的宽高是 getSuggestedMinimumWidth 和 getSuggestedMinimumHeight 的返回值。


```
    protected int getSuggestedMinimumWidth() {
        return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
    }
```

如果 View 没有设置背景，那么 View 的宽为 mMinWidth，即 android:minWidth 属性所指定的值，默认为 0；如果设置了背景，则 View 的宽为 max(mMinWidth,，mBackground.getMinimumWidth())

```
    public int getMinimumWidth() {
        final int intrinsicWidth = getIntrinsicWidth();
        return intrinsicWidth > 0 ? intrinsicWidth : 0;
    }
```
getMinimumWidth 就是返回 Drawable 的原始宽度。

可以得出这样一个结论：直接继承 View 自定义控件需要重写 onMeasure 方法并设置 wrap_content 时自身大小，否则布局中使用 wrap_content 就相当于使用 match_parent。原因见 *普通 View 的 MeasureSpec 创建规则*

##### 3.1.2 ViewGrop 的 measure 过程

对 ViewGroup 来说，除了完成自己的 measure 过程，还会遍历调用所有子元素的 measure 方法，各个子元素再递归去执行这个过程。

ViewGroup 是一个抽象类，没有重写 View 的 onMeasure 方法，提供了一个 measureChildren 方法：

```
    protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }
```

ViewGroup 在 measure 是，会对每一个子元素进行 measure：

```
    protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

mesureChild 思想就是去除子元素的 LayoutParams，然后再通过 getChildMeasureSpec 来创建子元素的 MeasureSpec，接着将 MeasureSpec 直接传递给 View 的 measure 方法来进行测量。


ViewGroup 并没有定义其测量的具体过程，因为不同的布局实现细节不同，无法统一处理，通过 LinearLayout 的 onMeasure 方法分析 ViewGroup 的 measure 过程。




LinearLayout 的 onMeasure 方法：

```
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mOrientation == VERTICAL) {
            measureVertical(widthMeasureSpec, heightMeasureSpec);
        } else {
            measureHorizontal(widthMeasureSpec, heightMeasureSpec);
        }
    }
```

这里查看一下 measureVertical 方法：

```
 		// See how tall everyone is. Also remember max width.
 		
    	for (int i = 0; i < count; ++i) {
            final View child = getVirtualChildAt(i);
            ....
        // Determine how big this child would like to be. If this or
        / previous children have given a weight, then we allow it to
        / use all available space (and we will shrink things later
        // if needed).
        
        measureChildBeforeLayout(
               child, i, widthMeasureSpec, 0, heightMeasureSpec,
                totalWeight == 0 ? mTotalLength : 0);
            if (oldHeight != Integer.MIN_VALUE) {
                 lp.height = oldHeight;
             }
             final int childHeight = child.getMeasuredHeight();
             final int totalLength = mTotalLength;
             mTotalLength = Math.max(totalLength, totalLength + childHeight + lp.topMargin +
                    lp.bottomMargin + getNextLocationOffset(child));
         }
```



系统会遍历子元素并对每个子元素执行 measureChildBeforeLayout 方法，这个方法内部会调用子元素的 measure 方法，各个子元素就开始依次进入 measure 过程，并行会通过 mTotalLength 变量来存储 LinearLayout 在竖直方向的初步高度。

当子元素测量完毕后，LinearLayout 会测量自己的大小：

```
 // Add in our padding
 mTotalLength += mPaddingTop + mPaddingBottom;
 int heightSize = mTotalLength;

 // Check against our minimum height
 heightSize = Math.max(heightSize, getSuggestedMinimumHeight());
 
 // Reconcile our calculated size with the heightMeasureSpec
 int heightSizeAndState = resolveSizeAndState(heightSize, heightMeasureSpec, 0);
 heightSize = heightSizeAndState & MEASURED_SIZE_MASK;
 setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
                heightSizeAndState);
        
```


View 的 measure 过程是三大流程中最复杂的一个，measure 完成以后，通过 getMeasuredWidth/Height 方法可以正确地获取到 View 的测量宽高。在某些极端情况下，系统需要多次 measure 才能确定最终的测量宽高，这样在 onMeasure 方法中拿到测量的宽高可能是不准确的。一个比较好的方法是在 onLayout 方法中去获取 View 的测量宽高或者最终的宽高。


#### 3.2 layout 过程

Layout 的作用是 ViewGroup 用来确定子元素的位置，当 ViewGroup 的位置确定后，它在 onLayout 中会遍历所有子元素并调用 layout 方法，在 layout 方法中 onLayout 方法又会被调用。

layout 的方法确定 View 本身的位置，onLayout 方法则会确认所有子元素的位置。

View 的 layout 方法：

```
    public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);

            if (shouldDrawRoundScrollbar()) {
                if(mRoundScrollbarRenderer == null) {
                    mRoundScrollbarRenderer = new RoundScrollbarRenderer(this);
                }
            } else {
                mRoundScrollbarRenderer = null;
            }

            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
    }
```

首先，setFrame 方法设定 View 四个顶点的位置，即初始化 mLeft、mTop、mRight、mBottom 四个值，View 的四个顶点的位置一旦确定，View 在父容器中的位置也就确定了；接着会调用 onLayout 方法，作用是父容器确定子元素的位置，其实现与具体的布局有关。

LinearLayout 的 onLayout 方法：

```
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        if (mOrientation == VERTICAL) {
            layoutVertical(l, t, r, b);
        } else {
            layoutHorizontal(l, t, r, b);
        }
    }
```

layoutVertical 方法：

```
    void layoutVertical(int left, int top, int right, int bottom) {
	
		......

        for (int i = 0; i < count; i++) {
            final View child = getVirtualChildAt(i);
            if (child == null) {
                childTop += measureNullChild(i);
            } else if (child.getVisibility() != GONE) {
                final int childWidth = child.getMeasuredWidth();
                final int childHeight = child.getMeasuredHeight();
                
                final LinearLayout.LayoutParams lp =
                        (LinearLayout.LayoutParams) child.getLayoutParams();
                
                int gravity = lp.gravity;
                if (gravity < 0) {
                    gravity = minorGravity;
                }
                final int layoutDirection = getLayoutDirection();
               	
				......

                if (hasDividerBeforeChildAt(i)) {
                    childTop += mDividerHeight;
                }

                childTop += lp.topMargin;
                setChildFrame(child, childLeft, childTop + getLocationOffset(child),
                        childWidth, childHeight);
                childTop += childHeight + lp.bottomMargin + getNextLocationOffset(child);

                i += getChildrenSkipCount(child, i);
            }
        }
    }
```


此方法会遍历所有子元素并调用 setChildFrame 方法来为子元素指定对应位置，childTop 会逐渐变大，这意味着后面的子元素会被放置在靠下的位置。

setChildFrame 方法仅仅是调用子元素的 layout 方法，这样父元素在 layout 方法中完成自己的定位后，就通过调用 onLayout 方法去调用子元素的 layout 方法，子元素会通过自己的 layout 方法确定自己的位置，这样一层一层传递下去就完成整个 View 树的 layout 过程。

```
    private void setChildFrame(View child, int left, int top, int width, int height) {        
        child.layout(left, top, left + width, top + height);
    }
```

#### 3.3 draw 过程

Draw 过程是将 View 绘制到屏幕上，View 的绘制流程遵循一下几个步骤：

1. 绘制背景 background.draw(canvas)
2. 绘制自己 (onDraw)
3. 绘制 children (dispatchDraw)
4. 绘制装饰 (onDrawScrollBars)

```
    public void draw(Canvas canvas) {
        final int privateFlags = mPrivateFlags;
        final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
                (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
        mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

        // Step 1, draw the background, if needed
        int saveCount;

        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

            // Overlay is part of the content and draws beneath Foreground
            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // Step 6, draw decorations (foreground, scrollbars)
            onDrawForeground(canvas);

            // we're done...
            return;
        }
        ......
```


