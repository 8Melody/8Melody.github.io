---
layout: post
title: "RecyclerView ItemDecoration使用小记"
subtitle: "记录RecyclerView ItemDecoration的用法"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Android
  - ItemDecoration
---

> 记录RecyclerView ItemDecoration的使用方法。

# RecyclerView ItemDecoration。

> 本文记录如何使用ItemDecoration实现RecyclerView分割线跟如何使用ItemDecoration实现stickyHeader，俗称粘性Header。


## 前言

 以前使用`ListView`的时候设置分割线是一件非常简单的事情，后来Google推出了`RecyclerView`，但是`RecyclerView`并没有直接设置分割线的API，不过好在`RecyclerView`提供的接口够多，其中`ItemDecoration`接口可以用于干预`RecyclerView`内部itemView绘制的api。
 
## 介绍ItemDecoration
 
 `ItemDecoration`需要实现的方法只有3个：
 
 ```java
  public abstract static class ItemDecoration {
        /**
         * 这个方法会在RecyclerView绘制itemView之前调用，并且绘制的内容会在itemView下方。
         * @param c Canvas to draw into
         * @param parent RecyclerView this ItemDecoration is drawing into
         * @param state The current state of RecyclerView
         */
        public void onDraw(Canvas c, RecyclerView parent, State state) {
           
        }
        /**
         * 这个方法会在RecyclerView绘制itemView之后调用，并且绘制的内容会覆盖在itemView上。
         * @param c Canvas to draw into
         * @param parent RecyclerView this ItemDecoration is drawing into
         * @param state The current state of RecyclerView.
         */
        public void onDrawOver(Canvas c, RecyclerView parent, State state) {
           
        }
        /**
         * 获取每个itemView的偏移量
         * @param outRect Rect to receive the output.
         * @param view    The child view to decorate
         * @param parent  RecyclerView this ItemDecoration is decorating
         * @param state   The current state of RecyclerView.
         */        
        public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
          
        }
    }
 ```
 
 这些方法在`RecyclerView`中被调用的位置：
 
 其中`ItemDecoration#onDraw(Canvas c, RecyclerView parent, State state)`会在`RecyclerView`中
 
 ```java
  @Override
    public void onDraw(Canvas c) {
        super.onDraw(c);

        final int count = mItemDecorations.size();
        for (int i = 0; i < count; i++) {
            mItemDecorations.get(i).onDraw(c, this, mState);
        }
    }
 ```
 `ItemDecoration#onDrawOver(Canvas c, RecyclerView parent, State state)`在`RecyclerView`中的
 
 ```java    
   @Override
    public void draw(Canvas c) {
        super.draw(c);

        final int count = mItemDecorations.size();
        for (int i = 0; i < count; i++) {
            mItemDecorations.get(i).onDrawOver(c, this, mState);
        }
        xxx
    }    
 ```
 
 `ItemDecoration#getItemOffsets(Rect outRect, View view, RecyclerView parent, State state)`在`RecyclerView`中调用位置如下：
 
 ```java
 Rect getItemDecorInsetsForChild(View child) {
        final LayoutParams lp = (LayoutParams) child.getLayoutParams();
        if (!lp.mInsetsDirty) {
            return lp.mDecorInsets;
        }

        if (mState.isPreLayout() && (lp.isItemChanged() || lp.isViewInvalid())) {
            // changed/invalid items should not be updated until they are rebound.
            return lp.mDecorInsets;
        }
        final Rect insets = lp.mDecorInsets;
        insets.set(0, 0, 0, 0);
        final int decorCount = mItemDecorations.size();
        for (int i = 0; i < decorCount; i++) {
            mTempRect.set(0, 0, 0, 0);
            mItemDecorations.get(i).getItemOffsets(mTempRect, child, this, mState);
            insets.left += mTempRect.left;
            insets.top += mTempRect.top;
            insets.right += mTempRect.right;
            insets.bottom += mTempRect.bottom;
        }
        lp.mInsetsDirty = false;
        return insets;
    }
 ```
 
 
## 实现RecyclerView的分割线
 
 简述了上述几个方法之后，如何绘制分割线应该是比较轻松的事情了，好记性不如烂笔头，下次看到这里的时候不要打脸了。
 
 ```kotlin
 class StickyItemDecoration(private val context: Context) : RecyclerView.ItemDecoration() {
    /**
     * 分割线颜色
     */
    var dividerColor = Color.parseColor("#E8E8E8")
    /**
     * 分割线高度
     */
    var dividerHeight = context.dpToPixel(1)
    /**
     * 分割线画笔
     */
    private val dividerPaint = Paint(Paint.ANTI_ALIAS_FLAG).also {
        it.color = dividerColor
    }
    
    /**
     * 这个方法会在RecyclerView绘制itemView之前调用，并且绘制的内容会在itemView下方。
     * @param c Canvas to draw into
     * @param parent RecyclerView this ItemDecoration is drawing into
     * @param state The current state of RecyclerView
     */
    override fun onDraw(c: Canvas?, parent: RecyclerView, state: RecyclerView.State?) {
        super.onDraw(c, parent, state)
        val childCount = parent.childCount
        for (i in 0 until childCount) {
            val itemView = parent.getChildAt(i)
            c?.drawRect(0F, (itemView.top - dividerHeight).toFloat(), itemView.right.toFloat(),
                    itemView.top.toFloat(), dividerPaint)
        }

    }

    /**
     * 这个方法会在RecyclerView绘制itemView之后调用，并且绘制的内容会覆盖在itemView上。
     * @param c Canvas to draw into
     * @param parent RecyclerView this ItemDecoration is drawing into
     * @param state The current state of RecyclerView.
     */
    override fun onDrawOver(c: Canvas?, parent: RecyclerView, state: RecyclerView.State?) {
        super.onDrawOver(c, parent, state)
    }

    /**
     * 获取每个itemView的偏移量
     * @param outRect Rect to receive the output.
     * @param view    The child view to decorate
     * @param parent  RecyclerView this ItemDecoration is decorating
     * @param state   The current state of RecyclerView.
     */
    override fun getItemOffsets(outRect: Rect?, view: View?, parent: RecyclerView?, state: RecyclerView.State?) {
        super.getItemOffsets(outRect, view, parent, state)
        outRect?.top = dividerHeight
    }
}
 ```
 
设置`RecyclerView`的`ItemDecoration`之后，效果图如下：

![image](http://qfxl.oss-cn-shanghai.aliyuncs.com/images/divider.gif)


## 实现RecyclerView的分割线 + 分组
 
 ```kotlin
 class StickyItemDecoration(private val context: Context) : RecyclerView.ItemDecoration() {
    /**
     * 分割线颜色
     */
    var dividerColor = Color.parseColor("#E8E8E8")
    /**
     * 分割线高度
     */
    var dividerHeight = context.dpToPixel(1)
    /**
     * sticky item高度
     */
    var stickyItemHeight = context.dpToPixel(30)
    /**
     * 绘制sticky
     */
    private val stickyPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        color = 0xFFE8E8E8.toInt()
    }
    /**
     * 绘制sticky文本
     */
    private val stickyTextPaint = Paint(Paint.ANTI_ALIAS_FLAG or Paint.DITHER_FLAG).also {
        it.color = 0xFF999999.toInt()
        it.textSize = context.spToPx(16F)
    }
    /**
     * sticky文本的rect
     */
    private val stickyTextRect = Rect()
    /**
     * 分割线画笔
     */
    private val dividerPaint = Paint(Paint.ANTI_ALIAS_FLAG).also {
        it.color = dividerColor
    }
    /**
     * 获取sticky文本
     */
    private var textCallback: DividerTextCallback? = null


    init {

    }


    /**
     * 这个方法会在RecyclerView绘制itemView之前调用，并且绘制的内容会在itemView下方。
     * @param c Canvas to draw into
     * @param parent RecyclerView this ItemDecoration is drawing into
     * @param state The current state of RecyclerView
     */
    override fun onDraw(c: Canvas?, parent: RecyclerView, state: RecyclerView.State?) {
        super.onDraw(c, parent, state)
        val childCount = parent.childCount
        for (i in 0 until childCount) {
            val itemView = parent.getChildAt(i)
            val position = parent.getChildAdapterPosition(itemView)
            if (isFirstItemInGroup(position)) {
                val top = itemView.top - stickyItemHeight
                val bottom = itemView.top
                c?.drawRect(
                    0F, top.toFloat(), itemView.right.toFloat(),
                    bottom.toFloat(), stickyPaint
                )
                drawStickyText(textCallback?.getDividerText(position), c, top, bottom)
            } else {
                c?.drawRect(
                    0F, (itemView.top - dividerHeight).toFloat(), itemView.right.toFloat(),
                    itemView.top.toFloat(), dividerPaint
                )
            }
        }

    }

    /**
     * 这个方法会在RecyclerView绘制itemView之后调用，并且绘制的内容会覆盖在itemView上。
     * @param c Canvas to draw into
     * @param parent RecyclerView this ItemDecoration is drawing into
     * @param state The current state of RecyclerView.
     */
    override fun onDrawOver(c: Canvas?, parent: RecyclerView, state: RecyclerView.State?) {
        super.onDrawOver(c, parent, state)

    }

    /**
     * 获取每个itemView的偏移量
     * @param outRect Rect to receive the output.
     * @param view    The child view to decorate
     * @param parent  RecyclerView this ItemDecoration is decorating
     * @param state   The current state of RecyclerView.
     */
    override fun getItemOffsets(outRect: Rect?, view: View, parent: RecyclerView, state: RecyclerView.State?) {
        super.getItemOffsets(outRect, view, parent, state)
        val position = parent.getChildAdapterPosition(view)
        if (isFirstItemInGroup(position)) {
            outRect?.top = stickyItemHeight
        } else {
            outRect?.top = dividerHeight
        }
    }

    /**
     * 设置分组内容回调
     */
    fun setDividerTextCallback(action: (position: Int) -> String) {
        textCallback = object : DividerTextCallback {
            override fun getDividerText(position: Int): String = action(position)
        }
    }

    /**
     * 是否是分组的第一个item
     */
    private fun isFirstItemInGroup(position: Int): Boolean {
        return if (position == 0) {
            true
        } else {
            val currentText = textCallback?.getDividerText(position)
            val previewText = textCallback?.getDividerText(position - 1)
            currentText != previewText
        }
    }

    /**
     * 绘制sticky内容
     */
    private fun drawStickyText(text: String?, c: Canvas?, top: Int, bottom: Int) {
        stickyTextRect.left = context.dpToPixel(10)
        stickyTextRect.top = top
        stickyTextRect.right = stickyTextPaint.measureText(text).toInt()
        stickyTextRect.bottom = bottom
        val fontMetrics = stickyTextPaint.fontMetricsInt
        val baseline = (stickyTextRect.bottom + stickyTextRect.top - fontMetrics.bottom - fontMetrics.top) / 2
        c?.drawText(text, stickyTextRect.left.toFloat(), baseline.toFloat(), stickyTextPaint)
    }

    /**
     * 分组text
     */
    interface DividerTextCallback {
        /**
         * 获取分组text
         */
        fun getDividerText(position: Int): String
    }
}

 ```
 
 效果图：
 
 ![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/decoration.gif)
 
## 实现sticky效果
 
```kotlin
class StickyItemDecoration(private val context: Context) : RecyclerView.ItemDecoration() {
    /**
     * 分割线颜色
     */
    var dividerColor = Color.parseColor("#E8E8E8")
    /**
     * 分割线高度
     */
    var dividerHeight = context.dpToPixel(1)
    /**
     * sticky item高度
     */
    var stickyItemHeight = context.dpToPixel(30)
    /**
     * 绘制sticky
     */
    private val stickyPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        color = 0xFFE8E8E8.toInt()
    }
    /**
     * 绘制sticky文本
     */
    private val stickyTextPaint = Paint(Paint.ANTI_ALIAS_FLAG or Paint.DITHER_FLAG).also {
        it.color = 0xFF999999.toInt()
        it.textSize = context.spToPx(16F)
    }
    /**
     * sticky文本的rect
     */
    private val stickyTextRect = Rect()
    /**
     * 分割线画笔
     */
    private val dividerPaint = Paint(Paint.ANTI_ALIAS_FLAG).also {
        it.color = dividerColor
    }
    /**
     * 获取sticky文本
     */
    private var textCallback: DividerTextCallback? = null


    init {

    }


    /**
     * 这个方法会在RecyclerView绘制itemView之前调用，并且绘制的内容会在itemView下方。
     * @param c Canvas to draw into
     * @param parent RecyclerView this ItemDecoration is drawing into
     * @param state The current state of RecyclerView
     */
    override fun onDraw(c: Canvas?, parent: RecyclerView, state: RecyclerView.State?) {
        super.onDraw(c, parent, state)
        val childCount = parent.childCount
        for (i in 0 until childCount) {
            val itemView = parent.getChildAt(i)
            val position = parent.getChildAdapterPosition(itemView)
            if (isFirstItemInGroup(position)) {
                val top = itemView.top - stickyItemHeight
                val bottom = itemView.top
                c?.drawRect(
                    0F, top.toFloat(), itemView.right.toFloat(),
                    bottom.toFloat(), stickyPaint
                )
                drawStickyText(textCallback?.getDividerText(position), c, top, bottom)
            } else {
                c?.drawRect(
                    0F, (itemView.top - dividerHeight).toFloat(), itemView.right.toFloat(),
                    itemView.top.toFloat(), dividerPaint
                )
            }
        }

    }

    /**
     * 这个方法会在RecyclerView绘制itemView之后调用，并且绘制的内容会覆盖在itemView上。
     * @param c Canvas to draw into
     * @param parent RecyclerView this ItemDecoration is drawing into
     * @param state The current state of RecyclerView.
     */
    override fun onDrawOver(c: Canvas?, parent: RecyclerView, state: RecyclerView.State?) {
        super.onDrawOver(c, parent, state)
        val childCount = parent.childCount

        if (childCount > 0) {
            //sticky效果其实就是处理第一个itemView，然后让悬浮内容置于第一个itemView之上。
            val firstView = parent.getChildAt(0)

            val position = parent.getChildAdapterPosition(firstView)
            val text = textCallback?.getDividerText(position)

            if (firstView.bottom <= stickyItemHeight && isFirstItemInGroup(position + 1)) {
                c?.drawRect(0F, 0F, firstView.width.toFloat(), firstView.bottom.toFloat(), stickyPaint)
                drawStickyText(text, c, firstView.bottom - stickyItemHeight, firstView.bottom)
            } else {
                c?.drawRect(0F, 0F, firstView.width.toFloat(), stickyItemHeight.toFloat(), stickyPaint)
                drawStickyText(text, c, 0, stickyItemHeight)
            }
        }
    }

    /**
     * 获取每个itemView的偏移量
     * @param outRect Rect to receive the output.
     * @param view    The child view to decorate
     * @param parent  RecyclerView this ItemDecoration is decorating
     * @param state   The current state of RecyclerView.
     */
    override fun getItemOffsets(outRect: Rect?, view: View, parent: RecyclerView, state: RecyclerView.State?) {
        super.getItemOffsets(outRect, view, parent, state)
        val position = parent.getChildAdapterPosition(view)
        if (isFirstItemInGroup(position)) {
            outRect?.top = stickyItemHeight
        } else {
            outRect?.top = dividerHeight
        }
    }

    /**
     * 设置分组内容回调
     */
    fun setDividerTextCallback(action: (position: Int) -> String) {
        textCallback = object : DividerTextCallback {
            override fun getDividerText(position: Int): String = action(position)
        }
    }

    /**
     * 是否是分组的第一个item
     */
    private fun isFirstItemInGroup(position: Int): Boolean {
        return if (position == 0) {
            true
        } else {
            val currentText = textCallback?.getDividerText(position)
            val previewText = textCallback?.getDividerText(position - 1)
            currentText != previewText
        }
    }

    /**
     * 绘制sticky内容
     */
    private fun drawStickyText(text: String?, c: Canvas?, top: Int, bottom: Int) {
        stickyTextRect.left = context.dpToPixel(10)
        stickyTextRect.top = top
        stickyTextRect.right = stickyTextPaint.measureText(text).toInt()
        stickyTextRect.bottom = bottom
        val fontMetrics = stickyTextPaint.fontMetricsInt
        val baseline = (stickyTextRect.bottom + stickyTextRect.top - fontMetrics.bottom - fontMetrics.top) / 2
        c?.drawText(text, stickyTextRect.left.toFloat(), baseline.toFloat(), stickyTextPaint)
    }

    /**
     * 分组text
     */
    interface DividerTextCallback {
        /**
         * 获取分组text
         */
        fun getDividerText(position: Int): String
    }
}
```

效果图：

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/sticky.gif)

## 源码地址

[stickyItemDecoration](https://github.com/qfxl/AndroidDemos/tree/master/stickyItemDecoration)