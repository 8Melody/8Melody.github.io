---
layout: post
title: "Dart学习第三章"
subtitle: "Dart Operators"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Dart
---

> 记录Dart学习第三章，操作符。

# Operators（方法）

`Dart`的操作符大部分可以重载，具体参见[重载操作符](http://dart.goodev.org/guides/language/language-tour#overridable-operators)。

## 概述

`Dart`的操作符大部分同其他类型语言，不记录普通操作符用法，仅记录部分需要注意的操作符。

## 类型判断操作符

|操作符|说明|
|:-:|:-:|
|as|类型转换|
|is|如果对象是指定类型则返回true|
|is!|如果对象是指定类型则返回false|

如：

```dart
class Person {
  int age;
  String name;
}

class Qfxl extends Person {
  Qfxl(String name) {
    this.name = name;
  }
}

void main() {
  var qfxl = Qfxl("qfxl");
  if (qfxl is Person) {
      print(qfxl.name);
  }
}
```

或者使用`as`。

```dart
print((qfxl as Person).name);
```

## 赋值操作符

正常情况使用`=`来赋值，还有一个`??=`用来指定值为null的值。

```dart
a = value;   // 给 a 变量赋值
b ??= value; // 如果 b 是 null，则赋值给 b；
             // 如果不是 null，则 b 的值保持不变
```

```dart
var me = Qfxl("qfxl");
var per = Qfxl("hello");
per ??= me;

print(per.name); // hello
```

```dart
var me = Qfxl("qfxl");
var per = null;
per ??= me;

print(per.name); // qfxl
```

## 位操作符

|操作符|说明|
|:-:|:-:|
|&|AND|
|\||OR|
|^|XOR|
|<<|左移位|
|>>|右移位|

具体使用类似`Java`。

## 条件表达式

`Dart`有两个特殊操作符来代替`if else`。

### condition ? expr1 : expr2
跟`Java`的三元运算符一致。

```dart
String getMSg(int age) => age < 25 ? "is Student" : "is not Student";
```

### expr1 ?? expr2
如果`expr1`为`non-null`，则返回`expr1`否则执行并返回返回`expr2`。

```dart
class Person {
  int age;
  String name;

  getMsg() => "name is $name";
}

class Qfxl extends Person {
  Qfxl(String name) {
    this.name = name;
  }

  String msg;

  @override
  getMsg() {
    return msg ?? super.getMsg();
  }
}
```

## 级联操作符(..)

级联操作符`..`可以在同一个对象连续调用多个函数以及访问成员变量。
如：

```dart
TextView()
      ..text = "按钮"
      ..textColor = Color.RED
      ..setOnClickListener((v) => toast("click"));
```

级联操作符并不是没有限制，如下就会报错。

```dart
var sb = new StringBuffer();
sb.write('foo')..write('bar');
```

```dart
sb.write()
```
返回的是一个`void`，无法在`void`上使用级联操作符。


## 其他操作符

|操作符|名称|解释|
|:-:|:-|:-|
|()|调用方法|代表调用一个方法|
|[]|访问list|访问list中指定元素|
|.|访问成员元素|如`foo.bar`|
|?.|条件成员元素访问|和`.`类似，但是左边的操作符不能为`null`，如`fool?.bar`，如果`fool`为`null`则返回`null`否则返回`bar`成员。|

其他具体使用方法，参见[classes](http://dart.goodev.org/guides/language/language-tour#classes)。