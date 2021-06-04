---
layout: post
title: "Android存储空间整理"
subtitle: "Scope Storage"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags: 
  - Storage
  - Scope Storage
  - Android
---

Android文件系统类似于其他平台上基于磁盘的文件系统。该系统提供了以下几种保存应用数据的选项：

* **应用专属存储空间**：存储仅供应用使用的文件，可以存储到内部存储卷中的专属目录或外部存储空间中的其他专属目录。使用内部存储空间中的目录保存其他应用不应访问的敏感信息。
* **共享存储**：该目录下的文件，包括媒体、文档和其他文件可以与其他应用共享。
* **偏好设置**：以键值对形式存储私有原始数据。
* **数据库**：持久化数据，比如[Room](https://developer.android.com/jetpack/androidx/releases/room?gclid=Cj0KCQjw78yFBhCZARIsAOxgSx3DOpSqDYJ2pBdH0ipGWDXDLHrOKzt2GXEz0tEu3Q_QxSha6pKI200aAhVJEALw_wcB&gclsrc=aw.ds)。

以下表格对比各个目录的差异：

|目录|目录说明|访问方法|所需权限|其他应用是否可以访问|是否跟随APP卸载自动删除|
|:-:|:-:|:-:|:-:|:-:|:-:|
|应用专属目录|仅供本应用使用的文件|从内部存储空间访问，可以使用 getFilesDir() 或 getCacheDir()方法，从外部存储空间访问，可以使用 getExternalFilesDir()或getExternalCacheDir() 方法|从内部存储空间访问不需要任何权限如果应用在搭载 Android 4.4（API 级别 19）或更高版本的设备上运行，从外部存储空间访问不需要任何权限|如果文件存储在内部存储空间中的目录内，则不能访问如果文件存储在外部存储空间中的目录内，则可以访问|是|
|媒体目录|可共享的媒体文件（图片、音频文件、视频）|MediaStore API|在 Android 10（API 级别 29）或更高版本中，访问其他应用的文件需要 READ_EXTERNAL_STORAGE或WRITE_EXTERNAL_STORAGE 权限在 **Android 9（API 级别 28）或更低版本中，访问所有文件均需要相关权限**|是，但其他应用需要READ_EXTERNAL_STORAGE 权限|否|
|文档和其他文件|其他类型的可共享内容，包括已下载的文件|存储访问框架|无|是，可以通过系统文件选择器访问|否|
|应用偏好设置|键值对|JetpackPreferences 库|无|否|是|
|数据库|结构化数据|Room 持久性库|无|否|是|

# `getFilesDir()`和`getExternalFilesDir()`区别

`getFilesDir()`对应的是内部存储，只有当前APP能访问，并且该目录无法导出，只有root之后才能查看。

`getExternalFilesDir()`对应的是外部存储，其他APP可以访问，并且可以导出，对应的目录是`data/data/应用包名`。

# getFilesDir()
//演示如何读写文件

```java
val file = File(filesDir, "test.txt")
FileOutputStream(file).use { fos ->
    fos.write("你好，这是一段来自外部存储的文本".toByteArray())
}
val savedResult = String(file.inputStream().readBytes())
```

# getExternalFilesDir()

```java
val file = File(getExternalFilesDir(null), "test.txt")
FileOutputStream(file).use { fos ->
    fos.write("你好，这是一段来自外部存储的文本".toByteArray())
}
val savedResult = String(file.inputStream().readBytes())
```
其中`getExternalFilesDir(type)`， `type`的值可能为：

* Environment.DIRECTORY_MUSIC
* Environment.DIRECTORY_PODCASTS
* Environment.DIRECTORY_RINGTONES
* Environment.DIRECTORY_ALARMS
* Environment.DIRECTORY_NOTIFICATIONS
* Environment.DIRECTORY_PICTURES
* Environment.DIRECTORY_MOVIES

**如果`type`为`null`则代表获取的是根目录**


# MediaStore

演示如何下载图片并存入相册，需要权限为`INTERNET`，在Android10（Q）以下需要`WRITE_EXTERNAL_STORAGE`。

```java
/**
  * 下载图片
  */
private fun downloadImage(url: String) {
    lifecycleScope.launch(Dispatchers.IO) {
        val connection =
            URL(url).openConnection() as? HttpURLConnection
            connection?.run {
            requestMethod = "GET"
            val bitmap = BitmapFactory.decodeStream(inputStream)
            //将下载的图片插入系统图库
            saveImageToGallery(bitmap)
        }
    }
}

 /**
    * 存储照片到相册
    * @param bitmap source
    */
private fun saveImageToGallery(bitmap: Bitmap) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
        //插入不需要权限
        val insertValues = ContentValues().apply {
            put(MediaStore.MediaColumns.DISPLAY_NAME, "测试图片")
            put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg")
            //API 29 
            put(MediaStore.MediaColumns.RELATIVE_PATH, Environment.DIRECTORY_DCIM)
            }
            var uri: Uri? = null
            runCatching {
                with(contentResolver) {
                    insert(
                        MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                        insertValues
                    )?.also { insertedUri ->
                        uri = insertedUri
                        openOutputStream(insertedUri)?.use { stream ->
                            if (!bitmap.compress(Bitmap.CompressFormat.JPEG, 95, stream)) {
                                throw IOException("存储图片失败")   
                            }
                                
                        } ?: throw IOException("获取图片流失败")

                    } ?: throw IOException("插入图片失败")
                }
            }.getOrElse {
                uri?.let { orphanUri ->
                    contentResolver.delete(orphanUri, null, null)
                }
                throw it
            }
        } else {
            //需要WRITE_EXTERNAL_STORAGE权限
            if (ContextCompat.checkSelfPermission(
                    this,
                    Manifest.permission.WRITE_EXTERNAL_STORAGE
                ) == PackageManager.PERMISSION_GRANTED
            ) {
                val imagesDir =
                    Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM)
                        .toString()
                val destFile = File(imagesDir, "测试图片.jpg")
                FileOutputStream(destFile).use { fos ->
                    bitmap.compress(Bitmap.CompressFormat.JPEG, 95, fos)
                }
            } else {
                ActivityCompat.requestPermissions(
                    this,
                    arrayOf(Manifest.permission.WRITE_EXTERNAL_STORAGE),
                    0x01
                )
            }
        }
    }
```

**需要注意的是：如果应用使用分区存储，则应仅针对搭载 Android 9（API 级别 28）或更低版本的设备请求存储相关权限。可以通过将 `android:maxSdkVersion` 属性添加到应用清单文件中的权限声明来应用此条件：**

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="28" />
```

下面演示如何查询时长>5s视频，**该操作需要`READ_EXTERNAL_STORAGE`权限**。

```java
private fun queryVideos() {
    val videoList = mutableListOf<Video>()
    lifecycleScope.launch(Dispatchers.IO) {
        val projection = arrayOf(
            MediaStore.Video.Media._ID,
            MediaStore.Video.Media.DISPLAY_NAME,
            MediaStore.Video.Media.DURATION,
            MediaStore.Video.Media.SIZE,
        )
        val selection = "${MediaStore.Video.Media.DURATION} >= ?"
        val selectionArgs =
            arrayOf(TimeUnit.MILLISECONDS.convert(5, TimeUnit.SECONDS).toString())
            val sortOrder = "${MediaStore.Video.Media.DISPLAY_NAME} ASC"
            val query = contentResolver.query(
                MediaStore.Video.Media.EXTERNAL_CONTENT_URI,
                projection,
                selection,
                selectionArgs,
                sortOrder
            )
        query?.use { cursor ->
            val idColumn = cursor.getColumnIndexOrThrow(MediaStore.Video.Media._ID)
            val nameColumn =
                cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DISPLAY_NAME)
            val durationColumn =
                cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DURATION)
                val sizeColumn = cursor.getColumnIndexOrThrow(MediaStore.Video.Media.SIZE)
            while (cursor.moveToNext()) {
                val id = cursor.getLong(idColumn)
                val name = cursor.getString(nameColumn)
                val duration = cursor.getInt(durationColumn)
                val size = cursor.getInt(sizeColumn)

                val contentUri =
                ContentUris.withAppendedId(MediaStore.Video.Media.EXTERNAL_CONTENT_URI,id)
                videoList += Video(contentUri, name, duration, size)
            }
        }
    }
}

data class Video(
    val uri: Uri,
    val name: String,
    val duration: Int,
    val size: Int
)
```

# 使用存储框架访问文件
`Android4.4（API 19）`引入了存储访问框架`(SAF)`，全称为`Storage Access Framework`借助 SAF，用户可轻松浏览和打开各种文档、图片及其他文件，而不用管这些文件来自其首选文档存储提供程序中的哪一个。用户可通过易用的标准界面，跨所有应用和提供程序以统一的方式浏览文件并访问最近用过的文件。
`SAF`包含以下元素：
* **文档提供程序：** 一种内容提供程序，可让存储服务（如 Google云端硬盘）提供其管理的文件。文档提供程序以[DocumentsProvider](https://developer.android.com/reference/android/provider/DocumentsProvider?hl=zh-cn)的子类形式实现。文档提供程序的架构基于传统的文件层次结构，但其实际的数据存储方式由您决定。Android 平台包含若干内置的文档提供程序，如 `Downloads`、`Images` 和 `Videos`。
* **客户端应用：** 一种定制化的应用，它会调用 `ACTION_CREATE_DOCUMENT`、`ACTION_OPEN_DOCUMENT` 和 `ACTION_OPEN_DOCUMENT_TREE` intent 操作并接收文档提供程序返回的文件。
* **选择器：** 一种系统界面，可让用户访问所有文档提供程序内满足客户端应用搜索条件的文档。

以下为 SAF 提供的部分功能：
* 让用户浏览所有文档提供程序的内容，而不仅仅是单个应用的内容。
* 让应用获得对文档提供程序所拥有文档的长期、持续访问权限。用户可通过此访问权限添加、修改、保存和删除提供程序中的文件。
* 支持多个用户帐号和临时根目录，如只有在插入 U 盘后才会出现的“USB 存储提供程序”。

本文记录日常用的比较多的**客户端应用**，应用经常会出现让用户在手机中选择图片作为头像这一需求。

在 `Android 4.3` 及更低版本中，如果您想让应用从其他应用中检索文件
用户看到一个系统选择器，供其浏览文档提供器并选择将执行存储相关操作的位置或文档。
应用获得对代表用户所选位置或文档的 URI 的读写访问权限。，则该应用必须调用 `ACTION_PICK` 或 `ACTION_GET_CONTENT` 等 intent。然后，用户必须选择一个要从中选取文件的应用，并且所选应用必须提供用户界面，以便用户浏览和选择可用文件。

在 `Android 4.4（API 级别 19）`及更高版本中，还可选择使用 `ACTION_OPEN_DOCUMENT` intent，此 intent 会显示由系统控制的选择器界面，以便用户浏览其他应用提供的所有文件。借助此界面，用户便可从任何受支持的应用中选择文件。

在 `Android 5.0（API 级别 21）`及更高版本中，还可以使用 `ACTION_OPEN_DOCUMENT_TREE` intent，借助此 intent，用户可以选择供客户端应用访问的目录。

需要注意的是:
`ACTION_OPEN_DOCUMENT` 并非用于替代 `ACTION_GET_CONTENT` 要按照应用的使用场景按需使用。

* 如果只想让应用读取/导入数据，使用 `ACTION_GET_CONTENT`。使用此方法时，应用会导入数据（如图片文件）的副本。
* 如果想让应用获得对文档提供程序所拥有文档的长期、持续访问权限，需要使用 `ACTION_OPEN_DOCUMENT`。例如，照片编辑应用可让用户编辑存储在文档提供程序中的图片。

**由于用户参与选择您的应用可以访问的文件或目录，因此该机制`无需任何系统权限`，同时用户控制和隐私保护也得到了增强。此外，这些文件存储在应用专属目录和媒体库之外，在应用卸载后仍会保留在设备上。**

比如让用户在手机中选择一张图片作为头像。

```java
private const val REQUEST_CODE_PICK_IMAGE = 0x02
private fun openDocument() {
    val pickIntent = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
        addCategory(Intent.CATEGORY_OPENABLE)
        type = "*/*"
        putExtra(Intent.EXTRA_MIME_TYPES, arrayOf("image/*"))
    }
    startActivityForResult(pickIntent, REQUEST_CODE_PICK_IMAGE)
}

override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == REQUEST_CODE_PICK_IMAGE && resultCode == Activity.RESULT_OK) {
        val pickedUri = data?.data
        //TODO 处理uri
    }
}
```
**注意：如果应用需要访问外部存储中的媒体文件，考虑使用[媒体库](https://developer.android.com/training/data-storage/shared/media?hl=zh-cn)，比如上面的查询外部存储上的视频。**

> 如果应用使用媒体库，必须请求 `READ_EXTERNAL_STORAGE`权限才能访问其他应用的媒体文件，包括由本应用创建的媒体文件。

# 小结

Android 10（API 29）开启了分区存储，虽然可以在Manifest通过声明`android:requestLegacyExternalStorage="true"`来暂缓适配，但是到了Android11（API30）已经强制执行分区存储。所以建议对存储空间的操作做好分区存储的适配。

Android 10（API 29）之前拿到`WRITE_EXTERNAL_STORAGE`就能对外部存储狂轰滥炸了，比如你SD卡上一堆莫名其妙的目录，一个应用可能在你SD卡上创建了10几个文件夹，好消息是Android10之后这种情况不会出现了，应用只能操作系统单独为应用准备的外部存储目录。Google是这么解释的。

> 在搭载 Android 9（API 级别 28）或更低版本的设备上，只要其他应用具有相应的存储权限，任何应用都可以访问外部存储空间中的应用专属文件。为了让用户更好地管理自己的文件并减少混乱，以 Android 10（API 级别 29）及更高版本为目标平台的应用在默认情况下被授予了对外部存储空间的分区访问权限（即分区存储）。启用分区存储后，应用将无法访问属于其他应用的应用专属目录。

建议：
* 如果不是媒体类应用建议使用`getExternalFilesDir`或者`getExternalCacheDir`，如果应用数据比较隐私可以使用`getFilesDir`或者`getCacheDir`，
应用访问自己的存储目录**不需要任何权限。**
* 参考[Google的存储空间用例和最佳用法](https://developer.android.com/training/data-storage/use-cases)。











