---
layout: post
title:  "Java Lambda Expression"
date:   2020-01-31 10:02:10 +0800
categories: jekyll update
---

> The biggest language change in Java 8 is the introduction of lambda expressions—a compact way of passing around behavior.

> Lambda expressions let you express instances of single-method classes more compactly.

匿名内部类使得我们可以把代码/行为当作参数传递，当匿名内部类只有一个方法时，lambda表达式提供了一种更紧凑的传递行为的方式。

```java
Runnable noArguments = () -> System.out.println("Hello World");
```

```java
ActionListener oneArgument = event -> System.out.println("button clicked");
```

```java
Runnable multiStatement = () -> {
  System.out.print("Hello");
  System.out.println(" World");
};
```

```java
BinaryOperator<Long> add = (x, y) -> x + y;
```

```java
BinaryOperator<Long> addExplicit = (Long x, Long y) -> x + y;
```

[Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
