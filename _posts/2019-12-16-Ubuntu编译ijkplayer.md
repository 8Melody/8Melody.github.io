---
layout: post
title: "编译ijkplayer"
subtitle: "Ubuntu编译ijkplayer"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Android
---

# 编译ijkplayer

鉴于公司即将开始的直播APP开发，对市面上主流播放器进行了考究之后决定采用Bilibili开源的[ijkplayer](https://github.com/bilibili/ijkplayer)作为播放器，本文记录如何在Ubuntu下编译ijkplayer，Windows安装虚拟机的教程可以参考[这里](https://zhuanlan.zhihu.com/p/38797088)。

# 环境准备
本次编译需要的环境如下：
* JDK 
* SDK
* NDK
* Git
* Make
* vim
* yasm

## JDK
在Ubuntu下访问[orace.com](https://www.oracle.com/technetwork/java/javase/downloads/index.html)

## SDK
在Terminal下输入：

`wget https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip`

解压下载好的文件：

`unzip sdk-tools-linux-3859397.zip`

检查当前sdk安装信息：

`sdkmanager –list`

安装对应的package：

`sdkmanager “build-tools;28.0.1”`

## NDK
ijkplayer编译的版本为[NDK r10e](http://developer.android.com/tools/sdk/ndk/index.html)
所以本文采用的环境也相同。

Terminal：

`wget http://dl.google.com/android/ndk/android-ndk-r10e-linux-x86_64.bin`

依次执行：
`chmod a+x android-ndk-r10e-linux-x86_64.bin  `

`./android-ndk-r10e-linux-x86_64.bin`

等截面提示（Everything is ok）。

## Git

`sudo apt-get install git`

## yasm

`sudo apt-get install yasm`

## vim

`sudo apt install vim`

## Make

`sudo apt-get install ubuntu-make`

等所有的需要的环境准备好之后，下一步是配置环境变量。

使用`vim`打开`/etc/profile`，在文件末尾追加以下环境变量，本文以我的虚拟机目录为例。

```json
export JDK_HOME=/home/jdk/jdk1.8.0_231
export PATH=${JDK_HOME}/bin:$PATH
export ANDROID_SDK=/home/sdk/tools
export PATH=${ANDROID_SDK}/bin:${PATH}
export ANDROID_NDK=/home/ndk/android-ndk-r10e
export PATH=$PATH:$ANDROID_NDK
```

至此环境准备完毕，接下来是编译步骤。

# 编译ijkplayer

编译流程如下：
* 获取ijkplayer的代码
* 切换到最新的分支
* 初始化android
* 编译脚本配置
* 获取openssl获取https支持
* 编译openssl
* 编译ffmpeg
* 编译ijkplayer

## 下载ijkplayer-android的代码

```java
git clone https://github.com/Bilibili/ijkplayer ijkplayer-android
```

切换到最新的分支

```java
git checkout -B latest k0.8.8
```

## 初始化Android
```java
./init-android.sh
```

## 编译脚本配置

ijkplayer默认编译配置在`config/`目录下，默认使用的是`module-default.sh`，可以使用`vim`命令打开里面内容查看具体支持的播放音视频类型等。

官方提供的四个编译脚本分别为：

* module-default.sh：最多的编码解码支持。
* module-lite-hevc.sh：主流的编码解码支持（支持hevc），更小的体积。
* module-lite.sh：主流的编码解码支持，更小的体积。（默认编译使用的是这个脚本）

本文以编译最全的so为例。

需要执行的操作如下：

```java
cd config
rm module.sh
ln -s module-default.sh 
```

## 支持https

返回到根目录执行：
```java
./init-android-openssl.sh
```

## 编译openssl
```java
cd android/contrib
./compile-openssl.sh clean
./compile-openssl.sh all
```

## 编译ffmpeg

如果需要编译指定的so可以使用：
`./compile-ffmpeg.sh armv7a `
本文以全部为例：
```java
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```

## 编译ijkplayer

```java
./compile-ijk.sh all
```

编译完成
```java
ijkplayer-android/android/ijkplayer 
```
目录下各so目录下可以找到编译好的so文件。

[伸手党？](https://github.com/qfxl/ijkplayer-so)
