---
layout: post
title:  "kotlin extension note"
date:   2020-02-01 10:02:10 +0800
categories: jekyll update
---

不光有扩展方法，还有扩展属性。

扩展属性没有backing field。

扩展方法是静态解析的：被调用的方法是在调用定义时决定的，不是由运行时的调用对象类型决定的。
```java
open class Shape
class Rectangle: Shape()
fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"
fun printClassName(s: Shape) {
    println(s.getName())
}
printClassName(Rectangle())// print Shape not Rectangle
```

Note on visibility:
- An extension declared on top level of a file has access to the other private top-level declarations in the same file;
- If an extension is declared outside its receiver type, such an extension cannot access the receiver's private members.

[Extensions](https://kotlinlang.org/docs/reference/extensions.html)