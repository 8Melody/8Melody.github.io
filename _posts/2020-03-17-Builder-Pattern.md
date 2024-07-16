---
layout: post
title: "设计模式-Builder模式"
subtitle: "Builder模式"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - Design
  - Builder
---

> -wiki百科

生成器模式（英：Builder Pattern）是一种设计模式，又名：建造模式，是一种对象构建模式。它可以将复杂对象的建造过程抽象出来（抽象类别），使这个抽象过程的不同实现方法可以构造出不同表现（属性）的对象。

在Android SDK里有很多地方也使用了Builder模式，如经典的`AlertDialog#Builder`。

**优点：**
* 建造者独立，易扩展。

**缺点：**
* 构造对象需要有共同点，范围有限制。

例子：
麦当劳点餐的时候有个套餐，`红白 1+1`，红区选择1个商品，白区选择1个商品。

当然这是个简单的例子，但是如果要选5-6个类型的商品的时候，使用Builder模式的好处就能更加体现出来了。


```java
class McDonaldOrder {

    private final String white;
    private final String red;

    private McDonaldOrder(Builder builder) {
        this.white = builder.white;
        this.red = builder.red;
    }

    public String getRed() {
        return red;
    }

    public String getWhite() {
        return white;
    }

    static class Builder {
        private String white;
        private String red;

        public Builder withWhite(String white) {
            this.white = white;
            return this;
        }

        public Builder withRed(String red) {
            this.red = red;
            return this;
        }

        public McDonaldOrder build() {
            if (red == null || white == null) {
                throw new NullPointerException("套餐务必选择1红1白");
            }
            return new McDonaldOrder(this);
        }

    }
}
```

选择套餐。

```java
 McDonaldOrder mOrder = new McDonaldOrder.Builder()
                .withWhite("薯条")
                .withRed("可乐")
                .build();
```






