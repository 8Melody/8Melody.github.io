---
layout: post
title: "Android指纹识别简记"
subtitle: "Android指纹识别简记。"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Android
  - 指纹识别
---

# 前言

Google在`Android6.0`开始已经有官方API支持指纹识别，每次讨论这个话题总绕不开国产五彩缤纷的ROM，国产部分手机在`Android6.0`之前有已经实现了自己的指纹识别功能，具体的适配需要去查阅厂商开发文档，本文不深入讨论。

想要实现指纹识别在`AndroidM（6.0）～ Android P（9.0）`有 [FingerprintManager](https://developer.android.com/reference/android/hardware/fingerprint/FingerprintManager)，而在`Android P（9.0）之后`，Google推荐使用
[BiometricPrompt](https://developer.android.com/reference/android/hardware/biometrics/BiometricPrompt?hl=en)来实现生物识别认证，当然这个生物识别不止是指纹识别，可能会有虹膜、人脸识别等，本文记录`Android6.0～Android9.0`指纹识别以及`Android9.0`以上的指纹识别处理。

# Android6.0。

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/fingerprint6.0.gif)

---

Android6.0实现指纹识别需要用到的核心类为[FingerprintManager](https://developer.android.google.cn/reference/kotlin/android/hardware/fingerprint/FingerprintManager?hl=en)，为了更好的兼容及完善API，Google推荐使用的是[FingerprintManagerCompat](https://developer.android.google.cn/reference/kotlin/androidx/core/hardware/fingerprint/FingerprintManagerCompat?hl=en)。

需要的权限：

`<uses-permission android:name="android.permission.USE_FINGERPRINT" />`

```java
 private val fingerprint by lazy {
        FingerprintManagerCompat.from(context)
    }
```

在开始指纹识别之前，需要判断硬件是否支持，Google提供的检测方法为：

```java
context.packageManager.hasSystemFeature(PackageManager#FEATURE_FINGERPRINT)
```
也可以使用

```java
FingerprintManagerCompat.from(context).isHardwareDetected
```

确定了设备支持指纹识别之后，方可调用认证方法

```java
FingerprintManagerCompat#authenticate(crypto: FingerprintManager.CryptoObject?, cancel: CancellationSignal?, flags: Int, callback: FingerprintManager.AuthenticationCallback, handler: Handler?)
```

**调用该方法之后设备的指纹识别模块会进入监听状态，在认证失败、成功、取消或者超时之后才会停止监听，在这期间无法再次调用指纹识别。**

参数解释：

|参数|解释|
|:-:|:-|
|[crypto:CryptoObject](https://developer.android.google.cn/reference/kotlin/android/hardware/fingerprint/FingerprintManager.CryptoObject.html?hl=en)|[FingerprintManager](https://developer.android.google.cn/reference/kotlin/android/hardware/fingerprint/FingerprintManager?hl=en)提供的数据加密包装类，支持[Signature](https://developer.android.google.cn/reference/kotlin/java/security/Signature.html?hl=en)、[Cipher](https://developer.android.google.cn/reference/kotlin/javax/crypto/Cipher.html?hl=en)。|
|[cancel:CancellationSignal](https://developer.android.google.cn/reference/kotlin/androidx/core/os/CancellationSignal?hl=en)|用于监听指纹识别的取消操作|
|flags|填0即可|
|[callback:AuthenticationCallback](https://developer.android.google.cn/reference/kotlin/android/hardware/fingerprint/FingerprintManager.AuthenticationCallback?hl=en)|指纹识别回调|
|handler:Handler|用于接收回调事件的Handler，可以为null。|

需要重点了解一下[CryptoObject](https://developer.android.google.cn/reference/kotlin/android/hardware/fingerprint/FingerprintManager.CryptoObject.html?hl=en)，这个提供了指纹数据的加密及解密。

该包装类目前支持的加密对象分别为
* [Signature](https://developer.android.google.cn/reference/kotlin/java/security/Signature.html?hl=en)一般用于提供数字签名，保证数据的完整性。
* [Cipher](https://developer.android.google.cn/reference/kotlin/javax/crypto/Cipher.html?hl=en)主要用于加密，如进行AES/DES/RSA等。


指纹信息保存需要用到AES加密，创建一个[Cipher](https://developer.android.google.cn/reference/kotlin/javax/crypto/Cipher.html?hl=en)，系统提供的方法为`Cipher.getInstance(transformation)`。
`transformation`的格式为：
* "algorithm/mode/padding"
* "algorithm"

如：
```java
val mCipher = Cipher.getInstance("<i>DES/CBC/PKCS5Padding</i>")
```
具体的算法规则参考：[Cipher#Algorithm](https://developer.android.google.cn/reference/kotlin/javax/crypto/Cipher.html?hl=en)。
所以创建一个[Cipher](https://developer.android.google.cn/reference/kotlin/javax/crypto/Cipher.html?hl=en)可以这么写：

```java
class CipherCreator {
    companion object {
        const val KEY_ALGORITHM = KeyProperties.KEY_ALGORITHM_AES
        const val BLOCK_MODE = KeyProperties.BLOCK_MODE_CBC
        const val ENCRYPTION_PADDING = KeyProperties.ENCRYPTION_PADDING_PKCS7
        const val TRANSFORMATION = "$KEY_ALGORITHM/$BLOCK_MODE/$ENCRYPTION_PADDING"
    }


    private fun createCipher(): Cipher {
        return try {
            Cipher.getInstance(TRANSFORMATION)
        } catch (e: NoSuchAlgorithmException) {
            throw Exception("Could not create the cipher for fingerprint authentication.", e)
        }
    }
}
```
创建好了[Cipher](https://developer.android.google.cn/reference/kotlin/javax/crypto/Cipher.html?hl=en)需要进行初始化，调用的方法为`Cipher#init(opmode: Int, key: Key!)`，api解释为。

```java
fun init(opmode: Int, key: Key!): Unit
```
Initializes this cipher with a key.

The cipher is initialized for one of the following four operations: encryption, decryption, key wrapping or key unwrapping, depending on the value of opmode.

大意为根据`opmode`对[Cipher](https://developer.android.google.cn/reference/kotlin/javax/crypto/Cipher.html?hl=en)进行初始化，一般`opmode`对应的操作为：
* 加密
* 解密
* 包装key
* 解包key

另一个参数[Key](https://developer.android.google.cn/reference/java/security/Key?hl=en)的获取方式为：

```java
class CipherCreator {

    companion object {
        const val KEY_NAME = "com.example.github.CipherCreator"
        const val KEYSTORE_NAME = "AndroidKeyStore"
        const val KEY_ALGORITHM = KeyProperties.KEY_ALGORITHM_AES
        const val BLOCK_MODE = KeyProperties.BLOCK_MODE_CBC
        const val ENCRYPTION_PADDING = KeyProperties.ENCRYPTION_PADDING_PKCS7
        const val TRANSFORMATION = "$KEY_ALGORITHM/$BLOCK_MODE/$ENCRYPTION_PADDING"
    }

    private val keystore by lazy {
        KeyStore.getInstance(KEYSTORE_NAME).apply {
            load(null)
        }
    }


    private fun getKey(): Key {
        if (!keystore.isKeyEntry(KEY_NAME)) {
            createKey()
        }
        return keystore.getKey(KEY_NAME, null)
    }

    private fun createKey() {
        val keyGen = KeyGenerator.getInstance(KEY_ALGORITHM, KEYSTORE_NAME)
        val keyGenSpec = KeyGenParameterSpec.Builder(
            KEY_NAME,
            KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
        )
            .setBlockModes(BLOCK_MODE)
            .setEncryptionPaddings(ENCRYPTION_PADDING)
            .setUserAuthenticationRequired(true)
            .build()
        keyGen.init(keyGenSpec)
        keyGen.generateKey()
    }
}
```
所以获取一个[Cipher](https://developer.android.google.cn/reference/kotlin/javax/crypto/Cipher.html?hl=en)的完整代码如下：

```java
class CipherCreator {

    companion object {
        const val KEY_NAME = "com.example.github.CipherCreator"
        const val KEYSTORE_NAME = "AndroidKeyStore"
        const val KEY_ALGORITHM = KeyProperties.KEY_ALGORITHM_AES
        const val BLOCK_MODE = KeyProperties.BLOCK_MODE_CBC
        const val ENCRYPTION_PADDING = KeyProperties.ENCRYPTION_PADDING_PKCS7
        const val TRANSFORMATION = "$KEY_ALGORITHM/$BLOCK_MODE/$ENCRYPTION_PADDING"
    }

    private val keystore by lazy {
        KeyStore.getInstance(KEYSTORE_NAME).apply {
            load(null)
        }
    }

    fun createCipher(): Cipher {
        val key = getKey()
        return try {
            Cipher.getInstance(TRANSFORMATION).apply {
                init(Cipher.ENCRYPT_MODE or Cipher.DECRYPT_MODE, key)
            }
        } catch (e: Exception) {
            throw Exception("Could not create the cipher for fingerprint authentication.", e)
        }
    }

    private fun getKey(): Key {
        if (!keystore.isKeyEntry(KEY_NAME)) {
            createKey()
        }
        return keystore.getKey(KEY_NAME, null)
    }

    private fun createKey() {
        val keyGen = KeyGenerator.getInstance(KEY_ALGORITHM, KEYSTORE_NAME)
        val keyGenSpec = KeyGenParameterSpec.Builder(
            KEY_NAME,
            KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
        )
            .setBlockModes(BLOCK_MODE)
            .setEncryptionPaddings(ENCRYPTION_PADDING)
            .setUserAuthenticationRequired(true)
            .build()
        keyGen.init(keyGenSpec)
        keyGen.generateKey()
    }
}
```

指纹验证的所需参数获取完毕，调用指纹验证。

```java
  /**
     * core
     */
    private val fingerprint by lazy {
        FingerprintManagerCompat.from(holder)
    }
    /**
     * crypto
     */
    private val cryptoObj by lazy {
        FingerprintManagerCompat.CryptoObject(CipherCreator().createCipher())
    }
    /**
     * cancelSignal
     */
    private val cancellationSignal by lazy {
        CancellationSignal()
    }
    
    fun authenticate() {
        fingerprint.authenticate(cryptoObj, 0, cancellationSignal, object :
            FingerprintManagerCompat.AuthenticationCallback() {

            override fun onAuthenticationError(errMsgId: Int, errString: CharSequence) {

            }

            override fun onAuthenticationFailed() {

            }

            override fun onAuthenticationHelp(helpMsgId: Int, helpString: CharSequence) {

            }

            override fun onAuthenticationSucceeded(result: FingerprintManagerCompat.AuthenticationResult?) {

            }
        }, null)
    }
```


# Android 9.0

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/fingerprint9.0.gif)

---

`Android9.0`实现指纹识别的核心类为[BiometricPrompt](https://developer.android.google.cn/reference/androidx/biometric/BiometricPrompt?hl=en)，在`Android6.0`中指纹识别的对话框需要自己实现，而[BiometricPrompt](https://developer.android.google.cn/reference/androidx/biometric/BiometricPrompt?hl=en)提供了系统的对话框来进行验证。
[BiometricPrompt](https://developer.android.google.cn/reference/androidx/biometric/BiometricPrompt?hl=en)指纹识别的内部实现还是使用了[FingerprintManager](https://developer.android.com/reference/android/hardware/fingerprint/FingerprintManager)，不过在其之上做了微小的调整，比如提供系统的Dialog，更加人性化的回调，核心实现还是[FingerprintManager](https://developer.android.com/reference/android/hardware/fingerprint/FingerprintManager)。

*在`Android9.0`判断设备是否支持指纹识别，可以调用`BiometricManager#canAuthenticate`该方法回返回一个int。*
* BIOMETRIC_SUCCESS 
* BIOMETRIC_ERROR_NONE_ENROLLED 没有录入指纹
* BIOMETRIC_ERROR_NO_HARDWARE 硬件不支持

需要的权限为：

`<uses-permission android:name="android.permission.USE_BIOMETRIC" />`


使用[BiometricPrompt](https://developer.android.google.cn/reference/androidx/biometric/BiometricPrompt?hl=en)来实指纹识别的步骤为：

## 创建BiometricPrompt

```java
  val mBiometricPrompt = BiometricPrompt.Builder(context)
                .setTitle("指纹验证")
                .setNegativeButton("取消", context.mainExecutor,
                    DialogInterface.OnClickListener { _, _ ->

                    }).build()
```

## 调用authenticate

```java
mBiometricPrompt.authenticate(
                cryptObject,
                cancellationSignal,
                context.mainExecutor,
                object : BiometricPrompt.AuthenticationCallback() {
                    override fun onAuthenticationError(errMsgId: Int, errString: CharSequence) {
                        
                    }

                    override fun onAuthenticationFailed() {
                        
                    }

                    override fun onAuthenticationHelp(helpMsgId: Int, helpString: CharSequence) {
                        
                    }

                    override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult?) {
                        
                    }
                })
```

调用认证所需参数跟`Android6.0`参数生成一致。

[本文Demo地址。](https://gitee.com/xuyonghong/FingerPrint)