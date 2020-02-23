---
layout: post
title:  "Understanding Android memory usage"
date:   2020-02-23 11:02:10 +0800
categories: jekyll update
---

### How memory use impacts a device
内存在设备上是分块存储的：
![physical_memory_on_device](/images/physical_memory_on_device.jpg)
当应用需要内存，而Free Memory数量小于kswapd threshold时，linux进程kswapd会回收Cached Memory以获得更多的Free Momory；  
如果kswapd回收Cached Memory时，如果Cached Memory数量小于lmk threshold时，系统会杀死低优先级的应用以获得内存；
![kswapd-lmk](/images/kswapd-lmk.jpg)
Android应用优先级如下所示，优先级从上往下由高到低：
![Android-process-list](/images/Android-process-list.jpg)
当前台应用消耗内存持续增加，而Cached Memory数量一直小于lmk threshold时，系统就会不断杀死进程，用户就会感觉手机越来越慢，甚至前台应用Crash，甚至手机重启
![critical_memory_status](/images/critical_memory_status.jpg)

### Evaluating application memery impact
计算应用消耗的内存要考虑共享内存：
![Shared-pages](/images/Shared-pages.jpg)
可以用如下命令查看应用消耗内存情况:  
adb shell dumpsys meminfo -s [processName]  
应用为了提供更多的应用价值而使用更多内存是ok的，重要的是如何平衡：
![user-value-vs-memory-impact](/images/user-value-vs-memory-impact.jpg)

### Reducing your application's memory impact
- Profile your Java heap with Android Studio's memory profiler
- Reduce your APK size

### References
[Understanding Android memory usage (Google I/O '18)](https://www.youtube.com/watch?v=w7K0jio8afM&list=PL1MI6gyw-RfmxpYRHnHQcN0vTIQHB2WTi&index=5&t=0s)