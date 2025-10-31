# CXR-M SDK 简介

**CXR-M SDK** 是面向移动端的开发工具包，主要用于构建手机端与 **Rokid Glasses** 的控制和协同应用。  
开发者可以通过 CXR-M SDK 与眼镜建立稳定连接，实现数据通信、实时音视频获取以及场景自定义。  
它适合需要在手机端进行界面交互、远程控制或与眼镜端配合完成复杂功能的应用。  
目前 **CXR-M SDK** 仅提供 **Android** 版本。

---

## CXR SDK 与 Rokid Glasses 架构

> 图片1：CXR SDK 与 Glasses 架构图

---

## Rokid Glasses 设备连接与管理

开发者可以使用 **CXR-M SDK** 与 **Rokid Glasses** 进行连接，  
并可获取 **Rokid Glasses** 的设备状态和进行设备管理操作。

---

## Rokid Glasses 自定义场景交互

开发者可通过 **CXR-M SDK** 快速接入 **YodaOS-Sprite** 操作系统定义的场景交互流程，  
并根据 **YodaOS-Sprite** 的交互场景定义，进行自定义功能开发。

---

## 自定义 AI 助手：Rokid Glasses Assist Service

开发者可通过 **CXR-M SDK** 高效使用 **Rokid Assist Service** 提供的各类服务，  
包括文件互传、录音、拍照等功能。

### 可用功能
- 录音功能  
- 拍照功能  
- 录像功能  
- 飞传功能

---

### Tips
**Rokid Assist Service** 是基于 **YodaOS-Sprite** 提供的一系列服务项。  
若这些服务被禁用，则无法在 **CXR-M SDK** 中使用。

# SDK 导入指南

本章节以 **Kotlin DSL（build.gradle.kts）** 为示例说明 **CXR-M SDK** 的导入与配置步骤。

---

## 一、配置 Maven 仓库

**CXR-M SDK** 采用 **Maven 在线仓库** 管理 SDK 包。

**Maven 仓库地址：**  
```

[https://maven.rokid.com/repository/maven-public/](https://maven.rokid.com/repository/maven-public/)

````

### 步骤

找到项目根目录下的 `settings.gradle.kts` 文件，  
在 `dependencyResolutionManagement` 节点的 `repositories` 中添加 Rokid Maven 仓库。

```kotlin
pluginManagement {
    repositories {
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        maven {
            url = uri("https://maven.rokid.com/repository/maven-public/")
        }
        google()
        mavenCentral()
    }
}
````

---

## 二、依赖导入

### 主依赖

在 `build.gradle.kts` 文件中添加以下依赖。

```kotlin
dependencies {
    implementation("com.rokid.cxr:client-m:1.0.1-20250812.080117-2")
}
```

### SDK 最低版本要求

```kotlin
android {
    defaultConfig {
        minSdk = 28
    }
}
```

### 其他依赖项

如项目中已有相同依赖但版本不同，请优先使用下列 SDK 推荐版本：

```kotlin
implementation("com.squareup.retrofit2:retrofit:2.9.0")
implementation("com.squareup.retrofit2:converter-gson:2.9.0")
implementation("com.squareup.okhttp3:okhttp:4.9.3")
implementation("org.jetbrains.kotlin:kotlin-stdlib:2.1.0")
implementation("com.squareup.okio:okio:2.8.0")
implementation("com.google.code.gson:gson:2.10.1")
implementation("com.squareup.okhttp3:logging-interceptor:4.9.1")
```

---

## 三、权限申请

### 1. 声明权限

**CXR-M SDK** 需要网络、Wi-Fi 与蓝牙权限（蓝牙权限需同步申请 `FINE_LOCATION`）。
以下为最小权限集示例，需在 `AndroidManifest.xml` 中声明：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
    <uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.INTERNET" />

    <application>
        <!-- Other Settings -->
    </application>
</manifest>
```

---

### 2. 动态申请权限

在使用 **CXR-M SDK** 前，必须动态申请必要权限。
若权限不足，SDK 将不可用。

以下为示例代码：

```kotlin
class MainActivity : AppCompatActivity() {

    companion object {
        const val TAG = "MainActivity"
        const val REQUEST_CODE_PERMISSIONS = 100

        private val REQUIRED_PERMISSIONS = mutableListOf(
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.BLUETOOTH,
            Manifest.permission.BLUETOOTH_ADMIN,
        ).apply {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
                add(Manifest.permission.BLUETOOTH_SCAN)
                add(Manifest.permission.BLUETOOTH_CONNECT)
            }
        }.toTypedArray()
    }

    private val permissionGrantedResult = MutableLiveData<Boolean?>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        permissionGrantedResult.postValue(null)
        requestPermissions(REQUIRED_PERMISSIONS, REQUEST_CODE_PERMISSIONS)

        permissionGrantedResult.observe(this) {
            if (it == true) {
                // Permission All Granted
            } else {
                // Some Permission Denied or Not Started
            }
        }
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == REQUEST_CODE_PERMISSIONS.hashCode()) {
            val allGranted = grantResults.all { it == PackageManager.PERMISSION_GRANTED }
            permissionGrantedResult.postValue(allGranted)
        }
    }
}
```

---

**注意**
在权限申请失败的情况下，CXR-M SDK 所有功能将不可使用。

# 设备连接

> 在阅读本章节前，请确保已完成《SDK 导入》章节的配置。

---

## 一、Bluetooth 连接

### 1. 查找蓝牙设备

通过 Android 标准 **Bluetooth 接口** 进行设备扫描。  
可使用 UUID `00009100-0000-1000-8000-00805f9b34fb` 来过滤 **Rokid** 设备。

以下示例展示了如何扫描并检测 Rokid Glasses 设备。

```kotlin
package com.rokid.cxrandroiddocsample.helpers
// imports

/**
 * Bluetooth Helper
 * @param context Activity Register Context
 * @param initStatus Init Status
 * @param deviceFound Device Found
 */
class BluetoothHelper(
    val context: AppCompatActivity,
    val initStatus: (INIT_STATUS) -> Unit,
    val deviceFound: () -> Unit
) {
    companion object {
        const val TAG = "Rokid Glasses CXR-M"
        const val REQUEST_CODE_PERMISSIONS = 100

        private val REQUIRED_PERMISSIONS = mutableListOf(
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.BLUETOOTH,
            Manifest.permission.BLUETOOTH_ADMIN,
        ).apply {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
                add(Manifest.permission.BLUETOOTH_SCAN)
                add(Manifest.permission.BLUETOOTH_CONNECT)
            }
        }.toTypedArray()

        enum class INIT_STATUS { NotStart, INITING, INIT_END }
    }

    val scanResultMap = ConcurrentHashMap<String, BluetoothDevice>()
    val bondedDeviceMap = ConcurrentHashMap<String, BluetoothDevice>()

    private var manager: BluetoothManager? = null
    private var adapter: BluetoothAdapter? = null
    private val scanner get() = adapter?.bluetoothLeScanner
    val permissionResult = MutableLiveData<Boolean>()

    @SuppressLint("MissingPermission")
    fun startScan() {
        scanResultMap.clear()
        adapter?.bondedDevices?.forEach { d ->
            d.name?.let {
                if (it.contains("Glasses", false)) bondedDeviceMap[it] = d
                deviceFound.invoke()
            }
        }
        scanner?.startScan(
            listOf(
                ScanFilter.Builder()
                    .setServiceUuid(ParcelUuid.fromString("00009100-0000-1000-8000-00805f9b34fb"))
                    .build()
            ),
            ScanSettings.Builder().build(),
            scanListener
        )
    }

    val scanListener = object : ScanCallback() {
        override fun onScanResult(callbackType: Int, result: ScanResult?) {
            result?.device?.name?.let {
                scanResultMap[it] = result.device
                deviceFound.invoke()
            }
        }
    }
}
````

---

### 2. 初始化蓝牙设备

通过 `CxrApi` 的 `initBluetooth` 方法进行蓝牙初始化。

```kotlin
fun initDevice(context: Context, device: BluetoothDevice) {
    CxrApi.getInstance().initBluetooth(context, device, object : BluetoothStatusCallback {
        override fun onConnectionInfo(
            socketUuid: String?,
            macAddress: String?,
            rokidAccount: String?,
            glassesType: Int
        ) {
            socketUuid?.let { uuid ->
                macAddress?.let { address ->
                    connect(context, uuid, address)
                } ?: Log.e(TAG, "macAddress is null")
            } ?: Log.e(TAG, "socketUuid is null")
        }

        override fun onConnected() {}
        override fun onDisconnected() {}
        override fun onFailed(error: ValueUtil.CxrBluetoothErrorCode?) {}
    })
}
```

**回调说明：**

| 方法                 | 说明                           |
| -------------------- | ------------------------------ |
| `onConnected()`      | 蓝牙连接成功回调               |
| `onDisconnected()`   | 蓝牙连接丢失回调               |
| `onConnectionInfo()` | 设备信息更新                   |
| 参数说明：           |                                |
| `socketUuid`         | 设备 UUID                      |
| `macAddress`         | 设备 MAC 地址                  |
| `rokidAccount`       | Rokid 账号                     |
| `glassesType`        | 设备类型（0-无显示，1-有显示） |

---

### 3. 连接蓝牙模块

使用 `CxrApi.connectBluetooth()` 进行连接。

```kotlin
fun connect(context: Context, socketUuid: String, macAddress: String) {
    CxrApi.getInstance().connectBluetooth(context, socketUuid, macAddress, object : BluetoothStatusCallback {
        override fun onConnectionInfo(socketUuid: String?, macAddress: String?, rokidAccount: String?, glassesType: Int) {}
        override fun onConnected() { Log.d(TAG, "Connected") }
        override fun onDisconnected() { Log.d(TAG, "Disconnected") }
        override fun onFailed(error: ValueUtil.CxrBluetoothErrorCode?) { Log.e(TAG, "Failed") }
    })
}
```

---

### 4. 获取蓝牙连接状态

```kotlin
fun getConnectionStatus(): Boolean {
    return CxrApi.getInstance().isBluetoothConnected
}
```

返回值说明：

* `true`：蓝牙模块已连接
* `false`：蓝牙模块未连接

---

### 5. 反初始化蓝牙

```kotlin
fun deInit() {
    CxrApi.getInstance().deinitBluetooth()
}
```

---

### 6. 蓝牙重连

蓝牙重连同样使用 `connectBluetooth()` 方法。

```kotlin
fun reconnect(context: Context, socketUuid: String, macAddress: String) {
    CxrApi.getInstance().connectBluetooth(context, socketUuid, macAddress, object : BluetoothStatusCallback {
        override fun onConnected() { Log.d(TAG, "Connected") }
        override fun onDisconnected() { Log.d(TAG, "Disconnected") }
        override fun onFailed(p0: ValueUtil.CxrBluetoothErrorCode?) { Log.e(TAG, "Failed") }
        override fun onConnectionInfo(socketUuid: String?, macAddress: String?, rokidAccount: String?, glassesType: Int) {}
    })
}
```

---

## 二、Wi-Fi 连接

> 注意：使用 Wi-Fi 模块前，必须完成蓝牙连接。
> Wi-Fi 模块耗能较高，仅在必要时启用。

---

### 1. 初始化 Wi-Fi 通信模块

通过 `initWifiP2P()` 初始化 Wi-Fi 通信模块。

```kotlin
fun initWifi(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().initWifiP2P(object : WifiP2PStatusCallback {
        override fun onConnected() { Log.d(TAG, "onConnected") }
        override fun onDisconnected() { Log.d(TAG, "onDisconnected") }
        override fun onFailed(errorCode: ValueUtil.CxrWifiErrorCode?) {}
    })
}
```

**回调说明：**

| 方法               | 说明           |
| ------------------ | -------------- |
| `onConnected()`    | Wi-Fi 连接成功 |
| `onDisconnected()` | Wi-Fi 断开连接 |
| `onFailed()`       | Wi-Fi 连接失败 |

**错误码：**

* `CxrWifiErrorCode.WIFI_DISABLED`：Wi-Fi 未启用
* `CxrWifiErrorCode.WIFI_CONNECT_FAILED`：P2P 连接失败
* `CxrWifiErrorCode.UNKNOWN`：未知错误

---

### 2. 获取 Wi-Fi 模块连接状态

```kotlin
fun getWiFiConnectionStatus(): Boolean {
    return CxrApi.getInstance().isWifiP2PConnected
}
```

返回值说明：

* `true`：Wi-Fi 模块已连接
* `false`：Wi-Fi 模块未连接

---

### 3. 反初始化 Wi-Fi 通信模块

```kotlin
private fun deinitWifi() {
    CxrApi.getInstance().deinitWifiP2P()
}
```

---

**总结：**
CXR-M SDK 提供完整的蓝牙与 Wi-Fi 通信接口。开发者可通过 `CxrApi` 对设备进行扫描、连接、状态查询与反初始化操作，实现移动端与 Rokid Glasses 的稳定连接与协同控制。

# 控制与监听设备状态

> 注意：涉及眼镜端硬件信息的操作，需确保手机端与眼镜端的 **蓝牙通道** 已建立连接。

---

## 一、获取眼镜信息

通过 `fun getGlassInfo(callback: GlassInfoResultCallback): ValueUtil.CxrStatus` 方法获取眼镜的完整信息。

```kotlin
fun getGlassesInfo() {
    CxrApi.getInstance().getGlassInfo(object : GlassInfoResultCallback {
        override fun onGlassInfoResult(
            status: ValueUtil.CxrStatus?,
            glassesInfo: GlassInfo?
        ) {
            // Handle Glasses Info
        }
    })
}
````

**回调参数说明：**

* `status`：响应状态（`RESPONSE_SUCCEED`、`RESPONSE_INVALID`、`RESPONSE_TIMEOUT`）
* `glassesInfo`：眼镜信息对象（包含型号、固件版本、电量、亮度、音量等）

---

## 二、同步眼镜时间与时区

使用 `fun setGlassTime(): CxrStatus` 方法同步时间。

```kotlin
fun setTime(): CxrStatus {
    return CxrApi.getInstance().setGlassTime()
}
```

**返回值：**

* `REQUEST_SUCCEED`：同步成功
* `REQUEST_WAITING`：等待眼镜响应
* `REQUEST_FAILED`：同步失败

---

## 三、设置与监听眼镜亮度

亮度范围为 `[0, 15]`。
通过 `setGlassBrightness()` 设置亮度，并通过 `BrightnessUpdateListener` 监听变化。

```kotlin
private val glassesBrightnessUpdateListener = object : BrightnessUpdateListener {
    override fun onBrightnessUpdated(brightness: Int) {
        // brightness value [0–15]
    }
}

fun getGlassesBrightness(set: Boolean) {
    CxrApi.getInstance().setBrightnessUpdateListener(if (set) glassesBrightnessUpdateListener else null)
}

fun setBrightness(brightness: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().setGlassBrightness(brightness)
}
```

---

## 四、设置与监听眼镜音量

音量范围为 `[0, 15]`。
通过 `setGlassVolume()` 设置音量，并通过 `VolumeUpdateListener` 监听变化。

```kotlin
private val volumeUpdate = object : VolumeUpdateListener {
    override fun onVolumeUpdated(volume: Int) {
        // volume value [0–15]
    }
}

fun getGlassesVolume(set: Boolean) {
    CxrApi.getInstance().setVolumeUpdateListener(if (set) volumeUpdate else null)
}

fun setVolume(volume: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().setGlassVolume(volume)
}
```

---

## 五、设置眼镜音效模式

可选择三种音效模式：

* `"AdiMode0"`：洪亮模式
* `"AdiMode1"`：韵律模式
* `"AdiMode2"`：博客模式

```kotlin
fun setSoundEffect(value: String): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().setSoundEffect(value)
}
```

---

## 六、监听眼镜电量变化

通过 `BatteryLevelUpdateListener` 监听电量变化和充电状态。

```kotlin
CxrApi.getInstance().setBatteryLevelUpdateListener(object : BatteryLevelUpdateListener {
    override fun onBatteryLevelUpdated(level: Int, charging: Boolean) {
        // level: [0–100], charging: true/false
    }
})
```

---

## 七、设置自动熄屏时间

通过 `fun setScreenOffTimeout(seconds: Long)` 设置自动熄屏时间，单位为秒。

```kotlin
fun setScreenOffTimeout(seconds: Long): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().setScreenOffTimeout(seconds)
}
```

---

## 八、设置自动关机时间

通过 `fun setPowerOffTimeout(minutes: Int)` 设置自动关机时间，单位为分钟。

```kotlin
fun setPowerOffTimeout(minutes: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().setPowerOffTimeout(minutes)
}
```

---

## 九、通知眼镜重启

通过 `fun notifyGlassReboot(): ValueUtil.CxrStatus?` 控制眼镜重启。

```kotlin
fun rebootGlasses(): ValueUtil.CxrStatus {
    return CxrApi.getInstance().notifyGlassReboot()
}
```

---

## 十、通知眼镜关机

通过 `fun notifyGlassShutdown(): ValueUtil.CxrStatus?` 控制眼镜关机。

```kotlin
fun shutdownGlasses(): ValueUtil.CxrStatus {
    return CxrApi.getInstance().notifyGlassShutdown()
}
```

---

**总结：**
通过 `CxrApi` 提供的状态控制与监听接口，开发者可以实时获取眼镜设备的状态信息并进行调节操作，包括时间同步、亮度音量管理、电量监听、音效模式切换以及系统级控制（重启与关机）。
确保在操作前蓝牙连接稳定，以保证指令传输可靠。

# 拍照 / 录像 / 录音

> 拍照和录像属于高功耗操作，使用前请确保蓝牙连接稳定。  
> 根据场景需求不同，CXR-M SDK 提供三种拍照方式。

---

## 一、拍照

### 1. 设置按键拍照参数

通过 `fun setPhotoParams(width: Int, height: Int): ValueUtil.CxrStatus` 设置单机按键拍照参数。

```kotlin
fun setPhotoParams(width: Int, height: Int): ValueUtil.CxrStatus {
    return CxrApi.getInstance().setPhotoParams(width, height)
}
````

---

### 2. AI 场景拍照

通过以下两个接口控制相机与拍照：

* 打开相机：`openGlassCamera(width, height, quality)`
* 拍照：`takeGlassPhoto(width, height, quality, callback)`

分辨率可选值包括：
`[4032x3024, 4000x3000, 4032x2268, 3264x2448, 3200x2400, 2268x3024, 2876x2156, 2688x2016, 2582x1936, 2400x1800, 1800x2400, 2560x1440, 2400x1350, 2048x1536, 2016x1512, 1920x1080, 1600x1200, 1440x1080, 1280x720, 720x1280, 1024x768, 800x600, 648x648, 854x480, 800x480, 640x480, 480x640, 352x288, 320x240, 320x180, 176x144]`

**建议：** 由于图片通过蓝牙传输，请选择较低分辨率。

```kotlin
// Photo result callback
private val result = object : PhotoResultCallback {
    override fun onPhotoResult(
        status: ValueUtil.CxrStatus?,
        photo: ByteArray?
    ) {
        // Handle WebP photo data
    }
}

// 打开相机
fun aiOpenCamera(width: Int, height: Int, quality: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().openGlassCamera(width, height, quality)
}

// 拍照
fun takePhoto(width: Int, height: Int, quality: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().takeGlassPhoto(width, height, quality, result)
}
```

---

### 3. 唤起相机拍照（带路径回调）

使用 `fun takePhoto(width, height, quality, callback: PhotoPathCallback)` 可直接唤起相机并获取图片存储路径。

```kotlin
// Photo path callback
private val photoPathResult = object : PhotoPathCallback {
    override fun onPhotoPath(status: ValueUtil.CxrStatus?, path: String?) {
        // Handle photo file path
    }
}

fun takePhotoPath(width: Int, height: Int, quality: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().takeGlassPhoto(width, height, quality, photoPathResult)
}
```

---

## 二、录像

### 1. 设置录像参数

通过 `fun setVideoParams(duration, fps, width, height, unit)` 设置录像参数。

```kotlin
fun setVideoParams(duration: Int, fps: Int, width: Int, height: Int, unit: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().setVideoParams(duration, fps, width, height, unit)
}
```

* `duration`：录像时长
* `fps`：帧率（支持 30）
* `unit`：0 为分钟，1 为秒

---

### 2. 开启 / 关闭录像

通过 `fun controlScene()` 控制录像场景开关。
`sceneType` 传入 `ValueUtil.CxrSceneType.VIDEO_RECORD`。

```kotlin
fun openOrCloseVideoRecord(openOrClose: Boolean): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().controlScene(ValueUtil.CxrSceneType.VIDEO_RECORD, openOrClose, null)
}
```

---

## 三、录音

### 1. 设置录音监听器

通过 `AudioStreamListener` 监听录音流数据。

```kotlin
private val audioStreamListener = object : AudioStreamListener {
    override fun onStartAudioStream(codecType: Int, streamType: String?) {
        // codecType: 1-pcm, 2-opus
    }

    override fun onAudioStream(data: ByteArray?, offset: Int, length: Int) {
        // Process audio data
    }
}

fun setVideoStreamListener(set: Boolean) {
    CxrApi.getInstance().setAudioStreamListener(if (set) audioStreamListener else null)
}
```

---

### 2. 开启录音

通过 `fun openAudioRecord(codecType, streamType)` 开启录音。
`streamType` 例如 `"AI_assistant"`。

```kotlin
fun openAudioRecord(codecType: Int, streamType: String?): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().openAudioRecord(codecType, streamType)
}
```

---

### 3. 关闭录音

通过 `fun closeAudioRecord(streamType)` 停止录音。

```kotlin
fun closeAudioRecord(streamType: String): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().closeAudioRecord(streamType)
}
```

---

**总结：**
CXR-M SDK 的拍照、录像与录音接口覆盖了从参数配置、操作控制到结果回调的全流程。

* 拍照支持按键触发、AI 场景拍照与路径回调方式。
* 录像通过控制场景接口实现。
* 录音提供实时音频流监听。
  使用时需关注功耗与数据传输效率，建议结合实际场景合理配置分辨率与音视频参数。

# 数据操作

> 注意：进行数据传输或同步操作前，必须确保 **蓝牙或 Wi-Fi 通信模块** 已正确连接。

---

## 一、向眼镜端发送数据

可通过 `fun sendStream(type, stream, fileName, cb)` 向眼镜端发送流数据。

```kotlin
val streamCallback = object : SendStatusCallback {
    override fun onSendSucceed() {
        // Send succeed
    }

    override fun onSendFailed(errorCode: ValueUtil.CxrSendErrorCode?) {
        // Handle send failure
    }
}

fun sendStream(
    type: ValueUtil.CxrStreamType,
    stream: ByteArray,
    fileName: String
): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().sendStream(type, stream, fileName, streamCallback)
}
````

**参数说明：**

* `type`：数据类型，如 `ValueUtil.CxrStreamType.WORD_TIPS`（提示文本）
* `stream`：字节流数据
* `fileName`：文件名
* `cb`：状态回调

**回调接口：**

* `onSendSucceed()`：发送成功
* `onSendFailed()`：发送失败，返回 `CxrSendErrorCode`

---

## 二、读取眼镜端未同步媒体文件

通过 `fun getUnsyncNum(cb)` 获取眼镜端未同步的音频、图片、视频数量。

```kotlin
private val unSyncCallback = object : UnsyncNumResultCallback {
    override fun onUnsyncNumResult(
        status: ValueUtil.CxrStatus?,
        audioNum: Int,
        pictureNum: Int,
        videoNum: Int
    ) {
        // Handle unsynced file count
    }
}

fun getUnsyncNum(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().getUnsyncNum(unSyncCallback)
}
```

**回调结果：**

* `audioNum`：未同步音频数量
* `pictureNum`：未同步图片数量
* `videoNum`：未同步视频数量

---

## 三、监听眼镜端媒体文件更新

通过设置 `MediaFilesUpdateListener` 接口，可在眼镜端生成新媒体文件时收到通知。

```kotlin
private val mediaFileUpdateListener = object : MediaFilesUpdateListener {
    override fun onMediaFilesUpdated() {
        // Triggered when media files updated
    }
}

fun setMediaFilesUpdateListener(set: Boolean) {
    CxrApi.getInstance().setMediaFilesUpdateListener(if (set) mediaFileUpdateListener else null)
}
```

---

## 四、同步媒体文件（需启用 Wi-Fi 模块）

### 4.1 同步指定类型的所有文件

通过 `fun startSync(savePath, types, callback)` 从眼镜端同步媒体文件。

```kotlin
private val syncCallback = object : SyncStatusCallback {
    override fun onSyncStart() {}
    override fun onSingleFileSynced(fileName: String?) {}
    override fun onSyncFailed() {}
    override fun onSyncFinished() {}
}

fun startSync(savePath: String, types: Array<ValueUtil.CxrMediaType>): Boolean {
    return CxrApi.getInstance().startSync(savePath, types, syncCallback)
}
```

**参数说明：**

* `savePath`：文件保存路径（需存储权限）
* `types`：媒体类型数组，可同时指定多个

  * `AUDIO`：音频
  * `PICTURE`：图片
  * `VIDEO`：视频
  * `ALL`：全部文件

**回调接口：**

* `onSyncStart()`：开始同步
* `onSingleFileSynced(fileName)`：单个文件同步完成
* `onSyncFailed()`：同步失败
* `onSyncFinished()`：同步完成

---

### 4.2 同步单个文件

通过 `fun syncSingleFiles(savePath, mediaType, fileName, callback)` 同步指定文件。

```kotlin
private val syncCallback = object : SyncStatusCallback {
    override fun onSyncStart() {}
    override fun onSingleFileSynced(fileName: String?) {}
    override fun onSyncFailed() {}
    override fun onSyncFinished() {}
}

fun syncSingleFiles(
    savePath: String,
    mediaType: ValueUtil.CxrMediaType,
    fileName: String
): Boolean {
    return CxrApi.getInstance().syncSingleFile(savePath, mediaType, fileName, syncCallback)
}
```

**参数说明：**

* `savePath`：存储路径
* `mediaType`：媒体类型（如 `PICTURE`）
* `fileName`：文件名或路径

---

### 4.3 停止同步

在同步过程中可调用 `fun stopSync()` 停止当前同步任务。

```kotlin
private fun stopSync() {
    CxrApi.getInstance().stopSync()
}
```

---

**总结：**
CXR-M SDK 支持蓝牙与 Wi-Fi 通道的数据操作，开发者可：

* 向眼镜端发送文本或文件流；
* 查询与监听媒体更新；
* 执行全量或单文件同步；
* 控制同步进程。

所有操作均返回 `CxrStatus` 以反映请求结果，开发者应结合状态码与回调接口实现健壮的同步逻辑。

# AI 场景操作

> 注意：使用 AI 场景功能前，需确保手机与 Rokid Glasses 的蓝牙通道已连接。

---

## 一、监听眼镜端 AI 事件

通过 `fun setAiEventListener()` 设置 `AiEventListener`，可监听眼镜端的 AI 场景事件。

```kotlin
private val aiEventListener = object : AiEventListener {
    override fun onAiKeyDown() { /* Key long pressed */ }
    override fun onAiKeyUp() { /* Key released */ }
    override fun onAiExit() { /* AI Scene exited */ }
}

fun setAiEventListener(set: Boolean) {
    CxrApi.getInstance().setAiEventListener(if (set) aiEventListener else null)
}
````

**回调事件：**

* `onAiKeyDown()`：按下 AI 功能键
* `onAiKeyUp()`：释放 AI 功能键
* `onAiExit()`：退出 AI 场景

---

## 二、发送 Exit 事件至眼镜端

通过 `fun sendExitEvent()` 主动退出 AI 场景。

```kotlin
fun sendExitEvent(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().sendExitEvent()
}
```

---

## 三、发送 ASR 内容至眼镜端

当手机端获取到 ASR 结果后，可将识别内容推送至眼镜端。
若识别为空或出错，也需同步相应状态。

```kotlin
fun sendAsrContent(content: String): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().sendAsrContent(content)
}

fun notifyAsrNone(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().notifyAsrNone()
}

fun notifyAsrError(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().notifyAsrError()
}

fun notifyAsrEnd(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().notifyAsrEnd()
}
```

**说明：**

* `sendAsrContent()`：发送识别结果文本
* `notifyAsrNone()`：通知“无内容”
* `notifyAsrError()`：通知识别错误
* `notifyAsrEnd()`：通知识别结束

---

## 四、AI 流程中的相机操作

在 AI 场景中可调用相机获取实时图片。

```kotlin
private val result = object : PhotoResultCallback {
    override fun onPhotoResult(status: ValueUtil.CxrStatus?, photo: ByteArray?) {
        // photo: WebP image data
    }
}

fun aiOpenCamera(width: Int, height: Int, quality: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().openGlassCamera(width, height, quality)
}

fun takePhoto(width: Int, height: Int, quality: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().takeGlassPhoto(width, height, quality, result)
}
```

**参数说明：**

* `width`、`height`：分辨率
* `quality`：压缩质量 [0–100]
* `photo`：WebP 图片数据字节数组

---

## 五、AI 流程中的 AI 返回结果

当 AI 返回文本结果后，可通过 `TTS` 进行播报。
使用以下接口将结果发送至眼镜端，并在播放完成后通知结束。

```kotlin
fun sendTTSContent(content: String): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().sendTtsContent(content)
}

fun notifyTtsAudioFinished(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().notifyTtsAudioFinished()
}
```

**说明：**

* `sendTTSContent()`：将 AI 结果文本发送到眼镜端进行语音播报
* `notifyTtsAudioFinished()`：通知 TTS 播放完成

---

## 六、AI 过程错误处理

在 AI 执行过程中，如遇网络或图像上传问题，可调用以下接口上报错误状态：

```kotlin
fun notifyNoNetwork(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().notifyNoNetwork()
}

fun notifyPicUploadError(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().notifyPicUploadError()
}

fun notifyAiError(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().notifyAiError()
}
```

**错误类型：**

* `notifyNoNetwork()`：无网络连接
* `notifyPicUploadError()`：图片上传失败
* `notifyAiError()`：AI 请求错误

---

**总结：**
CXR-M SDK 提供完整的 AI 场景交互接口体系，涵盖：

* AI 事件监听；
* ASR 内容推送与状态通知；
* AI 过程中的拍照与视觉输入；
* TTS 结果播报与完成通知；
* 网络与 AI 异常处理。

通过这些接口，开发者可快速构建基于 Rokid Glasses 的智能语音与视觉交互体验。

# 提词器场景

> 注意：在使用提词器功能前，请确保手机与 Rokid Glasses 蓝牙连接已建立。

---

## 一、打开或关闭提词器场景

使用以下接口控制提词器场景的打开或关闭：

```kotlin
fun openOrCloseWordTips(toOpen: Boolean): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().controlScene(ValueUtil.CxrSceneType.WORD_TIPS, toOpen, null)
}
````

**参数说明：**

* `toOpen = true`：打开提词器
* `toOpen = false`：关闭提词器

**返回值：**

* `REQUEST_SUCCEED`：请求成功
* `REQUEST_WAITING`：等待响应
* `REQUEST_FAILED`：请求失败

---

## 二、发送提词器数据

可通过 `sendStream()` 方法发送提词器文本内容。

```kotlin
private val sendCallback = object : SendStatusCallback {
    override fun onSendSucceed() { /* Data sent successfully */ }
    override fun onSendFailed(p0: ValueUtil.CxrSendErrorCode?) { /* Handle failure */ }
}

fun setWordTipsText(text: String, fileName: String): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().sendStream(
        ValueUtil.CxrStreamType.WORD_TIPS,
        text.toByteArray(),
        fileName,
        sendCallback
    )
}
```

**参数说明：**

* `text`：提词器文本内容
* `fileName`：文件名，用于标识发送的内容
* 回调接口：`SendStatusCallback`

  * `onSendSucceed()`：发送成功
  * `onSendFailed()`：发送失败，返回错误码

---

## 三、设置提词器显示参数

通过以下接口配置提词器文本显示参数。

```kotlin
fun configWordTipsText(
    textSize: Float,
    lineSpace: Float,
    mode: String,
    startPointX: Int,
    startPointY: Int,
    width: Int,
    height: Int
): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().configWordTipsText(
        textSize, lineSpace, mode, startPointX, startPointY, width, height
    )
}
```

**参数说明：**

* `textSize`：文字大小
* `lineSpace`：行间距
* `mode`：

  * `"normal"`：普通模式
  * `"ai"`：AI 模式（当 ASR 内容抵达末尾自动滚动）
* `startPointX`、`startPointY`：起始位置坐标
* `width`、`height`：显示区域大小

**返回值：**

* `REQUEST_SUCCEED`：设置成功
* `REQUEST_WAITING`：等待响应
* `REQUEST_FAILED`：设置失败

---

## 四、发送提词器 ASR 结果

在提词器处于 `"ai"` 模式时，可通过以下接口发送 ASR 识别结果。当检测到 ASR 内容抵达末尾时，文本会自动上滑。

```kotlin
fun sendWordTipsAsrContent(content: String): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().sendAsrContent(content)
}
```

**参数说明：**

* `content`：ASR 识别结果字符串

**返回值：**

* `REQUEST_SUCCEED`：发送成功
* `REQUEST_WAITING`：等待响应
* `REQUEST_FAILED`：发送失败

---

### 总结

提词器（Word Tips）功能模块主要包括：

1. 打开/关闭提词器场景；
2. 发送并显示提词内容；
3. 配置文本显示参数（字体大小、行距、位置等）；
4. 在 AI 模式下根据 ASR 识别结果自动滚动。

通过 CXR-M SDK 的 `CxrApi` 接口，开发者可轻松实现 Rokid Glasses 提词功能的自定义与控制。

# 翻译场景

> 注意：进入翻译场景后，Rokid Glasses 将自动进入远场拾音模式，仅采集对话方语音内容，佩戴者语音将不会被拾取。

---

## 一、打开或关闭翻译场景

可通过以下接口打开或关闭翻译场景：

```kotlin
fun openOrCloseTranslation(toOpen: Boolean): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().controlScene(ValueUtil.CxrSceneType.Translation, toOpen, null)
}
````

**参数说明：**

* `toOpen = true`：打开翻译场景
* `toOpen = false`：关闭翻译场景

**返回值：**

* `REQUEST_SUCCEED`：操作成功
* `REQUEST_WAITING`：等待响应
* `REQUEST_FAILED`：操作失败

---

## 二、发送翻译内容给眼镜端

翻译场景用于展示对话方的实时翻译内容。
在该场景中，眼镜端显示来自手机端识别并翻译后的文本内容。

使用以下接口发送翻译内容：

```kotlin
fun sendTranslationContent(
    vadId: Int,
    subId: Int,
    temporary: Boolean,
    finished: Boolean,
    content: String
): ValueUtil.CxrStatus {
    return CxrApi.getInstance().sendTranslationContent(vadId, subId, temporary, finished, content)
}
```

**参数说明：**

* `vadId`：语音分段序号（VAD ID）
* `subId`：子序号，用于同一段语音的多次翻译更新
* `temporary`：

  * `true`：临时内容（实时中间翻译）
  * `false`：最终内容
* `finished`：

  * `true`：该段翻译结束
  * `false`：该段仍在更新
* `content`：翻译文本内容

**返回值：**

* `REQUEST_SUCCEED`：发送成功
* `REQUEST_WAITING`：等待响应
* `REQUEST_FAILED`：发送失败

---

## 三、配置翻译内容显示参数

通过以下接口可调整翻译文字在眼镜端的显示样式与布局。

```kotlin
fun configTranslationText(
    textSize: Int,
    startPointX: Int,
    startPointY: Int,
    width: Int,
    height: Int
): ValueUtil.CxrStatus {
    return CxrApi.getInstance().configTranslationText(textSize, startPointX, startPointY, width, height)
}
```

**参数说明：**

* `textSize`：文字大小
* `startPointX`、`startPointY`：起始坐标（左上角位置）
* `width`、`height`：显示区域宽高

**返回值：**

* `REQUEST_SUCCEED`：配置成功
* `REQUEST_WAITING`：等待响应
* `REQUEST_FAILED`：配置失败

---

### 总结

翻译场景（Translation Scene）是 Rokid Glasses 的远场语音翻译模块。
通过 CXR-M SDK，开发者可：

1. 打开或关闭翻译场景；
2. 发送实时翻译结果；
3. 配置翻译文字显示位置与样式。

该功能常用于会议同传、跨语言对话等实时翻译应用场景。

# 自定义页面场景

CXR-M SDK 提供自定义页面场景功能，允许开发者在**无需开发眼镜端应用**的情况下，通过手机端在 Rokid Glasses 上显示自定义界面内容。

---

## 一、打开自定义界面

使用以下接口可打开自定义界面：

```kotlin
fun openCustomView(content: String): ValueUtil.CxrStatus {
    return CxrApi.getInstance().openCustomView(content)
}
````

**参数说明：**

* `content`：以 JSON 格式定义的页面结构与样式。

**返回值：**

* `REQUEST_SUCCEED`：打开成功
* `REQUEST_WAITING`：等待响应
* `REQUEST_FAILED`：打开失败

---

## 二、监听自定义页面状态

设置 `CustomViewListener` 可监听自定义界面状态：

```kotlin
private val customViewListener = object : CustomViewListener {
    override fun onIconsSent() {}
    override fun onOpened() {}
    override fun onOpenFailed(p0: Int) {}
    override fun onUpdated() {}
    override fun onClosed() {}
}

fun setCustomViewListener(set: Boolean) {
    CxrApi.getInstance().setCustomViewListener(if (set) customViewListener else null)
}
```

**监听事件：**

* `onIconsSent()`：图标资源发送完成
* `onOpened()`：自定义界面打开成功
* `onOpenFailed()`：界面打开失败
* `onUpdated()`：界面更新成功
* `onClosed()`：界面关闭

---

## 三、更新自定义页面

通过以下接口更新页面内容：

```kotlin
fun updateCustomView(content: String): ValueUtil.CxrStatus {
    return CxrApi.getInstance().updateCustomView(content)
}
```

**参数：**

* `content`：更新内容的 JSON 描述，包含要修改的控件 `id` 及其属性。

---

## 四、关闭自定义页面

通过以下接口关闭自定义页面：

```kotlin
fun closeCustomView(): ValueUtil.CxrStatus {
    return CxrApi.getInstance().closeCustomView()
}
```

---

## 五、上传图片资源

若页面中包含图片元素，需提前上传，图片要求如下：

* 分辨率不超过 **128×128 px**
* 建议上传数量不超过 **10 张**
* 仅显示图片的**绿色通道**

上传接口：

```kotlin
fun sendCustomIcons(icons: List<IconInfo>): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().sendCustomViewIcons(icons)
}
```

**`IconInfo` 参数：**

* `name`：图片标识名（对应布局中 `name` 字段）
* `data`：图片的 Base64 数据

---

## 六、初始化页面 JSON 字段

页面使用 **JSON** 格式定义布局。
支持布局类型：`LinearLayout` 与 `RelativeLayout`
支持控件类型：`TextView`、`ImageView`

### 示例：自定义页面初始化 JSON

```json
{
  "type": "LinearLayout",
  "props": {
    "layout_width": "match_parent",
    "layout_height": "match_parent",
    "orientation": "vertical",
    "gravity": "center_horizontal",
    "paddingTop": "140dp",
    "paddingBottom": "100dp",
    "backgroundColor": "#FF000000"
  },
  "children": [
    {
      "type": "TextView",
      "props": {
        "id": "tv_title",
        "layout_width": "wrap_content",
        "layout_height": "wrap_content",
        "text": "Init Text",
        "textSize": "16sp",
        "textColor": "#FF00FF00",
        "textStyle": "bold",
        "marginBottom": "20dp"
      }
    },
    {
      "type": "RelativeLayout",
      "props": {
        "width": "match_parent",
        "height": "100dp",
        "backgroundColor": "#00000000",
        "padding": "10dp"
      },
      "children": [
        {
          "type": "ImageView",
          "props": {
            "id": "iv_icon",
            "layout_width": "60dp",
            "layout_height": "60dp",
            "name": "icon_name0",
            "layout_alignParentStart": "true",
            "layout_centerVertical": "true"
          }
        },
        {
          "type": "TextView",
          "props": {
            "id": "tv_text",
            "layout_width": "wrap_content",
            "layout_height": "wrap_content",
            "text": "Text to the end of Icon",
            "textSize": "16sp",
            "textColor": "#FF00FF00",
            "layout_toEndOf": "iv_icon",
            "layout_centerVertical": "true",
            "marginStart": "15dp"
          }
        }
      ]
    }
  ]
}
```

---

## 七、更新页面 JSON 字段

更新操作同样使用 JSON 结构。仅需传递需要更新的控件 `id` 与属性。

### 示例：更新内容

```json
[
  {
    "action": "update",
    "id": "tv_title",
    "props": {
      "text": "Update Text"
    }
  },
  {
    "action": "update",
    "id": "iv_icon",
    "props": {
      "name": "icon_name1"
    }
  }
]
```

---

### 总结

| 功能           | 方法                      | 说明                 |
| -------------- | ------------------------- | -------------------- |
| 打开自定义界面 | `openCustomView()`        | 显示自定义页面       |
| 监听页面状态   | `setCustomViewListener()` | 监听页面生命周期事件 |
| 更新页面内容   | `updateCustomView()`      | 动态修改控件属性     |
| 关闭页面       | `closeCustomView()`       | 关闭当前页面         |
| 上传图片资源   | `sendCustomIcons()`       | 上传图标资源至眼镜端 |

**适用场景：**

* 企业定制展示界面
* 智能提示页、状态监控页
* 远程任务提示或流程指引界面# FuckingRokidGlasses
