---
layout: post
title: "设计模式-单例模式"
subtitle: "Singleton模式"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - Design
  - Singleton
---

> wiki百科

单例模式，也叫单子模式，是一种常用的软件设计模式，属于创建型模式的一种。在应用这个模式时，单例对象的类必须保证只有一个实例存在。许多时候整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为。

优点：

* 内存只有一个实例，节约内存。
* 避免对资源的重复占用。

缺点：

* 没有接口，不能继承，与单一职责原则冲突。

单例实现方法：

# 懒汉式，线程不安全。

最基本的单例实现，不支持多线程。

```java
public class LazyUnSafeSingle {
    private static LazyUnSafeSingle instance;

    private LazyUnSafeSingle() { }

    public static LazyUnSafeSingle getInstance() {
        if (instance == null) {
            instance = new LazyUnSafeSingle();
        }
        return instance;
    }
}
```

# 懒汉式，线程安全。

因为通过`synchronized`加锁，所以效率不高。

```java
public class LazySafeSingle {
    private static LazySafeSingle instance;

    private LazySafeSingle() {
    }

    public synchronized static LazySafeSingle getInstance() {
        if (instance == null) {
            instance = new LazySafeSingle();
        }
        return instance;
    }
}
```

# 饿汉式，线程安全。

比较常用的一种方式，但是容易产生垃圾对象，没有加锁，所以效率更高，但是类加载的时候就初始化了，浪费内存。

```java
public class HungrySingle {
    private static HungrySingle instance = new HungrySingle();

    private HungrySingle() {
    }

    public static HungrySingle getInstance() {
        return instance;
    }
}
```

# 双重校验锁模式，线程安全。

这种采用双锁机制，安全且在多线程中情况下性能高效，不过实现起来比较复杂。

```java
public class DCLSingle {
    private volatile static DCLSingle instance;

    private DCLSingle() {
    }

    public static DCLSingle getInstance() {
        if (instance == null) {
            synchronized (DCLSingle.class) {
                if (instance == null) {
                    instance = new DCLSingle();
                }
            }
        }
        return instance;
    }
}
```

# 静态内部类模式，线程安全。

这种方式能达到双重校验锁模式一样的效果，但实现更简单，静态域使用延迟初始化。
线程安全通过classloader机制来保证初始化instance的时候只有一个线程。

```java
public class StaticSingle {
    private static class StaticSingleHolder {
        private static final StaticSingle instance = new StaticSingle();
    }

    private StaticSingle() {
    }

    public static StaticSingle getInstance() {
        return StaticSingleHolder.instance;
    }
}
```

# 枚举模式，线程安全。

这种是单例的最佳模式，这种方式更加简洁，自动支持序列化机制，绝对防止多次实例化，同时这也是Effective Java作者推荐的模式。

```java
public enum EnumSingle {
    INSTANCE;

    public void test() {
        System.out.println("Hello World");
    }
}
```

**除了枚举模式，其他方式的单例都能通过反射机制来破解。** 为了防止反射侵入破解，可以在构造方法进行一次实例判断。

如第一种：

```java
class LazyUnSafeSingle {
    private static LazyUnSafeSingle instance;

    private LazyUnSafeSingle() {
        //防止杯反射调用，多次调用抛出异常
        if (instance != null) {
            throw new RuntimeException();
        }
    }

    public static LazyUnSafeSingle getInstance() {
        if (instance == null) {
            instance = new LazyUnSafeSingle();
        }
        return instance;
    }
}
```

