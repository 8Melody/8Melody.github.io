---
layout: post
title: "Dart之泛型"
subtitle: "Dart泛型简记"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Dart
---

> 本文记录Dart泛型。

# 为何使用泛型
在 `Dart` 中类型是可选的，有些情况下可能想使用类型来表明你的意图，不管是使用泛型还是具体类型。

这点跟其他强类型语言非常类似，Dart泛型跟Java的泛型基本一致。

```dart
  var list = new List<String>();
  list.addAll(["a","b","c"]);
```

另外一个使用泛型的原因是减少重复的代码。 泛型可以在多种类型之间定义同一个实现， 同时还可以继续使用检查模式和静态分析工具提供的代码分析功能。 例如，创建一个保存缓存对象的接口：

```dart
abstract class ObjectCache{
  Object getCache();
  setByKey(String key,Object value);
}
```

如果再需要一个缓存String的Cache。

```dart
abstract class StringCache{
  String getCache();
  setByKey(String key, String value);
}
```

为了避免重复，泛型就能利用上了。

```dart
abstract class Cache<T> {
  T getCache();
  setByKey(String key, T value);
}
```

# 使用集合字面量

List 和 map 字面量也是可以参数化的。 参数化定义 list 需要在中括号之前 添加 `<type>` ， 定义 `map` 需要在大括号之前 添加 `<keyType, valueType>`。 如果需要更加安全的类型检查，则可以使用 参数化定义。下面是一些示例：

```dart
  var names = <String>["A", "B", "C"];
  var maps = <String, String>{"A": "a", "B": "b"};
```

# 在构造函数中使用泛型

```dart
  var list = new List<String>();
  list.addAll(["a","b","c"]);
  
  var set = new Set<String>.from(list);
```

# 限制泛型类型

当使用泛型类型的时候，可能想限制泛型的具体类型。 使用 `extends` 可以实现这个功能：

```dart
class Person {
  
}

class Worker<T extends Person> {
  
}
```

# 使用泛型函数

```dart
T first<T>(List<T> ts) {
  return ts[0];
}

Map<K, V> singleMap<K, V>(K k, V v) {
  return <K, V>{k : v};
}
```
