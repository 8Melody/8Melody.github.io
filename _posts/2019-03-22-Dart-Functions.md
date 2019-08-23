---
layout: post
title: "Dart学习第二章"
subtitle: "Dart Functions"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #24b94a, #38ef7d);"
tags:
  - Dart
---

记录Dart学习第二章，方法。

# Functions（方法）

## 概述

`Dart`是一个真正的面向对象语言，其方法也是对象，类型为[Function](https://api.dartlang.org/stable/dart-core/Function-class.html)。这意味着，**方法可以赋值给变量，也可以当作其他方法的参数。** 也可以把`Dart`的实例当作方法来调用，具体参见 [Callable classes](http://dart.goodev.org/guides/language/language-tour#callable-classes)。

定义方法：

```dart
bool isOldMan(int age) {
  return age > 60;
}
```

或者（不推荐）：

```dart
isOldMan(int age) {
  return age > 60;
}
```
在Effective Dart中推荐[在公开的api上指定类型](http://dart.goodev.org/guides/language/effective-dart/design#api-)。

对于只有一个表达式的方法，可以选择 使用缩写语法来定义：

```dart
bool isOldMan(int age) => age > 60;
```

这个 `=> expr` 语法是 `{ return expr; }` 形式的缩写。`=>` 形式 有时候也称之为 `胖箭头` 语法。

> **注意：** 在箭头 (=>) 和冒号 (;) 之间只能使用一个 表达式 – 不能使用 语句。 例如：你不能使用 if statement，但是可以 使用条件表达式 [conditional expression](http://dart.goodev.org/guides/language/language-tour#conditional-expressions)。

## 方法参数

方法参数有两种

* 必须参数
* 可选参数

### 必须参数

如上面实例。

```dart
bool isOldMan(int age) => age > 60;
```

`age`就是必须参数。

### 可选参数

可选参数包含

* 可选命名参数
* 可选位置参数

#### 可选命名参数：
在定义方法的时候，使用 `{param1, param2, …}` 的形式来指定命名参数:

```dart
void config({bool showImage, bool loadLocal}) {

}
```

调用方法的时候，可以使用这种形式 `paramName: value` 来指定命名参数的值：

```dart
config(loadLocal: true, showImage: false);
```

#### 可选位置参数

将方法参数放到`[]`中就变成了可选位置参数。

```dart

String getInfo(String name,int age,[String language]) {
  var info = "name is $name, age is $age";
  if (language != null) {
     info = "$info say $language";
  }
  return info;
}
```

调用：

```dart
print(getInfo("qfxl", 25));
```

输出：
> name is qfxl, age is 25

或者：

```dart
print(getInfo("qfxl", 25, "Chinese"));
```

输出：
> name is qfxl, age is 25 say Chinese

#### 默认参数值

在定义**可选参数**方法的时候，可以使用 `=` 来定义可选参数的默认值。 默认值只能是编译时常量。 如果没有提供默认值，则默认值为 `null`。

```dart
void config({bool showImage= false,bool loadLocal= false}) {
    print("showImage: $showImage loadLocal:$loadLocal");
}
```

在调用的时候可以直接：

```dart
config();
```

或者选择某个参数调用，也可以补齐所有参数调用。

```dart
config(loadLocal:true);
config(showImage:true, loadLocal:true);
```

针对以上例子：

```dart
void startConfig(String name,[int age=20, bool loadLocal]) {
  var msg = "$name age is $age";
  if (loadLocal != null) {
      msg = "$msg config loadLocal $loadLocal";
  }
  print(msg);
}
```

调用：

```dart
startConfig("qfxl");
startConfig("qfxl", 100);
startConfig("qfxl", 100, true);
```

还可以使用`list`、`map`做为默认值。

```dart
void printStudents(
    {List<String> names = const ["Anna", "Bruce", "Cindy"],
    Map<String, int> ages = const {"Anna": 28, "Bruce": 25, "Cindy": 20}}) {
  print("list : $names");
  print("ages : $ages");
}
```

### 将方法当作另一个函数的参数

```dart
invokePrint(element) {
  print(element);
}

main() {
  var list = const ["A", "B", "C"];
  list.forEach(invokePrint);
}
```

方法赋值给变量。

```dart
var name = (msg) => "${msg.toUpperCase()}";
print(name("qfxl"));
```

## 匿名方法

匿名方法又称 *lambda* 或者 *closure闭包* ，可以把匿名方法赋值给一个变量， 然后你可以使用这个方法，比如添加到集合或者从集合中删除。
匿名函数和命名函数看起来类似— 在括号之间可以定义一些参数，参数使用逗号 分割，也可以是可选参数。 后面大括号中的代码为函数体：

```dart
([[Type] param1[, …]]) { 
  codeBlock; 
};
```

如：

```dart
var list = const ["A", "B", "C"];
    list.forEach((item) {
    print(item);
});
```

如果方法中只有一个语句，则可以使用胖箭头`=>`:

```dart
var list = const ["A", "B", "C"];
list.forEach((item) => print(item));
```

## 静态作用域

`Dart`是静态作用域语言，变量的作用域在声明的时候就确定了，作用域规则与`Java`类似。

## closures（闭包）

一个*闭包*是一个**方法对象**，不管该对象在何处被调用，该对象都可以访问其作用域内的变量。

```dart
Function autoIncrement(int origin) {
  return (int param) => origin + param;
}
```

调用：

```dart
//创建方法
var addMethod = autoIncrement(100);
//调用方法
print(addMethod(50));
```

例子：

```dart
Function compareClosures([bool origin = false]) {
  return (bool param) => origin && param;
}


String getMsg(String name, {int age = 0, bool isStudent}) {
  var msg = "$name $age years old";
  return msg = isStudent != null && isStudent ? "$msg , is a student." : "$msg , is not a student.";
}

```

调用：

```dart
void main() {
  print(getMsg("qfxl",age: 30,isStudent: compareClosures()(true)));
}
```

输出：
> qfxl 30 years old , is not a student.


