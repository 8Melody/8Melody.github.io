---
layout: post
title: "leetcode之两数之和"
subtitle: "leetcode two sum"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - leetcode
---

> leetcode经典算法之两数之和。

# 两数之和

[该题为leetcode入门算法之一。][1]

给定一个int数组`nums`，给定一个指定值`target`，在数组中找到2个数的下标，它们之和等于指定值`target`，数组`nums`的数据不能重复使用，并且指定值`target`在数组中任意数据之和只有一种组合。

```kotlin
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1]
```

首先分析该题：

数组中元素不会重复；
不能重复使用相同元素；
有这两个做前提下这题变得比较简单起来，这里用两种方案来实现。

## 方法一

使用“选择相加法”，这也是最容易想到的办法，选择某一个元素，然后与其他元素分别相加，如果结果等于`target`，则返回这两个数的角标。

```java
private int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[i] + nums[j] == target) {
                    return new int[]{i, j};
                }
            }
        }
        return null;
    }
```

## 方法二

利用一个Map存储`nums`的信息，key为nums里面的元素，value为其对应的角标。再使用一个变量`diff`去记住`target`跟当前元素的差值，最后去Map里查找这个差值，如果能找到，则返回。

```java
private int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> record = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            record.put(nums[i], i);
        }
        int diff;
        for (int i = 0; i < nums.length; i++) {
            diff = target - nums[i];
            if (record.containsKey(diff) && record.get(diff) != i) {
                return new int[]{i, record.get(diff)};
            }
        }
        return null;
    }
```

  [1]: https://leetcode.com/problems/two-sum/