---
layout: post
title: "仿抖音时钟"
subtitle: "一款类似抖音时钟的自定义View"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Android
---


# 前言
最近在抖音上看到的一个效果，觉得蛮好玩的，于是想着自己写一个，顺便复习一下自定义View。

# 效果图

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/tik-tok.gif)

# 思路

该效果稍微麻烦一点的就是3个圈的旋转控制，其余都是基础的绘制内容。
* 绘制中间文本（时间、日期、星期）
* 绘制时钟圈
* 绘制分钟圈
* 绘制秒钟圈
* 旋转
* 绘制选中框

绘制比较简单，通过`canvas.rotate()`旋转绘制时分秒，其中需要计算的点是时、分、秒的`x`坐标，效果图上绘制计算如下：

```java
private val centerX by lazy {
        (width shr 1).toFloat()
}
private val centerY by lazy {
        (height shr 1).toFloat()
}
private val textPaint by lazy {
    Paint(Paint.ANTI_ALIAS_FLAG or Paint.DITHER_FLAG).apply {
        color = Color.WHITE
        textSize = spToPx(16f)
        textAlign = Paint.Align.CENTER
    }
}
override fun onLayout(changed: Boolean, left: Int, top: Int, right: Int, bottom: Int) {
        super.onLayout(changed, left, top, right, bottom)
        val itemMaxWidth = width / 2 / 3
        hourX = centerX + itemMaxWidth + textPaint.measureText("00时")
        minuteX = centerX + itemMaxWidth * 2
        secondsX = centerX + itemMaxWidth * 3 - textPaint.measureText("00秒")
    }
```

# 绘制中间文本

```java
 /**
 * 绘制中间内容
 * @param canvas
 */
private fun drawCenterText(canvas: Canvas?) {
    textPaint.apply {
        textSize = spToPx(16f)
        textAlign = Paint.Align.CENTER
    }
    canvas?.run {
        drawText(
            currentTime,
            centerX,
            centerY + textPaint.descent() + textPaint.ascent(),
            textPaint
        )
        textPaint.textSize = spToPx(14f)
        drawText(
            currentDate,
            centerX,
            centerY - textPaint.descent() - textPaint.ascent(),
            textPaint
        )
        textPaint.apply {
            textSize = spToPx(12f)
            textAlign = Paint.Align.LEFT
        }
    }
}
```

# 绘制时

```java
/**
 * 绘制时
 * @param canvas
 */
private fun drawHour(canvas: Canvas?) {
    canvas?.run {
        for (h in 0 until 24) {
            save()
            rotate(360 / 24f * h, centerX, centerY)
            val hourText = if (h < 10) {
                "0${h}时"
            } else {
                "${h}时"
            }
            drawText(hourText, hourX, centerY, textPaint)
            restore()
        }
    }
}
```

# 绘制分

```java
/**
 * 绘制分钟
 * @param canvas
 */
private fun drawMinute(canvas: Canvas?) {
    canvas?.run {
        for (min in 0 until 60) {
            save()
            rotate(360 / 60f * min, centerX, centerY)
            val minText = if (min < 10) {
                "0${min}分"
            } else {
                "${min}分"
            }
            drawText(minText, minuteX, centerY, textPaint)
            restore()
        }
    }
}
```

# 绘制秒

```java
/**
 * 绘制秒
 * @param canvas
 */
private fun drawSeconds(canvas: Canvas?) {
    canvas?.run {
        for (sec in 0 until 60) {
            save()
            rotate(360 / 60f * sec, centerX, centerY)
            val secText = if (sec < 10) {
                "0${sec}秒"
            } else {
                "${sec}秒"
            }
            drawText(secText, secondsX, centerY, textPaint)
            restore()
        }
    }
}
```

# 绘制选中框
选中框是一个Rect，对应的计算。
```java
  private val currentTimeRect by lazy {
    val hourBounds = Rect()
    textPaint.getTextBounds("${hour}时", 0, "${hour}时".length, hourBounds)
    RectF(
        hourX - dpToPx(5),
        centerY - hourBounds.height(),
        secondsX + textPaint.measureText("00秒") + dpToPx(5),
        centerY + textPaint.descent()
    )
}
```

以上代码实现的绘制的效果如下：

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/tik-tok.png)

# 表盘转起来

转动表盘就是定时让canvas旋转一定的角度，为了实现更好的转动动画，这里采用了`ValueAnimator`。
以秒盘为例，分针跟时针转动同理。

绘制秒盘的时候增加一个旋转的参数`secondsDegree`。

```java
/**
* 绘制秒
 * @param canvas
 */
private fun drawSeconds(canvas: Canvas?) {
    canvas?.run {
        save()
        rotate(secondsDegree, centerX, centerY)
        for (sec in 0 until 60) {
            save()
            rotate(360 / 60f * sec, centerX, centerY)
            val secText = if (sec < 10) {
                "0${sec}秒"
            } else {
                "${sec}秒"
            }
            drawText(secText, secondsX, centerY, textPaint)
            restore()
        }
        restore()
    }
}
```

定时更新`secondsDegree`来实现表盘的转动。

```java
 private var mAnimator = ValueAnimator.ofFloat(0f, 6f).apply {
    interpolator = LinearInterpolator()
    duration = 400
}

private fun refresh() {
    Calendar.getInstance().run {
        hour = get(Calendar.HOUR_OF_DAY)
        minute = get(Calendar.MINUTE)
        seconds = get(Calendar.SECOND)
        secondsDegree = -360 / 60f * (seconds - 1)
    }    
    mAnimator.removeAllUpdateListeners()
    val sd = secondsDegree
    mAnimator.addUpdateListener { v ->
        val value = v.animatedValue as Float
        secondsDegree = sd - value
        invalidate()
    }
    postDelayed({
        refresh()
    }, 1000)
    mAnimator.start()
}
```
至此，秒盘已经可以实现转动效果。

## 与分针、时针联动。

秒针转到0的时候，分针进一格，分针秒针都为0的时候，时针进一格。

核心代码：
```java
mAnimator.addUpdateListener { v ->
    val value = v.animatedValue as Float
    if (seconds == 0) {
        minutesDegree = md - value
    }
    if (seconds == 0 && minute == 0) {
        hourDegree = hd - value * 60 / 24
    }
    secondsDegree = sd - value
    invalidate()
}
```

完整实现参考完整代码。

至此一个简单的抖音时钟效果就实现了，[完整代码](https://github.com/qfxl/AndroidDemos/blob/master/customview/src/main/java/com/github/customview/views/ClockView.kt)。