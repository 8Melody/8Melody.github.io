---
layout: post
title: "Android10 行为变更"
subtitle: "Android 行为变更。"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Android行为变更
  - Android10
---

## 限制非SDK接口

为了帮助确保应用的稳定性和兼容性，Android 平台开始限制应用在 Android 9（API 级别 28）中使用[非 SDK 接口](https://developer.android.com/distribute/best-practices/develop/restrictions-non-sdk-interfaces)。Android 10 包含更新后的受限制非 SDK 接口列表（基于与 Android 开发者之间的协作以及最新的内部测试）。我们的目标是在限制使用非 SDK 接口之前确保有可用的公开替代方案。

如果您不打算以 Android 10（API 级别 29）为目标平台，那么其中一些变更可能不会立即对您产生影响。虽然您目前仍然可以使用灰名单中的一些非 SDK 接口（取决于您的应用的[目标 API 级别](https://developer.android.com/distribute/best-practices/develop/target-sdk)），但如果您使用任何非 SDK 方法或字段，则应用无法运行的风险终归较高。

如果您不确定自己的应用是否使用了非 SDK 接口，则可以[测试该应用](https://developer.android.com/distribute/best-practices/develop/restrictions-non-sdk-interfaces#test-for-non-sdk)，进行确认。如果您的应用依赖于非 SDK 接口，则应该开始计划迁移到 SDK 替代方案。不过，我们知道某些应用具有使用非 SDK 接口的有效用例。如果您无法为应用中的某项功能找到使用非 SDK 接口的替代方案，则应该[请求新的公共 API](https://developer.android.com/distribute/best-practices/develop/restrictions-non-sdk-interfaces#feature-request)。

要了解详情，请参阅 [Android 10 中有关限制非 SDK 接口的更新](https://developer.android.com/about/versions/10/non-sdk-q)以及[针对非 SDK 接口的限制。](https://developer.android.com/distribute/best-practices/develop/restrictions-non-sdk-interfaces)

## 手势导航

从 Android 10 开始，用户可以在设备中启用手势导航。用户启用后，手势导航会影响设备上的所有应用，无论应用是否以 API 级别 29 为目标平台。例如，如果用户从屏幕边缘向内滑动，系统会将该手势解读为“返回”导航，除非应用针对屏幕的相应部分[明确替换该手势](https://developer.android.com/guide/navigation/gesturenav#conflicting-gestures)。

为了确保您的应用与手势导航兼容，您需要将应用内容扩展到屏幕边缘，并适当地处理存在冲突的手势。有关信息，请参阅[手势导航文档](https://developer.android.com/guide/navigation/gesturenav)。

## NDK

Android 10 包含 NDK 方面的以下变更。

### 共享对象不得包含文本重定位

Android 6.0（API 级别 23）[已禁止在共享对象中使用文本重定位](https://android.googlesource.com/platform/bionic/+/master/android-changes-for-ndk-developers.md#text-relocations-enforced-for-api-level-23)。代码必须按原样加载，且不得修改。此变更可以缩短应用的加载时间并提高安全性。

SELinux 针对以 Android 10 或更高版本为目标平台的应用强制执行此限制。如果这些应用继续使用包含文本重定位的共享对象，则它们出现中断的风险较高。

### Bionic 库和动态链接器路径变更

从 Android 10 开始，多个路径不再采用常规文件形式，而是采用符号链接形式。如果应用一直以来依赖的都是采用常规文件形式的路径，则可能会出现中断：

* `/system/lib/libc.so` -> `/apex/com.android.runtime/lib/bionic/libc.so`
* `/system/lib/libm.so` -> `/apex/com.android.runtime/lib/bionic/libm.so`
* `/system/lib/libdl.so` -> `/apex/com.android.runtime/lib/bionic/libdl.so`
* `/system/bin/linker` -> `/apex/com.android.runtime/bin/linker`

这些变更也会影响文件的 64 位版本，对于这些版本，系统会将 `lib/` 替换为 `lib64/`。

为了确保兼容性，符号链接会基于旧路径提供。例如，`/system/lib/libc.so` 是指向 `/apex/com.android.runtime/lib/bionic/libc.so` 的符号链接。因此，`dlopen(“/system/lib/libc.so”)` 会继续工作，但当应用尝试通过读取 `/proc/self/maps` 或类似项来检测已加载的库时，将会发现不同之处。这并不常见，但我们发现一些应用会将这种做法作为对抗黑客攻击的一项举措。如果是这样，则应该将 `/apex/…` 路径添加为 Bionic 文件的有效路径。

### 系统二进制文件/库会映射到只执行内存

从 Android 10 开始，系统二进制文件和库的可执行部分会映射到只执行（不可读取）内存，作为防范代码重用攻击的一种安全强化技术。如果您的应用针对已标记为只执行的内存段执行读取操作（无论此读取操作是来自错误、漏洞还是有意的内存检查），系统都会向您的应用发送 `SIGSEGV`信号。

您可以通过检查 `/data/tombstones/` 中的相关 tombstone 文件来确定此行为是否会导致崩溃。与只执行相关的崩溃包含以下中止消息：

>  Cause: execute-only (no-read) memory access error; likely due to data in .text.

要解决此问题以执行内存检查等操作，可以通过调用 `mprotect()` 将只执行内存段标记为“读取+执行”。不过，我们强烈建议您事后将其重新设为只执行，因为此访问权限设置可以更好地保护您的应用和用户。

**注意：此行为不会对 ptrace 的调用产生影响，可允许您调试 ptrace。**

## 安全

Android 10 包含安全方面的以下变更。

### TLS 1.3 默认处于启用状态

在 Android 10 及更高版本中，系统默认会为所有 TLS 连接启用 TLS 1.3。以下是有关 TLS 1.3 实现的一些重要的详细信息：

* TLS 1.3 加密套件不可自定义。在启用 TLS 1.3 后，受支持的 TLS 1.3 加密套件会始终保持启用状态。任何尝试通过调用 [setEnabledCipherSuites()](https://developer.android.com/reference/javax/net/ssl/SSLSocket.html#setEnabledCipherSuites(java.lang.String%5B%5D)) 停用该加密套件的操作均会被忽略。
* 在协商 TLS 1.3 时，系统会在将会话添加到会话缓存之前调用 [HandshakeCompletedListener](https://developer.android.com/reference/javax/net/ssl/HandshakeCompletedListener) 对象。（在 TLS 1.2 和之前的其他版本中，系统会在将会话添加到会话缓存之后调用这些对象。）
* 在某些情况下，SSLEngine 实例会在之前的 Android 版本中抛出 [SSLHandshakeException](https://developer.android.com/reference/javax/net/ssl/SSLHandshakeException)，而这些实例在 Android 10 及更高版本中会改为抛出 [SSLProtocolException](https://developer.android.com/reference/javax/net/ssl/SSLProtocolException)。
* 不支持 0-RTT 模式

如有需要，可以通过调用 [SSLContext.getInstance("TLSv1.2")](https://developer.android.com/reference/javax/net/ssl/SSLContext.html#getInstance(java.lang.String)) 来获取已停用 TLS 1.3 的 SSLContext。您还可以对相关对象调用 [setEnabledProtocols()](https://developer.android.com/reference/javax/net/ssl/SSLSocket.html#setEnabledProtocols(java.lang.String%5B%5D))，从而为每个连接启用或停用协议版本。

### TLS 不信任使用 SHA-1 签名的证书

在 Android 10 中，使用 SHA-1 哈希算法的证书在 TLS 连接中不受信任。自 2016 年以来，根 CA 未再颁发过此类证书，因为它们不再受 Chrome 或其他主流浏览器的信任。

如果某网站使用的是 SHA-1 证书，则任何尝试连接该网站的操作都将失败。

### KeyChain 行为变更和改进

当 TLS 服务器在 TLS 握手中发送证书请求消息时，某些浏览器（如 Google Chrome）允许用户选择证书。从 Android 10 开始，[KeyChain](https://developer.android.com/reference/android/security/KeyChain) 对象会在调用 `KeyChain.choosePrivateKeyAlias()` 时信任颁发机构和密钥规范参数，以向用户显示证书选择提示。需要注意的是，此提示不包含不符合服务器规范的选项。

如果没有可用的用户可选证书（当没有与服务器规范匹配的证书或设备没有安装任何证书时，便会出现这种情况），则完全不会出现证书选择提示。

此外，在 Android 10 或更高版本上，无需具备设备屏幕锁定功能，就能将密钥或 CA 证书导入 `KeyChain` 对象中。

### 其他 TLS 和加密更改

Android 10 中引入的 TLS 和加密库方面的一些细小变更包括：

* AES/GCM/NoPadding 和 ChaCha20/Poly1305/NoPadding 加密会从 `getOutputSize()` 中返回更准确的缓冲区大小。
* 使用 TLS 1.2 或更高版本的最高协议在尝试连接时会忽略 `TLS_FALLBACK_SCSV` 加密套件。由于 TLS 服务器实现方面的改进，我们不建议尝试 TLS 外部回退。不过，我们建议依赖于 TLS 版本协商。
* ChaCha20-Poly1305 是 ChaCha20/Poly1305/NoPadding 的别名。
* 带有尾随点的主机名不属于有效的 SNI 主机名。
* 为证书响应选择签名密钥时，将遵循 `CertificateRequest` 中的 supported_signature_algorithms 扩展。
* 不透明的签名密钥（如 Android 密钥库中的密钥）可在 TLS 中与 RSA-PSS 签名一起使用。

## WLAN 直连广播

在 Android 10 中，以下与[ WLAN 直连](https://developer.android.com/training/connect-devices-wirelessly/wifi-direct)相关的广播不具有粘性：

* [WIFI_P2P_CONNECTION_CHANGED_ACTION](https://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager#WIFI_P2P_CONNECTION_CHANGED_ACTION)
* [WIFI_P2P_THIS_DEVICE_CHANGED_ACTION](https://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager#WIFI_P2P_THIS_DEVICE_CHANGED_ACTION)

如果您的应用依赖于在注册时接收这些广播（因为其之前一直具有粘性），请在初始化时使用适当的 `get()` 方法获取信息。


## WLAN 感知功能

Android 10 扩大了支持范围，现在可以使用 WLAN 感知数据路径轻松创建 TCP/UDP 套接字。要创建连接到 `ServerSocket` 的 TCP/UDP 套接字，客户端设备需要知道服务器的 IPv6 地址和端口。这在之前需要通过频外方式进行通信（例如使用 BT 或 WLAN 感知第 2 层消息传递），或者使用其他协议（例如 mDNS）通过频内方式发现。而借助 Android 10，可以将此类消息作为网络设置的一部分进行传递。

服务器可以执行以下任一操作：

* 初始化 `ServerSocket` 并设置或获取要使用的端口。
* 将端口信息指定为 WLAN 感知网络请求的一部分。

以下代码示例显示了如何将端口信息指定为网络请求的一部分：

```Java
    val ss = ServerSocket()
    val ns = WifiAwareNetworkSpecifier.Builder(discoverySession, peerHandle)
      .setPskPassphrase("some-password")
      .setPort(ss.localPort)
      .build()

    val myNetworkRequest = NetworkRequest.Builder()
      .addTransportType(NetworkCapabilities.TRANSPORT_WIFI_AWARE)
      .setNetworkSpecifier(ns)
      .build()
    
```
然后，客户端会执行 WLAN 感知网络请求来获取服务器提供的 IPv6 和端口：

```Java
val callback = object : ConnectivityManager.NetworkCallback() {
      override fun onAvailable(network: Network) {
        ...
      }

      override fun onLinkPropertiesChanged(network: Network,
          linkProperties: LinkProperties) {
        ...
      }

      override fun onCapabilitiesChanged(network: Network,
          networkCapabilities: NetworkCapabilities) {
        ...
        val ti = networkCapabilities.transportInfo
        if (ti is WifiAwareNetworkInfo) {
           val peerAddress = ti.peerIpv6Addr
           val peerPort = ti.port
        }
      }
      override fun onLost(network: Network) {
        ...
      }
    };

    connMgr.requestNetwork(networkRequest, callback)
```

## Go 设备上的 SYSTEM_ALERT_WINDOW

在 Android 10（Go 版本）设备上运行的应用无法获得 [SYSTEM_ALERT_WINDOW](https://developer.android.com/reference/android/Manifest.permission#SYSTEM_ALERT_WINDOW) 权限。这是因为绘制叠加层窗口会使用过多的内存，这对低内存 Android 设备的性能十分有害。

如果在搭载 Android 9 或更低版本的 Go 版设备上运行的应用获得了 `SYSTEM_ALERT_WINDOW` 权限，则即使设备升级到 Android 10，也会保留此权限。不过，尚不具有此权限的应用在设备升级后便无法获得此权限了。

如果 Go 设备上的应用发送具有 [ACTION_MANAGE_OVERLAY_PERMISSION](https://developer.android.com/reference/android/provider/Settings#ACTION_MANAGE_OVERLAY_PERMISSION) 操作的 intent，则系统会自动拒绝此请求，并将用户转到设置屏幕，上面会显示不允许授予此权限，原因是它会减慢设备的运行速度。如果 Go 设备上的应用调用 Settings[.canDrawOverlays()](https://developer.android.com/reference/android/provider/Settings#canDrawOverlays(android.content.Context))，则此方法始终返回 false。同样，这些限制不适用于在设备升级到 Android 10 之前便已收到 `SYSTEM_ALERT_WINDOW` 权限的应用。

## 关于以旧版 Android 系统为目标平台的应用的警告

在搭载 Android 10 或更高版本的设备上，如果用户首次运行以 Android 5.1（API 级别 22）或更低版本为目标平台的应用，则会看到警告。如果此应用要求用户授予权限，则系统会先向用户提供调整应用权限的机会，然后才会允许此应用首次运行。

由于 Google Play 的[目标 API 方面的要求](https://support.google.com/googleplay/android-developer/answer/113469#targetsdk)，用户只有在运行最近未更新的应用时才会看到这些警告。对于通过其他商店分发的应用，我们也将于 2019 年引入类似的目标 API 方面的要求。如需详细了解这些要求，请参阅在[ 2019 年扩展目标 API 级别方面的要求。](https://android-developers.googleblog.com/2019/02/expanding-target-api-level-requirements.html)


## 移除了 SHA-2 CBC 加密套件

以下 SHA-2 CBC 加密套件已从平台中移除：

* `TLS_RSA_WITH_AES_128_CBC_SHA256`
* `TLS_RSA_WITH_AES_256_CBC_SHA256`
* `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256`
* `TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384`
* `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256`
* `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384`

这些加密套件不如使用 GCM 的类似加密套件安全，并且大多数服务器要么同时支持这些加密套件的 GCM 变体和 CBC 变体，要么二者均不支持。

**注意：应用和库应该让其所需的加密套件集与 [getSupportedCipherSuites()](https://developer.android.com/reference/javax/net/ssl/SSLSocket.html#getSupportedCipherSuites()) 中返回的值相交，以便提前防范加密套件日后遭到移除。**


## 应用使用情况

Android 10 引入了与应用使用情况相关的以下行为变更：

* **UsageStats 应用使用情况方面的改进** - 当在分屏或画中画模式下使用应用时，Android 10 现在能够使用 UsageStats 准确地跟踪应用使用情况。此外，Android 10 可以正确地跟踪免安装应用的使用情况。

* **按应用开启灰度模式** - Android 10 可针对各个应用设置灰度显示模式。

* **按应用开启干扰模式** - Android 10 可以选择性地将应用设置为“干扰模式”，此时系统会禁止显示其通知，并且不会将其显示为推荐的应用。

* **暂停和播放** - 在 Android 10 中，暂停的应用无法播放音频。


## HTTPS 连接变更

如果在 Android 10 上运行的应用将 `null` 传递给 [setSSLSocketFactory()](https://developer.android.com/reference/javax/net/ssl/HttpsURLConnection.html#setSSLSocketFactory(javax.net.ssl.SSLSocketFactory))，则会出现 [IllegalArgumentException](https://developer.android.com/reference/java/lang/IllegalArgumentException)。在以前的版本中，将 `null` 传递给 `setSSLSocketFactory()` 与传入当前的默认 SSL 套接字工厂效果相同。

## android.preference 库已弃用

从 Android 10 开始，将弃用 `android.preference` 库。开发者应该改为使用 AndroidX preference 库，这是 [Android Jetpack](https://developer.android.com/jetpack) 的一部分。如需获取其他有助于迁移和开发的资源，请查看经过更新的[设置指南](https://developer.android.com/guide/topics/ui/settings)以及我们的[公开示例应用](https://github.com/android/user-interface/tree/master/PreferencesKotlin)和[参考文档。](https://developer.android.com/reference/androidx/preference/package-summary)


## ZIP 文件实用程序库变更

Android 10 对 `java.util.zip` 软件包（用于处理 ZIP 文件）中的类进行了以下变更。这些变更会让库的行为在 Android 和使用 `java.util.zip` 的其他平台之间更加一致。

### Inflater

在以前的版本中，如果在调用 [end()](https://developer.android.com/reference/java/util/zip/Inflater#end()) 之后调用 Inflater 类中的某些方法，这些方法会抛出 [IllegalStateException](https://developer.android.com/reference/java/lang/IllegalStateException)。在 Android 10 中，这些方法会改为抛出 [NullPointerException。](https://developer.android.com/reference/java/lang/NullPointerException)

### ZipFile

在 Android 10 及更高版本中，如果所提供的 ZIP 文件不包含任何文件，则 [ZipFile 的构造函数](https://developer.android.com/reference/java/util/zip/ZipFile.html#ZipFile(java.io.File,%20int,%20java.nio.charset.Charset))（采用的参数类型为 `File`、`int` 和 `Charset`）不会抛出 [ZipException](https://developer.android.com/reference/java/util/zip/ZipException)。

### ZipOutputStream

在 Android 10 及更高版本中，如果 `ZipOutputStream` 中的 [finish()](https://developer.android.com/reference/java/util/zip/ZipOutputStream#finish()) 方法尝试为不包含任何文件的 ZIP 文件写入输出流，则此方法不会抛出 `ZipException。`

## 摄像头变更

很多使用摄像头的应用都会假定如果设备采用纵向配置，则物理设备也会处于纵向，正如[摄像头方向](https://source.android.com/compatibility/9/android-9-cdd.html#7_5_5_camera_orientation)中所述。在过去可以做出这样的假定，但随着可用的设备类型（例如可折叠设备）的扩展，这一情况发生了变化。针对这些设备做出这样的假定可能导致相机取景器的显示产生错误的旋转和/或缩放。

以 API 级别 24 或更高级别为目标平台的应用应该明确设置 `android:resizeableActivity`，并提供必要的功能来处理多窗口操作。

## 电池用量跟踪

从 Android 10 开始，只要在发生重大充电事件之后拔下设备电源插头，[SystemHealthManager](https://developer.android.com/reference/android/os/health/SystemHealthManager) 就会重置其电池用量统计信息。一般来说，重大充电事件指的是设备电池已充满，或者设备电量从几乎耗尽变为即将充满。

在 Android 10 之前，无论何时拔下设备电源插头，无论电池电量有多微小的变化，电池用量统计信息都会重置。

## Android Beam 已弃用

在 Android 10 中，正式弃用了 Android Beam，这是一项旧版功能，可通过近距离无线通信 (NFC) 在多个设备之间启动数据共享。我们还弃用了一些相关的 NFC API。Android Beam 仍可供需要的设备制造商合作伙伴使用，但它已不再处于积极的开发阶段。不过，Android 仍将继续支持其他的 NFC 功能和 API，并且从标签和付款中读取数据等使用场景仍将继续按预期执行。