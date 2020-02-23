---
layout: post
title:  "Custom View Tips"
date:   2020-01-31 10:02:10 +0800
categories: jekyll update
---

1. 自定义View的构造函数最好至少包含View(Context context)和View(Context context, AttributeSet attrs)这两个，以支持在xml中声明和在代码中new
2. 直接继承View的自定义View应该重写onMeasure方法避免warp_content不生效（变成match_parent）
3. invalidate reflect state change, redrawn at some point in the future. invalidate(l, t, r, b)更高效，只会重新绘制标记的rect区域
4. 因为onDraw的执行很频繁，不要在onDraw中new对象
5. invalidate only when u need.invalidate only what u need.
6. View的测量过程像是子View和父View的沟通，子View通过LayoutParameter告诉父View它期望的大小，父View通过child.measure(measureSpec)将大小限制告诉子View: AT_MOS、EXACTLY、UNSPECIFIED，父View的MeasureSpec和子View的LayoutParams共同决定了子View的MeasureSpec
7. onMeasure最重要的步骤在于获取合适的宽高，大部分情况可以通过View.resolveSize方法获取
8. 自定义ViewGroup需要负责子View的测量和布局child.measure child.layout
9. ViewGroup中有些静态方法可以帮助我们测量子View:getChildMeasureSpec、measureCHild、measureChildWithMargins等等

[CustomView](https://youtu.be/4NNmMO8Aykw?list=PL1MI6gyw-RfkNXRX-rAjNlWrt3c8kCRV3)
