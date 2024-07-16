---
layout: post
title: "设计模式-工厂模式"
subtitle: "Factory模式"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - Design
  - Factory
---

# 工厂模式

> -wiki百科

工厂方法模式（英语：Factory method pattern）是一种实现了“工厂”概念的面向对象设计模式。就像其他创建型模式一样，它也是处理在不指定对象具体类型的情况下创建对象的问题。工厂方法模式的实质是“定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例化推迟到子类中进行。”


**优点：**
* 屏蔽了类的实现过程，用户只用关心类的接口。
* 拓展性高，需要增加新的实现只需要增加一个工厂类即可。

**缺点：**
* 代码变得复杂，类的依赖变得模糊。

例子：
买奶茶的时候，会选择大杯、中杯、小杯。


```java
public interface CupSize {
    void fillCup();
}

public class LargeCup implements CupSize {

    @Override
    public void fillCup() {
        System.out.println("大杯");
    }
}

public class MediumCup implements CupSize {

    @Override
    public void fillCup() {
        System.out.println("中杯");
    }
}

public class SmallCup implements CupSize {

    @Override
    public void fillCup() {
        System.out.println("小杯");
    }
}

public class CupSizeFactory {
    public CupSize getCupSize(String cupName) {
        if (cupName == null) {
            return null;
        }
        if (cupName.equalsIgnoreCase("大杯")) {
            return new LargeCup();
        } else if (cupName.equalsIgnoreCase("中杯")) {
            return new MediumCup();
        } else if (cupName.equalsIgnoreCase("小杯")) {
            return new SmallCup();
        } else {
            return null;
        }
    }
}
```

来杯大杯的奶茶。


```java
CupSizeFactory mCupSizeFactory = new CupSizeFactory();
CupSize large = mCupSizeFactory.getCupSize("大杯");
large.fillCup();//大杯
```

# 抽象工厂模式

> wiki百科

抽象工厂模式（英语：Abstract factory pattern）是一种软件开发设计模式。抽象工厂模式提供了一种方式，可以将一组具有同一主题的单独的工厂封装起来。在正常使用中，客户端程序需要创建抽象工厂的具体实现，然后使用抽象工厂作为接口来创建这一主题的具体对象。客户端程序不需要知道（或关心）它从这些内部的工厂方法中获得对象的具体类型，因为客户端程序仅使用这些对象的通用接口。抽象工厂模式将一组对象的实现细节与他们的一般使用分离开来。

例：
喝奶茶的时候不止要选择杯子的大小，有时候要调一下甜度，这个时候就可以使用抽象工厂来实现。

奶茶需要甜度及杯量。

```java
public abstract class MilkTeaAbstractFactory {

    abstract CupSize getCupSize(String cupName);

    abstract Sweetness getSweetness(String sweetName);

}
```

再来两个工厂，一个用来确定甜度，一个用来确定杯量。

甜度：

```java
public class SweetnessFactory extends MilkTeaAbstractFactory {

    @Override
    CupSize getCupSize(String cupName) {
        return null;
    }

    @Override
    public Sweetness getSweetness(String sweetName) {
        if (sweetName == null) {
            return null;
        }
        if (sweetName.equalsIgnoreCase("全糖")) {
            return new FullSweet();
        } else if (sweetName.equalsIgnoreCase("7分糖")) {
            return new SeventySweet();
        } else if (sweetName.equalsIgnoreCase("无糖")) {
            return new NoneSweet();
        } else {
            return null;
        }
    }
}

public interface Sweetness {
    void fillSugar();
}

public class FullSweet implements Sweetness {

    @Override
    public void fillSugar() {
        System.out.println("全糖");
    }
}

public class SeventySweet implements Sweetness {

    @Override
    public void fillSugar() {
        System.out.println("七分糖");
    }
}

public class NoneSweet implements Sweetness {

    @Override
    public void fillSugar() {
        System.out.println("无糖");
    }
}
```

杯量：

```java
public class CupSizeFactory extends MilkTeaAbstractFactory {

    @Override
    public CupSize getCupSize(String cupName) {
        if (cupName == null) {
            return null;
        }
        if (cupName.equalsIgnoreCase("大杯")) {
            return new LargeCup();
        } else if (cupName.equalsIgnoreCase("中杯")) {
            return new MediumCup();
        } else if (cupName.equalsIgnoreCase("小杯")) {
            return new SmallCup();
        } else {
            return null;
        }
    }

    @Override
    Sweetness getSweetness(String sweetName) {
        return null;
    }
}

public interface CupSize {
    void fillCup();
}

public class LargeCup implements CupSize {

    @Override
    public void fillCup() {
        System.out.println("大杯");
    }
}

public class MediumCup implements CupSize {

    @Override
    public void fillCup() {
        System.out.println("中杯");
    }
}

public class SmallCup implements CupSize {

    @Override
    public void fillCup() {
        System.out.println("小杯");
    }
}

```

奶茶工厂：

```java
class MilkTeaFactory {
    public static MilkTeaAbstractFactory getFactory(String factoryName) {
        if (factoryName == null) {
            return null;
        }
        if (factoryName.equalsIgnoreCase("cup")) {
            return new CupSizeFactory();
        } else if (factoryName.equalsIgnoreCase("sweetness")) {
            return new SweetnessFactory();
        } else {
            return null;
        }
    }
}
```

来一杯全糖、大杯奶茶。

```java
MilkTeaAbstractFactory mCupFactory = MilkTeaFactory.getFactory("cup");
MilkTeaAbstractFactory mSweetnessFactory = MilkTeaFactory.getFactory("sweetness");
CupSize mSize = mCupFactory.getCupSize("大杯");
Sweetness mSweet = mSweetnessFactory.getSweetness("全糖");
mSize.fillCup();
mSweet.fillSugar();
```