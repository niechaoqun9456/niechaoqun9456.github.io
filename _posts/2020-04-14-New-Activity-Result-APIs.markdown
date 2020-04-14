---
layout: post
title:  "Activity Result APIs"
date:   2020-04-14 09:02:10 +0800
categories: jekyll update
---
### 介绍
> The Activity Result APIs provide components for registering for a result, launching the result, and handling the result once it is dispatched by the system.

AndroidX在androidx.activity:activity:1.2.0-alpha02和androidx.fragment:fragment:1.3.0-alpha02引入了Activity Result APIs，推荐使用

### 使用
#### 定义ActivityResultContract和ActivityResultCallback
```java
// 在ComponentActivity或Fragment中可以通过prepareCall()方法注册ActivityResultCallback,相当于之前的onActivityResult
// prepareCall返回ActivityResultLauncher用于启动Activity,相当于之前的startActivityForResult
// prepareCall可以在Activity或Fragment创建前调用
val getContent = prepareCall(GetContent()) { uri: Uri? ->
    // Handle the returned Uri
}
```
#### 启动Activity
```java
val getContent = prepareCall(GetContent()) { uri: Uri? ->
    // Handle the returned Uri
}

override fun onCreate(savedInstanceState: Bundle?) {
    // ...

    val selectButton = findViewById<Button>(R.id.select_button)

    selectButton.setOnClickListener {
        // Use the Kotlin extension in activity-ktx
        // passing it the mime type you'd like to allow the user to select
        getContent("image/*")
    }
}
```
#### 在非Activity和Fragment类中利用ActivityResultRegistry绑定生命周期
```java
class MyLifecycleObserver(private val registry : ActivityResultRegistry)
        : DefaultLifecycleObserver {
    lateinit var getContent : ActivityResultLauncher<String>

    fun onCreate(owner: LifecycleOwner) {
        getContent = registry.register("key", owner, GetContent()) { uri ->
            // Handle the returned Uri
        }
    }

    fun selectImage() {
        getContent("image/*")
    }
}

class MyFragment : Fragment() {
    lateinit var observer : MyLifecycleObserver

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...

        observer = MyLifecycleObserver(requireActivity().activityResultRegistry)
        lifecycle.addObserver(observer)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val selectButton = view.findViewById<Button>(R.id.select_button)

        selectButton.setOnClickListener {
            // Open the activity to select an image
            observer.selectImage()
        }
    }
}
```


### Reference
[Getting a result from an activity](https://developer.android.com/training/basics/intents/result#prepare)