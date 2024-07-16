---
layout: post
title: "leetcode宝石与石头"
subtitle: "leetcode经典入门算法"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - leetcode
---

> leetcode算法之宝石与石头。


# 宝石与石头

[该题为leetcode入门算法之一。][1]

给定字符串J 代表石头中宝石的类型，和字符串 S代表你拥有的石头。 S 中每个字符代表了一种你拥有的石头的类型，你想知道你拥有的石头中有多少是宝石。

J 中的字母不重复，J 和 S中的所有字符都是字母。字母区分大小写，因此"a"和"A"是不同类型的石头。

如：

```kotlin
输入: J = "aA", S = "aAAbbbb"
输出: 3
```

```kotlin
输入: J = "z", S = "ZZ"
输出: 0
```

**注意**

* S 和 J 最多含有50个字母。
* J 中的字符不重复。

## 方法一

最容易想到的办法，依次循环S，判断里面是否包含J。

```java
public static int findJewelsInStones1(String jewels, String stones) {	
	char[] jls = jewels.toCharArray();
	char[] sts = stones.toCharArray();
	int count = 0;
	for (int i = count; i < sts.length; i++) {
		for (int j = 0; j < jls.length; j++) {
			if (sts[i] == jls[j]) {
				count++;
			}
		}
	}
	return count;
}
```

## 方法二

利用`String`中的`indexOf()`方法。

```java
public static int findJewelsInStones2(String jewels, String stones) {
	int count = 0;
	for (int i = 0; i < stones.toCharArray().length; i++) {
		if (jewels.indexOf(stones.charAt(i)) != -1) {
			count++;
		}
	}
	return count;
}
```

  [1]: https://leetcode-cn.com/problems/jewels-and-stones/