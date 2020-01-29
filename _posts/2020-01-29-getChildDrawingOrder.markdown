---
layout: post
title:  "getChildDrawingOrder"
date:   2020-01-29 18:02:10 +0800
categories: jekyll update
---

```java
/**
 * Converts drawing order position to container position. Override this
 * if you want to change the drawing order of children. By default, it
 * returns drawingPosition.
 * <p>
 * NOTE: In order for this method to be called, you must enable child ordering
 * first by calling {@link #setChildrenDrawingOrderEnabled(boolean)}.
 *
 * @param drawingPosition the drawing order position.
 * @return the container position of a child for this drawing order position.
 *
 * @see #setChildrenDrawingOrderEnabled(boolean)
 * @see #isChildrenDrawingOrderEnabled()
 */
protected int getChildDrawingOrder(int childCount, int drawingPosition) {
    return drawingPosition;
}
```

ViewGroup子View的绘制顺序和添加顺序一致，如果想要改变子View的绘制顺序需要重写此方法。该方法的输入是drawingPosition，输出是子View的container position。意思是ViewGroup中第x（container position）个子View应该在第drawingPosition个绘制。

注意：越后面绘制的子View越在上层（Z轴）

例如：SwipeRefreshLayout需要将刷新icon绘制在最上层，所以其重写方法如下：

```java
/**
 * i is drawingPosition
 */
@Override
protected int getChildDrawingOrder(int childCount, int i) {
    if (mCircleViewIndex < 0) {
        return i;
    } else if (i == childCount - 1) {
        // Draw the selected child last
        return mCircleViewIndex;
    } else if (i >= mCircleViewIndex) {
        // Move the children after the selected child earlier one
        return i + 1;
    } else {
        // Keep the children before the selected child the same
        return i;
    }
}
```