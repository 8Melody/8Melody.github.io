---
layout: post
title: "ConstraintLayout小知识汇总"
subtitle: "常用的小知识"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - ConstraintLayout
  - Android
---

本文记录ConstraintLayout使用中的一些小知识点。当前使用的ConstraintLayout版本为：

```kotlin
 implementation "androidx.constraintlayout:constraintlayout:2.1.0"
```

# 基线对齐

![baseline](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_baseline.jpg)


```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/tv_price"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="199.99"
        android:textSize="24sp"
        android:textColor="@color/black"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="¥"
        android:layout_marginStart="4dp"
        android:textSize="16sp"
        android:textColor="#333333"
        app:layout_constraintStart_toEndOf="@id/tv_price"
        app:layout_constraintBaseline_toBaselineOf="@id/tv_price"/>
</androidx.constraintlayout.widget.ConstraintLayout>
```

# 宽高比

比如常见的16:9宽高比。

![ratio](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_ratio.jpg)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <View
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintDimensionRatio="H,16:9"
        android:background="@color/purple_200"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>
</androidx.constraintlayout.widget.ConstraintLayout>
```

# 流式布局

> Flow VirtualLayout. Added in 2.0 Allows positioning of referenced widgets horizontally or vertically, similar to a Chain. The elements referenced are indicated via constraint_referenced_ids, as with other ConstraintHelper implementations.

水平

![flow_horizontal](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_flow_hor.jpg)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.constraintlayout.helper.widget.Flow
        android:id="@+id/flow"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:constraint_referenced_ids="tv_tel,tv_dial,tv_email,tv_sms,tv_wechat"
        app:flow_horizontalAlign="start"
        app:flow_horizontalGap="8dp"
        app:flow_maxElementsWrap="3"
        app:flow_verticalGap="8dp"
        app:flow_wrapMode="aligned"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/tv_tel"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="telphone" />

    <Button
        android:id="@+id/tv_dial"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="dial" />

    <Button
        android:id="@+id/tv_email"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="email" />

    <Button
        android:id="@+id/tv_sms"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="sms" />


    <Button
        android:id="@+id/tv_wechat"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="wechat" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

垂直

![flow_vertical](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_flow_ver.jpg)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.constraintlayout.helper.widget.Flow
        android:id="@+id/flow"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:constraint_referenced_ids="tv_tel,tv_dial,tv_email,tv_sms,tv_wechat"
        app:flow_horizontalAlign="start"
        app:flow_horizontalGap="8dp"
        app:flow_maxElementsWrap="3"
        app:flow_verticalGap="8dp"
        app:flow_wrapMode="aligned"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/tv_tel"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="telphone" />

    <Button
        android:id="@+id/tv_dial"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="dial" />

    <Button
        android:id="@+id/tv_email"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="email" />

    <Button
        android:id="@+id/tv_sms"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="sms" />


    <Button
        android:id="@+id/tv_wechat"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="wechat" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

flow_wrapMode效果如下：

![flow_wrapMode](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/flow_wrapmode.gif)

具体参阅[Flow](https://developer.android.com/reference/androidx/constraintlayout/helper/widget/Flow)

# 角度约束

参阅[18年的博客](https://blog.csdn.net/xuyonghong1122/article/details/82185347)。

# 参考线（Guideline）

![guideline](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_guideline.jpg)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_vertical"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintGuide_percent="0.5" />

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline_horizontal"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.5" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginBottom="8dp"
        android:text="Hello World"
        app:layout_constraintBottom_toTopOf="@id/guideline_horizontal"
        app:layout_constraintEnd_toStartOf="@+id/guideline_vertical" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

# 屏障（Barrier）

> A Barrier references multiple widgets as input, and creates a virtual guideline based on the most extreme widget on the specified side. For example, a left barrier will align to the left of all the referenced views.

`Barrier`跟`Guideline`的区别是`Barrier`更加灵活，它会根据referenced_ids的宽高变化而变化，而`Guideline`需要提前定义好尺寸。

![barrier](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_barrier.jpg)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/tv_left"
        android:layout_width="wrap_content"
        android:layout_height="100dp"
        android:gravity="center"
        android:textSize="24sp"
        android:textColor="@color/black"
        android:text="left"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />


    <androidx.constraintlayout.widget.Barrier
        android:id="@+id/barrier"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:barrierDirection="bottom"
        app:constraint_referenced_ids="tv_left" />

    <TextView
        android:id="@+id/tv_right"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:textSize="24sp"
        android:textColor="@color/black"
        android:text="lorem_ipsum"
        app:layout_constraintTop_toBottomOf="@+id/barrier"
        app:layout_constraintStart_toStartOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

常见的登录页使用Barrier实现，以最长文案为参考点。

![barrier](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_barrier2.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/tv_account"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="用户名"
        android:textColor="@color/black"
        android:textSize="20sp"
        app:layout_constraintBottom_toTopOf="@id/tv_password"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/tv_password"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:text="密码"
        android:textColor="@color/black"
        android:textSize="20sp"
        app:layout_constraintStart_toStartOf="@id/tv_account"
        app:layout_constraintTop_toBottomOf="@id/tv_account" />

    <androidx.constraintlayout.widget.Barrier
        android:id="@+id/barrier"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:barrierDirection="right"
        app:constraint_referenced_ids="tv_account" />

    <EditText
        android:id="@+id/et_account"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:inputType="text"
        android:textColor="@color/black"
        android:textSize="20sp"
        app:layout_constraintBottom_toBottomOf="@id/tv_account"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/barrier"
        app:layout_constraintTop_toTopOf="@id/tv_account"
        tools:text="HelloWorld" />

    <EditText
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:inputType="textPassword"
        android:textColor="@color/black"
        android:textSize="20sp"
        app:layout_constraintBottom_toBottomOf="@id/tv_password"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/barrier"
        app:layout_constraintTop_toTopOf="@id/tv_password"
        tools:text="123456" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

# 等分

![constraintlayout_weight](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_weight.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <View
        android:id="@+id/view_1"
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:background="@color/purple_200"
        app:layout_constraintEnd_toStartOf="@id/view_2"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintWidth_percent="0.3" />

    <View
        android:id="@+id/view_2"
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:background="@color/purple_500"
        app:layout_constraintEnd_toStartOf="@id/view_3"
        app:layout_constraintStart_toEndOf="@id/view_1"
        app:layout_constraintTop_toTopOf="@id/view_1"
        app:layout_constraintWidth_percent="0.3" />

    <View
        android:id="@+id/view_3"
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:background="@color/purple_700"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/view_2"
        app:layout_constraintTop_toTopOf="@id/view_2"
        app:layout_constraintWidth_percent="0.3" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

此外`layout_constraintHorizontal_weight`同样也能实现等分效果，类似LinearLayout的weight功能。

# Chains

> Chains provide group-like behavior in a single axis (horizontally or vertically). The other axis can be constrained independently.

以Horizontal为例，chainStyle分为

* spread（默认方式）
* packed
* spread_inside

## spread

![spread](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_spread.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <View
        android:id="@+id/view_1"
        android:layout_width="100dp"
        android:layout_height="100dp"
        app:layout_constraintHorizontal_chainStyle="spread"
        android:background="@color/purple_200"
        app:layout_constraintEnd_toStartOf="@id/view_2"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <View
        android:id="@+id/view_2"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="@color/purple_500"
        app:layout_constraintEnd_toStartOf="@id/view_3"
        app:layout_constraintStart_toEndOf="@id/view_1"
        app:layout_constraintTop_toTopOf="@id/view_1" />

    <View
        android:id="@+id/view_3"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="@color/purple_700"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/view_2"
        app:layout_constraintTop_toTopOf="@id/view_2" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

## packed

![packed](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_packed.png)

## spread_inside

![spread_inside](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_spread_inside.png)

# Layer

> Control the visibility and elevation of the referenced views.

![layer](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_layer.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.constraintlayout.helper.widget.Layer
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/drawable_text_bg"
        app:constraint_referenced_ids="tv_title" />

    <TextView
        android:id="@+id/tv_title"
        android:layout_width="wrap_content"
        android:layout_height="0dp"
        android:gravity="center"
        android:padding="16dp"
        android:text="Hello World"
        android:textColor="@color/white"
        android:textSize="24sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintDimensionRatio="1:1"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

将Layer放到最后。

![layer_1](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_layer_1.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/tv_title"
        android:layout_width="wrap_content"
        android:layout_height="0dp"
        android:gravity="center"
        android:padding="16dp"
        android:text="Hello World"
        android:textColor="@color/white"
        android:textSize="24sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintDimensionRatio="1:1"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <androidx.constraintlayout.helper.widget.Layer
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/drawable_text_bg"
        app:constraint_referenced_ids="tv_title" />
    
</androidx.constraintlayout.widget.ConstraintLayout>
```

# ImageFilterView

> An ImageView that can display, combine and filter images. Added in 2.0

> Subclass of ImageView to handle various common filtering operations

[文档地址](https://developer.android.com/reference/androidx/constraintlayout/utils/widget/ImageFilterView)

以圆形图片为例：

![ImageFilterView](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_imagefilterview.png)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.constraintlayout.utils.widget.ImageFilterView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:roundPercent="1" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

# 宽度自适应

![ellipsize](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_ellipsize.jpg)

![ellipsize1](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_ellipsize1.jpg)

这个需求可能经常会遇到，文字带背景，当内容少的时候宽度自适应，当内容很多的时候文尾用...代替。

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:maxLines="1"
        android:ellipsize="end"
        android:padding="8dp"
        android:text="This is long paragraph, This is long paragraph, This is long paragraph, This is long paragraph"
        android:background="@color/purple_200"
        android:textColor="@color/white"
        app:layout_constraintHorizontal_bias="0"
        app:layout_constraintWidth_max="wrap"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintEnd_toEndOf="parent"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

需要用的属性。

> app:layout_constraintHorizontal_bias="0"      app:layout_constraintWidth_max="wrap"

## Bias

The default when encountering such opposite constraints is to center the widget; but you can tweak the positioning to favor one side over another using the bias attributes:

如：

```xml
  <View
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="@color/purple_200"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
```

![bias](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_bias.png)

默认是居中的情况，通过修改bias可以自定义左右边距占比。

比如左边30%，而不是50%。

![bias1](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_bias1.png)

```xml
   <View
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="@color/purple_200"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.3"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

```

## layout_constraintWidth_max

设置为`wrap`内容自适应，但是允许视图比约束要求更小。

进阶效果：

![ellipsize2](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_ellipsize2.jpg)

![ellipsize3](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/constraintlayout_ellipsize3.jpg)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/tv_nick_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ellipsize="end"
        android:maxLines="1"
        android:text="This is a nick name"
        app:layout_constraintEnd_toStartOf="@id/iv_avatar"
        app:layout_constraintHorizontal_bias="0"
        app:layout_constrainedWidth="true"
        app:layout_constraintHorizontal_chainStyle="packed"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ImageView
        android:id="@+id/iv_avatar"
        android:layout_width="24dp"
        android:layout_height="24dp"
        android:layout_marginStart="8dp"
        android:src="@mipmap/ic_launcher"
        app:layout_constraintBottom_toBottomOf="@id/tv_nick_name"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/tv_nick_name"
        app:layout_constraintTop_toTopOf="@id/tv_nick_name" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

[更加详细的文档。](https://developer.android.com/reference/androidx/constraintlayout/widget/ConstraintLayout#developer-guide)