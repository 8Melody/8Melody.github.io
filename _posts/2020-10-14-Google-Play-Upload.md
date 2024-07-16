---
layout: post
title: "记一次Google  Play上架遇到的问题"
subtitle: "Google Play上架问题"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - Android
---

# 前言

开发的直播项目[VoLive](https://play.google.com/store/apps/details?id=com.zby.live)在上架Google Play的时候被拒了2次，记录这2个问题。

# SSL Error Handler

第一次上架的时候被拒绝，然后邮件告知：

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/google_play_ssl_error.png)

**问题**：WebView遇到SSL错误应该告知用户，让用户处理是否继续或退出。

**分析**：问题代码出现在WebView重写`WebViewClient#onReceivedSslError(view: WebView?, handler: SslErrorHandler?, error: SslError?)`，在接受到SSL错误的时候没有正确处理，一开始的写法是
```Java
override fun onReceivedSslError(
                    view: WebView?,
                    handler: SslErrorHandler?,
                    error: SslError?
                ) {
                    handler?.proceed()
                }
```
然后就被拒绝了。

**正确处理方式**：

邮件的内容大致告知，遇到SSL错误的时候需要提示用户，所以处理方式如下。

```Java
 override fun onReceivedSslError(
                    view: WebView?,
                    handler: SslErrorHandler?,
                    error: SslError?
                ) {
                    AlertDialog.Builder(context).apply {
                        val errorMsg = when (error?.primaryError) {
                            SslError.SSL_UNTRUSTED -> "The certificate authority is not trusted."
                            SslError.SSL_EXPIRED -> "The certificate has expired."
                            SslError.SSL_IDMISMATCH -> "The certificate Hostname mismatch."
                            SslError.SSL_NOTYETVALID -> "The certificate is not yet valid."
                            SslError.SSL_DATE_INVALID -> "The date of the certificate is invalid"
                            else -> "SSL Certificate error."
                        }
                        setMessage("$errorMsg Do you want to continue anyway?")
                        setPositiveButton("continue") { _, _ ->
                            handler?.proceed()
                        }
                        setNegativeButton("cancel") { _, _ ->
                            handler?.cancel()
                        }
                    }.create().show()
                }
```

# 违反Google Play隐私政策

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/google_play_provicy_error.jpg)

因为使用的个推SDK有读取安装应用的权限，wtf？，一个推送SDK访问安装应用干啥？，后来看文档发现个推有专门针对Google Play的SDK，区别对待吗？

**问题**：个推SDK使用了读取已安装应用权限，但是没有作出披露，就是没有说明为何要收集，属于违法收集[用户信息](https://support.google.com/googleplay/android-developer/answer/9888076/)。

**分析**：遇到这种情况，首先更换个推的Google Play版推送SDK，然后将涉及到收集用户信息的情况需要作出披露。
附上完整的[Google Play开发政策条款](https://play.google.com/about/developer-content-policy/)。

**正确处理方式**：

* 更换个推SDK为Google Play版。
* 对应用可能涉及到收集用户信息的地方做出披露。

在这次拒绝之后，我把应用闪屏页的用户隐私政策弹窗做了修改，效果如下：

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/google_play_provicy.gif)

修改之后成功通过审核。
