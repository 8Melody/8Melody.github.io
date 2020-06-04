---
layout: post
title: "Android Q深色模式"
subtitle: "关于深色模式的一些简记"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - Android
---

# 前言

一篇晚来的记录，其实应该在3月份就可以整理出来，可还是疫情的原因，让原本就不轻松的工作变得更加繁忙起来，虽然也想过以后不写这些东西了，可是等若干年回头看，自己曾经写过的东西何尝又不是一种奇妙的感觉呢。

# 正文

在Android Q及更高的版本，Google提供了深色模式（或者叫夜间模式？），我最早在用网易云音乐、B站的时候就体验过了夜间模式，后来B站也把他们的方案开源了，[MagicaSakura](https://github.com/bilibili/MagicaSakura)，不过现在似乎也不维护了，而且对于一些自定义的控件的引用比较麻烦，在使用了一个版本之后就放弃了。扯远了，首先想一下深色模式给用户带来什么好处，我在晚上玩手机的时候如果手机特别亮会觉得眼睛很难受，这个时候如果调整深色模式就会舒服很多，这是最直接的体验。
Google是这么说的：

* 可大幅减少耗电量（具体取决于设备的屏幕技术）。
* 为弱视以及对强光敏感的用户提高可视性。
* 让所有人都可以在光线较暗的环境中更轻松地使用设备。

好处还挺多的，那整吧。

# 在应用中支持深色模式

如果要适配深色模式，首页APP的主题需要这么设置。

```xml
<style name="AppTheme" parent="Theme.AppCompat.DayNight">
```

如果是Materia主题的话。

```xml
<style name="AppTheme" parent="Theme.MaterialComponents.DayNight">
```

这会将应用的主要主题背景与系统控制的夜间模式标记相关联，并将应用的默认主题背景设置为深色主题背景（如果已启用）。

举个最简单的例子：

新建一个空项目，让主题指向 `Theme.AppCompat.DayNight.NoActionBar`。

```xml
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.DayNight.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

</resources>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="?android:attr/colorBackground"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        android:textColor="?android:attr/textColorPrimary"
        android:textSize="40sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

默认情况下：

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/darkmode_day.jpg)

开启深色模式：

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/darkmode_night.jpg)


* `?android:attr/colorBackground` 默认的主题背景色。
* `?android:attr/textColorPrimary` 通用型文本颜色。它在浅色主题背景下接近于黑色，在深色主题背景下接近于白色。

# 更改应用内主题背景

应用在 Android 10 (API 级别 29) 及更高版本上运行时，推荐的选项有所不同，目的是允许用户替换系统默认设置：

* 浅色
* 深色
* 系统默认（推荐的默认选项）

在不进行任何设置的时候，应用会跟随系统进行主题设置，如果系统当前处于深色模式下，则应用会开启深色主题模式。
除此之外，应用也可以让用户主动选择当前的主题模式，主动切换主题的代码如下：

```java
 AppCompatDelegate.setDefaultNightMode(mode)
```

其中mode的对应：

* `AppCompatDelegate.MODE_NIGHT_NO` 浅色模式。
* `AppCompatDelegate.MODE_NIGHT_YES` 深色模式。
* `AppCompatDelegate.MODE_NIGHT_FOLLOW_SYSTEM` 跟随系统。

检查当前应用主题模式。

```java
val currentNightMode = resources.configuration.uiMode and Configuration.UI_MODE_NIGHT_MASK
    when (currentNightMode) {
        Configuration.UI_MODE_NIGHT_NO -> {} // Night mode is     not active, we're using the light theme
        Configuration.UI_MODE_NIGHT_YES -> {} // Night mode is     active, we're using dark theme
    }
```

整理一下，在应用内主动设置主题的代码如下：

```java
    /**
     * 手动切换深色、浅色模式
     */
    fun changeTheme(view: View) {
        //判断当前是否为深色模式
        if (resources.configuration.uiMode and Configuration.UI_MODE_NIGHT_MASK == Configuration.UI_MODE_NIGHT_YES) {
            AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO)
        } else {
            AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES)
        }
    }

    /**
     * 设置主题跟随系统
     */
    private fun setThemeByFollowSystem() {
        AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_FOLLOW_SYSTEM)
    }
```


**注意：从 AppCompat v1.1.0 开始，setDefaultNightMode() 会自动重新创建任何已启动的 Activity。**

如果不想Activity被重新创建，可以在`Manifest`给`Activity`声明手动处理`uiMode`配置变更。

```xml
<activity
    android:name=".MyActivity"
    android:configChanges="uiMode" />
```

override `Activity#onConfigurationChanged`：

```java
 override fun onConfigurationChanged(newConfig: Configuration) {
        super.onConfigurationChanged(newConfig)
        val currentNightMode = resources.configuration.uiMode and Configuration.UI_MODE_NIGHT_MASK
        when (currentNightMode) {
            Configuration.UI_MODE_NIGHT_NO -> {} // Night mode is not active, we're using the light theme
            Configuration.UI_MODE_NIGHT_YES -> {} // Night mode is active, we're using dark theme
        }
    }
```

# Force Dark

这个在实际应用中基本不会使用的策略，此功能可让开发者快速实现深色主题背景，而无需明确设置 DayNight 主题背景。

如果应用采用浅色主题背景，则 `Force Dark` 会分析应用的每个视图，并在相应视图在屏幕上显示之前，自动应用深色主题背景。

使用的方法：

创建文件夹`values-v29`，在文件下创建`styles.xml`内容从`values`中的`styles.xml`复制，并加上一行`<item name="android:forceDarkAllowed">true</item>`。

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!--无需指向DayNight主题-->
    <style name="AppTheme" parent="Theme.AppCompat.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
        <item name="android:forceDarkAllowed">true</item>
    </style>
</resources>
```

这种策略无法做到令人满意的效果，所以不多记录，日常开发不会使用。


# 最佳实践

在实际应用中，除了需要背景颜色，还要对图片、drawable之类的进行适配，以下为适配步骤。

## 适配颜色

创建`values-night`文件夹，在文件夹下创建文件`colors.xml`。

## 适配图片

创建`drawable-night-?dpi`，在文件夹下放置对应的深色模式图片。

## 适配drawable

创建`drawable-night`，在文件夹下放置对应的深色`drawable`。

# Demo

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/forexample.png)

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/daynight_demo.gif)


[Demo地址](https://github.com/qfxl/AndroidDemos/tree/master/dark)