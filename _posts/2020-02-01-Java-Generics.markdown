---
layout: post
title:  "Java Generics"
date:   2020-02-01 11:02:10 +0800
categories: jekyll update
---

### Introduction
Generics was introduced in jdk 5.0, allow you to abstract over types.

```java
List myIntList = new LinkedList(); // 1
myIntList.add(new Integer(0)); // 2
Integer x = (Integer) myIntList.iterator().next(); // 3  
```
better:
```java
List<Integer> 
    myIntList = new LinkedList<Integer>(); // 1'
myIntList.add(new Integer(0)); // 2'
Integer x = myIntList.iterator().next(); // 3'
```
List<Integer>. We say that List is a generic interface that takes a type parameter--in this case, Integer.

### Defining Simple Generics
```java
public interface List <E> {
    void add(E x);
    Iterator<E> iterator();
}
```
List<Integer> is invocation of generic type declaration.

A generic type declaration is compiled once and for all, and turned into a single class file, just like an ordinary class or interface declaration.

### Generics and Subtyping
If Foo is a subtype (subclass or subinterface) of Bar, and G is some generic type declaration, it is not the case that G<Foo> is a subtype of G<Bar>. 
```java
// below code do not compile
List<String> ls = new ArrayList<String>(); // 1
List<Object> lo = ls; // 2 
lo.add(new Object()); // 3  
String s = ls.get(0); // 4
```

### Wildcards
```java
Collection<?> c = new ArrayList<String>();
c.add(new Object()); // Compile time error
```
When the actual type parameter is ?, it stands for some unknown type. Any parameter we pass to add would have to be a subtype of this unknown type. Since we don't know what type that is, we cannot pass anything in. The sole exception is null, which is a member of every type.

List<? extends Shape> is an example of a bounded wildcard. The ? stands for an unknown type, just like the wildcards we saw earlier. However, in this case, we know that this unknown type is in fact a subtype of Shape. (Note: It could be Shape itself, or some subclass; it need not literally extend Shape.) We say that Shape is the upper bound of the wildcard.

There is, as usual, a price to be paid for the flexibility of using wildcards. That price is that it is now illegal to write into shapes in the body of the method. For instance, this is not allowed:

```java
public void addRectangle(List<? extends Shape> shapes) {
    // Compile-time error!
    shapes.add(0, new Rectangle());
}
```

[generics](https://docs.oracle.com/javase/tutorial/extra/generics/intro.html)