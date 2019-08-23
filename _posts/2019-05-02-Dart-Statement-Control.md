---
layout: post
title: "Dart学习第四章"
subtitle: "记录Dart第四章-流程控制。"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #24b94a, #38ef7d);"
tags:
  - Dart
---

> 记录Dart第四章-流程控制。

# 流程控制

> 本文记录`Dart`流程控制及异常处理。

* `if` and `else`
* `for` loops
* `while` and `do while` loops
* `break` and `continue`
* `switch` and `case`
* `assert`

## if and else

与大部分通用语言一致

```dart
void testIfElse(int age) {
  var msg;
  if (age > 0 && age <= 20) {
    msg = "青少年";
  } else if (age > 20 && age < 100) {
    msg = "中老年";
  } else {
    msg = "仙人";
  }
  print(msg);
}
```

## for loops

与大部分通用语言一致

```dart
void testForLoops() {
  const list = ["a","b","c"];
  for (var i = 0; i < list.length; i++) {
    print(list[i]);
  }
}
```

`Dart`中的`for`循环中的闭包会捕获循环的`index`。

```dart
void testElsLoops() {
  var list = [];
  for (var i = 0; i < 2; i++) {
    list.add(() => print(i));
  }
  list.forEach((c) => c());
}
```

如果要遍历的对象实现了`Iterable` 接口，则可以使用 [forEach()](https://api.dartlang.org/stable/dart-core/Iterable/forEach.html) 方法。如果没必要当前遍历的索引，则使用[forEach()](https://api.dartlang.org/stable/dart-core/Iterable/forEach.html)更加简洁：

```dart
var list = ["a", "b", "c"];

list.forEach((s) => print(s));
```

 `List` 和 `Set` 等实现了 `Iterable` 接口的类还支持 `for-in` 形式的遍历：

```dart
var list = ["a", "b", "c"];

for (var item in list) {
    print(item);
  }
```

## while and  do...while

与其他通用语言一致。

```dart
void testWhile() {
    var list = ["a", "b", "c"];

    var index = 0;

    while (index < list.length) {
        print(list[index]);
        index++;
  }
}
```

```dart
void testDoWhile() {
  var list = ["a", "b", "c"];

  var index = 0;
  do {
    print(list[index]);
    index++;
  } while (index < list.length);
}

```

## break and continue

`break`

```dart
void testBreak() {
  const list = ["a", "b", "c"];
  for (var i = 0; i < list.length; i++) {
    print(list[i]);
    if (i == 1) {
      break;
    }
  }
}
```

`continue`

```dart
void testContinue() {
  const list = ["a", "b", "c"];
  for (var i = 0; i < list.length; i++) {
    if (i == 1) {
      continue;
    }
    print(list[i]);
  }
}
```

或者

```dart
list.where((c) => c != "b").forEach((c) => print(c));
```

## switch case

`Dart`中的`Switch`语句使用`==`比较`integer`、`string`或者编译时常量，比较的对象必须时同一个类的实例，`class`必须没有覆盖`==`操作符。

每个非空的`case`语句必须有一个`break`语句，另外可以通过`continue`、`throw`或者`return`来结束非空`case`语句，当没有`case`语句匹配的时候，可以使用`default`语句来匹配这种情况。

```dart
 var CMD = "OPEN";
  switch (CMD) {
    case "CLOSED":
      print("closed");
      break;
    case "OPEN":
      print("open");
      break;
    case "PENDING":
      print("pending");
      break;
    default:
      print("default");
  }
```

也可以通过`continue`跳转到指定的label处执行：

```dart
 var CMD = "OPEN";
  switch (CMD) {
    case "CLOSED":
      print("closed");
      break;
    case "OPEN":
      print("open");
      continue pending;
    pending:
    case "PENDING":
      print("pending");
      break;
    default:
      print("default");
  }
```

输出：
> open
  pending


## Assert（断言）

如果条件表达式结果不满足需要，则可以使用 `assert` 语句俩打断代码的执行。

> **注意：** 断言只在检查模式下运行有效，如果在生产模式 运行，则断言不会执行。

```dart
var name = "qfxl";
assert(name == "hello");
print("now you see me!");
```

`assert` 方法的参数可以为任何返回布尔值的表达式或者方法。 如果返回的值为 true， 断言执行通过，执行结束。 如果返回值为 false， 断言执行失败，会抛出一个异常 `AssertionError`。


## Exceptions

抛出一个异常

```dart
throw new FormatException("msg is null");
```

或者

```dart
throw "msg is null";
```

## Catch

可以使用 `on` 或者 `catch` 来声明捕获语句，也可以 同时使用。使用 `on` 来指定异常类型，使用 `catch` 来 捕获异常对象。

捕获异常可以避免异常继续传递（重新抛出rethrow异常除外）

```dart
try {
    throw new FormatException("msg is null");
  } on FormatException{
    print("catched a FormatException");
  } 
```
函数 `catch()` 可以带有一个或者两个参数， 第一个参数为抛出的异常对象， 第二个为堆栈信息 (一个 [StackTrace](https://api.dartlang.org/stable/2.2.0/dart-core/StackTrace-class.html) 对象)。

```dart
 try {
    throw new FormatException("msg is null");
  } on FormatException catch(e,s){
    print("catched a FormatException $e $s");
  }
```

### rethrow

使用 `rethrow` 可以将异常重新抛出。

```dart
try {
    throw new FormatException("msg is null");
  } on FormatException{
    print("catched a FormatException");
    rethrow;
  }
```

输出：

```java 
catched a FormatException
Unhandled exception:
FormatException: msg is null
xxx
```

## Finally

要确保某些代码执行，不管有没有出现异常都需要执行，可以使用 一个     `finally` 语句来实现。如果没有 `catch` 语句来捕获异常， 则在执行完 `finally` 语句后， 异常被抛出了：

```dart
try {
    throw new FormatException("出错了");
  } finally {
    print("我执行在抛出异常之前");
  }
```

定义的 `finally` 语句在匹配的 `catch` 语句之后执行：

```dart
 try {
    throw new FormatException("出错了");
  } on FormatException {
    print("我捕获了一个FormatException");
  } finally {
    print("我执行在抛出异常之前");
  }
```

输出：

```dart
我捕获了一个FormatException
我执行在抛出异常之前
```

更多[Exception](http://dart.goodev.org/guides/libraries/library-tour#exceptions)。