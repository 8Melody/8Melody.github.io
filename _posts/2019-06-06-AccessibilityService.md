---
layout: post
title: "使用AccessibilityService实现微信自动发送消息"
subtitle: "记录AccessibilityService的简易使用"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #24b94a, #38ef7d);"
tags:
  - Android
  - AccessibilityService
---

> 使用AccessibilityService实现微信自动发送消息。

# 使用AccessibilityService实现微信自动发送消息

最近公司需要一个小工具，能够通过导入`Excel`，读取 `Excel` 内容精准微信一对一发送消息，本来想过通过微信Web版模拟点击，但是这样限制有点多了，于是想到了 `Android` 上的 [AccessibilityService](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService) 来实现模拟点击功能。

## 效果图

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/AccessibilityService.gif)

导入Excel读取内容之后，发送消息的操作就能全部不需要手动操作了。

# 开始

除去文件选择、Excel的读取之外的步骤：

* 启动微信
* 监测微信首页的搜索按钮
* 复制内容填充到搜索框并点击
* 选中搜索的联系人
* 找到聊天界面的输入框并填充内容
* 找到发送按钮并点击
* 微信界面返回、搜索界面返回。

介绍[AccessibilityService](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService)

官方说明

> Accessibility services should only be used to assist users with disabilities in using Android devices and apps. 

大意就是`Accessibility services`是用于帮助那些无法正常使用Android设备及APP的残障人士准备的。

`AccessibilityService`可以在后台接收到系统的操作反馈，比如按钮点击，获取焦点等。


## 使用

使用`AccessibilityService`只需要三步。

### 继承系统AccessibilityService

```java
class WeChatAccessService : AccessibilityService() {
    override fun onInterrupt() {
        
    }

    override fun onAccessibilityEvent(event: AccessibilityEvent?) {
        
    }
}

```

### 在资源目录res下新建xml文件夹，新建accessibility-service.xml文件

```xml
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeNotificationStateChanged|typeWindowStateChanged"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:accessibilityFlags="flagDefault"
    android:canRetrieveWindowContent="true"
    android:notificationTimeout="100"/>
```

* accessibilityEventTypes：表示该服务对界面中的哪些变化感兴趣，即哪些事件通知,比如窗口打开，滑动，焦点变化，长按等.具体的值可以在[AccessibilityEvent](https://developer.android.com/reference/android/view/accessibility/AccessibilityEvent?hl=en)类中查到。如typeAllMask表示接受所有的事件通知。
* accessibilityFeedbackType：反馈方式，比如语音播放、震动。
* accessibilityFlags：辅助服务额外的flag信息，例如 `FLAG_REPORT_VIEW_IDS`可以使回调的事件带上view的ID。
* canRetrieveWindowContent：是否可以获取窗口内容。
* android:notificationTimeout：两个同样类型的辅助事件接收的最小时间间隔。
* packageNames：指明了自己的辅助服务关心哪些应用发出的事件，多个应用包名之间用逗号分隔，如果不填，则关注手机上所有应用发出的事件。如果只关注微信发出的事件，则这里填`com.tencent.mm`。

### 注册

在Manifest中注册

```xml
 <service
        android:name=".WeChatAccessService"
        android:enabled="true"
        android:exported="true"
        android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>

            <meta-data
                    android:name="android.accessibilityservice"
                    android:resource="@xml/access_service_config" />
        </service>
```

由于这个功能过于开放，所以每次使用都需要用户授权，Google的初衷是好的，但是好像国人发力的方向似乎有点问题。我做这个微信自动发消息干嘛！！

## 实现微信自动发送消息

由于每次微信更新，对应的view id都会变化，本文以微信7.0.4版本为例，查找view id可以用`Hierarchy Viewer`，能看布局id也能看布局层级。

剩下的都是一些具体的代码，没啥好记录的了。

[具体代码](https://github.com/qfxl/WxAutoController)