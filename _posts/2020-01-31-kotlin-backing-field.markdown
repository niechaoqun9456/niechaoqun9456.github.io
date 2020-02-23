---
layout: post
title:  "kotlin backing field"
date:   2020-02-01 10:02:10 +0800
categories: jekyll update
---

> Fields cannot be declared directly in Kotlin classes. However, when a property needs a backing field, Kotlin provides it automatically. This backing field can be referenced in the accessors using the field identifier.

> The field identifier can only be used in the accessors of the property.

```java
var counter = 0 // note the keyword field
    set(value) {
        if (value >= 0) field = value
    }
```

### 为什么需要backing field
避免自定义get set时递归引起的StackOverflowError
```java
var selectedColor: Int = someDefaultValue
        get() = selectedColor
        set(value) {
            this.selectedColor = value
            doSomething()
        }
```
以上代码会形成递归，引起StackOverflowError

用backing field替换变量selectedColor可以解决该问题
```java
var selectedColor: Int = someDefaultValue
        get() = field
        set(value) {
            field = value
        }
```

> A backing field will be generated for a property if it uses the default implementation of at least one of the accessors, or if a custom accessor references it through the field identifier.

例如：
```java
val isEmpty: Boolean
    get() = this.size == 0
```
因为这里没有使用默认get实现，并且没在自定义get中引用backing filed，所以这里不会生成backing field


[backing-field-in-kotlin](https://medium.com/@agrawalsuneet/backing-field-in-kotlin-bd9c2d5b6da5)

[properties](https://kotlinlang.org/docs/reference/properties.html)