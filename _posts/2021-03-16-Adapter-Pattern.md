---
layout: post
title: "设计模式-适配器模式"
subtitle: "适配器模式"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - Design
  - Adapter
---

> -wiki百科

在设计模式中，适配器模式（英语：adapter pattern）有时候也称包装样式或者包装（英语：wrapper）。将一个类的接口转接成用户所期待的。一个适配使得因接口不兼容而不能在一起工作的类能在一起工作，做法是将类自己的接口包裹在一个已存在的类中。

**首先要注意的是适配器模式不适合在设计阶段使用，更多的是用于解决已存在项目中遇到的不兼容的问题。**

**优点：**
* 可以让两个没有关联的类融合在一起。
* 提高类的复用，增加类的透明度。
* 灵活。

**缺点：**
* 适配器让代码的可读性变差，比如实现A的代码，在适配器中变成了另外一个实现，如果项目中用了很多的适配器模式，那阅读起来就是一场灾难。

适配器模式中有3个角色。

* 目标角色
* 适配器角色
* 源角色

通过适配器角色将源角色API适配成目标角色的API。

适配器根据结构分`类适配器模式`和`对象适配器模式`。

# 类适配器模式

这种适配器模式下，适配器继承自己实现的类（一般多重继承）。

UML图：

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/ClassAdapter.png)

在生活中有很多适配器模式的例子，如常用的手机充电器，手机充电器名为`电源适配器`。

如苹果祖传的5V1A“高功率”充电器，需要将220V交流电转换成5V的直流电对手机进行充电。

如果用代码实现的话：

- 准备220V的电压（源角色）。

```java
class V220 {
    protected int provide220V() {
        return 220;
    }
}
```

- 需要的电压5V。（目标角色），因为Java只能单继承，所以把目标角色抽象成接口。

```java
interface V5 {
    int provide5V();
}

```

- 电源适配器，工作就是将220V转换成5V。

```java
class V220ToV5Adapter extends V220 implements V5 {

    @Override
    public int provide5V() {
        return provide220V() / 44;
    }
}    
```

将要求电压为5V的电源适配器插到220V的插座上。

```java
 V5 v5 = new V220ToV5Adapter();
 int safetyVoltage = v5.provide5V();
 System.out.println("输出电压为" + safetyVoltage + "V");
```

> 输出电压为5V

至此电源适配器的工作完成了。

# 对象适配器模式

适配器容纳一个它包裹的类的实例,在这种情况下，适配器调用被包裹对象的实例。

UML图：
![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/ObjectAdapter.png)

电源适配器例子换个实现。

```java
class V220ToV5Adapter implements V5 {

    private final V220 v220;

    public V220ToV5Adapter(V220 v220) {
        this.v220 = v220;
    }

    @Override
    public int provide5V() {
        return v220.provide220V() / 44;
    }
}
```

开始充电：

```java
V220 v220 = new V220();
V5 v5 = new V220ToV5Adapter(v220);
int safetyVoltage = v5.provide5V();
System.out.println("输出电压为" + safetyVoltage + "V");
```

> 输出电压为5V







