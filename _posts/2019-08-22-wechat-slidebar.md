---
layout: post
title: "微信小程序滑块验证"
subtitle: "本文记录微信小程序滑块验证功能"
author: "XYH"
header-img: "img/wechat-mina.jpg"
header-mask: 0.4
tags:
  - Wechat
---

>本文记录微信小程序实现滑块验证。

# 前言

最近做小程序的时候有个需求要求用户登录之前需要滑块验证，这个在很多网页上面也有使用防止模拟登录或者限制登录频率。

本文记录实现的思路，基本的思路是使用微信提供的[movable-area](https://developers.weixin.qq.com/miniprogram/dev/component/movable-area.html)配合[movable-view](https://developers.weixin.qq.com/miniprogram/dev/component/movable-view.html)。

具体的使用方法参考文档。
实现步骤如下：

* 记录滑动的`x`并且判断`x`是否到达了滑动的边界，如果未到边界则将`x`置为0，反之设为边界的`x`。

最简单的效果图如下：

![基础demo](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/wechat_slide_bar.gif)


如果稍加修饰的话，可实现下面的效果：

![精修](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/wechat_login_slide_bar.gif)

本文以最简单的效果为例。

## wxml

```xml
<view class="container">
  <movable-area class="move-area">
    <movable-view class="move-view" inertia="true" damping="35" x="{{blockMovedX}}" direction="horizontal" bindchange="onBlockMove" bindtouchend="onBlockMoveEnd">
    </movable-view>
  </movable-area>
  <button class="btn" type="primary" bindtap="resetBlock">重置</button>
</view>
```

## wxss
```css
.move-area{
  width: 100%;
 height: 98rpx;
  background-color: #e8f0fc;
}
.move-view{
  width: 98rpx;
  height: 98rpx;
  background-color: #1b6fe1;
}
.btn{
  width: 100%;
  margin-top: 90rpx;
}
```

## js
```javascript
Page({

  /**
   * 页面的初始数据
   */
  data: {
    movedX: 0,
    blockMovedX: 0,
    moveBlockWidth: 0,
    moveAreaWidth: 0
  },

  onBlockMove(e) {
    let moveX = e.detail.x;
    this.setData({
      movedX: moveX
    })
  },
  onBlockMoveEnd() {
    let areaWidth = this.data.moveAreaWidth;
    let blockWidth = this.data.moveBlockWidth;
    let isFinished = this.data.isFinished;
    if (areaWidth == 0 || blockWidth == 0 || isFinished) {
      return;
    }
    let maxX = areaWidth - blockWidth;
    if (this.data.movedX < maxX) {
      this.setData({
        blockMovedX: 0
      })
    } else {
      this.setData({
        blockMovedX: maxX
      })
      wx.showToast({
        title: '验证成功',
        icon: "none"
      })
    }
  },
  resetBlock() {
    this.setData({
      blockMovedX: 0
    })
  },
  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function(options) {
    var query = wx.createSelectorQuery();
    var self = this;
    query.select('.move-area').boundingClientRect(function(rect) {
      self.setData({
        moveAreaWidth: rect.width
      })
    }).exec();
    query.select('.move-view').boundingClientRect(function(rect) {
      self.setData({
        moveBlockWidth: rect.width,
      })
    }).exec();
  }
})
```

## 总结

实现效果比较简单，如果有特殊需求的话按照微信提供的文档自己进行二次开发比较容易实现。