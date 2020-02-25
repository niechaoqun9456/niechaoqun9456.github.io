---
layout: post
title:  "SharedPreferences commit vs apply"
date:   2020-02-24 18:SharedPreferences02:10 +0800
categories: jekyll update
---
### When to use SharedPreferences
> This class provides strong consistency guarantees. It is using expensive operations which might slow down an app. Frequently changing properties or properties where loss can be tolerated should use other mechanisms.

> If you have a relatively small collection of key-values that you'd like to save, you should use the SharedPreferences APIs. 

### commit or apply
commit和apply的目的都是将你的修改写到文件（/data/data/[packageName]/shared_prefs/***.xml）中去，差别在于commit是同步的，apply是异步的，内部实现用到了CountDownLatch

```java
@Override
public void apply() {
    final long startTime = System.currentTimeMillis();
    final MemoryCommitResult mcr = commitToMemory();
    final Runnable awaitCommit = new Runnable() {
            @Override
            public void run() {
                try {
                    mcr.writtenToDiskLatch.await();
                } catch (InterruptedException ignored) {
                }
                if (DEBUG && mcr.wasWritten) {
                    Log.d(TAG, mFile.getName() + ":" + mcr.memoryStateGeneration
                            + " applied after " + (System.currentTimeMillis() - startTime)
                            + " ms");
                }
            }
        };
    QueuedWork.addFinisher(awaitCommit);
    Runnable postWriteRunnable = new Runnable() {
            @Override
            public void run() {
                awaitCommit.run();
                QueuedWork.removeFinisher(awaitCommit);
            }
        };
    SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);
    // Okay to notify the listeners before it's hit disk
    // because the listeners should always get the same
    // SharedPreferences instance back, which has the
    // changes reflected in memory.
    notifyListeners(mcr);
}
```

```java
@Override
public boolean commit() {
    long startTime = 0;
    if (DEBUG) {
        startTime = System.currentTimeMillis();
    }
    MemoryCommitResult mcr = commitToMemory();
    SharedPreferencesImpl.this.enqueueDiskWrite(
        mcr, null /* sync write on this thread okay */);
    try {
        mcr.writtenToDiskLatch.await();
    } catch (InterruptedException e) {
        return false;
    } finally {
        if (DEBUG) {
            Log.d(TAG, mFile.getName() + ":" + mcr.memoryStateGeneration
                    + " committed after " + (System.currentTimeMillis() - startTime)
                    + " ms");
        }
    }
    notifyListeners(mcr);
    return mcr.writeToDiskResult;
}
```

### Attention
SP只适用于单进程应用