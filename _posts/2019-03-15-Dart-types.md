---
layout: post
title: "Dart学习第一章"
subtitle: "Dart开篇"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Dart
---

> 记录Dart学习第一章，内置类型。

# Hello World

喜闻乐见的Hello World

```dart
void main() {
  print('Hello Dart');
}
```

# 变量

```dart
var name = 'qfxl';
```

变量是一个引用。上面名字为`name`的变量引用了 一个内容为 “qfxl” 的 String 对象。

## 默认值

**在`Dart`中所有变量的默认值都为`null`，即使是`int`其默认值依然是`null`。**

```dart
int age;
assert(age == null);
print('age is null');
```

上面程序会输出

> age is null

## 可选类型

上面定义的`name`为`var`，但是其类型是`String`，也可以直接使用具体类型声明。

```dart
String name = 'qfxl';
```

添加类型可以更加清晰的表达你的意图。 IDE 编译器等工具有可以使用类型来更好的帮助你， 可以提供代码补全、提前发现 bug 等功能。

不过根据[Effective Dart](http://dart.goodev.org/guides/language/effective-dart/design#type-annotations)上面推荐，使用`var`而不是具体的类型来定义变量。

## Final和const
如果不打算修改一个变量，则可以使用`final`和`const`。
一个`final`变量只能赋值一次；一个`const`变量是编译时常量。（`const`变量同时也是`final`变量）顶级的`final`变量或者类中的`final`变量在第一次使用的时候初始化。

> **注意:** 实例变量可以为`final`但不能是`const`。

`final`变量实例。

```dart
final name = 'qfxl';
final String name = 'qfxl'; //或者这样声明
final var name = 'qfxl'; //这样会报错，var不能用final修饰。
```

`const`变量为编译时常量，如果`const`定义在类中需要使用`static const`，可以直接定义`const`的值或者使用其他`const`的值来初始化。

```dart
const name = 'qfxl';
const newName = name;
const String name = 'qfxl'; //或者这样声明
const var name = 'qfxl'; //这样会报错，var不能用final修饰。
```

`const`不仅可以定义常量，也可以创建不变的值，还能定义构造函数为`const`类型的类，这种类型构建出来的类是不可变的。任何变量都可以有一个不变的值。

`[]`代表`list`。

```dart
//定义的list为unmodifiable的
var list1 = const [];
const list2 = const [];
final list3 = const [];
//这样赋值没问题
list1 = list2;
//如果修改list则会报错。
list1.add('1');
list2.add('1');
list3.add('3');
```

# 内置类型
    
`Dart`内置类型如下：

* numbers
* strings
* booleans
* Collections
* runes(用于在字符串中表示Unicode字符)
* symbols
 
## numbers(数值)

`Dart`支持两种数字类型。

### int

整数值，取值范围为 `$-2^{53}$` 和 `$2^{53}$` 之间。

### double
64-bit(双精度)浮点数。

`int`和`double`都是`num`的子类，`num`类型定义了基本的操作符，如`+`、`-`、`*`、`/`，还定义了`abs()`、`ceil()`和`floor()`等函数。位操作符，如`>>`定义在`int`中。

定义整数的实例：

```dart
var x = 1;
int y = 0xFFEEBB;
const z = 31234534534545345;
final a = -100;
```

如果一个数带小数，则其为`double`。

```dart
var t = 3.1415;
double u = -1.3333;
const v = 1.0;
final w = -0.1;
```

下面为字符串跟数字之间的转换

```dart
//String -> int
var one = int.parse('1');
//String -> double
var two = double.parse('3.14');
//int -> String
var strOne = 1.toString();
//double -> String
var strTwo = 3.14159265.toStringAsFixed(2);//输出3.14
```

整数类型支持位运算。

```dart
var x = 4;
var y = x >> 1; //结果为2
```

**很多算术表达式 只要其操作数是常量，则表达式结果 也是编译时常量。**

```dart
const a = 1;
const b = 2;
const c = a + b;
```

## Strings(字符串)

`Dart`字符串是`UTF-16`编码的字符序列。可以使用单引号或者双引号来定义字符串。

```dart
var name = 'qfxl';
String firstName = "xu";
final lastName = "yh";
const EngName = "qfxl";
```

可以在字符串中使用表达式，方法跟`Kotlin`一致`${expression}`。

```dart
var a = 20;
var b = 7;
var name = 'qfxl';
print('my name is $name age is ${a + b}');
```

`==` 操作符判断两个对象的内容是否一样。 如果两个字符串包含一样的字符编码序列， 则他们是相等的。

```dart
var a = 'qfxl';
var b = 'qfxl';
assert(a == b);
```

拼接字符串可以使用`+`，也可以把多个字符串放一起来实现。

```dart
var a = 'hello'
        "world";
var b = "hello" + 'world';
assert(a == b);
```

可以使用3个`''`或者`""`来创建多行文本，这点跟`Kotlin`也一致。

```dart
var a = '''
        This 
        is 
        a 
        Text
        ''';
```

如果字符串中包含特殊符，如`\n`、`\t`等，如果想取消这些转移符的作用，在字符前面加上`r`，这点跟`Python`类似。

```dart
var a = r"This is \n a text";
```

其他`String`的方法还有`contains()`、`indexOf()`、`startsWith()`、`endsWith()`、`split()`等，[具体API。](https://api.dartlang.org/stable/2.2.0/dart-core/String-class.html)

## Booleans

为了代表布尔值，`Dart`有一个`bool`类型，在`Dart`中只有`true`对象才是true，其余都为false。

```dart
bool isSuccess = true;
var isFailed = false;
```

## Collections
`Dart` 提供了一些核心的集合 API，包含 `List`, `Set`, 和 `Map`

### List

也称`array`或者有序集合，是所有编程语言中最常见的集合类型。在`Dart`中数组就是`List`。

```dart
var list1 = [1, 2, 3, 4];
List list3 = const [];
final list2 = new List();
```

`List`下标从0开始，跟其他编程语言的特性类似，尤其类似`js`。

```dart
var list = [];
list.add("hello");
list.add("world");
var size = list.length;
for (var i = 0; i < size; i++) {

}
list[1] = "Dart";
list.removeLast();

var fruits = ['bananas', 'apples', 'oranges'];
// Sort a list.
fruits.sort((a, b) => a.compareTo(b));
assert(fruits[0] == 'apples');
fruits.clear();

var fruits = new List<String>();

fruits.add('apples');
var fruit = fruits[0];
assert(fruit is String);
```

[List具体API。](https://api.dartlang.org/stable/2.2.0/dart-core/List-class.html)

### Set
`Dart`中`Set`是一个无序的集合，里面不能有重复数据，由于是无序的，所以不能用索引来获取数据。

```dart
var names = new Set();
names.addAll(['a','b','c']);
names.add('a');
assert(names.length == 3);
names.remove('a');
assert(names.length == 2);
```

判断`Set`中是否包含某些元素，方法为`contains()`、`containsAll()`。

```dart
var names = new Set();
names.addAll(['a','b','c']);
assert(names.contains("a"));
assert(names.containsAll(['a','b']));
```

获取两个`Set`中的交集，方法为`set.intersection(set)`。

```dart
var setA = new Set();
setA.addAll(['a','b','c']);
var setB = new Set.from(['b','c','d']);

var intersection = setA.intersection(setB);
print(intersection); // 输出{b,c}
```


### Map

也称为字典，也是一个无序集合。

定义`Map`。

```dart
var mapA = {
  "s1":"hello",
  "s2":"world"
};

var mapB = new Map();
mapB["s1"] = "hello";
mapB["s2"] = "world";
  
Map mapC = new Map<String,String>();
mapC.addAll(mapB);
```

`Map`的增删改。

```dart
var mapA = new Map<String,int>();
//增
mapA["a"] = 1;
mapA["b"] = 2;
mapA["c"] = 3;
//删
mapA.remove("a");
mapA.removeWhere((key,value) => key == "b");
//改
mapA["c"] = 0;
print(mapA); //输出{c:0}
```

判断`Map`是否包含某个key。

```dart
var mapA = new Map<String,int>();
mapA["a"] = 1;
mapA["b"] = 2;
mapA["c"] = 3;
assert(mapA.containsKey("a"));
```

`Map`的`putIfAbsent()`方法来修改`Map`的值，但是只有该 key 在 map 中不存在的时候才设置这个值，否则key 的值保持不变。如果key存在，则返回原始key对应的值，如果key不存在则返回表达式的值：

```dart
var mapA = new Map<String,int>();
mapA["a"] = 1;

var value1 = mapA.putIfAbsent("a", () => 2); //value1 = 1
assert(mapA.length == 1);
var value2 = mapA.putIfAbsent("b", () => 2); //value2 = 2
assert(mapA.length == 2);
```

`Map`循环。

```dart
var mapA = new Map<String,int>();
mapA["a"] = 1;
mapA["b"] = 2;
mapA["c"] = 3;

for (MapEntry<String,int> entry in mapA.entries) {
  print("${entry.key} = ${entry.value}");
}

mapA.forEach((k,v) {
  print("$k = $v");
});
```

`Map`的子类有：

* `HashMap`
* `LinkedHashMap`
* `MapMixin`
* `MapView`
* `HttpSession`

`Map`的详细信息[参考这里。](https://api.dartlang.org/stable/2.2.0/dart-core/Map-class.html)


## Runes

在`Dart`中，`runes`代表字符串的`UTF-32 code points`。
`Unicode`为每一个字符、标点符号、表情符号等都定义了 一个唯一的数值。由于`Dart`字符串是`UTF-16 code units` 字符序列，所以在字符串中表达`32-bit Unicode`值就需要 新的语法了。

通常使用`\uXXXX`的方式来表示`Unicode code point`， 这里的`XXXX`是4个16进制的数。例如，心形符号 (♥) 是 `\u2665`。 对于非 4 个数值的情况，把编码值放到大括号中即可。 例如，笑脸 emoji (😆) 是`\u{1f600}`。

`String` 类 有一些属性可以提取`rune`信息。`codeUnitAt `和`codeUnit`属性返回`16-bit code units`。使用`runes` 属性来获取字符串的`runes`信息。

```dart
var clapping = '\u{1f44f}';
print(clapping);
print(clapping.codeUnits);
print(clapping.runes.toList());

var input = new Runes(
      '\u2665  \u{1f605}  \u{1f60e}  \u{1f47b}  \u{1f596}  \u{1f44d}');
print(new String.fromCharCodes(input));
```

> **注意:**  使用 list 操作 runes 的时候请小心。 根据所操作的语种、字符集等， 这种操作方式可能导致你的字符串出问题。 更多信息参考 Stack Overflow 上的一个问题：[Dart如何反转一个字符串。](http://stackoverflow.com/questions/21521729/how-do-i-reverse-a-string-in-dart)

## Symbols

`symbols`代表`Dart`程序中声明的操作符或者标识符，这个基本很少会用到。该功能对于通过名字来引用标识符的情况 是特别有用，特别是混淆后的代码， 标识符的名字被混淆了，但是`symbol`的名字不会改变。

使用 Symbol 字面量来获取标识符的`symbol` 对象，也就是在标识符 前面添加一个`#`符号：

```dart
#hello;
#world;
```

`symbol`字面量定义是编译时常量。

[具体使用。](http://dart.goodev.org/guides/libraries/library-tour#dartmirrors---reflection)