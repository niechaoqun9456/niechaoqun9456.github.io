---
layout: post
title:  "Kotlin Scope Funtion"
date:   2020-01-29 18:02:10 +0800
categories: jekyll update
---

### 种类
一共五种：let, run, with, apply, also

> They are funtions

> When you call such a function on an object with a lambda expression provided, it forms a temporary scope.

### 差别
> These functions do the same: execute a block of code.What's different is how this object becomes available inside the block and what is the result of the whole expression.

在lambda表达式内通过it或this引用对象，使代码更简洁、可读性更高，差别如下：
- 获取对象（context objext）的方式：this or it
- 返回值：对象本省 或者 lambda表达式的返回值

获取对象方式：  
run、apply、with通过this引用对象，通常用于操作对象的成员变量场景：调用对象的方法、为对象的属性赋值。大多数时候this可省略。  
let、also通过it引用对象，常用于将对象作为参数传递的场景，必要时可以重命名it。

返回值：  
apply、also返回对象本身,可以链式调用  
run、let、with返回lambda的返回值

### let常用场景
1. 只在非空值的情况下执行代码片段，相当于Java总的if (object != null)
```java
val str: String? = "Hello"   
//processNonNullString(str)       // compilation error: str can be null
val length = str?.let { 
    println("let() called on $it")        
    processNonNullString(it)      // OK: 'it' is not null inside '?.let { }'
    it.length
}
```

2. 在scope中引入局部变量提高代码可读性：
```java
val numbers = listOf("one", "two", "three", "four")
val modifiedFirstItem = numbers.first().let { firstItem ->
    println("The first item of the list is '$firstItem'")
    if (firstItem.length >= 5) firstItem else "!" + firstItem + "!"
}.toUpperCase()
println("First item after modifications: '$modifiedFirstItem'")
```

### with（非扩展函数）常用场景
1. We recommend with for calling functions on the context object without providing the lambda result.
```java
// with this object, do the follwing
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    println("'with' is called with argument $this")
    println("It contains $size elements")
}
```
2. 利用对象的属性或方法来计算一个新值作为lambda结果返回
```java
val numbers = mutableListOf("one", "two", "three")
val firstAndLast = with(numbers) {
    "The first element is ${first()}," +
    " the last element is ${last()}"
}
println(firstAndLast)
```

### apply常用场景
object configuration:apply the following assignments to the object.
```java
val adam = Person("Adam").apply {
    age = 32
    city = "London"        
}
```

### run常用场景
1. 当你需要初始化对象，同时需要在lambda中做计算并返回计算值时适合用run
```java
val service = MultiportService("https://example.kotlinlang.org", 80)
val result = service.run {
    port = 8080
    query(prepareRequest() + " to port $port")
}
// the same code written with let() function:
val letResult = service.let {
    it.port = 8080
    it.query(it.prepareRequest() + " to port ${it.port}")
}
```
2. 作为非扩展函数使用( called without the context object)
```java
val hexNumberRegex = run {
    val digits = "0-9"
    val hexDigits = "A-Fa-f"
    val sign = "+-"

    Regex("[$sign]?[$digits$hexDigits]+")
}
for (match in hexNumberRegex.findAll("+1234 -FFFF not-a-number")) {
    println(match.value)
}
```

### also常用场景
常用于不修改对象的额外操作，将对象作为参数传递
```java
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("The list elements before adding new one: $it") }
    .add("four")
```
