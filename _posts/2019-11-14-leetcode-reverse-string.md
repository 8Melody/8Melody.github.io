---
layout: post
title: "leetcode字符串反转"
subtitle: "leetcode入门算法。"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - leetcode
---

# 字符串反转

题目要求：
编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 char[] 的形式给出。

不要给另外的数组分配额外的空间，**你必须原地修改输入数组、使用 O(1) 的额外空间解决这一问题。**

你可以假设数组中的所有字符都是 ASCII 码表中的可打印字符。

示例：

```Java
输入：["h","e","l","l","o"]
输出：["O","l","l","e","h"]
```

分析：

直接从两头往中间走，同时交换两边的字符。

解法如下：

```Java
private void reverseString() {
	String str = "hello";
	char[] c = str.toCharArray();
	int i = 0;
	int j = c.length-1;
	while (i < j) {
		char temp = c[i];
		c[i] = c[j];
		c[j] = temp;
		i++;
		j--;
	}
	System.out.println(c);
}
```
