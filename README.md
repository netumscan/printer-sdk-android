# NETUM Printer SDK for Android

开发预览 **`0.3.0-preview.10`**，原生 SDK **`0.3.0`**、C ABI **`3/r2`**。
Android API 21+、Java 8；AAR 包含 `arm64-v8a`、`armeabi-v7a`、`x86_64`。
接入时核对精确分发版本、制品 SHA-256 和运行库身份；候选契约本身不证明已经线上发布。

[客户指南](docs/README.md) 包含权限、连接、图片、回调、错误码和完整公开 API。
本版移动 API 为需要迁移和重新编译的新契约；原生 C ABI 保持 27 个 C／23 个 JNI 导出。
SDK 实现源码不随 Android 分发提供，许可及第三方声明见 [LICENSE](LICENSE) 和 [LICENSE-NOTICE.md](LICENSE-NOTICE.md)。

## 依赖

实际发布后使用精确 Maven 坐标；发布前使用已核验的本地 AAR：

```kotlin
dependencyResolutionManagement {
    repositories { google(); mavenCentral() }
}
dependencies {
    implementation("com.netumscan:printer-sdk-android:0.3.0-preview.10")
}
```

SDK 静态链接私有 C++ 运行库并提供 consumer R8 规则。App 保留其他依赖需要的运行库，整套包装与 native 库应使用相同分发版本。

## 连接与图片

每个 `PrinterManager(context)` 同时管理一个设备。`scanBle` 返回发现设备，`bondedDevices` 返回系统已配对列表；权限、配对和用户选择由 App 处理。经典蓝牙使用 `connectSpp`，BLE 使用 `connectBle`。

设备原名可推荐内置 `PrinterModel`；改名的已收录设备可以显式选择型号，未知型号不得按 M90 猜测。通信参数由 SDK 内部管理。以下代码适用于调用方已确认属于 LT-M90 的设备：

```java
PrinterManager manager = new PrinterManager(context);
manager.connectBle(device, PrinterModel.LT_M90, connection -> {
    if (connection.outcome != ManagerResult.Outcome.SUCCESS) return;
    manager.printEncodedImage(pngJpeg, 50.0, 50.0, result -> {
        // Check result.code, completion and outputMayHaveOccurred.
    });
}, disconnected -> {
    // Update the App state; reconnect only after an explicit user action.
});
```

`printEncodedImage` 接收完整 PNG/JPEG `byte[]`；已有 `Bitmap` 可直接使用 `printImage(bitmap, widthMm, heightMm, callback)`，无需压缩后再解码。宽高为页面毫米，SDK 按型号精确点密度换算，不使用图片 DPI 或屏幕 density 决定纸面尺寸。

两入口返回前取得独立快照；复制期间不得并发修改输入，返回后调用方可管理原图。SDK 后台解码、修正方向、合成白底并等比缩小后居中补白，不放大、不裁剪；固定一份、阈值 128、不反色，不隐式 FORM、切纸或下一标签定位。文本、条码等由 App 先绘成图片。

`setPaperType` 以独立设备请求采用 `GAP_LABEL`／`BLACK_MARK`／`CONTINUOUS`，不隐式校准或持久化；`locateNextLabel` 单独定位。连接后空闲时可 `setPrinterModel`，结果以成功回调返回的 `ConnectionInfo` 为准。型号选择只是本地配置，不是硬件认证。

## 请求与生命周期

请求立即返回 `RequestId`，唯一终态通过主线程异步 `Callback.onComplete` 交付。App 持有 Manager 和请求身份；取消调用 `manager.cancel(requestId)`。`disconnect` 完成后可显式重连，所有者结束时异步 `close`，关闭后不能复用；不要同步等待主线程回调。

所有打印、Raw、纸型、定位和查询共享同一设备 FIFO。SDK 不自动重连、续打或重放；失败与取消也可能已有输出。`DATA_SENT` 只表示数据传输完成，不能推断纸面完成。`capabilities` 保留三态支持程度，状态结果的缺纸／开盖／纸将尽／切刀错误保持可空值；未知不代表正常。

Flutter 通过标准 codec 将完整 PNG/JPEG `Uint8List` 映射为 Android `byte[]`，插件直接调用编码入口；不需要 Base64 或裸 RGBA。文件选择、缓存、页面绑定和后台策略仍由 App／插件管理。

软件分发与消费者检查不等于真实无线、纸面或峰值内存验收，`hardware_verified=false`。SDK 不公开底层作业、图片准备对象、平台句柄或队列控制接口。
