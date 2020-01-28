---
layout: post
title:  "How Android Main Thread Work"
date:   2020-01-25 18:02:10 +0800
categories: jekyll update
---
### What is Android Main Thread
>When the user launches your app, Android creates a new Linux process along with an execution thread. This main thread, also known as the UI thread, is responsible for everything that happens onscreen.

用户启动应用时，Android系统创建了一个Linux进程和一个执行线程，这个线程就是主线程，也叫UI线程，负责所有屏幕上发生的事情。

>This thread is very important because it is in charge of dispatching events to the appropriate user interface widgets, including drawing events. It is also almost always the thread in which your application interacts with components from the Android UI toolkit (components from the android.widget and android.view packages). As such, the main thread is also sometimes called the UI thread

这个线程很重要，它负责分发事件给UI控件，包括绘制事件。android.widget和android.view包下的组件（UI toolkit组件）几乎总是和主线程（UI线程）交互。

### What Does Android Main Thread Do
>The main thread has a very simple design: Its only job is to take and execute blocks of work from a thread-safe work queue until its app is terminated. The framework generates some of these blocks of work from a variety of places. These places include callbacks associated with lifecycle information, user events such as input, or events coming from other apps and processes. In addition, app can explicitly enqueue blocks on their own, without using the framework.

主线程有一个非常简单的设计：它唯一的工作是执行一个线程安全的工作队列中的一个个的工作块，知道进程退出。这些工作队列中的工作块一部分由Framework产生，一部分由APP自己生成。Framework产生的那部分包括生命周期回调、用户输入事件或者来自其他app的事件。

### How Android Main Thread Work
App启动时，会在主线程开启一个消息循环：`Looper`不断的从其`MessageQueue`中取`Message`进行处理，直到进程退出。

```java
/**
 * This manages the execution of the main thread in an
 * application process, scheduling and executing activities,
 * broadcasts, and other operations on it as the activity
 * manager requests.
 *
 * {@hide}
 */
public final class ActivityThread {
  ...
  public static void main(String[] args) {
    ...
    Looper.prepareMainLooper();
    ...
    Looper.loop();

    throw new RuntimeException("Main thread loop unexpectedly exited");
  }
}
```

### Looper
>Class used to run a message loop for a thread.Threads by default do not have a message loop associated with them; to create one, call {@link #prepare} in the thread that is to run the loop, and then {@link #loop} to have it process messages until the loop is stopped. Most interaction with a message loop is through the {@link Handler} class.

一个线程对应0个或1个Looper；

线程通过调用Looper.prepare来创建Looper；

线程通过调用Looper.loop来开启消息循环；

```java
/**
 * Run the message queue in this thread
 */
public static void loop() {
  final Looper me = myLooper();
  ...
  final MessageQueue queue = me.mQueue;
  ...
  for (;;) {
    Message msg = queue.next();
    ...
    msg.target.dispatchMessage(msg);
    ...
  }
  ...
}
```

### MessageQueue
>Low-level class holding the list of messages to be dispatched by a {@link Looper}.  Messages are not added directly to a MessageQueue, but rather through {@link Handler} objects associated with the Looper. You can retrieve the MessageQueue for the current thread with {@link Looper#myQueue() Looper.myQueue()}.

>A queue of messages to be processed at given time(message.when: earliest time it can be processed)

>next() returns the next message that should be processed. Will not return messages that will be processed at future times.

>Timeing based on SystemClock.uptimeMillis()

想要要在主线程处理的事件，必须想办法发送Message到主线程Looper的MessageQueue，MessageQueue不提供添加Message的API，只能通过Handler来发送Meaasge到MessageQueue.

### How to handle Message
```java
/**
 * Handle system messages here.
 */
public void dispatchMessage(@NonNull Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```

### Handler
一个Looper对应任意个Handler，Handler的初始化方法决定其对应的Looper。  
Handler的MessageQueue和其对用的Looper的MessageQueue是同一个。
```java
public Handler(@NonNull Looper looper, @Nullable Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```
Handler通过sendMessage(message)和post(runnable)等方法向MessageQueue发送Message，最终都会走到enqueueMessage
```java
private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
        long uptimeMillis) {
    msg.target = this;
    msg.workSourceUid = ThreadLocalWorkSource.getUid();
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

### Implication
>when the main thread’s messaging queue contains tasks that are either too numerous or too long for the main thread to complete the update fast enough, the app should move this work to a worker thread. If the main thread cannot finish executing blocks of work within 16ms, the user may observe hitching, lagging, or a lack of UI responsiveness to input. If the main thread blocks for approximately five seconds, the system displays the Application Not Responding (ANR) dialog, allowing the user to close the app directly.

如果主线程Looper的MessageQueue有太多的Message或者有的Message的处理耗时太长，应该将这些任务移到工作线程，如果主线程16ms内不能处理完执行单元，用户会观察到不流畅，也就是所谓的丢帧（drop frame）。如果主线程阻塞5秒，会发生ANR

Thus, there are simply two rules to Android's single thread model:

1. Do not block the UI thread
2. Do not access the Android UI toolkit from outside the UI thread


### Reference
https://www.youtube.com/watch?v=eAtMon8ndfk&list=PL1MI6gyw-RfkNXRX-rAjNlWrt3c8kCRV3&index=7&t=1130s

https://developer.android.com/guide/components/processes-and-threads

https://developer.android.com/topic/performance/threads

