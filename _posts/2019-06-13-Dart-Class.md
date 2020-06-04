---
layout: post
title: "Dart第五章"
subtitle: "Dart之class"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Dart
---

Dart第五章-Class。

# Class

Dart是一个面向对象语言，同时支持基于 mixin 的继承机制。每个对象都是一个实例，所有的类都继承自 [Object](https://api.dartlang.org/stable/dart-core/Object-class.html)。
基于 Mixin 的继承意味着每个类（Object 除外） 都**只有一个父类**，一个类的代码可以在其他 多个类继承中重复使用。

使用 `new` 关键字和构造函数来创建新的对象。


```dart
var p = new Point();

class Point {
    
}
```

每个实例变量都会自动生成一个 `getter` 方法（隐含的）。 `Non-final` 实例变量还会自动生成一个 `setter` 方法。

## 定义实例变量

```dart
class Point {
    num a;//null
    num b;//null
    num c = 0;
}
```

```dart
var p = new Point();
p.a = 100;
assert(p.a == 100);
```

所有没有初始化的变量值都是 `null`。

每个实例变量都会自动生成一个 `getter` 方法（隐含的）。 `Non-final` 实例变量还会自动生成一个 setter 方法，参见下文`Getter&Setter`。

## 构造函数

定义一个和类名字一样的方法就定义了一个构造函数 还可以带有其他可选的标识符。

```dart
class Point {
  num a;
  num b;
  num c = 0;
  //不推荐
  Point(num a, num b) {
    this.a = a;
    this.b = b;
  }
}
```

因为这种赋值操作太常见了，所以Dart提供了如下语法糖。

```dart
class Point {
  num a;
  num b;
  num c = 0;

  Point(this.a, this.b);
}  
```

### Default constructors（默认构造函数）

如果你没有定义构造函数，则会有个默认构造函数。 默认构造函数没有参数，并且会调用超类的没有参数的构造函数。

### Constructors aren’t inherited（构造函数不会继承）

子类不会继承超类的构造函数。子类如果没有定义构造函数，则只有一个默认构造函数 （没有名字没有参数）。

### Named constructors（命名构造函数）

使用命名构造函数可以为一个类实现多个构造函数， 或者使用命名构造函数来更清晰的表明你的意图：

```dart
class Point {
  num a;
  num b;
  num c = 0;
  
  Point.fromJson(Map map) {
    this.a = map['a'];
    this.b = map['b'];
  }
}
```

**注意：构造函数不能继承，所以超类的命名构造函数 也不会被继承。如果希望 子类也有超类一样的命名构造函数，必须在子类中自己实现该构造函数。**

### Invoking a non-default superclass constructor（调用超类构造函数）

默认情况下，子类的构造函数会自动调用超类的 无名无参数的默认构造函数。

超类的构造函数在子类构造函数体开始执行的位置调用。 如果提供了一个 `initializer list（初始化参数列表）[见下文]`，则初始化参数列表在超类构造函数执行之前执行。下面是构造函数执行顺序：

* 1，initializer list（初始化参数列表）
* 2，superclass’s no-arg constructor（超类的无名构造函数）
* 3，main class’s no-arg constructor（主类的无名构造函数）

```dart
void main() {
  var stu = Student.fromJson({});
}

class Person {
  String name;

  Person.fromJson(Map json) {
    print("invoke Person constructor");
  }
}

class Student extends Person {
  Student.fromJson(Map json) : super.fromJson(json) {
    print("invoke Student constructor");
  }
}

```

执行顺序为：

```dart
invoke Person constructor
invoke Student constructor
```

由于超类构造函数的参数在构造函数执行之前执行，所以 参数可以是一个表达式或者 一个方法调用：

```dart
class Student extends Person {
  // ...
  Student() : super.fromJson(findDefaultData());
}
```

**注意： 如果在构造函数的初始化列表中使用 super()，需要把它放到最后。**

### Initializer list（初始化列表）

在构造函数体执行之前除了可以调用超类构造函数之外，还可以初始化实例参数。 使用逗号分隔初始化表达式。
    
```dart
class Point {
  num a;
  num b;
  num square;
  
  Point(x, y)
      : a = x,
        b = y,
        square = x * y;
}   

var p = Point(10,10);
print(p.square);//100
```

### Redirecting constructors（重定向构造函数）

有时候一个构造函数会调动类中的其他构造函数。 一个重定向构造函数是没有代码的，在构造函数声明后，使用冒号调用其他构造函数。

```dart
class Point {
  num a;
  num b;
  num square;

  Point(this.a, this.b);

  Point.alongX(x) : this(x, 0);
}  
```

### Constant constructors（常量构造函数）

如果类提供一个状态不变的对象，可以把这些对象 定义为编译时常量。要实现这个功能，需要定义一个 const 构造函数， 并且声明所有类的变量为 final。

```dart
class ImmutablePoint {
  final num x;
  final num y;
  const ImmutablePoint(this.x, this.y);
  static final ImmutablePoint origin =
      const ImmutablePoint(0, 0);
}
```

### Factory constructors（工厂方法构造函数）

如果一个构造函数并不总是返回一个新的对象，则使用 factory 来定义 这个构造函数。例如，一个工厂构造函数 可能从缓存中获取一个实例并返回，或者 返回一个子类型的实例。

```dart
void main() {
  var _logger = new Logger("qfxl");
}

class Logger {
  String name;

  Logger._internal(this.name);

  static final Map<String, Logger> _cache = Map<String, Logger>();

  factory Logger(name) {
    if (_cache.containsKey(name)) {
      return _cache[name];
    } else {
      var logger = new Logger._internal(name);
      _cache[name] = logger;
      return logger;
    }
  }

  void log(String content) {
    print(content);
  }
}
```

### Getters and setters

Getters 和 setters 是用来设置和访问对象属性的特殊 函数。每个实例变量都隐含的具有一个 getter， 如果变量不是 final 的则还有一个 setter。 你可以通过实行 getter 和 setter 来创建新的属性， 使用 get 和 set 关键字定义 getter 和 setter：

```dart
class Rectangle {
  num left;
  num top;
  num width;
  num height;

  Rectangle(this.left, this.top, this.width, this.height);

  num get right => left + width;

  set right(value) => left = value - width;

  num get bottom => height + top;

  set bottom(value) => top = value - height;
}
```
getter 和 setter 的好处是，可以开始使用实例变量，后来可以把实例变量用函数包裹起来，而调用代码的地方不需要修改。

## 定义函数

### Instance methods（实例函数）

```dart
class Point {
  num a;
  num b;
  num square;

  Point(this.a, this.b);

  num diffSqrt(Point other) {
    var diffA = a - other.a;
    var diffB = b - other.b;
    return diffA * diffB;
  }
}  
```

### Abstract methods（抽象函数）

实例函数、 getter、和 setter 函数可以为抽象函数， 抽象函数是只定义函数接口但是没有实现的函数，由子类来 实现该函数。如果用分号来替代函数体则这个函数就是抽象函数。

```dart
abstract class Docoer {
  void decor();
}

class MyDecoer extends Docoer {
  @override
  void decor() {
    
  }
}
```

### Overridable operators（可覆写的操作符）

具体可以复写的操作符如下：

|<|+|\||[]|
|:-|:-|:-|:-|
|>|/|^|[]=|
|<=|~/|&|~|
|>=|*|<<|==|
|-|%|>>||

如：

```dart
class MyPoint {
  num a;
  num b;

  MyPoint(this.a, this.b);

  MyPoint operator +(MyPoint point) {
    return new MyPoint(a + point.a, b + point.b);
  }
}
```

如果覆写了 `==` ，则还应该覆写对象的 `hashCode getter` 函数。

```dart
class Person {
  final String firstName, lastName;

  Person(this.firstName, this.lastName);

  // Override hashCode using strategy from Effective Java,
  // Chapter 11.
  int get hashCode {
    int result = 17;
    result = 37 * result + firstName.hashCode;
    result = 37 * result + lastName.hashCode;
    return result;
  }

  // You should generally implement operator == if you
  // override hashCode.
  bool operator ==(other) {
    if (other is! Person) return false;
    Person person = other;
    return (person.firstName == firstName &&
        person.lastName == lastName);
  }
}

main() {
  var p1 = new Person('bob', 'smith');
  var p2 = new Person('bob', 'smith');
  var p3 = 'not a person';
  assert(p1.hashCode == p2.hashCode);
  assert(p1 == p2);
  assert(p1 != p3);
}
```

## Abstract classes（抽象类）

使用 abstract 修饰符定义一个 抽象类—一个不能被实例化的类。 抽象类通常用来定义接口，以及部分实现。如果希望你的抽象类 是可示例化的，则定义一个工厂构造函数。

抽象类通常具有抽象函数。

```dart
abstract class Docoer {
  void decor();
}

class MyDecoer extends Docoer {
  @override
  void decor() {}
}
```

## Interfaces（接口）

每个类都隐式的定义了一个包含所有实例成员的接口， 并且这个类实现了这个接口。如果你想 创建类 A 来支持 类 B 的 api，而不想继承 B 的实现， 则类 A 应该实现 B 的接口。
一个类可以通过 implements 关键字来实现一个或者多个接口， 并实现每个接口定义的 API。

```dart
class People {
  final _name;

  People(this._name);

  String greet(who) => "hello, $who, i am $_name";
}

class Qfxl implements People {
  final _name = "";// 这句必须要有

  @override
  String greet(who) {
    return 'Hi $who. Do you know who I am?';;
  }
}
```

## 继承

```dart

class Human {
  void breath(){

  }
}

class Man extends Human {
  @override
  void breath() {
    super.breath();
  }
}
```

## 枚举
    
```dart
enum Color {
  red,
  green,
  blue
}
```

枚举类型中的每个值都有一个 `index` getter 函数， 该函数返回该值在枚举类型定义中的位置（从 0 开始）。 例如，第一个枚举值的位置为 0， 第二个为 1。

```dart
 var r = Color.red;
 print(r.index); //0
```

枚举的 `values` 常量可以返回 所有的枚举值。

```dart
List<Color> colors = Color.values;
print(colors);
```

输出：

```dart
[Color.red, Color.green, Color.blue]
```

可以在 switch 语句 中使用枚举。 如果在 `switch (e)` 中的 e 的类型为枚举类， 如果没有处理所有该枚举类型的值的话，则会抛出一个警告。

```dart
enum Color {
  red,
  green,
  blue
}
// ...
Color aColor = Color.blue;
switch (aColor) {
  case Color.red:
    print('Red as roses!');
    break;
  case Color.green:
    print('Green as grass!');
    break;
  default: // Without this, you see a WARNING.
    print(aColor);  // 'Color.blue'
}
```

## 拓展类（为类添加新功能）

`Mixins` 是一种在多类继承中重用一个类代码的方法。
使用 `with` 关键字后面为一个或者多个 `mixin` 名字来使用 `mixin`。

定义一个类继承 `Object`，该类没有构造函数，不能调用 super ，则该类就是一个 `mixin`。例如：

```dart
abstract class Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

拓展类

```dart
class Musician extends Performer with Musical {
  // ...
}
```

## 静态变量与函数

### 静态变量

```dart
void main() {
  assert(Color.red.name == "red");
}

class Color {
  static const red = const Color("red");
  final String name;

  const Color(this.name);
}
```

**静态变量在第一次使用的时候才被初始化。**

### 静态函数

```dart
void main() {
  Color.printColor();//输出qfxl
}

class Color {
  static const red = const Color("red");
  final String name;

  const Color(this.name);

  static void printColor() {
    print("qfxl");
  }
}
```

**对于通用的或者经常使用的静态函数，考虑 使用顶级方法而不是静态函数。**

静态函数还可以当做编译时常量使用。例如，可以把静态函数当做常量构造函数的参数来使用。
