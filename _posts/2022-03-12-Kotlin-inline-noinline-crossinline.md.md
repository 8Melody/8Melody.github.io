---
layout: post
title: "Kotlin inline noinline crossinline"
subtitle: ""
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - inline 
  - kotlin
---

# 前言

在Kotlin的官方代码中，这几个关键字经常会看到。

比如常用的拓展函数`apply`，源码就使用了`inline`关键字。

```java
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}

```

本文记录对于`inline`、`noinline`、`crossinline`的作用。

# inline

被`inline`修饰的函数称之为`内联函数`。

**作用：被`inline`修饰的函数，在被调用处会被平整铺开，就是函数的内容会被直插到调用处。**

使用场景：

在Kotlin的高阶函数中，如果使用lambda表达式作为函数的参数，那么这个参数在编译成字节码的时候，这个参数会变成一个名为`Function`的对象。

比如：

```java
fun noInlined(block: () -> Unit) {
    println("before")
    block()
    println("after")
}
```
对应Java代码大概如下：
```java
public final void noInlined(Function block) {
    System.out.println("before");
    block.invoke();
    System.out.println("after");
}
```

在Kotlin中调用该函数：
```java
fun main() {
    noInlined {
        println("Hello World")
    }
}
```
对应的Java代码大致为：
```java
public final void main() {
    nonInlined(new Function() {
      @Override
      public void invoke() {
           System.out.println("Hello World");
        }
    }); 
}
```
此时lambda参数会直接变成一个对象，如果在小范围内调用那么对性能的影响微乎其微，但是如果在循环或者在更加频繁的场景调用则会产生很多无用对象。
此时就需要使用`inline`优化该函数。

使用`inline`：

```java
inline fun inlined(block: () -> Unit) {
    println("before")
    block()
    println("after")
}
```

再在Kotlin中调用该函数：
```
fun main() {
    inlined {
        println("Hello World")
    }
}
```

编译之后：

```java
public final void main() {
    System.out.println("before");
    System.out.println("Hello World");
    System.out.println("after");
}
```
此时的lambda参数会被直接调用，不会产生额外对象，整个调用会被“平整铺开”。

但是对应的缺陷也是有的，比如此时想return该lambda参数的话是行不通的，毕竟此时的lambda已经不被系统认为是一个对象了。如果想要解除这种限制的话，则需要使用另外一个关键字`noinline`。

# noinline

**作用：关闭局部内联优化。**

不同于`inline`，**`noinline`不是作用于函数，而是作用于函数的参数。**对于添加了`noinline`的lambda参数不会参与函数的内联。

比如，在inline修饰的函数中，如果想要使用lambda参数作为对象处理的话，需要加上noinline，不然编译器会报错。

错误示范：

```java
inline fun inlined(block: () -> Unit):() -> Unit {
    return block
}
```
此时编译器会提示：
> Illegal usage of inline-parameter 'block' in 'public inline fun inlined(block: () -> Unit): () -> Unit defined in root package in file InLineSample.kt'. Add 'noinline' modifier to the parameter declaration

正确示范：

```java
inline fun inlined(noinline block: () -> Unit):() -> Unit {
    return block
}
```

# crossinline

**作用：加强局部内联优化。**

`crossinline`也是**作用于函数的参数**。使用场景为，对于在内联函数中，lambda参数需要在内联函数中被间接调用的情况。

错误示范：

```java
inline fun crossInlined(block: () -> Unit) {
    Runnable {
        block()
    }
}
```
此时编译器会提示：
> Can't inline 'block' here: it may contain non-local returns. Add 'crossinline' modifier to parameter declaration 'block'

正确示范：

```java
inline fun crossInlined(crossinline block: () -> Unit) {
    Runnable {
        block()
    }
}
```

# 总结

* `inline` 可以使用内联（即函数内容直插到调用处）的方式编译函数。
* `noinline` 用来局部地、指向性地关闭内联优化。如果不使用`noinline`修饰`lambda`参数，则该`lambda`参数无法当作对象使用。
* `crossinline` 当需要在内联函数中，间接调用lambda参数的时候，需要使用`crossinline`，但是不能在调用处的lambda表达式中使用`return`。

`inline`使用建议：如果函数使用了对象类型的参数（如常见的`lambda`），建议给函数加上`inline`但是如果对于包的体积很敏感，则建议在频繁调用的场景下再使用`inline`。